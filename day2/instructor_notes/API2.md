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
    $query_params = array($person->name, $person->age); //add this array
}
```

Lastly, let's combine the SQL `$query` statement and the params:

```php
static function create($person){
    $query = "INSERT INTO people (name, age) VALUES ($1, $2)";
    $query_params = array($person->name, $person->age);
    pg_query_params($query, $query_params); //pass the query and the params to pg_query_params
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
} else if ($_REQUEST['action'] === 'post'){ //add this for post requests
    echo '{"test":true}'; //send back a test message
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
    echo file_get_contents('php://input'); //get the request body
}
```

Now if you go into Postman can add JSON to the body of the POST request to http://localhost:8888/people, you should see the same JSON data come back in the response.

Currently, `file_get_contents('php://input')` just returns a string.  Let's turn that into an object, so that we can easily retrieve values from it:

```php
if($_REQUEST['action'] === 'index'){
    echo json_encode(People::all());
} else if ($_REQUEST['action'] === 'post'){
    $request_body = file_get_contents('php://input');
    $body_object = json_decode($request_body); //change the request body from a JSON string into a PHP object
}
```

`json_decode()` will take the `$request_body` JSON string and convert it into an object

Don't forget that `People::create()`, takes a `Person` object as a parameter.  Let's create a `Person` object from the `$body_object`:

```php
if($_REQUEST['action'] === 'index'){
    echo json_encode(People::all());
} else if ($_REQUEST['action'] === 'post'){
    $request_body = file_get_contents('php://input');
    $body_object = json_decode($request_body);
    $new_person = new Person(null, $body_object->name, $body_object->age); //create a new Person from $body_object
}
```

Finally, we can pass `$new_person` off to `People::create()`:

```php
if($_REQUEST['action'] === 'index'){
    echo json_encode(People::all());
} else if ($_REQUEST['action'] === 'post'){
    $request_body = file_get_contents('php://input');
    $body_object = json_decode($request_body);
    $new_person = new Person(null, $body_object->name, $body_object->age);
    People::create($new_person); //pass $new_person off to People, so it can add the data to the db
    echo '{"worked":true}'; //return a success message
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

```SQL
SELECT * FROM people;
```

### Return data to the user

Currently, nothing helpful comes back to the user after they create a new person.  Let's change this to show all the users currently in the DB.  Add the following to `models/person.php` for the `create()` method:

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
    $new_person = new Person(null, $body_object->name, $body_object->age);
    $all_people = People::create($new_person); //store the return value of People::create into a var

    //send the return value of People::create (all people in the db) back to the user
    echo json_encode($all_people);
}
```

Now when you create a new person in Postman, you should get back all the People currently in the DB as a response.

## Update

### Set up the model

We're going to do the same as with `People::create()`, but with some minor changes.  Add the following to the `People` model in `models/person.php`:

```php
static function update($updated_person){
    $query = "UPDATE people SET name = $1, age = $2 WHERE id = $3";
    $query_params = array($updated_person->name, $updated_person->age, $updated_person->id);
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

On that same second line of the route, you'll notice `&id=$1` at the end of the rule.  This adds a second query parameter to `controllers/people.php` called `id` and sets it to whatever is inside the `()` of `^people/([0-9]+)$`.  In other words, if the URL is `people/2347346`.  The `id` query param will be 2347346.

Now let's update `controllers/people.php` to handle these requests.  Add the following at the end of your `if/else` section (note you'll have to remove an extra `}`):

```php
} else if ($_REQUEST['action'] === 'update'){
    $request_body = file_get_contents('php://input');
    $body_object = json_decode($request_body);
    $updated_person = new Person($_REQUEST['id'], $body_object->name, $body_object->age);
    $all_people = People::update($updated_person);

    echo json_encode($all_people);
}
```

This is very similar to the create action.  The only real difference is that we use `$_REQUEST['id']` to fetch the id of the person to be updated from the URL of the route.  Everything else for the new `Person` object comes from the request body as normal.

Note that we're not actually creating a new `Person` object in the database, even though we have `$updated_person = new Person($_REQUEST['id'], $body_object->name, $body_object->age);`.  Here, `$updated_person` is a new PHP object that resides in the computer's temporary memory, not in the DB.  We're temporarily creating this PHP object so that we can pass it to `People::update()`, which will then use the properties of that PHP object to update an already pre-existing row in Postgres.  Once we exit from the `else if` statement, `$updated_person` is destroyed in memory, since it is no longer needed.

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

Note that the `$id` is just going to be an integer that we pass into `People:delete()`.  In previous examples, it was an entire `Person` object, but we don't need that here.  Just the id of the person to be deleted.  Also, note that even if we only have one query param, we still need to put it in an array.

### Hook the controller up with the model

Add the following to `.htaccess`:

```
RewriteCond %{REQUEST_METHOD} ^DELETE$
RewriteRule ^people/([0-9]+)$ controllers/people.php?action=delete&id=$1
```

This is similar to update, but with the request method being DELETE and the `action` query param set to `delete`.

Now let's update `controllers/people.php` to handle these requests.  Add the following at the end of your `if/else` block (removing the extra `}` again):

```php
} else if ($_REQUEST['action'] === 'delete'){
    $all_people = People::delete($_REQUEST['id']);
    echo json_encode($all_people);
}
```

Note that we don't need to do anything with the request body.  We just pass `$_REQUEST['id']` to `People::delete()`.  Don't forget that `$_REQUEST['id']` is the query parameter we set up in `.htaccess`.
