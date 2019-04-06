# Lab - Landscaper

## Description

Remember the landscaper app you built in unit 1?  Well, now recreate it using PHP.  You'll do it in the terminal though, because it's actually really easy to run php in the terminal.  First off, the command to run a php file in the terminal is just:

```
php yourfile.php
```

Keep in mind you'll have to change `yourfile.php` to the name of the file you write your code in.

Next, the following code is all you need in order to get input from the user in the terminal:

```php
$response = trim(fgets( STDIN ));
```

Now `$response` will hold whatever the user types into the terminal (they'll have to hit enter when they're done typing).  If you want, research what each function does, but don't worry if you just want to copy paste.

Here's the beginnings of an app that starts the user off with an initial amount of apples/money and then asks them what they want to do.  It only runs once though.  If you feel like it, as a starting point, try to make it work so that this sample app will continue until the user runs out of money.

```php
<?php
$apples = 0;
$money = 20;
echo "What would you like to do?  Eat an apple or buy an apple?\n";
echo "eat/buy\n";
$response = trim(fgets( STDIN ));
if($response === "eat"){
    $apples--;
    echo "you have " . $apples . " apples and $" . $money ."\n";
} else {
    $apples++;
    $money--;
    echo "you have " . $apples . " apples and $" . $money ."\n";
}
?>
```

Note the `\n` that's used to add a new line in the terminal

In case you forgot how the landscaper should run, here's a [modified version of the original markdown](landscaper.md)
