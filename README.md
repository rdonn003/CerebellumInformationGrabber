# Cerebellar Layer Quantification Plugin

A FIJI/ImageJ plugin for quantitative morphometric analysis of Nissl-stained cerebellar
histology sections. All measurements are derived from user-drawn ROI geometry — no pixel
segmentation is performed.

---

## Table of Contents

- [What it measures](#what-it-measures)
- [Requirements](#requirements)
- [Setup — Java and Maven](#setup--java-and-maven)
- [Build](#build)
- [Install into FIJI](#install-into-fiji)
- [User workflow](#user-workflow)
  - [1. Open your image](#1-open-your-image)
  - [2. Draw the ROIs](#2-draw-the-rois)
  - [3. Run the plugin](#3-run-the-plugin)
- [Output table](#output-table)
- [Overlay colours](#overlay-colours)
- [Architecture](#architecture)
- [Troubleshooting](#troubleshooting)

---

## What it measures

| Measurement | Formula |
|---|---|
| Cerebellum area | Area of the Cerebellum ROI |
| Grey Matter area | Cerebellum − White Matter |
| Granular Layer area | (Granular+WM) − White Matter |
| Molecular Layer area | Grey Matter − Granular Layer |
| Purkinje length | Arc-length of the Purkinje polyline |

All five are computed for the **whole cerebellum**. The last three are also computed
individually for each fissure-defined subsection — 7 traced fissures give the standard
**eight rodent vermis lobules (2Cb – 10Cb)**; a different fissure count works too and
produces that many generically-labeled subsections instead (see *How many fissures?*
below).

Results are exported as an ImageJ Results Table, a CSV file, and a formatted `.xlsx`
workbook.

---

## Requirements

| Item | Minimum version | Notes |
|---|---|---|
| JDK | 17 | Must be a JDK, not a JRE — see below |
| Maven | 3.8 | |
| FIJI / ImageJ | Any recent release | |
| Windows / macOS / Linux | — | All platforms supported |

---

## Setup — Java and Maven

### Install the JDK

Download the **JDK 21** installer (LTS release) from one of these official sources:

| Source | URL |
|---|---|
| Microsoft OpenJDK *(recommended for Windows)* | https://learn.microsoft.com/en-us/java/openjdk/download |
| Adoptium | https://adoptium.net |
| Oracle JDK | https://www.oracle.com/java/technologies/downloads/ |

> **Important:** Download the **JDK**, not the JRE. The JRE does not include `javac`
> (the Java compiler) and the build will fail if you use it.

The **Windows `.msi` installer** from Microsoft or Adoptium sets `JAVA_HOME` and updates
your `PATH` automatically. After installing, open a **new** Command Prompt and verify:

```cmd
java -version
javac -version
```

Both commands should print a version number. If `javac` is not found, see
[Troubleshooting](#troubleshooting).

### Install Maven

1. Download the **binary zip** from https://maven.apache.org/download.cgi
2. Extract it anywhere, e.g. `C:\Program Files\Apache\maven`
3. Add `C:\Program Files\Apache\maven\bin` to your system `PATH`
   *(Control Panel → System → Advanced → Environment Variables → Path → Edit → New)*
4. Open a new Command Prompt and verify:

```cmd
mvn -version
```

---

## Build

Clone or unzip the project, then run:

```cmd
cd cerebellar-layer-plugin
mvn package
```

Maven downloads the required dependencies (ImageJ, SciJava, Apache POI) and produces a
single self-contained jar at:

```
target/cerebellar-layer-plugin-1.0.0.jar
```

All dependencies are bundled inside the jar so nothing else needs to be copied to FIJI.

> **Behind a firewall or without internet access?** Maven needs to reach Maven Central
> (`repo1.maven.org`) to download dependencies the first time. If that is blocked,
> install the jars manually into your local Maven repository with `mvn install:install-file`
> before building.

---

## Install into FIJI

1. Copy `target/cerebellar-layer-plugin-1.0.0.jar` into your FIJI `plugins/` folder.

   Default locations:

   | OS | Path |
   |---|---|
   | Windows | `C:\Program Files\Fiji.app\plugins\` |
   | macOS | `/Applications/Fiji.app/plugins/` |
   | Linux | `~/fiji/plugins/` |

2. Restart FIJI, or run **Help → Refresh Menus**.

3. The plugin appears under:

   **Plugins → Cerebellar Morphometry → Quantify Layers...**

---

## User workflow

### 1. Open your image

Open the Nissl-stained cerebellar section in FIJI.

Set the pixel calibration if you have a known scale bar
(*Image → Properties*, or use the *Analyze → Set Scale* tool).
The plugin uses whatever calibration is present — without calibration, areas are reported
in px² and lengths in px.

### 2. Draw the ROIs

Open the ROI Manager (*Analyze → Tools → ROI Manager*) and draw four fixed ROIs (the
whole-cerebellum outline, Granular+WM, White Matter, and the Purkinje line) plus one
fissure line per split you need. Matching is **case-insensitive**. Both full names and
common lab abbreviations are accepted — use whichever your lab already uses. The order
ROIs are added does not matter.

| ROI | Type in FIJI | Accepted names (any case, examples) |
|---|---|---|
| Entire cerebellum outline | Closed polygon | `Cerebellum`, `cerebellum`, **`CB`**, `cb`, `CB_left` |
| Granular layer + White Matter | Closed polygon | `Granular+WM`, `granular`, **`GL+WM`**, `GLWM`, `GL_WM`, `GL-WM`, `GL WM` |
| White Matter only | Closed polygon | `WhiteMatter`, `white matter`, **`WM`**, `wm` |
| Purkinje cell line | Open polyline | `Purkinje`, `purkinje`, **`PL`**, `pl` |
| Fissure 1, 2, 3, … | Open polyline | `Fissure1`, `fissure1`, **`FL1`**, `FL_1`, `FL-1`, `fl1`; `Fissure2`/`FL2`; and so on |

**How many fissures?** *N* fissures split the cerebellum into *N+1* subsections. **7
fissures → 8 subsections is the standard scheme** for a midline sagittal section of
rodent vermis, and gets the familiar anatomical labels (2Cb, 3Cb, 4/5Cb, 6Cb, 7Cb, 8Cb,
9Cb, 10Cb). That specific scheme doesn't apply to every section, though — off-midline
sagittal cuts, coronal or horizontal sections, damaged tissue, and other species can all
have a different number of visible lobules. Trace however many fissures the section
actually shows (at least one); anything other than 7 gets generic labels instead
("Section 1", "Section 2", …) since a specific anatomical name would be a guess. Name
extra fissures the same way (`FL8`, `FL9`, `fissure8`, …) — there's no upper limit.

> **Disambiguation note:** `GL+WM` is always matched as Granular+WM, never as White Matter,
> even though it contains the letters "WM". The plugin evaluates Granular+WM before
> White Matter specifically to handle this case.
>
> `CB` is matched as a whole word, so lobule labels like `2Cb` or `10Cb` in your ROI
> Manager are never mistaken for the whole-cerebellum outline.

**Example ROI Manager lists for the standard 7-fissure case — both of these work identically:**

Full names:
```
Cerebellum
Granular+WM
WhiteMatter
Purkinje
Fissure1
Fissure2
Fissure3
Fissure4
Fissure5
Fissure6
Fissure7
```

Abbreviated:
```
CB
GL+WM
WM
PL
FL1
FL2
FL3
FL4
FL5
FL6
FL7
```

#### Tracing tips

**Cerebellum, Granular+WM, WhiteMatter** — trace as closed polygons using the
*Polygon* or *Freehand* selection tool. Normally these are nested: White Matter ⊂
Granular+WM ⊂ Cerebellum. It's fine — and sometimes necessary, see *Closing the loop*
below — for White Matter or Granular+WM to reach or extend past its parent boundary
somewhere; the plugin logs a note rather than blocking you, since that's usually
deliberate. A very large mismatch is still worth double-checking, though, in case two
ROIs were accidentally swapped or traced on different sections.

#### Closing the loop (so the first and last subsections don't merge)

The Cerebellum ROI is one closed outline, but the fissures and the Purkinje line only
span the *foliated* part of it — they don't reach around to the peduncle, where the
cerebellum attaches to the brainstem. Left alone, that means the subsections at the two
ends of your trace (2Cb and 10Cb in the standard scheme) are still connected to each
other through that unfissured region, and the plugin can't tell where one ends and the
other begins: it'll report them as a single merged region instead of two.

The fix is to let White Matter trace out to meet the Granular+WM and Cerebellum
boundaries specifically at the peduncle, where there's no molecular or granular
layering anyway (it's just white matter connecting to the brainstem). Doing this pinches
the ring shut at that one point, which acts as an implicit extra cut — the first and
last subsections separate cleanly without needing another fissure. Concretely:

1. Trace Cerebellum, Purkinje, and the fissures as usual — Purkinje and the fissures
   naturally stop short of the peduncle since there's no cortex to trace there.
2. When tracing Granular+WM, extend it out to touch the Cerebellum boundary at the
   peduncle (rather than stopping short, as you would elsewhere).
3. When tracing White Matter, extend it out to touch (or cross) both the Granular+WM
   and Cerebellum boundaries at that same spot.

If you skip this, everything still runs — you'll just see the first or last subsection
(whichever ends up on the losing side of the merge) missing from one or both layers in
the results, along with a FIJI log message naming it. That's your cue to extend White
Matter a bit further at the peduncle and rerun.

**Purkinje line** — trace as an open polyline following the row of Purkinje cell bodies
from one end of the cerebellum to the other. Use the *Segmented Line* tool.

**Fissure lines** — each line should run from the **pial surface** all the way down to
**White Matter**, following the deepest point of the fold the whole way. You do *not*
need to stop precisely at the Purkinje cell line — the plugin automatically finds where
each fissure crosses it. This is deliberate: identifying the exact pial and White Matter
edges is much easier than picking out the thin Purkinje layer partway down a fold, so
trace the *whole* visible fold rather than trying to stop partway through it. You can
trace either end first; the plugin detects the direction automatically. Use the
*Segmented Line* tool.

> **Important:** trace all the way to both edges, and follow the *center* of the fold
> rather than cutting across it at an angle. A fissure line that stops short of the pial
> surface or White Matter, or cuts across the fold diagonally instead of running along
> its length, can fail to fully separate two neighbouring subsections — you'll see a
> warning in the FIJI log (`Window → Log`) naming the affected layer if this happens,
> and fewer subsection rows than expected (N+1 for N fissures) in the results table is
> the practical symptom to watch for.

### 3. Run the plugin

Go to **Plugins → Cerebellar Morphometry → Quantify Layers...**

An options dialog appears:

| Option | Effect |
|---|---|
| Show colour-coded layer overlay | Adds a vector overlay showing Grey, Granular, Molecular, and Purkinje |
| Show per-lobule transparent fills | Adds distinct semitransparent fills for each subsection |
| Add measurement ROIs to ROI Manager | Adds a named ROI for every number in the output table (see below) |
| Show ImageJ Results Table | Displays the measurement table interactively in ImageJ |
| Export CSV | Saves a UTF-8 comma-separated file |
| Export Excel (.xlsx) | Saves a formatted Excel workbook |

If either export option is checked, you will be prompted to choose a save folder. Files
are named after the image title.

### Measurement ROIs

With *Add measurement ROIs to ROI Manager* checked, the plugin appends one ROI per
measured region to the ROI Manager, alongside your original input ROIs:

| Name | What it is |
|---|---|
| `Grey Matter`, `Granular Layer`, `Molecular Layer` | The three whole-cerebellum layers |
| `<lobule>_Granular` | That lobule's clipped Granular Layer footprint, e.g. `2Cb_Granular` |
| `<lobule>_Molecular` | That lobule's clipped Molecular Layer footprint |
| `<lobule>_Purkinje` | That lobule's segment of the Purkinje line (an open polyline) |

Only lobules that were actually resolved get ROIs — if partitioning couldn't cleanly
separate every lobule (see the log-message troubleshooting entries below), the missing
ones simply won't appear rather than adding an empty or wrong ROI. Each added ROI is an
independent copy, so renaming, deleting, or re-measuring it in the ROI Manager has no
effect on the plugin's own results.

---

## Output table

The plugin produces a table matching this layout exactly:

```
Measurement  | Cerebellum | Grey Matter | Granular Layer | Molecular Layer | Purkinje
─────────────┼────────────┼─────────────┼────────────────┼─────────────────┼──────────
Area         | total      | total       | total          | total           | total area
Length       |            |             |                |                 | total length
2Cb          |            |             | area           | area            | length
3Cb          |            |             | area           | area            | length
4/5Cb        |            |             | area           | area            | length
6Cb          |            |             | area           | area            | length
7Cb          |            |             | area           | area            | length
8Cb          |            |             | area           | area            | length
9Cb          |            |             | area           | area            | length
10Cb         |            |             | area           | area            | length
```

Column headers include the calibrated unit, e.g. `Granular Layer (µm²)` and `Purkinje (µm)`.
Empty cells carry no measurement for that row/column combination.
All numeric values are rounded to 4 decimal places.

**About the Purkinje column's Area row:** a line doesn't really have an area, so this
isn't a biologically meaningful quantity — it's included because it's exactly what
you'd get selecting the Purkinje ROI in the ROI Manager and clicking *Measure* with Area
checked in *Set Measurements*. ImageJ reports a line selection's "area" as the number of
pixels its path visits, times the calibrated area per pixel, so the number here matches
that built-in behavior rather than anything computed independently. Only the whole-line
total is reported (not a per-lobule breakdown).

The Excel export adds bold formatting for the summary rows, auto-sized columns, and a
frozen header row and label column for easy scrolling.

---

## Overlay colours

| Layer | Colour |
|---|---|
| Grey Matter | Blue |
| Granular Layer | Green |
| Molecular Layer | Yellow |
| Purkinje line | Red |
| Partition boundaries (extended fissures) | White |
| Per-subsection fills (optional) | 12 distinct hues, cycling if there are more subsections than colours |

The overlay is non-destructive vector graphics. To remove it:
*Image → Overlay → Remove Overlay*.

---

## Architecture

```
org.cerebellum.morphometry
│
├── CerebellarMorphometryPlugin   FIJI entry point, options dialog, pipeline controller
│
├── model/
│   ├── LayerSet                  Validated, typed input ROI bundle
│   ├── ConstructedLayers         Three Boolean-derived whole-cerebellum layer shapes
│   ├── PartitionSet              Eight lobule regions with Purkinje arc-length bounds
│   ├── MorphometryResults        Final numeric output (areas, lengths, per-subsection)
│   └── ValidationException       Aggregates all ROI problems into one error dialog
│
├── geometry/
│   ├── ROIValidator              Name / type / nesting checks  →  LayerSet
│   ├── BooleanROIProcessor       Defensive-copy ShapeRoi AND / OR / NOT + area measurement
│   ├── GeometryUtils             Vector math, arc-length, polyline projection, interpolation
│   ├── LayerConstructor          The three Boolean subtractions (Grey, Granular, Molecular)
│   ├── FissurePartitioner        N fissures → N+1 subtract-and-split partitions
│   ├── RasterSplitUtils          Pixel-mask split (connected components) used by the above
│   ├── PartitionClipper          Clips granular/molecular shapes into per-partition footprints
│   └── PurkinjeLengthCalculator  Calibrated arc-length, total and per-subsection
│
├── measurement/
│   └── MeasurementEngine         Orchestrates the full pipeline  →  MorphometryResults
│
├── export/
│   └── SpreadsheetExporter       ImageJ ResultsTable + CSV + XLSX
│
└── visualization/
    ├── OverlayRenderer           Colour-coded vector overlay
    └── RoiManagerExporter        One named ROI per measurement, added to the ROI Manager
```

### How partitioning works

The lobule regions are built with a **subtract-and-split** strategy — the same
idea as selecting a shape in the ROI Manager, subtracting a set of thin cutting lines
from it, and using *Split* to break the result into its separate pieces — rather than by
computing which side of each cut line is "positive." Trying to track sides explicitly
was the source of most of the early bugs in this plugin: it only takes one place where
two cuts cross close together for a sign test to pick the wrong side, silently swapping
or merging entire lobules.

The split itself works on a **pixel mask**, not on FIJI's vector `ShapeRoi`/`Area`
machinery directly. That wasn't the original design — it's a direct response to a bug
found while testing against a real traced section: `java.awt.geom.Area`, which
`ShapeRoi`'s Boolean operations are built on, turned out to fragment even a single plain
subtraction between two correctly-nested, non-self-crossing traced polygons into dozens
of spurious extra pieces (verified case: `Cerebellum \ Granular+WM` on real data came
back as 77 disjoint pieces instead of 1, with the count scaling almost linearly with how
many points the polygons had — and zero actual crossings between the two boundaries).
Rasterizing to a pixel mask and using ordinary 8-connected flood-fill labeling sidesteps
that fragility completely; the same subtraction on the same data reliably comes back as
the single connected ring the anatomy actually has.

1. Each fissure is traced across the **full tissue depth** — pial surface to White
   Matter — rather than stopping at the Purkinje line (see the tracing tip above for
   why). The plugin samples finely along each fissure's own path to find where it comes
   closest to the Purkinje polyline; that crossing point's arc-length position sorts the
   fissures into anatomical order (2Cb → 10Cb for the standard 7-fissure scheme), and
   its position along the fissure's own path splits it into a molecular portion (pial
   end → crossing) and a granular portion (crossing → White Matter end).

2. The **Molecular layer** (`Cerebellum` minus `Granular+WM`) is split using each
   fissure's molecular portion, extended a small distance past its two ends to absorb
   ordinary hand-tracing mismatch — no direction is extrapolated beyond what the user's
   own trace already establishes.

3. The **Granular layer** (`Granular+WM` minus `WhiteMatter`) is split the same way,
   using each fissure's granular portion (crossing → White Matter end).

4. For each layer, the fissure portions are thickened into thin strips and subtracted
   from that layer's rasterized mask; connected-component labeling then finds the N+1
   disjoint pieces (N = fissure count), which are sorted back into anatomical order
   (2Cb → 10Cb for the standard 7-fissure scheme) by projecting each piece's centroid
   onto the Purkinje arc. A subsection's final region is the union of its Molecular
   piece and its Granular piece.

5. Each region's Purkinje length is the calibrated arc-length between its two bounding
   fissure attachment points — no separate polyline clipping is needed.

If a section is traced unusually finely, the strip width automatically retries a couple
of wider fallback values. A message is written to the FIJI log (`Window → Log`) naming
the affected layer if a clean 8-way split still can't be found — this now reliably means
a specific fissure doesn't fully cross that layer (too short, stops before reaching the
boundary, or cuts across the fold at an angle instead of along it) rather than a
numerical artifact, so it's worth checking that layer's fissures specifically.

---

## Troubleshooting

### Build errors

| Error | Cause | Fix |
|---|---|---|
| `JAVA_HOME is not defined correctly` | `JAVA_HOME` environment variable missing or wrong | Set `JAVA_HOME` to your JDK folder (not the `bin` subfolder). See [Setup](#setup--java-and-maven). |
| `javac: command not found` | JRE installed instead of JDK, or JDK `bin` not on `PATH` | Install a JDK. The JRE does not include `javac`. |
| `mvn: command not found` | Maven not on `PATH` | Add the Maven `bin` folder to your system `PATH`. |
| Download errors during `mvn package` | No internet / Maven Central blocked | Pre-install the three jars with `mvn install:install-file`. |

### ROI validation errors

| Error message | Fix |
|---|---|
| *"The ROI Manager is empty"* | Add all required ROIs before running the plugin |
| *"Missing the Cerebellum ROI"* | Name the closed polygon `Cerebellum`, `CB`, or anything containing "cerebellum" |
| *"Missing the Granular+WM ROI"* | Name the closed polygon `GL+WM`, `GLWM`, `Granular+WM`, or anything containing "granular" |
| *"Missing the White Matter ROI"* | Name the closed polygon `WM` or anything containing "white". Note: `GL+WM` is correctly identified as Granular+WM, not White Matter — add a separate `WM` ROI |
| *"No fissure ROIs found"* | Add at least one fissure polyline named `FL1`, `fissure1`, or similar |
| Subsections are labeled "Section 1", "Section 2", … instead of anatomical names | Expected whenever the fissure count isn't 7 — the anatomical names (2Cb, 3Cb, …) only apply to the standard scheme. Not an error |
| *"Missing the Purkinje ROI"* | Name the open polyline `PL`, `Purkinje`, or anything containing "purkinje" |
| *"The Purkinje ROI is a closed area"* | Re-trace as an open polyline using the *Segmented Line* tool |
| *"More than one ROI matches Cerebellum"* | Two ROI names both match the Cerebellum rules — rename or remove one |
| *"WhiteMatter is not fully inside Granular+WM"* | Redraw the outlines so that WM is contained within GL+WM |

### Measurement looks wrong

| Symptom | Likely cause |
|---|---|
| Per-partition areas don't add up to the whole | One fissure line doesn't reach all the way to both the pial surface and White Matter — retrace it so both ends land on (or very near) those boundaries |
| A subsection has zero or near-zero area | Two adjacent fissures are nearly coincident — check for accidental duplicates |
| FIJI log shows *"could not find a strip width that cleanly splits..."* | A specific fissure in the named layer doesn't fully cross it — check for one that's too short, cuts across the fold at an angle instead of along its length, or veers into a neighbouring fold |
| FIJI log shows *"did not cleanly separate every lobule"*, naming specific labels | If the named label is the **first or last subsection** (2Cb/10Cb in the standard scheme, or "Section 1"/the highest-numbered section otherwise), see *Closing the loop* above — extend White Matter out to the Granular+WM and Cerebellum boundaries at the peduncle. For any other label, it's usually the fissure bounding that subsection not fully crossing the named layer |
| Areas reported in px² instead of µm² | No pixel calibration set — run *Analyze → Set Scale* before measuring |
