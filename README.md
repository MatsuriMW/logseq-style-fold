# Logseq Style Fold

> **Your lists stay folded the way you left them — written the way Logseq writes it.**

An Obsidian plugin that remembers how each list item was collapsed, by writing a Logseq block property into the note instead of keeping the state somewhere outside it.

**English** · [简体中文](./README.zh-CN.md)

## Why

Logseq stores a folded block as a `collapsed:: true` property right under it. Obsidian does not understand that line: a vault that came from Logseq shows it as plain text, and Obsidian's own folds are forgotten as soon as the note is closed.

This plugin reads and writes that marker. **The Logseq style is the default**, so notes stay compatible with Logseq. If you prefer something else, set your own marker — for example an Obsidian comment `%% collapsed:: true %%`, or any line of text you like.

## What it does

Fold something in a list, close the note, come back tomorrow — it is still folded.

```markdown
- Weekly meeting
	collapsed:: true
	- Data sources
	- Definitions check
- To do
	- Not folded yet
```

The marker sits one indent level in, right under the item it belongs to — the same place Logseq puts it. Whatever style you choose, the plugin hides it **everywhere you look**:

- **Live Preview and Source mode** — the line is removed from the view, newline and all. It is not dimmed or squeezed; it is gone, and the cursor treats the spot as one atomic step, so you cannot land inside it while typing.
- **Reading view** — the marker is stripped from the rendered note (including Dataview's inline-field rendering of `collapsed:: true`). A `%% … %%` comment is never rendered by Obsidian in the first place.

Nothing else about your note changes.

## Markers

| Style | Written as | Notes |
|---|---|---|
| Logseq (default) | `collapsed:: true` | Same as Logseq; notes round-trip between the two apps |
| Obsidian comment | `%% collapsed:: true %%` | Invisible even without the plugin, in any Obsidian install |
| Custom | any single line, e.g. `[folded]` | Written as-is under each folded item |

`collapsed::` lines are always recognised, with or without `%%`, whichever style you write — so switching styles never loses existing folds. `collapsed:: false` (which Logseq can leave behind) is treated as unfolded and is reused when you fold that item again.

## How it folds

The folding itself is not drawn by this plugin — it calls the same CodeMirror fold that ⌥S uses. You get Obsidian's own arrow, the `…` at the end of the line, the same click to unfold.

Marker and fold are kept in sync in both directions:

- A marker in the file folds that item when the note opens, and again whenever another plugin or a sync rewrites the file and the fold gets lost. This direction only ever folds; it never unfolds something for you.
- Fold something yourself — ⌥S, the arrow, whatever you already use — and the marker is written for you. Unfold it and the marker is removed. Typing and rewrites of the file don't count as folding.
- Another plugin that folds on its own can wrap that in `app.plugins.plugins["logseq-style-fold"].quietly(() => …)`, and those folds are not recorded as yours.

## Why the state lives in the file

A JSON file in the plugin folder, keyed by line numbers or text hashes, falls apart on a note that keeps being rewritten: line numbers shift the moment anything is inserted above, and hashes break when the wording changes. A line written into the note travels with its own content through git, sync, export and any rewrite that keeps the block itself.

## Commands

| Command | What it does |
|---|---|
| Fold or unfold the list item at the cursor | Toggle the item the cursor is on (an item without children cannot be folded) |
| Fold every list item in this note | Fold everything at once |
| Unfold every list item in this note | Unfold everything |
| Fold only top-level list items | Fold the outermost items, leaving inner levels as they are |
| Remove all fold markers from this note | Delete every marker — back to the text as it was before the plugin |
| Show or hide fold markers | Show marker lines as plain text, for checking what is in the file |

Bind any of them to a hotkey in **Settings → Hotkeys**.

## Settings

| Setting | Default | What it does |
|---|---|---|
| Marker style | Logseq | Logseq `collapsed:: true`, Obsidian comment `%% collapsed:: true %%`, or a custom line |
| Show a fold button on hover | On | An unfolded item shows a small collapse button at the end of its line while hovered |
| Show descendant count on the fold button | On | `⌄ 3` instead of `⌄` |
| Fold in Reading view | On | Honour the markers in Reading view, with a clickable badge to unfold |
| Fold in Source mode | Off | Source mode hides the marker either way; turn this on to fold there too |
| Show fold markers | Off | Show marker lines as plain text for checking |

## Installing

Copy `main.js`, `manifest.json` and `styles.css` into `<vault>/.obsidian/plugins/logseq-style-fold/`, reload Obsidian, and enable **Logseq Style Fold** in **Settings → Community plugins**. Or add it through [BRAT](https://github.com/TfTHacker/obsidian42-brat) from `MatsuriMW/logseq-style-fold`.

## Notes and limits

- Outside Obsidian, `collapsed:: true` shows up as text — which is exactly what Logseq expects. Pick the comment style if you want it invisible in other Obsidian installs without the plugin too.
- Markers survive any tool that keeps the block they belong to. A rewrite that deletes or re-words the block itself loses its fold state.
- Code blocks, callouts, ordered lists and checklists are handled. Lists inside tables are not folded.
- Markers are matched by the item's own text fingerprint within its nesting level, so two identical items at the same depth in one note share a fold state.
- The plugin only ever writes its own marker lines; nothing else in a note is modified.
