# PHP

## Setup

1. Download [MAMP] (https://www.mamp.info/en/downloads/)
1. Double click .pkg file and follow prompts
1. Double click /Applications/MAMP/MAMP
1. Tell MAMP to where your files are
	- Click on Preferences
	- Click on Web Server
	- Click the folder icon next to "Document Root" and find a suitable directory to work out of
	- Click OK
1. In your Document Root, create `index.php`.
1. Go to <http://localhost:8888/>
	- If no file is specified in the URL after the port, MAMP will look for `index.php`
1. Error logs are in /Applications/MAMP/logs/
	- use `tail -f php_error.log` to watch the end of the log file in case something breaks
1. MAMP stands for Mac, Apache, MySQL, PHP
	- Mac
		- Your OS
	- Apache
		- A pre-build web server that serves static files
		- It is extendable with various modules that allows it to do many things easily
	- MySQL
		- Your Database
	- PHP
		- A module for Apache that allows it to serve dynamic data

## Basics

### Tags

Because this is all run on top of Apache, the initial assumption is that we're serving static HTML files

- We need `<?php ?>` tags to show that we're writing PHP
- Think of this as if Apache/PHP is server.js and we're writing EJS

Instead of `<%= %>` you have `<?= ?>` or `<?php echo ?>`

Instead of `<% %>` you have `<?php ?>`

### Comments

```php
// single line comment
```
```php
/*
multi
line
comment
*/
```

### Declaring/Assigning variables

Use a $ before a variable name to tell php it is a variable.  Assignment is standard.

```php
<?php
	$my_first_var; //declare
	$my_first_var = 2; //assignment
	$my_second_var = 3; //declare and assign
	echo $my_first_var; //print this variable to the page
	echo $my_second_var; //print this variable to the page
?>
```

### Data Types

PHP has the following basic data types:

Strings:

```php
<?php
$x = "my string";
var_dump($x);
?>
```

Integers:

```php
<?php
$x = 5985;
var_dump($x);
?>
```

Floats:

```php
<?php
$x = 10.365;
var_dump($x);
?>
```

Booleans:

```php
<?php
$x = true;
var_dump($x);
?>
```

Arrays:

```php
<?php
$cars = array("Volvo","BMW","Toyota");
var_dump($cars);
?>
```

NULL:

```php
<?php
$x = null;
var_dump($x);
?>
```

### String Operators

Use a `.` or `.=` to combine strings.  Works just like `+` and `+=`

```php
<?php
	$first_part = "first part";
	$second_part = "second part";
	$concatenation = $first_part . " " . $second_part; //combine strings
	$concatenation .= ".  Appended value"; //append strings
	echo $concatenation;
?>
```

Some kinds of string interpolation work:

```php
<?php
	$x = 5;
	echo "I have $x pizzas";
?>
```

### Arithmetic Operators

```php
<?php
	echo 1 + 1; //2
	echo 2 - 1; //1
	echo 3 * 2; //6
	echo 12 / 3; //4
	echo 5 % 2; //1 modulus
	echo 2 ** 3 //8 exponents
?>
```

### Increment/Decrement Operators

```php
<?php
	$x = 2;
	$x++; //increment by 1;
	echo $x;
	$x--; //decrement by 1;
	echo $x;
?>
```

### Assignment Operators

```php
<?php
	$my_var = 1;
	echo $my_var;
	$my_var += 3; //$my_var = $my_var + 3;
	echo $my_var;
	$my_var -= 2; //$my_var = $my_var - 2;
	echo $my_var;
	$my_var *= 2; //$my_var = $my_var * 2;
	echo $my_var;
	$my_var /= 2; //$my_var = $my_var / 2;
	echo $my_var;
?>
```

## Conditionals

### Formats

The traditional format works great:

```php
<?php
	$x = 1;
	if($x > 2){
		echo "x > 2";
	}
	elseif($x < 2){
		echo "x < 2";
	}
	else{
		echo "x == 2";
	}
?>
```

If you have html and don't want to have lines that look like `<?php } ?>`, you can use the following style of if/else:

```php
<?php $x = 1; ?>
<?php if($x > 2): ?>
	<code>x &gt; 2</code>
<?php elseif($x < 2): ?>
	<code>x &lt; 2</code>
<?php else: ?>
	<code>x == 2</code>
<?php endif; ?>
```

### Comparison Operators

Equality:

```php
<?php
	$x == $y; //equal
	$x === $y; //identical
	$x != $y; //not equal
	$x !== $y; //not identical
?>
```

Arithmetic:

```php
<?php
	$x < $y; //less than
	$x > $y; //greater than
	$x <= $y; //less than or equal to
	$x >= $y; //greater than or equal to
?>
```

### Logical Operators

```php
<?php
	true && false //AND operator
	true || false //OR operator
?>
```

## Arrays

### Indexed Arrays

Standard array functionality

```php
<?php
$cars = array("Volvo", "BMW", "Toyota");
$cars[4] = 'asdf'; //can be in indexes that don't yet exist
$cars[] = "added to end"; //pushes onto array
echo "I like " . $cars[0] . ", " . $cars[4] . " and " . $cars[5] . "."; //access arrays normally
echo count($cars); //prints length of array
print_r($cars); //prints contents of array in nicer format than var_dump
?>
```

### Associative Arrays (hashes)

These are very similar to JavaScript objects, but are accessed like arrays:

```php
<?php
	$age = array("Peter" => 35, "Ben" => 37, "Joe" => "43"); //declare
	$age["Bob"] = 105; //add at a new position
	echo "Bob is " . $age['Bob'] . " years old.";
?>
```

## Loops

### While

```php
<?php
$x = 1;

while($x <= 5) {
    echo "The number is: $x <br/>";
    $x++;
}
?>
```

Alternative syntax:

```php
<ul>
	<?php $x = 1;?>
	<?php while($x <= 5): ?>
		<li><?= $x ?></li>
		<?php $x++ ?>
	<?php endwhile; ?>
</ul>
```

### For

```php
<?php
for ($x = 0; $x <= 10; $x++) {
    echo "The number is: $x <br/>";
}
?>
```

Alternative syntax:

```php
<ul>
	<?php for ($x = 0; $x <= 10; $x++): ?>
	    <li>The number is: <?= $x ?></li>
	<?php endfor; ?>
</ul>
```

### Foreach

This is like `for of` in JS:

```php
<?php
$colors = array("red", "green", "blue", "yellow");

foreach ($colors as $key => $value) {
    echo $key . ": $value <br/>";
}
?>
```

Alternative syntax:

```php
<?php $colors = array("red", "green", "blue", "yellow"); ?>
<?php foreach ($colors as $key => $value): ?>
	<?= $key ?>: <?=$value?><br/>
<?php endforeach ?>
```

This works for associative arrays:

```php
<?php
$ages = array("Peter" => 35, "Ben" => 37, "Joe" => "43");

foreach ($ages as $key => $value) {
    echo $key . ": $value <br/>";
}
?>
```

Alternative syntax:

```php
<?php $ages = array("Peter" => 35, "Ben" => 37, "Joe" => "43"); ?>
<?php foreach ($ages as $key => $value): ?>
	<?= $key ?>: <?=$value?> <br/>
<?php endforeach ?>
```

## Functions

```php
<?php
function greet($name) {
    echo "Hello $name";
}

greet("Matt"); // call the function
?>
```
