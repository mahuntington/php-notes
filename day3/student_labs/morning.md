# Lab

Implement a show route for our people model (`/people/:id`) which will return to the client JSON representing the requested row in the `people` table along with a nested `home` attribute which will be the selected person's related location:

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

Do the same for locations (`/locations/:id`), but include a nested `inhabitants` attribute, which will be an array, containing objects representing the rows in the `people` table related to the selected location:

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
