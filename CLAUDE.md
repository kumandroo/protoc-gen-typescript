# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a protoc plugin that generates TypeScript type definitions from Protocol Buffer files. It's a Go-based compiler plugin following the standard protoc plugin architecture.

## Development Commands

```bash
# Build the plugin
go build -o protoc-gen-typescript

# Install globally (makes available to protoc)
go install

# Format code
go fmt ./...

# Clean up dependencies
go mod tidy

# Usage with protoc
protoc --typescript_out=. myfile.proto
```

## Architecture

The codebase consists of two main files:

1. **main.go**: Core plugin logic
   - Reads CodeGeneratorRequest from stdin
   - Processes proto descriptors
   - Orchestrates TypeScript generation
   - Key functions: `main()`, `getFieldType()`, `getScopedName()`

2. **templates.go**: TypeScript output templates
   - Contains Go templates for generating TypeScript code
   - Defines the structure of the generated `index.ts` file

## Type Mapping Strategy

- Numeric types (int32, float, etc.) → `number`
- 64-bit integers (int64, uint64) → `string` (JavaScript limitation)
- bytes → `Uint8Array`
- google.protobuf.Timestamp → `string`
- Repeated fields → Arrays
- Maps → `{ [key: T]: V }`
- All message fields are optional (`?:`) in TypeScript
- Enum zero values are skipped (undefined in TypeScript)
- Protobuf packages become TypeScript namespaces with `$` separator

## Key Implementation Details

- Generates a single `index.ts` file containing all types
- Enums are TypeScript union types of string literals
- Messages are TypeScript interfaces with optional fields
- Services follow gRPC-Web style with callback-based methods
- Nested types are fully supported
- Map types are inlined rather than creating separate type definitions

## Testing Approach

Currently no tests exist. To test changes:
1. Create sample `.proto` files
2. Run the plugin: `protoc --typescript_out=. test.proto`
3. Verify the generated TypeScript in `index.ts`