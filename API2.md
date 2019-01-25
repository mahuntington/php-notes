# Create an API - CREATE, DELETE, UPDATE

## Create

### Set up the model

First we want to create a static method in the `People` factory class that will take a `Person` object as a parameter and insert its values into Postgres.  Add the following in `models/person.php` as a static method to `People`:

```php
static function create($person){
    $query = "INSERT INTO people (name, age) VALUES ($1, $2)";
}
```

`$query = "INSERT INTO people (name, age) VALUES ($1, $2)";` looks complex, but it's just assigning a string to a variable.  The string is normal SQL, but with one minor change: the `$1` and `$2` stand for dynamic parameters which will be inserted into the SQL statement.  This way, we can easily insert values for name and age into the SQL statement.

To add these dynamic parameters, we'll need to create an array that has the values we want for name and age.  The first element in the array will be matched with `$1` in the `$query` SQL string and the second element in the array will be matched with `$2`.  Note that the numbers in the SQL statement don't match the array indexes (e.g. `$1` matches the element at index 0).  Let's populate the array with the `name` and `age` properties of the `$person` parameter that is passed to `create()`.

```php
static function create($person){
    $query = "INSERT INTO people (name, age) VALUES ($1, $2)";
    $query_params = array($person->name, $person->age);
}
```

Lastly, let's combine the SQL `$query` statement and the params:

```php
static function create($person){
    $query = "INSERT INTO people (name, age) VALUES ($1, $2)";
    $query_params = array($person->name, $person->age);
    pg_query_params($query, $query_params);
}
```

At this point, the SQL statement will be executed properly when `create()` is called and passed an appropriate `Person` object.  Next we'll hook up the controller to call this method.

### Hook the controller up with the model

First create a route in our `.htaccess` file:

```
RewriteCond %{REQUEST_METHOD} ^POST$
RewriteRule ^people$ controllers/people.php?action=post
```

This is basically the same as what we did for the `index` route, but it checks to see if the request method is `POST`.  It then rewrites the rule as before, but sets the `action` query parameter to `post`.

Now inside our `controllers/people.php` let's add an additional condition:

```php
if($_REQUEST['action'] === 'index'){
    echo json_encode(People::all());
} else if ($_REQUEST['action'] === 'post'){
    echo '{"test":true}';
}
```

Now if we make a POST `request` to http://localhost:8888/people in Postman, we should get back the following:

```json
{
    "test": true
}
```

The next thing we want to do is get the body of the request.  Soon, we'll extract data from the request body regarding the person that the user wants to be created.  We'll then pass that data to `People::create()` so that the new person can be stored in the DB.  For now let's just retrieve the body and send it back out to the user:

```php
if($_REQUEST['action'] === 'index'){
    echo json_encode(People::all());
} else if ($_REQUEST['action'] === 'post'){
    echo file_get_contents('php://input');
}
```

Now if you go into Postman can add JSON to the body of the POST request to http://localhost:8888/people, you should see the same JSON data come back in the response.

Currently, `file_get_contents('php://input')` just returns a string.  Let's turn that into an object, so that we can easily retrieve values from it:

```php
if($_REQUEST['action'] === 'index'){
    echo json_encode(People::all());
} else if ($_REQUEST['action'] === 'post'){
    $request_body = file_get_contents('php://input');
    $body_object = json_decode($request_body);
}
```

`json_decode()` will take the `$request_body` string and convert it into an object

Don't forget that `People::create()`, takes a `Person` object as a parameter.  Let's create a `Person` object from the `$body_object`:

```php
if($_REQUEST['action'] === 'index'){
    echo json_encode(People::all());
} else if ($_REQUEST['action'] === 'post'){
    $request_body = file_get_contents('php://input');
    $body_object = json_decode($request_body);
    $newPerson = new Person(null, $body_object->name, $body_object->age);
}
```

Finally, we can pass `$newPerson` off to `People::create()`:

```php
if($_REQUEST['action'] === 'index'){
    echo json_encode(People::all());
} else if ($_REQUEST['action'] === 'post'){
    $request_body = file_get_contents('php://input');
    $body_object = json_decode($request_body);
    $newPerson = new Person(null, $body_object->name, $body_object->age);
    People::create($newPerson);
    return '{"worked":true}';
}
```

