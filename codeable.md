# Codeable

The central question for this exploration was figuring out what would be needed
to fully implement something similar to the swift `Codeable` feature.

## Static interfaces

In order to declare something as `Decodeable`, you have to declare that it has
a constructor matching a certain pattern. Specifically, a constructor with a
certain name and signature (it would take a `Decoder`).

This allows you to declare that a generic type implements a static interface,
and you can then call those static methods/constructors on that generic type
argument.

This could look something like the following, although the particular syntax
would be up for debate:

```dart
/// Declares a static interface that requires a `decode` constructor which
/// takes a single argument of type `Decoder`.
abstract class Decodeable {
  // Self is a special placeholder for the current class.
  self.decode(Decoder);
} 

/// The actual `Decoder` interface. 
class Decoder {
  /// `T` must implement the static `Decodeable` interface. 
  T decodeField<T static implements Decodeable>(String fieldName) =>
      /// Static interfaces are now callable on `T`
      T.decode(/* create decoder for $fieldName */);
}

class User static implements Decodeable {
  final String name;
  // pretend this is also decodeable
  final Preferences preferences;

  User.decode(Decoder decoder)
    : /// Note that the generics are inferred here
      name = decoder.decodeField('name'),
      preferences = decoder.decodeField('preferences');
}
```

## Automagic `decode` constructors by mixing in `Decodeable`

```dart
/// Magic `decode` constructor since we mixed in the static interface instead
/// of implementing it!
class User static with Decodeable {
  final String name;
  // pretend this is also decodeable
  final Preferences preferences;
}
```

This magic constructor could either be specific to the `Decodeable` interface
and specified as a part of the language, or it could be generated using some
metaprogramming feature.

## Encoding

This needs a bit more fleshing out - specifically the generics make this more
complicated.

TBD

## Usability: 5 (Great)

Minimal boilerplate with easily overrideable behavior by implementing
`Decodeable` instead of mixing it in.

Expressed in terms of a combination of existing concepts.

## Functionality: 5 (Great)

Supports round tripping objects (encode/decode) since we can ensure the
constructor actually matches the fields.

Fully supports arbitrarily nested generics and collections.

Not specific to a given encoding for objects, fully generalizable.

Easy to provide platform specific Decoder implementations.

## Performance: 5 (Great)

This feature gives a lot of immediate and potential future performance
improvements compared to existing solutions.

It also enables custom user-created `Decoder` implementations which can easily
beat the existing `jsonDecode` performance on the web.

This provides around a 30% performance boost in dart2js when using a custom
decoder implementation.
