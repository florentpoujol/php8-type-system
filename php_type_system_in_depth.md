# Type system in PHP 8.1

## Table of content

- [Typing systems](#typing-systems)
- [type declaration syntax](#type-declaration-syntax)
- [Why use types ?](#why-use-types-why-make-your-code-stricter-)
- [Built-in type declarations](#built-in-type-declarations)
	- [Scalar type declarations](#scalar-type-declarations)
	- [Compound type declarations](#compound-type-declarations)
	- [Return-only type declarations](#return-only-type-declarations)
	- [Others](#others)
- [Objects](#objects)
	- [type declarations](#type-declarations)
	- [Types in inheritance](#types-in-inheritance)
	- [Classes as argument types](#classes-as-argument-types)
	- [Classes as method return types](#classes-as-method-return-types)
	- [Enums](#enums)
- [Numeric strings](#numeric-strings)
- [Union and intersection types](#union-and-intersection-types)
- [Getting type information from a variable](#getting-type-information-from-a-variable)
- [Casting](#casting)
- [Comparing and juggling types](#comparing-and-juggling-types)
- [Strict types](#strict-types)
- [Dealing with nulls](#dealing-with-nulls)


## Typing systems

A **type** refers to the **kind** of value that a variable (or property, or argument) holds.

Each type comes with a set of behaviours and properties inherent to it. For instance, you can only use numerical operations on numeric types.  

> Well actually, the `+` is also the string concatenation operator in JavaScript, and in PHP you can also add two arrays together (which merge them without overriding their keys)...

Typically, the lower level a language is, the most number of types it has.  

JavaScript as a single "number" type that is actually an 8 bytes float.  
PHP distinguish between integers and floats but on a 64 bits system, both always takes 8 bytes in memory.  
And C has many integer and float types, each taking a different amount of space in memory.

PHP is a **weakly** and **dynamically** typed language, as opposed to strongly and statically typed ones, like Java or C. 

This means a few things in practice, regarding PHP :

Type-checking only happens at runtime, there is no mandatory **compile step** that must succeed before your program can even run. And because of that, type checking is actually **optional**, PHP does not forces you to type anything. 

A particular variable is not tied to any type once created and can be reassigned to any value of any type at anytime.  

**Values instead of variable** have types, and the type that a variable holds can always be **inferred** by looking at the value's type (that is always known).  
Also, the type of the value can often be automatically **cast** (changed from one type to another), when compatible, like a numeric string and an integer.  


## Type declaration syntax

Type declaration in PHP is done by using a set of keywords (often also called **typehint**) to tell PHP, static analysis tools and also humans of which type the value (of an argument for instance) should be.  

When the type assigned to the argument is not the same as the one of the type declaration, PHP will try to convert it, or throw a type error if it can not, *or is not allowed to*.

Example:
```php
// here we declared that the $bar argument is expected to be an integer. 
// "int" is the type declaration for the integer type
function foo(int $bar) 
{
}

foo(1); // this works as expected because the value as the same type as the one expected

foo('1') // this may work under some conditions, because PHP knows how to transform such numeric string 
         // (a string that looks like a number) into an integer

foo(new stdClass()); // will always throw a type error, because PHP doesn't know 
                     // how to transform a stdClass instance to an int
```

PHP has a fixed number of built-in types, of which most of them have matching type declaration.  
There is also some more compound type declarations, that match several types at the same time.  
Finally, you can not create custom type declarations, but every interface and class names can also be used as type declaration.

In PHP, type declarations can only be defined in four places:
- on a property of a class, trait, or interface (only since 7.4)
- on arguments of a function
- on the returned value of a function
- on a class constant (since 8.3)

Here, "function" refers to regular named functions, both long and short closures (anonymous functions) as well as methods of a class.

So it is not possible to specify the type of a simple variable. Theses are always inferred from the assigned value.  

Type declarations for properties and arguments are defined with a keyword that must precede the property or argument variable definition.

Function return types follow a semicolon, and are located between the closing parenthesis of the signature and the opening brace.

As show in the example below, you are not limited to a single type per property/argument/return type, but you can define them to be `nullable` and/or even to accept several types in a `union` or an `intersection`.

Here is what a class that use most of the type features can look like :

```php
<?php

declare(strict_types=1);

final class Foo
{
	private int $someInt;

	public function getSomeInt(float $floatOrNull = null): int|float
	{
		if (is_float($floatOrNull))
		{
			return $floatOrNull;
		}

		return $this->someInt;
	}

	public function doSomething(int|string $foo, ?SomeInterface $object): void
	{
		$object?->expectSomeString((string) $foo);
	}
}
```


## Why use types, why make your code stricter ?

PHP has existed for years before any of the type features were first introduced, and many languages ,including JavaScript, today still have no comparable features.

We don't really **need** type declarations to build big and working programs, so why introducing them now in the language and our code ?

For the same reasons we write tests and have a staging environment.  
For the same reasons [TypeScript](https://www.typescriptlang.org/) and [Elm](https://elm-lang.org/) exists.  
For the same reasons some of Laravel's magic is disapproved of in some circles.

The flexibility of not typing has limits that are quickly reached when you build anything other than the most simple apps.

Apparently good art comes from constraints and so is good, resilient code design.  
Forcing yourself to type (and not overuse unions types or the `mixed` type declaration that are a little a cheat in that regard) will eventually help you build less flexible, but more organized, clearer code.

Also, typing **and the static analysis it allows** will catch situations that shouldn't exist where type mismatch occurs, *even in the parts that your tests do not cover*.

Typing helps tremendously make your program be more understandable, more reliable and more resilient.

In addition, when a type error fails to be caught by static analysis and occurs at runtime, the error is usually simpler to understand than what you may have if the type mismatch occurs later down the road.

Where tests can take a lot of time to set up in addition to the time you take to build the feature (it's still 100% worth it), typing your code is about adding literally a few characters now and then. This is something that you do naturally while you are coding, this is not a separate tasks.  

Especially if done since the start of a project, any time taken adding types, or perceived increase of complexity due to the additional keywords in the code, is rewarded over multiple times in the time savings over debugging or bug fixing.


## Built-in type declarations

[Type declarations on the PHP manual.](https://www.php.net/manual/en/language.types.declarations.php#language.types.declarations).

### Scalar type declarations

**`bool`**, **`float`**, **`int`** and **`string`** are the four **scalar** types and type declarations, added in PHP7.0.  
Unlike in other lower-level languages you can not choose the size they take in memory: integer and floating-point numbers are always 8 bytes on 64 bits (so, current) systems. So despite the simple name "float", they are most often actually **double-precision**.

Objects that implements the [`Stringable` interface](https://www.php.net/manual/en/class.stringable.php) (that have the `__toString(): string` magic method) do not match the `string` type declaration, but can automatically be cast to string, so it is safe to pass a stringable object to a property or argument of type string when you are not using [strict types](#strict-types).  

Example: 
```php
final class Foo implements \Stringable
{
	public function __toString(): string
	{
		return 42;
	}
}

function bar(string $foo, float $bar): int
{
	return (int) ($foo + $bar);
}

$foo = new Foo();

var_dump(bar($foo, 8.0)); // int(50)
```

This example also feature implicit and explicit [casting](#casting).


### Compound type declarations

**`iterable`** is a type declaration that will match an array or an  object that implement the [`\Traversable`](https://www.php.net/manual/en/class.traversable) interface, it is equivalent to the `array|\Traversable` union.  
With this type declaration, you know that the value is something that you can iterate over with `foreach()` for instance.

The **`callable`** type declaration ensure that the value is a valid [callable](https://www.php.net/manual/en/language.types.callable.php) **in the context of the variable**.  

The end of the last sentence is important because the way callable works, the same value can be a callable in one place but not in others.  (TODO add example, see RFC on typed properties)
A consequence is that the `callable` type declaration **can not be used on class properties**.

The closest union you can use instead is `array|object|string`, but you may question in the context of your project, if using `callable` is actually pertinent or if in practice all callables you ever use are anonymous functions for instance, in which case the `\Closure` type declaration could be preferred and can be used with properties.


### Return only type declarations

Three type declarations can only be used [as the return type declaration](https://www.php.net/manual/en/language.types.declarations.php#language.types.declarations.return-only) of a function, and can't be used with properties and arguments.

**`void`** is used to indicates that the function never return anything, even `null`.  

Note that even with the type declaration, functions in PHP technically always return `null` so the following code is actually valid PHP even if static analysis would pick-up the oddity that you are passing the return of a `void` function to a variable:
```php
function foo(): void
{
	return;	// only `return;` is allowed (or no return at all), even `return null;` isn’t allowed
}

$bar = foo(); // this is valid PHP but picked-up by static analyzers

var_dump($bar); // null
```

**`never`**: added in PHP8.1 it is useful for the rare cases where a function actually never returns anything because it makes the program exit, typically by throwing an exception, or calling `exit()` or `die()`.

**`static`**: see in the [objects chapter](#objects).


### Others

**`array`** : this is the first type declaration beside class names, added all the way back in PHP5.1. The value must be an array.  
It is not possible to type the *content* of the array, the types of the keys and values, but that can be declared via PHPDocs, or the `ArrayShape` annotation within PHPStorm.  
Also, complex arrays may be good candidates to be refactored to value objects or DTO with typed properties.

**`mixed`**: added in PHP8.0, this is a compound type equivalent to `array|bool|int|float|null|object|resource|string`, which match **everything including `null`**.  
The point of it is for the reader not to have to wonder if the type declaration was just omitted, or for the cases where truly all the types can be passed.

**`null`** and **`?`**: PHP7.1 added [**nullable type declarations**](https://www.php.net/manual/en/language.types.declarations.php#language.types.declarations.nullable), which allow values to be of the specified type(s), or `null`.  
It has two form, either `null` when used as part of a union type, of the short form `?` that precede a single type declaration.  

There is also an **implicit** third form for arguments, when their default value is `null`. Then you can pass `null` to the argument even if the type declaration is not expressly nullable.  
This form is deprecated as of PHP8.4 and will throw an error in the next version.

Example, where the property and all arguments are nullables:
```php
final class foo
{
	private null|int $property; // same as ?int

	function foo(null|int|string $null, ?array $questionMark, object $optional = null): ?string
	{

	}
}
```

Since PHP8.2, `null` can also be on its own as a return type.

**`false`** and **`true`**: (`false` existed before but `true` has been added in PHP8.2) both can be used on their own as a method return type.

**`object`**, **`self`**, **`parent`**, any class/interface name: See more info in the [objects chapter](#objects).

**`Closure`** : this is just a reminder that anonymous functions are under the hood instances of the `\Closure` built-in class.  
So it is a small subset of the `callable` type declaration, and that do not match classes using the `__invoke()` magic method.

**`resource`**: despite being a type of value a variable can have, **it is not a valid type declaration**, and never will be since all resource are progressively replaced by actual objects. It is however usable as PHPDoc.


## Objects

Object-Oriented Programming was the flagship feature of PHP5.0.  
They have since been improved upon with traits, better covariance and contravariance, typed properties and more recently, enums.

### type declarations

First, creating a class or an interface is currently the only way to "create" a new type, and type declaration.  
All interfaces/classes name can be used automatically both in PHPDocs and as type declarations.  
**But trait names can't.**

There is also a few built-in type declarations : 

**`object`** : The value must be an instance of any class, which include enums and anonymous functions.  

The three type declarations below can only be used inside a class/interface/trait:
- **`self`** means that the value must be an instance of the current class the type declaration is located in, or any of its children
- **`static`** can only be used as a method return type and means that the value is the actual class the method is used on, and unlike `self` takes late static binding into account
- **`parent`** the value must be an instance of any parent of the class

Examples: 
```php
abstract class A
{
	public function self(): self
	{
		return $this;
	}

	public function static(): static
	{
		return $this;
	}
}

final class B extends A
{

}

$b = new B();

$self = $b->self(); // returns an instance of B, but static analysers understands it only as instance of A, 
                    // because this is what the type declaration literally says

$static = $b->static(); // also returns an instance of B, and static analysers understands it as instance of B
```

### Types in inheritance

PHP's inheritance model follows [Liskov's Substitution Principle](https://en.wikipedia.org/wiki/Liskov_substitution_principle), and as such method return types are **covariant**, arguments types are **contravariant**, and properties types are **invariant**.

What this means is actually simpler than what it sounds like:

Overloaded method return types can be **the same** or become **stricter** (narrower) in children, than in their parent.  
Or, **overloaded methods can have return types if the parents have none**.

Overloaded method arguments, *on the contrary*, can only be **the same** or become **wider** in children, because the type declaration used in children is a superset of the one in the parent or because another type declaration has been added as a union.  
When the parent has no type declaration, the children can only gain the `mixed` type declaration.  
A *mandatory* argument in a parent can **become optional** in children, but the opposite is not true.

Finally, properties are simpler since their type (whether they have one or not) **can't change at all** in children.

A valid example: 
```php
abstract class A 
{
	public int $property;
	public $property2;

	public function method(stdClass $a, $b)
	{

	}
}

final class B extends A
{
	public int $property; // no change
	public $property2; // no change

	public function method(
		object $a, // the type for $a is now wider since it accept any objects instead of just stdClass
		mixed $b = null // $b still accept anything, but at least has a type declaration now :), and is also now optionnal
	): int { // the method has gained a strict return type
		return 1;
	}
}
```

### Classes as argument types

When the argument type declaration are class names, it becomes a little complicated.

For instance this code is not valid, because B is not wider than A since it's a children.
```php
abstract class A 
{
	public function method(A $a): void
	{

	}
}

final class B extends A
{
	public function method(B $a): void
	{

	}
}
```

The `self` type declaration can be used to represent the current class. In the example above, replacing both A and B by self would lead to the same invalid result.

This below however is valid because the type stays the same (which doesn't prevent to pass an instance of B to B's method):
```php
abstract class A 
{
	public function method(self $a)
	{

	}
}

final class B extends A
{
	public function method(A $a)
	{

	}
}
```

### Classes as method return types

Contrary to arguments, since return types can become stricter in children, overloaded methods that return an instance of the own class can return `self` both in parent and children :
```php
abstract class A 
{
	public function method(): self
	{
		return $this;
	}
}

final class B extends A
{
	public function method(): self
	{
		return $this;
	}
}
```

As we have seen in the other example above, A's method could have the `static` type declaration so that B doesn't have to overload the method to update the type declaration.

### Enums

Brought by PHP8.1, [enums](https://www.php.net/manual/en/language.enumerations.php) are actually a special case of objects and their (class/enum) name can likewise be used as type declaration.  

There is no generic `enum` type declaration, but all enums match the **`\UnitEnum`** class that is the parent of all enums.  
Backed enums, in addition, extends the **`\BackedEnum`** class, that extends `\UnitEnum`.

Backed enums use an `int` or `string` value as their cases, but there is no cast to/from the enum cases and their scalar values.  
To get an enum case from the scalar value, you have to use the built-in `BackedEnum::from(int|string $value): self` method.  
To get the scalar value from a backed enum, you have to use the `value` property on the enum case.

Examples: 
```php
// a regular enum
enum Letter
{
	case A;
	case B;
}

// a backed enum
enum Number: int
{
	case One = 1;
	case Two = 2;
}

function foo(UnitEnum $bar): void
{
}

foo(Number::Two); // OK
foo(Letter::A); // OK
foo(new stdClass()); // PHP Fatal error:  Uncaught TypeError: foo(): Argument #1 ($bar) must be of type BackedEnum, stdClass given, ...

// ----------

var_dump(Number::from(2), Number::One->value);
// enum(Number::Two)
// int(1)

function foo2(Number $bar): void
{
}

foo2(Number::Two); // OK
foo2(Number::from(1)); // OK
foo2(1); // PHP Fatal error:  Uncaught TypeError: foo2(): Argument #1 ($bar) must be of type Number, int given, ...
```

## Numeric strings

A [numeric string](https://www.php.net/manual/en/language.types.numeric-strings.php) is a string that only contains something that looks like a number:

- a regular `int`
- a `float` with the dot separator
- or a `float` in the exponent form, that is a `float` followed by the letter `e` or `E` and then an `int` as the exponent

White-spaces are allowed before or after the number, but the presence of anything else than that makes it not numeric.

Examples: `'123'`, `' -123 '`, `' 12.3 '`, `'12.3e4 '`,

Numeric strings can automatically be cast and compared to `int` or `float`.

However, strings that are not numeric but **that begins** by a number can still be expressly cast to int or float.

Example: 
```php
foo (int $foo): void
{
	var_dump($foo);
}

foo('123'); // int(123)

foo('123 A'); // PHP Fatal error:  Uncaught TypeError: foo(): Argument #1 ($foo) must be of type int, string given

foo((int) '123 A'); // int(123)
```

## Union and intersection types

For the situations where the value can be of one of several types, it is possible to list them all [in a union](https://www.php.net/manual/en/language.types.declarations.php#language.types.declarations.composite.union), not just one by separating them with the unary or operator `|` like it is in several examples above.

Intersection types are for the situation where a value must be of several types at the same time, which can only happen for objects. Intersection types use the `&` operator and are naturally limited to class names and are not nullable.

Since PHP8.2, it is even possible to [combine union and intersection](https://www.php.net/releases/8.2/en.php#dnf_types) types.

## Getting type information from a variable

### Check if a value is of a given type

To check if a variable is of a specific type, the `is_{type}(mixed $value): bool` family of functions can be used.  
There is one per type and type declaration, plus some others.

The ones that match type declarations are:

- `is_string()`
- `is_bool()`
- `is_int()`
- `is_float()`
- `is_array()`
- `is_object()`
- `is_callable()`
- `is_iterable()`
- `is_null()`

Others are:

- `is_resource()`
- `is_countable()` returns true when the value match the union type declaration `array|\Countable` so when it's an array, or an object that implements the `\Countable` interface
- `is_numeric()` returns true for `float`, `int` or numerical strings
- `is_scalar()` returns true for the 4 scalar types

### Get the type of a given value

When instead you want to get the name of the type of a variable, the `get_debug_type()` or `gettype()` functions can be used.

**[`get_debug_type(mixed $value): string`](https://www.php.net/manual/en/function.get-debug-type)** is typically the more pertinent to use.    
It returns the name of the value's type as a string (`null`, `bool`, `int`, `float`, `string`, `array`), but when the value is an object or an enum it returns the class/enum name, or when it is a resource it returns also the resource name `resource (resourcename)`, or its closed state `resource (closed)`.


Before PHP8.0, only **[`gettype(mixed $value): string`](https://www.php.net/manual/en/function.gettype.php)** (yes, without underscore) was available.  
It also returns the value's type as a string, but using a legacy longform version of the name (`NULL`, `boolean`, `integer`, `double`, `string`, `array`, `unknown type`, `object`, `resource` or `resource (closed)`).  
When the value is an object (or an enum), it just returns `object`. So if you want to also get the class name, you must call separately `get_class(object $object): string` (or if on PHP8.0+ use the `$object::class` magic constant).

### Check if an object is an instance of a class (or interface)

To check if an object is of a certain class name (or one of its children) a few more functions can be used:

- [`is_a()`](https://www.php.net/manual/en/function.is-a) that can be used in two ways : `is_a(object $object, string $className): bool` or `is_a(string $className, string $className, true): bool` returns true if the object or class name passed as first argument is an instance of the class name passed as second argument, or one of its children
- it's quite rare to see the `is_a()` function in the wild nowadays because the equivalent [`instanceof` operator](https://www.php.net/manual/en/language.operators.type.php) is a lot more used instead of the first usage when the compared value is an instance : `$object instanceof $className` is the same as `is_a($object, $className)`. The operator returns `false` is the left-hand value is anything else than an object. Note on the syntax of the operator that the right-hand value is either a variable that contain the FQCN of the class, or just the classname (like in a type declaration), without the `::class` magic constant.
- to specifically check if a value is a subclass (a children) of another, then the [`is_subclass_of()`](https://www.php.net/manual/en/function.is-subclass-of.php) function can be used, with the same signature as `is_a()`


## Casting

**[Casting](https://www.php.net/manual/en/language.types.type-juggling.php#language.types.typecasting)** is changing the type of a value to another type.

First, you can cast scalar types to other scalar types, except for non-numeric strings that can not be automatically cast to int or float (but they can be explicitly cast).

Objects that implements the `Stringable` interface can naturally be cast to string.  
The `Stringable` interface has the classic `__toString(): string` method, but it was introduced only in PHP8.0, where `__toString()` exists since the introduction OOP in PHP5.0.  
So because of that every classes that have the `__toString()` function are considered to implicitly implement the interface.

Any scalar and arrays can be cast to objects:

- Casting a scalar to an object will result in an instance of `\stdClass` (PHP's default built-in class) with a `scalar` property that contains the value.  
- Casting an array to an object will result in an instance of `\stdClass`, where each top-level array keys (numerical or not) are now the properties of the object.  

Any scalar and objects can be also cast to arrays:

- Casting a scalar to an array will result in an array with the 0 key and the scalar as value.  
- Casting an object to an array will result in an array where each of the object's properties become the top-level array keys. Public property names are kept as-is but protected and private properties are also exposed in the array with the name changed.

If casting a scalar to an object seems rather pointless, casting a scalar to an array is actually useful when for instance a function has an argument that accept a single value as a scalar or multiple values as an array. You can then just cast the argument to an array to make sure it is loopable.
```php
/**
 * @param array<string>|string $items
 */
function foo(array|string $items): void
{
	foreach ((array) $items as $item) {
		
	}
}
```

**Up and down casting** are not possible. An object can not be cast to another object, and can never change class name even in the rare case where it may make sense like casting to a parent.

### Syntax 

Casting can be done by the typical syntax used by many other programming languages: prepending the value or variable by the target type surrounded by parenthesis.

Example where the value of the `$source` variable is cast to `int` when assigned to the `$target` variable: `$target = (int) $source;`.  
The cast doesn't happen in-place, the value of `$source` variable is not itself changed.  
But since variable's value can change type, you can also write  `$source = (int) $source;` to change the type of the source variable (actually of its value).

Alternatively, it can be done with the **`settype(mixed &$var, string $type)`** function where the variable is here changed by reference: `settype($source, 'int');`.  
The `type` argument accept the four scalar type names, as well as `array`, `null` and `object`.

In addition, the scalar types also have a dedicated `*val()` function for conversion, which can be handy when used for instance as the callback of `array_map()`:

- `boolval(mixed $value): bool`
- `floatval(mixed $value): float`
- `intval(mixed $value, int $base = 10): int`
- `strval(mixed $value): string`

Examples: 
```php
$array = [1, 2, 3, 4];

$array = array_map('strval', $array); // returns ["1", "2", "3", "4"]

// or 
$array = array_map(fn (int $value) => (string) $value, $array);

// or, when not using strict types
$array = array_map(fn (int $value): string => $value, $array);
```

## Comparing and juggling types

**[Type juggling](https://www.php.net/manual/en/language.types.type-juggling.php)** is automatic casting that happens when a statement is working with different types, like comparing, adding, concatenating different types.


Like JavaScript, PHP has two different equality operators: `==` and `===` (as well as `!=` and `!==`).

`===` and `!==` will not attempt to cast values on either sides and will only actually compare the values if they are exactly of the same type, otherwise the comparison will just be `false`.

`==` and `!=` on the other hand, when comparing values of different types will attempt to convert one of the two into the other before actually comparing the values.  
If one of the type is numerical, the other value will be tried to be converted to its type.

Other comparison operators are always non-strict and will cast one side or both to number if possible. If the cast is not possible, then a `\TypeError` exception will be thrown.

Some built-in comparison functions like `in_array` have a `strict` argument that is `false` by default, but that can be set to `true` to use strict comparison.

Examples:
```php
1 == '1' // true
1 === '1' // false

in_array('0', ['0000']) // true 
in_array('0', [0]) // true
 
in_array('0', ['0000'], true) // false
in_array('0', [0], true) // false
```

The PHP manual [has a whole page with tables](https://www.php.net/manual/en/types.comparisons.php) for comparing comparison with `==`, `===` and functions like `gettype()`, `isset()`, `empty()`, etc...

## Strict types

As we have seen above values can be automatically cast in multiple situations.  
Three of these are:
- during a function call when a value is passed to a typed parameter
- when a function that has a return type declaration returns a value.
- when a value is assigned to a typed class property

This is the default behavior and is known as "weak types" mode.  
But, introduced in PHP7.0 at the same time as the scalar type declarations, there is also a [**strict types** mode](https://www.php.net/manual/en/language.types.declarations.php#language.types.declarations.strict), that specifically prevent these three behaviors (and only theses).

The strict types mode can be activated on a per-file basis by adding the `declare(strict_types=1);` directive as the very first line in the file after the PHP opening tag.

All it does is prevent type juggling:  
- for function calls that **happens** in that file, no matter where the function is declared (in the core, in extensions or any other userland script)
- for returned values of functions **declared** in that file
- for assignment to typed class properties that **happens** in that file, no matter where the class is defined

Any need for type juggling in such places in file that have that directive will throw a `TypeError`, with one exception: **juggling from `int` to `float` is always allowed**.

Examples:
```php
// without strict types:
$a = null;
$a = trim($a); // string(0) "" 
// because null is automatically cast to '' when passed to the trim() function

// ----------
// with strict types:
declare(strict_types=1);

$a = null;
$a = trim($a); // PHP Fatal error:  Uncaught TypeError: trim(): Argument #1 ($string) must be of type string, null given

// So in that case, if you know the $a variable may not be a string, you have to cast is expressly:
$a = trim((string) $a);
```

*Using the directive or not is as much controversial as declaring classes `final` or not in libraries.*

On one hand, it is difficult and even dangerous to use because it makes otherwise perfectly valid and working code throw a `TypeError` and thus cause a runtime crash.  
So if you use it, you have to be sure that you are working with the correct type.  
In that situation, both good tests and using type declarations helps because when a function has typed arguments, you **know** that they are of that type inside the function body.

On the other hand, as for many small things like the `final` keyword, or precisely type declarations, it gives a little bit more control and confidence about the type safety and overall quality of the code that use it.  
**But it only works if it is properly tested.**

**On a personal note here**, I like to add the directive on all **new** files, or on older files that do not have it yet only when they are **fully tested and fully refactored** to use type declarations everywhere.  
And even then I sometimes purposefully restrain myself from adding it because in the end, I don't find it worth risking a runtime error in a critical system in production.

Of all the features discussed here, I find it overall the less impactful, and in a project that already exists, an actual risk of generating more errors than it would prevent.

Since the strict types mode is activated by a `declare` directive in the files, it can not be changed dynamically like for most `ini` settings.  
However, for the most adventurous of you, you *could* add the directive on all the files before running the tests in a CI environment or even during the deployment step to your staging environment, for instance.  
Tools like [PHP CS Fixer](https://cs.symfony.com) can make it very easy.


## Dealing with nulls

Whether or not it is a billion dollar mistake, and even if you are very good at using the [null object pattern](https://en.wikipedia.org/wiki/Null_object_pattern) you will have to deal with variables that can be `null` at some point.  

Thankfully PHP makes it pretty easy nowadays.

`null` is its own type that has no other value:
```php
var_dump(gettype(null), get_debug_type(null), null);
// string(4) "NULL"
// string(4) "null"
// NULL
```

Null is always **a falsy value** (it behave the same as `false` in conditions), and can automatically cast to the scalars `false`, `0`, `0.0` and `''` (empty sting) which are also all falsy.  
It can also be expressly cast to an empty array or `stdClass` instance.

`null` is the default value for un-typed properties that have no explicit default initializers.

Checking if a variable is `null` can be done with the [`is_null(mixed $value): bool`](https://www.php.net/manual/en/function.is-null) function, the strict equality operator, or the [`isset(mixed $value): bool`](https://www.php.net/manual/en/function.isset) and [`empty(mixed $value): bool`](https://www.php.net/manual/en/function.empty) constructs:
```php
$nullVariable = null;

is_null($nullVariable) // true
$nullVariable == null // true
$nullVariable === null // true

isset($nullVariable) // false
empty($nullVariable) // true
```

Using the equality operator with only two equal signs is probably not what you want since `null` will be cast to the other operand's type except when it's an object:
```php
$iAmNull == null // true
null == null // true
false == null // true
0.0 == null // true
0 == null // true
'' == null // true
[] == null // true
new stdClass == null // false
((object) null) == null // false
```

The `isset()` construct is also most frequently used to check if an array contains a particular key.  
However it will return `false` even when the array key does exist but with `null` as its value.  

In the cases where you need to check that the array has a key, even when the value is `null`, you can use the [`array_key_exists(int|string $key, array $array): bool`](https://www.php.net/manual/en/function.array-key-exists) function.

```php
$array = [
	'null' => null,
];

isset($array['null']); // false
array_key_exists('null', $array); // true

isset($array['non_existant_key']); // false
array_key_exists('non_existant_key', $array); // false
```

### Short ternary operator

A very common use case it to assign the value of a variable if it's not null, or another value.  

We can make it look like really nice and short by taking advantage that `null` is falsy with the short form of the ternary operator (`?:`, also called the "elvis" operator) that return the value from the condition if it's thruthy, or the right-most value.

```php
// $amINull is mixed
$default = 'default';

$var = $default;
if ($amINull !== null) {
	$var = $amINull;
}

// the same with a ternary
$var = $amINull !== null ? $amINull : $default;

// the same with a short ternary
$var = $amINull ?: $default;
```

As for any comparisons, be aware that it may not work as expected when the value is not `null`, but still falsy.  
All the cases below would return `'default'`:  
```php
false ?: 'default';
0.0 ?: 'default';
0 ?: 'default';
'' ?: 'default';
'0' ?: 'default';
[] ?: 'default';
null ?: 'default';

[][0] ?: 'default'; // PHP Warning:  Undefined array key 0 in ...
```

### Null-coalescing operator

Accessing an array key that doesn't exist throw a warning as in the last line above, and the code required to make sure that the key exists beforehand can become quickly cumbersome, especially with nested arrays.  

That's mostly why PHP7.0 added the **null coalescing operator** (**`??`**) which has two distinct behaviors from the short ternary regarding arrays and nulls:

- it gracefully checks if the array key exists. If it exists, it evaluates its value, but returns the right-hand value if it doesn't.
- it checks if the left-hand value (which may be the array key's value) is **exactly** `null` instead of just if it's falsy

Examples:
```php
false ?? 'default'; // false
0.0 ?? 'default'; // 0.0
0 ?? 'default'; // 0
'' ?? 'default'; // ''
'0' ?? 'default'; // '0'
[] ?? 'default'; // []
[][0] ?? 'default'; // 'default'
null ?? 'default'; // 'default'

$array = [
	'null' => null,
];
$array['null'] ?? 'default'; // 'default'
```

The **null-coalescing assignment operator** (**`??=`**), added in PHP7.4 behave the same.  

The right-hand value is only assigned to the left-hand variable (or array key) when it:
- is exactly `null`
- or doesn't exists (an uninitialized variable)
- or an array key that doesn't exist
- or an array key that exists but has the `null` value

Examples:
```php
$var = false; 
var_dump($var ??= 'default'); // bool(false)

$var = 0.0;
var_dump($var ??= 'default'); // float(0.0)

$var = 0;
var_dump($var ??= 'default'); // int(0)

$var = '';
var_dump($var ??= 'default'); // string(0) ""

$var = '0';
var_dump($var ??= 'default'); // string(1) "0"

$var = [];
var_dump($var ??= 'default'); // array(0) {}

$var = [];
var_dump($var[0] ??= 'default'); // string(7) "default"

$var = null;
var_dump($var ??= 'default'); // string(7) "default"

var_dump($varThatDontExists); // PHP Warning:  Undefined variable $varThatDontExists in ...
var_dump($varThatDontExists ??= 'default'); // 'default'
var_dump($varThatDontExists); // 'default'

$array = [
	'null' => null,
	'zero' => 0,
];
var_dump($array['null'] ??= 'default'); // string(7) "default"
var_dump($array['zero'] ??= 'default'); // int(0)
```


### Null-safe (object) operator

The null-coalescing operator also works when accessing objects properties, even when chained, and the whole expression would return the right-hand value if any of the properties happens to be `null` instead of an object.

```php
$object = new stdClass();
$object->property1 = new stdClass();

$object
	->property1
	->property2
	->property3
	->getStuff() 
	?? 'default'; // 'default'
```

However, it doesn't work when a method is called in the chain and returns `null` instead of an object:
```php
$object = new stdClass();
$object
	->property1 = new class { 
	public function someMethod(): ?object
	{
		return null;
	}
};

$object
	->property1
	->someMethod()
	->property3
	->getStuff() 
	?? 'default';

// outputs: 
// PHP Warning:  Attempt to read property "property3" on null in ...
// PHP Fatal error:  Uncaught Error: Call to a member function getStuff() on null in ...
```

This is the kind of situation where you had to resort to a lot of null checks inside if statements to get a code that doesn't explode.

PHP8.0, praise be, added the **null-safe operator** (**`?`**) (sometimes called "optional chaining"), that is to be used just before the arrow `->` to access a property or a method on an object.

If the value is `null` instead of an object, then the whole chain returns `null`, which you can then default to another value with the null-coalescing operator:
```php
$object = new stdClass();
$object
	->property1 = new class { 
	public function someMethod(): ?object
	{
		return null;
	}
};

$object
	->property1
	->someMethod()
	?->property3
	->getStuff() // NULL

$object
	->property1
	->someMethod()
	?->property3
	->getStuff()
	?? 'default' // string(7) "default"
```
