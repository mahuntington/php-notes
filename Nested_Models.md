# Nesting Models

## Objectives

Currently, we only have two unrelated models: people and locations.  Let's add the functionality so that whenever we have JSON that represents a person, it will contain a `home` property, which is the associated `location` object:

```JSON
{
    "id":4,
    "name":"Bob",
    "age":25,
    "home": {
        "id":2,
        "street": "123 Fake Street",
        "city": "Aurora",
        "state": "NY"
    }
}
```

Also, when we have a location, it will contain an array of people objects:

```JSON
{
    "id":2,
    "street": "123 Fake Street",
    "city": "Aurora",
    "state": "NY",
    "inhabitants":[
        {
            "id":4,
            "name":"Bob",
            "age":25
        },        
        {
            "id":7,
            "name":"Sally",
            "age":74
        },        
    ]
}
```

This will be a one-to-many relationship.

## Add home_id to people

First, go into `psql` and add a `home_id` INT column to the `people` table:

```sql
ALTER TABLE people ADD COLUMN home_id INT;
```

Now we can give people homes like so:

```sql
UPDATE people SET home_id = 2 WHERE id = 4;
```

The previous code will find the row with an `id` of 4 and set the `home_id` column to 2.

## Update People SQL

The first thing we want to do is to update the SQL code in the `People` class so that each row from the `people` table is joined onto a corresponding row from the `locations` table.  Once we have the additional columns for each row in the `people` table, we can use that information to create a home for each `Person` object.  Try the following in `psql`:

```SQL
SELECT * FROM people JOIN locations ON people.home_id = locations.id;
```

The above SQL statement will only show rows where the people have a value in the `home_id` column.  This is because Postgres is attempting to create rows by matching rows in the `people` table with rows in the `locations` table, based on whether `people.home_id = locations.id`.  When it encounters a row from the `people` table where the `home_id` column is `NULL`, it can't find any rows from the `locations` table to match it with, since no rows from `locations` have an `id` column of `NULL`.  Since no matches can be found, it doesn't include that row from `people`.  We need to perform the join and then add any missing rows from `people`:

```SQL
SELECT * FROM people LEFT JOIN locations ON people.home_id = locations.id;
```

Now that we have all the people rows, we're ready to plug that into our php.  Alter the `pg_query` code in our `People::all()` function:

```php
$results = pg_query("SELECT * FROM people LEFT JOIN locations ON people.home_id = locations.id");
```

If you view http://localhost:8888/people, you'll see something funky.  Some people have 0 ids, and other rows have unexpected `id` columns.  The reason for this can be discovered by looking back in `psql` at our previous `LEFT JOIN` statement results.  You'll notice there are two `id` columns, one for the `people` table and one for `locations` table.  When PHP attempts to convert a row into an object, it reads the `people` table's `id` column and creates an `id` property for the object.  It then goes through, adding `name`, `age`, and `home_id` properties.  It then reaches the `id` column for the `locations` table and overwrites the `id` property on the `$row_object` with the value from the `id` column of the `locations` table.

To fix this, we can alter our SQL statement to rename one or both of the id columns.  Let's rename the `id` column from the `locations` table.

```SQL
SELECT
    people.*,
    locations.id AS location_id,
    locations.street,
    locations.city,   
    locations.state   
FROM people
LEFT JOIN locations
    ON people.home_id = locations.id;
```

Now if we change our SQL statement in PHP, we'll see better results:

```php
$results = pg_query("SELECT
    people.*,
    locations.id AS location_id,
    locations.street,
    locations.city,
    locations.state
FROM people
LEFT JOIN locations
    ON people.home_id = locations.id;");
```

## Give People Homes

If we put a `var_dump` inside the while loop in our `People::all()` function, we can see `$row_object` now contains information about each person's location (if you're using Postman to view http://localhost:8888/people, you might need to switch to the `Raw` view).

```php
while($row_object){
    var_dump($row_object); //print $row_object so we can see what it looks like now

    $new_person = new Person(
        $row_object->id,
        $row_object->name,
        $row_object->age
    );
    $people[] = $new_person;

    $row_object = pg_fetch_object($results);
}
```

Now lets use the additional information on `$row_object` to create Location objects and add them to `$new_person` where necessary.  First, let's include the `Location` model at the top of `models/person.php`.

```php
include_once __DIR__ . '/location.php';
```

Now, inside the `People::all()` `while` loop, we'll add the logic to create a location and add it to `$new_person`:

```php
while($row_object){
    $new_person = new Person(
        $row_object->id,
        $row_object->name,
        $row_object->age
    );

    if($row_object->location_id){ //test if location_id is truthy
        $new_location = new Location( //create a location from the row data
            intval($row_object->location_id), //turn the string into an int
            $row_object->street,
            $row_object->city,
            $row_object->state
        );
        $new_person->home = $new_location; //attach $new_location to $new_person->home
    }

    $people[] = $new_person;

    $row_object = pg_fetch_object($results);
}
```

Here we test to see if `$row_object->location_id` is truthy.  If it has a value -- a string representation of the id column of the `locations` table -- then the block of code belonging to the `if` statement will run, creating `$new_location` and attaching it to the `home` property of `$new_person`.

## Give Locations Inhabitants

Again, let's start with some SQL to get all of our locations, with their respective inhabitants:

```SQL
SELECT * FROM locations JOIN people ON locations.id = people.home_id;
```

