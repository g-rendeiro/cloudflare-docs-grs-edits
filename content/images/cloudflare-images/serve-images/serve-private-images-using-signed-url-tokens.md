---
pcx_content_type: how-to
title: Serve private images
layout: single
meta:
  title: Serve private images using signed URL tokens
weight: 3
---

# Serve private images using signed URL tokens

If an image is marked to require a signed URL, it cannot be accessed without a token unless it is being requested for a variant that is set to always allow public access.

1. Log in to the [Cloudflare dashboard](https://dash.cloudflare.com/login), and select your account.
2. Select **Images** > **Keys**.
3. Copy your key and use it to generate an expiring tokenized URL. 

Here is an example Worker that takes in a regular URL without a signed token and returns a tokenized URL that expires after one day. You can, however, set this expiration period to whatever you need, by changing the `const EXPIRATION` value.

```js
const KEY = 'YOUR_KEY_FROM_IMAGES_DASHBOARD';
const EXPIRATION = 60 * 60 * 24; // 1 day

const bufferToHex = buffer =>
  [...new Uint8Array(buffer)].map(x => x.toString(16).padStart(2, '0')).join('');

async function generateSignedUrl(url) {
  // `url` is a full imagedelivery.net URL
  // e.g. https://imagedelivery.net/cheeW4oKsx5ljh8e8BoL2A/bc27a117-9509-446b-8c69-c81bfeac0a01/mobile

  const encoder = new TextEncoder();
  const secretKeyData = encoder.encode(KEY);
  const key = await crypto.subtle.importKey(
    'raw',
    secretKeyData,
    { name: 'HMAC', hash: 'SHA-256' },
    false,
    ['sign']
  );

  // Attach the expiration value to the `url`
  const expiry = Math.floor(Date.now() / 1000) + EXPIRATION;
  url.searchParams.set('exp', expiry);
  // `url` now looks like
  // https://imagedelivery.net/cheeW4oKsx5ljh8e8BoL2A/bc27a117-9509-446b-8c69-c81bfeac0a01/mobile?exp=1631289275

  const stringToSign = url.pathname;
  // for example, /cheeW4oKsx5ljh8e8BoL2A/bc27a117-9509-446b-8c69-c81bfeac0a01/mobile

  // Generate the signature
  const mac = await crypto.subtle.sign('HMAC', key, encoder.encode(stringToSign));
  const sig = bufferToHex(new Uint8Array(mac).buffer);

  // And attach it to the `url`
  url.searchParams.set('sig', sig);

  return new Response(url);
}

addEventListener('fetch', event => {
  const url = new URL(event.request.url);
  const imageDeliveryURL = new URL(
    url.pathname.slice(1).replace('https:/imagedelivery.net', 'https://imagedelivery.net')
  );
  event.respondWith(generateSignedUrl(imageDeliveryURL));
});
```
