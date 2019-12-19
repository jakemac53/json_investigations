# jsonAutoDecode<T>(String json, {Map<Type, Config> config})

This approach exposes a new api which is basically just "magic". It can give
you an instance of any type `T` from a json string, including nested generic
types.

The `config` parameter gives you a way of customizing which factory functions
to invoke for each type (it uses the unnamed constructor by default), and other
configuration such as field name mapping etc.

## Example Usage:

```dart
var user = jsonAutoDecode<MyCustomClass<WithAGenericType>>(someJson);
var deeplyTypedCollection = jsonAutoDecode<Map<String, List<int>>>(someJson);
```

## Usability: 4 (Good)

This function feels very magic and is very intuitive to use. It does not
require any annotations on the class, and has _zero_ boilerplate.

The "magic" has its own downsides though, due to how it is implemented. It
relies on compile time code generation to replace the static invocation with a
new implementation. This means it is illegal to invoke the method with a type
paramter (a `T`) - you must give it an explicit type argument.

The result is you cannot wrap this function call in another generic method and
forward the generic type to it, which reduces the usability and could cause
confusion.

This api also isn't generalizable to other serialization formats. It is a
weirdly specialized function just for JSON.

## Functionality: 4 (Good)

While this function works great for decoding, and it supports arbitrarily deep
nested generics it is really only appropriate for _decoding_ objects and not
_encoding_ objects.

This creates a weird hole in functionality that is hard to explain to users,
and it won't solve all use cases that existing codegen solves.

## Performance: 5 (Great)

This feature gives a lot of immediate and potential future performance
improvements compared to existing solutions.

- It can easily support lazy collections such as `Iterable` which can do lazy
  decoding. As well as optional `LazyMap` and `LazyList` implementations which
  lazily decode on access.
- It hides from the user any intermediate objects, allowing backends to avoid
  creating intermediate maps/lists, so they can go directly to the desired
  instances from the parsed JSON.
  - Especially in dart2js this can has major benefits.
- Provides information to the parsers ahead of time about exactly what fields
  are provided and the types for those fields. This allows for more efficient
  parsing, and skipping of unneeded fields.


## Prototype

https://github.com/jakemac53/json_auto_decode
