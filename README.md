[![license](https://img.shields.io/github/license/rpsimao/invoicexpress-api.svg)]()
[![GitHub release](https://img.shields.io/github/release/rpsimao/invoicexpress-api.svg)]()
[![Packagist release](https://img.shields.io/packagist/v/rpsimao/invoicexpress-api.svg)]()


# Laravel InvoiceXpress API

Laravel package to interact with InvoiceXpress API

**Tested with Laravel 5.5.***

## Table of Contents
- [1 - Installation](#1---installation)
  * [1.1 - Publish configuration](#11---publish-configuration)
  * [1.2 - Migrations](#12---migrations)
- [2 - Configuration](#2---configuration)
- [3 - Usage](#3---usage)
  * [3.1 - Eloquent Model](#31---eloquent-model)
  * [3.1.1 - One-to-One relationship with Laravel::Auth()](#311---one-to-one-relationship-with-laravel--auth--)
  * [3.2 - Interact with the API](#32---interact-with-the-api)
- [4 - Tests](#4---tests)
- [5 - Messages](#5---messages)
  * [5.1 - Error messages](#51---error-messages)
  * [5.2 - Success Messages](#52---success-messages)



## 1 - Installation

Via Composer

``` bash
$ composer require rpsimao/invoicexpress-api
```

In your config/app.php, register Providers and the Facade
(Not needed for Laravel 5.5 upwards)

``` php
'providers' => [
....

rpsimao\InvoiceXpressAPI\InvoiceXpressAPIServiceProvider::class,

.....

'aliases' => [

.....

'InvoiceXpressClients' =>  rpsimao\InvoiceXpressAPI\InvoiceXpressAPIFacade::class,

.....


```

### 1.1 - Publish configuration

```bash
$ php artisan vendor:publish --tag=ivxapi-config
```

In the configuration file, all the API endpoints are accessible, so for example you need to generate an invoice PDF:

```php
config(invoicexpress.enpoints.invoice.generate_pdf);

```

All endpoints are generic like: `'api/pdf/{invoice-id}.xml'`, so there is a helper function for replacing the generic endpoint with the real value:

```php

endpoint_replace(['the-real-value'], config(invoicexpress.enpoints.invoice.generate_pdf));


```

The first argument MUST be an array, and the **number of the itens to replace, must match the items to be replaced in the endpoint**. If not an exception is raised, and a fatal error is thrown.



### 1.2 - Migrations

```bash
$ php artisan vendor:publish --tag=ivxapi-migrations
$ php artisan migrate
```

## 2 - Configuration

Add to your .env file your API Key and Account name

```

INVOICEXPRESS_API_KEY=
INVOICEXPRESS_ACCOUNT_NAME=


```

These will be read by the config file:

```php
....

'api_key'      => env('INVOICEXPRESS_API_KEY'),
'account_name' => env('INVOICEXPRESS_ACCOUNT_NAME'),

....

```

>If you do not want to put your API key in the .env file, or prefer to get it on every request, you can call the `getAPIKey()` method. This way you can change the API key in your account frequently and not need to update the app.
>
>
>In the config file `invoicexpress`, there are 2 empty fields `['username', 'password']` so you can put the username/password there, if you want.

```php
$client = new InvoiceXpressAPI();
$api_key = $client->getAPIKey('my-username', 'my-password');
....
//later in the query
....
$client->setQuery(['api_key' => $api_key]);
....

```

## 3 - Usage


>
> **Check the documentation for the params of the actions.**
>
> See: [ https://invoicexpress.com/api/overview ]()
>




**There are 2 Classes for working with the API:**

### 3.1 - Eloquent Model

It has one custom function, for retrieve all your customers and put them into the DB.

You can make a cron job for retrieving them periodically.



```php

//Accepts a flag (true or false[default])
InvoiceXpressClients::getAllClientsFromAPI(true);


```
If you pass the `true` flag, the function inserts the clients into the database. `False` or none, returns an array with all your clients.

If the client already exists, it updates the values.

### 3.1.1 - One-to-One relationship with Laravel::Auth()

If you wish to have a relationship between the InvoiceXpress and your app Users, do the following:

```bash
$ php artisan vendor:publish --tag=ivxapi-migrateauth
$ php artisan migrate
```

In your Users Model, add the following method:

```php
class User extends Model
{
.......

//Get the InvoiceXpress Client record associated with the user.

public function invoicexpress()
{
	return $this->hasOne('InvoiceXpressClients');
}


```

You now have a one-to-one relationship. Now you only have to insert the user_id in the InvoiceXpress table.


### 3.2 - Interact with the API
```php

use rpsimao\InvoiceXpressAPI\Service\InvoiceXpressAPI;

//Making a GET REQUEST

$client = new InvoiceXpressAPI();
$client->setMethod('GET');
$client->setUrl(config('invoicexpress.my_url'));
$client->setEndpoint(config('invoicexpress.endpoints.clients.list_all'));
$client->setQuery(['api_key' => config('invoicexpress.api_key')]);

$response = $client->talkToAPI();

.....



// Another GET Request to generate a PDF for an invoice

$client = new InvoiceXpressAPI();
$client->setMethod('get');
$client->setUrl(config('invoicexpress.my_url'));
$client->setEndpoint(
    endpoint_replace(['12759480'], config('invoicexpress.endpoints.invoices.generate_pdf'))
);
$client->setQuery([
        'api_key' => config('invoicexpress.api_key'),
        'invoice-id' => '12759480',
        'second_copy' => true
    ]);
$response = $client->talkToAPI();



//Making a POST REQUEST
// Creating a new Client

client = new InvoiceXpressAPI();
$client->setMethod('post');
$client->setUrl(config('invoicexpress.my_url'));
$client->setEndpoint( config('invoicexpress.endpoints.clients.create'));
$client->setQuery([
        'api_key' => config('invoicexpress.api_key'),
        'client' => [
        	'name' => 'My name',
        	'code' => 'My Client Code',
        	'email' => 'client@email.com'
        	//.... insert more values ....
        ]
    ]);
$response = $client->talkToAPI();


//Do whatever you need with the response

//Making a PUT REQUEST

$client = new InvoiceXpressAPI();
$client->setMethod('put');
$client->setUrl(config('invoicexpress.my_url'));
$client->setEndpoint(endpoint_replace(['123456789'], config('invoicexpress.endpoints.clients.update')));
$client->setQuery([
	'api_key' => config('invoicexpress.api_key'),
	'client-id' => '123456789',
	'client' => [
		'name' => 'My awesome Client',
		'code' => '123',
		'phone' =>  999888777
		//.... insert more values ....
		]
	]);
$response = $client->talkToAPI();


//Do whatever you need with the response


```

## 4 - Tests

Currently there are 4 tests available.

1. A GET Request to Retrieve all Clients
2. A PUT Request to Update Client Information
3. A GET Request for retrieve your API key
4. A GET Request for generate a Invoice PDF 


For them to work, you have to fill with you own credentials | data:

```php
class GetTest extends TestCase {

// Use your own credentials|data to run the tests

	protected $url          = '';
	protected $api_key      = '';
	protected $username     = '';
	protected $password     = '';
	protected $client_id    = '';
	protected $client_name  = '';
	protected $client_code  = '';
	protected $client_phone = '';
	protected $invoice      = '';

.......


```

Then you can run the tests:

```bash
$ cd your-laravel-project-folder
$ vendor/bin/phpunit vendor/rpsimao/invoicexpress-api
```

If all goes well, you should receive:

```bash
OK (4 tests, 4 assertions)
```

## 5 - Messages

By default all Error / Success messages are returned in XML format.
If you wish to change to JSON, just add the `setMsgFormat()` method and pass the JSON flag:

```php
.....
$client = new InvoiceXpressAPI();
$client->setMsgFormat('json');
......
```

### 5.1 - Error messages

The` api_code` and `api_msg` are the real messages that the InvoiceXpress API returns, the others are just for debugging.

The debugging tags only appears if in the .env file: `APP_DEBUG=true`

This is how the Error Messages are returned:

#### XML
```xml
<?xml version="1.0"?>
<response>
	<api_code>500</api_code>
	<api_msg>Server error: `GET https://mycompany.app.invoicexpress.com/api/pdf/1234567.xml?api_key=11111abc2222def33333&amp;invoice-id=1234567` resulted in a `500 Internal Server Error` response: An error occured on the server. We have been notified.</api_msg>
	<file>/Code/testapi/vendor/rpsimao/invoicexpress-api/src/Service/InvoiceXpressAPI.php</file>
	<line>408</line>
	<message>simplexml_load_string(): Entity: line 1: parser error : Start tag expected, '&amp;lt;' not found</message>
</response>

```

#### JSON
```json
{
"api_code":"500",
"api_msg":"Server error: `GET https:\/\/mycompany.app.invoicexpress.com\/api\/pdf\/1234567.xml?api_key=11111abc2222def33333&invoice-id=1234567` resulted in a `500 Internal Server Error` response:\nAn error occured on the server. We have been notified.\n\n",
"file":"\/Code\/testapi\/vendor\/rpsimao\/invoicexpress-api\/src\/Service\/InvoiceXpressAPI.php",
"line":385,
"message":"simplexml_load_string(): Entity: line 1: parser error : Start tag expected, '&lt;' not found"
} 

```

### 5.2 - Success Messages
This is how the Success Messages are returned:
#### XML
```xml
<?xml version="1.0"?>
<response>
	<api_code>200</api_code>
	<api_msg>OK</api_msg>
	<api_values><pdfUrl>https://invoicexpress-downloads.s3.amazonaws.com/store_00000_Factura-0000-a.pdf?AWSAccessKeyId=AAKKAAKKK&amp;Expires=1501762080&amp;Signature=vmePXICWhUBRLygwVO6y2Lx6x4M%3D&amp;response-content-disposition=attachment%3B%20filename%3DFactura-0000-a.pdf&amp;response-content-type=application%2Fpdf</pdfUrl></api_values>
</response>

```
#### JSON
```json
{
  "response": {
    "api_code": "200",
    "api_msg": "OK",
    "api_values": { "pdfUrl": "https://invoicexpress-downloads.s3.amazonaws.com/store_00000_Factura-0000-a.pdf?AWSAccessKeyId= AAKKAAKKK&Expires=1501762080&Signature=vmePXICWhUBRLygwVO6y2Lx6x4M%3D&response-content-disposition=attachment%3B%20filename%3DFactura-0000-a.pdf&response-content-type=application%2Fpdf" }
  }
}

```
