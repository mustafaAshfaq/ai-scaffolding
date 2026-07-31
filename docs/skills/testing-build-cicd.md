# Testing, Build Tools & CI/CD

## What it is

Modern streaming front-ends use **Webpack**, **Vite**, or framework bundlers (Next.js uses Turbopack/Webpack) to bundle player SDKs and split code. **Jest** runs unit tests; **React Testing Library (RTL)** tests components from the user's perspective; **Playwright** runs end-to-end browser tests including real playback flows. **CI/CD pipelines** (Azure DevOps, GitHub Actions) automate lint, test, build, and deploy for production-ready releases.

## Why it matters for this role

The job requires hands-on experience with Webpack, Vite, Rollup, CI/CD pipelines, and automated testing (Jest, RTL, Playwright) — ensuring high-quality A/B experiments and production-ready releases. Your Azure DevOps background maps directly; learn Jest/RTL/Playwright as the Angular Karma/Jasmine equivalent.

## Core concepts checklist

- [ ] Next.js bundling: automatic code splitting per route; `next/dynamic` for manual splits
- [ ] Webpack concepts: entry, output, loaders, plugins (Angular CLI uses Webpack under the hood)
- [ ] Vite: dev server with ESM; Rollup for production — know when teams choose it over CRA/Next
- [ ] Tree shaking unused SDK exports to reduce player bundle size
- [ ] Jest: `describe`, `it`, `expect`, mocks, `beforeEach`
- [ ] RTL: `render`, `screen.getByRole`, `userEvent.click` — test behavior not implementation
- [ ] Mock Shaka Player in unit tests; never load real streams in Jest
- [ ] Playwright: `page.goto`, locators, assertions, video recording on failure
- [ ] Playwright E2E: real `<video>` element `paused` property check after play
- [ ] Azure DevOps: pipelines YAML, stages, jobs, artifacts, deployment gates
- [ ] GitHub Actions equivalent: `on: pull_request`, matrix builds, secrets
- [ ] Lighthouse CI in pipeline for performance regression gates

## Angular → React testing map

| Angular | React |
|---------|-------|
| Karma + Jasmine | Jest |
| TestBed.configureTestingModule | RTL `render(<Component />)` |
| ComponentFixture.detectChanges | RTL auto re-render on state change |
| Protractor / Cypress | Playwright |
| HttpClientTestingModule | `msw` or `jest.mock('fetch')` |
| `ng test --code-coverage` | `jest --coverage` |

## Learning resources

| Resource | Type | URL |
|----------|------|-----|
| Jest docs | Official | https://jestjs.io/docs/getting-started |
| React Testing Library | Official | https://testing-library.com/docs/react-testing-library/intro/ |
| Playwright docs | Official | https://playwright.dev/docs/intro |
| Next.js — Testing | Guide | https://nextjs.org/docs/app/building-your-application/testing |
| Azure Pipelines YAML schema | Official | https://learn.microsoft.com/en-us/azure/devops/pipelines/yaml-schema |
| GitHub Actions docs | Official | https://docs.github.com/en/actions |
| Lighthouse CI | Tool | https://github.com/GoogleChrome/lighthouse-ci |
| MSW (Mock Service Worker) | Tool | https://mswjs.io |

## Sample Playwright playback test

```typescript
import { test, expect } from '@playwright/test';

test('catalog to playback', async ({ page }) => {
  await page.goto('/');
  await page.getByRole('link', { name: /cardiology fundamentals/i }).click();
  await page.getByRole('button', { name: /play/i }).click();
  const video = page.locator('video');
  await expect(video).toBeVisible();
  await page.waitForFunction(() => {
    const v = document.querySelector('video');
    return v && !v.paused && v.readyState >= 3;
  }, { timeout: 30000 });
});
```

## Azure DevOps pipeline sketch (CareStream)

```yaml
trigger:
  branches: [main, develop]

stages:
  - stage: Build
    jobs:
      - job: WebApp
        steps:
          - script: pnpm install && pnpm lint && pnpm test && pnpm build
          - publish: apps/web/.next
      - job: CatalogApi
        steps:
          - script: pnpm --filter catalog-api test && pnpm --filter catalog-api build
      - job: AnalyticsApi
        steps:
          - script: pnpm --filter analytics-api test && pnpm --filter analytics-api build

  - stage: E2E
    dependsOn: Build
    jobs:
      - job: Playwright
        steps:
          - script: pnpm exec playwright test

  - stage: Deploy
    dependsOn: E2E
    condition: eq(variables['Build.SourceBranch'], 'refs/heads/main')
    jobs:
      - deployment: Production
        environment: carestream-prod
```

## Hands-on exercises

1. **Jest unit test:** Test player state reducer — given `BUFFERING` action, state becomes `{ status: 'buffering' }`.
2. **RTL test:** Render `PlayerControls`; click play button; assert callback invoked.
3. **Mock Shaka:** Jest mock `shaka-player`; verify `load()` called with manifest URL on mount.
4. **Playwright E2E:** Full catalog → watch → video playing flow against local dev servers.
5. **Azure pipeline:** Create `azure-pipelines.yml` running lint + test for all monorepo packages.
6. **GitHub Actions doc:** Write one-page comparison of your Azure pipeline vs equivalent GitHub Actions YAML.

## Interview / on-the-job topics

- Why mock Shaka in unit tests but use real streams in E2E only
- Code splitting player SDK: measuring bundle size with `@next/bundle-analyzer`
- Flaky E2E tests with network-dependent playback — retry and timeout strategies
- CI parallelization: running Playwright sharded across agents
- Deployment gates for A/B experiment readiness (feature flags + metrics baseline)
- Mapping your OpenShift/Kubernetes deploy experience to static front-end + API services

## Related skills

- [typescript-react-nextjs.md](typescript-react-nextjs.md) — Next.js test setup
- [video-player-sdks.md](video-player-sdks.md) — Mocking Shaka
- [performance-qos-analytics.md](performance-qos-analytics.md) — Lighthouse CI
- [angular-to-react-bridge.md](angular-to-react-bridge.md) — Karma/Jasmine migration
