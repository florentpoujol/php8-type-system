

- Main PHP website: https://www.php.net/
- PHP manual and function reference: https://www.php.net/manual/en/ (also fully available in several other languages)
- Language specification about types: https://github.com/php/php-langspec/blob/be010b4435e7b0801737bb66b5bbdd8f9fb51dde/spec/05-types.md


## Relevant pages in the manual

- Types table of content : https://www.php.net/manual/en/language.types.php
- Covariance and contravariance: https://www.php.net/manual/en/language.oop5.variance.php
- Comparison: https://www.php.net/manual/en/language.operators.comparison.php
- Type comparison table: https://www.php.net/manual/en/types.comparisons.php


## RFCs

RFC are Request For Comments, documents that needs to be written by PHP core developers to propose a non-trivial change to PHP.

They explain why the new features is needed, give examples, and all pertinent information.

Here are the types-related RFCs:

### PHP5.4 (released march 2012)

- Callable typehint: https://wiki.php.net/rfc/callable

### PHP7.0 (december 2015)

- Scalar typehints and strict types: https://wiki.php.net/rfc/scalar_type_hints_v5
- Return type: https://wiki.php.net/rfc/return_types
- Nullcoalesce operator: https://wiki.php.net/rfc/isset_ternary
- Reserve more types in PHP7: https://wiki.php.net/rfc/reserve_more_types_in_php_7

### PHP7.1 (december 2016)

- Nullable typehint: https://wiki.php.net/rfc/nullable_types
- Void typehint: https://wiki.php.net/rfc/void_return_type

### PHP7.2 (november 2017)

- Object typehint: https://wiki.php.net/rfc/object-typehint
- Parameter type widening: https://wiki.php.net/rfc/parameter-no-type-variance

### PHP7.4 (november 2019)

- Typed property: https://wiki.php.net/rfc/typed_properties_v2
- Null coalescing assignment Operator: https://wiki.php.net/rfc/null_coalesce_equal_operator

### PHP8.0 (november 2020)

- Mixed typehint: https://wiki.php.net/rfc/mixed_type_v2
- Enum: https://wiki.php.net/rfc/enumerations
- Union types: https://wiki.php.net/rfc/union_types_v2
- Static return typehint: https://wiki.php.net/rfc/static_return_type
- Nullsafe operator: https://wiki.php.net/rfc/nullsafe_operator
- Stringable interface: https://wiki.php.net/rfc/stringable
- Saner numeric strings: https://wiki.php.net/rfc/saner-numeric-strings
- Saner string to number comparison: https://wiki.php.net/rfc/string_to_number_comparison

### PHP8.1 (december 2021)

- Pure Intersection types: https://wiki.php.net/rfc/pure-intersection-types
- Never typehint: https://wiki.php.net/rfc/noreturn_type


## Articles

- PHPStan PHPDocs reference: https://phpstan.org/writing-php-code/phpdocs-basics  
- Psalm PHPDocs reference: https://psalm.dev/docs/annotating_code/type_syntax/atomic_types/

### Generics

- [Generics in PHP using PHPDocs, in PHPStan's blog (from 2019)](https://phpstan.org/blog/generics-in-php-using-phpdocs) 
- [Template annotation, in Psalm documentation](https://psalm.dev/docs/annotating_code/templated_annotations) 
- https://wiki.php.net/rfc/generics
- https://wiki.php.net/rfc/generic-arrays
- https://externals.io/message/111875 A message from Brent Roose to the PHP core developper advocating for runtime-erased generics
- https://www.reddit.com/r/PHP/comments/iuhtgd/ive_proposed_an_approach_to_generics_on_internals/ The same original message as above but on Reddit
- https://github.com/PHPGenerics/php-generics-rfc/issues/45 Two comments by Nikita Popov
- [Brent Roose Youtube video series about Generics](https://www.youtube.com/watch?v=c8hQ1fWU_mQ&list=PL0bgkxUS9EaKyOugEDffRzsvupBE2YEoD)

### Stitcher.io blog

Brent Roose is regularly talking about PHP's type system, static analysis and advocating for a stricter PHP.

- 06/11/2021 https://stitcher.io/blog/generics-in-php-video
- 26/07/2021 https://stitcher.io/blog/we-dont-need-runtime-type-checks
- 17/11/2021 https://stitcher.io/blog/php-8-nullsafe-operator
- 17/09/2020 https://stitcher.io/blog/the-case-for-transpiled-generics
- 09/06/2021 https://stitcher.io/blog/type-system-in-php-survey-results
- 03/06/2021 https://stitcher.io/blog/typed-properties-in-php-74
- 19/05/2019 https://stitcher.io/blog/liskov-and-type-safety
- 17/05/2017 https://stitcher.io/blog/php-generics-and-why-we-need-them

### Other

- https://thephp.website/en/issue/php-type-system/
- https://en.wikipedia.org/wiki/Type_system
- https://en.wikipedia.org/wiki/Strong_and_weak_typing
- https://en.wikipedia.org/wiki/Type_safety
