# Create an API

## Route URLs to php files

Routing can be accomplished with a `.htaccess` file placed in your root directory for the app.  This is not PHP related.  It's actually something that is a part of Apache.  It allows the user to take requests and change them in some way, so that the server thinks the original request came in with the modifications specified.  For instance, you could take all requests to `/people/1` and rewrite it to `/people?id=1`.  We'll use this file to specify how our routes map to particular files.

First, tell Apache to allow the ability to rewrite URLs.  Create a `.htaccess` file and add the following:

```
RewriteEngine On
```

Now add a route after the previous line that will map any `GET` request to `/people` to `people.php`

```
RewriteCond %{REQUEST_METHOD} ^GET$
RewriteRule ^people$ controllers/people.php?action=index
```

Let's break down the various parts of the first line of the last chunk (`RewriteCond %{REQUEST_METHOD} ^GET$`):

- `RewriteCond`
    - this specifies that a rewrite will happen given a following condition
- `%{REQUEST_METHOD}`
    - this tells apache to look at the request method (GET, POST, PUT/PATCH, DELETE) and compare it to something
- `^GET$`
    - this is what the request method will be compared to.  In this case, it's the regular expression `^GET$`.  The `^` denotes the beginning of the string and the `$` denotes the end of it.
    - basically, it checks to see if the request method is GET

Now let's break down the second line (`RewriteRule ^people$ controllers/people.php?action=index`):

- `RewriteRule`
    - this tells Apache that a rewrite rule follows
- `^people$`
    - again this is a regular expression where `^` denotes the beginning of the string and `$` denotes the end of the string
    - this says to map any request with a URL that matches `people` to the file that comes next
- `controllers/people.php`
    - this is the "controller" file that we'll create shortly.  It will handle all routes concerning people
- `?action=index`
    - this says to pass a query parameter to that file called `action` with the value `index`.  You'll see later that we'll use this to query parameter to determine which action (index, show, create, update, delete, etc...) to render

## Create a controller file

Create a `controllers` directory to hold our controller files.  Within that directory, create a `people.php` file.  This will handle all routes that pertain to people models.

In this file put the following PHP code (**NOTE:** I've ommitted `<?php` and `?>` for ease of reading.  Don't forget to add each to the beginning/end of **every** file you create from here on out):

```php
if($_REQUEST['action'] === 'index'){
    echo "index route";
}
```

Go to http://localhost:8888/people to see this in action.  Apache will look at our `.htaccess` file and rewrite the request internally, acting as if the original request was `http://localhost:8888/controllers/people.php?action=index`

The PHP code we wrote looks at the `action` query parameter that our `.htaccess` file tells Apache to create.  If its value equals "index", we're going to render the text "index route" to the browser.  It does this with the `$_REQUEST` variable, which we'll look at next.

Every time we use a PHP file to render something server-side, that file has access to several global variables that contain information about the request that was made.  `$_REQUEST` is an associative array that has info about the query parameters that were used in the request. To retrieve the `action` query parameter that our `.htaccess` file told Apache to create, we can just look at `$_REQUEST['action']`.

Later on, when we have more routes, we'll have the `.htaccess` file set the `action` query parameter to different values depending on what the route is (e.g. show, delete, update, etc).  Then in the controller files, we'll test what that value is and use that to determine what JSON to render.  Right now we only have `index` set up in our `.htaccess` file, so our `if` statement is a little unnecessary, but later on in will become important.

## Create a model file

Now we're going to a create a file which will eventually query the relational database and turn that information into PHP objects.  We call that an Object Relational Mapper (ORM).  First, create a `models` directory, and inside of that create `person.php`.  Add the following code to that file, which will define the `Person` class used to create people objects:

```php
class Person {
    public $id;
    public $name;
    public $age;
    public function __construct($id, $name, $age) {
        $this->id = $id;
        $this->name = $name;
        $this->age = $age;
    }
}
```

This defines the `Person` class as having three possible properties (id, name, age) which can be edited after the object has been created.

Now, we're going to create a "factory" which will eventually be responsible for generating objects from the database.  For now, we're just going to create some dummy data.  Add the following to the bottom of `models/person.php`:

```php
class People {
    static function all(){
        //create an empty array
        $people = array();

        //create some random people
        $person1 = new Person(1,'Bob', 32);
        $person2 = new Person(1,'Bob', 32);
        $person3 = new Person(1,'Bob', 32);

        //push the people onto the array
        $people[] = $person1;
        $people[] = $person2;
        $people[] = $person3;

        //return the array of people
        return $people;
    }
}
```

## Use the people model in the controller

Now that we have our `People`/`Person` model set up, we need to incorporate it into our `People` controller.  First, include it at the top of `controllers/people.php`:

```php
include_once __DIR__ . '/../models/person.php';
```

- `include_once` will include the file, unless it's already been included somewhere else in our app.  We could use `include`, but `include_once` keeps us from accidentally running `models/people.php` more than once.
- `__DIR__` just spits out the full absolute path to the current file that's running that line of code
- `/../models/person.php`: since we're currently in the `controllers` directory, we need to travel up to the parent directory of the `controllers` directory (the project's root dir) and then into the `models/` directory.  We then reference the `person.php` file in that directory

