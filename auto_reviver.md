# Decoder + AutoReviver<T>

This approach takes some inspiration from swifts `Codeable`, and tries to fit
that into the Dart language. We don't have static interfaces which makes this
ultimately look a lot different though.

It also takes inspiration from the [json_auto_decode](json_auto_decode.md)
approach to auto-generate a reviver for a given type.

By combining these ideas we get something that solves a lot of the usability
concerns of [json_auto_decode](json_auto_decode.md) while retaining the nice
ergonomics of it.

## Approach

A core part of this approach is to separate the construction of objects from
the deserialization formats. Concretely, we create these interfaces:

```dart
abstract class Reviver<T> {
  T revive(Decoder decoder);
}

/// Base class for all decoder types, you must cast to a specific type of
/// decoder to read the decoded values.
abstract class Decoder {}

/// Note that this would likely need to be modified somehow for non-json based
/// decoders where the `raw` value isn't sufficient. This might mean adding
/// specific `decodeInt` methods etc.
abstract class ValueDecoder extends Decoder {
  /// The raw object currently being decoded.
  Object get raw;

  T decode<T>(Reviver<T> reviver) => reviver.revive(this);
}

/// Provides keyed access to data.
abstract class KeyedDecoder<K> extends Decoder {
  Iterable<K> get keys;

  T decodeKey<T>(K key, [Reviver<T> reviver]);
}

/// Provides indexed access to data.
abstract class IndexedDecoder extends Decoder {
  int get length;

  T decodeAt<T>(int index, [Reviver<T> reviver]);
}
```

## Example Reviver Implementation:

These look very similar to existing `fromJson` methods, except they are going
through the `KeyedDecoder` interface instead of a `Map`. This is important for
performance reasons, as it allows you to skip creating the underlying `Map`.

```dart
class UserReviver implements Reviver<User> {
  @override
  User revive(Decoder decoder) {
    var keyed = decoder as KeyedDecoder<String>;
    return User(
      name: keyed.decodeKey('name'),
      age: keyed.decodeKey('age'),
    );
  }
}
```

## Example Decoder Implementation:

Below is an example JSON Decoder implementation, which works off of parsed json
objects from a normal `jsonDecode` call.

**Note:** It is also trivial to make one of these that is based on JS interop
with native JS objects which is significantly faster than todays `jsonDecode`.

```dart
import 'dart:convert' show jsonDecode;
import 'package:codeable/codeable.dart';

class JsonDecoder extends Decoder
    implements KeyedDecoder<String>, IndexedDecoder, ValueDecoder {
  /// Map<String, dynamic>|List<dynamic>|String|num|bool|Null
  final dynamic raw;

  JsonDecoder(this.raw);

  @override
  T decodeKey<T>(String key, [Reviver<T> reviver]) {
    if (reviver == null) return raw[key] as T;
    return reviver.revive(JsonDecoder(raw[key]));
  }

  @override
  Iterable<String> get keys => raw.keys;

  @override
  T decodeAt<T>(int index, [Reviver<T> reviver]) {
    if (reviver == null) return raw[index] as T;
    return reviver.revive(JsonDecoder(raw[index]));
  }

  @override
  int get length => raw.length;
}

/// Helper function for easy json parsing.
T jsonDecode<T>(String json, Reviver<T> reviver) =>
    reviver.revive(JsonDecoder(jsonDecode(json)));
```

## Example AutoReviver Usage

Today you could use codegen to generate a reviver for an arbitrarily nested
structure.

This doesn't require special annotations on the class and the boilerplate
is 1:1 with the number of revivers you want to create, but each one can be
arbitrarily complex:

```dart
part 'my_file.g.dart';

@AutoReviver(/*config*/)
final Reviver<Map<String, User>> usermapReviver = $_userMapReviver;

main() {
  /// **Note:** This is the custom Helper function above not the
  /// jsonDecode from dart:convert.
  var myUserMap = jsonDecode(someJson, userMapReviver);
}
```

## Usability: 4 (Good) - Potential for 5 (Great)

This is nearly as magic as `jsonAutoDecode` - but it requires a build step.

However it also has a lot less magic actually happening, and its harder to use
wrong.

It also requires a lot less boilerplate than existing codegen solutions since
you don't have to annotate every class. 

We could potentially add some static metaprogramming to the language in order
to eliminate the (user visible) build step, and get this to a 5. 

## Functionality: 5 (Great)

This is a very functional API, supporting arbitrarily deep nesting of objects
with full support for generics and collections.

## Performance: 5 (Great)

This feature gives a lot of immediate and potential future performance
improvements compared to existing solutions.

It also enables custom user-created `Decoder` implementations which can easily
beat the existing `jsonDecode` performance on the web.

- It can easily support lazy collections such as `Iterable` which can do lazy
  decoding, as well as optional `LazyMap` and `LazyList` implementations which
  lazily decode on access.
- The `Decoder` interface can be implemented much more cheaply on the web than
  the `Map` interface, using native JS objects and JS interop calls.
