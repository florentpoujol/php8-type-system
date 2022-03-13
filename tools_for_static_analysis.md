# Tools for static analysis

## Table of content

- [PHPDocs](#phpdocs)
	- [Type related tags](#type-related-tags)
- [Static analysers](#static-analysers)
	- [Custom PHPDoc types](#custom-phpdoc-types)
	- [Generics](#generics)
	- [PHPStan](#phpstan)
- [Tools to helps with PHPDocs and typehints](#tools-to-helps-with-phpdocs-and-typehints)
	- [PHP CS Fixer](#php-cs-fixer)
	- [Rector](#rector)
	- [Runtime assertions](#runtime-assertions)

## PHPDocs

PHPDocs are a way to add metadata about the code via some **DocBlocks**, structured comments that begins by `/**`, that contain **tags**, keyword that begins by `@`.  
Since they are comments they are completely ignored by PHP itself and are only meant to be read by humans and static analysers.

PHPDocs have a lot more usages than the tags related to typings, and they allowed to type variables, arguments, return value and properties long before the PHP type system existed.  
But even if today some can be replaced by typehints, most PHPDocs remains pertinent because they allow to extend the built-in type system to be more precise.

Also since they are comments and do not require any changes to the language, PHPDocs are very evolutive and any tools can just "invent" new tags or interpret existing one in a different way.  

They are a convenient way to add features that are used today by static analysers to add more and stricter types and enable generics.  
They were also used by Doctrine and Symfony to emulate annotations before they became a built-in feature in PHP8.0

### Type-related tags

In this section we will precisely checkout the tags that concerns the type system.  
In all cases, the type is written first, then the variable/argument name, and then a description can be added.

**@var** is used to type a class constant, a property or a variable. It is followed by the type and then the variable name:
```php
// when used with variable, the variable name is mandatory
/** @var int $theVariable */
$theVariable = 1;

final class Foo
{
	/** @var callable */
	private $callback;

	/** @var array<string> */
	public const CARDS = ['club', 'heart', 'spade', 'diamond'];
}
```

**@param** is used to type an argument, and **@return** the return type of a function or method.
```php
/**
 * @param int $value
 * 
 * @return string
 */
function intToString($value)
{
	return (string) $value;
}
```

It is also possible to "extends" a class by defining properties and methods via PHPDocs.

**@property** is used over a class to declare a property on that class. There is also two tags for read-only or even write-only properties: **@property-read** and **@property-write**. These properties are always assumed public and can be made static if the tag is immediately followed by the `static` keyword.

```php
/**
 * @property callable $callback Some description
 * @property-read static null|string $someStaticProperty
 */
final class Foo
{
}

Foo::$someStaticProperty = 'bar'; // static analysers would flag this as an error since the property is marked read-only
```

Similarly, **@method** is used to declare a public method on a class. In this case the method name isn't preceded by the dollars sign, and unlike return typehints, the return type is before the signature

```php
/**
 * @method float getPrice(object $someObject) Get the price of the object
 * @method static float getPrice(object $someObject) Get the price of the object
 */
final class Foo
{
}
```

Finally there is the **@mixin** tag which is used over a class and target a class. It will "import" all symbols of the target to the current

```php
/**
 * @property mixed $foo
 */
final class Foo
{
	public function foobar()
	{

	}
}

/**
 * @mixin Foo
 */
final class Bar
{

}

$bar = new Bar();
$bar->foo = 'struff';
$bar->foobar();

// these two statements above are valid from a static analysis standpoint even if the method call would cause a runtime error
```

## Static analysers

### Custom PHPDoc types

Static analysers define their own new tags and/or understand "extended" usages of existing tags.

In this section we will list all type you can use via PHPDocs and can be understood by PHPStan. Other tools like Psalm may define some others.

All PHP's built-in typehint as well as all classes can be used as PHPDocs.

**class-string**: the value must be a string that match an existing fully qualified class name. Even when not using generics, you can express the class name between brackets : `class-string<\App\MyClass>`

Other string specific variant exists : `callable-string` (match a callable but only as a string, so functions and static classes), `numeric-string`, and `non-empty-string`

**resource**: resource is actually the type a variable can have in PHP, but it's a legacy type that represent basically stream/file pointers and are progressively replaced by actual objects, so there will never be a matching typehint.

**true** and **false**: false is a typehint but that can only be used as return type, but for the rare situations you need them, you can used them as PHPDocs

integer ranges: even if a variable expect an int, it may not accept any value, so range of value can be expressed like so: `positive-int` (include 0), `negative-int`, `int<0, 100>`, `int<min, 100>`, `int<50, max>`.

You can even use scalar values and constant names as PHPDocs to make sure the passed value is 
```php
final class Foo
{
	public const FOO_CASE_1 = 1;
	public const FOO_CASE_2 = 2;

	/**
	 * @param 'some'|'value' $value
	 * @param self::FOO_CASE_* $value2
	 * 
	 */
	function foo(string $value, int $value2)
	{
	}
}
```

When a value is a callable, not only it is possible to use the simple callable PHPDocs especially on properties that can't use the typehint, you can also define callable signature like you would for method with the `@method` tag.


### Generics

Ah, generics. A hot topic in the PHP world nowadays.

Generics is a way for the language/static analyser to know that a class or another language construct (like arrays) is **working with** another.

A typical example is a collection, so a class that "contains" other values.

Consider the following collection:
```php
final class Collection
{
	private array $items = [];

	public function add(mixed $value): self
	{
		$this->items[] = $value;

		return $this;
	}

	public function pop(): mixed
	{
		return array_pop($this->items);
	}
}
```

With just this code we could pass any kind of values to the add() method and we can't expect to get nything relaiable from the pop() method.

Using generics would be to expressly tell that an instance of this class is expected to work only with a single type. Ideally we would then get a type error if we pass another type to the add() function and we would then be sure of whaat type the pop() function returns.

```php
$collection = new Collection<int>();

$collection->add(1);
$collection->add('1'); // type error

$id = $collection->pop(); // we *know* that id can only be of type int
```

You can built that feature with a collection that will manually check at runtime.

We could imagine a similar syntax for arrays:
```php
$arrayClass = <stdClass>[
	new stdClass(), 
	'foo', // type error
];

$arrayInt = <string, int>[
	'foo' => 1,
	1 => 2, // type error on the key
	'bar' => 'foo', // type error on the value
];

function foo(array<stdClass> $array)
{
	//
}

foo($arrayClass);
foo($arrayInt); // type error
```

**As of PHP8.1, generics are not available in the language, and it is likely that it won't be the case any time soon.**

But during the PHP5 time, where only the `array` and later the `callable` typehints were available, the same thing was said about the scalar typehints for instance.  
And yet scalar and return typehints were there in PHP7.0, and in a sense kickstarted the move toward a more complete type system that almost every new minor and major version until them contributed to.

So who knows ? Many generics will be the flagship feature for PHP9.0 after all ?
Or we will find that with full-time developer to work on the core (paid by [the PHP Foundation](https://opencollective.com/phpfoundation)), it is actually more pertinent to implement them in a minor release until that...

THanksfull this chapter doesn't end there and generics can actually be used today, through PHPDocs and static analysers.

TODO (in the meantime, see theses links):
- [Generics in PHP using PHPDocs, in PHPStan's blog (from 2019)](https://phpstan.org/blog/generics-in-php-using-phpdocs) 
- [Template annotation, in Psalm documentation](https://psalm.dev/docs/annotating_code/templated_annotations/)  
- [See also the Generics section is the references](references.md#generics) 


### PHPStan

TODO (see https://phpstan.org/user-guide/getting-started for now)


## Tools to helps with PHPDocs and typehints

Beside the static analysers themselves, there are other tools that will help you manage PHPDocs, typehints, enforce the use of some features over others, and overall help make you code stricter.

This is particularly useful on an existing codebase that needs modernising where doing all that especialy in a team can be extremely time consuming, or downright impossible.

But it is also useful on a newer codebase to keep consistent and easily keep-up with both increasing team, size and newer PHP versions.

### PHP CS Fixer

[PHP CS Fixer](https://cs.symfony.com) is a popular tool to lint and fix coding standards.  

Coding standards are about more that just visual style and can be about enforcing using some language features over others, which can helps to make our code stricter.

PHP CS Fixer has [many built-in rules](https://cs.symfony.com/doc/rules/index.html), which a good number relates to types/typing.  
Some are just about styling (like having a space or not between casts and the variables), but others will actually change to make use of newer features, or standardise how you do things (like how you check for `null` values, with `=== null` instead of `is_null()`).

The rules in italic are considered **risky** because the modifications they make can actually cause bugs and runtime type errors, especially when used over an existing project.   
That's why in the tool configuration, you have to explicitly allow to run risky rules, which is off by default.

- Typehints
	- [mark expressly nullable arguments with null default value](https://cs.symfony.com/doc/rules/function_notation/nullable_type_declaration_for_default_null_value.html): add or remove the nullable operator (`?`) on arguments that have `null` as default value (that are implicitly nullable)
	- *[phpdoc to argument typehint](https://cs.symfony.com/doc/rules/function_notation/phpdoc_to_param_type.html)*: replace an argument PHPDoc type to the corresponding typehint
	- *[phpdoc to property typehint](https://cs.symfony.com/doc/rules/function_notation/phpdoc_to_property_type.html)*: replace a property PHPDoc type to the corresponding typehint
	- *[phpdoc to return typehint](https://cs.symfony.com/doc/rules/function_notation/phpdoc_to_return_type.html)*: replace a return type PHPDoc to the corresponding typehint
	- *[void return type](https://cs.symfony.com/doc/rules/function_notation/void_return.html)*: add the `void` return typehint to all functions that do not have return typehint or PHPDoc and that do not return anything
	- [fully qualified typehint](https://cs.symfony.com/doc/rules/import/fully_qualified_strict_types.html): force classes typehints to use their base class name instead of their FQCN
	- [compact nullable typehint](https://cs.symfony.com/doc/rules/whitespace/compact_nullable_typehint.html): remove spaces before the `?`operator in nullable typehints
	- [union operator space](https://cs.symfony.com/doc/rules/whitespace/types_spaces.html): force zero or one space around the union `|` operator (only in the context of union typehints)
- Strict
	- *[declare strict types](https://cs.symfony.com/doc/rules/strict/declare_strict_types.html)*: add the `declare(strict_types=1)` directive to all files
	- *[strict comparison](https://cs.symfony.com/doc/rules/strict/strict_comparison.html)*: forces all comparison to be strict (replace `==` by `===`)
	- *[strict param](https://cs.symfony.com/doc/rules/strict/strict_param.html)*: force the third `bool $strict` argument on some built-in functions like (`in_array()`)) to be `true`
- Operators
	- *[is_null](https://cs.symfony.com/doc/rules/language_construct/is_null.html)*: replace the `is_null($var)` function usages by `$var === null`
	- [null coalescing to coalesce equal operators](https://cs.symfony.com/doc/rules/operator/assign_null_coalescing_to_coalesce_equal.html): replace `??` by `??=` where applicable
	- *[ternary to elvis operator](https://cs.symfony.com/doc/rules/operator/ternary_to_elvis_operator.html)*: replace ternary operator by the short ternary, where applicable (`$a ? $a : $b` > `$a ?: $b`)
- Casts
	- *[settype() to cast](https://cs.symfony.com/doc/rules/alias/set_type_to_cast.html)*: replace usages of the `settype()` function to casts using the `(type)` syntax
	- *[`*val()` function to cast](https://cs.symfony.com/doc/rules/cast_notation/modernize_types_casting.html)*: replace usages of the scalar `*val()` functions (like `intval(): int`) to casts using the `(type)` syntax
	- [cast spaces](https://cs.symfony.com/doc/rules/cast_notation/cast_spaces.html): to have zero or one space between the cast and the 	variable
	- [lowercase cast](https://cs.symfony.com/doc/rules/cast_notation/lowercase_cast.html): force all cast keywords to be lowercase
	- [no short bool cast](https://cs.symfony.com/doc/rules/cast_notation/no_short_bool_cast.html): replace usages of two consecutive `!` operators that could be used to cast a variable to bool
	- [no unset cast](https://cs.symfony.com/doc/rules/cast_notation/no_unset_cast.html): replace usages of the `(unset)` cast to simply setting the variable to `null`
	- [short scalar cast](https://cs.symfony.com/doc/rules/cast_notation/short_scalar_cast.html): replace "non-standard" casts for the scalar types (like `(boolean)`, `(real)`) by their more common ones (like `(bool)` and `(float)`)
- PHPDocs
	- *[comment to PHPDoc](https://cs.symfony.com/doc/rules/comment/comment_to_phpdoc.html)*: properly use docblock with annotations so that they aren't ignored
	- [no superfluous PHPDoc tag](https://cs.symfony.com/doc/rules/phpdoc/no_superfluous_phpdoc_tags.html): removes redundant PHPDocs for arguments and return type when there already is a typehint with the same information
	- [PHPDoc scalar](https://cs.symfony.com/doc/rules/phpdoc/phpdoc_scalar.html): standardize the types used in PHPDoc (`int` instead of `boolean`)
- classes
	- [no null property initialization](https://cs.symfony.com/doc/rules/class_notation/no_null_property_initialization.html): prevent expressly defining `null` as the default value of untyped properties (since it is already their default value).
	- [self over static access](https://cs.symfony.com/doc/rules/class_notation/self_static_accessor.html): in final classes, use the `self` keyword instead of `static` (since `static` always equal to `self`)
	- [void don't return null](https://cs.symfony.com/doc/rules/return_notation/simplified_null_return.html): prevent `void` function to expressly return `null` (even if technically this is implicitly the case)
- PHPUnit
	- *[PHPUnit strict](https://cs.symfony.com/doc/rules/php_unit/php_unit_strict.html)*: use PHPUnit's `assertSame()` instead of `assertEquals()` to do strict comparisons instead of a loose ones


It is then up to you to not blindly trust the modifications made by the tool but complete them with careful code review, static analysis and tests to make sure the code still works.

PHP CS Fixer has many other non-type related rules, but you can also [create your own rules](https://cs.symfony.com/doc/custom_rules.html) if you are not satisfied with the existing ones.


### Rector

[Rector](https://getrector.org/) is another popular tool that is more marketed toward general refactoring rather than coding standards specifically.

It has even more rules that can do a whole lot more than PHP CS Fixer, so I will not completely list them all here.

- [Strict](https://github.com/rectorphp/rector/blob/main/docs/rector_rules_overview.md#strict): these rules are about refactoring case where a non-bool variable is used as-is in condition, relying on type juggling to make ti falsy or not. Whenever it can, thanks to typehints or PHPDocs, turn theses cases to strict comparison with null, empty array or empty string.
- [Type declaration](https://github.com/rectorphp/rector/blob/main/docs/rector_rules_overview.md#typedeclaration): 20+ fine-grained rules, mostly to add typehints where they can be inferred

Other rule sections are not dedicated to type-related rules but still contain several interesting ones:

- Code quality
	- [CountArrayToEmptyArrayComparisonRector](https://github.com/rectorphp/rector/blob/main/docs/rector_rules_overview.md#countarraytoemptyarraycomparisonrector): replace usage of the `count()` function on array to comparison to and empty array
	- [NullableCompareToNullRector](https://github.com/rectorphp/rector/blob/main/docs/rector_rules_overview.md#nullablecomparetonullrector): changes negate of empty comparison of nullable value to explicit `===` or `!==` compare
	- [StrictArraySearchRector](https://github.com/rectorphp/rector/blob/main/docs/rector_rules_overview.md#strictarraysearchrector): makes `array_search()` search for identical elements
	- [VarConstantCommentRector](https://github.com/rectorphp/rector/blob/main/docs/rector_rules_overview.md#varconstantcommentrector): constant should have a `@var` comment with type
- Coding style
	- [IntvalToTypeCastRector](https://github.com/rectorphp/rector/blob/main/docs/rector_rules_overview.md#intvaltotypecastrector): change `intval()` to faster and readable `(int) $value`
	- [NewStaticToNewSelfRector](https://github.com/rectorphp/rector/blob/main/docs/rector_rules_overview.md#newstatictonewselfrector): change unsafe new `static()` to new `self()`
	- [ReplaceMultipleBooleanNotRector](https://github.com/rectorphp/rector/blob/main/docs/rector_rules_overview.md#replacemultiplebooleannotrector): replace the Double not operator (`!!`) by type-casting to boolean
	- [SetTypeToCastRector](https://github.com/rectorphp/rector/blob/main/docs/rector_rules_overview.md#settypetocastrector): changes `settype()` to `(type)` where possible
	- [SimplifyEmptyArrayCheckRector](https://github.com/rectorphp/rector/blob/main/docs/rector_rules_overview.md#simplifyemptyarraycheckrector): simplify `is_array()` and `empty()` functions combination into a simple identical check for an empty array
	- [StrlenZeroToIdenticalEmptyStringRector](https://github.com/rectorphp/rector/blob/main/docs/rector_rules_overview.md#strlenzerotoidenticalemptystringrector): changes `strlen()` comparison to 0 to direct empty string compare
	- [UseIdenticalOverEqualWithSameTypeRector](https://github.com/rectorphp/rector/blob/main/docs/rector_rules_overview.md#useidenticaloverequalwithsametyperector): use `===`/`!==` over `==`/`!=`, it values have the same type
- Dead code
	- [RemoveConcatAutocastRector](https://github.com/rectorphp/rector/blob/main/docs/rector_rules_overview.md#removeconcatautocastrector): remove `(string)` casting when it comes to concat, that does this by default
	- [RemoveNullPropertyInitializationRector](https://github.com/rectorphp/rector/blob/main/docs/rector_rules_overview.md#removenullpropertyinitializationrector): remove initialization with `null` value from property declarations
	- [RemoveUselessParamTagRector](https://github.com/rectorphp/rector/blob/main/docs/rector_rules_overview.md#removeuselessparamtagrector): remove `@param` docblock with same type as parameter type
	- [RemoveUselessReturnTagRector](https://github.com/rectorphp/rector/blob/main/docs/rector_rules_overview.md#removeuselessreturntagrector): remove `@return` docblock with same type as defined in PHP
	- [RemoveUselessVarTagRector](https://github.com/rectorphp/rector/blob/main/docs/rector_rules_overview.md#removeuselessvartagrector): remove unused `@var` annotation for properties


### Runtime assertions

You can very easily manually assert (make sure) at runtime that a variable holds a value of the type you expect: a simple if that then throws an exception does the job:
```php
if (! is_int($value)) {
	throw new TypeError("Value should be an int");
}
```

However this has a few disadvantages :
- this is quite verbose with in this example a poor error message, and is always on 3 lines if you follow PSR's coding standard
- you can not easily turn it off in production
- static analysers may not completely understand that after the condition the variable is always an int

As always, both the PHP core and Composer packages comes to the recues to provide a better experience.


#### Built-in assertions

PHP has a built-in `assert(bool $condition, string|Throwable $descriptionOrException = null)` language construct, that basically throws an exception when the first argument is falsy.

The first argument should be an expression like in any condition.

When the assertion fail, it will throw an `AssertionError`, optionally with the provided error message as second argument.  
When the second argument is an exception instance, then this exception will be thrown instead of the `AssertionError`.

Examples: 
```php
$value = false;

assert(is_int($value)); // PHP Fatal error:  Uncaught AssertionError: assert(is_int($value))
assert(is_int($value), 'Value should be an int'); // PHP Fatal error:  Uncaught AssertionError: Value should be an int in ...
assert(is_int($value), new TypeError('Value should be an int')); // PHP Fatal error:  Uncaught TypeError: Value should be an int in ...
```

The very interesting thing with built-in assertions is that it is possible to completely turn them off so that they have **zero** cost in production.  
And when I say zero cost, I mean that the line(s) will not even be compiled into opcode. So the whole assert exception will  just be replaced by empty lines in the code that actually run.

This is controlled by the `zend.assertions` ini directive, that can not be changed at runtime (for the reason explained just above).  
It should be `1` to enable assertions, or `0` to disable them.

Check [the page on the PHP manual](https://www.php.net/manual/en/function.assert.php) for other ini directives you can use to further control assertion's behaviour, like emitting a warning instead of throwing an exception.

A nice thing also is that static analyser understand assertions and in the example above that the variable can only be an int for all the code below the assertion.

A note of caution, though, **assertions are not validation**. The `assert()` function must not replace proper validation and filtering of input, and should only be used for cases that should actually never be false.  
Since you should turn them off in production, your code must work as intended without assertions.


#### Composer libraries

The `assert()` function is built-in but also quite barebone.  

If you have need for more expressive assertions and easier error messages, you can instead rely on two similar composer packages: 
- https://github.com/beberlei/assert 
- https://github.com/webmozarts/assert

They both provide a static class with a lot of assertion methods for many use cases and a default and customizable exception when the assertion(s) fails.

A few examples, using Benjamin Eberlei's package:
```php
Assertion::max(2, 1); // PHP Fatal error:  Uncaught Assert\InvalidArgumentException: Number "2" was expected to be at most "1". in ...

Assertion::isJsonString('not a json string'); // PHP Fatal error:  Uncaught Assert\InvalidArgumentException: Value "not a json string" is not a valid JSON string. in ...

$value = '2022-01-23';
$format = 'Y-m-d H:i:s';
Assertion::date($value, $format); // PHP Fatal error:  Uncaught Assert\InvalidArgumentException: Date "2022-01-23" is invalid or does not match format "Y-m-d H:i:s". in ...
Assertion::date($value, $format, "The date '%s' is not in the right format '%s'"); // PHP Fatal error:  Uncaught Assert\InvalidArgumentException: The date '2022-01-23' is not in the right format 'Y-m-d H:i:s' in ...
```

Both packages differ slightly on their usability and handling of error messages.  
So check their readme for other features that just static checks as shown in the example and how you can customize the exception handling.

