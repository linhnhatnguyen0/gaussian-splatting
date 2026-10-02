# Editing: Theoretical Comparison

Theoretical comparison of Gaussian splat editing options: what editing involves, how each tool or approach works, and its strengths and weaknesses.

Editing takes a trained splat and prepares it for viewing. Training only optimizes what is visible from the training cameras, so the raw result usually contains artifacts and content that must be removed, and it sits in an arbitrary coordinate system.

---

## 1. Key concepts

### Editing operations

| Operation | Purpose |
|---|---|
| Selection and deletion | Remove floaters, stray Gaussians and unwanted objects |
| Cropping | Keep only the area of interest, discarding reconstructed background (through doors, windows) |
| Transform | Level the floor, set the origin and orientation, set real-world scale |
| Merging | Combine several splats into one scene |
| Color adjustment | Correct brightness, contrast or white balance |
| Reduction and compression | Remove low-value Gaussians, reduce view-dependent color detail, convert to compact formats |

### Limits of editing

Editing can only remove or modify existing Gaussians. Deleting an object leaves a hole where nothing behind it was ever observed, and blurry regions caused by poor coverage cannot be sharpened. Editing fixes what training added, not what capture missed.

### Manual vs. automated

- **Manual editing:** an operator selects and deletes Gaussians in a 3D interface. Precise but time-consuming, and results depend on the operator.
- **Rule-based automation:** Gaussians are filtered by properties such as opacity, size or distance from the scene center. Fast and reproducible, but crude: rules can remove valid detail or miss artifacts.
- **Semantic automation (research):** Gaussians are grouped by object using segmentation models, so an object can be selected or removed by label. Powerful, but immature and not integrated into mainstream tools.

### File formats

| Format | Characteristics |
|---|---|
| `.ply` | Standard, lossless, supported everywhere, large |
| `.splat` | Simple compact format, drops view-dependent color detail |
| `.spz` | Compressed format with good size reduction and wide adoption |
| `.sog` / compressed `.ply` | Compressed formats from the PlayCanvas ecosystem, aimed at web delivery |

Format support differs between editors and viewers, so the chosen format must be supported by both ends.

---

## 2. Options

### SuperSplat (PlayCanvas)

**How it works:** a browser-based splat editor, also installable as a desktop web app. Splats are loaded locally, edited with selection tools, and exported.

| Strengths | Weaknesses |
|---|---|
| Open source (MIT), free, runs in any modern browser | Manual editing only, with no batch automation |
| Complete set of selection tools (brush, box, sphere, lasso, color) | Very large splats can be slow in the browser |
| Transform, crop, merge and color adjustment | Undo history and project management are basic compared with 3D software |
| Exports to `.ply` and compressed formats, and can publish web viewers | |

**Effect on the result:** the reference tool for manual cleanup, covering every common editing operation without installation.

### SplatTransform (PlayCanvas, command line)

**How it works:** an open-source command-line tool that converts, transforms, filters and merges splat files.

| Strengths | Weaknesses |
|---|---|
| Scriptable and reproducible: the same operations can be applied to every scene | No visual feedback, so parameters must be known in advance |
| Format conversion between `.ply`, compressed formats and others | Filtering is rule-based only |
| Fits into an automated pipeline | Cannot target a specific artifact the way manual selection can |

**Effect on the result:** complements a visual editor: manual editing defines the operations, the command line applies repeatable steps such as transforms, filtering and compression.

### LichtFeld Studio (built-in editing)

**How it works:** the training application includes selection, cropping and deletion tools, so cleanup can happen right after training.

| Strengths | Weaknesses |
|---|---|
| No export/import round trip between training and editing | Editing features are less complete than a dedicated editor |
| Cropping can be applied during training, avoiding wasted Gaussians | Tied to one trainer and its NVIDIA requirement |
| Open source | Young project, so features change between releases |

**Effect on the result:** convenient for quick cleanup when this trainer is already in use.

### Game engine plugins (Unity, Unreal)

**How it works:** plugins load splats into a game engine. Some, such as the open-source Unity Gaussian Splatting project, include selection, deletion and cut-out volumes inside the editor.

| Strengths | Weaknesses |
|---|---|
| Editing happens directly in the final application environment | Editing tools are secondary to rendering, and less complete than a dedicated editor |
| Cut-out volumes are non-destructive and can be adjusted later | Plugin quality, maintenance and licenses vary widely |
| Splats can be combined with meshes, lighting and interactions | Requires game engine knowledge |

**Effect on the result:** useful when the final deliverable is built in a game engine, for adjustments that depend on the final scene.

### Blender add-ons

**How it works:** add-ons (such as the free 3DGS Render add-on by KIRI Engine) import splats into Blender for viewing, editing and combining with other 3D content.

| Strengths | Weaknesses |
|---|---|
| Uses Blender's full toolset for transforms, alignment and composition | Blender is not designed for millions of Gaussians, so performance is limited on large scenes |
| Free, and familiar to 3D artists | Editing is less direct than in a dedicated splat editor |
| Good for aligning splats with meshes or measured geometry | Each add-on's license and maintenance must be checked |

**Effect on the result:** useful for composition and precise alignment, less so for routine cleanup.

### Scripted editing (Python)

**How it works:** custom scripts read the `.ply` file (for example with `plyfile` or NumPy) and filter or transform Gaussians programmatically.

| Strengths | Weaknesses |
|---|---|
| Fully controllable and reproducible | Requires development time |
| Any rule can be implemented: opacity, size, density, bounding volume | No visual feedback without a separate viewer |
| Integrates directly into a benchmark or processing pipeline | Must handle each format's specifics (spherical harmonics, compressed attributes) |

**Effect on the result:** the most flexible option for automation, particularly for applying identical cleanup to every scene in a comparison.

### Semantic editing methods (research)

**How it works:** methods such as Gaussian Grouping or SAGA attach object labels to Gaussians using 2D segmentation models, so objects can be selected, removed or extracted by name or click.

| Strengths | Weaknesses |
|---|---|
| Object-level selection instead of manual brushing | Research code, not integrated into mainstream tools |
| Can remove objects consistently in 3D | Requires training with the method, not just editing an existing splat |
| | Removed objects still leave unobserved regions behind them |

**Effect on the result:** promising for removing specific objects, but not yet practical as a general editing tool.

---

## 3. Summary

| Option | Open source | Interface | Automation | Operations covered | Maturity |
|---|---|---|---|---|---|
| SuperSplat | Yes (MIT) | Browser / desktop | No | All common operations | High |
| SplatTransform | Yes | Command line | Yes | Transform, filter, merge, convert | Medium |
| LichtFeld Studio | Yes (copyleft) | Desktop GUI | No | Selection, crop, delete | Medium |
| Unity / Unreal plugins | Varies | Game engine | Partial | Selection, cut-outs, composition | Varies |
| Blender add-ons | Varies | Blender | Partial (Python) | Transform, composition | Medium |
| Python scripts | Yes | Code | Yes | Any rule-based operation | Custom |
| Semantic methods | Yes (research) | Code | Yes | Object selection and removal | Low |

## 4. Hypotheses

Hypotheses suggested by this comparison, to be tested experimentally:

1. Manual cleanup in a dedicated editor removes most visible artifacts, but its time cost and variability grow with scene size.
2. Rule-based filtering (opacity, size, bounding volume) removes a large share of floaters automatically, with limited loss of valid detail.
3. A combination of scripted operations for repeatable steps and manual editing for remaining artifacts gives the best ratio of quality to effort.
4. Compression to a compact format reduces file size substantially with no visible quality loss at normal viewing distances.
