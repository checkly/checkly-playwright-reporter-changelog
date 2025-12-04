# @checkly/playwright-reporter

> Official Playwright reporter for Checkly - Upload test results and assets automatically

[![npm version](https://badge.fury.io/js/@checkly%2Fplaywright-reporter.svg)](https://www.npmjs.com/package/@checkly/playwright-reporter)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

Seamlessly integrate Playwright test results with Checkly monitoring. Automatically upload test results, screenshots, videos, and traces to gain visibility into your application's health across test runs.

> [!NOTE]  
> Find the complete documentation in [the Checkly Playwright Reporter docs](https://www.checklyhq.com/docs/detect/testing/playwright-reporter/).

> [!WARNING]
> This package is currently in public beta. [Contact the Checkly team](https://app.checklyhq.com/?support=true) for support or feedback.  
> Always ensure you're using the latest version. Expect breaking changes, features in development, and possible bugs.

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

-----

## Official docs

Learn more about [the Checkly Playwright Reporter in the official documentation on `checklyhq.com`](https://www.checklyhq.com/docs/detect/testing/playwright-reporter/).
