PHP Odoo API client
===================

[![Build Status](https://travis-ci.org/Ang3/php-odoo-api-client.svg?branch=master)](https://travis-ci.org/Ang3/php-odoo-api-client) [![Latest Stable Version](https://poser.pugx.org/ang3/php-odoo-api-client/v/stable)](https://packagist.org/packages/ang3/php-odoo-api-client) [![Latest Unstable Version](https://poser.pugx.org/ang3/php-odoo-api-client/v/unstable)](https://packagist.org/packages/ang3/php-odoo-api-client) [![Total Downloads](https://poser.pugx.org/ang3/php-odoo-api-client/downloads)](https://packagist.org/packages/ang3/php-odoo-api-client)

Odoo API client using 
[XML-RPC Odoo ORM External API](https://www.odoo.com/documentation/12.0/webservices/odoo.html).

**Main features**

- Authentication
- ORM methods
- Expression builder **added in v5.0**

If you are in Symfony application, I suggest you to install the bundle 
[ang3/odoo-api-bundle](https://github.com/Ang3/odoo-api-bundle). 
It provides a registry you can configure easily and deploys clients as services.

**Coming:**

The ORM (Object relational mapper) is in development: 
[ang3/php-odoo-orm](https://github.com/Ang3/php-odoo-orm) (need tests). 
Of course if you are in Symfony application you should be interested in the bundle: 
[ang3/odoo-bundle](https://github.com/Ang3/odoo-bundle) (ORM integration).

Requirements
============

- The PHP extension ```php-xmlrpc``` must be enabled.

| Odoo server | Compatibility | Comment |
| --- | --- | --- |
| v13.0 | Yes | Some Odoo model names changed (e.g account.invoice > account.move) |
| v12.0 | Yes | First tested version
| v11.0 | Unknown |
| v10.0 | Unknown |
| < v10 | Unknown |

Installation
============

Open a command console, enter your project directory and execute the
following command to download the latest stable version of the client:

```console
$ composer require ang3/php-odoo-api-client
```

This command requires you to have Composer installed globally, as explained
in the [installation chapter](https://getcomposer.org/doc/00-intro.md)
of the Composer documentation.

Summary
=======

- [Usage](#usage)
- [Built-in ORM methods](#built-in-orm-methods)
    - [Write records](#write-records) - Create and update records
    - [Search records](#search-records) - Search and read records
    - [Delete records](#delete-records)
- [Expression builder](##expression-builder) - Oodo array expressions
    - [Get the expression builder](#get-the-expression-builder) - Create or get a builder
    - [Domains](#domains) - Build Odoo domain expressions
    - [Collection operations](#collection-operations) - Build Odoo collection field operations
- [Upgrades](#upgrades) - Major changes

Usage
=====

First, you have to create a client instance:

```php
<?php

require_once 'vendor/autoload.php';

use Ang3\Component\Odoo\Client;

// Option 1: by calling the constructor...
$client = new Client('<host>', '<database>', '<username>', '<password>', $logger = null);

// Option 2 : by calling the static method ::createFromConfig() with configuration as array
$client = Client::createFromConfig([
    'host' => '<host>',
    'database' => '<database>',
    'user' => '<user>',
    'password' => '<password>',
], $logger = null);
```

Exceptions:
- ```Ang3\Component\Odoo\Exception\MissingConfigParameterException``` when a required parameter is missing 
from the static method ```createFromConfig()```.

Then, make your call:

```php
$result = $client->call($name, $method, $parameters = [], $options = []);
```

Exceptions:
- ```Ang3\Component\Odoo\Exception\AuthenticationException``` when authentication failed.
- ```Ang3\Component\Odoo\Exception\RequestException``` when request failed.

These previous exception can be thrown by all methods of the client.

Built-in ORM methods
====================

Write records
-------------

For all these methods, the parameter ```$data``` can contains *collection field operations*.

Please see the section [Expression builder](#expression-builder) to manage *collection fields* easily.

**Create a record**

```php
$data = [
    'field_name' => 'value'
];

$recordId = $client->create('model_name', $data);
```

The method returns the ID of the created record.

**Update a record**

```php
$ids = [1,2,3]; // Can be a value of type int|array<int>

$data = [
    'field_name' => 'value'
];

$client->update('model_name', $ids, $data); // void
```

The method returns ```void```.

Search records
--------------

For all these methods, the parameter ```$criteria``` must be an array. I suggest you to create tour criteria 
with the [Expression builder](#expression-builder).

**Read records**

Get a list of records by ID.

```php
$ids = [1,2,3]; // Can be a value of type int|array<int>
$records = $client->read('model_name', $ids);
```

The method returns an array of records of type ```array<array>```.

**Find a record by ID**

```php
$id = 1; // Must be an integer
$record = $client->find('model_name', $id, $options = []);
```

The method returns the record as ```array```, or ```NULL``` is the record was not found.

---

For each method below, the value of the parameter ```$criteria``` can be ```NULL```, 
an array or a *domain expression*.

Please see the section [Expression builder](#expression-builder) to build *domain expressions*.

**Find ONE record by criteria and options**

```php
$record = $client->findOneBy('model_name', $criteria = null, $options = []);
```

The method returns the record as ```array```, or ```NULL``` is the record was not found.

**Find records by criteria and options**

```php
$records = $client->findBy('model_name', $criteria = null, $options = []);
```

The method returns an array of records of type ```array<array>```.

**Search ONE record ID by criteria and options**

```php
$recordIds = $client->searchOne('model_name', $criteria = null, $options = []);
```

**Search all record IDs by options**

```php
$recordIds = $client->searchAll('model_name', $options = []);
```

**Search record(s)**

Get a list of ID for matched record(s).

```php
$recordIds = $client->search('model_name', $criteria = null, $options = []);
```

The method returns a list of ID of type ```array<int>```.

**Check if a record exists by ID**

```php
$id = 1; // Must be an integer
$recordExists = $client->exists('model_name', $id);
```

**Count records by criteria**

```php
$nbRecords = $client->count('model_name', $criteria = null);
```

The method returns an ```integer```.

Delete records
--------------

You can delete many records at one time.

```php
$ids = [1,2,3]; // Can be a value of type int|array<int>
$client->delete('model_name', $ids);
```

The method returns ```void```.

Expression builder
====================

There are two kinds of expressions : ```domains``` for criteria 
and ```collection operations``` in data writing.
Odoo has its own array format for those expressions. 
The aim of the expression builder is to provide some 
helper methods to simplify a programmer's life.

Get the expression builder
--------------------------

Here is an example of how to build a ```ExpressionBuilder``` object from a ```Client``` instance:

```php
$expr = $client->getExpressionBuilder();
// Or $expr = $client->expr();
```

You can still use the expression builder as standalone by creating an instance yourself.

```php
use Ang3\Component\Odoo\Expression\ExpressionBuilder;

$expr = new ExpressionBuilder();
```

Domains
-------

For all **search** queries (```search```, ```findBy```, ```findOneBy``` and ```count```), 
Odoo is waiting for an array of [domains](https://www.odoo.com/documentation/13.0/reference/orm.html#search-domains) 
with a *polish notation* for logical operations (```AND```, ```OR``` and ```NOT```).

It could be quickly ugly to do a complex domain, but don't worry the builder makes all 
for you. :)

Each domain builder method creates an instance of ```Ang3\Component\Odoo\Expression\DomainInterface```. 
The only one method of this interface is ```toArray()``` in order to get a normalized array of the expression.

To illustrate how to work with it, here is an example using ```ExpressionBuilder``` helper methods:

```php
// $client instanceof Client

// Get the expression builder
$expr = $client->expr();

$result = $client->findBy('model_name', $expr->andX( // Logical node "AND"
	$expr->gte('id', 10), // id >= 10
	$expr->lte('id', 100), // id <= 10
), $options = []);
```

Of course, you can nest logical nodes:

```php
$result = $client->findBy('model_name', $expr->andX(
    $expr->orX(
        $expr->eq('A', 1),
        $expr->eq('B', 1)
    ),
    $expr->orX(
        $expr->eq('C', 1),
        $expr->eq('D', 1),
        $expr->eq('E', 1)
    )
), $options = []);
```

The client formats automatically all domains by calling the special builder 
method ```normalizeDomains()``` internally.

Here is a complete list of helper methods available in ```ExpressionBuilder``` for domain expressions:

```php
/**
 * Create a logical operation "AND".
 */
public function andX(DomainInterface ...$domains): CompositeDomain;

/**
 * Create a logical operation "OR".
 */
public function orX(DomainInterface ...$domains): CompositeDomain;

/**
 * Create a logical operation "NOT".
 */
public function notX(DomainInterface ...$domains): CompositeDomain;

/**
 * Check if the field is EQUAL TO the value.
 *
 * @param mixed $value
 */
public function eq(string $fieldName, $value): Comparison;

/**
 * Check if the field is NOT EQUAL TO the value.
 *
 * @param mixed $value
 */
public function neq(string $fieldName, $value): Comparison;

/**
 * Check if the field is UNSET OR EQUAL TO the value.
 *
 * @param mixed $value
 */
public function ueq(string $fieldName, $value): Comparison;

/**
 * Check if the field is LESS THAN the value.
 *
 * @param mixed $value
 */
public function lt(string $fieldName, $value): Comparison;

/**
 * Check if the field is LESS THAN OR EQUAL the value.
 *
 * @param mixed $value
 */
public function lte(string $fieldName, $value): Comparison;

/**
 * Check if the field is GREATER THAN the value.
 *
 * @param mixed $value
 */
public function gt(string $fieldName, $value): Comparison;

/**
 * Check if the field is GREATER THAN OR EQUAL the value.
 *
 * @param mixed $value
 */
public function gte(string $fieldName, $value): Comparison;

/**
 * Check if the variable is LIKE the value.
 *
 * An underscore _ in the pattern stands for (matches) any single character
 * A percent sign % matches any string of zero or more characters.
 *
 * If $strict is set to FALSE, the value pattern is "%value%" (automatically wrapped into signs %).
 *
 * @param mixed $value
 */
public function like(string $fieldName, $value, bool $strict = false, bool $caseSensitive = true): Comparison;

/**
 * Check if the field is IS NOT LIKE the value.
 *
 * @param mixed $value
 */
public function notLike(string $fieldName, $value, bool $caseSensitive = true): Comparison;

/**
 * Check if the field is IN values list.
 */
public function in(string $fieldName, array $values = []): Comparison;

/**
 * Check if the field is NOT IN values list.
 */
public function notIn(string $fieldName, array $values = []): Comparison;
```

Collection operations
---------------------

In data writing context, Odoo allows you to manage ***toMany** collection fields with special commands. 
Please read the [ORM documentation](https://www.odoo.com/documentation/13.0/reference/orm.html#openerp-models-relationals-format) to known what we are talking about.

The expression builder provides helper methods to build a well-formed *operation command*: 
each operation method returns the operation as array.

To illustrate how to work with operations, here is an example using ```ExpressionBuilder``` helper methods:

```php
// $client instanceof Client

// Get the expression builder
$expr = $client->expr();

// Prepare new record data
$data = [
    'foo' => 'bar',
    'bar_ids' => [ // Field of type "manytoMany"
        $expr->addRecord(3), // Add the record of ID 3 to the set
        $expr->createRecord([  // Create a new sub record and add it to the set
            'bar' => 'baz'
            // ...
        ])
    ]
];

$result = $client->create('model_name', $data);
```

The client formats automatically the whole query parameters for all writing methods 
(```create``` and ```update```) by calling the special builder 
method ```normalizeData()``` internally.

Here is a complete list of helper methods available in ```ExpressionBuilder``` for operation expressions:

```php
/**
 * Adds a new record created from data.
 */
public function createRecord(array $data): array;

/**
 * Updates an existing record of id $id with data.
 * /!\ Can not be used in record insert query.
 */
public function updateRecord(int $id, array $data): array;

/**
 * Adds an existing record of id $id to the collection.
 */
public function addRecord(int $id): array;

/**
 * Removes the record of id $id from the collection, but does not delete it.
 * /!\ Can not be used in record insert query.
 */
public function removeRecord(int $id): array;

/**
 * Removes the record of id $id from the collection, then deletes it from the database.
 * /!\ Can not be used in record insert query.
 */
public function deleteRecord(int $id): array;

/**
 * Replaces all existing records in the collection by the $ids list,
 * Equivalent to using the command "clear" followed by a command "add" for each id in $ids.
 */
public function replaceRecords(array $ids = []): array;

/**
 * Removes all records from the collection, equivalent to using the command "remove" on every record explicitly.
 * /!\ Can not be used in record insert query.
 */
public function clearRecords(): array;
```

Upgrades & updates
==================

### v6.1.1 (last stable)

- Fixed logging.
- Deleted useless files and updated ```.gitignore```

### v6.1.0

- Replaced package [darkaonline/ripcord](https://packagist.org/packages/DarkaOnLine/Ripcord) by 
[ang3/php-xmlrpc-client](https://packagist.org/packages/ang3/php-xmlrpc-client).
- Implemented interface ```Ang3\Component\Odoo\Exception\ExceptionInterface``` for all client exceptions.

### v6.0.1

- Removed dependency of package [ang3/php-dev-binaries](https://packagist.org/packages/ang3/php-dev-binaries).

### v6.0.0

- Added methods ```searchOne``` and ```searchAll```.
- Back to package [darkaonline/ripcord](https://packagist.org/packages/DarkaOnLine/Ripcord).
- Removed XML-RPC client.
- Removed remote exception.
- Removed trace back feature.

### v5.1.2

- Fixed logging messages.

### v5.1.1

- Fixed method ```findAll```.
- Updated endpoint logging level to ```debug``` and added client logging with level ```info```

### v5.1.0

- Added method ```findAll```.
- Fixed empty array result issue.

### v5.0.6

- Catched remote type exception when the method returns no value.
- Fixed composer dev dependencies
- Updated travis configuration
- Updated ```README.md```

### v5.0.5

- Fixed method ```Client::create()```.

### v5.0.4

- Added method ```Client::exists()```.

### From 4.* to 5.*

What you have to do:
- Create the client with the constructor ```new Client(array $config)``` instead of calling static method ```Client::createFromConfig(array $config)```.
- Replace usages of the method ```searchAndRead``` by the method ```findBy```.

Logs:
- Replaced package [darkaonline/ripcord](https://packagist.org/packages/DarkaOnLine/Ripcord) to [phpxmlrpc/phpxmlrpc](https://github.com/gggeek/phpxmlrpc).
- Created XML-RPC client.
- Deleted static method ```Client::createFromConfig(array $config)```.
- Renamed ORM method ```searchAndRead(...)``` to ```findBy(...)```.
- Added ORM methods ```find(...)``` to ```findOneBy(...)```.
- Added expression builder support.

### From 3.* to 4.*

- Updated namespace ```Ang3\Component\Odoo\Client``` to ```Ang3\Component\Odoo```.

### From 2.* to 3.*

- Updated namespace ```Ang3\Component\OdooApiClient``` to ```Ang3\Component\Odoo\Client```.

That's it!