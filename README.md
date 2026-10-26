# wahlwerk-data

The data archive for [wahlwerk](../wahlwerk), the rules-as-code engine for German
electoral law. It is meant to hold normalised election bundles (every Bundestagswahl since
1949, the sixteen Länder, kommunal) and lives apart from the engine so the engine's clone
stays small.

## Contents

| Path | What it holds |
| --- | --- |
| `parties/de/de.bund.json` | Parties that have contested elections in Germany, federal or Land |
| `elections/de/landtag/be/2026/` | Wahl zum Abgeordnetenhaus von Berlin, 20 September 2026 (final result, per Wahlbezirk) |
| `elections/de/landtag/mv/1990/` | Landtagswahl Mecklenburg-Vorpommern, 14 October 1990 (final result, per Gemeinde, without Briefwahl) |
| `elections/de/landtag/mv/1994/` | Landtagswahl Mecklenburg-Vorpommern, 16 October 1994 (final result, per Wahlbezirk) |
| `elections/de/landtag/mv/1998/` | Landtagswahl Mecklenburg-Vorpommern, 27 September 1998 (final result, per Wahlkreis) |
| `elections/de/landtag/mv/2002/` | Landtagswahl Mecklenburg-Vorpommern, 22 September 2002 (final result, per Wahlbezirk) |
| `elections/de/landtag/mv/2006/` | Landtagswahl Mecklenburg-Vorpommern, 17 September 2006 (final result, per Wahlbezirk) |
| `elections/de/landtag/mv/2011/` | Landtagswahl Mecklenburg-Vorpommern, 4 September 2011 (final result, per Wahlbezirk) |
| `elections/de/landtag/mv/2016/` | Landtagswahl Mecklenburg-Vorpommern, 4 September 2016 (final result, per Wahlbezirk) |
| `elections/de/landtag/mv/2021/` | Landtagswahl Mecklenburg-Vorpommern, 26 September 2021 (final result, per Wahlbezirk) |
| `elections/de/landtag/mv/2026/` | Landtagswahl Mecklenburg-Vorpommern, 20 September 2026 (preliminary result, per Wahlbezirk) |
| `elections/de/landtag/st/2021/` | Landtagswahl Sachsen-Anhalt, 6 June 2021 (final result, per Wahlbezirk) |
| `elections/de/landtag/st/2026/` | Landtagswahl Sachsen-Anhalt, 6 September 2026 (final result, per Wahlbezirk) |

Bundles live at `elections/<country>/<body>/<land>/<year>/`.

## Election bundles

A bundle is a directory read by the engine with `wahlwerk.io.read_bundle(path)`:

```
elections/de/landtag/st/2026/
  election.toml        metadata and hierarchies
  tally.csv            the votes
  options.csv          every option and the party it stands for
  electoral.csv        the electoral hierarchy: Land, Wahlkreis
  administrative.csv   the administrative hierarchy (optional)
  as_found/            the source files as downloaded
    Ergebnisse_LT_2026.xlsx
```

Every bundle is schema 4 (decision 0009 in `.github-private`), converted from schema 3
on 2026-10-26 by `wahlwerk_data_processing.migrate` without loss: the same counts per
Wahlkreis and Land, the same Landtag seats from 2011 on, mandate by mandate.

- `election.toml`: `schema = 4`, a `[source]` table (`publisher`, `title`, `url`,
  `licence`, `attribution`, `retrieved`, `sha256` of the original file) and one
  `[hierarchies.<name>]` table per hierarchy, its levels broadest first, each with its
  tree in `<name>.csv`: `electoral` (`levels = ["land", "wahlkreis"]`), the one the laws
  count along, and the alternative ones (`levels = ["land", "kreis", "gemeinde"]`). The
  engine still reads schemas 1 to 3, whose `[levels]` table spelled the electoral
  hierarchy into the unit ids.
