# Background

Json deserialization in Dart is cumbersome and requires a lot of boilerplate,
especially when compared to some other languages.

Most of this boilerplate comes from a few places:

- Duplication of all field names (typically 3-4x)
  - The original field declaration
  - The constructor declaration
  - The map key access when passing arguments to the constructor
  - Named parameters in the constructor invocation (the optional 4th)
- Repeated references to the Map object O(fields)
  - `User(name: json['name'], age: json['age'})`
- Nested casting
  - `json['foo'] as List).map((entry) => (entry as Map).cast<String, int>`

## Current Approaches

The current commonly used solutions to this problem are all codegen based. I
won't go in depth into these approaches here, but some of the common ones are:

- [json_serializable](https://pub.dev/packages/json_serializable)
- [built_value](https://pub.dev/packages/built_value)

### Pros

- Boilerplate scales with the number of classes you want to serialize and not
  the number of fields.
- Lots of features.
- Developed as open source packages and not coupled with the SDK.

### Cons  

- Still a fair amount of boilerplate (depends on the solution)
  - Required references to generated symbols in users Dart code.
  - Analyzer shows errors until after codegen is done.
  - **Note**: Extension methods and/or
    [partial classes](https://github.com/dart-lang/language/pull/680) could
    eliminate some of the boilerplate, if we wanted to redesign these packages.
    I will talk more about these later.
- Required codegen step which slows down development and is another point of
  friction for users.
  - The code generators also get somewhat frequently broken in practice due
    to the analyzer dependency and tight coupling with the language itself.
  - Due to a lack of supported compiler hooks this process is inefficient, as
    it lives completely on the side of the compiler in its own process,
    effectively duplicating all the work required as a part of the development
    workflow. 

## Explorations

I have explored a variety of potential solutions to this problem, both working
within the existing language as well as extending the language itself, as well
as magic compiler hacks.

I have evaluated each of these approaches in terms of 3 primary criteria:
**usability**, **functionality**, and **performance**. These are evaluated on
a 5 point scale.

The experiments are not listed in any particular order.

### jsonAutoDecode<T>(String json, {Map<Type, Config> config})

Scores: Usability 4, Functionality 4, Performance 5

See [details](json_auto_decode.md).

### Decoder + Reviver + AutoReviver<T>

Scores: Usability 4*, Functionality 5, Performance 5

**Note**: The useability of this approach could be improved to potentially a 5
with language changes. 

See [details](auto_reviver.md).

### Reflectable/Mirrors

Scores: Usability 4, Functionality 3, Performance 1*

**Note**: The performance is heavily impacted by
`Function.apply`/`ClassMirror.newInstance` performance. These could potentially be
improved.

See [details](reflection.md).

### Json Schema

Scores: Usability 3*, Functionality 5, Performance 4

See [details](json_schema.md).

### Codeable

Scores: Usability 5, Functionality 5, Performance 5

**Note:** Requires language changes

See [details](codeable.md)

### Const Functions for Reflection

This is more of an implementation idea which could implement several of the
above proposals. It is a form of meta-programming based on const functions
which can reflect on the program arbitrarily with no direct cost to code size.

See [const_functions](const_functions.md)
