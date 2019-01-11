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

This will be a one-to-many relationship

## Add home_id to people

First, go into `psql` and add a `home_id` INT column to the `people` table:

```sql
ALTER TABLE people ADD COLUMN home_id INT;
```

Now we can give people homes like so:

```sql
UPDATE people SET home_id = 2 WHERE id = 4;
```

The previous code will find the row with an `id` of 4 and set the `home_id` column to 2;
