# Homework

Finish [today's labs](../student_labs/)

As a stretch, once the labs are complete, implement a show route (`find()`) for People (`/people/:id`),  Locations (`/locations/:id`),  Companies (`/companies/:id`).  Each route should return JSON to the client for only the specified person, location, or company in the DB.  For example, a call to `/locations/1` would return something like:

```JSON
{
    "id":1,
    "street":"123 Fake Street",
    "city":"Awesometown",
    "state":"CA"
}
```

This is different from the index route we wrote in lecture which returns all specified models in the DB in an array.  For example:

```JSON
[
    {
        "id":1,
        "street":"123 Fake Street",
        "city":"Awesometown",
        "state":"CA"
    },
    {
        "id":2,
        "street":"456 Super Circle",
        "city":"Funton",
        "state":"NY"
    },
    {
        "id":3,
        "street":"789 Radical Road",
        "city":"Coolburg",
        "state":"PA"
    }
]
```
