# Capture Methods: Theoretical Comparison

Theoretical comparison of capture choices for 3D Gaussian Splatting: how each option works, its strengths and weaknesses, and its effect on pose estimation and training.

Four factors are compared: the camera, the camera movement, the wall texture, and depth sensing. A final section covers lighting and camera settings, which affect every option.

---

## 1. Camera

### Phone

**How it works:** a small sensor behind a wide-angle lens, with heavy automatic processing (HDR merging, noise reduction, sharpening) applied to every frame.

| Strengths | Weaknesses |
|---|---|
| Always available, no cost, easy to reproduce | Small sensor: more noise in low light, less fine detail |
| Wide lens gives good overlap between frames | Automatic processing can change from frame to frame, creating color and brightness inconsistencies between views |
| Many apps can lock exposure and focus | Video is heavily compressed, losing detail on subtle textures like walls |
| Some models include LiDAR (see section 4) | Rolling shutter: fast movement distorts frames |

**Effect on reconstruction:** inconsistent processing between frames forces the trainer to reconcile views that disagree, which produces color flickering or floaters. Compression removes exactly the faint texture that pose estimation could have used on walls.

### Dedicated camera (mirrorless)

**How it works:** a large sensor, interchangeable lenses, full manual control, and the option to shoot RAW or lightly compressed images.

| Strengths | Weaknesses |
|---|---|
| Large sensor: less noise, more detail, better in dim rooms | Cost, and requires some photography skill |
| Full manual control: exposure, white balance and focus can be locked for the whole capture | Heavier, slower to operate |
| Sharp still photos at high resolution, no heavy processing | Harder to reproduce by non-experts |
| Choice of lens (a wide lens increases overlap) | Very wide lenses add distortion, which must be modeled correctly in pose estimation |

**Effect on reconstruction:** consistent, detailed images make both pose estimation and training more reliable. Generally the highest-quality option for a given capture path.

### 360° camera

**How it works:** two or more fisheye lenses capture the whole sphere around the camera at once.

| Strengths | Weaknesses |
|---|---|
| Sees the entire room in each shot, so full coverage needs far fewer passes | Resolution is spread over the whole sphere, so each wall gets fewer pixels |
| Every frame shares content with many others, which helps pose estimation | Stitching seams between lenses can create artifacts |
| Fast capture | The operator is always in frame unless hidden or removed |
| | Images must be converted to standard perspective views, or the trainer must support fisheye or spherical cameras |

**Effect on reconstruction:** very robust for poses thanks to the large overlap, but lower sharpness per surface. Good for fast, complete coverage, weaker for fine detail.

### Side note: video vs. photos

Whatever the camera, extracting frames from a video is fast and guarantees overlap, but frames suffer from motion blur, compression and rolling shutter. Still photos are sharper and higher quality, but slower to take and coverage depends on the operator's discipline.

---

## 2. Camera movement

### Handheld

**How it works:** the operator walks through the room holding the camera.

| Strengths | Weaknesses |
|---|---|
| No equipment, fully flexible, can reach any angle | Shake causes motion blur |
| Fast | Coverage depends on the operator: easy to miss areas or vary height |
| | Not reproducible from one capture to the next |

### Gimbal

**How it works:** a motorized stabilizer cancels hand shake and keeps the camera orientation steady.

| Strengths | Weaknesses |
|---|---|
| Smooth movement, much less motion blur | Still depends on the operator's path, so coverage gaps remain possible |
| Cheap and easy to use | Adds some weight and setup |

### Tripod robot on a predefined path

**How it works:** the camera is mounted on a mobile tripod or robot that follows a programmed pattern (for example, stops at fixed positions and heights, rotating to capture every direction).

| Strengths | Weaknesses |
|---|---|
| Systematic coverage: same heights, angles and spacing everywhere | Development or purchase cost, and setup time |
| Camera can stop for each shot: no motion blur | Only reaches positions accessible from the floor at the robot's heights |
| Fully reproducible across captures and rooms | Slower capture |
| Poses could come from the robot's own position tracking | Obstacles in the room constrain the path |

**Effect on reconstruction:** movement mainly affects sharpness (blur hurts both pose estimation and training) and coverage (gaps become holes or floaters). A controlled path addresses both, and its reproducibility matters when many rooms must be captured.

---

## 3. Wall texture

### Bare walls

**How it works:** the room is captured as it is.

