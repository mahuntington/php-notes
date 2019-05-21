[![General Assembly Logo](/ga_cog.png)](https://generalassemb.ly)

# HELLO - Start Building Your Own ORM

Use PHP to reshape the kinds of objects coming from a SQL database

#### Learning Objectives

- Learn about the challenges of converting SQL rows into JSON
- Practice using PHP to reshape objects
- etc.

#### Prerequisites

- Experience with RESTful routes
- Basic PHP
- Familiarity with JSON
- Familiarity with SQL

---

## Getting Started

1. You'll be working in one PHP file, and building out your answers as you work through this hw.


## Deliverables

## Technical Requirements

1. Must run without syntax errors (comment out code that doesn't work and add comments)
2. Must complete parts 1 - 6


## Submission Guidelines

- Must be submitted no later than before the start of next class

## Context

The founder of the start-up H.E.L.L.O (Housing Elite Leveraged Location Organization) wants you to help launch her amazing idea.

Too many college graduates these days have too high loans to pay back. This is preventing them from being able to get into premium housing locations, where everything is in walking distance and work is a sweet sweet short commute away.

One night, our brilliant entrepreneur was reminiscing about how much she enjoyed living in a dorm and thought, why can't people keep on living in a dorm?

And thus H.E.L.L.O. was founded. A premier housing solution. For the low low starting price of $3999 a month for a bunk bed shared with one other person and a co-ed bathroom shared by everyone on the same floor, anyone's housing dreams can come true!

H.E.L.L.O already has funding for one location!

Your first task is to help build a tenant tracker.

## Part 1 - explanation

SQL stores data in rows and columns, so in order to get the data to display on a web page, we need to convert the data from rows and columns into associative arrays with keys and values.  Fortunately, PHP does this for us, but we need to do some manual coding to get it to work exactly as we want it to.

If we just have one table with the following keys and values:

| name | age | home_id |
|:-:|:-:|:-:|
| Chase | 30 | 1 |

PHP can turn this into an associative array which would look something like this:

```php
[
    [
        "name" => "Chase",
        "age" => "30",
        "home_id" => "1"
    ]
]
```

Remember, that PHP doesn't allow us to spontaneously create an object with any arbitrary properties that we want like JavaScript does.  We can use associative in this manner, though.  It's a good way to keep track of a data structure that requires key/value pairs.

If we had more than one person...

| name | age | home_id |
|:-:|:-:|:-:|
| Chase | 30 | 1 |
| Gert | 23 | 1 |

PHP will give us something like this:

```php
[
    [
        "name" => "Chase",
        "age" => "30",
        "home_id" => "1"
    ],
    [
        "name" => "Gert",
        "age" => "23",
        "home_id" => "1"
    ]
]
```

This is straightforward. However, we start to get a few wrinkles once we start joining tables. In this homework's situation, our location HAS MANY people (and each person HAS ONE location). This is also called a `ONE to MANY` relationship.  If we were to join Chase's info with his location's info, our table could look like the following:

| name | age | home_id | home_street | home_city | home_state | home_zip_code |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| Chase | 30 | 1 | "1600 Broadway" | "New York" | "NY" | 10019 |

PHP would turn this all this info into just one associative array:

**Original Format**

```php
[
    [
        "name"=>"Chase",
        "age"=>"30",
        "home_id"=>"1",
        "home_street"=>"1600 Broadway",
        "home_city"=>"New York",
        "home_state"=>"NY",
        "home_zip_code"=>"10019"
    ]
]
```

But what we really want is to change the shape of the data and have `home` as a nested associative array. We'd also want to convert our numbers back into numbers from strings:

**New Format**

```php
[
    "name"=>"Chase",
    "age"=>30,
    "home"=> [
        "id"=> 1,
        "street"=>"1600 Broadway",
        "city"=>"New York",
        "state"=>"NY",
        "zip_code"=> 10019
    ]
]
```

This might seem relatively minor, but it makes more sense to store the data this way.  It's much more representative of the real world.

## Part 2 - code along

First, let's define a new function `dataTransform`. It will take a single associative array inside of an indexed array called `person_array`:

```php
function dataTransform($person_array){

}
```

Now let's create a fake `$people` array, containing a single associative array that represents one person.  This `$people` array is similar to what PHP will give us after it retrieves data from Postgres.  In reality, the `$people` array would contain multiple people (associative arrays), but for now let's just put one in.  Next, call `dataTransform` and pass that "fake" `$people` array in as a parameter:

```php
$people = [
    [
        "name"=>"Chase",
        "age"=>"30",
        "home_id"=>"1",
        "home_street"=>"1600 Broadway",
        "home_city"=>"New York",
        "home_state"=>"NY",
        "home_zip_code"=>"10019"
    ]
];

function dataTransform($person_array){

}

dataTransform($people);
```

Second, inside of `dataTransform`, let's return an empty associative array called `$person`:

```php
function dataTransform($person_array){
    $person = [];
    return $person;
}
```

Set the result of `dataTransform($people);` to a variable and then `var_dump()` it:

```php
$result = dataTransform($people);
var_dump($result);
```

We can make whatever keys we want for our new object:

```php
function dataTransform($person_array){
    $person = [
        "name" => "Chase",
        "age" => "30"
    ];
    return $person;
}
```

And we can ALMOST place our values from our `$person_array` into our new hash

```php
function dataTransform($person_array){
    $person = [
        "name" => $person_array["name"],
        "age" => $person_array["age"]
    ];
    return $person;
}
```

Check it!  This won't quite work because our `$person_array` is an associative array inside an indexed array (remember, `$person_array` will eventually contain multiple people). We can easily take the associative array out of the indexed array though:

```php
function dataTransform($person_array){
    $first_person = $person_array[0];
    $person = [
        "name" => $first_person["name"],
        "age" => $first_person["age"]
    ];
    return $person;
}
```

Now try again!  The full code:

```php
$people = [
    [
        "name"=>"Chase",
        "age"=>"30",
        "home_id"=>"1",
        "home_street"=>"1600 Broadway",
        "home_city"=>"New York",
        "home_state"=>"NY",
        "home_zip_code"=>"10019"
    ]
];

function dataTransform($person_array){
    $first_person = $person_array[0];
    $person = [
        "name" => $first_person["name"],
        "age" => $first_person["age"]
    ];
    return $person;
}

$result = dataTransform($people);
var_dump($result);
```

This should print a single associative array containing just the name and the age of the person:

```php
array(2) {
  ["name"]=>
  string(5) "Chase"
  ["age"]=>
  string(2) "30"
}
```

It *ALMOST* matches the **New Format**, but the `age` is a string. We can use a special PHP function called `intval()` to fix that:

```php
$person = [
    "name" => $first_person["name"],
    "age" => intval($first_person["age"])
];
```

Check it again!

```php
array(2) {
  ["name"]=>
  string(5) "Chase"
  ["age"]=>
  int(30)
}
```

## Part 3 - build it

Don't forget, we want our `dataTransform` function to return an object that also includes the other information about our person.

```php
[
    "name"=>"Chase",
    "age"=>30,
    "home"=> [
        "id"=> 1,
        "street"=>"1600 Broadway",
        "city"=>"New York",
        "state"=>"NY",
        "zip_code"=> 10019
    ]
]
```

How do we get that nested `home` associative array in the results?

## Part 4 - build it

As we get new clients, not all of them will have a home right away. We need to add a way to check `if` a person has a home address, and `if` that's the case, `return` the reshaped associative array `else` just `return` the `name` and `age` properties like we did in part 2.

Don't forget to test it with something like this:

```php
$people = [
    [
        "name"=>"Chase",
        "age"=>"30",
        "home_id"=>"1",
        "home_street"=>"1600 Broadway",
        "home_city"=>"New York",
        "home_state"=>"NY",
        "home_zip_code"=>"10019"
    ]
];

$result = dataTransform($people);
var_dump($result);

$people2 = [
    [
        "name" => "Gert",
        "age" => "23"
    ]
];

$result2 = dataTransform($people2);
var_dump($result2);
```

## Part 5 - build it

HELLO already has three tenants! Converting each object's format one by one is a drag.  Update `dataTransform` so that it converts an array of many `people` into the **New Format** shape:

### Original Format

```php
$people = [
    [
        "name"=>"Gert",
        "age"=>"23",
        "home_id"=> "1",
        "home_street"=>"1600 Broadway",
        "home_city"=>"New York",
        "home_state"=>"NY",
        "home_zip_code"=> "10019"
    ],
    [
        "name"=>"Alex",
        "age"=>"42",
        "home_id"=> "1",
        "home_street"=>"1600 Broadway",
        "home_city"=>"New York",
        "home_state"=>"NY",
        "home_zip_code"=> "10019"
    ],
    [
        "name"=>"Nico",
        "age"=>"61"
    ]
];
```

Check yourself: does your method still work if a person doesn't yet have a home address?

### New Format

```php
[
    [
        "name"=>"Gert",
        "age"=>23,
        "home"=> [
          "id"=> 1,
          "street"=>"1600 Broadway",
          "city"=>"New York",
          "state"=>"NY",
          "zip_code"=> 10019
        ]
    ],
    [
        "name"=>"Alex",
        "age"=>42,
        "home"=> [
          "id"=> 1,
          "street"=>"1600 Broadway",
          "city"=>"New York",
          "state"=>"NY",
          "zip_code"=> 10019
        ]
    ],
    [
        "name"=>"Nico",
        "age"=> 61
    ]
]
```

## Part 6

For this part, write a brand new `dataTransformLocations` method.  Save your previous one(s)

HELLO is growing rapidly and has acquired an amazing high profile space for their next luxury dorm experience. So next, we'd like to be able to show a list of all our addresses with their corresponding `tenants`:

### Original Format

**NOTE** assume all people with `home_id` of 1 are next to each other in the array.  Same with `home_id` of 2, 3, etc

```php
$people = [
    [
        "name"=>"Gert",
        "age"=>"23",
        "home_id"=> "1",
        "home_street"=>"1600 Broadway",
        "home_city"=>"New York",
        "home_state"=>"NY",
        "home_zip_code"=> "10019"
    ],
    [
        "name"=>"Alex",
        "age"=>"42",
        "home_id"=> "1",
        "home_street"=>"1600 Broadway",
        "home_city"=>"New York",
        "home_state"=>"NY",
        "home_zip_code"=> "10019"
    ],
    [
        "name"=>"Nico",
        "age"=>"61",
        "home_id"=> "1",
        "home_street"=>"1600 Broadway",
        "home_city"=>"New York",
        "home_state"=>"NY",
        "home_zip_code"=> "10019"
    ],
    [
        "name"=>"Molly",
        "age"=>"55",
        "home_id"=> "2",
        "home_street"=>"1 Alcatraz Island",
        "home_city"=>"San Francisco",
        "home_state"=>"CA",
        "home_zip_code"=> "94133"
    ],
    [
        "name"=>"Karolina",
        "age"=>"82",
        "home_id"=> "2",
        "home_street"=>"1 Alcatraz Island",
        "home_city"=>"San Francisco",
        "home_state"=>"CA",
        "home_zip_code"=> "94133"
    ]
];
```

### New Format

```php
[
    [
        "id"=> 1,
        "street"=>"1600 Broadway",
        "city"=>"New York",
        "state"=>"NY",
        "zip_code"=> 10019,
        "tenants" => [
            [
                "name"=>"Gert",
                "age"=>23
            ],
            [
                "name"=>"Alex",
                "age"=>42
            ],
            [
                "name"=>"Nico",
                "age"=>61
            ]
        ]
    ],
    [
        "id"=> 2,
        "street"=>"1 Alcatraz Island",
        "city"=>"San Francisco",
        "state"=>"CA",
        "zip_code"=> 94133,
        "tenants" => [
            [
                "name"=>"Molly",
                "age"=>23
            ],
            [
                "name"=>"Karolina",
                "age"=>42
            ]
        ]
    ]
]
```

## Part 7 - Hungry for more

Create the ability for `dataTransformLocations` to optionally return a specific home (with its `tenants`), based on an optional second `home_id` param:

```php
$result = dataTransform($people, 2);
```

should return

```php
[
    "id"=> 2,
    "street"=>"1 Alcatraz Island",
    "city"=>"San Francisco",
    "state"=>"CA",
    "zip_code"=> 94133,
    "tenants" => [
        [
            "name"=>"Molly",
            "age"=>23
        ],
        [
            "name"=>"Karolina",
            "age"=>42
        ]
    ]
]
```

You did it! You started building your own [ORM](https://en.wikipedia.org/wiki/Object-relational_mapping) (Object Relational Mapping)

---

*Copyright 2019, General Assembly Space. Licensed under [CC-BY-NC-SA, 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)*
