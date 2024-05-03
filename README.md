# Serialize Dart Enums without Source Code Generation
[![Dart](https://github.com/simphotonics/serialize_enum/actions/workflows/dart.yml/badge.svg)](https://github.com/simphotonics/serialize_enum/actions/workflows/dart.yml)


## Introduction

Persisting objects in Dart and Flutter typically consists in
transforming the object into a `Map<String, dynamic>` using a
method called `toJson()`, converting the map into a `String`
using the function [`jsonEncode`][jsonEncode], and
storing the resulting Sting in a file or database.

To revive the object, the stored string is retrieved,
converted back into a `Map<String, dynamic>` using the function
[`jsonDecode`][jsonDecode], and a clone of the original object is created using
 a factory constructor usually named `.fromJson`.

## Motivation
Writing the method `toJson()` and the factory constructor `.fromJson`
for a *large* number of data classes can be tedious and error prone
and this task is often acomplished by annotating the
data classes and building the source code using packages like:
[`json_serializable`][json_serializable].

For small projects, however, source code generation might add too much
complexity. Dart enumerations often represent settings or options
that need to be persisted after exiting an app or program.
The package [`serialize_enum`][serialize_enum]
provides mixins to serialize Dart enums without source code
generation. All that is needed is a `with` statement when declaring the
enum and defining a `factory` constructor that calls a static method provided by
the mixin.

## Usage

Include [`serialize_enum`][serialize_enum] as a `dev_dependency`
in your `pubspec.yaml` file.

The example below shows the enum `AlphabeticOrder`. The generic mixin
[SerializeByName][SerializeByName] provides the method `toJson`.
The enum factory constructor
calls the static method `SerializeByName.fromJson` provided by the mixin:

```Dart
import 'package:serialize_enum/serialize_enum.dart';

enum AlphabeticOrder with SerializeByName<AlphabeticOrder> {
  asc,
  desc;

  /// Reads a json map and returns the corresponding
  /// instance of `AlphabeticOrder`.
  factory AlphabeticOrder.fromJson(Map<String, dynamic> json) =>
      SerializeByName.fromJson(json: json, values: values);
}
```

Note: The generic type parameter of [SerializeByName][SerializeByName]
must be specified. It is used to generate the json map
*key* under which the enum *name* is stored.

```Dart
// Code shown above goes here ...
void main() {
  const order = AlphabeticOrder.asc;

  print('Json map:');
  print(order.toJson());

  print('\nRevived enum:');
  print(AlphabeticOrder.fromJson(order.toJson()));
}
```
Running the program produces the following console output:
```Console
$ dart alphabetic_order_example.dart
Json map:
{alphabeticOrder: asc}

Revived enum:
AlphabeticOrder.asc
```

## Further Serialization Options

To serialize the enum instance by storing its *index* instead of its *name*
use the mixin [`SerializeByIndex`][SerializeByIndex].

In order to use a *custom* key when serializing an enumeration, [`SerializeByIndex`][SerializeByIndex] or [`SerializeByName`][SerializeByName] may be implemented:

```Dart

import 'package:serialize_enum/serialize_enum.dart';

enum AlphabeticOrder implements SerializableByName {
  asc,
  desc;

  /// Key used to serialize the enum.
  static const key = 'customKey';

  @override
  Map<String, dynamic> toJson() => {key: name};

  /// instance of `AlphabeticOrder`.
  factory AlphabeticOrder.fromJson(Map<String, dynamic> json) =>
      SerializableByNameCustomKey.fromJson(
        json: json,
        values: values,
        key: key,
      );
}
```
For benchmark scores see folder [`benchmark`][benchmark].

## Features and bugs

Please file feature requests and bugs at the [issue tracker][tracker].

[tracker]: https://github.com/simphotonics/serialize_enum/issues

[benchmark]: https://github.com/simphotonics/serialize_enum/tree/main/benchmark

[jsonEncode]: https://api.dart.dev/dart-convert/jsonEncode.html

[jsonDecode]: https://api.dart.dev/dart-convert/jsonDecode.html

[json_serializable]: https://pub.dev/packages/json_serializable

[serialize_enum]: https://pub.dev/packages/serialize_enum

[SerializableByIndex]: https://pub.dev/documentation/serialize_enum/latest/serialize_enum/SerializableByIndex-class.html

[SerializableByName]: https://pub.dev/documentation/serialize_enum/latest/serialize_enum/SerializableByName-class.html

[SerializeByIndex]: https://pub.dev/documentation/serialize_enum/latest/serialize_enum/SerializeByIndex-mixin.html

[SerializeByName]: https://pub.dev/documentation/serialize_enum/latest/serialize_enum/SerializeByName-mixin.html