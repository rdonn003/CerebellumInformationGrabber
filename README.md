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
individually for each of the **eight fissure-defined lobule subsections (2Cb – 10Cb)**.

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

Open the ROI Manager (*Analyze → Tools → ROI Manager*) and draw the following eleven ROIs.
Names are matched **case-insensitively by substring** — the order you add them does not matter.

| ROI | Type in FIJI | Required word in name |
|---|---|---|
| Entire cerebellum outline | Closed polygon | `cerebellum` |
| Granular layer + White Matter | Closed polygon | `granular` |
| White Matter only | Closed polygon | `white` or exactly `wm` |
| Purkinje cell line | Open polyline | `purkinje` |
| Fissure 1 | Open polyline | `fissure` |
| Fissure 2 | Open polyline | `fissure` |
| Fissure 3 | Open polyline | `fissure` |
| Fissure 4 | Open polyline | `fissure` |
| Fissure 5 | Open polyline | `fissure` |
| Fissure 6 | Open polyline | `fissure` |
| Fissure 7 | Open polyline | `fissure` |

**Example ROI Manager list** (any order):
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

#### Tracing tips

**Cerebellum, Granular+WM, WhiteMatter** — trace as closed polygons using the
*Polygon* or *Freehand* selection tool. These must be correctly nested:
White Matter ⊂ Granular+WM ⊂ Cerebellum. The plugin checks this and reports any
violation.

**Purkinje line** — trace as an open polyline following the row of Purkinje cell bodies
from one end of the cerebellum to the other. Use the *Segmented Line* tool.

**Fissure lines** — each line should run from the **pial surface** (outer edge of the
molecular layer) down to the **Purkinje cell line**. It does not need to cross into the
granular layer. You can trace either end first; the plugin detects the direction
automatically. Use the *Segmented Line* tool.

### 3. Run the plugin

Go to **Plugins → Cerebellar Morphometry → Quantify Layers...**

An options dialog appears:

| Option | Effect |
|---|---|
| Show colour-coded layer overlay | Adds a vector overlay showing Grey, Granular, Molecular, and Purkinje |
| Show per-lobule transparent fills | Adds distinct semitransparent fills for each of the 8 lobules |
| Show ImageJ Results Table | Displays the measurement table interactively in ImageJ |
| Export CSV | Saves a UTF-8 comma-separated file |
| Export Excel (.xlsx) | Saves a formatted Excel workbook |

If either export option is checked, you will be prompted to choose a save folder. Files
are named after the image title.

---

## Output table

The plugin produces a table matching this layout exactly:

```
Measurement  | Cerebellum | Grey Matter | Granular Layer | Molecular Layer | Purkinje
─────────────┼────────────┼─────────────┼────────────────┼─────────────────┼──────────
Area         | total      | total       | total          | total           |
Length       |            |             |                |                 | total
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
| Per-lobule fills (optional) | 8 distinct hues |

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
│   ├── FissurePartitioner        Seven fissures → eight half-plane partitions
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
    └── OverlayRenderer           Colour-coded vector overlay
```

### How partitioning works

1. Each fissure's inner (Purkinje-proximal) endpoint is projected onto the Purkinje
   polyline to find its arc-length position. Sorting by arc-length gives the anatomical
   order (2Cb → 10Cb) without any image-orientation assumptions.

2. Each fissure is extended in a straight line — continuing its own pial→Purkinje
   direction — far enough to clear the entire cerebellum bounding box.

3. The extended line is thickened into a large half-plane polygon. Intersecting that
   polygon with the cerebellum shape (`ShapeRoi.and`) gives the part of the cerebellum
   on the lobule-10 side of the cut.

4. The eight lobule regions are consecutive differences of these half-planes. Each
   region's Purkinje length is the calibrated arc-length between its two bounding
   fissure attachment points — no separate polyline clipping is needed.

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
| *"Missing the Cerebellum ROI"* | Add a closed polygon with "cerebellum" in its name |
| *"Missing the WhiteMatter ROI"* | Add a closed polygon with "white" or "wm" in its name |
| *"Expected exactly 7 fissure ROIs, found N"* | Ensure all seven fissure polylines have "fissure" in their names; no extra ROIs match that word |
| *"The Purkinje ROI is a closed area ROI"* | Re-trace Purkinje as an open polyline using the Segmented Line tool |
| *"More than one ROI matches Cerebellum"* | Rename or remove the duplicate |
| *"WhiteMatter is not fully inside Granular+WM"* | Redraw the outlines so that White Matter is contained within Granular+WM |

### Measurement looks wrong

| Symptom | Likely cause |
|---|---|
| Per-partition areas don't add up to the whole | One fissure line doesn't reach the Purkinje polyline — extend it |
| A subsection has zero area | Two adjacent fissures are nearly coincident — check for accidental duplicates |
| Overlay partition boundaries look skewed | A fissure was traced at a very shallow angle; the straight-line extension may overshoot a neighbouring fold — retrace more steeply |
| Areas reported in px² instead of µm² | No pixel calibration set — run *Analyze → Set Scale* before measuring |
