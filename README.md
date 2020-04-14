# API Documentation

You might want to take a look at the [high level overview](high_level_overview.pdf) before digging into the more detailed documentation.

## Brokers

* [REST](brokers/rest.md)
* [Webhook](brokers/webhook.md)
* [CSV](brokers/csv.md)

## Factors

* [REST](factors/rest.md)
* [Webhook](factors/webhook.md)

## General Notes

In HubTran's terminology, a "load" represents the physical truck move and a "shipment" represents the unit of cargo that is picked up and
delivered. A load may have one shipment or multiple shipments. A shipment may move on one load or multiple loads. A load is billed on the carrier invoice. A shipment is billed on the broker invoice to its customer. If your TMS is not structured around the concept of loads and shipments, HubTran can map to your structure.

You'll also see references to "external_id". This is our way of linking objects in HubTran with objects in your TMS so it should be YOUR internal id for the object. It should be unique and immutable.

At various times we'll add more keys to our webhook payloads that might not be documented. Your implementation should be able to handle us adding keys without notice. However, we'll never remove a key from the payload without giving you plenty of notice.

## Document Types

We've also created a [document types endpoint](./brokers/rest.md#document-types) which returns all the
document type options as well as the ones currently enabled for your
account. These will help you when you are mapping our document types
in our webhook to your document storage types.

## Fetching Load Documents via the List Transmissions Endpoint
Load documents can be fetched from the [List Transmisions endpoint](./brokers/rest.md#list-transmissions).

Each transmission retrieved from the endpoint will have one or more `load_documents`. These `load_documents` are very much like the `documents` currently included in the [List Approved Invoices endpoint](./brokers/rest.md#list-approved-invoices) except they don't include the unused `visibility` field.

We have also added a `fingerprint` field to both the `load_documents` in _List Transmissions_ and the `documents` in _List Approved Invoices_. When a document is included in multiple transmissions, a change to the fingerprint indicates that the file contents have changed and should be downloaded again.

### Transmission Types
Currently, only `load_document_attachment_v1` transmissions will be made available in the _List Transmissions_ endpoint, but we anticipate adding more transmission types in the future. For this reason, we recommend always including the type parameter in requests to the _List Transmissions_ endpoint like this:

```
GET https://api.hubtran.com/tms/transmissions?type=load_documents_attached_v1
```

This way you won't receive other transmissions and payloads in the response when they're added in the future.

### Verifying Transmissions
The _List Transmissions_ endpoint returns the transmissions that have not yet been verified as having been received and processed by the client. So if you query the endpoint multiple times, you'll get the same transmissions over and over again (they are ordered oldest to newest.)

In order to verify that you have received and processed a transmission (and to remove it from the results returned in the endpoint), you have to POST the `transmission_id` to the [Mark Transmision as Verified endpoint](./brokers/rest.md#mark-transmission-as-verified).

### Pagination
Though the _List Transmissions_ endpoint supports pagination, we anticipate that the easiest way to integrate with it will simply be to fetch, process, and verify the first page of transmissions. This first page of results will continuously update as you verify transmissions.

