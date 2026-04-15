---
trigger: always_on
description: Tinycare-tui is a Go terminal UI application that displays git commits from the last 24 hours and week, current weather, self-care advice, and todo list tasks. It's built with Go 1.20+ using the tview library for the terminal interface.
---

# Tinycare-tui

Tinycare-tui is a Go terminal UI application that displays git commits from the last 24 hours and week, current weather, self-care advice, and todo list tasks. It's built with Go 1.20+ using the tview library for the terminal interface.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Bootstrap, Build, and Test the Repository
Run these commands in sequence to set up and validate the codebase:

```bash
# Install dependencies (takes ~8-10 seconds)
go mod download

# Build all packages (takes ~15-20 seconds)
go build ./...

# Install the binary (takes ~1 second)
go install ./cmd/tinycare-tui

# Run tests with coverage (takes ~4-7 seconds) - NEVER CANCEL
go test -v -cover ./internal/... -coverprofile coverage.out -coverpkg ./internal/...
```

**CRITICAL TIMING NOTES:**
- Build takes 15-20 seconds. NEVER CANCEL. Set timeout to 60+ seconds minimum.
- Tests take 4-7 seconds. NEVER CANCEL. Set timeout to 30+ seconds minimum.
- One test may fail (jokes API test) - this is expected and not a blocker.

### Code Quality and Linting
Always run these commands before submitting changes:

```bash
# Format code (takes <1 second)
go fmt ./...

# Vet code for issues (takes ~3 seconds)
go vet ./...
```

### Run the Application

#### Basic Run (Minimal Configuration)
```bash
# Create a test todo file for demonstration
echo -e "- [ ] Test task 1\n- [x] Completed task\n- [ ] Another task" > /tmp/test_todos.txt

# Run with minimal environment variables
TINYCARE_POSTAL_CODE=10001 TODO_FILE=/tmp/test_todos.txt tinycare-tui
```

#### Full Configuration Run
```bash
# Set all possible environment variables for complete functionality
export GITHUB_TOKEN=your_github_token_here
export OPEN_WEATHER_MAP_API_KEY=your_openweather_api_key_here
export TODOIST_TOKEN=your_todoist_token_here
export TINYCARE_POSTAL_CODE=10001
export TINYCARE_WORKSPACE=/path/to/git/repos,/another/path/to/repos
export TODO_FILE=/path/to/todolist/file

# Run the application
tinycare-tui
```

#### Test with Local Git Repository
```bash
# Create a test git repository with commits
cd /tmp
git init test_repo
cd test_repo
git config user.name "Test User"
git config user.email "test@test.com"
echo "# Test repo" > README.md
git add . && git commit -m "Initial commit"
echo "# Updated" >> README.md
git add . && git commit -m "Second commit"

# Run tinycare-tui with the test repository
TINYCARE_POSTAL_CODE=10001 TODO_FILE=/tmp/test_todos.txt TINYCARE_WORKSPACE=/tmp/test_repo tinycare-tui
```

## Validation

### Manual Validation Scenarios
After making changes, ALWAYS test these scenarios:

1. **Basic Startup Test:**
   - Run with minimal environment variables
   - Verify the application starts and shows the TUI interface
   - Check that appropriate error messages appear for missing API keys

2. **Git Integration Test:**
   - Create a test git repository with recent commits
   - Set TINYCARE_WORKSPACE to the test repository path
   - Verify commits appear in both Daily and Weekly commit panels

3. **Todo File Test:**
   - Create a test todo file with mixed completed/incomplete tasks
   - Set TODO_FILE environment variable
   - Verify tasks appear correctly in the Today's Tasks panel

4. **UI Navigation Test:**
   - Start the application
   - Press 'r' to refresh - verify self-care message changes
   - Press 'q' to quit - verify application exits cleanly
   - Press Tab to navigate between panels (if focus highlighting works)

### Expected Application Behavior
- **Startup:** Application loads within 2-3 seconds
- **Display:** Shows 4 main panels (Daily Commits, Weekly Commits, Weather, Self Care, Joke, Today's Tasks)
- **Controls:** 
  - 'q' exits the application
  - 'r' refreshes all content
  - Tab navigates between focusable panels
- **Auto-refresh:** Content refreshes every 30 seconds automatically
- **Error Handling:** Missing environment variables show helpful error messages

### CI Requirements
The GitHub Actions CI (.github/workflows/coverage.yml) will:
- Build the application with `go install ./...`
- Run tests with coverage requirement
- Generate coverage badge (currently 82.3%)

Always ensure your changes maintain or improve test coverage.

## Common Tasks

### Repository Structure
```
tinycare-tui/
├── cmd/tinycare-tui/        # Main application entry point
│   └── main.go             # Application setup and UI initialization
├── internal/               # Internal packages
│   ├── apis/              # External API integrations (GitHub, weather, jokes, Todoist)
│   ├── local/             # Local file and git repository handling
│   ├── ui/                # Terminal UI components and views
│   └── utils/             # Utility functions (formatting, etc.)
├── .github/workflows/     # CI/CD configuration
├── go.mod                 # Go module definition
└── go.sum                 # Go module checksums
```

### Key Files to Monitor
- `cmd/tinycare-tui/main.go` - Main application logic and UI setup

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DMcP89) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
