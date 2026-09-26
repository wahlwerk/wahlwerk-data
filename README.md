# wahlwerk-data

The data archive for [wahlwerk](../wahlwerk), the rules-as-code engine for German
electoral law. It is meant to hold normalised election bundles (every Bundestagswahl since
1949, the sixteen Länder, kommunal) and lives apart from the engine so the engine's clone
stays small.

## Contents

| Path | What it holds |
| --- | --- |
| `parties/de/de.bund.json` | Parties that have contested Bundestag elections since 1949 |

Election bundles are not added yet.

### Party registries

Each file under `parties/` has a `schema` (format version), a `name` (the scope
identifier), a `description` and a `parties` object keyed by party slug:

```json
{
  "schema": 1,
  "name": "de.bund",
  "description": "Parties that have contested Bundestag elections since 1949, keyed by party slug.",
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
