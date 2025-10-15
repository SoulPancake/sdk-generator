# Zig SDK Implementation Guide

This guide provides detailed instructions for implementing a complete Zig SDK wrapper around the generated C client library.

## Overview

The OpenAPI Generator produces a C client library using libcurl. To create a proper Zig SDK, you need to:

1. Create Zig bindings for the C API
2. Implement idiomatic Zig wrappers
3. Set up a Zig build system
4. Add proper error handling
5. Implement memory management
6. Write tests and documentation

## Step 1: Project Structure

Create the following structure in the generated SDK:

```
fga-zig-sdk/
├── build.zig              # Zig build configuration
├── build.zig.zon          # Zig package manifest
├── src/
│   ├── main.zig           # Main module entry point
│   ├── client.zig         # High-level client implementation
│   ├── config.zig         # Configuration types
│   ├── models.zig         # Zig model types
│   ├── c_bindings.zig     # C FFI bindings
│   └── errors.zig         # Error types and handling
├── c/                     # Generated C code (from OpenAPI Generator)
│   ├── api/
│   ├── model/
│   ├── include/
│   └── src/
└── test/
    └── client_test.zig    # Unit tests
```

## Step 2: Create build.zig

```zig
const std = @import("std");

pub fn build(b: *std.Build) void {
    const target = b.standardTargetOptions(.{});
    const optimize = b.standardOptimizeOption(.{});

    // Create a C library from generated code
    const c_lib = b.addStaticLibrary(.{
        .name = "openfga-c",
        .target = target,
        .optimize = optimize,
    });
    
    c_lib.linkLibC();
    c_lib.linkSystemLibrary("curl");
    
    // Add C source files
    c_lib.addCSourceFiles(.{
        .files = &.{
            "c/src/apiClient.c",
            "c/src/apiKey.c",
            "c/src/list.c",
            "c/src/binary.c",
            "c/external/cJSON.c",
            // Add all model/*.c files
            // Add all api/*.c files
        },
        .flags = &.{"-std=c99"},
    });
    
    c_lib.addIncludePath(.{ .path = "c/include" });
    c_lib.addIncludePath(.{ .path = "c/model" });
    c_lib.addIncludePath(.{ .path = "c/api" });
    c_lib.addIncludePath(.{ .path = "c/external" });
    
    // Create Zig module
    const openfga_module = b.addModule("openfga", .{
        .root_source_file = .{ .path = "src/main.zig" },
        .target = target,
        .optimize = optimize,
    });
    
    // Link C library to Zig module
    openfga_module.linkLibrary(c_lib);
    
    // Create tests
    const tests = b.addTest(.{
        .root_source_file = .{ .path = "test/client_test.zig" },
        .target = target,
        .optimize = optimize,
    });
    
    tests.root_module.addImport("openfga", openfga_module);
    
    const run_tests = b.addRunArtifact(tests);
    const test_step = b.step("test", "Run unit tests");
    test_step.dependOn(&run_tests.step);
}
```

## Step 3: Create C Bindings (src/c_bindings.zig)

```zig
const std = @import("std");

// Import C headers
pub const c = @cImport({
    @cInclude("apiClient.h");
    @cInclude("OpenFgaAPI.h");
    // Include model headers as needed
});

// Wrap C API client
pub const CApiClient = c.apiClient_t;

pub fn createApiClient() *CApiClient {
    return c.apiClient_create().?;
}

pub fn freeApiClient(client: *CApiClient) void {
    c.apiClient_free(client);
}

// Add more C function wrappers as needed
```

## Step 4: Create Configuration Types (src/config.zig)

```zig
const std = @import("std");

pub const ClientConfiguration = struct {
    api_url: []const u8,
    store_id: ?[]const u8 = null,
    authorization_model_id: ?[]const u8 = null,
    api_token: ?[]const u8 = null,
    
    pub fn validate(self: ClientConfiguration) !void {
        if (self.api_url.len == 0) {
            return error.InvalidApiUrl;
        }
        // Add more validation
    }
};
```

## Step 5: Create Client Wrapper (src/client.zig)

```zig
const std = @import("std");
const c = @import("c_bindings.zig").c;
const config = @import("config.zig");

pub const Client = struct {
    allocator: std.mem.Allocator,
    c_client: *c.apiClient_t,
    configuration: config.ClientConfiguration,
    
    pub fn init(
        allocator: std.mem.Allocator,
        configuration: config.ClientConfiguration,
    ) !Client {
        try configuration.validate();
        
        const c_client = c.apiClient_create() orelse 
            return error.ClientCreationFailed;
        
        // Set base path from configuration
        c_client.basePath = try allocator.dupeZ(u8, configuration.api_url);
        
        return Client{
            .allocator = allocator,
            .c_client = c_client,
            .configuration = configuration,
        };
    }
    
    pub fn deinit(self: *Client) void {
        if (self.c_client.basePath) |path| {
            self.allocator.free(std.mem.span(path));
        }
        c.apiClient_free(self.c_client);
    }
    
    // Implement API methods
    pub fn check(self: *Client, user: []const u8, relation: []const u8, object: []const u8) !CheckResult {
        // Call C API and wrap result
        // Handle errors and convert to Zig types
        _ = self;
        _ = user;
        _ = relation;
        _ = object;
        return error.NotImplemented;
    }
};

pub const CheckResult = struct {
    allowed: bool,
    
    pub fn deinit(self: CheckResult) void {
        _ = self;
    }
};
```

