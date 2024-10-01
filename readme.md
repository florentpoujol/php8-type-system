# PHP 8 type system in-depth, and tools for static analysis

This repo contains what I hope is a decent description of all aspects of **PHP 8.1+ type system**, the community tools to help work with type declarations and PHPDocs as well as the **static analysers** that rely on the type system to find inconsistencies and bugs in your programs.

<br>

- [PHP type system in depth](php_type_system_in_depth.md), a complete description of the type system, all type declarations and related topics like casting and dealing will `null` values
	- [Typing systems](php_type_system_in_depth.md#typing-systems)
	- [Type declaration syntax](php_type_system_in_depth.md#type-declaration-syntax)
	- [Why use types ?](php_type_system_in_depth.md#why-use-types-why-make-your-code-stricter-)
	- [Built-in type declarations](php_type_system_in_depth.md#built-in-type-declarations)
		- [Scalar type declarations](php_type_system_in_depth.md#scalar-type-declarations)
		- [Compound type declarations](php_type_system_in_depth.md#compound-type-declarations)
		- [Return-only type declarations](php_type_system_in_depth.md#return-only-type-declarations)
		- [Others](php_type_system_in_depth.md#others)
	- [Objects](php_type_system_in_depth.md#objects)
		- [type declarations](php_type_system_in_depth.md#type-declarations)
		- [Types in inheritance](php_type_system_in_depth.md#types-in-inheritance)
		- [Classes as argument types](php_type_system_in_depth.md#classes-as-argument-types)
		- [Classes as method return types](php_type_system_in_depth.md#classes-as-method-return-types)
		- [Enums](php_type_system_in_depth.md#enums)
	- [Numeric strings](php_type_system_in_depth.md#numeric-strings)
	- [Union and intersection types](php_type_system_in_depth.md#union-and-intersection-types)
	- [Getting type information from a variable](php_type_system_in_depth.md#getting-type-information-from-a-variable)
	- [Casting](php_type_system_in_depth.md#casting)
	- [Comparing and juggling types](php_type_system_in_depth.md#comparing-and-juggling-types)
	- [Strict types](php_type_system_in_depth.md#strict-types)
	- [Dealing with nulls](php_type_system_in_depth.md#dealing-with-nulls)

<br>

- [Tools for static analysis](tools_for_static_analysis.md)
	- [PHPDocs](tools_for_static_analysis.md#phpdocs)
		- [Type related tags](tools_for_static_analysis.md#type-related-tags)
	- [Static analysers](tools_for_static_analysis.md#static-analysers)
		- [Custom PHPDoc types](tools_for_static_analysis.md#custom-phpdoc-types)
		- [Generics](tools_for_static_analysis.md#generics)
		- [PHPStan](tools_for_static_analysis.md#phpstan)
	- [Tools to helps with PHPDocs and type declarations](tools_for_static_analysis.md#tools-to-helps-with-phpdocs-and-type-declarations)
		- [PHP CS Fixer](tools_for_static_analysis.md#php-cs-fixer)
		- [Rector](tools_for_static_analysis.md#rector)
		- [Runtime assertions](tools_for_static_analysis.md#runtime-assertions)

<br>

- [References](references.md) Various relevant links to the PHP manual, RFCs for all type related features over the years, and community links