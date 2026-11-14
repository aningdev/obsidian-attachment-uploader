# AGENTS.md

Desktop-only Obsidian plugin that uploads note attachments and rewrites their references to the returned destination URL.

## Commands

`bun` (lockfile `bun.lock`); `npm` works the same way.

- `bun run check` — format, lint, and type-aware lint. Does not rewrite files.
- `bun run fix` — the same checks, rewriting what they can.
- `bun run test` — unit tests.
- `bun run dev` — rebuild `main.js` while editing.
- `bun run build` — typecheck with `tsc`, then bundle `main.js`.
- `bun run version` — bump `manifest.json` and `versions.json` before a release.

## Where to read

- Terms: `docs/GLOSSARY.md`. Structure: `docs/ARCHITECTURE.md`.
- User docs: `README.md` and `README_ZH.md`. Release: `docs/release.md`.

## Constraints

When adding or changing user-facing text:

- Add the string to both `src/lang/locale/en.ts` and `src/lang/locale/zh-cn.ts`. English keys are canonical. `zh-cn` must satisfy every English key.
- Write for Obsidian users. Say what happens, what is kept, and what to do next. Name commands by their visible title. A button stays short. A setting description states scope and consequence. A result line carries counts and the reason.
- Use the terms in `docs/GLOSSARY.md`. Do not describe a recovery receipt as a backup or as remote deletion.

When changing an upload command, URL extraction, or the Markdown written for a destination URL:

- Run the command with `child_process.execFile` and `shell: false`. Replace exactly one standalone `%s` with the file path. Shell expansion and pipelines belong in a trusted executable wrapper, not in the plugin.
- A custom command prints one destination URL to stdout. Prefer a valid HTTP(S) result labelled `Output URL`, `Uploaded`, `result`, or `URL` (case-insensitive, inline after a colon or on the next line); otherwise use standalone URL lines. The chosen priority has exactly one distinct candidate. Identical repeats are allowed. Different results fail. Incidental prose URLs are ignored. Trim surrounding quotes.
- Normalize percent-encoding, Unicode, and whitespace in the URL written into Markdown. Wrap a URL that contains parentheses in `<...>`.
- Image extensions `avif`, `bmp`, `gif`, `jpeg`, `jpg`, `png`, `svg`, and `webp` produce `![]()` links. Other uploaded files produce `[]()` links.
- Parse standard inline and reference images, then merge Wiki embeds by UTF-16 offsets. Keep code, comments, raw HTML, image ranges, and definitions out of the Wiki scan. Resolve vault paths through Obsidian metadata. Do not treat every Obsidian extension or ordinary non-image link as an attachment.

When changing deletion of a local attachment:

- Scan at most 128 disk Markdown or Canvas files, 4 Mi UTF-16 code units, and 100 ms, yielding every 16 files. Open-view checks use their own 4 Mi / 100 ms budget. Text above 256 Ki code units is kept before normalization.
- An exhausted budget, or an uncertain read or parse, keeps the file. Elapsed-time checks do not preempt a read or synchronous work. The scan and the trash are not atomic. Repeat the reads for each deletion.

When a successful upload cannot be applied to the note:

- Write `~/.obsidian-attachment-uploader-recovery/upload-*/receipt.json` with `URL`, `sourcePath`, `filename`, and `timestamp`. Root and directory mode `0700`. File mode `0600`.
- Show a persistent notice with the path and URL, or the URL and the error if saving fails.
- Recovery is manual. A running command may finish after cancellation; receipt that URL when it was not applied.
- The current-note task shares one progress notice, cancellation controller, and summary.

When changing batch migration:

- Preview never uploads. Process only attachments referenced by Markdown, never every vault file. Skip open notes. Honor excluded folders. Always keep local source files.
- Re-check each item before work. Write a note through `Vault.process` only when its content, file identity, and open state are unchanged. Re-stat a local file immediately before replacing its link.
- Reuse across notes requires the same resolved vault path and unchanged size and mtime, or the same source URL for a remote image. Do not compare content hashes.
- Cancellation stops new work. An upload that is already running still returns its URL for a recovery receipt.
- Do not add scheduling, background scanning, or resume across Obsidian restarts.
- Closing the migration view does not own or cancel the task. Progress is not stored.

When changing the settings UI:

- Use declarative `getSettingDefinitions()` with groups. Use `Notice.messageEl`, `Vault.getFileByPath`, and `Vault.process`.
- `typescript/no-deprecated` is an error. Do not suppress it. Stay on APIs supported by the dependencies and by `minAppVersion` in `manifest.json`.

When changing a timeout, size, concurrency, interval, or recovery path that users can hit:

- Update the “Limits and defaults” section in both `README.md` and `README_ZH.md` in the same change. State what is skipped, kept, or bounded.

When changing the build or the release:

- Do not commit `main.js`. Tagging is described in `docs/release.md`.
