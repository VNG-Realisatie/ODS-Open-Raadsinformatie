# Upgrade JSON-HAL to JSON-LD

[Joep Meindertsma, ](https://twitter.com/@joepmeindertsma)29 Aug 2018

[JSON-HAL](http://stateless.co/hal_specification.html) and [JSON-LD](https://json-ld.org/) are pretty popular formats for desiging APIs. They both have [their merits](https://sookocheff.com/post/api/on-choosing-a-hypermedia-format/), but they aren’t compatible with each other. However, with some additions, we can turn JSON-HAL into valid JSON-LD, without changing any of the original keys. This way, you can keep compatibility with HAL clients and benefit from the [advantages of linked data](http://ontola.io/what-is-linked-data), such as conversion to other RDF formats.

Let’s start with a JSON-HAL body object from the [docs](https://tools.ietf.org/html/draft-kelly-json-hal-08).

```
{
  "_links": {
    "self": { "href": "/orders/523" },
    "warehouse": { "href": "/warehouse/56" },
    "invoice": { "href": "/invoices/873" }
  },
  "currency": "USD",
  "status": "shipped",
  "total": 10.20
}
```

To identify the main resource, we add an `@id` value:

```
{
  "@id": "https://example.com/orders/123",
  "_links": {
    "self": { "href": "/orders/523" },
    "warehouse": { "href": "/warehouse/56" },
    "invoice": { "href": "/invoices/873" }
  },
  "currency": "USD",
  "status": "shipped",
  "total": 10.20
}
```

Let’s add an `@context` object to map our keys to URIs:

```
{
  "@id": "https://example.com/orders/123",
  "@context": {
    "currency": "https://example.com/ns/shipping#currency",
    "status": "https://example.com/ns/shipping#status",
    "total": "https://example.com/ns/shipping#total",
    "warehouse": "https://example.com/ns/shipping#warehouse",
    "invoice": "https://example.com/ns/shipping#invoice"
  },
  "_links": {
    "self": { "href": "/orders/523" },
    "warehouse": { "href": "/warehouse/56" },
    "invoice": { "href": "/invoices/873" }
  },
  "currency": "USD",
  "status": "shipped",
  "total": 10.20
}
```

If we try to [parse this as JSON-LD](https://json-ld.org/playground/#startTab=tab-nquads&json-ld={"%40id"%3A"https%3A%2F%2Fexample.com%2Forders%2F123"%2C"%40context"%3A{"currency"%3A"https%3A%2F%2Fexample.com%2Fns%2Fshipping%23currency"%2C"status"%3A"https%3A%2F%2Fexample.com%2Fns%2Fshipping%23status"%2C"total"%3A"https%3A%2F%2Fexample.com%2Fns%2Fshipping%23total"%2C"warehouse"%3A"https%3A%2F%2Fexample.com%2Fns%2Fshipping%23warehouse"%2C"invoice"%3A"https%3A%2F%2Fexample.com%2Fns%2Fshipping%23invoice"}%2C"_links"%3A{"self"%3A{"href"%3A"%2Forders%2F523"}%2C"warehouse"%3A{"href"%3A"%2Fwarehouse%2F56"}%2C"invoice"%3A{"href"%3A"%2Finvoices%2F873"}}%2C"currency"%3A"USD"%2C"status"%3A"shipped"%2C"total"%3A10.2}), we only get three valid triples. That’s because the items in `_links` are not yet parsed. To do this, we need to let the JSON-LD parser know three things:

- The object in `_links` describes the same resource as the root object. We do this by adding an `@id` value in the `_links` object with a value identical to the `@id` of the root object.
- The `href` values in the objects in `_links` are actually the URI values of these properties. We do this by adding an2 `@context` object to the `_links`body that binds `href` to `@id`.
- The `_links` object contains useful information, don’t skip it, parser! For the JSON-LD playground parser, we can do this by mapping `_links` to some arbitrary URL. Since documenting such a hacky solution might be useful, let’s link to this article. This is the most hacky part of my solution, so let me know if you find something better!

```
{
  "@id": "https://example.com/orders/123",
  "@context": {
    "currency": "https://example.com/ns/shipping#currency",
    "status": "https://example.com/ns/shipping#status",
    "total": "https://example.com/ns/shipping#total",
    "warehouse": "https://example.com/ns/shipping#warehouse",
    "invoice": "https://example.com/ns/shipping#invoice",
    "_links": "https://ontola.io/blog/json-hal-ld"
  },
  "_links": {
    "@id": "https://example.com/orders/123",
    "@context": { "href": "@id" },
    "self": { "href": "/orders/523" },
    "warehouse": { "href": "/warehouse/56" },
    "invoice": { "href": "/invoices/873" }
  },
  "currency": "USD",
  "status": "shipped",
  "total": 10.20
}
```

If we [parse this as JSON-LD](https://json-ld.org/playground-dev/#startTab=tab-nquads&json-ld={"%40id"%3A"https%3A%2F%2Fexample.com%2Forders%2F123"%2C"%40context"%3A{"currency"%3A"https%3A%2F%2Fexample.com%2Fns%2Fshipping%23currency"%2C"status"%3A"https%3A%2F%2Fexample.com%2Fns%2Fshipping%23status"%2C"total"%3A"https%3A%2F%2Fexample.com%2Fns%2Fshipping%23total"%2C"warehouse"%3A"https%3A%2F%2Fexample.com%2Fns%2Fshipping%23warehouse"%2C"invoice"%3A"https%3A%2F%2Fexample.com%2Fns%2Fshipping%23invoice"%2C"_links"%3A"https%3A%2F%2Fontola.io%2Fblog%2Fjson-hal-ld"}%2C"_links"%3A{"%40id"%3A"https%3A%2F%2Fexample.com%2Forders%2F123"%2C"%40context"%3A{"href"%3A"%40id"}%2C"self"%3A{"href"%3A"%2Forders%2F523"}%2C"warehouse"%3A{"href"%3A"%2Fwarehouse%2F56"}%2C"invoice"%3A{"href"%3A"%2Finvoices%2F873"}}%2C"currency"%3A"USD"%2C"status"%3A"shipped"%2C"total"%3A10.2}), we get all six triples:

```
<https://example.com/orders/123> <https://example.com/ns/shipping#currency> "USD" .
<https://example.com/orders/123> <https://example.com/ns/shipping#invoice> <https://json-ld.org/invoices/873> .
<https://example.com/orders/123> <https://example.com/ns/shipping#status> "shipped" .
<https://example.com/orders/123> <https://example.com/ns/shipping#total> "1.02E1"^^<http://www.w3.org/2001/XMLSchema#double> .
<https://example.com/orders/123> <https://example.com/ns/shipping#warehouse> <https://json-ld.org/warehouse/56> .
<https://example.com/orders/123> <https://ontola.io/blog/json-hal-ld> <https://example.com/orders/123> .
```

Awesome! Just by adding a few keys, we converted JSON-HAL into JSON-LD, without breaking anything. However, the resulting object is not that pretty and quite verbose. Only use this is you really *have* to support both JSON-HAL and JSON-LD. If you haven’t invested in JSON-HAL yet, but do need a linked data API, I recommend you go with JSON-LD plus [Hydra](https://www.hydra-cg.com/spec/latest/core/), which also offers standardization for things like actions, collections and pagination.

## TL;DR

- Add an `@id` to both the root object and the `_links` object with the main resouce URL.
- In the root `@context`, add your usual JSON-LD mapping.
- In the root `@context`, map`_links` to some URL.
- In the `_links` `@context`, map `href` to `@id`.
