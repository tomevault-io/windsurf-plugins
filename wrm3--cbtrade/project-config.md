---
trigger: always_on
description: - You have less problems reading the results from the terminal when they are run in the background.
---

# PowerShell Command Reference for Windows Development
- You have less problems reading the results from the terminal when they are run in the background.

## CRITICAL: Terminal Integration Self-Diagnosis

### � AI Self-Check: Am I Reading PowerShell Results Correctly?

**MANDATORY CHECK before any PowerShell operation:**

#### Symptoms of Terminal Integration Failure:
- ❌ Waiting indefinitely for command results that already appeared
- ❌ Treating error messages as successful outputs  
- ❌ Missing command output entirely
- ❌ Cursor position errors in terminal
- ❌ PSReadLine integration problems

#### Self-Diagnostic Questions:
1. **Did I see actual command output?** If not, terminal integration may be broken
2. **Am I treating errors as success?** Check if output contains error keywords
3. **Am I waiting for results that already came?** Look for completed command patterns
4. **Is the terminal showing cursor position errors?** This indicates integration failure

#### Emergency Fallback Methods:
```powershell
# Method 1: Explicit output capture and display
$result = python -c "print('test')" 2>&1
Write-Host "RESULT: $result"

# Method 2: Output to file for verification
python -c "print('test')" > temp_output.txt 2>&1
Get-Content temp_output.txt

# Method 3: Use explicit error handling
try {
    $output = python -c "print('test')" 2>&1
    Write-Host "SUCCESS: $output"
} catch {
    Write-Host "ERROR: $($_.Exception.Message)"
}

# Method 4: Force UTF-8 encoding
$OutputEncoding = [Console]::OutputEncoding = [Text.Encoding]::UTF8
python -c "print('test')"
```

### Self-Fixing Protocol:

#### Step 1: Detect Integration Failure
```powershell
# Test command to verify terminal integration
Write-Host "TERMINAL_TEST_START"
python -c "print('TERMINAL_TEST_SUCCESS')"
Write-Host "TERMINAL_TEST_END"
```

#### Step 2: Apply Immediate Workarounds
```powershell
# Set proper encoding
$OutputEncoding = [Console]::OutputEncoding = [Text.Encoding]::UTF8

# Use explicit output capture
function Invoke-SafePythonCommand {
    param([string]$Command)
    try {
        $result = python -c $Command 2>&1
        Write-Host "PYTHON_OUTPUT: $result"
        return $result
    } catch {
        Write-Host "PYTHON_ERROR: $($_.Exception.Message)"
        return $null
    }
}
```

#### Step 3: Alternative Command Strategies
When terminal integration fails, use these patterns:

```powershell
# Instead of direct commands, use wrapped versions:

# OLD (integration-dependent):
python script.py

# NEW (integration-safe):
$result = python script.py 2>&1
Write-Host "EXECUTION_RESULT: $result"

# For file operations:
python script.py > output.log 2>&1
Write-Host "Command completed. Output:"
Get-Content output.log
```

## Powershell on Windows 10/11
- You are running powershell on windows 10/11, ensure that your commands and powershell usage is adjusted accordingly
- You have issues with using curl as its interpretted as a 'Invoke-Webrequest command and you will end up hung up on a prompt waiting for uri:
- && does not work as a command seperator well, you perform better when using ;

## Powershell Date Time
- when using powershell to get the time, use this command first "powershell -Command "(Get-Date).ToUniversalTime().ToString('yyyy-MM-ddTHH:mm:ssZ')"


## Curl/HTTP Request Commands

**CRITICAL:** In PowerShell on Windows, `curl` is an alias for `Invoke-WebRequest` cmdlet, NOT the actual curl executable. This causes interactive prompts that block automation.

### ❌ AVOID - Gets stuck on Uri prompt:
```powershell
curl -s http://localhost:5000/api/status
# This will prompt: "Uri:" and hang
```

### ✅ USE INSTEAD:
```powershell
# Option 1: Use Invoke-WebRequest with proper syntax
Invoke-WebRequest -Uri "http://localhost:5000/api/status" -UseBasicParsing

# Option 2: Use shorthand 'iwr'
iwr "http://localhost:5000/api/status" -UseBasicParsing

# Option 3: Force actual curl executable
curl.exe -s http://localhost:5000/api/status

# Option 4: For JSON responses
(Invoke-WebRequest -Uri "http://localhost:5000/api/status").Content | ConvertFrom-Json
```

## Directory Navigation
```powershell
# Change directory and run command in one line
cd flask-vpn-manager; python main.py

# Create multiple directories
New-Item -ItemType Directory -Path "app\templates", "app\static" -Force
```

## Powershell Flask Servers
- You often get hung when running a flask web server in chat... its better if you run it as a background task

## Flask Development
```powershell
# Start Flask in background
python main.py &

# Test endpoints
Invoke-WebRequest -Uri "http://127.0.0.1:5000/" -UseBasicParsing
Invoke-WebRequest -Uri "http://127.0.0.1:5000/api/status" -UseBasicParsing
```

## Recognition Patterns for Failed Commands

### Error Keywords to Watch For:
- "Exception", "Error", "Failed", "Cannot", "Unable"
- "Access denied", "Permission denied", "File not found"
- "Traceback", "SyntaxError", "ImportError"
- "Connection refused", "Timeout", "Network error"

### Success Indicators:
- Actual expected output content
- Exit codes of 0
- Expected file creation/modification
- Proper JSON/data structure responses

### Integration Failure Indicators:
- No output when output is expected
- Cursor remaining in "waiting" state

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wrm3/CBTrade](https://github.com/wrm3/CBTrade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
