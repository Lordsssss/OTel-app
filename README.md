# OpenTelemetry C++ SDK Setup Guide (OTLP + OStream, Bazel Only)

This guide walks you through how to build and install the OpenTelemetry C++ SDK using Bazel,
with support for:

- ✅ OTLP/gRPC Exporter
- ✅ OStream Exporter

---

## ✅ Prerequisites

Install required system dependencies:

```bash
sudo apt update && sudo apt install -y \
  git build-essential cmake \
  libcurl4-openssl-dev \
  libprotobuf-dev protobuf-compiler \
  libssl-dev zlib1g-dev \
  pkg-config \
  libgrpc++-dev grpc-proto
```

You also need:

- `grpc_cpp_plugin` installed and available at `/usr/bin/grpc_cpp_plugin`:
  ```bash
  which grpc_cpp_plugin
  ```

---

## 📦 Clone the SDK with Submodules

```bash
git clone --recurse-submodules --depth 1 https://github.com/open-telemetry/opentelemetry-cpp.git
cd opentelemetry-cpp
```

---

## 🧪 Run an Example Using Bazel (OStream Exporter)

Bazel examples are already configured in the repo.

To run a working span example:

```bash
bazel run //examples/simple:example_simple
```

To build it manually:

```bash
bazel build //examples/simple:example_simple
./bazel-bin/examples/simple/example_simple
```

You should see span data printed to the terminal using the `OStreamSpanExporter`.

---

## 🧱 Using OpenTelemetry in Your Own Bazel Project

### 1. Example Project Structure

```
my_otel_app/
├── WORKSPACE
├── BUILD.bazel
└── src/
    ├── BUILD.bazel
    └── main.cc
```

### 2. src/main.cc

```cpp
#include <memory>
#include "opentelemetry/exporters/ostream/span_exporter.h"
#include "opentelemetry/sdk/trace/simple_processor.h"
#include "opentelemetry/sdk/trace/tracer_provider.h"
#include "opentelemetry/trace/provider.h"

namespace trace = opentelemetry::trace;
namespace sdktrace = opentelemetry::sdk::trace;

int main()
{
    auto exporter = std::unique_ptr<sdktrace::SpanExporter>(
        new opentelemetry::exporter::trace::OStreamSpanExporter);
    auto processor = std::unique_ptr<sdktrace::SpanProcessor>(
        new sdktrace::SimpleSpanProcessor(std::move(exporter)));
    auto provider = std::make_shared<sdktrace::TracerProvider>(std::move(processor));
    trace::Provider::SetTracerProvider(provider);

    auto tracer = trace::Provider::GetTracer("example");
    auto span = tracer->StartSpan("bazel-span");
    span->End();

    return 0;
}
```

### 3. src/BUILD.bazel

```python
cc_binary(
    name = "main",
    srcs = ["main.cc"],
    copts = ["-std=c++14", "-I/usr/local/include"],
    linkopts = [
        "-L/usr/local/lib",
        "-lopentelemetry_trace",
        "-lopentelemetry_exporter_ostream_span",
        "-lopentelemetry_common",
        "-lopentelemetry_resources",
        "-lcurl",
        "-lgrpc++",
        "-lprotobuf"
    ],
)
```

---

## 🔍 Notes

- This setup assumes OpenTelemetry C++ SDK is installed globally via CMake.
- If you want to vendor the SDK inside your repo, you can copy necessary Bazel targets from `opentelemetry-cpp` repo.

---

## 🧼 Cleanup

If needed:

```bash
sudo rm -rf /usr/local/include/opentelemetry
sudo rm -f /usr/local/lib/libopentelemetry_*
```

---

Happy tracing! 🎯