But this only gets the locations that have people associated with them.  If there aren't any rows in the `people` table that have the `home_id` column set to the `id` of a specific row in the `locations` table, then the row won't show up.  It's the same issue we had previously with some of the rows in the `people` column not showing up because they didn't have a `home_id`.  We need to perform the join like normal, and then add in any missing `location` rows:

```SQL
SELECT * FROM locations LEFT JOIN people ON locations.id = people.home_id;
```

Now we have the additional missing rows from the `locations` table.

Currently, our `Locations:all()` should look something like this:

```php
static function all(){
    //create an empty array
    $locations = array();

    //query the database
    $results = pg_query("SELECT * FROM locations");

    $row_object = pg_fetch_object($results);
    while($row_object){

        $new_location = new Location( //create a new location
            $row_object->id,
            $row_object->street,
            $row_object->city,
            $row_object->state
        );
        $locations[] = $new_location; //push new person object onto $$locations array

        $row_object = pg_fetch_object($results);
    }

    return $locations;
}
```

Let's adjust the code to include our new SQL code with the `JOIN`:


```php
$results = pg_query("SELECT * FROM locations LEFT JOIN people ON locations.id = people.home_id");
```

If you visit http://localhost:8888/locations, you'll notice that we now have duplicate locations whenever a `location` has more than one person associated with it.  This makes sense because we have extra rows for those locations when we perform the SQL query in `psql`.  We'll fix that soon.

You'll also notice that the `id` properties for the various locations are incorrect.  This is the same problem that we encountered before when we were creating `People:all()`.  If we look at the SQL statement in `psql`, we'll notice that there are two `id` columns.  The second `id` column overwrites the values of the first one, even though the correct `id` is the first one.  Let's alias the second column:

```sql
SELECT
    locations.*,
    people.id AS person_id,
    people.name,
    people.age
FROM locations
LEFT JOIN people
    ON locations.id = people.home_id;
```

We can plug this into our PHP code:

```php
$results = pg_query("SELECT
    locations.*,
    people.id AS person_id,
    people.name,
    people.age
FROM locations
LEFT JOIN people
    ON locations.id = people.home_id;");
```

Now our `id` properties are correct.

Next, let's deal with our duplicate locations.  Currently, our `while` loop looks like this:

```php
$row_object = pg_fetch_object($results);
while($row_object){

    $new_location = new Location(
        $row_object->id,
        $row_object->street,
        $row_object->city,
        $row_object->state
    );
    $locations[] = $new_location;

    $row_object = pg_fetch_object($results);
}
```

The issue here is that our SQL produces multiple rows for the same location, if it has more than one person associated with it.  This is okay, because we need multiple rows to exist in order to have data for each person associated with a given location.  What we need to do is, each time a duplicate location row is encountered, skip the process for creating the location:

```php
$row_object = pg_fetch_object($results);
$last_location_id = null; //keep track of the last location id encountered
while($row_object){

    //if the current row's id is different from the last row's id, create a new location
    //otherwise skip the process of creating a new location
    if($row_object->id !== $last_location_id){
        $new_location = new Location(
            $row_object->id,
            $row_object->street,
            $row_object->city,
            $row_object->state
        );
        $locations[] = $new_location;
        $last_location_id = $row_object->id; //update the $last_location_id to be the current row's id
    }

    $row_object = pg_fetch_object($results);
}
```

We start off by initializing a `$last_location_id` variable.  Then, each time we loop through to the next row, we check to see if the new row's id is the same as the last row's id.  If it is, we don't create a new `Location` object.

With that working, let's add people to locations when necessary.  This is similar to, but not the same as, the last section when we created `Location` objects and added them to people.  First, whenever we create a new `Location` object, we want to automatically give it an inhabitants property which is set to an array:

```php
class Location {
    public $id;
    public $street;
    public $city;
    public $state;
    public function __construct($id, $street, $city, $state) {
        $this->id = $id;
        $this->street = $street;
        $this->city = $city;
        $this->state = $state;
        $this->inhabitants = [];
    }
}
```

Now include the `People` model at the top of `models/location.php`:

```php
include_once __DIR__ . '/person.php';
```

Next, update the `while` loop in `Locations::all()`:

```php
while($row_object){

    if($row_object->id !== $last_location_id){
        $new_location = new Location(
            $row_object->id,
            $row_object->street,
            $row_object->city,
            $row_object->state
        );
        $locations[] = $new_location;
        $last_location_id = $row_object->id;
    }

    //test if person_id is truthy
    if($row_object->person_id){
        //create a person from the row data
        $new_person = new Person(
            intval($row_object->person_id), //turn the string into an int
            $row_object->name,
            $row_object->age
        );

        //add the new person as an inhabitant of the last element of the locations array
        $locations_length = count($locations); //count() returns the # of elements in an array
        $last_index_of_locations = $locations_length-1;
        $most_recently_added_location = $locations[$last_index_of_locations];
        $most_recently_added_location->inhabitants[] = $new_person;
    }

    $row_object = pg_fetch_object($results);
}
```

This is pretty similar to when we added homes to people, except for the last section.  It finds last element in the `$locations` array and adds the `$new_person` as an inhabitant to it.

If we sort our results in the SQL statement by `locations.id`, we can be assured that whenever we create a new `Person` object, even if we didn't create a location during that particular loop of the `while` statement, the last element on the `$locations` array will always be the location that needs to have an inhabitant added it it (as opposed to some other location).

```php
$results = pg_query("SELECT
    locations.*,
    people.id AS person_id,
    people.name,
    people.age
FROM locations
LEFT JOIN people
    ON locations.id = people.home_id
ORDER BY locations.id ASC");
```
