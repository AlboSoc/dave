# Dave Collection System — Design Notes

## 1. Purpose

This document captures the current design direction for a durable, phone-friendly collection system for Dave's buses and related transport interests.

Dave currently records buses, trains, locomotives, trams, coaches and Wetherspoons visits using a mixture of handwritten notes, OpenDocument files, colour highlighting, stars, numbers and free-form annotations. The repository already preserves the original `.odt` and `.ods` files and publishes HTML and PDF previews for easier viewing.

The proposed system should improve day-to-day use without making Dave dependent on custom software that may eventually stop working.

> **Governing principle: the application may become obsolete, but Dave's collection must not.**

---

## 2. User context

Dave:

- is highly motivated by buses, trains, locomotives, Wetherspoons and collection completion;
- can show exceptional persistence with repetitive or difficult activities;
- already understands his own existing registers and their conventions;
- has diminished mental capacity, so unfamiliar tools, ambiguous controls and accidental mistakes may cause disproportionate frustration;
- is not technically sophisticated and should never be expected to maintain software, schemas, hosting or recovery procedures;
- may benefit from a gradual acclimatisation process rather than a sudden replacement of his current system.

His existing conventions may include:

- vehicle or fleet numbers;
- operator, location or vehicle-type headings;
- colours to indicate status;
- stars to indicate photographs;
- free-form words or notes;
- uncertain, partial or irregular identifiers;
- grouping by town, operator, railway, category or other personally meaningful structure.

These conventions must be studied before they are normalised.

---

## 3. Existing repository

The GitHub repository currently acts as a document archive and publication mechanism.

It contains:

- original OpenDocument files in `files/`;
- generated browser-friendly HTML previews;
- generated PDF previews;
- an index page for navigating the collection from a phone.

This existing publication layer should be preserved. It is useful independently of any future application and already gives Dave a low-friction way to browse familiar material.

The future system should therefore evolve the current repository rather than discard it.

---

## 4. Proposed architecture

The preferred long-term architecture is a small constellation of services rather than one monolithic application.

```text
Dave's phone or computer
        |
        +-- Google Sheet directly
        |
        +-- simple custom web application
        |
        +-- optional forms or other capture interfaces
                    |
                    v
         canonical Google spreadsheet
                    |
        +-----------+------------+
        |                        |
        v                        v
Google Drive photos       dated CSV/XLSX exports
        |
        v
generated HTML/PDF catalogue
        |
        v
GitHub Pages publication
```

### Responsibilities

**Google Sheets**

- human-readable canonical data store;
- direct editing and bulk correction;
- filtering, sorting and review;
- emergency fallback if the custom application stops working.

**Google Drive**

- photographs;
- dated exports;
- archival snapshots;
- supporting source material.

**GitHub**

- source code;
- design and recovery documentation;
- issue tracking;
- deployment history;
- generated HTML/PDF publication;
- possibly static application hosting.

**Custom web application**

- simple phone-oriented interaction;
- searching for a bus;
- recording seen, ridden or photographed status;
- reviewing migrated entries;
- displaying progress and recent activity.

**Cloud coding agents**

- substantial implementation work;
- tests and refactoring;
- documentation;
- generation of pull requests or repository changes.

**Phone**

- main development-control, review, deployment and testing interface for Albo;
- everyday use interface for Dave.

---

## 5. Data ownership and durability

The production Google spreadsheet should belong to Dave's Google account.

Dave should create the spreadsheet and share edit access with Albo. This ensures that the canonical collection is not trapped inside Albo's account or a developer-owned backend.

The system should support:

- CSV export;
- XLSX export;
- dated Drive backups;
- recovery without the application;
- clear documentation explaining where the canonical data is stored;
- periodic copies outside the live spreadsheet where practical.

The repository may remain public for source code and public documents, but private collection data should not be published accidentally.

---

## 6. The Sheet is also an interface

The Google Sheet should not be treated merely as a hidden backend. It is one legitimate interface to the same data.

Direct Sheet editing is likely to be most useful for:

- reviewing the whole collection;
- correcting mistakes;
- entering batches;
- filtering and sorting;
- importing old material;
- exporting and recovery.

