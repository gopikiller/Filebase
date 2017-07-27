# Filebase
A Very Simple Flat File Database Storage.

[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/timothymarois/Filebase/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/timothymarois/Filebase/?branch=master)


## Installation

Use [Composer](http://getcomposer.org/) to install package.

Run `composer require timothymarois/filebase` or add to your main `composer.json` file.

## Usage

```php
// setting the access and configration to your database
$my_database = new \Filebase\Database([
    'dir' => 'path/to/database/dir'
]);

// in this example, you would replace user_name with the actual user name.
// It would technically be stored as user_name.json
$item = $my_database->get('user_name');

// display property variables
echo $item->first_name;
echo $item->last_name;
echo $item->email;

// change existing or add new values
$item->email = 'example@example.com';

// quickly just use, and thats it!
$item->save();

```


## (1) Config Options

The config is *required* when defining your database. The options are *optional* as they have their own defaults.

```php
$db = new \Filebase\Database([
    'dir'      => 'path/to/database/dir',
    'format'   => \Filebase\Format\Json::class
]);
```

|Name				|Type		|Default Value	    |Description												|
|---				|---		|---			         	|---														|
|`dir`				|string		|current directory          |The directory where the database files are stored. 	    |
|`format`			|object		|`\Filebase\Format\Json`   |The format class used to encode/decode data				|
|`validate`			|array		|   |Check [Validation Rules](https://github.com/timothymarois/Filebase#5-validation-optional) for more details |


## (2) Formatting

You can write your own or change the existing format class in the config. The methods in the class must be `static` and the class must implement `\Filebase\Format\FormatInterface`

The Default Format Class: `JSON`
```php
\Filebase\Format\Json::class
```


## (3) GET (and methods)

After you've loaded up your database config, then you can use the `get()` method to retrieve a single document of data.

```php

// my user id
$user_id = '92832711';

// get the user information by id
$item = $db->get($user_id);
```

`get()` returns `\Filebase\Document` object and has its own methods which you can call.

|Name|Params|Returns|Info|
|---|---|---|---|
|`->save()`                         | (none) |`bool` (true/false)    |Saves document in current state|
|`->delete()`                       | (none) |`bool` (true/false)    |Deletes current document (can not be undone)|
|`->toArray()`                      | (none) |array of items in document    | |
|`->getId()`                        | (none) |Document Id    | |
|`->createdAt()`                    | `Date` Format |Date Format (default Y-m-d H:i:s)    |When document was created|
|`->updatedAt()`                    | `Date` Format |Date Format (default Y-m-d H:i:s)    |When document was updated|
|`->field('main_key.nested_key')`   | `key` (a key from the array which you want to find)		|(string) Value of field    |You can also use `.` dot delimiter to find values from nested arrays |
|`->customFilter()`                 | `field`, `closure` |Items that matched filter criteria   |Refer to the [Custom Filters](https://github.com/timothymarois/Filebase#6-custom-filters)|

Example:

```php
// get the timestamp when the user was created
echo $db->get($user_id)->createdAt();
```


## (4) Create | Update | Delete

As listed in the above example, its **very simple**. Use `$item->save()`, the `save()` method will either **Create** or **Update** an existing document by default. It will log all changes with `createdAt` and `updatedAt`. If you want to replace *all* data within a single document pass the new data in the `save($data)` method, otherwise don't pass any data to allow it to save the current instance.

```php

// SAVE or CREATE
// this will save the current data and any changed variables
// but it will leave existing variables that you did not modify unchanged.
// This will also create a document if none exist.
$item->title = 'My Document';
$item->save()

// This will replace all data within the document
// Allows you to reset the document and put in fresh data
// Ignoring any above changes or changes to variables, since
// This sets its own within the save method.
$item->save([
    'title' => 'My Document'
]);

// DELETE
// This will delete the current item
// This action can not be undone.
$item->delete();

```

## (5) Validation *(optional)*

When invoking `save()` method, the document will be checked for validation rules (if set).
These rules MUST pass in order for the document to save. The rules will shoot off an error message if validation has failed.

```php
$db = new \Filebase\Database([
    'dir' => '/path/to/database/dir',
    'validate' => [
        'name'   => [
            'type' => 'string',
            'required' => true
        ],
        'description' => [
            'type' => 'string',
            'required' => false
        ],
        'emails' => [
            'type'     => 'array',
            'required' => true
        ],
        'config' => [
            'settings' => [
                'type'     => 'array',
                'required' => true
            ]
        ]
    ]
]);
```

In the above example `name`, `description`, `emails` and `config` array keys would be replaced with your own that match your data. Notice that `config` has a nested array `settings`, yes you can nest validations.

**Validation rules:**

|Name				|Allowed Values		|Description		                |
|---				|---		                                            |---		|
|`type`				|`string`, `str`, `integer`, `int`, `array`		|Checks if the variable is the current type		|
|`required`			|`true`, `false`		                                |Checks if the variable is on the document		|


## (6) Custom Filters

*NOTE Custom filters only run on a single document*

Item filters allow you to customize the results, and do simple querying. These filters are great if you have an array of items within one document. Let's say you store "users" as an array in `users.json`, then you could create a filter to show you all the users that have a specific tag, or field matching a specific value.

This example will output all the emails of users who are blocked.

```php
$users = $db->get('users')->customFilter('data',function($item) {
    return (($item['status']=='blocked') ? $item['email'] : false);
});
```


## Why Filebase?

I originally built Filebase because I needed more flexibility, control over the database files, how they are stored, query filtration and a design with very intuitive API methods.

Inspired by [Flywheel](https://github.com/jamesmoss/flywheel) and [Flinetone](https://github.com/fire015/flintstone).

## Contributions

Accepting contributions and feedback. Send in any issues and pull requests.


## TODO

- Indexing (adding indexed "tags" for all document searching)
- Indexing (single document filtering, applied with all `save()` actions from validation closure)
- Querying (searching for fields, and pulling in multiple doc results)
- Infinite Custom Filter Search (ability to access a filter from validation)
- Auto-Increment ID or Create a hash ID
