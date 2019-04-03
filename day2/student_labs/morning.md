# Morning Lab

Update your code from the lecture this morning so that it has READ (index) functionality for a `Locations` model.  The model should have the following attributes (columns):

1. id (SERIAL)
1. street (VARCHAR(32))
1. city (VARCHAR(32))
1. state (VARCHAR(2))

It's suggested that you have a php class `Location` that will represent each row in a table called `locations`.  This is similar to the `Person` class from the lecture.  You can create a "factory" class `Locations` that will be in charge of generating `Location` objects from the DB.  This is similar to the `People` class from the lecture.

If you finish this early, update your code so that it has READ (index) functionality for a `Companies` model.  The model should have the following attributes (columns):

1. id (SERIAL)
1. name (VARCHAR(32))
1. website (VARCHAR(64))
1. industry (VARCHAR(16))

It's suggested that you have a php class `Company` that will represent each row in a table called `companies`.  This is similar to the `Person` class from the lecture.  You can create a "factory" class `Companies` that will be in charge of generating `Company` objects from the DB.  This is similar to the `People` class from the lecture.
