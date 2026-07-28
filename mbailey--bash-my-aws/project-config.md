---
trigger: always_on
description: This document outlines the coding conventions used in our project, particularly for bash scripts and AWS CLI commands.
---

# Coding Conventions

This document outlines the coding conventions used in our project, particularly for bash scripts and AWS CLI commands.

## Bash Scripts

### Function Naming
- Use lowercase with hyphens for function names (e.g., `vpc-subnets`, `vpc-route-tables`).
- Function names should be descriptive of their purpose.
- For AWS resource-related functions, use the format `resource-type-action` (e.g., `vpc-igw`, `vpc-nat-gateways`).

### Comments
- Use `#` for single-line comments.
- Include a brief description of the function's purpose immediately after the function name.
- For complex functions, add usage examples in the comments.

### AWS CLI Commands
- Use `aws` followed by the service name (e.g., `aws ec2`, `aws rds`).
- Prefer `--output text` for consistent and easily parseable output.
- Use `--query` with JMESPath expressions for filtering and formatting output.

### Output Formatting
- Use `columnise` (a custom function) to format output into columns.
- Sort output when appropriate (e.g., `sort -k 5`).
- For listing functions, output should typically include resource IDs and names.
- Include 'NO_NAME' for resources without a Name tag.

### Error Handling
- Check for required arguments using `[[ -z "$vpc_ids" ]] && __bma_usage "vpc-id [vpc-id]" && return 1`.

### Looping
- Use `for` loops to iterate over multiple VPC IDs or other resources.

### Variable Naming
- Use lowercase for local variables (e.g., `local vpc_ids`, `local filters`).
- Use uppercase for environment variables or constants (e.g., `$AWS_DEFAULT_REGION`).

### Input Handling
- Use `skim-stdin` (a custom function) to handle both piped input and arguments.
- Allow functions to accept input from both command-line arguments and piped input.

#### skim-stdin

The `skim-stdin` function is a utility function that allows functions to accept input from both command-line arguments and piped input. It appends the first token from each line of STDIN to the argument list.

```bash
skim-stdin() {

  # Append first token from each line of STDIN to argument list
  #
  # Implementation of `pipe-skimming` pattern.
  #
  #     $ stacks | skim-stdin foo bar
  #     foo bar huginn mastodon grafana
  #
  #     $ stacks
  #     huginn    CREATE_COMPLETE  2020-01-11T06:18:46.905Z  NEVER_UPDATED  NOT_NESTED
  #     mastodon  CREATE_COMPLETE  2020-01-11T06:19:31.958Z  NEVER_UPDATED  NOT_NESTED
  #     grafana   CREATE_COMPLETE  2020-01-11T06:19:47.001Z  NEVER_UPDATED  NOT_NESTED
  #
  # Typical usage within Bash-my-AWS functions:
  #
  #     local asg_names=$(skim-stdin "$@") # Append to arg list
  #     local asg_names=$(skim-stdin)      # Only draw from STDIN

  local skimmed_stdin="$([[ -t 0 ]] || awk 'ORS=" " { print $1 }')"

  printf -- '%s %s' "$*" "$skimmed_stdin" |
    awk '{$1=$1;print}'  # trim leading/trailing spaces

}
```

### Function Structure
- Start with input validation and error handling.
- Use local variables to store intermediate results.
- Prefer using AWS CLI with appropriate filters and queries over post-processing with grep/awk when possible.

## AWS Resource Conventions

### Resource Listing
- Functions that list resources should follow a consistent output format.
- Include relevant identifiers (e.g., VPC ID, Subnet ID) in the output.
- Include resource names when available, use 'NO_NAME' for unnamed resources.

### Resource Manipulation
- Functions that modify resources should include safety checks.
- For destructive actions, include comments or echo statements explaining the process.

## Security Considerations
- Be cautious with functions that perform destructive actions (e.g., `vpc-default-delete`).
- Include safety checks before performing destructive actions.
- When dealing with sensitive information, use appropriate AWS CLI options to handle credentials securely.

## Utility Functions
- Create and use utility functions for common operations (e.g., `__bma_read_filters`, `__bma_usage`).
- Prefix internal utility functions with double underscores (e.g., `__bma_error`).

## Documentation
- Include a brief description of each function's purpose in comments.
- For complex functions or those with multiple options, include usage examples in the comments.

## Consistency
- Maintain consistent indentation (prefer 2 spaces).
- Use consistent naming conventions across all functions.

## Shared Functions

### skim-stdin
The `skim-stdin` function is a crucial utility for handling input in our scripts. It allows functions to accept input from both command-line arguments and piped input, providing flexibility in how users interact with our tools.

```bash
skim-stdin() {
  # Append first token from each line of STDIN to argument list
  #
  # Implementation of `pipe-skimming` pattern.
  #
  #     $ stacks | skim-stdin foo bar
  #     foo bar huginn mastodon grafana
  #
  #     $ stacks
  #     huginn    CREATE_COMPLETE  2020-01-11T06:18:46.905Z  NEVER_UPDATED  NOT_NESTED
  #     mastodon  CREATE_COMPLETE  2020-01-11T06:19:31.958Z  NEVER_UPDATED  NOT_NESTED
  #     grafana   CREATE_COMPLETE  2020-01-11T06:19:47.001Z  NEVER_UPDATED  NOT_NESTED

  local skimmed_stdin="$([[ -t 0 ]] || awk 'ORS=" " { print $1 }')"

  printf -- '%s %s' "$*" "$skimmed_stdin" |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mbailey/bash-my-aws](https://github.com/mbailey/bash-my-aws) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
