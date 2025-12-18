# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a learning/demo project demonstrating Playwright end-to-end browser testing with pytest and GitHub Actions CI. The project tests a simple static HTML calculator app that computes the cube of a number.

## Repository Structure

- `cubeit/index.html` — Static single-page app that calculates cubes using vanilla JavaScript and Bootstrap
- `tests/test_cubeit.py` — Playwright + pytest tests that validate the calculator behavior
- `.github/workflows/test.yml` — CI workflow that installs dependencies, browsers, serves the app, and runs tests

## Development Commands

### Initial Setup

```bash
# Create and activate virtual environment
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
python -m pip install --upgrade pip
pip install pytest-playwright

# Install Playwright browser binaries (Chromium)
python -m playwright install chromium --with-deps
```

Note: `--with-deps` is essential on Linux to install system libraries; on macOS it's ignored (native binaries are used).

### Running Tests

```bash
# Start the static HTTP server in the background (from repository root)
cd cubeit
python -m http.server 8000 &
cd ..

# Run all tests
pytest -v

# Run tests quietly
pytest -q

# Run a specific test
pytest tests/test_cubeit.py::test_cube -v
```

The tests expect the site to be reachable at `http://127.0.0.1:8000/`.

### Stopping the Background Server

```bash
# Find and kill the http.server process
lsof -ti:8000 | xargs kill -9
```

## Test Architecture

### Test Framework
- Uses `pytest-playwright` plugin which provides Playwright fixtures automatically
- Tests use Playwright's `sync_api` (synchronous API)
- The `page` fixture is injected by pytest-playwright and manages browser context/lifecycle

### Test File: `tests/test_cubeit.py`
- `BASE_URL = "http://127.0.0.1:8000/"` — Tests expect local server on this URL
- Tests use Playwright's locator strategies: `get_by_placeholder()`, `get_by_role()`, `locator()`
- Assertions use Playwright's `expect()` API with matchers like `to_contain_text()` and `to_have_text()`

### Key Test Patterns
1. Navigate to page: `page.goto(BASE_URL)`
2. Locate elements: Use semantic locators (roles, placeholders) or CSS selectors
3. Interact: `.fill()` for inputs, `.click()` for buttons
4. Assert: `expect(locator).to_contain_text()` or `.to_have_text()`

## Application Under Test

The `cubeit/index.html` app:
- Single-page calculator with one number input and a "Cube" button
- Input has placeholder "enter number..."
- Result displayed in `<p id="result">`
- Behavior:
  - Empty input → displays "Enter something!"
  - Numeric input → displays formatted result (e.g., "5³ = 125")
- Uses vanilla JavaScript (`onclick="onClick()"`) and Bootstrap 5 for styling

## CI/CD (GitHub Actions)

Workflow: `.github/workflows/test.yml`
- Triggers on every push
- Runs on `ubuntu-latest` with Python 3.11
- Steps:
  1. Checkout code
  2. Setup Python
  3. Install pytest-playwright
  4. Install Chromium browser with system dependencies
  5. Start HTTP server in background
  6. Run pytest with verbose output

## Known Issues & Troubleshooting

- **Port 8000 occupied**: Kill existing processes using `lsof -ti:8000 | xargs kill -9`
- **Playwright browsers not found**: Run `python -m playwright install chromium --with-deps`
- **Old pip version**: Upgrade with `python -m pip install --upgrade pip`
- **Tests hang**: Verify HTTP server started successfully and port 8000 is accessible
- **PATH warnings**: Add suggested directories to PATH or use virtualenv (recommended)

## Future Improvements (from README)

- Add helper script (`scripts/run-tests.sh`) to automate server startup and test execution
- Add CI matrix to test multiple Python versions
- Replace background `http.server` with pytest fixture that manages server lifecycle