The custom phone application is likely to be most useful for:

- quickly entering a fleet number;
- checking whether Dave already has it;
- recording seen, ridden or photographed status;
- adding date, route and location;
- reviewing one migration item at a time.

The application should not attempt to reproduce a desktop spreadsheet on a small screen.

---

## 7. Explicit data, not formatting-only meaning

Colour, stars and other visual conventions may still be displayed, but must not be the only storage of meaning.

For example:

```text
ridden: true
photographed: true
```

may be rendered as a yellow row and a star, but the underlying facts must survive CSV export.

Every meaningful status should have an explicit field.

---

## 8. Initial bus data model

The final schema should be derived from representative existing files, especially the bus `.ods` registers. The following is a starting hypothesis only.

### Vehicles

- `vehicle_id`
- `fleet_number`
- `registration`
- `operator`
- `vehicle_type`
- `status`
- `notes`

### Encounters

- `encounter_id`
- `vehicle_id`
- `date_time`
- `location`
- `latitude`
- `longitude`
- `route`
- `seen`
- `ridden`
- `photographed`
- `photo_link`
- `notes`

A vehicle and an encounter should normally be distinct concepts. One bus may be seen, ridden or photographed multiple times.

For an early prototype, a simpler single-table structure may be acceptable if it is clearly documented and exportable.

---

## 9. Migration as reviewable transcription

Migration from the existing ODT/ODS files should not be a one-shot automatic conversion.

The preferred workflow is:

```text
original source file
        |
        v
automatic extraction
        |
        v
staging and proposed interpretation
        |
        v
Dave accepts, edits, rejects or defers
        |
        v
canonical structured record
```

Unreviewed imports must not silently become canonical data.

### Suggested staging areas

- `Import_Raw`
- `Import_Review`
- `Vehicles`
- `Encounters`

### Suggested provenance fields

- `import_id`
- `source_file`
- `source_section`
- `source_location`
- `source_text`
- `proposed_identifier`
- `proposed_description`
- `proposed_status`
- `parser_confidence`
- `review_status`
- `review_notes`
- `reviewed_at`

The exact original text must always be retained.

### Suggested review statuses

- `pending`
- `accepted`
- `accepted_with_edits`
- `rejected`
- `duplicate`
- `leave_for_later`

Rejected items should remain in the audit history so they are not repeatedly re-imported.

---

## 10. Migration as acclimatisation

The migration process should also serve as Dave's introduction to the new system.

Each review screen should begin with familiar information from his old register, then ask for a small decision using the same controls that will later be used during normal collection activity.

Example:

```text
From your old Sheffield register

Original:
35124 *

We think this means:

Fleet number: 35124
Photographed: Yes

[ YES, THAT'S RIGHT ]
[ CHANGE IT ]
[ LEAVE FOR LATER ]
```

This provides:

- familiarity;
- repeated practice;
- gradual learning;
- useful real work;
- immediate visible results;
- observation of where Dave hesitates or becomes confused.

The review interface should be built before the full application if practical, because it can reveal the interaction model Dave actually understands.

---

## 11. Interaction and accessibility principles

### Keep decisions narrow

Normally show one item and one principal decision at a time.

### Reveal complexity only when needed

The default action should be acceptance. Editing controls should appear only when Dave chooses to change something.

### Use words, not unexplained icons

Buttons should use explicit labels. Familiar stars may be retained where they already have meaning, but generic interface icons should not be relied upon.

### Make mistakes reversible

Every action should be undoable. Rejection should not permanently delete source material.

### Make stopping safe

Dave should be able to stop at any time without losing progress or being penalised.

### Make resumption obvious

The application should return directly to the next unfinished item.

### Separate Dave mode and maintainer mode

Dave should not see parser confidence, import identifiers, duplicate algorithms, schema controls or deployment details.

---

## 12. Reward and progress cycle

Dave's persistence with trainspotting and Candy Crush suggests that he may respond well to:

- small concrete actions;
- immediate confirmation;
- visible accumulation;
- meaningful categories;
- clear completion states;
- occasional difficult entries;
- an obvious next item;
- the ability to continue over many sessions.

The system should borrow the constructive structure of that reward cycle without using manipulative game mechanics.

