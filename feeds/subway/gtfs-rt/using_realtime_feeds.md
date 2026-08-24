# Using MTA Realtime Feeds

## 1. Scope

This guide explains how to compile the Protocol Buffer definitions used by MTA's New York City Subway GTFS Realtime (GTFS-RT) feeds and deserialize their messages.

This guide supplements the canonical [GTFS Realtime documentation](https://developers.google.com/transit/gtfs-realtime/). Where this guide is silent, consumers should defer to the official GTFS Realtime and Protocol Buffers documentation.

## 2. GTFS Realtime and MTA Extensions

GTFS Realtime is an open feed specification used by transit agencies to publish updates such as vehicle locations and service information. Feed messages are serialized using [Protocol Buffers](https://protobuf.dev/).

To decode MTA Subway data, developers need both:

- the standard [GTFS Realtime Protocol Buffer definition](https://developers.google.com/transit/gtfs-realtime/gtfs-realtime-proto), which defines the fields shared by GTFS-RT producers; and
- the [NYCT Subway extension definition](https://api.mta.info/nyct-subway.proto.txt), which defines MTA fields added to the standard schema.

The Subway extension uses Protocol Buffers `proto2` syntax and imports `gtfs-realtime.proto`. Keep both definitions available to the Protocol Buffer compiler so the import and extension references can be resolved.

## 3. Compiling the Definitions

Use a current release of the [Protocol Buffers compiler](https://github.com/protocolbuffers/protobuf). Current releases [support `proto2`](https://protobuf.dev/support/version-support/); a 2.x compiler is not required. Generated code and its runtime library should use compatible versions.

To generate language-specific classes:

1. Download the standard definition and save it as `gtfs-realtime.proto`.
2. Download the Subway extension and save it as `nyct-subway.proto` in the same directory.
3. Run `protoc` with that directory on its include path and select the output option for the target language.

For example, the following command generates Python classes in the current directory:

```sh
protoc --proto_path=. --python_out=. gtfs-realtime.proto nyct-subway.proto
```

Use the equivalent output option for another supported language, such as `--cpp_out`, `--java_out`, or `--csharp_out`. See the official [Protocol Buffers tutorials](https://protobuf.dev/getting-started/) for installation and language-specific instructions.

### 3.1 Optional .NET Workflow

.NET applications may use the official C# Protocol Buffers runtime or a third-party implementation such as [protobuf-net](https://github.com/protobuf-net/protobuf-net).

When using the [protobuf-net online generator](https://protogen.marcgravell.com/), append the contents of the Subway extension definition to the standard GTFS Realtime definition after removing these lines from the extension:

```proto
option java_package = "com.google.transit.realtime";
import "gtfs-realtime.proto";
```

This concatenation is specific to that online workflow. A normal local compilation should retain the import and provide both files through the compiler's include path.

## 4. Parsing a Feed

After generating language-specific classes from the standard definition and the Subway extension, fetch a GTFS-RT feed as binary data and deserialize it as a `FeedMessage`.

Applications should follow the deserialization API for their selected Protocol Buffers runtime. For example, a .NET application using `protobuf-net` can deserialize a stream after adding the `protobuf-net` NuGet package and generated class file:

```csharp
FeedMessage feed = Serializer.Deserialize<FeedMessage>(gtfsFileStream);
```

Applications should also validate HTTP responses, handle malformed or unavailable feed data, and preserve unknown fields where supported so that unrecognized extensions do not prevent standard fields from being read.

## 5. Developer Notes

Links to third-party tools are provided for convenience and informational purposes. Their inclusion does not constitute MTA endorsement; developers are responsible for evaluating external tools and documentation.
