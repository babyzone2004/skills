---
name: wechat-miniprogram-e2e
description: Run and debug real WeChat Mini Program end-to-end tests through WeChat DevTools and miniprogram-automator. Use when Codex must verify mini program UI flows, diagnose DevTools automation hangs, distinguish HTTP service ports from auto WebSocket ports, handle compile/runtime readiness, or report whether WeChat Cloud deployment actually succeeded.
---

# WeChat Mini Program E2E

## Objective

Use the real WeChat DevTools runtime as the source of truth for mini program UI verification. Do not treat model tests, page model tests, or a successful connection to DevTools as proof that the user flow works.

## Preconditions

- Confirm the project path for the mini program, usually `apps/miniprogram`.
- Confirm WeChat DevTools CLI exists, usually `/Applications/wechatwebdevtools.app/Contents/MacOS/cli`.
- Ensure DevTools service port is enabled. The HTTP port such as `14122` is not the automator WebSocket port.
- If connecting to an existing DevTools window, read the `autoPort` value from the DevTools address bar and use `WECHAT_DEVTOOLS_WS_ENDPOINT=ws://127.0.0.1:<autoPort>`.
- If no stable window exists, let `miniprogram-automator.launch` start an isolated auto port.

## Core Workflow

1. Extract acceptance criteria from the PRD or task before changing tests.
2. Write or update a WeChat DevTools E2E test that covers the full user path, not just routing or page data.
3. Open or launch DevTools automation.
4. Compile once in DevTools before trusting automator commands. The simulator must show the mini program page, not the DevTools welcome screen.
5. Run the smallest failing WeChat E2E test first.
6. If it fails, preserve the exact failure and localize by layer:
   - connection or wrong port
   - runtime not compiled or stuck on welcome screen
   - page loading stuck
   - navigation race
   - element tap instability
   - product logic bug
7. Fix the root cause, then rerun the focused WeChat E2E.
8. Run the complete WeChat E2E suite, normal tests, typecheck, and build.
9. Commit only relevant files. Do not include unrelated local config, `.DS_Store`, or user edits.

## Automation Rules

- Prefer stable helpers:
  - `withTimeout()` around every DevTools operation.
  - `waitForPage(expectedPath)` instead of reading `currentPage()` immediately after navigation.
  - serial execution for DevTools tests because one DevTools session is shared.
- Verify visible UI before triggering actions: title, button text, list text, status text.
- When `Element.tap()` is flaky on custom `view` buttons, first assert the element exists and has the right text, then call the same page method with `Page.callMethod()`. Still assert the resulting route and rendered UI.
- Use `miniProgram.evaluate()` for state reset or diagnostic probes only. Do not replace the whole user path with state mutation.
- Avoid monkeypatching global `wx` methods unless diagnosing a navigation issue. If you monkeypatch, restart or recompile DevTools before final tests.

## Debugging Playbook

When a command hangs:

1. Check listening ports:
   ```bash
   lsof -nP -iTCP -sTCP:LISTEN | rg "wechatweb|14122|auto"
   ```
2. Inspect the DevTools window. If the simulator shows the welcome page, click `编译` and wait for `pages/home/index`.
3. If `evaluate()` hangs, the app runtime is not ready; compile or restart DevTools before debugging product code.
4. If navigation calls happen but callbacks do not fire, suspect a navigation race or stale runtime. Add a short page settle wait and rerun from a clean compile.
5. If the page is stuck on `加载中`, verify cloud calls have local fallback or timeout behavior for non-deployed environments.
6. If a previously good auto port stops responding, close/reopen the project or `cli quit`, then open, compile, and enable auto again.

## Deployment Verification

- If the user asks to deploy, actually run the project deployment commands.
- Do not claim deployment success when scripts are placeholders or credentials are missing.
- Check for required environment variables or project config, such as cloud environment ID, upload credentials, appid, and template IDs.
- Report deployment as one of:
  - succeeded with command output
  - not attempted because a required input is missing
  - attempted but blocked by a specific script or credentials error

## Done Criteria

- Real DevTools E2E covers the critical user path end to end.
- Failures were diagnosed at the correct layer, not hidden by retries.
- `npm run test:e2e:wechat` or the project equivalent passes against a real DevTools runtime.
- Typecheck, existing tests, and build pass, or unrelated failures are explicitly separated.
- Final response includes what UI paths were covered, exact verification commands, deployment result, and review findings.
