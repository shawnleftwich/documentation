# REST API

Very little information is required to actually create objects in
TriumphPay Audit, often just a name and external_id. However, in order for
TriumphPay Audit to work well, we need as much information as you can give us.

All dates + times should be in
[iso8601](https://en.wikipedia.org/wiki/ISO_8601) format. All `external_id`s should be strings.

If you receive a status code other than 2xx, please retry your request once with a slight delay - there may be a temporary issue that resolves itself upon retry.

* [Authentication](../rest_authentication.md)
* [Create + Update Loads](#create--update-loads)
* [Load Details](#load-details)
* [Create + Update Shipments](#create--update-shipments)
* [Create + Update Carriers](#create--update-carriers)
* [Create + Update Customers](#create--update-customers)
* [Create + Update Customer Invoices](#create--update-customer-invoices)
* [Create Payments](#create-payments)
* [Clear Exceptions](#clear-exceptions)
* [List Approved Invoices](#list-approved-invoices)
* [Mark Approved Invoice as Verified](#mark-approved-invoice-as-verified)
* [Mark Approved Invoice as Not Verified](#mark-approved-invoice-as-not-verified)
* [Create Carrier Invoice Submissions](#create-carrier-invoice-submissions)
* [Create Documents Only Carrier Invoice Submissions](#create-documents-only-carrier-invoice-submissions)
* [List Transmissions](#list-transmissions)
* [Mark Transmission as Verified](#mark-transmission-as-verified)
* [Update Account](#update-account)
* [Document Types](#document-types)

## Create + Update Loads

Below you'll see we have "load_id" and "external_id". "load_id" is what the load would be called in the user interface of your application.  Often it is unique and immutable but some TMSs do not have that guarantee. "external_id" is YOUR internal id for the load. "external_id" should always be unique and immutable.

PUT https://api.audit.triumphpay.com/tms/loads/:load_id

```
curl -X PUT https://api.audit.triumphpay.com/tms/loads/example_id \
  -H "Content-Type: application/json" \
  -H "Authorization: Token token=YOUR_TOKEN" \
  -d '{"load": {"load_id": "example_id", "external_id": "example_id"}}'
```

### Request

```
{
  "load": {
    "load_id": "load-id",                                 // Required, human-friendly, but for backwards compatibility, will default to external_id
    "external_id": "external-id",                         // Required, machine-friendly
    "status": "new",                                      // Recommended
    "brokered": true,
    "tms_created_at": "2016-07-15 19:00:00 +0200",
    "tms_updated_at": "2016-07-17 19:00:00 +0200",
    "target_ship_start": "2016-07-16 12:00:00 +0200",
    "target_ship_end": "2016-07-16 14:00:00 +0200",
    "actual_shipped_at": "2016-07-16 13:00:00 +0200",     // Recommended
    "target_delivery_start": "2016-07-17 19:00:00 +0200",
    "target_delivery_end": "2016-07-17 19:00:00 +0200",
    "actual_delivered_at": "2016-07-17 19:00:00 +0200",   // Recommended
    "require_pod": false,
    "require_weight_ticket": false,
    "require_customer_rate_confirmation": false,
    "require_originals": false,
    "pro_number": "pro-number",
    "quantity": 1.0,
    "weight": 1000,
    "distance": 500,
    "mode": "truck",
    "freight_class": "class",
    "memo": "memo",
    "delivery_number": "delivery-number",
    "ppe": "ppe",
    "quote_number": "quote-number",
    "load_note": "load was stolen",
    "payables_note": "do not pay",
    "quickpay": true,
    "days_to_pay": 2,                                     // Used with quickpay to specify terms. Leave empty if you don't have multiple quickpay terms
    "office": "Chicago",
    "sales_rep": "Randy",
    "business_unit": "Managed Trans",
    "priority": "Tier 1",
    "blind": false,
    "origin": {                                           // Recommended
      "name": "origin-name",
      "address_line_1": "origin-address-line-1",
      "address_line_2": "origin-address-line-2",
      "city": "origin-city",
      "state": "origin-state",
      "postal_code": "origin-postal-code",
      "country": "origin-country"
    },
    "destination": {                                      // Recommended
      "name": "destination-name",
      "address_line_1": "destination-address-line-1",
      "address_line_2": "destination-address-line-2",
      "city": "destination-city",
      "state": "destination-state",
      "postal_code": "destination-postal-code",
      "country": "destination-country"
    },
    "carrier": {                                          // Recommended
      "name": "carrier-name",
      "scac": "carrier-scac",
      "external_id": "carrier-external-id",
      "dot_number": "dot number",
      "mc_number": "mc number"
    },
    "customer": {                                         // Recommended
      "external_id": "customer-external-id",
      "name": "customer-name",
      "account_number": "customer-account-number"
    },
    "shipments": [                                        // Used to link the load to shipments created via the shipment API
      // Shipments will be created if they don't exist
      {
        "shipment_id": "shipment-id",
        "external_id": "shipment-external-id"
      }
    ],
    "charges": {                                          // Recommended but ignored if line items are sent
      "total": 1200.23,
      "currency": "USD",
      "customer_total": 1500.00
    },
    "picks": [
      {
        "external_id": "pick-external-id",                // Required
        "name": "pick-name",
        "address_line_1": "pick-address-line-1",
        "address_line_2": "pick-address-line-2",
        "city": "pick-city",
        "state": "pick-state",
        "postal_code": "pick-postal-code",
        "country": "pick-country",
        "shipments": [                                    // Required
          {
            "external_id": "shipment-external-id"
          }
        ]
      }
    ],
    "drops": [
      // same format as picks
    ],
    "line_items": [                                       // Recommended, displayed and used to calculate the carrier_charge.
      {
        "external_id": "line_item_1",                     // You're id for matching up line items when sending back to your TMS
        "description": "line item description",
        "total": 123.45,                                  // Required
        "customer_total": 150.00,
        "quantity": 5.0,
        "rate": 2.0,
        "rate_qualifier": "PM",                           // Optional, e.g. flat or per-mile
        "type_code": "abc",                               // Optional, i.e. the charge code or EDI code
        "carrier": {                                      // Used to calculate carrier_charge for each carrier
          "external_id": "carrier-external-id",
          "scac": "scac",
          "name": "carrier name"
        }
      }
    ],
    "shipped_items": [{
      "external_id": "1234",                              // Required
      "description": "Shipped item",                      // Required
      "class_name": '400',
      "item_type": "CARTON",
      "nmfc": "82270",
      "hazardous_material": false,
      "quantity": 2.0,
      "weight": 417.0,
      "weight_unit": "lb",
      "width": 10.0,
      "height": 12.0,
      "length": 20.0
    }],
    "references": {                                       // Recommended
      "Example Reference Label 1": "Value1",
      "Example Reference Label 2": "Value2",
      // To send multiple references with the same label, comma-separate the values:
      "Example Reference Label 3": "Value3a, Value3b"
      ... as many label-value pairs as you want
    }
  }
}
```

### Response

#### Success

Same as the response for the [load details](#load-details) API.

#### Failure

##### 422 Unprocessable Entity

```
{
  "errors": [] // each element is a String describing an error
}
```

##### 423 Locked

```
{
  "errors": ["record locked"]
}
```

## Load Details

GET https://api.audit.triumphpay.com/tms/loads/:id

Where :id is the TriumphPay Audit id you previously saved.

If you don't have the TriumphPay Audit id you can alternatively use one of the
following forms:

GET https://api.audit.triumphpay.com/tms/loads?load_id=:your_load_id
GET https://api.audit.triumphpay.com/tms/loads?external_id=:your_load_external_id

load_id is what the user sees in the UI and external_id is what you
would have passed us as your internal, immutable id for the load.
Sometimes they are the same thing.

```
curl -X GET https://api.audit.triumphpay.com/tms/loads/example_id \
  -H "Content-Type: application/json" \
  -H "Authorization: Token token=YOUR_TOKEN"
```

### Response

#### Success

##### 200 Ok 

```
{
  "load": {
    "id": 5,                                            // TriumphPay Audit's internal id for the load
    "load_id": "load-id",                               // The load_id you sent us when creating the load
    "external_id": "external-id",                       // The external_id you sent us when creating the load
    "status": "new",
    "brokered": true,
    "tms_created_at": "2016-07-15 19:00:00 +0200",
    "tms_updated_at": "2016-07-17 19:00:00 +0200",
    "target_delivery_start": "2016-07-17 19:00:00 +0200",
    "target_delivery_end": "2016-07-17 19:00:00 +0200",
    "actual_delivered_at": "2016-07-17 19:00:00 +0200",
    "target_ship_start": "2016-07-16 12:00:00 +0200",
    "target_ship_end": "2016-07-16 14:00:00 +0200",
    "actual_shipped_at": "2016-07-16 13:00:00 +0200",
    "require_pod": false,
    "require_weight_ticket": false,
    "require_customer_rate_confirmation": false,
    "require_originals": false,
    "pro_number": "pro-number",
    "quantity": 1.0,
    "weight": 1000,
    "distance": 500,
    "mode": "truck",
    "freight_class": "class",
    "memo": "memo",
    "delivery_number": "delivery-number",
    "ppe": "ppe",
    "quote_number": "quote-number",
    "load_note": "load was stolen",
    "payables_note": "do not pay",
    "quickpay": true,
    "days_to_pay": 2,
    "office": "Chicago",
    "sales_rep": "Randy",
    "business_unit": "Managed Trans",
    "priority": "Tier 1",
    "blind": false,
    "origin": {                                 // null if all fields are empty
      "name": "origin-name",
      "address_line_1": "origin-address-line-1",
      "address_line_2": "origin-address-line-2",
      "city": "origin-city",
      "state": "origin-state",
      "postal_code": "origin-postal-code",
      "country": "origin-country"
    },
    "destination": {                            // null if all fields are empty
      "name": "destination-name",
      "address_line_1": "destination-address-line-1",
      "address_line_2": "destination-address-line-2",
      "city": "destination-city",
      "state": "destination-state",
      "postal_code": "destination-postal-code",
      "country": "destination-country"
    },
    "carrier": {                                // null if no carrier
      "name": "carrier-name",
      "scac": "carrier-scac",
      "external_id": "carrier-external-id",
      "dot_number": "dot number",
      "mc_number": "mc number"
    },
    "customer": {                               // null if no customer
      "external_id": "customer-external-id",
      "name": "customer-name",
      "account_number": "customer-account-number"
    },
    "shipments": [                              // Used to link the load to shipments created via the shipment API. Empty array if no shipments.
      {
        "external_id": "shipment-external-id"
      }
    ],
    "charges": {                                // object with null values if null
      "total": 1200.23,
      "currency": "USD",
      "customer_total": 1500.00
    },
    "picks": [                                  // empty array if no picks
      {
        "external_id": "pick-external-id",
        "name": "pick-name",
        "address_line_1": "pick-address-line-1",
        "address_line_2": "pick-address-line-2",
        "city": "pick-city",
        "state": "pick-state",
        "postal_code": "pick-postal-code",
        "country": "pick-country",
        "shipments": [
          {
            "external_id": "shipment-external-id"
          }
        ]
      }
    ],
    "drops": [                                  // empty array if no drops
      // same format as picks
    ],
    "line_items": [                             // Displayed and used to calculate the carrier_charge. Empty array if no line items.
      {
        "external_id": "line_item_1",           // You're id for matching up line items when sending back to your TMS
        "description": "line item description",
        "total": 123.45,
        "customer_total": 150.00,
        "quantity": 5.0,
        "rate": 2.0,
        "rate_qualifier": "PM",
        "type_code": "abc",
        "carrier": {                            // Used to calculate carrier_charge for each carrier
          "external_id": "carrier-external-id",
          "scac": "scac",
          "name": "carrier name"
        }
      }
    ],
    "shipped_items": [                          // empty array if no shipped items
      {
        "external_id": "1234",
        "description": "Shipped item",
        "class_name": '400',
        "item_type": "CARTON",
        "nmfc": "82270",
        "hazardous_material": false,
        "quantity": 2.0,
        "weight": 417.0,
        "weight_unit": "lb",
        "width": 10.0,
        "height": 12.0,
        "length": 20.0
      }
    ],
    "carrier_invoices": [                       // empty array if no invoices
      {
        "number": "invoice-number",
        "state": "approved",
        "date": "1981-08-11",
        "date_to_pay": "1981-08-13",
        "amount_to_pay": "1110.2",
        "currency": "USD",
        "quickpay": false,
        "approver": {
          "email": "test@example.com"
        }
        "carrier": {
          "external_id": "carrier-external-id"
        },
        "documents": [
          {
            "id": 14,
            "type": "billOfLading",
            "hubtran_type_name": "Bill of Lading / Packing Slip",
            "proof_of_delivery": true,
            "url": "https://api.audit.triumphpay.com/downloads/documents/unique-id",
            "visibility": {
              "carrier": true,
              "customer": true
            },
            "pages": [
              {
                "png_url": "https://api.audit.triumphpay.com/downloads/documents/unique-id/pages/xxx.png"
              }
            ]
          }
        ],
        "combined_document_urls": [             // All documents of the same type, combined
          {
            "type": "billOfLading",
            "hubtran_type_name": "Bill of Lading / Packing Slip",
            "url": "https://api.audit.triumphpay.com/downloads/documents/combined/unique-id",
            "proof_of_delivery": true,
            "visibility": {
              "carrier": true,
              "customer": true
            }
          }
        ],
        "remit_to": {
          "name": "name",
          "address_line_1": "address1",
          "address_line_2": "address2",
          "city": "city",
          "state": "state",
          "postal_code": "12345",
          "country": "USA"
        }
      }
    ],
    "references": {                             // empty object if no references
      "Example Reference Label 1": "Value1",
      "Example Reference Label 2": "Value2",
      "Example Reference Label 3": "Value3a, Value3b"
    },
    "documents": [             // Documents on the load
      {
        "id": 14,
        "type": "billOfLading",
        "hubtran_type_name": "Bill of Lading / Packing Slip",
        "proof_of_delivery": true,
        "url": "https://api.audit.triumphpay.com/downloads/documents/unique-id",
        "visibility": {
          "carrier": true,
          "customer": true
        },
        "pages": [
          {
            "png_url": "https://api.audit.triumphpay.com/downloads/documents/unique-id/pages/xxx.png"
          }
        ]
      }
    ],
  }
}
```

#### Failure

##### 404 Not Found

```
{
  "errors": "Not found"
}
```

## Create + Update Shipments

POST https://api.audit.triumphpay.com/tms/shipments

```
curl -X PUT https://api.audit.triumphpay.com/tms/shipments \
  -H "Content-Type: application/json" \
  -H "Authorization: Token token=YOUR_TOKEN" \
  -d '{"shipments": [{"external_id": "example_id"}]}'
```

### Request

```
{
  "shipments": [
    {
      "shipment_id": "shipment-id",                       // Required, human-friendly, but for backwards compatibility, will default to external_id
      "external_id": "external-id",                       // Required, machine-friendly
      "status": "new",                                    // Recommended
      "tms_created_at": "2016-07-10 20:43:00 +0300",
      "tms_updated_at": "2016-07-15 20:43:00 +0300",
      "target_ship_start": "2016-07-16 12:00:00 +0200",
      "target_ship_end": "2016-07-16 14:00:00 +0200",
      "actual_shipped_at": "2016-07-16 13:00:00 +0200",   // Recommended
      "target_delivery_start": "2016-07-17 19:00:00 +0200",
      "target_delivery_end": "2016-07-17 19:00:00 +0200",
      "actual_delivered_at": "2016-07-17 19:00:00 +0200", // Recommended
      "quantity": 1.0,
      "weight": 1000,
      "customer_mode": "FSC and Rate Review",
      "owner": "Owner name",
      "origin": {                                         // Recommended
        "name": "origin-name",
        "address_line_1": "origin-address-line-1",
        "address_line_2": "origin-address-line-2",
        "city": "origin-city",
        "state": "origin-state",
        "postal_code": "origin-postal-code",
        "country": "origin-country"
      },
      "destination": {                                    // Recommended
        "name": "destination-name",
        "address_line_1": "destination-address-line-1",
        "address_line_2": "destination-address-line-2",
        "city": "destination-city",
        "state": "destination-state",
        "postal_code": "destination-postal-code",
        "country": "destination-country"
      },
      "customer": {                                       // Recommended
        "external_id": "customer-external-id",
        "name": "customer-name",
        "account_number": "customer-account-number"
      },
      "loads": [                                          // Used to link shipments and loads representing a "delivery"
        // Loads will be created if they don't already exist
        {
          "load_id": "load-id",
          "external_id": "load-external-id"
        }
      ],
      "charges": {                                        // Recommended
        "total": 1200.23,
        "currency": "USD"
      },
      "line_items": [
        {
          "external_id": "line_item_1",                   // The line external_id you sent us
          "description": "line item description",
          "total": 123.45,                                // Required
          "quantity": 5.0,
          "rate": 2.0,
          "rate_qualifier": "PM",
          "type_code": "abc"
        }
      ],
      "shipped_items": [
        {
          "external_id": "1234",                          // Required
          "description": "Shipped item",                  // Required
          "class_name": '400',
          "item_type": "CARTON",
          "nmfc": "82270",
          "hazardous_material": false,
          "quantity": 2.0,
          "weight": 417.0,
          "weight_unit": "lb",
          "width": 10.0,
          "height": 12.0,
          "length": 20.0
        }
      ],
      "references": {                                     // Recommended
        "Example Reference Label 1": "Value1",
        "Example Reference Label 2": "Value2",
        // To send multiple references with the same label, comma-separate the values:
        "Example Reference Label 3": "Value3a, Value3b"
        ... as many label-value pairs as you want
      }
    }
  ]
}
```

### Response

#### Success

```
{
  "shipments": [
    {
      "id": 1,                                              // TriumphPay Audit's internal id for the shipment
      "external_id": "shipment-external-id"                 // YOUR internal id for the shipment
      "status": "new",
      "tms_created_at": "2016-07-10 20:43:00 +0300",
      "tms_updated_at": "2016-07-15 20:43:00 +0300",
      "target_ship_start": "2016-07-16 12:00:00 +0200",
      "target_ship_end": "2016-07-16 14:00:00 +0200",
      "actual_shipped_at": "2016-07-16 13:00:00 +0200",
      "target_delivery_start": "2016-07-17 19:00:00 +0200",
      "target_delivery_end": "2016-07-17 19:00:00 +0200",
      "actual_delivered_at": "2016-07-17 19:00:00 +0200",
      "quantity": 1.0,
      "weight": 1000,
      "customer_mode": "FSC and Rate Review",
      "owner": "Owner name",
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
      "loads": [                                            // Used to link shipments and loads representing a "delivery"
        {
          "external_id": "load-external-id"
        }
      ],
      "charges": {
        "total": 1200.23,
        "currency": "USD"
      },
      "line_items": [
        {
          "external_id": "line_item_1",                     // The line external_id you sent us
          "description": "line item description",
          "total": 123.45,
          "quantity": 5.0,
          "rate": 2.0,
          "rate_qualifier": "PM",
          "type_code": "abc"
        }
      ],
      "shipped_items": [
        {
          "external_id": "1234",          // Required
          "description": "Shipped item",  // Required
          "class_name": '400',
          "item_type": "CARTON",
          "nmfc": "82270",
          "hazardous_material": false,
          "quantity": 2.0,
          "weight": 417.0,
          "weight_unit": "lb",
          "width": 10.0,
          "height": 12.0,
          "length": 20.0
        }
      ],
      "references": {                       // empty object if no references
        "Example Reference Label 1": "Value1",
        "Example Reference Label 2": "Value2",
        "Example Reference Label 3": "Value3a, Value3b"
      }
    }
  ]
}
```

#### Failure

##### 422 Unprocessable Entity

```
{
  "errors": [] // each element is a String describing an error
}
```

## Create + Update Carriers

PUT https://api.audit.triumphpay.com/tms/carriers/:external_id

```
curl -X PUT https://api.audit.triumphpay.com/tms/carriers/example_id \
  -H "Content-Type: application/json" \
  -H "Authorization: Token token=YOUR_TOKEN" \
  -d '{"carrier": {"name": "example name", "external_id": "example_id"}}'
```

Request:

```
{
  "carrier": {
    "name": "carrier name",               // Required
    "external_id": "external-id",         // Required
    "external_vendor_id": "evid",         // A reference the user will use to make sure it matches TMS
    "active": true,                       // Recommended
    "scac": "scac",                       // Recommended
    "mc_number": "mc_number",             // Recommended
    "dot_number": "dot_number",
    "ein": "ein",
    "address_name": "Address name",       // Recommended
    "address_line_1": "Address line 1",   // Recommended
    "address_line_2": "Address line 2",   // Recommended
    "city": "City",                       // Recommended
    "state": "State",                     // Recommended
    "postal_code": "Postal code",         // Recommended
    "country": "Country",                 // Recommended
    "account_exec": "Bob",
    "priority": "Tier 1",
    "days_to_pay": 10,                    // Recommended
    "quickpay": false,                    // Optional. Defaults to false.
    "payment_strategy": "direct",         // direct or triumphpay, defaults to direct
    "labels": ["LABEL1","LABEL2"],        // Omit this key unless you want to overwrite existing labels set through the API
    "contacts": [                         // Recommended
      "joe@example.com",
      "tom@example.com"
    ],
    "remit_to": {                         // Recommended, should be
factor remit to if carrier is factored
      "name": "Address name",             // Required if remit_to block
is sent
      "address_line_1": "Address line 1",
      "address_line_2": "Address line 2",
      "city": "City",
      "state": "State",
      "postal_code": "Postal code",
      "country": "Country"
    }
  }
}
```

### Response

#### Success

```
{
  "carrier": {
    "id": 1,                                // TriumphPay Audit's internal id for the carrier
    "external_id": "carrier-external-id",   // YOUR internal id for the carrier
    "external_vendor_id": "evid",
    "active": true,
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
    "account_exec": "Bob",
    "days_to_pay": 10,
    "quickpay": false,                      //If you don’t pass this key you’ll still get it back with the accurate value in our system
    "payment_strategy": "direct",
    "labels": ["LABEL1","LABEL2"],          // Empty list if there aren't any
labels
    "contacts": [                           // Empty list if there aren't any contacts
      "joe@example.com",
      "tom@example.com"
    ],
    "remit_to": {                           // null if there is no remit to address
      "name": "Address name",
      "address_line_1": "Address line 1",
      "address_line_2": "Address line 2",
      "city": "City",
      "state": "State",
      "postal_code": "Postal code",
      "country": "Country"
    }
  }
}
```

#### Failure

##### 422 Unprocessable Entity

```
{
  "errors": { "name": ["can't be blank"], "external_id": ["can't be blank"] } // This is just an example payload. It will have this format though.
}
```

##### 423 Locked

```
{
  "errors": ["record locked"]
}
```

##### 400 Bad Request

```
{
  "errors": ["ID in URL does not match the external_id in the body"]
}
```

## Create + Update Customers

PUT https://api.audit.triumphpay.com/tms/customers/:external_id

```
curl -X PUT https://api.audit.triumphpay.com/tms/customers/example_id \
  -H "Content-Type: application/json" \
  -H "Authorization: Token token=YOUR_TOKEN" \
  -d '{"customer":  {...}}'
```

### Request

```
{
  "customer": {
    "external_id": "customer-external-id",                      // Required
    "name": "customer-name",                                    // Required
    "account_number": "customer-account-number",                // Required
    "carrier_invoice_reminders": [                              // Optional
      "Confirm with Sandy before approving.",
      "Check this other thing before approving."
    ],
    "invoicing": {
      "bill_to": {
        "name": "name",
        "address_line_1": "address1",
        "address_line_2": "address2",
        "city": "city",
        "state": "state",
        "postal_code": "12345",
        "country": "US"
      },
      "billing_interval": "weekly",                            // Optional. One of "daily", "weekly", "monthly", "never". Defaults to "daily".
      "trigger": "scheduled",                                  // Optional. One of "scheduled", "manual", or "default".
      "scheduled_day": 4,                                      // Optional. Defaults to 1. See "About the scheduled_day setting" section below.
      "document_visibility": {                                 // Optional. Leaves settings unchanged if omitted.
        "use_account_defaults": false,                         // Required.
        "include_pod": true,                                   // Optional. Required if use_account_defaults is false. Will include pod document even if the document type is not included below.
        "include_document_types": [                            // Optional. Required if use_account_defaults is false.
          "customerInvoice",                                   // Use document_types endpoints to find possible values.
          "billOfLading"
        ]
      },
      "document_requirements": {                               // Optional. Leaves settings unchanged if omitted. Only customer-visible documents may be required.
        "use_account_defaults": false,                         // Required.
        "require_pod": true,                                   // Optional. Required if use_account_defaults is false.
        "require_document_types": [                            // Optional. Required if use_account_defaults is false.
          "customerInvoice",                                   // Use document_types endpoints to find possible values.
          "billOfLading"
        ]
      },
      "invoice_document": {
          "generate": true,                                    // Optional. If omitted, leaves unchanged.
          "template_name": "generic_template.doc"              // Optional. If set to nil, uses account's default template.
      },
      "statement_invoice_document": {
          "generate": true,                                    // Optional. If omitted, leaves unchanged.
          "template_name": "generic_template.doc"              // Optional. Needs to exist if generate is true.
      },
      "method": "email",                                       // Required. One of "print", "email".
      "email_settings": {                                      // Use when "method" is "email"
        "billing_email": "billing@customer.com",
        "link_or_attachment": "link",                          // One of "link", "attachment". Default is "link".
        "invoice_grouping_strategy": "email_single"            // See "Possible invoice_grouping_strategy values" section below.
      }
    },
    "references": {
      "Example Reference Label 1": "Value1",
      "Example Reference Label 2": "Value2"
    }
  }
}
```

### Response

#### Success

##### 200 Ok

```
{
  "customer": {
    "id": 20,                               // TriumphPay Audit's internal id for the customer
    "external_id": "customer-external-id",  // YOUR internal id for the customer
    "name": "customer-name",
    "account_number": "customer-account-number",
    "carrier_invoice_reminders": [
      "Confirm with Sandy before approving.",
      "Check this other thing before approving."
    ],
    "invoicing": {
      "bill_to": {
        "name": "name",
        "address_line_1": "address1",
        "address_line_2": "address2",
        "city": "city",
        "state": "state",
        "postal_code": "12345",
        "country": "US"
      },
      "billing_interval": "monthly",
      "trigger": "scheduled",
      "scheduled_day": 15,
      "document_visibility": {
        "use_account_defaults": false,
        "include_pod": true,
        "include_document_types": [
          "customerInvoice",
          "billOfLading"
        ]
      },
      "document_requirements": {
        "use_account_defaults": false,
        "require_pod": true,
        "require_document_types": [
          "customerInvoice",
          "billOfLading"
        ]
      },
      "invoice_document": {
          "generate": true,
          "template_name": "generic_template.doc"
      },
      "statement_invoice_document": {
          "generate": true,
          "template_name": "generic_template.doc"
      },
      "method": "email",
      "email_settings": {
        "billing_email": "billing@customer.com",
        "link_or_attachment": "link",
        "invoice_grouping_strategy": "email_single"
      }
    },
    "references": {
      "Example Reference Label 1": "Value1",
      "Example Reference Label 2": "Value2"
    }
  }
}
```

#### Failure

##### 400 Bad Request

```
{
  "errors": ["..."] // List of Strings
}
```

### About the `scheduled_day` setting
If the `billing_interval` is set to "weekly", this is the day of the week (0-6, Sunday is zero)

If the `billing_interval` is set to "monthly", this is the calendar day of the month (1-28)

Invoices are sent at approximately 4:00 AM UTC of the scheduled days, which may actually occur
in the evening of the previous day in your local timezone or the customer's local time zone.

### Possible `invoice_grouping_strategy` values

| Value | Explanation |
| ----- | ----- |
| `email_single` | Send a consolidated email containing all invoices |
| `email_per_invoice` | Send a separate email for each invoice |
| `email_account_default` | Defer to the setting on account level |

## Create + Update Customer Invoices

PUT https://api.audit.triumphpay.com/tms/customer_invoices

```
curl -X PUT https://api.audit.triumphpay.com/tms/customer_invoices \
  -H "Content-Type: application/json" \
  -H "Authorization: Token token=YOUR_TOKEN" \
  -d '{"customer_invoice": {...}}'
```

### Request

```
{
  "customer_invoice": {
    "customer": {
      "external_id": "customer-external-id",           // Required
      "name": "example-name"                           // Optional, recommended if new customer
    },
    "number": "invoice-number",                        // Required
    "amount": 1000.00,                                 // Required
    "date": "2019-04-09",                              // Required, in ISO 8601 format
    "currency": "USD",                                 // Optional, in alphabetic ISO 4217 format. Defaults to "USD".
    "invoice_document": {                              // Optional
      data: "base-64-data"                             // In Base64 encoding for MIME
    },
    "line_items": [                                    // Optional
      {
        "description": "line item description",
        "edi_code": "abc",
        "total": 123.45,
        "quantity": 3.0,
        "rate": 2.0,
        "rate_qualifier": "PM",
      }
    ],
    "shipper": {
      "name": "name",
      "address_line_1": "address1",
      "address_line_2": "address2",
      "city": "city",
      "state": "state",
      "postal_code": "12345",
      "country": "US"
    },
    "consignee": {
      "name": "name",
      "address_line_1": "address1",
      "address_line_2": "address2",
      "city": "city",
      "state": "state",
      "postal_code": "12345",
      "country": "US"
    },
    "bill_to": {                                       // Optional, to be used if you want to use a specific "bill-to" other than the one set on this invoice's customer
      "external_id": "111",                            // Optional, your internal id for this address
      "name": "name",
      "address_line_1": "address1",
      "address_line_2": "address2",
      "city": "city",
      "state": "state",
      "postal_code": "12345",
      "country": "US"
    },
    "carrier_pay": 800.00,                             // Optional
    "dropped_off_at": "2019-06-02T18:43:26.000Z",      // Optional
    "picked_up_at": "2019-06-01T18:43:26.000Z",        // Optional
    "references": {                                    // Optional
      "Example Reference Label 1": "Value1",
      "Example Reference Label 2": "Value2"
    },
    "shipments": [                                     // Required
      {"external_id": "123"},
      {"external_id": "456"}
    ],
    "loads": [                                         // Optional
      {"external_id": "123"}
    ],
    "business_unit": {                                 // Optional, if this invoice belongs to a specific business unit inside your account.
      "external_id": "business-unit-external-id",      // Required, your internal id for this model
      "federal_id": "355151",                          // Optional, Federal Employer Identification Number.  Only affects creation.
      "logo_id": 1000,                                 // Optional, the TriumphPay Audit Logo ID for this business unit. Only affects creation.
      "name": "example-name"                           // Required if creating a new business unit.  Only affects creation.
    }
  }
}
```

### Response

#### Success

##### 200 Ok

```
{
  "customer_invoice": {
    "id": 10,                                       // TriumphPay Audit's internal id for the customer invoice
    "customer": {
      "external_id": "customer-external-id",
      "name": "example-name"
    },
    "number": "invoice-number",
    "amount": 1000.00,
    "date": "2019-04-09",
    "currency": "USD",
    "invoice_document": {
      data: "base-64-data"
    },
    "line_items": [
      {
        "description": "line item description",
        "edi_code": "abc",
        "total": 123.45,
        "quantity": 3.0,
        "rate": 2.0,
        "rate_qualifier": "PM",
      }
    ],
    "shipper": {
      "name": "name",
      "address_line_1": "address1",
      "address_line_2": "address2",
      "city": "city",
      "state": "state",
      "postal_code": "12345",
      "country": "US"
    },
    "consignee": {
      "name": "name",
      "address_line_1": "address1",
      "address_line_2": "address2",
      "city": "city",
      "state": "state",
      "postal_code": "12345",
      "country": "US"
    },
    "bill_to": {
      "external_id": "111",
      "name": "name",
      "address_line_1": "address1",
      "address_line_2": "address2",
      "city": "city",
      "state": "state",
      "postal_code": "12345",
      "country": "US"
    },
    "carrier_pay": 800.00,
    "dropped_off_at": "2019-06-02T18:43:26.000Z",
    "picked_up_at": "2019-06-01T18:43:26.000Z",
    "references": {
      "Example Reference Label 1": "Value1",
      "Example Reference Label 2": "Value2"
    },
    "shipments": [
      {"external_id": "123"},
      {"external_id": "456"}
    ],
    "loads": [
      {"external_id": "123"}
    ],
    "business_unit": {
      "external_id": "business-unit-external-id",
      "federal_id": "355151",
      "logo_id": 1000,
      "name": "example-name"
    }
  }
}
```

#### Failures

##### 422 Unprocessable Entity (Missing Required Parameter)

```
{
  "errors": { "customer_invoice": ["can't be blank"] }
}
```

##### 422 Unprocessable Entity (Invalid Document)

```
{
  "errors": { "customer_invoice": [{ "invoice_document": "Invalid content type" }] }
}
```

##### 422 Unprocessable Entity (Missing TMS Loads)

```
{
  "errors": { "shipments": ["not found"] }
}
```

##### 422 Unprocessable Entity (Missing Customer)

```
{
  "errors": { "customer": ["not found"] }
}
```

##### 422 Unprocessable Entity (Other Errors)

```
{
  "errors": { "number": ["can't be blank"], "amount":["can't be blank"] } // This is just an example payload. It will have this format though.
}
```

## Create Payments

POST https://api.audit.triumphpay.com/tms/payments

```
curl -X POST https://api.audit.triumphpay.com/tms/payments \
  -H "Content-Type: application/json" \
  -H "Authorization: Token token=YOUR_TOKEN" \
  -d '{"payment": {...}}'
```

### Request

```
{
  "payment": {
    "external_id": "1234",                  // Required
    "status": "Paid",                       // Required, can be any value you want, but is usually something like Pending, Paid, Voided, etc
    "amount": 123.1,                        // Required
    "date": "2015-12-09",
    "method": "check",
    "reference": "payment_reference",
    "invoice_number": "invoice_number",     // Optional, but highly desired if it is a payment for an invoice
    "description": "some description",
    "source_system": "your_tms",            // Something other than "hubtran"; Required
    "load": {
      "load_id": "load_id",                 // Required if no load external_id
      "external_id": "load_external_id",    // Required if no load_id
    },
    "carrier": {
      "external_id": "carrier_external_id", // Required
    }
  }
}
```

### Response

#### Success

##### 201 Created

```
{
  "payment": {
    "id": 1,                              // TriumphPay Audit's internal id for the payment
    "external_id": "1234",
    "amount": 123.1,
    "date": "2015-12-09",
    "method": "check",
    "reference": "payment_reference",
    "invoice_number": "invoice_number",
    "description": "some description",
    "source_system": "your_tms",
    "load": {
      "load_id": "load_id",
      "external_id": "load_external_id",
    },
    "carrier": {
      "external_id": "carrier_external_id"
    }
  }
}
```

#### Failures

##### 422 Unprocessable Entity (Other Errors)

```
{
  "errors": { "payee": ["can't be blank"] } // This is just an example payload. It will have this format though.
}
```

##### 422 Unprocessable Entity (Not Enough Info)

```
{
  "errors": "We couldn't find..."
}
```

## Clear Exceptions

POST https://api.audit.triumphpay.com/tms/carrier_invoices/:id/exceptions/clear

```
curl -X POST https://api.audit.triumphpay.com/tms/carrier_invoices/:id/exceptions/clear \
  -H "Content-Type: application/json" \
  -H "Authorization: Token token=YOUR_TOKEN" \
  -d '{}'
```

### Request

```
{}
```

### Response

#### Success

##### 200 Ok

```
{
  "ok": true
}
```

## List Approved Invoices

GET https://api.audit.triumphpay.com/tms/carrier_invoices/approved

```
curl -X GET https://api.audit.triumphpay.com/tms/carrier_invoices/approved \
  -H "Content-Type: application/json" \
  -H "Authorization: Token token=YOUR_TOKEN"
```

Optional URL params:

1. `results_per_page` - number of results to return per page, max 50
2. `page` - which page of results to return based on `results_per_page`

### Response

#### Success

##### 200 Ok

```
{
  "results_per_page": 20,   // how many results per page
  "page_count": 2,          // number of pages using results_per_page
  "current_page": 1,        // current page
  "carrier_invoices": [
    {
      "id": 123,
      "number": "invoice-number",
      "state": "approved",
      "date": "1981-08-11",
      "date_to_pay": "1981-08-13",
      "amount_to_pay": 1110.2,
      "currency": "USD",
      "quickpay": false,
      "approver": {
        "email": "approver@example.com"
      },
      "processor": {
        "email": "processor@example.com"
      },
      "carrier": {
        "external_id": "carrier-external-id"
      },
      load: {
        "external_id": "load-external-id",
        "load_id": "load-id"
      },
      "line_items": [
        {
          "external_id": "line_item_1",           // The line external_id you sent us
          "description": "line item description",
          "total": 123.45,
          "customer_total": 150.00,
          "quantity": 3.0,
          "rate": 2.0,
          "rate_qualifier": "PM",
          "type_code": "abc",
          "carrier": {
            "external_id": "carrier-external-id"
          }
        }
      ],
      "shipped_items": [{
        "external_id": "1234",
        "description": "Shipped item",
        "class_name": '400',
        "item_type": "CARTON",
        "nmfc": "82270",
        "hazardous_material": false,
        "quantity": 2.0,
        "weight_unit": "lb",
        "weight": 417.0,
        "width": 10.0,
        "height": 12.0,
        "length": 20.0
      }],
      "documents": [
        {
          "id": 14,
          "type": "billOfLading",
          "hubtran_type_name": "Bill of Lading / Packing Slip",
          "proof_of_delivery": true,
          "fingerprint": "09e79148e4ba61d971b7f39c9dc245821b890916",
          "url": "https://api.audit.triumphpay.com/downloads/documents/unique-id",
          "tiff_url": "https://api.audit.triumphpay.com/downloads/documents/tiff-url",  //omitted if account does not require TIFF files
          "pages": [
            {"png_url": "http://api.audit.triumphpay.com/downloads/documents/png-url"}
          ],
          "visibility": {
            "carrier": true,
            "customer": true
          },
          "load": {"external_id": "load-external-id"},
          "shipments": [
            {"external_id": "shipment-external-id"}
          ]
        }
      ],
      "combined_document_urls": [ // All documents of the same type, combined
        {
          "type": "billOfLading",
          "hubtran_type_name": "Bill of Lading / Packing Slip",
          "url": "https://api.audit.triumphpay.com/downloads/documents/combined/unique-id",
          "proof_of_delivery": true,
          "visibility": {
            "carrier": true,
            "customer": true
          }
        }
      ],
      "remit_to": {
        "name": "name",
        "address_line_1": "address1",
        "address_line_2": "address2",
        "city": "city",
        "state": "state",
        "postal_code": "12345",
        "country": "USA"
      }
    }
  ]
}
```

## Mark Approved Invoice as Verified

POST https://api.audit.triumphpay.com/tms/carrier_invoices/:id/verified

```
curl -X POST https://api.audit.triumphpay.com/tms/carrier_invoices/:id/verified \
  -H "Content-Type: application/json" \
  -H "Authorization: Token token=YOUR_TOKEN"
```

### Response

#### Success

##### 200 Ok

```
{
  "ok": true
}
```

## Mark Approved Invoice as Not Verified

This is more for when you're developing and you want to reset an invoice
to mark as verified again.

POST https://api.audit.triumphpay.com/tms/carrier_invoices/:id/not_verified

```
curl -X POST https://api.audit.triumphpay.com/tms/carrier_invoices/:id/not_verified \
  -H "Content-Type: application/json" \
  -H "Authorization: Token token=YOUR_TOKEN"
```

### Response

#### Success

##### 200 Ok

```
{
  "ok": true
}
```

## Create Carrier Invoice Submissions

If you already have an invoice submission from a carrier (paperwork and/or invoice details)
you may want to programatically flow that into TriumphPay Audit. This will create an item in the "New" queue
just like if TriumphPay Audit received an email with paperwork.

For invoices submitted via this API, TriumphPay Audit will not perform machine learning and data extraction
on the submitted documents. Instead we will use the invoice data you submit, as the sole source of data
about the invoice.

POST https://api.audit.triumphpay.com/broker/invoice_submissions

```
curl -X POST https://api.audit.triumphpay.com/broker/invoice_submissions \
  -H "Content-Type: application/json" \
  -H "Authorization: Token token=YOUR_TOKEN" \
  -d '{"invoice": {...}}'
```

### Request

```
{
  "invoice": {
    "external_id": "456",    // Required, your internal ID
    "source": "web_billing", // Required, valid sources are "web_billing", "triumphpay"
    "number": "111",         // Required
    "amount": 123.0,         // Required
    "date": "2019-04-09",    // Required, in iso8601 format
    "quickpay": true,
    "other": {
      "trailer": "333",  // Optional
      "factoring": true, // Optional
    },
    "vendor": {
      "external_id": "789", // Required, the vendor or carrier id that matches id in TMS.
      "sender_email": "carrier@example.com" // Optional, email address of carrier who originally submitted paperwork.
    },
    "load_id": "156", // Optional, ID of load in TMS
    "documents": [
      {
        "external_id": "d123",      // Optional, your internal ID for the document.
        "file_name": "invoice.pdf", // Required
        "data": DATA                // Required, base64-encoded document data
        "document_type": "BOL".     // Optional document_type for the document, see Document Type endpoint for valid values.
        "POD": false,               // Optional flag if document is POD, boolean
      },
      {
        "external_id": "d456",
        "file_name": "bol.pdf",
        "data": DATA
      }
    ]
  }
}
```

### Response

#### Success

```
HTTP Status Code 201
{
  "invoice": {
    "id": 123 // The TriumphPay Audit ID of the invoice if you want to store it
  }
}
```

#### Failures

```
HTTP Status Code 422
{
  "errors": [
    {
      "message": "carrier is missing" // Error message describing the issue.
    }
  ]
}
```

## Create Documents Only Carrier Invoice Submissions

This endpoint allows documents to be sent via the endpoint for future processing. This will create an item in the "New" queue just like if TriumphPay received an email with paperwork.

POST https://api.audit.triumphpay.com/broker/invoice_submission_documents

```
curl -X POST https://api.audit.triumphpay.com/broker/invoice_submission_documents \
  -H "Content-Type: application/json" \
  -H "Authorization: Token token=YOUR_TOKEN" \
  -d '{...}'
```

### Request

```
{
  "external_id": "123123",
  "documents": [
     {
        "external_id": "123",
        "document_type": "invoice", // Optional document_type for the document, see Document Type endpoint for valid values.
        "POD": false,               // Optional flag if document is POD, boolean
        "file_name": "foo.txt",
        "data": DATA                // base64-encoded document data
      },
      {
        "external_id": "456",
        "document_type": "invoice",  // Optional document_type for the document, see Document Type endpoint for valid values.
        "POD": true,                 // Optional flag if document is POD, boolean
        "file_name": "bar.txt",
        "data": DATA                 // base64-encoded document data
      }
   ],
   // If load_id is provided, we'll be able to automatically match the documents to that load. 
   // If vendor.external_id is provided, we'll be able to automatically match the submission ("email") to that carrier.
   // If vendor.sender_email is provided, we'll be able to show the sender email inside the UI, and provide that email as the "reply to"
   // on response emails.
   // Any combination of those three (including none) is allowed.
   "vendor": {
     "external_id": "a_valid_external_id", 
     "sender_email": "carrier@example.com" 
    },
     "load_id": "valid_load_id", // ID of load in TMS
 }
 ```

### Response

#### Success

```
HTTP Status Code 201
{
  "invoice": {
    "id": 123 // The HubTran ID of the invoice if you want to store it
  }
}
```

#### Failure

```
HTTP Status Code 422
{
  "errors": [
    {
      "message": "load_id is missing" // Error message describing the issue.
    }
  ]
}
```

## List Transmissions

In order to remove transmissions from this API call you'll have to call the
transmission verification API with the transmission_id

GET https://api.audit.triumphpay.com/tms/transmissions

```
curl -X GET https://api.audit.triumphpay.com/tms/transmissions \
  -H "Content-Type: application/json" \
  -H "Authorization: Token token=YOUR_TOKEN"
```

Optional URL params:

1. `type` - only one type is currently supported: `load_documents_attached_v1` (default is all types)
2. `results_per_page` - number of results to return per page (default is 20, max is 50)
3. `current_page` - which page of results to return based on `results_per_page` (default is 1)

### Response

#### Success

```
HTTP Status Code 200
{
  "results_per_page": 20,   // how many results per page
  "page_count": 2,          // total number of pages using results_per_page
  "current_page": 1,        // current page
  "transmissions": [
    {
      "type": "load_documents_attached_v1",           // matches `type` param in request
      "transmission_id": 123456,                      // used when verifying the transmission
      "created_at": "2020-04-10T14:39:39Z",
      "load_documents": [
        {
          "id": 12345,
          "type": "billOfLading",
          "hubtran_type_name": "Bill of Lading / Packing Slip",
          "fingerprint": "09e79148e4ba61d971b7f39c9dc245821b890916",
          "proof_of_delivery": true,
          "url": "https://api.audit.triumphpay.com/downloads/documents/unique-id",
          "tiff_url: "https://api.audit.triumphpay.com/downloads/documents/tiff-url",  // omitted if account is not configured to use TIFF images
          "pages": [
            {"png_url": "http://api.audit.triumphpay.com/downloads/documents/png-url"}
          ],
          "load": {
            "external_id": "external-id",
            "scraped_carrier_invoice": {
              "total_amount": {
                "amount": "100",
                "currency": "USD"
              }
            }
          },
          "shipments": [{"external_id": "external-id"}],
          "visibility": {
            "carrier": true,
            "customer": true
          }
        }
      ]
    }
  ]
}
```

#### Failure

```
HTTP Status Code 422
{
  "errors": [
    {
      "message": "Fetching more than 50 transmissions per request is not permitted." // Error message describing the issue.
    }
  ]
}
```

## Mark Transmission as Verified

POST https://api.audit.triumphpay.com/tms/transmissions/:id/verified

```
curl -X POST https://api.audit.triumphpay.com/tms/transmissions/:id/verified \
  -H "Content-Type: application/json" \
  -H "Authorization: Token token=YOUR_TOKEN"
```

### Response

#### Success

##### 200 Ok

```
{
  "ok": true
}
```

## Update Account

PUT https://api.audit.triumphpay.com/accounts/ours

```
curl -X PUT https://api.audit.triumphpay.com/accounts/ours \
  -H "Content-Type: application/json" \
  -H "Authorization: Token token=YOUR_TOKEN" \
  -d '{"account": {"webhook_url": "http://example.com"}}'
```

### Request

```
{
  "account": {
    "webhook_url": "http://example.com"
  }
}
```

### Response

#### Success

##### 200 Ok

```
{
  "account": {
    "webhook_url": "http://example.com"
  }
}
```

#### Failure

##### 422 Unprocessable Entity

```
{
  "message": "this is not a valid endpoint for factor accounts"
}
```

## Document Types

This is more of a utility endpoint so it's easy for you to see
what kinds of document type values you will be receiving via webhook.

GET https://api.audit.triumphpay.com/document_types

```
curl -X GET https://api.audit.triumphpay.com/document_types \
  -H "Content-Type: application/json" \
  -H "Authorization: Token token=YOUR_TOKEN"
```

Response:

```
{
  "enabled": [  // document types currently enabled for the account
    {"name": "Bill of Lading / Packing Slip", "value":"billOfLading"},
    // ...
  ],
  "all": [      // all the document types that COULD be enabled
    {"name": "Bill of Lading / Packing Slip", "value":"billOfLading"},
    // ...
  ]
}
```
