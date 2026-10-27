# Chrome Mobile iOS stream bug probe

Standalone vanilla HTML/JavaScript investigation of [React Router #15486](https://github.com/remix-run/react-router/issues/15486) and [PR #15487](https://github.com/remix-run/react-router/pull/15487). No dependencies, build step, framework, external assets, analytics, or API patches.

**Live page:** https://brophdawg11.github.io/repro-chrome-mobile-ios-stream-bug/

## Test on a device

1. Open the live page directly in Chrome on iOS, Safari on the same device, and any affected in-app browser. Tests run automatically on page load.
2. Enter the exact iOS version and browser/app version in the notes field. The user agent alone may not identify either accurately.
3. Copy the report. It includes observed results, full error stacks, early initialization state, user agent, and API function sources.
4. If any check fails, follow its fresh-page link to run it alone in a new document. Compare the isolated results with the combined run.

| Check | What it exercises |
| --- | --- |
| Original | The original React Router stream initialization in an early inline script, a separate inline transfer script, and byte consumption |
| Encoder only | `TextEncoderStream` construction and string writes, without `pipeThrough` |
| Pipe only | `ReadableStream.pipeThrough(new TransformStream())`, without `TextEncoderStream` |
| Replacement | The proposed `TextEncoder` controller wrapper, without either avoided API |

Each round trip checks multiple chunks containing ASCII, accented text, CJK, and an emoji. Reads have a four-second diagnostic timeout. A timeout is reported separately from an exception or an unsupported API; it is not evidence of a stack overflow.

## Interpretation

This is a **candidate reproduction**, not a claim that the reported iOS failure has been independently reproduced. A passing desktop run establishes only that the probe works there. A passing run on the affected device means this minimal page did not reproduce the failure in that environment.

The page does not load React Router or test actual application hydration. It isolates the stream expression and transfer contract from the issue. It also does not simulate recursive browser hooks, infer an injected script's identity from minified names, or modify any built-in API. Function `toString()` output is only a diagnostic hint and cannot prove whether an API was patched.

An original failure together with a passing replacement supports the workaround. It does not establish the underlying cause. Inspect the constructor-only and pipe-only results to narrow that down. Browser scripts and their enabled features may differ by app, version, settings, and navigation context.

Results remain in the page until manually copied. No report is submitted automatically.

## Run locally

Open `index.html` directly, or serve this directory with any static HTTP server. For example, with Python installed:

```sh
python3 -m http.server 8000
```

Then open http://localhost:8000/. The page itself requires no installation.

## GitHub Pages

Publish from the `main` branch, `/ (root)` directory. `.nojekyll` keeps the source as static files. No custom Actions workflow or dependency installation is needed.
