1. [Checkout](#checkout)
    * [Integration types](#integration-types)
        * [Hosted page](hostedPage.md)
        * [Server-To-Server](serverToServer.md)
    * [IPN (instant payment notification)](#ipn)
        * [IPN Request example](#ipn-request-example)
1. [Card tokenization](createCardToken.md)
1. [Check payment status](checkInvoiceStatus.md)
1. [Merchant payment methods](getMerchantPaymentMethods.md)
1. [Payment Routing](paymentRouting.md)

# Checkout

----
**Note:** If you are using an old (legacy) project, you have to use the old [documentation](https://old.payop.com/apidoc/). 
In case you are still using old api we recommend you switch your application to new api.
----

All projects created after the transition to the new system work according to the new documentation.

You can distinguish an old project from a new one by a public key.

In the old project, the public key has a similar form:

**application-777**

For a new project, the public key looks like this:

**application-7cccbe4b-e448-45d3-93d0-35f1a65df87e**

## Integration types

There are currently 2 options for using the API.

* [Hosted page](hostedPage.md) - very simple integration with showing Payop pages
* [Server-To-Server](serverToServer.md) - more hard integration using Payop API.
 

## IPN

After finish the payment and assigning a transaction
to one of the [final statuses](../Transaction/getTransaction.md#transaction-statuses),
the payment gateway sends to the Merchant's server a Instant Payment Notification (IPN).

IPN will be send only in case of successfully created transaction.
Internal Payop transaction created only after successful request to Acquirer.
IPN request will be send to ipn url which you setup for selected project.

For greater security, we highly recommend that you accept IPN only from our IP address:
* 52.49.204.201 
* 54.229.170.212

----
**Note:** Sometimes IPN that is related to a particular order may be sent multiple times. If you are receiving multiple IPNs with the same status and data (order ID. transaction ID etc.) that refer to an order, **please make sure that the payment has been credited only once**.

So, what do you need to check: if you received IPNs with the same status and data (order ID, transaction ID etc.) related to a transaction, you have to accept only the first notification and ignore the others.

However, several IPN notifications should be accepted for a particular transaction if they carry different statuses. In this case, you don't need to ignore them, but update the current transaction state.

For instance: 

case 1. You received several IPNs with the same data and "success" state for the order ID 1111; 
case 2. You received IPN for the order ID 1111 with status 'failed'; later on you received the other IPN notification with the same transaction data for the order ID 1111 but with the other state "success".

Solution: 

case 1. You need to accept only the first notification and ignore the others;
case 2. You shouldn't ignore notification with a different state to let the transaction status become updated.

----
**Note:** Please note! A notification is sent to the merchant's server within 24 hours
 and until the Payop server, upon this request, receives the HTTP status code "200 OK".

----

### IPN Request example

`Content-Type: application/json`

`POST https://url from your project`

```json
{
    "invoice": {
        "id": "d024f697-ba2d-456f-910e-4d7fdfd338dd",
        "status": 1,
        "txid": "dca59ca5-be19-470d-9494-9b76944e0241",
        "metadata": {
            "internal merchant id": "example",
            "any other merchant data which were passed to invoice on create it": {
                "orderId": "test",
                "amount": 3,
                "customerId": 15487            
            }
        }
    }, 
    "transaction": {
        "id": "dca59ca5-be19-470d-9494-9b76944e0241",
        "state": 2,
        "order": {
            "id": "ANY_ORDER_ID"        
        },
        "error": {
            "message": "3DS authorization error or 3DS canceled by payer",
            "code": ""
        }
    }
}
```

**Parameters**

Parameter                       |  Type   |                 Description     |
--------------------------------|---------|---------------------------------| 
invoice.id                      | string  | Invoice identifier              |
invoice.status                  | number  | Invoice status                  |
invoice.txid                    | string  | Transaction identifier          |
transaction.id                  | string  | Transaction identifier          |
transaction.state               | number  | Transaction state               |
transaction.error.message       | string  | Transaction error message       |
transaction.error.code          | string  | Always empty string             |

Using transaction id (txid) you [can get transaction](../Transaction/getTransaction.md) 
find there your order id and process this order in your system.



