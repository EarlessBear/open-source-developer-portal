---
layout: twoColumn
section: resources
type: access-api-article
title:  "Same Day ACH"
description: "A walkthrough of Same Day ACH and leveraging 'next available' processing times for faster transfers."
---

# Overview
Dwolla enables Access API partners to take advantage of Same Day ACH credit transfers on a [per transfer request](https://docsv2.dwolla.com/#initiate-a-transfer) basis. A `clearing` request parameter is supplied in the request to the Dwolla API which tells Dwolla to expedite clearing for the destination account involved in the transaction. Same Day ACH currently only supports credits, however NACHA (the "managers" of the ACH network) is expected to rollout Same Day debits in late 2017. Same-day ACH is a simple and powerful feature for platforms looking to more to differentiate themselves, streamline cash flows, and improve their end-user experiences.

A few key differences between Standard ACH and Same Day ACH are:

*  Funds are available the **same day**, not days later.
*  New payment settlement windows are available.
*  A transaction limit of $25,000 in enforced by NACHA.
*  It is more expensive than standard ACH.

#### Same Day ACH processing times and settlement
Your application can initiate a transfer from a Dwolla balance prior to 12:00PM Central Time and funds will be available in the recipient's bank account by the end of the day. The table below includes times we have observed for the availability of funds. All times are in Central Time. For more information on the timing of transactions, reference our resource article on [processing times](https://developers.dwolla.com/resources/bank-transfer-workflow/processing-times.html).

| Transfer cutoff time | Earliest funds may be available | Latest funds are available* |
|:---------:|:----------:|:---------:|
| 12:00 PM | 4:00 PM  | 5:00 PM |

*&ast;This time is the latest observed clearing to customer accounts, but the Fed only requires funds to be available by the end of the day.*

### Creating a Customer and attaching a bank account
Before you can initiate a transfer using same day clearing, you must first have a [Customer created](https://docsv2.dwolla.com/#create-a-customer) and a [funding source](https://docsv2.dwolla.com/#create-a-funding-source-for-a-customer) attached for the user receiving the Same Day ACH credit. 

Once your customer has connected a bank account, you'll then want to store the funding source id (e.g. `https://api-sandbox.dwolla.com/funding-sources/ecf993e2-fa22-4cea-8022-c7861200288f`) which will be used when specifying the bank account as the destination href in the request to the [Transfers API](https://docsv2.dwolla.com/#initiate-a-transfer).

### Initiating a Same Day ACH credit transfer
In order to initiate a transfer with Same Day processing, an optional `clearing` JSON object must be included in the transfer request. The clearing object contains `source` and `destination` keys with respective values of `standard` or `next-available`. 

Specifying the destination clearing as `next-available` will allow requests to default to the earliest available processing window based on the time submitted. In addition, transfers greater than $25,000 will default to a processing window permitting larger amounts.

The following example assumes the sending party has a verified account and a verified funding source. We're sending a payout from our Dwolla account balance to our Customer’s funding source which represents the receiving bank account.

```raw
POST https://api-sandbox.dwolla.com/transfers
Accept: application/vnd.dwolla.v1.hal+json
Content-Type: application/vnd.dwolla.v1.hal+json
Authorization: Bearer pBA9fVDBEyYZCEsLf/wKehyh1RTpzjUj5KzIRfDi0wKTii7DqY
Idempotency-Key: 19051a62-3403-11e6-ac61-9e71128cae77

{
    "_links": {
        "source": {
            "href": "https://api-sandbox.dwolla.com/funding-sources/b268f6b9-db3b-4ecc-83a2-8823a53ec8b7"
        },
        "destination": {
            "href": "https://api-sandbox.dwolla.com/funding-sources/ecf993e2-fa22-4cea-8022-c7861200288f"
        }
    },
    "amount": {
        "currency": "USD",
        "value": "10000.00"
    },
    "clearing": {
        "destination": "next-available"
    }
}

...

HTTP/1.1 201 Created
Location: https://api-sandbox.dwolla.com/transfers/636de847-7d02-e711-80ee-0aa34a9b2388
```
```ruby
request_body = {
  :_links => {
    :source => {
      :href => "https://api-sandbox.dwolla.com/funding-sources/b268f6b9-db3b-4ecc-83a2-8823a53ec8b7"
    },
    :destination => {
      :href => "https://api-sandbox.dwolla.com/funding-sources/ecf993e2-fa22-4cea-8022-c7861200288f"
    }
  },
  :amount => {
    :currency => "USD",
    :value => "10000.00"
  },
  :metadata => {
    :paymentId => "12345678",
    :note => "payment for completed work Dec. 1"
  },
  :clearing => {
    :destination => "next-available"
  }
}

# Using DwollaV2 - https://github.com/Dwolla/dwolla-v2-ruby (Recommended)
transfer = app_token.post "transfers", request_body
transfer.response_headers[:location] # => "https://api-sandbox.dwolla.com/transfers/636de847-7d02-e711-80ee-0aa34a9b2388"
```
```php
<?php
$transfersApi = new DwollaSwagger\TransfersApi($apiClient);

$transfer = $transfersApi->create([
  '_links' => [
    'source' => [
      'href' => 'https://api-sandbox.dwolla.com/funding-sources/b268f6b9-db3b-4ecc-83a2-8823a53ec8b7',
    ],
    'destination' => [
      'href' => 'https://api-sandbox.dwolla.com/funding-sources/ecf993e2-fa22-4cea-8022-c7861200288f'
    ]
  ],
  'amount' => [
    'currency' => 'USD',
    'value' => '10000.00'
  ],
  'metadata' => [
    'paymentId' => '12345678',
    'note' => 'payment for completed work Dec. 1',
  ],
  'clearing' => [
    'destination' => 'next-available'
  ]
]);
$transfer; # => "https://api-sandbox.dwolla.com/transfers/636de847-7d02-e711-80ee-0aa34a9b2388"
?>
```
```python
request_body = {
  '_links': {
    'source': {
      'href': 'https://api-sandbox.dwolla.com/funding-sources/b268f6b9-db3b-4ecc-83a2-8823a53ec8b7'
    },
    'destination': {
      'href': 'https://api-sandbox.dwolla.com/funding-sources/ecf993e2-fa22-4cea-8022-c7861200288f'
    }
  },
  'amount': {
    'currency': 'USD',
    'value': '10000.00'
  },
  'metadata': {
    'paymentId': '12345678',
    'note': 'payment for completed work Dec. 1'
  },
  'clearing': {
    'destination': 'next-available'
  }
}

# Using dwollav2 - https://github.com/Dwolla/dwolla-v2-python (Recommended)
transfer = app_token.post('transfers', request_body)
transfer.headers['location'] # => 'https://api-sandbox.dwolla.com/transfers/636de847-7d02-e711-80ee-0aa34a9b2388'
```
```javascript
var requestBody = {
  _links: {
    source: {
      href: 'https://api-sandbox.dwolla.com/funding-sources/b268f6b9-db3b-4ecc-83a2-8823a53ec8b7'
    },
    destination: {
      href: 'https://api-sandbox.dwolla.com/funding-sources/ecf993e2-fa22-4cea-8022-c7861200288f'
    }
  },
  amount: {
    currency: 'USD',
    value: '10000.00'
  },
  metadata: {
    paymentId: '12345678',
    note: 'payment for completed work Dec. 1'
  },
  clearing: {
    destination: 'next-available'
  }
};

appToken
  .post('transfers', requestBody)
  .then(res => res.headers.get('location')); // => 'https://api.dwolla.com/transfers/636de847-7d02-e711-80ee-0aa34a9b2388'
```


### Retrieving a transfer with Same Day clearing
When retrieving the [transfer from the API](https://docsv2.dwolla.com/#retrieve-a-transfer), the response should contain the clearing object with a `destination` key and a value of either `same-day` or `next-day` depending on if the transfer was initiated prior to the last same day processing window and the transfer amount is less than $25,000 (as mentioned above). 

```raw
GET https://api-sandbox.dwolla.com/transfers/636de847-7d02-e711-80ee-0aa34a9b2388
Accept: application/vnd.dwolla.v1.hal+json
Authorization: Bearer pBA9fVDBEyYZCEsLf/wKehyh1RTpzjUj5KzIRfDi0wKTii7DqY

...

{
  "_links": {
    "source": {
      "href": "https://api-sandbox.dwolla.com/accounts/ad5f2162-404a-4c4c-994e-6ab6c3a13254",
      "type": "application/vnd.dwolla.v1.hal+json",
      "resource-type": "account"
    },
    "destination-funding-source": {
      "href": "https://api-sandbox.dwolla.com/funding-sources/ecf993e2-fa22-4cea-8022-c7861200288f",
      "type": "application/vnd.dwolla.v1.hal+json",
      "resource-type": "funding-source"
    },
    "self": {
      "href": "https://api-sandbox.dwolla.com/transfers/636de847-7d02-e711-80ee-0aa34a9b2388",
      "type": "application/vnd.dwolla.v1.hal+json",
      "resource-type": "transfer"
    },
    "funded-transfer": {
      "href": "https://api-sandbox.dwolla.com/transfers/646de847-7d02-e711-80ee-0aa34a9b2388",
      "type": "application/vnd.dwolla.v1.hal+json",
      "resource-type": "transfer"
    },
    "source-funding-source": {
      "href": "https://api-sandbox.dwolla.com/funding-sources/b268f6b9-db3b-4ecc-83a2-8823a53ec8b7",
      "type": "application/vnd.dwolla.v1.hal+json",
      "resource-type": "funding-source"
    },
    "destination": {
      "href": "https://api-sandbox.dwolla.com/customers/99dd22de-6ec6-4ba1-a0d1-09eb169a4bb1",
      "type": "application/vnd.dwolla.v1.hal+json",
      "resource-type": "customer"
    }
  },
  "id": "636de847-7d02-e711-80ee-0aa34a9b2388",
  "status": "processed",
  "amount": {
    "value": "10000.00",
    "currency": "usd"
  },
  "created": "2017-03-06T14:57:56.803Z",
  "clearing": {
    "destination": "same-day"
  }
}
```
```ruby
transfer_url = 'https://api-sandbox.dwolla.com/transfers/636de847-7d02-e711-80ee-0aa34a9b2388'

# Using DwollaV2 - https://github.com/Dwolla/dwolla-v2-ruby (Recommended)
transfer = app_token.get transfer_url
transfer.status # => "processed"
```
```php
<?php
$transferUrl = 'https://api-sandbox.dwolla.com/transfers/636de847-7d02-e711-80ee-0aa34a9b2388';

$transfersApi = new DwollaSwagger\TransfersApi($apiClient);

$transfer = $transfersApi->byId($transferUrl);
$transfer->status; # => "processed"
?>
```
```python
transfer_url = 'https://api-sandbox.dwolla.com/transfers/636de847-7d02-e711-80ee-0aa34a9b2388'

# Using dwollav2 - https://github.com/Dwolla/dwolla-v2-python (Recommended)
transfer = app_token.get(transfer_url)
transfer.body['status'] # => 'processed'
```
```javascript
var transferUrl = 'https://api-sandbox.dwolla.com/transfers/636de847-7d02-e711-80ee-0aa34a9b2388';

appToken
  .get(transferUrl)
  .then(res => res.body.status); // => 'processed'
```