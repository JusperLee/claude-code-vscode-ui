# Claude Code VSIX Learning Notes

[中文说明](./README-cn.md)

![Claude Code Usage Stats](page.png)   

## Download the Matching Version

The patch in this directory targets `Anthropic/claude-code` version `2.1.100`.

You can download the corresponding original extension version from Open VSX here:

- https://open-vsx.org/extension/Anthropic/claude-code

When using this patch as a reference, please make sure the base extension version matches `2.1.100`.

## Patch Summary

This local patch currently includes these visible changes:

- Added a `Context` statistic to the usage area.
- Added a `Cost` statistic to the usage area.
- Fixed the circular progress indicator so it now reflects the real percentage visually.
- Added a hover popup for more detailed usage information.
- Fixed the earlier white-screen issue caused by helper-name collisions in the minified bundle.

## Notice

This document is a local learning note for unpacking, studying, and patching the VSIX package. It does not replace the official documentation and does not change the original license terms.

This repository copy and the notes below are for learning and research only. Do not treat them as an official release, official support material, or a redistributable commercial package.

## Original License Notice

The original extension license notice is declared in [`./extension/package.json`](./extension/package.json):

`© Anthropic PBC. All rights reserved. Use is subject to the Legal Agreements outlined here: https://code.claude.com/docs/en/legal-and-compliance.`

These local notes follow the original License / Legal Agreements and are provided only for study purposes.

## What Was Changed

This patch works on the unpacked VSIX contents and mainly adjusts the usage display in the Claude Code UI:

- Added a `Context` statistic.
- Added a `Cost` statistic.
- Fixed the circular progress indicator so it now renders the real percentage instead of looking like a mostly fixed half-ring.
- Reused the existing usage widget instead of creating a separate panel.
- Added hover details so usage information is easier to inspect.

Relevant modified files:

- [`./extension/webview/index.js`](./extension/webview/index.js)
- [`./extension/webview/index.css`](./extension/webview/index.css)

## Modification Approach

Because the available artifact is a built VSIX package rather than the original source project, the patch follows a minimal-change approach:

1. Unpack the VSIX and inspect the built frontend bundle directly.
2. Locate the rendering logic for the existing usage widget.
3. Reuse existing state data instead of introducing new message channels or new state plumbing.
4. Apply only local display and style changes to reduce the risk of broader regressions.

The patch reuses these frontend data fields that were already present:

- `totalTokens`
- `totalCost`
- `contextWindow`
- `maxOutputTokens`

So this is mainly a frontend exposure patch, not a new backend accounting feature.

## White Screen Issue and Fix

The first patch briefly caused a white screen.

The root cause was a naming collision inside the minified frontend bundle. Short helper names added during patching conflicted with existing minified symbols and broke runtime execution.

The fix was:

- Stop using short helper names.
- Replace them with sufficiently unique names to avoid symbol collisions.
- Use the final helper names `formatUsageTokensP2QnnQ` and `formatUsageCostP2QnnQ`.

After that fix, the UI rendered normally again and the package was rebuilt successfully.

There was also a separate display correction for the usage circle:

- The earlier ring-like indicator could appear visually misleading and did not reflect the actual percentage precisely.
- The final version replaces that fixed-shape behavior with an SVG circular progress ring driven directly by the real usage percentage.
- As a result, values such as `5%`, `23%`, or `68%` now map to visibly different ring lengths.

## How to Unpack

A `.vsix` file can essentially be treated as an archive.

Possible approaches:

- Extract the `.vsix` as a zip archive into a directory.
- Open it with any archive tool that supports zip-compatible packages.

The unpacked working directory used here is:

`Anthropic.claude-code-2.1.100@darwin-arm64`

## How to Repack

After making changes, repack from the unpacked root directory itself, not from its parent directory.

Change into the unpacked directory:

```bash
cd Anthropic.claude-code-2.1.100@darwin-arm64
```

Then run the repack command:

```bash
zip -qr 'Anthropic.claude-code-2.1.100@darwin-arm64-patched-fixed.vsix' .
```

Notes:

- `zip -q` keeps output quiet.
- `-r` recursively packages everything under the current directory.
- The trailing `.` is important because it packages the contents of the unpacked root.

If you run the command from the wrong directory, the resulting VSIX may gain an extra folder level and fail to install or run correctly.

## Output Artifacts

Relevant artifacts in this workspace:

- Original unpacked directory: `Anthropic.claude-code-2.1.100@darwin-arm64`
- Early test build with the white-screen issue: `Anthropic.claude-code-2.1.100@darwin-arm64-patched.vsix`
- Fixed working build: `Anthropic.claude-code-2.1.100@darwin-arm64-patched-fixed.vsix`

## Additional Notes

This document does not replace the official README. The official extension README remains [`./extension/README.md`](./extension/README.md).

If more UI patching is needed later, it is safer to:

- Search first to see whether the needed state already exists.
- Reuse the original component or layout position whenever possible.
- Avoid short or generic helper names inside minified bundles.
- Run at least a syntax check and a real UI-open verification after each patch.

## Learning Only

This supplemental document and the local patch are for learning, analysis, research, and personal experimentation only. Please comply with the original copyright notice and Anthropic's Legal Agreements.
