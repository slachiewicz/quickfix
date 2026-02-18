# QuickFIX/Go - Agent Guidelines

This document provides guidelines for agentic coding agents working in the QuickFIX/Go repository.

## Build, Test, and Lint Commands

### Primary Commands
- `make` - Runs go vet and all unit tests (default target)
- `make test` - Runs unit tests with verbose output and coverage (excludes gen/)
- `make vet` - Runs go vet on all non-generated packages
- `make fmt` - Formats code with gofmt -l -w -s
- `make lint` - Runs golangci-lint (v1.64.6) on all non-generated packages

### Running Single Tests
```bash
# Run all tests in a specific package
go test -v ./path/to/package

# Run a specific test function
go test -v ./path/to/package -run TestFunctionName

# Run tests in verbose mode with coverage
go test -v -cover ./path/to/package
```

### Code Generation
- `make generate` - Regenerates FIX protocol code from spec/*.xml (cleans gen/ first)
- `make generate-udecimal` - Generates code using udecimal type

### Acceptance Tests
- `make build-test-srv` - Builds test server for acceptance tests
- `make accept` - Runs full acceptance test suite (requires Ruby)
- Individual test suite targets: fix40, fix41, fix42, fix43, fix44, fix50, fix50sp1, fix50sp2

### CI Commands
- `make build` - Builds source and test server
- `make build-src` - Builds all packages
- `make test-ci` - Runs tests for CI

## Code Style Guidelines

### Import Organization
- Use `goimports` for import ordering (included in lint)
- Group imports: stdlib -> third-party -> internal
- No blank lines between import groups

### Formatting
- Use `gofmt -s` (simplify enabled)
- Line length: follow standard Go conventions
- Use tabs for indentation (Go standard)

### Naming Conventions
- Exported types/functions: PascalCase (`Application`, `ToAdmin`)
- Unexported: camelCase (`session`, `shouldSendReset`)
- Constants: lowerCamelCase or UPPER_SNAKE_CASE (`rejectReasonInvalidTagNumber`, defaultBufSize)
- Interfaces: Simple names, no "I" prefix (`Application`, `Field`, `FieldValue`)
- Test files: `*_test.go` suffix
- Test functions: `TestXxx` or `BenchmarkXxx` for benchmarks

### File Structure
- All files begin with 15-line copyright header
- Package doc in `doc.go` (file-level documentation)
- Types and their methods typically in same file
- Interfaces defined at package level
- Generated files use `.generated.go` suffix (located in gen/ directory)

### Error Handling
- Errors are returned as values, never ignored
- Use `errors.Is()` and `errors.As()` for error wrapping/inspection
- Custom error types implement `error` interface
- For FIX-specific errors, use `MessageRejectError` interface
- Use `require` for test setup (fail on error), `assert` for assertions

### Testing
- Use `github.com/stretchr/testify/suite` for test suites
- Embed `QuickFIXSuite` in test suites
- Use table-driven tests where appropriate
- Benchmark functions start with `Benchmark`
- Test helpers can be unexported
- Use `suite.Run(t, new(MySuite))` pattern

### Type Definitions
- Use embedding for composition (`type Body struct{ FieldMap }`)
- Define ordering functions for FieldMap: `initWithOrdering(func(i, j Tag) bool)`
- Field types use FIX prefix (`FIXString`, `FIXInt`, `FIXBoolean`)
- Message sections: `Header`, `Body`, `Trailer`

### Concurrency
- Use `sync.Mutex`, `sync.RWMutex` for mutual exclusion
- Use channels for message passing
- Document locking order to prevent deadlocks
- Prefer `sync.Once` for one-time initialization

### Package Organization
- Internal utilities go in `internal/` package
- Third-party libraries in separate directories (store/, log/)
- Generated code excluded: `gen/` directory
- Test utilities: `internal/testsuite/`

## Linter Configuration
Uses golangci-lint v1.64.6 with these enabled linters:
- dupl (threshold: 400)
- gofmt, goimports
- gosimple, govet, ineffassign
- misspell
- revive, unused, staticcheck
- godot (for doc comments)

Excludes: gen/, vendor/

## Development Notes
- Go 1.23+ required
- Generated code changes must be captured in `cmd/generate-fix/`
- When modifying generated code, run `make generate` after changes
- Test coverage expected for contributions
- Do not edit generated files directly
- For MongoDB tests: set `MONGODB_TEST_CXN=mongodb://db:27017` environment variable