Now that we wrote that code, we have our `Person` and `People` classes available to us.  Let's replace

```php
echo "index route";
```

with

```php
echo json_encode(People::all());
```

This will render the results of `People::all()` (an array of Person objects) as JSON.  Refresh http://localhost:8888/people to see the difference.


## Add a Content-Type header

Most JavaScript libraries/frameworks that deal with AJAX (e.g. Angular, Axios, fetch, jQuery, etc) expect a special header to be set in all AJAX responses, telling the requesting client what kind of data is being sent back.  Let's add this to the top of `controllers/people.php`:

```php
header('Content-Type: application/json');
```

This is meta data meant just for the client application (the browser).  It isn't part of the response body, so the end user won't see it.  If you look at http://localhost:8888/people, you won't see anything about `Content-Type: application/json`, but it will be available to your AJAX libraries/frameworks.

**NOTE**, this code has to come at the top of the file, before any content is written to response body.  

## Prepare Postgres for the app

In order to have PHP interact with Postgres, there are a few things we need to do before hand

First, start postgres.  In the terminal, type:

```
postgres -D /usr/local/var/postgres/
```

Next, open up a new terminal tab/window and type:

```
psql postgres
```

Now we want to create a sub-database for our application (we'll name it `contacts`).  In the `psql` tab, type:

```sql
CREATE DATABASE contacts;
```

and now connect to it:

```sql
\c contacts
```

Now we want to create a table for our people:

```sql
CREATE TABLE people (id SERIAL, name VARCHAR(16), age INT);
```

Let's insert some people:

```sql
INSERT INTO people ( name, age ) VALUES ( 'Matt', 38 );
INSERT INTO people ( name, age ) VALUES ( 'Sally', 54 );
INSERT INTO people ( name, age ) VALUES ( 'Zanthar', 4892 );
```

## Connect the People model to Postgres

Currently, our `People` model's `all` function arbitrarily generates three `Person` objects when invoked.  Let's have it connect to `postgres` so that it can use the rows we inserted into the `people` table to create the `Person` objects.  At the top of `models/person.php` add the following:

```php
$dbconn = pg_connect("host=localhost dbname=contacts");
```

Now that we're connected to Postgres, we can have PHP query the database.  In `models/person.php`, let's alter our `all` function:

```php
class People {
    static function all(){
        //create an empty array
        $people = array();

        //query the database
        $results = pg_query("SELECT * FROM people");

        return $people;
    }
}
```

Our `$results` variable is a result set.  It's basically like a PHP version of the rows retrieved from Postgres.  Now we'll use the `pg_fetch_object` function to remove the first row from the `$results` dataset and turn it into an object:

```php
$results = pg_query("SELECT * FROM people");
$row_object = pg_fetch_object($results); //remove a row from the results and turn it into an object
```

To see what our `$row_object` looks like, let's add the following:

```php
$results = pg_query("SELECT * FROM people");
$row_object = pg_fetch_object($results);
var_dump($row_object); //print values of the object
die(); //halt execution
```

Now we can see the various values that are available to us for the row that was returned.

- `var_dump` will take any object and print its values
- `die` will halt execution from continuing so that you can read what was printed

If we keep removing rows from `$results`, we'll eventually run out of rows.  When this happens, `pg_fetch_object` returns `false`

```php
$results = pg_query("SELECT * FROM people");

$row_object = pg_fetch_object($results);
var_dump($row_object);

$row_object = pg_fetch_object($results);
var_dump($row_object);

$row_object = pg_fetch_object($results);
var_dump($row_object);

$row_object = pg_fetch_object($results);
var_dump($row_object);

die();
```

Let's change this so it does this with a `while` loop.  This way, no matter how many rows are in the results, all will be printed:

```php
class People {
    static function all(){
        //create an empty array
        $people = array();

        //query the database
        $results = pg_query("SELECT * FROM people");

        $row_object = pg_fetch_object($results);
        while($row_object){
            var_dump($row_object);
            $row_object = pg_fetch_object($results);
        }
        die();

        return $people;
    }
}
```

Now, as long as `$results` has rows, `while($row_object)` will run its code. As soon as all the rows have been removed, `$row_object` will be `false` and the `while` loop will exit.

Now that we can see the various rows in our `$results` as objects, we can use this info to create `Person` objects which we'll push onto our `$people` array.

```php
class People {
    static function all(){
        //create an empty array
        $people = array();

        //query the database
        $results = pg_query("SELECT * FROM people");

        $row_object = pg_fetch_object($results);
        while($row_object){

            $new_person = new Person( //create a new person
                $row_object->id,
                $row_object->name,
                $row_object->age
            );
            $people[] = $new_person; //push new person object onto $people array

            $row_object = pg_fetch_object($results);
        }

        return $people;
    }
}
```

One thing you might notice is that the `id` and `age` values for each person are strings.  Let's change those into integers:

```php
$new_person = new Person(
    intval($row_object->id),
    $row_object->name,
    intval($row_object->age)
);
```