Useful feedback might include:

```text
Bus 35124 added

Sheffield buses reviewed: 127 of 420
Photographs confirmed: 84

[ NEXT BUS ]
```

Meaningful completion summaries might include:

```text
Section complete

First South Yorkshire — 35000 series

82 buses checked
61 photographed
44 ridden
7 left for later
```

Avoid:

- countdown timers;
- streak pressure;
- limited lives;
- penalties for stopping;
- fabricated scarcity;
- deliberately obstructive difficulty.

The challenge should come from the collection itself, not from operating the software.

---

## 13. Migration confidence and batching

The importer should distinguish high-confidence and ambiguous material.

### High confidence

- one clear identifier;
- optional recognised photo marker;
- known section and category.

### Medium confidence

- identifier plus short description;
- unusual prefixes or punctuation;
- multiple status markers.

### Low confidence

- multiple possible identifiers;
- incomplete numbers;
- words that may be headings, names or descriptions;
- unclear grouping;
- uncertain source boundaries.

High-confidence records may be offered for carefully presented batch approval. Ambiguous records should be reviewed individually.

No batch rule should become canonical without human approval.

---

## 14. Development approach

A phone-only workflow is viable when the phone is treated as the control and review surface rather than the machine doing all compilation and editing.

Suggested workflow:

```text
phone
  |
  +-- discuss requirements
  +-- issue cloud-agent tasks
  +-- review diffs and pull requests
  +-- test live application
  +-- report defects with screenshots
          |
          v
cloud coding agent
          |
          v
GitHub repository
          |
          +-- application
          +-- tests
          +-- documentation
          +-- deployment instructions
```

Google permissions and first-time Apps Script deployment may still require some manual work in the phone browser.

---

## 15. Recommended implementation stages

### Stage 0 — Study and preserve

- inspect representative existing bus and railway files;
- document the meaning of colours, stars, groupings and annotations;
- retain originals unchanged;
- preserve existing HTML/PDF generation.

### Stage 1 — Personal Google Sheets experiment

- create a disposable spreadsheet in Albo's account;
- test phone editing;
- use explicit checkbox and text fields;
- practise filtering, sharing and export;
- verify that CSV retains all important meaning.

### Stage 2 — Migration-review prototype

- import one small representative section;
- present one item at a time;
- support accept, edit, reject and defer;
- retain exact provenance;
- observe Dave using it.

### Stage 3 — Dave-owned production spreadsheet

- Dave creates the canonical spreadsheet;
- shares editor access with Albo;
- approved records are copied or migrated into it;
- establish backups and recovery notes.

### Stage 4 — Everyday bus application

- search by fleet number;
- add a new vehicle;
- mark seen, ridden or photographed;
- show recent records and progress;
- work well on Android.

### Stage 5 — Encounter history and photographs

- support repeated sightings and rides;
- link photographs in Drive;
- record routes, dates and locations.

### Stage 6 — Optional enrichment

- live vehicle positions;
- nearby uncollected buses;
- operator completion reports;
- richer maps and analytics.

Optional integrations must never be required to access the canonical collection.

---

## 16. Immediate next questions

Before finalising the schema:

1. What do the existing colours and stars mean in each representative file?
2. Does Dave distinguish seen, ridden and photographed consistently?
3. Is a fleet number sufficient to identify a bus over time?
4. How are operator changes represented?
5. Are entries grouped primarily by operator, town, depot, type or trip?
6. What information does Dave actually enter while outside?
7. What does he usually add later at home?
8. Which parts of the existing documents are authoritative, uncertain or merely layout?
9. Does Dave find a one-item review screen satisfying and understandable?
10. Which progress measures are meaningful to him?

---

## 17. Success criteria

The project is successful when:

- Dave can record and find buses with minimal frustration;
- Dave understands the everyday controls;
- migration gives him useful practice with the new system;
- original source data and provenance remain available;
- accidental actions are reversible;
- the canonical collection belongs to Dave;
- all important meaning survives CSV/XLSX export;
- the application can fail without destroying access to the collection;
- another competent person can recover the data using written instructions;
- HTML/PDF publication can continue independently of the editing application.

