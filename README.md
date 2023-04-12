# Laravel Ebay

[![Latest Stable Version](https://poser.pugx.org/hkonnet/laravel-ebay/v)](//packagist.org/packages/hkonnet/laravel-ebay)
[![Total Downloads](https://poser.pugx.org/hkonnet/laravel-ebay/downloads)](//packagist.org/packages/hkonnet/laravel-ebay)
[![Monthly Downloads](https://poser.pugx.org/hkonnet/laravel-ebay/d/monthly)](//packagist.org/packages/hkonnet/laravel-ebay)
[![License](https://poser.pugx.org/hkonnet/laravel-ebay/license)](//packagist.org/packages/hkonnet/laravel-ebay)

This package is a fork from [laravel-ebay](https://github.com/hkonnet/laravel-ebay)

## UPDATE:

Added support for php 7.2+ by updating dependancy from [Ebay-Sdk](https://github.com/hkonnet/laravel-ebay) by David T. Sadler to [Ebay-Sdk](https://github.com/BenMorel/ebay-sdk-php) by benMorel 

This package will organize all the configuration according to laravel and make you use the SDK with out doing any exceptional work or configurations. 

## Getting Started

Follow the instruction to install and use this package.

### Prerequisites

This is package use [**Ebay Php SDK**](https://github.com/BenMorel/ebay-sdk-php)

### Installing

Add Laravel-Ebay to your composer file via the composer require command:


```bash
$ composer require hkonnet/laravel-ebay
```

Or add it to `composer.json` manually:

```json
"require": {
    "hkonnet/laravel-ebay": "^1.2"
}
```

Register the service provider by adding it to the providers key in config/app.php. Also register the facade by adding it to the aliases key in config/app.php.

**Laravel 5.1 or greater**
```php
'providers' => [
    ...
    Hkonnet\LaravelEbay\EbayServiceProvider::class, 
],

'aliases' => [
    ...
    'Ebay' => Hkonnet\LaravelEbay\Facade\Ebay::class,
]
```
**Laravel 5**
```php
'providers' => [
    ...
    'Hkonnet\LaravelEbay\EbayServiceProvider', 
],

'aliases' => [
    ...
    'Ebay' => 'Hkonnet\LaravelEbay\Facade\Ebay',
]
```

Next to get started, you'll need to publish all vendor assets:

```bash
$ php artisan vendor:publish --provider="Hkonnet\LaravelEbay\EbayServiceProvider"
```
This will create a config/ebay.php file in your app that you can modify to set your configuration.

###Configuration
After installation, you will need to add your ebay settings. Following is the code you will find in config/ebay.php, which you should update accordingly.

```php
return [
    'mode' => env('EBAY_MODE', 'sandbox'),

    'siteId' => env('EBAY_SITE_ID','0'),

    'sandbox' => [
        'credentials' => [
            'devId' => env('EBAY_SANDBOX_DEV_ID'),
            'appId' => env('EBAY_SANDBOX_APP_ID'),
            'certId' => env('EBAY_SANDBOX_CERT_ID'),
        ],
        'authToken' => env('EBAY_SANDBOX_AUTH_TOKEN'),
        'oauthUserToken' => env('EBAY_SANDBOX_OAUTH_USER_TOKEN'),
    ],
    'production' => [
        'credentials' => [
            'devId' => env('EBAY_PROD_DEV_ID'),
            'appId' => env('EBAY_PROD_APP_ID'),
            'certId' => env('EBAY_PROD_CERT_ID'),
        ],
        'authToken' => env('EBAY_PROD_AUTH_TOKEN'),
        'oauthUserToken' => env('EBAY_PROD_OAUTH_USER_TOKEN'),
    ]
];
```

## Usage

Following are few examples for using this package.

### Ex 1: Get the official eBay time

Following are the two ways you can do it

**Method 1:**

```php
use \Hkonnet\LaravelEbay\EbayServices;
use \DTS\eBaySDK\Shopping\Types;
  
// Create the service object.
$ebay_service = new EbayServices();
$service = $ebay_service->createShopping();

// Create the request object.
$request = new Types\GeteBayTimeRequestType();

// Send the request to the service operation.
$response = $service->geteBayTime($request);

// Output the result of calling the service operation.
printf("The official eBay time is: %s\n", $response->Timestamp->format('H:i (\G\M\T) \o\n l jS Y'));
```

**Method 2:**

> **Tip:** If you prefer to use **DTS** library class you need to pass the configuration.

```php
use \DTS\eBaySDK\Shopping\Services;
use \DTS\eBaySDK\Shopping\Types;

$config = Ebay::getConfig();

// Create the service object.
$service = new Services\ShoppingService($config);

// Create the request object.
$request = new Types\GeteBayTimeRequestType();

// Send the request to the service operation.
$response = $service->geteBayTime($request);

// Output the result of calling the service operation.
printf("The official eBay time is: %s\n", $response->Timestamp->format('H:i (\G\M\T) \o\n l jS Y'));
```

### Ex 2: Find items by keyword

This example will call the findItemsByKeywords operation

**Method 1:**
```php
use \Hkonnet\LaravelEbay\EbayServices;
use \DTS\eBaySDK\Finding\Types;
  
// Create the service object.
    $ebay_service = new EbayServices();
    $service = $ebay_service->createFinding();

// Assign the keywords.
    $request = new Types\FindItemsByKeywordsRequest();
    $request->keywords = 'Harry Potter';

// Ask for the first 25 items.
    $request->paginationInput = new Types\PaginationInput();
    $request->paginationInput->entriesPerPage = 25;
    $request->paginationInput->pageNumber = 1;

// Ask for the results to be sorted from high to low price.
    $request->sortOrder = 'CurrentPriceHighest';

    $response = $service->findItemsByKeywords($request);

    // Output the response from the API.
    if ($response->ack !== 'Success') {
        foreach ($response->errorMessage->error as $error) {
            printf("Error: %s <br>", $error->message);
        }
    } else {
        foreach ($response->searchResult->item as $item) {
            printf("(%s) %s:%.2f <br>", $item->itemId, $item->title, $item->sellingStatus->currentPrice->value);
        }
    }

```

**Method 2:**
```php
use DTS\eBaySDK\Finding\Services\FindingService;
use \DTS\eBaySDK\Finding\Types;
  
// Create the service object.
    $config = Ebay::getConfig();
    $service = new FindingService($config);

// Assign the keywords.
    $request = new Types\FindItemsByKeywordsRequest();
    $request->keywords = 'Harry Potter';

// Ask for the first 25 items.
    $request->paginationInput = new Types\PaginationInput();
    $request->paginationInput->entriesPerPage = 25;
    $request->paginationInput->pageNumber = 1;

// Ask for the results to be sorted from high to low price.
    $request->sortOrder = 'CurrentPriceHighest';

    $response = $service->findItemsByKeywords($request);

    // Output the response from the API.
    if ($response->ack !== 'Success') {
        foreach ($response->errorMessage->error as $error) {
            printf("Error: %s <br>", $error->message);
        }
    } else {
        foreach ($response->searchResult->item as $item) {
            printf("(%s) %s:%.2f <br>", $item->itemId, $item->title, $item->sellingStatus->currentPrice->value);
        }
    }

```

### Ex 3. Get my ebay selling

```php
use \DTS\eBaySDK\Constants;
use \DTS\eBaySDK\Trading\Types;
use \DTS\eBaySDK\Trading\Enums;
use \Hkonnet\LaravelEbay\EbayServices;

/**
 * Create the service object.
 */
$ebay_service = new EbayServices();
$service = $ebay_service->createTrading();

/**
 * Create the request object.
 */
$request = new Types\GetMyeBaySellingRequestType();

/**
 * An user token is required when using the Trading service.
 */
$request->RequesterCredentials = new Types\CustomSecurityHeaderType();
$authToken = Ebay::getAuthToken();
$request->RequesterCredentials->eBayAuthToken = $authToken;

/**
 * Request that eBay returns the list of actively selling items.
 * We want 10 items per page and they should be sorted in descending order by the current price.
 */
$request->ActiveList = new Types\ItemListCustomizationType();
$request->ActiveList->Include = true;
$request->ActiveList->Pagination = new Types\PaginationType();
$request->ActiveList->Pagination->EntriesPerPage = 10;
$request->ActiveList->Sort = Enums\ItemSortTypeCodeType::C_CURRENT_PRICE_DESCENDING;
$pageNum = 1;

do {
    $request->ActiveList->Pagination->PageNumber = $pageNum;
    
    /**
     * Send the request.
     */
    $response = $service->getMyeBaySelling($request);
    
    /**
     * Output the result of calling the service operation.
     */
    echo "==================\nResults for page $pageNum\n==================\n";
    if (isset($response->Errors)) {
        foreach ($response->Errors as $error) {
            printf(
                "%s: %s\n%s\n\n",
                $error->SeverityCode === Enums\SeverityCodeType::C_ERROR ? 'Error' : 'Warning',
                $error->ShortMessage,
                $error->LongMessage
            );
        }
    }
    if ($response->Ack !== 'Failure' && isset($response->ActiveList)) {
        foreach ($response->ActiveList->ItemArray->Item as $item) {
            printf(
                "(%s) %s: %s %.2f\n",
                $item->ItemID,
                $item->Title,
                $item->SellingStatus->CurrentPrice->currencyID,
                $item->SellingStatus->CurrentPrice->value
            );
        }
    }
    $pageNum += 1;
} while (isset($response->ActiveList) && $pageNum <= $response->ActiveList->PaginationResult->TotalNumberOfPages);

```
> **Note:**
> - There are lots for more example available at [Ebay SDK Examples](https://github.com/davidtsadler/ebay-sdk-examples).
> - Follow above examples but read the Important note below.

## Important Note
Using method 1 in both examples we did
```php
use \Hkonnet\LaravelEbay\EbayServices;
// Create the service object.
$ebay_service = new EbayServices();
$service = $ebay_service->createFinding();
```  
to get service object..

Following methods are available to create services using `EbayServices` class.

* createAccount(array $args = [])
* createAnalytics(array $args = [])
* createBrowse(array $args = [])
* createBulkDataExchange(array $args = [])
* createBusinessPoliciesManagement(array $args = [])
* createFeedback(array $args = [])
* createFileTransfer(array $args = [])
* createFinding(array $args = [])
* createFulfillment(array $args = [])
* createHalfFinding(array $args = [])
* createInventory(array $args = [])
* createMarketing(array $args = [])
* createMerchandising(array $args = [])
* createMetadata(array $args = [])
* createOrder(array $args = [])
* createPostOrder(array $args = [])
* createProduct(array $args = [])
* createProductMetadata(array $args = [])
* createRelatedItemsManagement(array $args = [])
* createResolutionCaseManagement(array $args = [])
* createReturnManagement(array $args = [])
* createShopping(array $args = [])
* createTrading(array $args = [])

These services methods can be used to get appropriate service object to perform operations on Ebay.

## Forked By

* **Mudassir Maqbool** - [Mudassir Maqbool](https://github.com/mudassirmaqbool/)


## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details

## Acknowledgments

* [David T. Sadler](https://github.com/davidtsadler)  for his awesome Ebay SDK.
* **Haroon Khan** - [hkonnet](https://github.com/hkonnet) For the original wrapper 
* [BenMorel](https://github.com/BenMorel/ebay-sdk-php)  for adding php 7.2+ support to Ebay SDK.


