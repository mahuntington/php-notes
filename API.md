# Create an API

## Route URLs to php files

Routing can be accomplished with a `.htaccess` file placed in your root directory for the app.  This is not PHP related.  It's actually something that is a part of Apache.  It allows the user to take requests and change them in some way, so that the server thinks the original request came in with modifications specified.  For instance you could take all requests to `/people/1` and rewrite it to `/people?id=1`.  We'll use this file to specify how our routes map to particular files.

First, tell Apache to allow the ability to rewrite URLs.  In `.htaccess`, add the following:

```
RewriteEngine On
```

Now create a route that will map any `GET` request to `/people` to `people.php`

```
RewriteCond %{REQUEST_METHOD} ^GET$
RewriteRule ^people$ controllers/people.php?action=index
```

Let's break down the various parts of the first line of the last chunk (`RewriteCond %{REQUEST_METHOD} ^GET$`):

- `RewriteCond`
    - this specifies that a rewrite will happen given the following condition
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

In this file put the following PHP code:

```php
if($_REQUEST['action'] === 'index'){
    echo "index route";
}
```

Go to http://localhost:8888/people to see this in action.  Apache will look at our `.htaccess` file and rewrite the request internally, acting as if the original request was `http://localhost:8888/controllers/people.php?action=index`

The PHP code we wrote looks at the `action` query parameter that our `.htaccess` file tells Apache to create.  If its value equals "index", we're going to render the text "index route" to the browser.  It does this with the `$_REQUEST` variable, which we'll look at next.

Every time we use a PHP file to render something server-side, that file has access to several global variables that contain information about the request that was made.  `$_REQUEST` is an associative array that has info about the query parameters that were used in the request. To retrieve the `action` query parameter that we told Apache to create in our `.htaccess` file, we can just look at `$_REQUEST['action']`.

Later on, when we have more routes, we'll have the `.htaccess` file set the `action` query parameter to different values depending on what the route is (e.g. show, delete, update, etc).  Then in the controller files, we'll test what that value is and use that to determine what JSON to render.  Right now we only have `index` set up in our `.htaccess` file, so our `if` statement is unnecessary, but later on in will become important.

## Create a model file

Now we're going to a create a file which will end up querying the relational database and turning that information into PHP objects.  We call that an Object Relational Mapper (ORM).  First, create a `models` directory, and inside of that create `person.php`.  Add the following code to that file, which will define the `Person` class used to create people objects:

```php
class Person {
    public $id;
    public $name;
    public $age;
}
```

This defines the `Person` class as having three possible properties (id, name, age) which can be edited after the object has been created.

Now, we're going to create a "factory" which will be responsible for generating objects from the database.  At the moment, we're just going to have to create some dummy data:

```php
class People {
    static function find(){
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

Now that we have our `People`/`Person` model set up, we need to incorporate it into our `People` controller.  First include it into our `controllers/people.php`:

```php
include_once __DIR__ . '/../models/person.php';
```

- `include_once` will include the file, unless it's already been included somewhere else in our app.  We could use `include`, but `include_once` keeps us from accidentally running the `models/people.php` more than once.
- `__DIR__` just spits out the full absolute path the directory of the current file that's running that line of code
- `/../models/person.php`: since we're currently in the `controllers` directory, we need to travel up a level to the project's root dir and then into the `models/` directory.  We then reference the `person.php` file in that directory

Now that we wrote that code, we have our `Person` and `People` classes available to us.  Let's replace

```php
echo "index route";
```

with

```php
echo json_encode(People::find());
```

This will render the results of `People::find()` (an array of Person objects) as JSON.  Refresh http://localhost:8888/people to see the difference.