| Strengths | Weaknesses |
|---|---|
| The reconstruction matches the real room, nothing to remove | White walls give no features to detect and match |
| No preparation needed | Pose estimation may fail or give wrong poses on wall-facing images |
| | Training has little to anchor Gaussians on walls: blurry surfaces, holes and floaters |

### Added texture (posters, stickers, markers)

**How it works:** temporary visual patterns are placed on walls to create features.

| Strengths | Weaknesses |
|---|---|
| Gives pose estimation features to match, directly addressing the main failure point | The patterns appear in the final reconstruction and must be accepted or edited out |
| Cheap and simple | Editing them out of the splat leaves the original problem (little information about the bare wall behind them) |
| Special markers (such as ArUco or AprilTag) also give a known real-world scale | Setup time, and may not be allowed in some environments |

**Variant:** placing textured objects in the room (on the floor or furniture, not on walls) helps pose estimation while keeping walls clean, but helps less for images that only see a wall.

**Effect on reconstruction:** this is the cheapest way to make pose estimation work on white walls. The trade-off is fidelity: the scene is no longer exactly the real room.

---

## 4. Depth sensing

### Camera only

**How it works:** 3D structure and camera poses are deduced from the images alone, by pose estimation (Structure-from-Motion).

| Strengths | Weaknesses |
|---|---|
| Cheapest, works with any camera | Fully dependent on visual features, so fragile on white walls |
| All open-source pipelines support it directly | No real-world scale unless a known reference is in the scene |

### Phone LiDAR

**How it works:** a small LiDAR sensor on recent iPhone and iPad Pro models measures depth, which the phone combines with its motion sensors to track its own position.

| Strengths | Weaknesses |
|---|---|
| Tracking does not rely only on image features, so more robust on white walls | Short range (about 5 m) and coarse resolution |
| Real-world scale | Mostly used through proprietary apps, harder to feed into an open-source pipeline |
| No extra cost if the device is already available | Limited to specific phone models |

### Professional LiDAR scanner (LiDAR + cameras)

**How it works:** a dedicated device combines a high-quality LiDAR with calibrated cameras, and tracks its own position with SLAM (Simultaneous Localization and Mapping).

| Strengths | Weaknesses |
|---|---|
| Accurate geometry and poses regardless of texture: white walls are not a problem | Expensive (thousands of euros) |
| Real-world scale | Often tied to proprietary software |
| Dense point cloud gives training a strong starting point | Bulky and requires handling training |
| | LiDAR struggles with reflective and transparent surfaces, such as metal and glass |

**Effect on reconstruction:** LiDAR largely replaces image-based pose estimation and improves the starting point of training, but cameras are still required for appearance. It addresses the white-wall problem at the source, at the price of cost and closed tooling.

---

## 5. Lighting and camera settings (all options)

These are not options to choose between, but conditions that affect every choice above:

- **Constant lighting:** changing light (daylight through windows, lights switched on or off) makes the same surface look different across images, which the trainer interprets as view-dependent effects or floaters.
- **Locked exposure, white balance and focus:** automatic adjustments create the same inconsistencies.
- **Reflective surfaces:** lamps, metal and glass create reflections that move with the viewpoint. Gaussian Splatting models some view-dependent color, but strong reflections remain a weak point for every capture method.

---

## 6. Summary

| Choice | Cost | Expected quality | Robustness on white walls | Ease of reproduction |
|---|---|---|---|---|
| Phone | Very low | Medium | Low | High |
| Dedicated camera | Medium | High | Low to medium | Medium |
| 360° camera | Medium | Medium | Medium | Medium |
| Handheld | None | Medium | Unchanged | High |
| Gimbal | Low | Medium to high | Unchanged | High |
| Tripod robot | Medium to high | High | Unchanged | High once built |
| Bare walls | None | Depends on the rest | Low | High |
| Added texture | Low | Higher, with visible patterns | High | Medium |
| Camera only | None | Depends on the rest | Low | High |
| Phone LiDAR | Low if available | Medium | Medium to high | Medium |
| Professional LiDAR | High | High | High | Low |

## 7. Hypotheses

Hypotheses suggested by this comparison, to be tested experimentally:

1. Wall texture and depth sensing have the largest effect on pose estimation success.
2. Movement and camera choice mainly affect sharpness and fine detail, once poses are correct.
3. A dedicated camera on a controlled path, combined with some added texture, can approach LiDAR-level results at a fraction of the cost.
4. Reflective surfaces remain a limitation whatever the capture method.
