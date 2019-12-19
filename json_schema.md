# Json Schema

This idea draws heavily from the [cast](https://github.com/leafpetersen/cast)
package prototype, but brings support for that type of schema directly to the
`jsonDecode` function.

This has performance and usability benefits because collections can be created
with the desired type intially, eliminating the need for deep casting.

It also enables custom types to be deserialized directly as the json object is
parsed, eliminating the need for intermediate Maps/Lists.

## Approach

This approach requires re-writing the `jsonDecode` method entirely into a
visitor-like pattern.

I started with the [json_alt](https://github.com/kevmoo/json_alt) experimental
api for this. Essentially, the api becomes `jsonDecode(String, JsonReader)`,
where the `JsonReader` interface is as follows:

```dart
abstract class JsonReader<T> {
  void handleString(String value);

  void handleNumber(num value);

  void handleBool(bool value);

  void handleNull();

  void objectStart();

  void propertyName();

  void propertyValue();

  void objectEnd();

  void arrayStart();

  void arrayElement();

  void arrayEnd();

  T get result;
}
```

The `Schema` class itself ultimately provides implementation of this interface.

## Usage

```dart
/// `createJsonReader` gives you a reader from a `Schema`.
final mySchemaReader = createJsonReader(Schema({
  "listOfInt": Json.array<int>(),
  "jsObjectofBool:": Json.object<bool>(),
  "string": Json.string
}));

main() {
  var parsed = jsonDecode(myJson, mySchemaReader);
  // You can now directly cast the collection objects, without calling
  // `.cast()`
  var listOfInt = parsed["listOfInt"] as List<int>;
}
```

For custom classes you can implement a `CustomObjectReader` which are nestable
within the `Schema`.

## Usability: 3 (OK)

This approach unfortunately doesn't really remove any boilerplate other than
the deep casting required for collections. You can codegen the custom object
reader implementations, but that requires a build step.

You have to fully specify the schema for every object you want to parse.

## Functionality: 5 (Great)

This is a very functional API, supporting arbitrarily deep nesting of objects
with full support for generics and collections.

It also allows deep casting per-key for Map objects (or per-index for lists)
while most other solutions would essentially require you to create an actual
class with typed fields to get that level of per-entry type safety for
collections.

## Performance: 5* (Great)

Even with a custom JSON parser this outperforms existing solutions on the VM
by eliminating the need to do deep casting of objects and create intermediate
maps and lists.

**Note**: As implemented in my prototype this has poor performance on the web
due to the custom parser implementation being a lot slower than the native
JSON.parse method. This could almost certainly be re-implemented for the web
to use the native JSON.parse and get that performance back, but I haven't
confirmed.
