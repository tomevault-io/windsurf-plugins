---
trigger: always_on
description: This is a LinkedIn automation bot built with Python and Selenium that automates connection requests with personalized messages. The bot navigates LinkedIn profiles based on search criteria and sends connection requests with custom notes.
---

# LinkedIn Auto Connector Bot - Development Documentation

## Project Overview

This is a LinkedIn automation bot built with Python and Selenium that automates connection requests with personalized messages. The bot navigates LinkedIn profiles based on search criteria and sends connection requests with custom notes.

**⚠️ Important:** This tool should be used responsibly and within LinkedIn's usage limits (max 80-100 requests per week) to avoid account restrictions.

## Architecture

### Technology Stack
- **Language:** Python 3.8+
- **Web Automation:** Selenium WebDriver 4.22.0
- **Browser:** Mozilla Firefox with Geckodriver
- **Logging:** Python's built-in logging module

### Project Structure
```
LinkedIn_Auto_Connector_Bot/
├── Linkedin_auto_connector_bot.py  # Main bot script
├── requirements.txt                 # Python dependencies
├── geckodriver32.exe               # Firefox WebDriver (Windows)
├── banner.jpg                      # Repository banner image
├── README.md                       # User documentation
├── LICENSE                         # MIT License
├── .gitignore                      # Git ignore rules
└── CLAUDE.md                       # This development documentation
```

## Core Components

### 1. Authentication Module
- **Function:** `login_to_linkedin(driver, username, password)`
- **Location:** Linkedin_auto_connector_bot.py:47-66
- Handles LinkedIn login process
- Waits for manual CAPTCHA solving if required

### 2. Navigation Module
- **Function:** `go_to_next_page(driver)`
- **Location:** Linkedin_auto_connector_bot.py:67-83
- Handles pagination through search results
- Implements scroll-down functionality

### 3. Connection Request Handler
- **Function:** `handle_connect_button_with_retry(driver, button)`
- **Location:** Linkedin_auto_connector_bot.py:92-127
- Sends personalized connection requests
- Implements retry mechanism for failed attempts
- Handles "Add a note" workflow

### 4. Main Processing Loop
- **Function:** `process_buttons(driver)`
- **Location:** Linkedin_auto_connector_bot.py:136-185
- Orchestrates the entire bot workflow
- Counts and limits connection requests
- Processes both "Connect" and "Follow" buttons

### 5. Error Recovery
- **Function:** `refresh_page(driver, retries)`
- **Location:** Linkedin_auto_connector_bot.py:187-201
- Implements page refresh recovery mechanism
- Handles maximum retry attempts

## Configuration

### Current Configuration Method
- Hardcoded credentials in script (security risk)
- Hardcoded search URL
- Hardcoded connection message
- Fixed WebDriver paths

### Environment Variables Needed
```bash
# .env file (to be implemented)
LINKEDIN_USERNAME=your_email@example.com
LINKEDIN_PASSWORD=your_password
SEARCH_LINK=https://www.linkedin.com/search/results/people/...
MAX_CONNECT_REQUESTS=20
FIREFOX_BINARY_PATH=/path/to/firefox
GECKODRIVER_PATH=/path/to/geckodriver
```

## Security Considerations

### Current Issues
1. **Hardcoded Credentials:** Username and password are directly in the code
2. **No Encryption:** Credentials stored in plain text
3. **Version Control Risk:** Sensitive data could be committed to repository
4. **No Rate Limiting:** Manual limit enforcement only

### Recommended Improvements
1. Use environment variables for all sensitive data
2. Implement proper rate limiting with daily/weekly tracking
3. Add credential encryption/keyring support
4. Create separate config file for non-sensitive settings
5. Implement session persistence to avoid repeated logins

## Testing Guidelines

### Manual Testing Checklist
- [ ] Test login with valid credentials
- [ ] Test login with invalid credentials
- [ ] Test CAPTCHA handling
- [ ] Test connection request sending
- [ ] Test pagination navigation
- [ ] Test error recovery mechanisms
- [ ] Test rate limiting enforcement
- [ ] Test with different search queries
- [ ] Test on different operating systems

### Automated Testing (To Be Implemented)
```python
# Example test structure
def test_login_success():
    # Mock WebDriver and test successful login
    pass

def test_connection_limit():
    # Test that bot stops at MAX_CONNECT_REQUESTS
    pass

def test_error_recovery():
    # Test retry mechanism
    pass
```

## Improvement Suggestions

### High Priority
1. **Environment Configuration**
   - Move all sensitive data to .env file
   - Use python-dotenv for environment variable management

2. **Error Handling**
   - Add comprehensive exception handling
   - Implement proper logging to files
   - Add screenshot capture on errors

3. **Rate Limiting**
   - Implement database to track daily/weekly counts
   - Add automatic scheduling/delays
   - Create warning system near limits

### Medium Priority
1. **Code Structure**
   - Split into multiple modules (auth, navigation, messaging)
   - Create configuration class
   - Implement command-line arguments

2. **Features**
   - Add profile filtering options
   - Implement message templates with variables
   - Add connection acceptance monitoring

3. **Cross-Platform Support**
   - Detect OS and adjust paths automatically

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OfficialCodeVoyage/LinkedIn_Auto_Connector_Bot](https://github.com/OfficialCodeVoyage/LinkedIn_Auto_Connector_Bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
