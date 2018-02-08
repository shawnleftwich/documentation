# REST API

Very little information is required to actually create objects in
HubTran, often just a name and external_id. However, in order for
HubTran to work well, we need as much information as you can give us.

All dates + times should be in
[iso8601](https://en.wikipedia.org/wiki/ISO_8601) format.

* [Authentication](../rest_authentication.md)
* [Create + Update Debtors](#create--update-debtors)
* [Bulk Create + Update Debtors](#bulk-create--update-debtors)
* [Invoice Purchased](#invoice-purchased)

## Create + Update Debtors

PUT https://api.hubtran.com/tms/debtors/:external_id

```
curl -X PUT https://api.hubtran.com/tms/debtors/example_id \
  -H "Content-Type: application/json" \
  -H "Authorization: Token token=YOUR_TOKEN" \
  -d '{"debtor": {"name": "example name", "external_id": "example_id"}}'
```

Request:

```
{
  "debtor": {
    "name": "debtor name", // Required
    "external_id": "external-id", // Required
    "active": true,
    "remit_to": {
      "name": "Address name",
      "address_line_1": "Address line 1",
      "address_line_2": "Address line 2",
      "city": "City",
      "state": "State",
      "postal_code": "Postal code",
      "country": "Country",
      "contact": {
        "email": "joe@example.com"
      }
    }
  }
}
```

Response:

```
{
  "debtor": {
    "id": 1, // HubTran's internal id for the debtor
    "external_id": "debtor-external-id" // YOUR internal id for the debtor
  }
}

```

## Bulk Create + Update Debtors

PUT https://api.hubtran.com/tms/debtors/bulk

```
curl -X PUT https://api.hubtran.com/tms/debtors/bulk \
  -H "Content-Type: application/json" \
  -H "Authorization: Token token=YOUR_TOKEN" \
  -d '{"debtors": [{"name": "example name", "external_id": "example_id"}]}'
```

Request:

See [Create + Update Debtors](#create--update-debtors) for the rest of
the supported debtor params.

```
{
  "debtors": [
    {
      "name": "debtor name", // Required
      "external_id": "external-id" // Required
    }
  ]
}
```

Response:

```
{
  "debtors": [
    {
      "id": 1, // HubTran's internal id for the debtor
      "external_id": "debtor-external-id", // YOUR internal id for the debtor
      "errors": {} // If this debtor had errors and couldn't be updated, they'll be here
    }
  ]
}

```

## Invoice Purchased

PUT https://api.hubtran.com/tms/invoices/:hubtran_id/purchase

```
curl -X PUT https://api.hubtran.com/tms/invoices/example_id/purchase \
  -H "Content-Type: application/json" \
  -H "Authorization: Token token=YOUR_TOKEN" \
  -d '{"ignored": "true"}'
```

Request:

The `hubtran_id` in the URL is the value of the `id` field in the payload we send you.

Response:

Empty JSON on success, error status code on failure

