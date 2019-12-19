# Reflection Based Deserialization

Another approach to the [json_auto_decode](json_auto_decode.md) implementation
is to do it with reflection.

This can be done with either `dart:mirrors` or `package:reflectable`, the
latter being supported for all platforms but requiring a build step and the
former being limited to the Dart VM in JIT mode.

## Usability: 4 (Good)

This has great use-site usability but requires either a global build step.

## Functionality: 3 (OK)

Lacks general support for generics. While you can reflect on the type arguments
you are given you cannot forward those type arguments to a constructor call,
and thus you can't create for instance a `List<int>` at runtime.

You can sort of paper over this for a fixed set of known types, but it requires
extreme hackery and comes at the cost of additional performace degredation.

For non-generic types this a very functional API though.

## Performance: 1 (Very Poor)

Today the performance of both of these approaches is a least an order of
magnitude slower than other approaches.

This is due to how the constructors are invoked, either with `Function.apply`
or `ClassMirror.newInstance`. Most of the performance loss is attributable to
these functions and they could potentially be improved, but that is outside the
scope of this document.

It also comes with a code size overhead, although `reflectable` seems to do a
reasonable job of keeping this in check (at least compared to `dart:mirrors`).
