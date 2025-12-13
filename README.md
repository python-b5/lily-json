# lily-json

lily-json is a JSON parser and serializer for Lily. It aims to be simple and
compliant with the JSON specification.

## Features

- Compliant with the
  [RFC 8259](https://datatracker.ietf.org/doc/html/rfc8259.html) JSON
  specification, including full support for UTF-16 surrogate pairs.

- Errors in JSON data are displayed with visual context, in the same format Lily
  uses for its own error messages.

- Optional auto-formatting of serialized output.

- Supports common extensions to the JSON syntax:

    - JavaScript-style comments (`// ...` and `/* ... */`).

    - Trailing commas (`{"a": 1, "b": 2, "c": 3,}`, `[1, 2, 3,]`).

    - Infinite and NaN values (`Infinity`, `-Infinity`, `NaN`).

- Easy to use, with an idiomatic API.

## Compliance

lily-json passes most tests in the
[JSON Parsing Test Suite](https://github.com/nst/JSONTestSuite). The exceptions
involve JSON data that is not valid UTF-8, and escape sequences that produce
characters that are not valid UTF-8. They occur because Lily's `String`s are
required to only contain valid UTF-8 data - so, in the first case the data
cannot be read in the first place, and in the second case Lily fails to encode
the final `String` after parsing the JSON token. The only way around this would
be to use `ByteString`s instead of `String`s, but that is not especially
idiomatic in Lily.

## Limitations

Performance is sufficient for most use cases, but was not the main focus. Very
large files may cause some slowdown. Performance could be drastically improved
by rewriting lily-json as a C extension, but it was written in native Lily
intentionally to allow it to be used in sandbox mode (which blocks importing
foreign libraries). If you need to parse extremely large JSON files, a native
Lily library is simply not the right tool for the job - in that case, you may
want to look into writing Lily bindings for a performant C JSON parser.

lily-json requires Lily 2.2 or higher, due to the use of variant shorthand to
improve code readability.
[v1.0.0](https://github.com/python-b5/lily-json/releases/tag/v1.0.0) supports
Lily 2.1, if support for that is needed. Versions older than 2.1 are not
supported by any release, though it probably wouldn't be too difficult to patch
the module if you're stuck on an old Lily version for whatever reason.

## Comparison to tinyjson

The only other JSON library currently available for Lily (and the inspiration
for this one!) is [tinyjson](https://github.com/Zhorander/tinyjson), which has a
similar API to lily-json. For many JSON files, both libraries would be perfectly
sufficient. However, lily-json improves on tinyjson in a few ways:

- lily-json aims to be fully compliant with the JSON specification, while
  tinyjson differs from it regarding some specifics. The main examples are with
  strings and numbers - the specification describes specific rules for how those
  should be treated, which tinyjson does not follow. For instance, escape
  sequences for strings and exponents for numbers are not implemented in
  tinyjson.

- lily-json optionally supports several extensions to the JSON syntax, while
  tinyjson does not. lily-json also allows disabling all these extensions, if
  strictly adhering to the specification is desired.

- tinyjson's parser is implemented recursively, meaning the maximum depth of
  JSON data it can parse is restricted by Lily's recursion limit. lily-json
  parses using a stack instead, so it has no such restriction.

- lily-json is significantly faster than tinyjson (partially because it is
  iterative instead of recursive, though there are other factors as well). For
  many JSON files, this probably doesn't matter, but for large files it can make
  a big difference.

    - As an example: while this is far from an exhaustive benchmark,
      [this](https://microsoftedge.github.io/Demos/json-dummy-data/5MB.json) 5
      MB sample JSON file parses on my machine in ~3 seconds with tinyjson, and
      in ~0.6 seconds with lily-json.

- lily-json's API, while similar to tinyjson's, is slightly more idiomatic; the
  main difference being that lily-json defines operations on JSON values as
  methods on the `Value` enum, while tinyjson defines them as separate functions
  that take an otherwise identical `Json` enum. lily-json also formats symbol
  names in `snake_case` (like in the Lily core modules), rather than tinyjson's
  `camelCase`. Other than those differences, though, the two APIs are extremely
  similar, and so porting from one library to the other would be quite easy.

## Documentation

The API has been kept as minimal as possible. Documentation for it (generated
with [docgen](https://lily-lang.org/tooling-docgen.html), using the `--no-sort`
option) can be found [here](https://python-b5.com/doc/json/module.json.html).

## Licensing

lily-json has been made available under the zlib/libpng license. See
`LICENSE.md` for details.
