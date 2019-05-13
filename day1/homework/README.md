[![General Assembly Logo](/ga_cog.png)](https://generalassemb.ly)

# HELLO - Start Building Your Own ORM

Use ruby to reshape the kinds of objects coming from a SQL database (via a gem called PG)

#### Learning Objectives

- Learn about the challenges with converting SQL (rows and tables into JSON)
- Practice using ruby to reshape objects
- etc.

#### Prerequisites

- Experience with RESTful routes
- Basic Ruby
- Familiarity with JSON
- Familiarity with SQL

---

## Getting Started

1. You'll be working in one ruby file, and building out your answers as you work through this hw.


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

SQL stores data in rows and columns, so in order to get the data to display on a web page, we need to convert the data from rows and columns into hashes with keys and values.  We can use the ruby gem PG to query Postgres and give us back an array of hashes(objects) that represents the rows and columns of the DB.

If we just have one table with the following keys and values:

| name | age |
|:-:|:-:|
| Chase| 30|

PG will turn this into the following:

```ruby
[
    {
        "name" => "Chase",
        "age" => "30"
    }
]
```

If we had more than one person...

| name | age |
|:-:|:-:|
| Chase | 30 |
| Gert | 23 |

PG will give us:

```ruby
[
    {
        "name" => "Chase",
        "age" => "30"
    },
    {
        "name" => "Gert",
        "age" => "23"
    }
]
```

This is straightforward. However, we start to get a few wrinkles once we start joining tables. In this homework's situation, our location HAS MANY people (and each person HAS ONE location). This is also called a `ONE to MANY` relationship.  If we were to join Chase's info with his location's info, our table would look like the following:

| name | age | home_id | home_street | home_city | home_state | home_zip_code |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| Chase | 30 | 1 | "1600 Broadway" | "New York" | "NY" | 10019 |

PG would give us just one blob:

**Original Format**

```ruby
[
    {
        "name"=>"Chase",
        "age"=>"30",
        "home_id"=>"1",
        "home_street"=>"1600 Broadway",
        "home_city"=>"New York",
        "home_state"=>"NY",
        "home_zip_code"=>"10019"
    }
]
```

But what we really want is to change the shape of the data and have `home` as a nested object. We'd also want to convert our numbers back into numbers from strings:

**New Format**

```ruby
{
    "name"=>"Chase",
    "age"=>30,
    "home"=> {
      "id"=> 1,
      "street"=>"1600 Broadway",
      "city"=>"New York",
      "state"=>"NY",
      "zip_code"=> 10019
    }
}
```

## Part 2 - code along

First, let's define a new method `data_transform`. It will take a single hash inside of an array called `person_array`:

```rb
def data_transform(person_array)
end
```

Now let's create a fake `people` array, containing a single hash.  This will represent what the PG gem will give us from Postgres when we use it in rails.  Next, call `data_transform` and pass that fake `people` array in as a parameter:

```rb
people = [
    {
        "name"=>"Chase",
        "age"=>"30",
        "home_id"=>"1",
        "home_street"=>"1600 Broadway",
        "home_city"=>"New York",
        "home_state"=>"NY",
        "home_zip_code"=>"10019"
    }
]

def data_transform(person_array)
end

p data_transform people
```

Second, inside of `data_transform`, let's make a new object called `person`

```rb
def data_transform(person_array)
    person = {}
end
```

We can make whatever keys we want for our new object:

```rb
def data_transform(person_array)
    person = {
      "name" => "Chase",
      "age" => "30"
    }
end
```

And we can ALMOST place our values from our `person_array` into our new hash

```rb
person = {
  "name" => person_array["age"],
  "age" => person_array["age"]
}
```

Check it!  This won't quite work because our `person_array` is an array. We can easily take the single hash out of the array using a ruby method `.first`

```rb
def data_transform(person_array)
    person = person_array.first
    return {
      "name" => person["name"],
      "age" => person["age"]
    }
end
```

Now try again!  The full code:

```rb
people = [
    {
        "name"=>"Chase",
        "age"=>"30",
        "home_id"=>"1",
        "home_street"=>"1600 Broadway",
        "home_city"=>"New York",
        "home_state"=>"NY",
        "home_zip_code"=>"10019"
    }
]

def data_transform(person_array)
    person = person_array.first
    return {
      "name" => person["name"],
      "age" => person["age"]
    }
end

p data_transform people
```

should print a hash containing just the name and the age of the person:

```rb
{"name"=>"Chase", "age"=>"30"}
```

It *ALMOST* matches the **New Format**, but the `age` is a string. We can use a method to fix that in a snap!

```rb
return {
    "name" => person["name"],
    "age" => person["age"].to_i
}
```

Check it again!

```rb
{"name"=>"Chase", "age"=>30}
```

## Part 3 - build it

Don't forget, we want our `data_transform` function to return an object that also includes the other information about our person.

```rb
{
    "name"=>"Chase",
    "age"=>30,
    "home"=> {
      "id"=> 1,
      "street"=>"1600 Broadway",
      "city"=>"New York",
      "state"=>"NY",
      "zip_code"=> 10019
    }
}
```

How do we get that nested `home` hash in the results?

## Part 4 - build it

As we get new clients, not all of them will have a home right away. We need to add a way to check `if` a person has a home address, and if true, return the reshaped hash `else` just return the `person` hash

Don't forget to test it, there is sample 'data' available in Part 1 for you to use

## Part 5 - build it

HELLO already has three tenants! Converting each object's format one by one is a drag.  Write a new function `data_transforms` that converts an array of many `people` hashes into the **New Format** shape:

### Original Format

```rb
people = [
    {
        "name"=>"Gert",
        "age"=>"23",
        "home_id"=> "1",
        "home_street"=>"1600 Broadway",
        "home_city"=>"New York",
        "home_state"=>"NY",
        "home_zip_code"=> "10019"
    },
    {
        "name"=>"Alex",
        "age"=>"42",
        "home_id"=> "1",
        "home_street"=>"1600 Broadway",
        "home_city"=>"New York",
        "home_state"=>"NY",
        "home_zip_code"=> "10019"
    },
    {
        "name"=>"Nico",
        "age"=>"61",
        "home_id"=> "1",
        "home_street"=>"1600 Broadway",
        "home_city"=>"New York",
        "home_state"=>"NY",
        "home_zip_code"=> "10019"
    }
]
```

Check yourself: does your method still work if a person doesn't yet have a home address?

### New Format

```rb
people = [
    {
        "name"=>"Gert",
        "age"=>23,
        "home"=> {
          "id"=> 1,
          "street"=>"1600 Broadway",
          "city"=>"New York",
          "state"=>"NY",
          "zip_code"=> 10019
        }
    },
    {
        "name"=>"Alex",
        "age"=>42,
        "home"=> {
          "id"=> 1,
          "street"=>"1600 Broadway",
          "city"=>"New York",
          "state"=>"NY",
          "zip_code"=> 10019
        }
    },
    {
        "name"=>"Nico",
        "age"=> 61,
        "home"=> {
          "id"=> 1,
          "street"=>"1600 Broadway",
          "city"=>"New York",
          "state"=>"NY",
          "zip_code"=> 10019
        }
    }
]
```

## Part 6

For this part, write a brand new `data_transforms_locations` method.  Save your previous one(s)

HELLO is growing rapidly and has acquired an amazing high profile space for their next luxury dorm experience. So next, we'd like to be able to show a list of all our addresses with their corresponding `tenants`:

### Original Format

**NOTE** assume all people with `home_id` of 1 are next to each other in the array.  Same with `home_id` of 2, 3, etc

```rb
people = [
    {
        "name"=>"Gert",
        "age"=>"23",
        "home_id"=> "1",
        "home_street"=>"1600 Broadway",
        "home_city"=>"New York",
        "home_state"=>"NY",
        "home_zip_code"=> "10019"
    },
    {
        "name"=>"Alex",
        "age"=>"42",
        "home_id"=> "1",
        "home_street"=>"1600 Broadway",
        "home_city"=>"New York",
        "home_state"=>"NY",
        "home_zip_code"=> "10019"
    },
    {
        "name"=>"Nico",
        "age"=>"61",
        "home_id"=> "1",
        "home_street"=>"1600 Broadway",
        "home_city"=>"New York",
        "home_state"=>"NY",
        "home_zip_code"=> "10019"
    },
    {
        "name"=>"Molly",
        "age"=>"55",
        "home_id"=> "2",
        "home_street"=>"1 Alcatraz Island",
        "home_city"=>"San Francisco",
        "home_state"=>"CA",
        "home_zip_code"=> "94133"
    },
    {
        "name"=>"Karolina",
        "age"=>"82",
        "home_id"=> "2",
        "home_street"=>"1 Alcatraz Island",
        "home_city"=>"San Francisco",
        "home_state"=>"CA",
        "home_zip_code"=> "94133"
    }
]
```

### New Format

```rb
[
    {
      "id"=> 1,
      "street"=>"1600 Broadway",
      "city"=>"New York",
      "state"=>"NY",
      "zip_code"=> 10019,
      "tenants" => [
            {
                "name"=>"Gert",
                "age"=>23
            },
            {
                "name"=>"Alex",
                "age"=>42
            },
            {
                "name"=>"Nico",
                "age"=>61
            }
        ]
    },
    {
      "id"=> 2,
      "street"=>"1 Alcatraz Island",
      "city"=>"San Francisco",
      "state"=>"CA",
      "zip_code"=> 94133,
      "tenants" => [
            {
                "name"=>"Molly",
                "age"=>23
            },
            {
                "name"=>"Karolina",
                "age"=>42
            }
        ]
    }
]
```

## Part 7 - Hungry for more

Create the ability for `data_transform_location` to optionally return a specific home (with its `tenants`), based on an optional second `home_id` param:

```rb
p data_transform_location people, 2
```

should return

```rb
{
    "id"=> 2,
    "street"=>"1 Alcatraz Island",
    "city"=>"San Francisco",
    "state"=>"CA",
    "zip_code"=> 94133,
    "tenants" => [
        {
            "name"=>"Molly",
            "age"=>23
        },
        {
            "name"=>"Karolina",
            "age"=>42
        }
    ]
}
```

You did it! You started building your own [ORM](https://en.wikipedia.org/wiki/Object-relational_mapping) (Object Relational Mapping)

---

*Copyright 2019, General Assembly Space. Licensed under [CC-BY-NC-SA, 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)*