## Step 6: Create Main Module (src/main.zig)

```zig
const std = @import("std");

pub const Client = @import("client.zig").Client;
pub const ClientConfiguration = @import("config.zig").ClientConfiguration;
pub const CheckResult = @import("client.zig").CheckResult;

// Export error types
pub const Error = error{
    InvalidApiUrl,
    ClientCreationFailed,
    NotImplemented,
    // Add more errors
};

test {
    std.testing.refAllDecls(@This());
}
```

## Step 7: Error Handling

Create comprehensive error types:

```zig
// src/errors.zig
pub const FgaError = error{
    // Configuration errors
    InvalidConfiguration,
    InvalidApiUrl,
    
    // Client errors
    ClientCreationFailed,
    ClientNotInitialized,
    
    // API errors
    Unauthorized,
    Forbidden,
    NotFound,
    RateLimited,
    ServerError,
    
    // Network errors
    NetworkError,
    Timeout,
    
    // Parsing errors
    JsonParseError,
    InvalidResponse,
};

pub fn mapCError(c_error: i32) FgaError {
    return switch (c_error) {
        401 => FgaError.Unauthorized,
        403 => FgaError.Forbidden,
        404 => FgaError.NotFound,
        429 => FgaError.RateLimited,
        else => FgaError.ServerError,
    };
}
```

## Step 8: Memory Management

Ensure proper memory management:

```zig
pub const ManagedString = struct {
    allocator: std.mem.Allocator,
    value: []const u8,
    
    pub fn init(allocator: std.mem.Allocator, value: []const u8) !ManagedString {
        const owned = try allocator.dupe(u8, value);
        return ManagedString{
            .allocator = allocator,
            .value = owned,
        };
    }
    
    pub fn deinit(self: ManagedString) void {
        self.allocator.free(self.value);
    }
};
```

## Step 9: Testing

Create comprehensive tests:

```zig
// test/client_test.zig
const std = @import("std");
const testing = std.testing;
const openfga = @import("openfga");

test "client initialization" {
    const allocator = testing.allocator;
    
    const config = openfga.ClientConfiguration{
        .api_url = "https://api.example.com",
        .store_id = "test-store",
    };
    
    var client = try openfga.Client.init(allocator, config);
    defer client.deinit();
    
    // Test client was created
    try testing.expect(client.c_client != null);
}

test "configuration validation" {
    const config = openfga.ClientConfiguration{
        .api_url = "",
    };
    
    try testing.expectError(error.InvalidApiUrl, config.validate());
}
```

## Step 10: Documentation

Add comprehensive documentation:

```zig
//! OpenFGA Zig SDK
//!
//! This module provides a high-level, idiomatic Zig interface for the OpenFGA API.
//!
//! Example usage:
//! ```zig
//! const openfga = @import("openfga");
//!
//! var client = try openfga.Client.init(allocator, .{
//!     .api_url = "https://api.fga.example",
//!     .store_id = "my-store",
//! });
//! defer client.deinit();
//!
//! const result = try client.check("user:anne", "viewer", "document:1");
//! defer result.deinit();
//! ```

pub const Client = @import("client.zig").Client;
```

## Implementation Checklist

- [ ] Set up build.zig to compile C library
- [ ] Create C bindings for all API functions
- [ ] Implement configuration types with validation
- [ ] Create high-level Client wrapper
- [ ] Implement all API methods (Check, ListObjects, Write, etc.)
- [ ] Add comprehensive error handling
- [ ] Implement proper memory management
- [ ] Write unit tests for all functionality
- [ ] Add integration tests
- [ ] Write documentation
- [ ] Set up CI/CD pipeline
- [ ] Add examples
- [ ] Update README with Zig-specific instructions

## Resources

- [Zig Language Reference](https://ziglang.org/documentation/master/)
- [Zig Build System](https://ziglang.org/learn/build-system/)
- [C Integration](https://ziglang.org/documentation/master/#C)
- [OpenFGA API Reference](https://openfga.dev/api)

## Getting Help

If you need help implementing the Zig SDK:

1. Review existing SDKs (Go, Python, JavaScript) for patterns
2. Check the Zig community forums for C interop questions
3. Open an issue in the repository
4. Join the OpenFGA Discord for real-time help
