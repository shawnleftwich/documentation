# REST API

Very little information is required to actually create objects in
HubTran, often just a name and external_id. However, in order for
HubTran to work well, we need as much information as you can give us.

## Authentication

In order to authenticate you need to send an Authorization header with the token associated with your account. It should look something like:

```
Authorization: Token token=YOUR_TOKEN
```

Email support@hubtran.com to get your API token.

## Create + Update Loads

Below you'll see we have "load_id" and "external_id". "load_id" is what the load would be called in the user interface of your application.  Often it is unique and immutable but some TMSs do not have that guarantee. "external_id" is YOUR internal id for the load. "external_id" should always be unique and immutable.

PUT https://api.hubtran.com/tms/loads/:load_id

```
curl -X PUT https://api.hubtran.com/tms/loads/example_id \
  -H "Content-Type: application/json" \
  -H "Authorization: Token token=YOUR_TOKEN" \
  -d '{"load": {"load_id": "example_id", "external_id": "example_id"}}'
```

Request:

```
{
  "load": {
    "load_id": "load-id", // Required
    "external_id": "external-id", // Required
    "actual_delivered_at": "2016-07-17 19:00:00 +0200",
    "pro_number": "pro-number",
    "primary_reference": "primary-reference",
    "primary_reference_type": "primary-reference-type",
    "quantity": 1,
    "weight": 1000,
    "status": "new",
    "require_pod": false,
    "origin": {
      "name": "origin-name",
      "address_line_1": "origin-address-line-1",
      "address_line_2": "origin-address-line-2",
      "city": "origin-city",
      "state": "origin-state",
      "postal_code": "origin-postal-code",
      "country": "origin-country"
    },
    "destination": {
      "name": "destination-name",
      "address_line_1": "destination-address-line-1",
      "address_line_2": "destination-address-line-2",
      "city": "destination-city",
      "state": "destination-state",
      "postal_code": "destination-postal-code",
      "country": "destination-country"
    },
    "carrier": {
      "name": "carrier-name",
      "scac": "carrier-scac",
      "external_id": "carrier-external-id"
    },
    "charges": {
      "total": 1200.23,
      "currency": "USD",
      "line_haul": 1100.32,
      "fuel": 129.32,
      "detention": 22.22,
      "other": 87.87
    },
    "tms_created_at": "2016-07-15 19:00:00 +0200",
    "distance": "500",
    "mode": "truck",
    "freight_class": "class",
    "po": "po",
    "pickup_reference": "pickup-reference",
    "memo": "memo",
    "tender_user": "jim",
    "tms_updated_at": "2016-07-17 19:00:00 +0200",
    "delivery_number": "delivery-number",
    "ppe": "ppe",
    "quote_number": "quote-number",
    "ship_ref": "ship-ref",
    "customer": {
      "external_id": "customer-external-id",
      "name": "customer-name",
      "account_number": "customer-account-number"
    },
    "shipments": [ // Used to link the load to shipments created via the shipment API
      {
        "external_id": "shipment-external-id"
      }
    ],
    "line_items": [ // Displayed and used to calculate the carrier_charge.
      {
        "description": "line item description",
        "total": 123.45,
        "quantity": 5,
        "carrier": { // Used to calculate carrier_charge for each carrier
          "external_id": "carrier-external-id",
          "scac": "scac",
          "name": "carrier name"
        }
      }
    ]
  }
}
```

Response:

```
{
  "load": {
    "id": 5 // HubTran's internal id for the load
  }
}
```

## Create + Update Carriers

PUT https://api.hubtran.com/tms/carriers/:external_id

```
curl -X PUT https://api.hubtran.com/tms/carriers/example_id \
  -H "Content-Type: application/json" \
  -H "Authorization: Token token=YOUR_TOKEN" \
  -d '{"carrier": {"name": "example name", "external_id": "example_id"}}'
```

Request:

```
{
  "carrier": {
    "name": "carrier name", // Required
    "external_id": "external-id", // Required
    "external_vendor_id": "evid", // A reference the user will use to make sure it matches TMS
    "scac": "scac",
    "mc_number": "mc_number",
    "dot_number": "dot_number",
    "ein": "ein",
    "address_name": "Address name",
    "address_line_1": "Address line 1",
    "address_line_2": "Address line 2",
    "city": "City",
    "state": "State",
    "postal_code": "Postal code",
    "country": "Country",
    "remit_to": {
      "name": "Address name",
      "address_line_1": "Address line 1",
      "address_line_2": "Address line 2",
      "city": "City",
      "state": "State",
      "postal_code": "Postal code",
      "country": "Country"
    },
    "contacts": [ // Helps us associate carriers and incoming invoices
      "joe@example.com",
      "tom@example.com"
    ]
  }
}
```

Response:

```
{
  "carrier": {
    "id": 1, // HubTran's internal id for the carrier
    "external_id": "carrier-external-id" // YOUR internal id for the carrier
  }
}

```

## Create + Update Shipments

POST https://api.hubtran.com/tms/shipments

```
curl -X POST https://api.hubtran.com/tms/shipments \
  -H "Content-Type: application/json" \
  -H "Authorization: Token token=YOUR_TOKEN" \
  -d '{"shipments": [{"external_id": "example_id"}]}'
```

Request:

```
{
  "shipments": [
    {
      "external_id": "1234", // Required
      "origin": {
        "code": "origin-code",
        "name": "origin-name",
        "address_line_1": "origin-address-line-1",
        "address_line_2": "origin-address-line-2",
        "city": "origin-city",
        "state": "origin-state",
        "postal_code": "origin-postal-code",
        "country": "origin-country"
      },
      "destination": {
        "code": "destination-code",
        "name": "destination-name",
        "address_line_1": "destination-address-line-1",
        "address_line_2": "destination-address-line-2",
        "city": "destination-city",
        "state": "destination-state",
        "postal_code": "destination-postal-code",
        "country": "destination-country"
      },
      "actual_delivered_at": "2016-07-17 20:43:00 +0300",
      "quantity": 1,
      "weight": 1000,
      "status": "new",
      "created_at": "2016-07-15 20:43:00 +0300",
      "customer_mode": "FSC and Rate Review",
      "type": "REGULAR",
      "owner": "Owner name",
      "po": "po",
      "pickup_reference": "pickup-reference",
      "charges": {
        "total": 1200.23,
        "currency": "USD",
        "line_haul": 1100.32,
        "fuel": 129.32,
        "detention": 22.22,
        "other": 87.87
      },
      "ship_ref": "C16091",
      "loads": [ // Used to link shipments and loads representing a "delivery"
        {
          "external_id": "load-external-id"
        }
      ]
    }
  ]
}
```

Response:

```
{
  "shipments": [
    {
      "id": 1, // HubTran's internal id for the shipment
      "external_id": "shipment-external-id" // YOUR internal id for the shipment
    }
  ]
}
```
