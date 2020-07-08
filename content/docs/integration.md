---
title: Integration
type: docs
---

# Set up Compayer

To start collecting data from your client:

## **Step 1.** Set up your client settings

Create or sign in to your Compayer account to get:

- CLIENT_ID (the unique identifier for your client)
- SECRET_KEY (the secret API key for your client)

## **Step 2.** Send data to Compayer

Follow the instruction to send events from your client to Compayer:

[Compayer PHP SDK](https://github.com/compayer/compayer-lib-php) library is designed to push stat messages to the Compayer analytics from the php-based projects.

**Create and configure a configuration object with your CLIENT_ID and SECRET_KEY:**

```php
$config = new Config(CLIENT_ID, SECRET_KEY);
$client = new Client($config);
```

**Create an instance of the Event class and set properties about a user and payment:**

{{< hint info >}}
The Event tries automatically determine the user IP address and address of the payment initiation page based on the data from the server request. If the user’s request is not available to the script, you can set the payment initiation page or user IP address by yourself (this is necessary for the geolocation filters to work correctly).
{{< /hint >}}

All fields are optional, but it's important to fill out one of the fields: `userEmails`, `userPhones` or `userAccounts` to identify the user made the payment.

You can set custom properties about a user or payment using `setExtra(['my_property' => 'value'])`.

```
$event = (new Event())
    ->setMerchantTransactionId('12345')
    ->setPaymentAmount(250.50)
    ->setPaymentCurrency('RUB')
    ->setUserLang('RUS')
    ->setUserEmails(['customer@compayer.com'])
    ->setUserAccounts(['54321'])
    ->setExtra(['my_property' => 'value']);
```

**Send the generated Event and get the response message with a transaction identifier and log:**

```
$response = $client->pushStartEvent($event);
```

**Get the transaction ID to chain events:**

The transaction identifier is a UUID string like _3677eb06-1a9a-4b6c-9d6a-1799cae1b6bb_.

```
$transactionId = $response->getTransactionId();
```

**Send the Event `success` (`fail` or `refund`) after the payment system responds about the result of the operation:**

```
$client->pushSuccessEvent($event);

$client->pushFailEvent($event)

$client->pushRefundEvent($event)
```

## Sample code of the Event `start`

```php
use Compayer\SDK\Client;
use Compayer\SDK\Config;
use Compayer\SDK\Event;
use Compayer\SDK\Exceptions\SdkException;

const CLIENT_ID = 'client_id';
const SECRET_KEY = 'secret_key';

$config = new Config(CLIENT_ID, SECRET_KEY);
// Enable debug mode.
$config->setDebugMode(true);

$client = new Client($config);

$event = (new Event())
    ->setMerchantTransactionId('12345')
    ->setPaymentAmount(250.50)
    ->setPaymentCurrency('RUB')
    ->setUserLang('RUS')
    ->setUserEmails(['customer@compayer.com'])
    ->setUserAccounts(['54321'])
    ->setExtra(['my_property' => 'value']);

try {
    $response = $client->pushStartEvent($event);
} catch (SdkException $e) {
    print_r($e->getMessage());
}

$transactionId = $response->getTransactionId();

print_r($response->getLog());
```

## Sample code of the Event `success`

```
use Compayer\SDK\Client;
use Compayer\SDK\Config;
use Compayer\SDK\Event;
use Compayer\SDK\Exceptions\SdkException;

const CLIENT_ID = 'client_id';
const SECRET_KEY = 'secret_key';

$config = new Config(CLIENT_ID, SECRET_KEY);
$config->setDebugMode(true);

$client = new Client($config);

// The transaction ID received on the start event
$transactionId = '3677eb06-1a9a-4b6c-9d6a-1799cae1b6bb';

$event = (new Event())
    ->setTransactionId($transactionId)
    ->setMerchantTransactionId('12345')
    ->setPaymentAmount(250.50)
    ->setPaymentCurrency('RUB')
    ->setPayoutAmount(3.87)
    ->setPayoutCurrency('USD')
    ->setUserEmails(['customer@compayer.com'])
    ->setUserAccounts(['54321'])
    ->setExtra(['my_property' => 'value'])
    ->setPaymentSystemResponse('Payment system response as a string');

try {
    $client->pushSuccessEvent($event);
} catch (SdkException $e) {
    print_r($e->getMessage());
}
```

***

{{< questions >}}{{< questions-text >}}{{< /questions-text >}}{{< /questions >}}