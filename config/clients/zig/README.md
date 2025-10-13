# Zig SDK Configuration

This directory contains the configuration for generating an OpenFGA SDK for the Zig programming language.

## Current Status: Experimental / Incomplete

**Important**: OpenAPI Generator v6.4.0 does not have native support for Zig. This configuration uses the `C-libcurl` generator as a base, which generates a C client library. To create a functional Zig SDK, additional work is required to wrap the C library with idiomatic Zig code.

## What Has Been Set Up

1. **Directory Structure**: All necessary configuration directories and files
2. **Base Configuration**: `config.overrides.json` with Zig SDK metadata
3. **Generator Selection**: Using `C-libcurl` as the base generator (closest match for a systems language)
4. **Template Files**: README templates with Zig-specific examples
5. **Build Infrastructure**: Makefile targets for building the Zig SDK
6. **CI Workflow**: GitHub Actions workflow template for Zig projects

## What Still Needs to Be Done

### 1. Zig Wrapper Implementation

The C-libcurl generator will produce C code. You'll need to:

- Create a Zig `build.zig` file to compile the C library
- Write Zig wrapper code that provides idiomatic Zig interfaces
- Implement memory management using Zig's allocator pattern
- Add proper error handling using Zig's error unions
- Create Zig-friendly types that wrap the C structs

### 2. Custom Templates

Consider creating custom mustache templates that generate:

- Zig-specific code comments
- Zig documentation format
- Proper Zig module structure

### 3. Build System Integration

- Update the Makefile to properly handle Zig compilation
- Add proper dependencies (libcurl, JSON parsing, etc.)
- Configure Zig package management

### 4. Testing

- Implement unit tests in Zig
- Add integration tests
- Set up CI/CD for Zig

### 5. Alternative Approaches

Consider these alternatives:

- **Wait for Official Support**: Monitor OpenAPI Generator for native Zig support
- **Custom Generator**: Contribute a Zig generator to OpenAPI Generator
- **Hand-Written SDK**: Write a Zig SDK from scratch based on the OpenAPI spec
- **FFI Bindings**: Use Zig's C interop to create thin wrappers around another SDK

## Building the SDK

To generate the C base code:

```bash
make build-client-zig
```

This will create a C client library in `clients/zig/`. You'll then need to add Zig wrapper code.

## Resources

- [Zig Documentation](https://ziglang.org/documentation/master/)
- [Zig Build System](https://ziglang.org/learn/build-system/)
- [Zig C Interop](https://ziglang.org/documentation/master/#C)
- [OpenAPI Generator Documentation](https://github.com/OpenAPITools/openapi-generator)
- [OpenFGA API Documentation](https://openfga.dev/api)

## Contributing

If you'd like to help complete this Zig SDK implementation, consider:

1. Implementing the Zig wrapper layer
2. Contributing a native Zig generator to OpenAPI Generator
3. Improving the templates and configuration
4. Adding comprehensive tests

For questions or contributions, please open an issue or pull request in the repository.
