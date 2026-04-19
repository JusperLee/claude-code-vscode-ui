# Claude Code 2.1.100 VSIX Case Study

## Scenario

This case study came from patching `Anthropic/claude-code` version `2.1.100` from a packaged VSIX, without access to the original source repository.

Primary goals:

- unpack the VSIX
- add `Context` and `Cost` statistics to the existing usage area
- fix a broken-looking circular progress indicator
- preserve upstream license and legal notices
- repack a working VSIX

## Relevant Files

- `extension/package.json`
- `extension/README.md`
- `extension/webview/index.js`
- `extension/webview/index.css`

## Data That Already Existed

The frontend already had the usage data needed for the UI patch:

- `totalTokens`
- `totalCost`
- `contextWindow`
- `maxOutputTokens`

Key lesson:

- The safest patch was to expose data that already existed in frontend state.
- No new backend accounting path was needed.

## UI Patch Strategy

Instead of adding a brand new panel, the patch reused the existing usage widget and extended it to show:

- `Context`
- `Cost`
- a hover popup with more detail

Why this worked well:

- less layout risk
- less state wiring
- fewer chances to break unrelated UI

## White-Screen Regression

An early patch caused a white screen.

Root cause:

- new helper names added to the minified bundle were too short
- they collided with existing minified symbols
- runtime behavior broke even though the change looked small

Fix:

- replace short helper names with globally distinctive names
- final working names were `formatUsageTokensP2QnnQ` and `formatUsageCostP2QnnQ`

Key lesson:

- in minified bundles, helper naming is a runtime safety issue, not just style

## Circular Progress Indicator Bug

The usage text could show values like `5%`, but the circle still looked like a mostly fixed half-ring.

Root cause:

- the old visual implementation used fixed shape buckets rather than a true percentage-driven ring
- the label and the icon were not driven by the same logic

Final fix:

- replace the old bucketed ring behavior with an SVG circular progress ring
- drive the ring directly from the real `percentage`
- use `strokeDasharray` and `strokeDashoffset` so values such as `5%`, `23%`, and `68%` produce visibly different arc lengths

Key lesson:

- if the text is correct but the icon is wrong, patch the rendering primitive itself

## Validation Used

- `node -c extension/webview/index.js`
- direct inspection of the modified function after replacement
- user confirmation that the UI recovered after the white-screen fix

Recommended additional validation whenever possible:

- open the extension UI after each risky change
- compare screenshots before and after

## Repackaging

The working VSIX was repacked from the unpacked root directory:

```bash
cd Anthropic.claude-code-2.1.100@darwin-arm64
zip -qr 'Anthropic.claude-code-2.1.100@darwin-arm64-patched-fixed.vsix' .
```

Key lesson:

- repack from the unpacked root itself
- repacking from the parent directory can introduce an extra folder layer and break installation

## License Handling

The upstream package declared its own license and legal terms in `extension/package.json`.

Safe practice:

- preserve upstream notices
- make it clear that local notes or patches are for learning or local experimentation only when appropriate
- do not replace the official extension README unless the user explicitly asks you to

## Reusable Lessons

- Read `package.json` before patching.
- Prefer existing frontend state over new data plumbing.
- Make the smallest possible bundle change.
- Avoid short helper names in minified code.
- Validate immediately after every risky replacement.
- Repack only from the unpacked root.
