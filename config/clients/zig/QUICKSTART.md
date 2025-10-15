# Zig SDK Quick Start Guide

This guide will help you get started with generating and building the Zig SDK for OpenFGA.

## Prerequisites

- Make
- Docker (for OpenAPI Generator)
- Zig 0.11.0 or later (for building the generated SDK)
- libcurl development headers
- Git

## Quick Generation

Generate the Zig SDK with one command:

```bash
make build-client-zig
```

This will:
1. Download the OpenFGA OpenAPI specification
2. Transform the spec (remove streaming endpoints, rename Object to FgaObject)
3. Generate C client library using OpenAPI Generator
4. Add Zig build system files (build.zig, build.zig.zon)

The generated SDK will be in `clients/fga-zig-sdk/`.

## What Gets Generated

```
clients/fga-zig-sdk/
├── build.zig              # Zig build configuration
├── build.zig.zon          # Zig package manifest
├── CMakeLists.txt         # C build configuration
├── README.md              # Generated documentation
├── api/
│   └── OpenFgaAPI.{c,h}  # API implementation
├── model/                 # All model types
├── src/                   # C source files
├── include/               # C headers
└── external/
    └── cJSON.{c,h}        # JSON library
```

## Building the Generated SDK

### Option 1: Using Zig (Requires Wrapper Implementation)

```bash
cd clients/fga-zig-sdk
zig build
```

> **Note**: The generated `build.zig` is a template. You'll need to add all model source files and implement Zig wrapper code. See IMPLEMENTATION_GUIDE.md for details.

### Option 2: Using CMake (C Library Only)

```bash
cd clients/fga-zig-sdk
mkdir build && cd build
cmake ..
make
```

This builds the C library that can then be wrapped with Zig code.

## Testing the Build

After generating the SDK, you can test that the C code compiles:

```bash
cd clients/fga-zig-sdk
make build-client-zig
```

Or run the test target:

```bash
make test-client-zig
```

> **Note**: The test target currently just displays a message since Zig wrapper tests need to be implemented.

## Next Steps

The generated code is a C library. To make it usable from Zig:

1. **Review the generated code**:
   ```bash
   ls clients/fga-zig-sdk/
   less clients/fga-zig-sdk/README.md
   ```

2. **Read the implementation guide**:
   ```bash
   less config/clients/zig/IMPLEMENTATION_GUIDE.md
   ```

3. **Create Zig wrapper structure**:
   ```bash
   cd clients/fga-zig-sdk
   mkdir zig
   touch zig/main.zig zig/client.zig zig/config.zig
   ```

4. **Update build.zig** to include all generated C files and add Zig module

5. **Implement Zig bindings** using `@cImport` and `@cInclude`

6. **Write tests** in `test/` directory

## Example: Simple Zig Wrapper

Here's a minimal example to get started:

```zig
// zig/main.zig
const std = @import("std");

pub const c = @cImport({
    @cInclude("apiClient.h");
    @cInclude("OpenFgaAPI.h");
});

pub const Client = struct {
    c_client: *c.apiClient_t,
    
    pub fn init() !Client {
        const client = c.apiClient_create() orelse 
            return error.ClientCreationFailed;
        
        return Client{ .c_client = client };
    }
    
    pub fn deinit(self: *Client) void {
        c.apiClient_free(self.c_client);
    }
};

test "basic client creation" {
    var client = try Client.init();
    defer client.deinit();
}
```

## Regenerating the SDK

If the OpenAPI spec changes, regenerate with:

```bash
make build-client-zig
```

This will clean the output directory and regenerate all files.

## Troubleshooting

### Issue: "libcurl not found"

Install libcurl development headers:
```bash
# Ubuntu/Debian
sudo apt-get install libcurl4-openssl-dev

# macOS
brew install curl

# Fedora/RHEL
sudo dnf install libcurl-devel
```

### Issue: "Zig build fails with missing C files"

The generated `build.zig` includes a template list of C files. You need to update it with all generated model files:

```bash
cd clients/fga-zig-sdk
ls model/*.c | sed 's/^/"/' | sed 's/$/"/' | tr '\n' ',' > model-files.txt
# Add contents to build.zig
```

### Issue: "OpenAPI Generator not found"

Make sure Docker is running:
```bash
docker ps
```

Pull the OpenAPI Generator image manually:
```bash
docker pull openapitools/openapi-generator-cli:v6.4.0
```

## Resources

- [Zig Documentation](https://ziglang.org/documentation/)
- [Zig Build System](https://ziglang.org/learn/build-system/)
- [OpenFGA API Docs](https://openfga.dev/api)
- [Implementation Guide](./IMPLEMENTATION_GUIDE.md)
- [Repository README](../../README.md)

## Getting Help

- Open an issue in the repository
- Check existing SDK implementations (Go, Python, JS) for patterns
- Consult the Zig community for C interop questions
- Review the generated C code documentation in `clients/fga-zig-sdk/docs/`

## License

The generated SDK is licensed under Apache-2.0, same as the OpenFGA project.
