# Intermediate PHP

## Convenience Methods

PHP has lots of global functions that will help you out:

### Strings

Count a string's length

```php
<?php
echo strlen("Hello world!"); // outputs 12
?>
```

Count number of words in a string

```php
<?php
echo str_word_count("Hello world!"); // outputs 2
?>
```

Reverse a string

```php
<?php
echo strrev("Hello world!"); // outputs !dlrow olleH
?>
```

Find a sub string in a string

```php
<?php
echo strpos("Hello world world!", "world"); // outputs 6
//search from the right
echo strrpos("Hello world world!", "world"); // outputs 12
?>
```

Replace text within a string

```php
<?php
echo str_replace("world", "Dolly", "Hello world!"); // outputs Hello Dolly!
?>
```

Get a substring based on character index

```php
<?php
//This gets all remaining characters starting at the index specified in the string (0 based, like an array)
echo substr("Hello world", 3); // outputs 'lo world'

//third param is length of string to retrieve
echo substr("Hello world", 3, 4); // outputs 'lo w'
?>
```

Turn a string into an array based on some delimiter:

```php
<?php
    $the_string = "Some sentence goes here";
    $string_array = explode(" ", $the_string);
    var_dump($string_array);
?>
```

### Sorting Arrays

Various functions for sorting arrays.  Can be done arithmetically or alphabetically, depending on content

- sort() - sort arrays in ascending order
- rsort() - sort arrays in descending order
- asort() - sort associative arrays in ascending order, according to the value
- ksort() - sort associative arrays in ascending order, according to the key
- arsort() - sort associative arrays in descending order, according to the value
- krsort() - sort associative arrays in descending order, according to the key

These functions affect the actual array they are called on.

```php
$cars = array("Volvo", "BMW", "Toyota");
sort($cars);
print_r($cars); //BMW, Toyota, Volvo
rsort($cars);
print_r($cars); //Volco, Toyota, BMW
```

Now with associative arrays:

```php
$ages = array("Peter" => 35, "Ben" => 37, "Joe" => "43");
asort($ages);
print_r($ages);
ksort($ages);
print_r($ages);
arsort($ages);
print_r($ages);
krsort($ages);
print_r($ages);
```

## Classes and Objects

### Public Members

```php
<?php
class Car {
	public $wheels = 4; // public var can be accessed outside class definition
}

$my_car = new Car();

print_r($my_car);
$my_car->wheels = 3;
print_r($my_car);
?>
```

### Constructors

```php
<?php
class Car {
	public $wheels;
	public function __construct($num_wheels){ // runs at beginning of object creation
		$this->wheels = $num_wheels;
	}
}
$my_car = new Car(4);
print_r($my_car);
?>
```

### Protected Members

But what if you don't want someone to modify the number of wheels of a car after it is created?
You can set a method or member to `private` and any instance will not be able to read or write that value.  It is only accessible from within the class itself

```php
<?php
class Car {
	protected $wheels; // cannot be accessed outside class definition

	//you must define public getters in order to see how many wheels a car has
	public function getWheels(){
		return $this->wheels;
	}

	public function __construct($num_wheels){
		$this->wheels = $num_wheels;
	}
}

$my_car = new Car(4);
echo $my_car->getWheels();
echo $my_car->wheels; //error: cannot access protected property
$my_car->wheels = 4; //error
?>
```

### Statics

You can have `static` members and methods that accessible only on the class itself, not on instances of the class.

```php
<?php
class Car {
	protected $wheels;
	public function getWheels(){
		return $this->wheels;
	}
	public function __construct($num_wheels){
		$this->wheels = $num_wheels;
	}
}

class CarFactory{
	static protected $cars = array(); // the array is protected
	static public function create($num_wheels){
		$new_car = new Car($num_wheels);
		self::$cars[] = $new_car;
		return $new_car;
	}
	static public function find($index){
		return self::$cars[$index];
	}
}

$my_car = CarFactory::create(2);
$my_car2 = CarFactory::create(4);
print_r($my_car);
print_r(CarFactory::find(1));
print_r(CarFactory::$cars); //errors out: protected
?>
```

### Inheritance

Protected members/methods can be used by inheriting classes, private members/methods cannot

```php
<?php
class Car {
	protected $wheels; //accessible by child class, not outside class, though
	private $engine_on = false; //unaccessible by child class
	public function __construct($num_wheels){
		$this->wheels = $num_wheels;
	}
	public function getWheels(){
		return $this->wheels;
	}
	public function start(){
		$this->engine_on = true;
	}
}

class Humvee extends Car {
	protected $armour = 10;
	public function takeDamage($damage){
		$this->armour -= $damage;
	}
	public function loseWheel(){
		$this->wheels--; //can access parent class $wheels member because it is protected
	}
	//can extend parent's start function
	public function start(){
		parent::start(); //run parent's start function
		//parent::$engine_on = false; //errors out: cannot access private member of parent
		echo "WELCOME TO THE JUNGLE...";//then do something else
	}
}

$my_humvee = new Humvee(6);
print_r($my_humvee);
$my_humvee->loseWheel();
print_r($my_humvee);
$my_humvee->takeDamage(6);
print_r($my_humvee);
$my_humvee->start();
print_r($my_humvee);
?>
```
