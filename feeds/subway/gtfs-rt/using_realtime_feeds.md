# Using MTA Realtime Feeds

## 1. Scope

This guide summarizes the MTA Developer Resources [Help Document](https://api.mta.info/#/HelpDocument) for developers getting started with MTA GTFS Realtime (GTFS-RT) feeds, with a focus on New York City Subway data.

The Help Document supplements the canonical [GTFS Realtime documentation](https://developers.google.com/transit/gtfs-realtime/) and points developers to the schemas and tools needed to decode a feed. It does not replace the GTFS Realtime specification.

## 2. GTFS Realtime and MTA Extensions

GTFS Realtime is an open feed specification used by transit agencies to publish updates such as vehicle locations and service information. Feed messages are serialized using [Protocol Buffers](https://protobuf.dev/).

To decode MTA Subway data, developers need both:

- the standard [GTFS Realtime Protocol Buffer definition](https://developers.google.com/transit/gtfs-realtime/gtfs-realtime-proto), which defines the fields shared by GTFS-RT producers; and
- the [NYCT Subway extension definition](https://api.mta.info/nyct-subway.proto.txt), which defines MTA fields added to the standard schema.

The Subway extension uses Protocol Buffers `proto2` syntax and imports `gtfs-realtime.proto`. Keep both definitions available to the Protocol Buffer compiler so the import and extension references can be resolved.

## 3. Compiling the Definitions

The Help Document links to the [Protocol Buffers compiler](https://github.com/protocolbuffers/protobuf) and recommends a 2.x compiler for its `proto2` workflow. It specifically points to version 2.7 examples for C++, Java, and Python.

Those instructions describe the legacy toolchain used when the Help Document was written. Current Protocol Buffers releases [continue to support `proto2`](https://protobuf.dev/support/version-support/), so developers may use a current compiler and runtime when they are compatible with the MTA and standard definitions. Generated code and runtime libraries should use compatible versions.

The Help Document also identifies third-party .NET options:

- [protobuf-net](https://github.com/protobuf-net/protobuf-net); and
- the [protobuf-net online generator](https://protogen.marcgravell.com/).

For the linked online generator, the Help Document instructs developers to append the contents of the NYCT Subway extension definition to the standard GTFS Realtime definition after removing these lines from the extension:

```proto
option java_package = "com.google.transit.realtime";
import "gtfs-realtime.proto";
```

This concatenation is specific to that online workflow. A normal local compilation should retain the import and provide both files through the compiler's include path.

## 4. Parsing a Feed

After generating language-specific classes from the standard definition and the Subway extension, fetch a GTFS-RT feed as binary data and deserialize it as a `FeedMessage`.

The Help Document links to the Protocol Buffers 2.7 [C++, Java, and Python examples](https://github.com/protocolbuffers/protobuf/tree/2.7.0/examples). These examples demonstrate the compile-and-parse pattern, but applications should follow the API of the Protocol Buffers version and language runtime they actually use.

For .NET with `protobuf-net`, the Help Document gives this deserialization pattern after adding the `protobuf-net` NuGet package and generated class file:

```csharp
FeedMessage feed = Serializer.Deserialize<FeedMessage>(gtfsFileStream);
```

Applications should also validate HTTP responses, handle malformed or unavailable feed data, and preserve unknown fields where supported so that unrecognized extensions do not prevent standard fields from being read.

## 5. Developer Notes

The linked MTA Help Document contains legacy compiler guidance. Treat its version-specific recommendations as historical implementation advice, while treating the linked GTFS Realtime specification and Protocol Buffer project documentation as the current authority for supported tooling.

Links to third-party tools are provided for convenience and informational purposes. Their inclusion does not constitute MTA endorsement; developers are responsible for evaluating external tools and documentation.
