---
pcx_content_type: configuration
title: Block Microsoft Exchange Autodiscover requests
---

# Block Microsoft Exchange Autodiscover requests

In some cases, Microsoft Exchange Autodiscover service requests can be "noisy", triggering large numbers of `HTTP 404` (`Not found`) errors.

This example blocks requests for `autodiscover.xml` and `autodiscover.src`:

<table>
  <thead>
    <tr>
      <th>Expression</th>
      <th style="width:20%">Action</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <code>(ends_with(http.request.uri.path, "/autodiscover.xml") or ends_with(http.request.uri.path, "/autodiscover.src"))</code>
      </td>
      <td>
        <em>Block</em>
      </td>
    </tr>
  </tbody>
</table>

Alternatively, customers on a Business or Enterprise plan can use the `matches` [comparison operator](/ruleset-engine/rules-language/operators/#comparison-operators) for the same purpose. For this example, the expression would be the following:

```txt
(http.request.uri.path matches "/autodiscover.(xml|src)$")
```