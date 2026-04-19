---
name: vsix-ui-patcher
description: Use when the user wants to unpack, inspect, patch, validate, document, or repack a VSIX extension, especially when only the built bundle is available and UI changes must be made directly in packaged webview JavaScript or CSS. Triggers on .vsix files, Open VSX or Marketplace extension packages, unpack/repack requests, built webview patching, UI tweaks, usage stats, progress rings, CSS/JS hotfixes, or white-screen regressions after bundle edits.
---

# VSIX UI Patcher

## Overview

Use this skill to modify packaged VSIX extensions safely when the original source project is unavailable. The workflow favors minimal edits to built assets, especially `extension/webview/*.js` and `extension/webview/*.css`, followed by verification and correct repackaging.

## Use This Skill When

- The user gives you a `.vsix` file and wants it unpacked or repacked.
- The user wants UI changes inside a packaged extension.
- The only editable code is a built or minified bundle.
- The user needs a patch note or learning note while preserving the upstream license.
- A previous bundle edit caused a white screen or broken UI.

## Workflow

### 1. Identify version and license first

- Read `extension/package.json` and `extension.vsixmanifest` before patching.
- Preserve upstream license and legal notices.
- Do not present a patched bundle as an official release.
- If the user wants documentation, do not overwrite the extension's official README unless they explicitly ask for that exact behavior.

### 2. Unpack or locate the unpacked directory

- Treat `.vsix` as a zip-compatible archive.
- Prefer unpacking into a sibling directory named after the package.
- Keep the original `.vsix` untouched when possible.

### 3. Find the real runtime assets

- Use `rg` to find the webview entry points and target strings.
- Common paths are `extension/webview/index.js` and `extension/webview/index.css`.
- Check whether the JS is line-broken or effectively one huge line before deciding how to patch it.

### 4. Reuse existing data before adding new plumbing

- Search the bundle for existing state or prop names before inventing new ones.
- For UI statistics, first look for fields already flowing through the frontend.
- Only add new message passing or backend plumbing if the data truly does not exist.

### 5. Patch minimally

- Change only the smallest render function, helper, or style block needed.
- Keep CSS additions local and namespaced.
- In minified bundles, avoid short helper names because symbol collisions can break runtime.
- If text and graphic disagree, patch the rendering primitive itself rather than only changing labels around it.

### 6. Validate immediately

- Run syntax checks such as `node -c extension/webview/index.js`.
- Confirm that the intended replacement actually landed.
- If possible, verify the live UI with the user or with a screenshot.
- Treat a white screen as a likely syntax error, bad replacement, or symbol collision until proven otherwise.

### 7. Repack from the correct directory

- Repack from the unpacked root, not from the parent directory.
- Example:

```bash
cd /path/to/unpacked-extension
zip -qr '/path/to/output-patched.vsix' .
```

- If you zip from the wrong directory level, the VSIX may gain an extra folder layer and fail to install correctly.

### 8. Optional notes only when requested

- If the user asks for patch documentation, create a separate local note unless they explicitly want a different filename.
- Include version, modified files, what changed, validation steps, repack command, and a learning-only notice when appropriate.
- Do not create extra documentation files if the user has said not to.

## Built-Bundle Rules

- Search first, patch second.
- Prefer exact string replacement for tiny hotfixes in heavily bundled code.
- Never assume a visual indicator is percentage-driven just because nearby text is correct.
- Avoid helper names like `a1`, `mT0`, or other short mixed-case names inside minified bundles.
- For visual meters, prefer deterministic SVG or CSS that is directly driven by the real percentage.

## Validation Checklist

- Run `node -c` on modified JavaScript bundles when possible.
- Check that modified functions or marker strings exist exactly as intended.
- Confirm that the repacked VSIX file is produced.
- If the UI breaks after patching, inspect the most recent helper names and replacements first.

## Example Requests

- "Unpack this VSIX and add a usage cost display."
- "Patch a VS Code extension webview even though only the built bundle is available."
- "Fix this progress ring in a packaged extension; it always looks half full."
- "Repack this patched VSIX and keep the upstream license notice."

## Reference

- For a real case study covering usage stats, white-screen debugging, progress-ring repair, and repackaging, read [references/claude-code-2.1.100-case-study.md](references/claude-code-2.1.100-case-study.md).
