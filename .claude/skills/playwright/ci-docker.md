# Playwright Python CI/CD & Docker Reference

## CI Setup Steps
1. Ensure CI agent can run browsers
2. Install Playwright
3. Run tests

```bash
pip install playwright
playwright install --with-deps
pytest
```

## GitHub Actions

### Standard Workflow
```yaml
name: Playwright Tests
on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]
jobs:
  test:
    timeout-minutes: 60
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v5
    - name: Set up Python
      uses: actions/setup-python@v6
      with:
        python-version: '3.13'
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
    - name: Ensure browsers are installed
      run: python -m playwright install --with-deps
    - name: Run your tests
      run: pytest --tracing=retain-on-failure
    - uses: actions/upload-artifact@v5
      if: ${{ !cancelled() }}
      with:
        name: playwright-traces
        path: test-results/
```

### Using Docker Container
```yaml
name: Playwright Tests
on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]
jobs:
  playwright:
    name: 'Playwright Tests'
    runs-on: ubuntu-latest
    container:
      image: mcr.microsoft.com/playwright/python:v1.57.0-noble
      options: --user 1001
    steps:
      - uses: actions/checkout@v5
      - name: Set up Python
        uses: actions/setup-python@v6
        with:
          python-version: '3.13'
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r local-requirements.txt
          pip install -e .
      - name: Run your tests
        run: pytest
```

### On Deployment
```yaml
name: Playwright Tests
on:
  deployment_status:
jobs:
  test:
    timeout-minutes: 60
    runs-on: ubuntu-latest
    if: github.event.deployment_status.state == 'success'
    steps:
    - uses: actions/checkout@v5
    - uses: actions/setup-python@v6
      with:
        python-version: '3.13'
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
    - name: Ensure browsers are installed
      run: python -m playwright install --with-deps
    - name: Run tests
      run: pytest
      env:
        PLAYWRIGHT_TEST_BASE_URL: ${{ github.event.deployment_status.target_url }}
```

## Azure Pipelines

```yaml
trigger:
- main
pool:
  vmImage: ubuntu-latest
steps:
- task: UsePythonVersion@0
  inputs:
    versionSpec: '3.13'
  displayName: 'Use Python'
- script: |
    python -m pip install --upgrade pip
    pip install -r requirements.txt
  displayName: 'Install dependencies'
- script: playwright install --with-deps
  displayName: 'Install Playwright browsers'
- script: pytest
  displayName: 'Run Playwright tests'
```

## GitLab CI

```yaml
stages:
  - test
tests:
  stage: test
  image: mcr.microsoft.com/playwright/python:v1.57.0-noble
  script:
    - pip install -r requirements.txt
    - pytest
```

## Jenkins

```groovy
pipeline {
   agent { docker { image 'mcr.microsoft.com/playwright/python:v1.57.0-noble' } }
   stages {
      stage('e2e-tests') {
         steps {
            sh 'pip install -r requirements.txt'
            sh 'pytest'
         }
      }
   }
}
```

## CircleCI

```yaml
executors:
  pw-noble-development:
    docker:
      - image: mcr.microsoft.com/playwright/python:v1.57.0-noble
```

## Docker

### Pull Image
```bash
docker pull mcr.microsoft.com/playwright/python:v1.57.0-noble
```

### Run (Trusted Code)
```bash
docker run -it --rm --ipc=host mcr.microsoft.com/playwright/python:v1.57.0-noble /bin/bash
```

### Run (Untrusted Sites)
```bash
docker run -it --rm --ipc=host --user pwuser --security-opt seccomp=seccomp_profile.json mcr.microsoft.com/playwright/python:v1.57.0-noble /bin/bash
```

### Build Custom Image
```dockerfile
FROM python:3.12-bookworm

RUN pip install playwright==1.57.0 && \
    playwright install --with-deps
```

### Remote Connection
```bash
# Start server
docker run -p 3000:3000 --rm --init -it mcr.microsoft.com/playwright:v1.57.0-noble \
  /bin/sh -c "npx -y playwright@1.57.0 run-server --port 3000 --host 0.0.0.0"
```

```python
# Connect from tests
from playwright.sync_api import sync_playwright
with sync_playwright() as p:
    browser = p.chromium.connect("ws://127.0.0.1:3000/")
```

### Docker Tips
- Use `--ipc=host` for Chromium (prevents OOM)
- Use `--init` to prevent zombie processes
- Use `--cap-add=SYS_ADMIN` for local dev if launch errors occur
- Pin to specific image versions
- Alpine Linux not supported (musl incompatibility)

## CI Tips

- **Don't cache browsers** - restore time ~ download time
- **Debug CI failures**: `DEBUG=pw:browser pytest`
- **Headed mode on Linux**: `xvfb-run pytest`
- **Tracing for CI**: `pytest --tracing=retain-on-failure`

## Available Image Tags
- `:v1.57.0` / `:v1.57.0-noble` - Ubuntu 24.04 LTS
- `:v1.57.0-jammy` - Ubuntu 22.04 LTS
