# Create an API

## Route URLs to php files

Routing can be accomplished with a `.htaccess` file placed in your root directory for the app.  This is not PHP related.  It's actually something that is a part of Apache.  We'll use this file to specify how our routes map to particular files.

First tell Apache to allow the ability to rewrite URLs so that they map to specific `.php` files:

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
