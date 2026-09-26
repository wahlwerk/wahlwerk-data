# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project state

This repository is **nearly empty**: `README.md`, `LICENSE` and one party registry,
`parties/de/de.bund.json`. There are no election bundles, scripts, tests, build tooling or
dependencies yet, so there are no commands to run. Do not invent a layout; build only what
is asked for. The rest of this file (apart from "Party registries" below) is the
role the sibling repos assign to this one, taken from `../wahlwerk/README.md`,
`../wahlwerk/CLAUDE.md` and `../wahlwerk-execute/CLAUDE.md`. Check those before relying
on it, since they are the source of truth and may have moved on.

## Party registries

`parties/<country>/<scope>.json` lists the parties for one scope; the file name is the
scope identifier (`de.bund`). Layout:

```json
{
  "schema": 1,
  "name": "de.bund",
  "description": "...",
  "parties": {
    "cdu": { "name": "Christlich Demokratische Union Deutschlands", "short_name": "CDU" }
  }
}
```

- `schema` is required. The engine reads only the versions it declares
  (`wahlwerk.io.parties.SCHEMAS`); a format change bumps it.
- `parties` is keyed by the party slug (lowercase ASCII, hyphens, umlauts transliterated:
  `gruene`, `team-todenhoefer`); the slug is not repeated inside the entry.
- An entry has `name` (required) and `short_name` (optional), both the official
  spellings in correct German (UTF-8, not escaped). No other fields: the engine rejects
  unknown ones. There is no `tags` field yet.
- Files are UTF-8 with LF line endings and 2-space indentation.
- The engine reads these with `PartyRegistry.from_json(path)` (reader in
  `wahlwerk/src/wahlwerk/io/parties.py`).

## Role among the sibling repositories

```
CODE/wahlwerk_/
  wahlwerk/          the engine       rules-as-code for German electoral law (Python, uv)
  wahlwerk-data/     the archive      <- you are here
  wahlwerk-execute/  notebooks        depends on both
```

- This repo is the **archive of normalised election bundles**: every Bundestagswahl since
  1949, the sixteen Länder, kommunal. It grows per election and exists separately so the
  engine's clone never bloats.
- The dependency is one way: the engine does **not** depend on this repo, and its tests
  pass with the archive absent. The real archive is meant to be exercised by
  **this repo's own tests**.
- The engine locates the archive in this order, first hit wins: `$WAHLWERK_DATA`, then
  `../wahlwerk-data` (sibling clone, searched upwards from the cwd), then
  `~/.cache/wahlwerk/data`. Keep the repo usable as a plain sibling clone.

## Bundle requirements stated by the engine

- Each bundle has an `election.toml` carrying `schema = N`. The engine declares which
  schema versions it reads, so a format change must bump `schema` (a loud break, not a
  silent mis-parse).
- `election.toml` carries source metadata: publisher, title, URL, licence and
  attribution (the engine keeps these on `Bundle.source`). Sources are recorded with URL,
  retrieval date and SHA-256 so a bundle is reproducible without committing a
  multi-megabyte original.
- Engine models use `extra="forbid"`: an unexpected or misspelled column is a hard error.
  Vote and seat counts are non-negative integers and shares are exact fractions; never
  store floats for them.
- Identifiers are lowercase ASCII slugs joined with dots, narrowest scope last:
  `de.bund.wk.001`, `de.bund.bundestag`, `de.by.landtag`.
- The repo **tags releases**; an analysis pins an engine version and a data tag together.

## Language

English for code and structure; German terms of art are kept (`Zweitstimme`,
`Wahlkreis`). Identifiers are ASCII-transliterated (`Aufloesung`); prose uses correct
German.

## Licence discrepancy

`LICENSE` is GPL-3.0, but the sibling repos describe this archive as **dl-de/by-2-0**
(Datenlizenz Deutschland, which requires attribution). Raise this with the user before
adding data or licence text rather than resolving it silently.