- `tally.csv`: the long vote table, one `TallyRow` per line, with the columns
  `unit,level,kind,section,channel,option,count`; a blank cell is an absent value.
  `unit` names its unit only (`de.st.gem.15081026.wbz.000001`: Land, Gemeinde,
  Wahlbezirk), never the Wahlkreis it lies in, which is law and is in `electoral.csv`.
  Only a unit the source numbers within its Wahlkreis keeps it in its id, since two units
  would otherwise share one (two Briefwahlbezirke 901 of Gemeinde 15088295 in
  Sachsen-Anhalt 2021: `de.st.wk.029.gem.15088295.wbz.000901` and `…wk.034…`). `option`
  is what the votes count for: a party (`cdu`, its list or its nominee), a candidate
  (`de.st.wk.002.einzelbewerber.1`) or an answer (`ja`).
- `options.csv`: columns `option,party`, every option the tally counts votes for, once,
  with the party it stands for, blank for an Einzelbewerber or an answer.
- `<name>.csv`, one per hierarchy: columns `unit,level,parent`, every unit of the
  hierarchy and every counted unit, each with the unit it lies in (blank at the broadest
  level). `electoral.csv` places each Wahlbezirk in its Wahlkreis and each Wahlkreis in the
  Land; where the rows are the Wahlkreise themselves (Mecklenburg-Vorpommern 1998) or
  Gemeinden (1990), its only level is the Land. `administrative.csv` places each
  Wahlbezirk in its Gemeinde, each Gemeinde in its Kreis, since Gemeinden like Halle span
  several Wahlkreise. Mecklenburg-Vorpommern's (from 2002) adds the Amt
  between Kreis and Gemeinde, because Briefwahl in amtsangehörige Gemeinden is counted
  per Amt: the source files it under one pseudo-Gemeinde per Amt (`13071751`,
  "Briefwahl Demmin-Land"), which the bundle keeps as a `gemeinde` unit under its Amt.
  The 1998 bundle is per Wahlkreis only and has no administrative hierarchy.
  Berlin's electoral hierarchy has the Bezirk between Land and Wahlkreis
  (`land`, `bezirk`, `wahlkreis`), since its Wahlkreise are numbered per Bezirk and
  Bezirkslisten are counted along it; its ids keep the Bezirk
  (`de.be.bez.01.wbz.101`), and its alternative hierarchies are `stadthaelfte` (Ost/West)
  and `bundestagswahlkreis`.
- `as_found/`: the source files exactly as downloaded (Excel, CSV, PDF), unchanged and
  under their original names (but see the note on line endings in
  wahlwerk-data-processing's CLAUDE.md: git stores the Sachsen-Anhalt 2021 CSVs with LF). The engine does not read this folder; it is kept so the
  normalised files can be regenerated and checked against `sha256`.

Bundles are generated by wahlwerk-data-processing, not edited by hand.

## Party registries

Each file under `parties/` has a `schema` (format version), a `name` (the scope
identifier), a `description` and a `parties` object keyed by party slug:

```json
{
  "schema": 1,
  "name": "de.bund",
  "description": "Parties that have contested elections in Germany, federal or Land, keyed by party slug.",
  "parties": {
    "cdu": {
      "name": "Christlich Demokratische Union Deutschlands",
      "short_name": "CDU"
    }
  }
}
```

Load one with the engine:

```python
from wahlwerk.party import PartyRegistry

registry = PartyRegistry.from_json("../wahlwerk-data/parties/de/de.bund.json")
registry["cdu"]
```

The engine rejects unknown fields, an unknown `schema` and a party slug listed twice.

## Using it with the engine

The engine finds this archive at `$WAHLWERK_DATA`, a sibling clone `../wahlwerk-data`, or
`~/.cache/wahlwerk/data`, first hit wins. Cloning it next to `wahlwerk` is enough.

## Licence

The repository's own work is GPL-3.0 (`LICENSE`). Election data keeps the licence of its
source, recorded with the required attribution in each bundle's `election.toml`. The
Sachsen-Anhalt bundles are dl-de/by-2-0 from the Statistisches Landesamt Sachsen-Anhalt.
The Berlin bundle is CC BY 3.0 DE from the Amt für Statistik Berlin-Brandenburg (stated
in the workbook's Impressum).
The Mecklenburg-Vorpommern sources state no licence (2021 and 2026 carry only a
copyright notice by the Landeswahlleitung); their terms are still to be clarified with
the Landeswahlleitung. The same applies to the originals in each bundle's `as_found/`.
