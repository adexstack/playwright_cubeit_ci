# Playwright CI learning project

This small repository demonstrates using Playwright with pytest and GitHub Actions to run end-to-end browser tests against a tiny static app.

It is intended as a learning / demo project for GitHub Actions CI that exercises:

- Python test automation with Playwright (via the `pytest-playwright` plugin).
- Installing browser engine binaries during CI runs.
- Serving a local static site for tests and running tests against it.

## Repository layout

- `cubeit/index.html` — a tiny static page that computes the cube of a number.
- `tests/test_cubeit.py` — Playwright + pytest tests that open the page on `http://127.0.0.1:8000/` and assert behaviour.
- `.github/workflows/test.yml` — GitHub Actions workflow that installs dependencies, ensures Playwright browsers are installed, serves `cubeit/` on port 8000, then runs `pytest`.

## Quick local setup

Prereqs: Python 3.8+ and network access to download Playwright engines. On macOS you may want to use a virtualenv.

1. Create and activate a virtual environment (recommended):

```bash
python3 -m venv venv
source venv/bin/activate
```

2. Upgrade pip and install test dependencies:

```bash
python -m pip install --upgrade pip
python -m pip install pytest-playwright
```

3. Install Playwright browser binaries (required to run tests):

```bash
python -m playwright install chromium --with-deps
```

Note: CI uses `--with-deps` on Ubuntu to ensure system libs are present. On macOS this flag is ignored (macOS binaries are used).

4. Start the static server and run tests (from repository root):

```bash
# start http server serving the `cubeit/` folder on port 8000 in the background
cd cubeit
python -m http.server 8000 &
cd ..

# run pytest (Playwright fixtures are available via pytest-playwright)
pytest -q
```

The tests expect the site to be reachable at `http://127.0.0.1:8000/`.

## How the GitHub Actions workflow works

Location: `.github/workflows/test.yml` — key steps:

- Checkout repository.
- Setup Python (uses `actions/setup-python@v4` with `python-version: '3.11'`).
- Upgrade pip and `pip install pytest-playwright`.
- Run `python -m playwright install chromium --with-deps` to download browser engines and required system libs on Ubuntu.
- Start a simple HTTP server that serves `cubeit/` on port 8000 in the background.
- Execute `pytest -v` to run the Playwright tests.

This sequence ensures a repeatable environment inside the GitHub runner so tests can run headless and reliably.

## Troubleshooting & tips

- If `pip install` fails inside a virtual environment, ensure pip is recent (upgrade with `python -m pip install --upgrade pip`). Older pip versions may not find wheels for some platform-specific packages.
- If you see warnings about scripts installed to a directory not on `PATH` (for example, `~/.local/bin` or `~/Library/Python/.../bin`), add that directory to your `PATH` in `~/.zshrc` or prefer using the virtualenv's `venv/bin` which should already be on `PATH` when activated.
- On Linux runners, Playwright may require additional system packages; CI uses `--with-deps` to help with that. On your own machine refer to Playwright docs if engines fail to launch.
- If tests hang, check whether the local http.server started successfully and whether port 8000 is already in use.

## Contract / expectations for this repo

- Inputs: The static site under `cubeit/` and tests in `tests/`.
- Outputs: Passing/test results reported by GitHub Actions; tests exercise browser automation via Playwright.
- Success criteria: CI job finishes with `pytest` exit code 0 and tests pass.

Edge-cases considered:

- Port 8000 occupied on runner / local machine.
- Playwright engines fail to download (network problems).
- Old `pip` preventing installation of platform wheels.

## Next steps / improvements

- Add a small script to start the server and run pytest so local runs are shorter (e.g., `scripts/run-tests.sh`).
- Add a GitHub Actions matrix to run tests on multiple Python versions or test both headless/headful runs.
- Replace `python -m http.server` with a tiny test fixture that starts and stops a server automatically in the test run to avoid background process management.

## References

- Playwright Python docs: https://playwright.dev/python/
- pytest-playwright: https://github.com/microsoft/playwright-pytest
- GitHub Actions docs: https://docs.github.com/actions

---

If you want, I can:

- Add a `scripts/run-tests.sh` helper and a minimal `Makefile` entry.
- Install the Playwright browsers in your venv now and run the test suite and report results.

Tell me which next step you'd like me to take.
