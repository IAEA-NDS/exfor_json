# EXFOR in JSON

This repository contains the EXFOR library converted to one JSON document per
entry. The source records come from
[IAEA-NDS/exfor_master](https://github.com/IAEA-NDS/exfor_master), and the
conversion is performed by
[IAEA-NDS/exforparser](https://github.com/IAEA-NDS/exforparser).

The files are the JSON data source used by the
[IAEA Nuclear Reaction Data Explorer](https://nds.iaea.org/dataexplorer/) and
its [REST API](https://nds.iaea.org/dataexplorer/api/). This JSON representation
is also the basis of the observable-based datasets published separately in
[IAEA-NDS/exfortables_py](https://github.com/IAEA-NDS/exfortables_py).
That repository extracts usable `(x, dx, y, dy)` tables from EXFOR and
organizes them by projectile, nuclide, reaction, and observable so that common
nuclear-reaction datasets can be found and used without interpreting the full
entry-level JSON structure.

> [!IMPORTANT]
> The JSON model is under active development and may change. Consumers should
> use the field names and nesting documented below rather than the older schema
> sketches that circulated during development.

## Data provenance and updates

The data flow is:

```text
EXFOR master entry (.x4)
        |
        v
exforparser
        |
        v
exfor_json
        |-- IAEA Data Explorer entry view
        |-- IAEA Data Explorer REST API
        `-- observable tabulation --> exfortables_py
```

Each data update is committed with the corresponding EXFOR master release in
the commit message. To see which snapshot your checkout contains, run:

```sh
git log -1 --format=%s
```

The `last_updated` and `number_of_revisions` fields in each document describe
that individual EXFOR entry, not the release date or revision of this
repository.

## Download

Clone the complete dataset with Git:

```sh
git clone https://github.com/IAEA-NDS/exfor_json.git
```

The repository is large because it contains the full converted library and its
history. For only the current snapshot, use a shallow clone:

```sh
git clone --depth 1 https://github.com/IAEA-NDS/exfor_json.git
```

An entry number is five alphanumeric characters. Its first three characters
select the directory, for example:

```text
json/128/12898.json
json/A01/A0144.json
```

## JSON model

The parser's
[JSON Schema](https://github.com/IAEA-NDS/exforparser/blob/dev/src/exforparser/schemas/schema_v1.json)
is the authoritative machine-readable description of the current model. At a
high level, each entry has this structure:

```text
entry
|-- entry                         five-character EXFOR entry number
|-- last_updated                  date of the latest entry revision
|-- number_of_revisions           revision count
|-- histories[]                   parsed HISTORY records
|-- bib_record                    entry-level bibliography from subentry 001
|   |-- title
|   |-- authors[]
|   |-- institutes[]
|   |-- references[]
|   `-- facilities[]
|-- reactions
|   `-- <subentry>                three-character subentry number
|       `-- <pointer>             EXFOR pointer, such as 0, 1, or A
|           |-- x4_code
|           |-- children[]        parsed reaction components
|           |-- math_expression   expression for a composite reaction
|           |-- operator
|           |-- free_text
|           `-- pointer
|-- experimental_conditions
|   `-- <subentry>
|       `-- <pointer>
|           `-- <keyword>[]       e.g. detector, facility, method, sample
|               |-- x4_code
|               `-- free_txt[]
`-- data_tables
    `-- <subentry>
        |-- common                COMMON block or null
        `-- data                  DATA block or null
            |-- heads[]
            |-- units[]
            `-- data[][]          column-major numeric values
```

Important details:

- Subentry and pointer keys are strings. Pointer `"0"` represents information
  without an explicit EXFOR pointer.
- A reaction `children` array has one item for a simple reaction and may have
  several items for a composite reaction. Parsed reaction components include
  `target`, `process`, `sf4`, `sf49`, and `sf5` through `sf9`.
- `common` can occur in subentry `001` and/or an individual subentry. The Data
  Explorer combines entry-level COMMON, subentry-level COMMON, and DATA when
  presenting a table.
- `common` or `data` is `null` when the corresponding EXFOR block is absent.
- Tables are **column-major**: `heads[i]`, `units[i]`, and `data[i]` describe
  the same column. Missing numeric cells are represented by `null`.
- Pointered table headings retain the pointer in the heading, for example
  `"DATA      1"`. The Data Explorer uses this suffix to select the columns for
  a reaction pointer.
- `free_txt` preserves supplementary EXFOR text as an array of lines.

### Reading a data table in Python

This example loads the DATA block from entry `12898`, subentry `002`, and
creates a pandas `DataFrame`. It keeps the heading and unit together so that
duplicate or pointered EXFOR headings remain distinguishable.

```python
import json
from pathlib import Path

import pandas as pd

path = Path("json/128/12898.json")
with path.open(encoding="utf-8") as stream:
    entry = json.load(stream)

table = entry["data_tables"]["002"]["data"]
if table is None:
    raise ValueError("Subentry 002 has no DATA block")

columns = {
    f"{head.strip()} [{unit}]": values
    for head, unit, values in zip(
        table["heads"], table["units"], table["data"], strict=True
    )
}
frame = pd.DataFrame(columns)

# DATA pointer 2 is in millibarns; ERR-T pointer 2 is a relative percentage.
frame["d(DATA 2) [MB]"] = (
    frame["DATA      2 [MB]"] * frame["ERR-T     2 [PER-CENT]"] / 100
)

ax = frame.plot.scatter(
    x="EN [MEV]",
    y="DATA      2 [MB]",
    yerr="d(DATA 2) [MB]",
)
ax.figure.tight_layout()
```

This example reads only the subentry's DATA block. If an analysis also needs
COMMON values, include `data_tables["001"]["common"]` and the selected
subentry's `common` block according to EXFOR COMMON/DATA semantics.

## Online access

### Data Explorer

The [Data Explorer](https://nds.iaea.org/dataexplorer/) provides reaction and
bibliographic searches, interactive plots, and entry pages. An entry page uses
the JSON document to display:

- entry metadata and compilation history;
- bibliography and parsed reactions;
- experimental conditions inherited from subentry `001` and the selected
  subentry;
- COMMON and DATA values for the selected reaction pointer.

For example, entry `12898`, subentry `002`, pointer `1` is available at:

<https://nds.iaea.org/dataexplorer/exfor/entry/12898-002-1>

### REST API

The production EXFOR API base URL is:

```text
https://nds.iaea.org/dataexplorer/api/exfor/
```

The entry endpoints directly expose documents from this repository:

| Request | Response |
| --- | --- |
| `GET /entry/<entry>` | Complete entry document |
| `GET /entry/<entry>/reactions` | `reactions` |
| `GET /entry/<entry>/bib` | `bib_record` |
| `GET /entry/<entry>/histories` | `histories` |
| `GET /entry/<entry>/data` | `data_tables` |
| `GET /entry/<entry>/experiment` | `experimental_conditions` |
| `GET /subentry/<entry>/<subentry>` | Subentry identifier, experimental conditions, and data tables |

Examples:

```sh
curl "https://nds.iaea.org/dataexplorer/api/exfor/entry/12898"
curl "https://nds.iaea.org/dataexplorer/api/exfor/entry/12898/reactions"
curl "https://nds.iaea.org/dataexplorer/api/exfor/subentry/12898/002"
```

The API also provides indexed searches. These responses are derived search
records rather than raw entry documents:

| Request | Purpose |
| --- | --- |
| `GET /bib` | Paginated entry-level bibliography search |
| `GET /index` | Paginated reaction-index search |
| `GET /entries` | Legacy reaction-level search |

At least one meaningful filter is required. Common filters include
`target_elem`, `target_mass`, `inc_pt`, `reaction`, `obs_type`, `obs_group`,
`facility`, `facility_type`, `first_author`, `authors`, `sf4`, `sf5`, `sf7`,
`sf8`, `start_en`, `end_en`, `year_start`, and `year_end`. Incident energies in
search requests and responses are in MeV.

```sh
curl "https://nds.iaea.org/dataexplorer/api/exfor/bib?target_elem=Fe&target_mass=56&year_start=2020"
curl "https://nds.iaea.org/dataexplorer/api/exfor/index?target_elem=Fe&target_mass=56&obs_type=SIG"
```

See the [online EXFOR API manual](https://nds.iaea.org/dataexplorer/docs/exfor-api)
and [OpenAPI documentation](https://nds.iaea.org/dataexplorer/docs/swagger) for
the complete public interface.

## Producing JSON with exforparser

After configuring the parser's `MASTER_GIT_REPO_PATH` and output paths, convert
one entry with the command-line interface:

```sh
exforparser --convert 12898
```

The equivalent Python call returns the entry dictionary before it is written:

```python
from exforparser.json_converter import convert_exfor_to_json

entry = convert_exfor_to_json("12898")
```

Use `exforparser --convert all` for the complete master library or
`exforparser --convert updated` for entries changed since the parser's stored
entry index. Parser setup and conversion development belong in the
[exforparser repository](https://github.com/IAEA-NDS/exforparser); this
repository is the generated dataset.

## Citation and license

When using EXFOR data in a publication, identify the EXFOR entry or subentry,
its revision date, and the related publication recorded in `bib_record`.
For the related
[exfortables_py](https://github.com/IAEA-NDS/exfortables_py) datasets, cite:

- S. Okumura, G. Schnabel, and A. Koning, “Development of an
  EXFORTABLES-Inspired Structured Database through EXFOR Parser,” Proceedings
  of the Joint Symposium on Nuclear Data and PHITS, November 15–17, 2023,
  Tokai, Japan. [JOPSS](https://jopss.jaea.go.jp) (submitted).

For this JSON database, EXFOR Parser, and the IAEA Data Explorer, cite:

- S. Okumura, G. Schnabel, and A. Koning, “Developing a New Web Service for
  Experimental Nuclear Reaction Database (EXFOR) Using RESTful API and JSON,”
  16th International Conference on Nuclear Reaction Mechanisms, June 11–16,
  2023, Varenna, Italy.
  [EPJ Web of Conferences](https://www.epj-conferences.org/) (accepted).

The dataset is distributed under the
[Creative Commons Attribution 4.0 International license](LICENSE).