Now, go to Postman and make a POST request to http://localhost:8888/people with an appropriate body like this one:

```json
{
  "name": "Jimbo",
  "age": 71
}
```

Check in `psql` to see if the person was created.

### Return data to the user

Currently, nothing comes back to the user after they create a new person.  Let's change this to show all the users currently in the DB.  Add the following to `models/person.php` for the `create()` method:

```php
static function create($person){
    $query = "INSERT INTO people (name, age) VALUES ($1, $2)";
    $query_params = array($person->name, $person->age);
    pg_query_params($query, $query_params);
    return self::all(); //find all people and return them
}
```

Now in `controllers/people.php` send the return value of `People::create()` back to the user:

```php
if($_REQUEST['action'] === 'index'){
    echo json_encode(People::all());
} else if ($_REQUEST['action'] === 'post'){
    $request_body = file_get_contents('php://input');
    $body_object = json_decode($request_body);
    $newPerson = new Person(null, $body_object->name, $body_object->age);
    $allPeople = People::create($newPerson);

    echo json_encode($allPeople);
}
```

Now when you create a new person in Postman, you should get back all the People currently in the DB as a response.

## Update

### Set up the model

We're going to do the same as with `People::create()`, but with some minor changes.  Add the following to the `People` model in `models/person.php`:

```php
static function update($updatedPerson){
    $query = "UPDATE people SET name = $1, age = $2, WHERE id = $3";
    $query_params = array($updatedPerson->name, $updatedPerson->age, $updatedPerson->id);
    $result = pg_query_params($query, $query_params);

    return self::all();
}
```

Pay careful attention to the order of the parameters in the `$query_params` array.  `id` comes last because it was assigned to `$3` in the SQL statement

### Hook the controller up with the model

Again, the route in `.htaccess` should be pretty similar to the `POST` route:

```
RewriteCond %{REQUEST_METHOD} ^PUT$
RewriteRule ^people/([0-9]+)$ controllers/people.php?action=update&id=$1
```

The first difference you'll see is `([0-9]+)`.  This is just more regex work.  It basically means any integer.  If you're interested in learning more about how this works, check out [these tutorials on regex](https://www.regular-expressions.info/tutorial.html).  What it allows us to do is have a route for urls like `people/123`, `people/5`, or `people/2347346`, etc.

One that same line second line of the route, you'll notice `&id=$1` at the end of the rule.  This adds a second query parameter to `controllers/people.php` called `id` and sets it to whatever is inside the `()` of `^people/([0-9]+)$`.  In other words, if the URL is `people/2347346`.  The id query param will be 2347346.

Now let's update `controllers/people.php` to handle these requests.  Add the following:

```php
} else if ($_REQUEST['action'] === 'update'){
    $request_body = file_get_contents('php://input');
    $body_object = json_decode($request_body);
    $updatedPerson = new Person($_REQUEST['id'], $body_object->name, $body_object->age);
    $allPeople = People::update($updatedPerson);

    echo json_encode($allPeople);
}
```

This is very similar to the create action.  The only real difference is that we use `$_REQUEST['id']` to fetch the id of the person to be updated from the URL of the route.  Everything else for the new `Person` object comes from the request body as normal.

## Delete

### Set up the model

Add the following to the `People` model in `models/person.php`:

```php
static function delete($id){
    $query = "DELETE FROM people WHERE id = $1";
    $query_params = array($id);
    $result = pg_query_params($query, $query_params);

    return self::all();
}
```

Note that the `$id` is just going to be an integer that we pass into `People:delete()`.  Even if we only have one query param, we still need to put it in an array.

### Hook the controller up with the model

Add the following to `.htaccess`:

```
RewriteCond %{REQUEST_METHOD} ^DELETE$
RewriteRule ^people/([0-9]+)$ controllers/people.php?action=delete&id=$1
```

This is similar to update, but with the request method being DELETE and the `action` query param set to `delete`.

Now let's update `controllers/people.php` to handle these requests.  Add the following:

```php
} else if ($_REQUEST['action'] === 'delete'){
    $allPeople = People::delete($_REQUEST['id']);
    echo json_encode($allPeople);
}
```

Note that we don't need to do anything with the request body.  We just pass `$_REQUEST['id']` to `People::delete()`.  Don't forget that `$_REQUEST['id']` is the query parameter we set up in `.htaccess`.
