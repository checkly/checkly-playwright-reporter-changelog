# @checkly/playwright-reporter

> Official Playwright reporter for Checkly - Upload test results and assets automatically

[![npm version](https://badge.fury.io/js/@checkly%2Fplaywright-reporter.svg)](https://www.npmjs.com/package/@checkly/playwright-reporter)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

Seamlessly integrate Playwright test results with Checkly monitoring. Automatically upload test results, screenshots, videos, and traces to gain visibility into your application's health across test runs.

> **Important**: This package is currently in an public beta.  
[Contact the Checkly team](https://app.checklyhq.com/?support=true) for support or feedback.  
Always ensure you're using the latest version. Expect breaking changes, features in development, and possible bugs.

## Installation

```bash
npm install --save-dev @checkly/playwright-reporter
```

**Requirements:**
- Node.js >= 18.0.0
- Playwright >= 1.40.0
- A Checkly account ([sign up here](https://www.checklyhq.com/signup))

## Quick Start

### 1. Get Your Checkly Credentials

1. Log in to [Checkly](https://app.checklyhq.com)
2. Go to **Account Settings > API Keys** and create a new API key
3. Note your Account ID from /settings/account/general

### 2. Configure Playwright

Add the reporter to your `playwright.config.ts`:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  reporter: [
    ['json', { outputFile: 'test-results/playwright-test-report.json' }],
    ['list'],
    ['@checkly/playwright-reporter']
  ],
});
```

> **Note**: The JSON reporter must come first to generate the report that this reporter consumes.

### 3. Set Environment Variables

```bash
export CHECKLY_API_KEY=your_api_key_here
export CHECKLY_ACCOUNT_ID=your_account_id_here
```

### 4. Run Your Tests

```bash
npx playwright test
```

You'll see a direct link to your test results:

```
Running 5 tests using 5 workers


🔗 View test session: https://chkly.link/l/XSX35

  ✓  1 [chromium] › tests/01-always-passing.spec.ts:4:7 › Always Passing Tests › basic math assertion (5ms)
  ✓  2 [chromium] › tests/01-always-passing.spec.ts:33:7 › Always Passing Tests › async operation with timeout (104ms)
  ✓  3 [chromium] › tests/01-always-passing.spec.ts:8:7 › Always Passing Tests › string manipulation (7ms)
  ✓  4 [chromium] › tests/01-always-passing.spec.ts:14:7 › Always Passing Tests › array operations (20ms)
  ✓  5 [chromium] › tests/01-always-passing.spec.ts:21:7 › Always Passing Tests › object properties (11ms)

  5 passed (550ms)

======================================================

🦝 Checkly reporter: 0.1.0
🎭 Playwright: 1.56.0
📔 Project: chromium
🔗 Test session URL: https://chkly.link/l/XSX35

======================================================
```

## Configuration

### Environment Variables

The reporter uses environment variables for configuration:

| Variable | Description | Required |
|----------|-------------|----------|
| `CHECKLY_API_KEY` | Your Checkly API key | Yes* |
| `CHECKLY_ACCOUNT_ID` | Your Checkly account ID | Yes* |

\* Required for uploading results. Without these, the reporter creates a local ZIP file only.

### Configuration Options

You can also pass options directly in `playwright.config.ts`:

```typescript
['@checkly/playwright-reporter', {
  apiKey: process.env.CHECKLY_API_KEY,      // Not recommended, use env vars
  accountId: process.env.CHECKLY_ACCOUNT_ID, // Not recommended, use env vars
  dryRun: false,                            // Skip API calls, only create local ZIP
  outputPath: 'checkly-report.zip',         // Custom ZIP output path
  jsonReportPath: 'test-results/report.json', // Custom JSON report path
  testResultsDir: 'test-results'            // Custom test results directory
}]
```

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `dryRun` | `boolean` | `false` | Skip all API calls and only create local ZIP file |
| `apiKey` | `string` | - | Checkly API key (use env var instead) |
| `accountId` | `string` | - | Checkly account ID (use env var instead) |
| `outputPath` | `string` | `'checkly-report.zip'` | Path for the generated ZIP file |
| `jsonReportPath` | `string` | `'test-results/playwright-test-report.json'` | Path to JSON report |
| `testResultsDir` | `string` | `'test-results'` | Directory containing test results |

> **Security Note**: Always use environment variables for credentials. Never commit API keys to version control.

## How It Works

### Suite-Level Test Sessions

This reporter creates **one test session per test run**, not per individual test. The session is named after your project directory:

```
Directory: /Users/john/my-app
Session:   Playwright Test Session: my-app
```

All test results, screenshots, videos, and traces are bundled together in a single session for easy analysis.

### Flaky Test Detection

The reporter automatically detects flaky tests:

- **Flaky Test**: A test that failed initially but passed on retry
- **Degraded Session**: A session with flaky tests but no complete failures
- **Failed Session**: A session where at least one test never passed

### What Gets Uploaded

- ✅ Test results and status (passed/failed/flaky)
- ✅ Test execution duration
- ✅ Screenshots (on failure or explicit capture)
- ✅ Videos (full recordings)
- ✅ Traces (Playwright traces for debugging)
- ✅ Complete JSON test report

## Usage Examples

### Dry Run Mode (No API Calls)

Use dry run mode to create ZIP files without making API calls:

```typescript
export default defineConfig({
  reporter: [
    ['json', { outputFile: 'test-results/playwright-test-report.json' }],
    ['@checkly/playwright-reporter', {
      dryRun: true // Skip all API calls, only create local ZIP
    }]
  ],
});
```

Perfect for:
- Local testing and validation
- CI/CD pipelines without credentials
- Debugging ZIP file contents

### Local Development (No Upload)

Run tests locally without uploading to Checkly:

```bash
# Simply don't set the environment variables
npx playwright test
```

The reporter will create a local `checkly-report.zip` file silently.

### Conditional Dry Run (CI vs Local)

Automatically use dry run in CI, but enable API calls locally:

```typescript
export default defineConfig({
  reporter: [
    ['json', { outputFile: 'test-results/playwright-test-report.json' }],
    ['@checkly/playwright-reporter', {
      dryRun: !!process.env.CI // Dry run in CI, normal mode locally
    }]
  ],
});
```

This allows you to:
- Test locally with API credentials when available
- Run validation in CI without requiring secrets
- Keep ZIP files in CI for artifact upload

### GitHub Actions

```yaml
name: Playwright Tests
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 18

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright browsers
        run: npx playwright install --with-deps

      - name: Run Playwright tests
        env:
          CHECKLY_API_KEY: ${{ secrets.CHECKLY_API_KEY }}
          CHECKLY_ACCOUNT_ID: ${{ secrets.CHECKLY_ACCOUNT_ID }}
        run: npx playwright test
```

### Multiple Reporters

Combine with other Playwright reporters:

```typescript
export default defineConfig({
  reporter: [
    ['json', { outputFile: 'test-results/playwright-test-report.json' }],
    ['@checkly/playwright-reporter'],
    ['html', { outputFolder: 'playwright-report' }],
    ['list']
  ],
});
```

Made with ❤️ by [Checkly](https://www.checklyhq.com)
