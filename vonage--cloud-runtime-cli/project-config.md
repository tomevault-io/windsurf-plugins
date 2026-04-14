---
trigger: always_on
description: When generating code for this repository:
---


# Copilot Instructions for Vonage Cloud Runtime CLI

## Priority Guidelines

When generating code for this repository:

1. **Factory Pattern First**: ALWAYS use the `cmdutil.Factory` interface for dependency injection - never instantiate API clients directly
2. **Go 1.24 Only**: Use only language features available in Go 1.24.0 - this is our version
3. **Error Wrapping**: Always wrap errors with context using `fmt.Errorf("context: %w", err)`
4. **Context Management**: Create deadline context from `opts.Deadline()` in command RunE functions
5. **Testing Required**: All code changes must include unit tests
6. **Consistent Code Style**: Match existing patterns exactly - scan similar files before generating new code

## Technology Stack

**Go Version**: 1.24.0

**Core Dependencies**:
- `github.com/spf13/cobra` v1.9.1 - CLI framework
- `github.com/go-resty/resty/v2` v2.16.5 - HTTP client
- `github.com/cli/cli/v2` v2.36.0 - IO streams and colors
- `github.com/AlecAivazis/survey/v2` v2.3.7 - Interactive prompts
- `github.com/golang/mock` v1.6.0 - Mocking (use mockgen)
- `gopkg.in/ini.v1` v1.67.0 - Config parsing
- `github.com/stretchr/testify` v1.10.0 - Test assertions

## Naming Conventions

### Files
- Lowercase with underscores: `create.go`, `create_test.go`
- Test files: Always `*_test.go` in same package
- Platform-specific: `command_gen_syscall_notwin.go`, `command_gen_syscall_win.go`

### Packages
- Lowercase, single-word: `api`, `config`, `cmdutil`, `format`
- Grouped by domain: `vcr/app/`, `vcr/deploy/`, `vcr/secret/`

### Types and Variables
- **Exported types**: `PascalCase` (e.g., `DeploymentClient`, `Options`)
- **Unexported types**: `camelCase` (e.g., `listRequest`, `createResponse`)
- **Interfaces**: End with `Interface` (e.g., `DeploymentInterface`, `AssetInterface`)
- **Functions**: Exported = `PascalCase`, Unexported = `camelCase`
- **Command factories**: `NewCmd<CommandName>` (e.g., `NewCmdDeploy`, `NewCmdAppCreate`)
- **Run functions**: `run<CommandName>` (e.g., `runDeploy`, `runCreate`)
- **Constants**: `PascalCase` (e.g., `DefaultTimeout`, `DefaultRegion`)

## Command Structure Pattern

Every CLI command MUST follow this exact structure:

```go
package commandname

import (
    "context"
    "fmt"
    "github.com/Vonage/vonage-cloud-runtime-cli/pkg/cmdutil"
    "github.com/spf13/cobra"
)

// Options holds the command-specific configuration
type Options struct {
    cmdutil.Factory  // ALWAYS embed the Factory interface

    // Command-specific fields matching flags
    Name        string
    SkipPrompts bool
}

// NewCmdCommandName creates the command
func NewCmdCommandName(f cmdutil.Factory) *cobra.Command {
    opts := Options{Factory: f}

    cmd := &cobra.Command{
        Use:   "commandname",
        Short: "Brief description",
        RunE: func(_ *cobra.Command, _ []string) error {
            // ALWAYS create context with deadline from Factory
            ctx, cancel := context.WithDeadline(context.Background(), opts.Deadline())
            defer cancel()
            return runCommandName(ctx, &opts)
        },
    }

    // Define flags
    cmd.Flags().StringVarP(&opts.Name, "name", "n", "", "Resource name")

    return cmd
}

// runCommandName executes the command logic
func runCommandName(ctx context.Context, opts *Options) error {
    io := opts.IOStreams()
    c := io.ColorScheme()

    // Get clients from Factory (NEVER instantiate directly)
    deployClient := opts.DeploymentClient()

    // Use spinners for long operations
    spinner := cmdutil.DisplaySpinnerMessageWithHandle("Processing...")
    result, err := deployClient.SomeOperation(ctx, opts.Name)
    spinner.Stop()

    if err != nil {
        return fmt.Errorf("operation failed: %w", err)
    }

    // Use color scheme for output
    fmt.Fprintf(io.Out, "%s Operation successful\n", c.SuccessIcon())
    fmt.Fprintf(io.Out, "%s ID: %s\n", c.Blue(cmdutil.InfoIcon), result.ID)

    return nil
}
```

## Testing Pattern

**All code changes require unit tests.** Table-driven tests are a common pattern in this codebase but not mandatory. Write tests that are clear and maintainable.

**Table-driven test example** (commonly used for commands with multiple scenarios):

```go
package commandname

import (
    "testing"
    "github.com/Vonage/vonage-cloud-runtime-cli/testutil"
    "github.com/golang/mock/gomock"
    "github.com/stretchr/testify/assert"
)

func TestCommandName(t *testing.T) {
    type mock struct {
        OperationTimes      int
        OperationReturnData interface{}
        OperationReturnErr  error
    }

    type want struct {
        errMsg string
        stdout string
    }

    tests := []struct {
        name string
        cli  string
        mock mock
        want want
    }{
        {
            name: "happy-path",
            cli:  "--name=Test",
            mock: mock{
                OperationTimes:      1,
                OperationReturnData: expectedData,
            },
            want: want{
                stdout: "Operation successful",
            },
        },
        {
            name: "error-case",
            cli:  "--name=Test",
            mock: mock{
                OperationTimes:     1,
                OperationReturnErr: errors.New("operation failed"),
            },
            want: want{

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Vonage) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
