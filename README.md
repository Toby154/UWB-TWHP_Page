# UWB-TWHP Dataset

English | [简体中文](README_zh.md)

UWB-TWHP is an ultra-wideband radar dataset for through-wall 3D human pose estimation in indoor scenes. The dataset was synchronously collected using a self-developed UWB through-wall radar system and an optical motion capture system. It provides pulse-compressed radar range profiles, synchronized 3D human skeleton labels, and action labels, and can be used for through-wall human pose reconstruction, multi-person sensing, action analysis, and radar signal processing research.

## Dataset Overview

| Item | Description |
| --- | --- |
| Sensor | UWB radar, S-band, 2.5 GHz center frequency, 1 GHz bandwidth, FMCW, 8T8R TDM-MIMO, 26 Hz |
| Reference labels | 3D human skeletons from a NOKOV optical motion capture system |
| Scenario | Indoor through-wall scene with an approximately 23.8 cm thick wall |
| Subjects | 15 volunteers, anonymized as H0-H14 |
| Number of people | One-person, two-person, and three-person scenes |
| Action classes | 8 common human actions |
| Dataset scale | 1,882 trials, stored as 3,764 `.mat` files, with 2,293,076 synchronized radar-pose frames |
| Dataset size | Approximately 524.1 GB |
| File format | MATLAB v7.3 `.mat` |
| Default split | The first 80% of frames in each trial are used for training, and the last 20% for testing (a splitting tool is provided for repartitioning) |
| Additional resources | Through-wall empty-background data, small-scale non-through-wall data, a dataset splitting tool, and a visualization tool |
| License | CC BY 4.0 |

## Data Access

The dataset is openly accessible on [Science Data Bank](https://www.scidb.cn/): [DOI: 10.57760/sciencedb.41175](https://doi.org/10.57760/sciencedb.41175), [CSTR: 31253.11.sciencedb.41175](https://cstr.cn/31253.11.sciencedb.41175). Use this entry to download the data.

The dataset includes the following release files:

| Archive | Contents |
| --- | --- |
| `UWB-TWHP.7z.001`–`UWB-TWHP.7z.027` | Multipart archive of the main through-wall dataset, approximately 524.1 GB after extraction |
| `UWB-TWHP_background.7z` | Through-wall empty-background range profiles; `part1` and `part2` each contain 29 `.mat` files, with `file_background_lookup.csv` for trial-to-background mapping |
| `UWB-TWHP_non-through-wall.7z` | Small-scale non-through-wall data: 126 one-person `.mat` files in `radardata/` and one empty-background `.mat` file in `background/` |
| `UWB-TWHP_Dataset_Splitter.zip` | MATLAB tool for repartitioning data, retaining complete trials, and optionally adding range coordinates |
| `UWB-TWHP_Dataset_Visualizer.zip` | MATLAB tool for inspecting radar data, poses, actions, and confidence labels |

Download all 27 main-dataset volumes into the same directory and extract only `UWB-TWHP.7z.001`. The extraction software will automatically read the remaining volumes and extract the complete dataset; do not extract each volume separately.

Related code repositories:

- Data post-processing and visualization code: <https://github.com/Toby154/UWB-TWHP_Post-processing.git>
- Baseline training, testing, and visualization code: <https://github.com/Toby154/UWB-TWHP_baseline.git>

## Directory Structure

The dataset is organized by the number of people and the train/test split:

```text
UWB-TWHP/
├── 1/
│   ├── train/radardata/*.mat
│   └── test/radardata/*.mat
├── 2/
│   ├── train/radardata/*.mat
│   └── test/radardata/*.mat
└── 3/
    ├── train/radardata/*.mat
    └── test/radardata/*.mat
```

![UWB-TWHP dataset directory structure](pic/fig7.png)

`1`, `2`, and `3` denote one-person, two-person, and three-person scenes, respectively.

## File Naming Rules

One-person scene:

```text
N1_Hh_Aa_Cc_train.mat
N1_Hh_Aa_Cc_test.mat
```

Two-person scene:

```text
N2_Hh1-h2_Aa1-a2_Cc_train.mat
N2_Hh1-h2_Aa1-a2_Cc_test.mat
```

Three-person scene:

```text
N3_Hh1-h2-h3_Aa1-a2-a3_Cc_train.mat
N3_Hh1-h2-h3_Aa1-a2-a3_Cc_test.mat
```

Naming fields:

- `N1`, `N2`, `N3`: number of people in the scene.
- `H`: subject ID. For example, `H0-1` denotes subjects H0 and H1.
- `A`: action ID. For example, `A2-3` means the corresponding subjects perform A2 and A3.
- `C`: repeated collection index for the same action or action combination under different positions or paths.
- `train`, `test`: training or testing split.

In multi-person scenes, the subject order strictly corresponds to the action order. For example:

```text
N2_H0-1_A2-3_C1_train.mat
```

This file denotes a two-person scene where subject H0 performs action A2 and subject H1 performs action A3. It is the training file from the first repeated collection.

## Variables in `.mat` Files

Each `.mat` file mainly contains the following variables:

| Variable | Size | Description |
| --- | --- | --- |
| `action` | `n × 1 × f` | Action labels, where `n` is the number of people and `f` is the number of frames |
| `joint` | `n × 48 × f` | 3D human pose labels in the radar coordinate system, in metres |
| `conf` | `n × 1 × f` | Target existence confidence label; all entries in the released main dataset are `1` |
| `radar_data_real` | `512 × 64 × f` | Real part of the pulse-compressed radar range profiles |
| `radar_data_imag` | `512 × 64 × f` | Imaginary part of the pulse-compressed radar range profiles |

The sizes above follow MATLAB dimension order. The radar arrays are stored as single-precision floating-point values. Channels follow `c = 8 × (p - 1) + q`, where `p` and `q` are the 1-based transmitter and receiver indices; the receiver index varies fastest.

The second dimension of `joint` has 48 values, representing the 3D coordinates of 16 joints:

- Indices `1-16`: `x` coordinates of the 16 joints;
- Indices `17-32`: `y` coordinates of the 16 joints;
- Indices `33-48`: `z` coordinates of the 16 joints.

Coordinate definitions:

- `x`: azimuth dimension;
- `y`: range dimension;
- `z`: height dimension.

The radar coordinate system is right-handed. Its origin is the vertical projection of the radar array centre onto the ground. The `x` axis is parallel to the wall, the positive `y` axis points from the radar towards the subject activity area, and the positive `z` axis points upwards. The array centre is at `(0, 0, 1.2)` m. All released pose labels have already been transformed into this common coordinate system across acquisition sessions.

The order of the 16 joints is:

| Index | Joint |
| --- | --- |
| 1 | Head |
| 2 | Neck |
| 3 | Spine |
| 4 | Spine3 |
| 5 | LeftArm |
| 6 | LeftForeArm |
| 7 | LeftHand |
| 8 | RightArm |
| 9 | RightForeArm |
| 10 | RightHand |
| 11 | LeftUpLeg |
| 12 | LeftLeg |
| 13 | LeftFoot |
| 14 | RightUpLeg |
| 15 | RightLeg |
| 16 | RightFoot |

![Definition of the 16 3D human joints](pic/fig6.png)

## Action Labels

| Action ID | Numeric label | Action name |
| --- | --- | --- |
| A0 | 0 | Sitting |
| A1 | 1 | Squatting |
| A2 | 2 | Standing |
| A3 | 3 | Punching |
| A4 | 4 | Kicking |
| A5 | 5 | Bending |
| A6 | 6 | Waving |
| A7 | 7 | Walking |

Action labels are derived from the acquisition protocol and directory naming rules. They are not frame-by-frame manual action annotations.

Each person's action label remains constant throughout a trial. For example, a bending trial is labelled A5 during standing, bending forward, and returning to standing. Tasks requiring instantaneous action states need additional annotations.

## Radar Processing and Range Coordinates

The released complex range profiles are generated by channel reorganization, fast-time FFT, channel-wise range-offset compensation, complex amplitude–phase calibration, and range-window selection. Each channel has 3,940 retained ADC samples; processing uses a 102,400-point FFT and retains 512 range bins at a stride of two FFT bins.

The two acquisition sessions use different range origins:

| Range-axis file | Two-way propagation distance (m) | Retained sampling interval (m) |
| --- | --- | --- |
| `range_axis_part1.csv` | 4.6875–16.6640625 | 0.0234375 |
| `range_axis_part2.csv` | 6.55078125–18.52734375 | 0.0234375 |

These axes express two-way propagation distance, whereas pose labels use spatial coordinates in the radar coordinate system. The range-axis sampling interval is distinct from the approximately 15 cm physical range resolution.

Use [`range_mapping/file_range_lookup.csv`](https://github.com/Toby154/UWB-TWHP_Post-processing/tree/main/range_mapping) in the post-processing repository to select the appropriate range-axis file. Its `file_name` field is the complete trial identifier without `_train`, `_test`, or `.mat`; for example, `N1_H0_A0_C1` applies to both portions of that trial. The splitting tool can optionally save this one-dimensional range vector as `range_bin` in generated `.mat` files. Non-through-wall recordings use `range_axis_part2.csv`.

Neither empty-background subtraction nor inter-frame cancellation is applied to the released range profiles. For background suppression, use the separately provided background data and `file_background_lookup.csv` to select the applicable `part1` or `part2` group. The training and test portions of a trial use the same background group.

## Python Loading Example

The dataset is stored in MATLAB v7.3 `.mat` files. We recommend using `h5py` in Python:

```python
import h5py

mat_path = "UWB-TWHP/1/train/radardata/N1_H0_A2_C1_train.mat"

with h5py.File(mat_path, "r") as f:
    print(list(f.keys()))

    # Read the first 26 frames in h5py dimension order.
    action = f["action"][:26]                    # [frames, 1, people]
    conf = f["conf"][:26]                        # [frames, 1, people]
    joint = f["joint"][:26]                      # [frames, 48, people]
    radar_real = f["radar_data_real"][:26]        # [frames, 64, 512]
    radar_imag = f["radar_data_imag"][:26]
    radar_complex = radar_real + 1j * radar_imag

print(action.shape)
print(joint.shape)
print(radar_complex.shape)
```

For these MATLAB v7.3 files, `h5py` exposes dimensions in reverse order to MATLAB: radar arrays are `[f, 64, 512]` and pose arrays are `[f, 48, n]`. The baseline uses 26-frame windows with stride 1 within each file, concatenates real and imaginary components, and rearranges the input to `[52, 512, 64]`; pose, confidence, and action targets come from the final frame. See [`data/dataset.py`](https://github.com/Toby154/UWB-TWHP_baseline/blob/main/data/dataset.py) for the training loader.

## Experimental Environment

The experiment was conducted in an indoor open room with a length of 16.2 m, a width of 7.7 m, and a height of 3.3 m. To build a controllable through-wall sensing scenario, a common red brick wall with a length of 5.8 m and a thickness of 23.8 cm was constructed in the experimental area.

The wall divided the space into a radar operation area and a subject activity area. The radar system and acquisition computer were placed on one side of the wall, while the subjects and the optical motion capture system were placed on the other side. During acquisition, the radar antenna array was placed close to the wall to reduce the free-space propagation distance between the radar and the wall and to improve the observability of human echoes behind the wall.

The subject activity area was equipped with optical motion capture cameras. Floor mats and position markers were used to indicate standing points, motion paths, and activity boundaries. In the radar coordinate system defined above, the subject activity area covered approximately -2.7 m to 2.2 m in the azimuth dimension and 1.6 m to 8.4 m in the range dimension.

![Experimental environment and spatial layout](pic/fig3.png)

## Data Acquisition and Synchronization

The radar recorded through-wall human echoes at 26 Hz, while the motion capture system output 3D skeletons at 100 Hz. To establish frame-level correspondence between radar data and motion capture data, the acquisition system used radar frame numbers, FPGA synchronization pulse counting, and unified software timestamps for online matching.

The synchronization procedure is:

1. The radar control software sends a reset message to the FPGA and sends a start-acquisition command to the radar.
2. The radar starts returning echo data and outputs a synchronization pulse at the beginning of each frame.
3. The FPGA receives the synchronization pulse, generates the synchronization frame number and message, and sends the message to the radar control software through a serial port.
4. The acquisition software parses the radar frame number from the radar echo data and the synchronization frame number from the FPGA synchronization message.
5. The software first binds the radar echo frame to the synchronization event according to frame-number consistency.
6. The optical motion capture system outputs pose data synchronously, and the acquisition software buffers the pose data and parses its timestamp.
7. Because the radar and motion capture systems have different frame rates, the software uses nearest-neighbor timestamp matching to bind each radar frame to the closest 3D human skeleton frame in time.
8. If the nearest-neighbor time difference is smaller than 38 ms, the match is considered valid; otherwise, the sample is discarded.

![Synchronization workflow between radar data and motion capture data](pic/fig5.png)

During acquisition, only valid matched samples are saved, including radar frame numbers, pulse-compressed radar range profiles, matched 3D skeletons, and pose timestamps. If packet loss prevents frame-number or timestamp matching, the sample is not saved. The released post-processed `.mat` files contain the five variables listed above; acquisition frame numbers and timestamps are not included in those files.

Note that the radar frame timestamp is recorded by the acquisition software when it receives the FPGA serial synchronization message, rather than being generated directly by the FPGA local hardware clock. Therefore, the timestamp may be affected by serial transmission delay and computer system scheduling delay. The dataset reduces this influence through a unified acquisition software clock, frame-number consistency checks, and timestamp threshold constraints.

The released dataset consists of manually checked and post-processed `.mat` files. It does not include raw radar echo files or raw motion capture JSON files.

## Baseline Results

The dataset provides a CNN + DETR-style Transformer baseline for multi-person pose estimation and action recognition. A CNN extracts radar features, and a Transformer decoder predicts target confidence, 3D bounding boxes, 16-joint poses, and eight action categories. The model has 161.02 M parameters and requires 94.10 GFLOPs for a single forward pass.

![Baseline architecture](pic/baseline_architecture.png)

### Evaluation Protocol

- **Time-based split:** the first 80% of frames in each trial are used for training and the remaining 20% for testing. This within-trial split is a relatively simple evaluation setting and carries a risk of data leakage from temporal correlation.
- **Subject-based split:** all complete trials containing H1 are assigned to testing: one-person H1, two-person H0-1 and H1-8, and three-person H0-1-2 and H1-8-14. All other trials form the training set. H1 is held out from training; other people appearing alongside H1 may also occur in training trials.
- **Pose and person detection:** predictions use a confidence threshold of 0.9 and 3D bounding-box NMS with IoU threshold 0.1. Hungarian matching minimizes 16-joint MPJPE; pairs exceeding 500 mm are rejected. MPJPE measures absolute 3D joint error in radar coordinates, without translation, rotation, or scale alignment, and is averaged over matched persons. Unmatched ground-truth persons and predictions count as misses and false alarms.
- **Action detection:** each action class is evaluated independently. Predictions are ranked by target confidence multiplied by action probability and assigned to the nearest unmatched ground-truth person of that class with MPJPE no greater than 500 mm.
- **Aggregation:** recall and miss rate use the ground-truth count as denominator; precision and false-alarm rate use the prediction count. F1 is the harmonic mean of precision and recall. Overall results pool person-level errors and detection counts rather than averaging group scores.

### Overall Results

MPJPE is in **mm**; all other values in the following tables are **percentages**.

| Metric | Time-based split | Subject-based split (H1) |
| --- | --- | --- |
| Pose MPJPE (mm) | 61.682 | 130.51 |
| Person miss rate | 1.2819 | 4.3624 |
| Person false-alarm rate | 3.0202 | 4.4854 |
| Person recall | 98.718 | 95.638 |
| Person precision | 96.98 | 95.515 |
| Person F1 | 97.841 | 95.576 |
| Action miss rate | 1.6125 | 32.671 |
| Action false-alarm rate | 3.3449 | 32.757 |
| Action recall | 98.387 | 67.329 |
| Action precision | 96.655 | 67.243 |
| Action F1 | 97.514 | 67.286 |

### Pose and Person Detection by Number of People

| Split | Scene | MPJPE (mm) | Miss rate | False-alarm rate | Recall | Precision | F1 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Time | One person | 59.04 | 0 | 0.01865 | 100 | 99.981 | 99.991 |
| Time | Two people | 67.576 | 2.0446 | 6.4903 | 97.955 | 93.51 | 95.681 |
| Time | Three people | 57.267 | 1.3482 | 1.1953 | 98.652 | 98.805 | 98.728 |
| Subject (H1) | One person | 113.32 | 0.42761 | 0.31149 | 99.572 | 99.689 | 99.63 |
| Subject (H1) | Two people | 135.06 | 3.5676 | 4.5798 | 96.432 | 95.42 | 95.924 |
| Subject (H1) | Three people | 131.25 | 5.8562 | 5.3792 | 94.144 | 94.621 | 94.382 |

### Pose Error and Action F1 by Action Category

| Action | Time MPJPE (mm) | Time action F1 | Subject MPJPE (mm) | Subject action F1 |
| --- | --- | --- | --- | --- |
| A0 Sitting | 25.791 | 99.709 | 130.12 | 85.984 |
| A1 Squatting | 31.509 | 99.902 | 169.24 | 89.554 |
| A2 Standing | 27.469 | 99.135 | 77.591 | 39.836 |
| A3 Punching | 53.849 | 99.412 | 139.12 | 47.061 |
| A4 Kicking | 58.86 | 98.912 | 102.07 | 56.896 |
| A5 Bending | 71.285 | 98.833 | 137.36 | 55.99 |
| A6 Waving | 56.743 | 99.01 | 132.74 | 52.674 |
| A7 Walking | 123.52 | 91.353 | 152.89 | 84.936 |

### Qualitative Results

Action confusion matrices are row-normalized, with true actions in rows and predicted actions in columns. In pose visualizations, blue denotes predictions and red denotes ground truth; action labels appear below the poses.

Time-based split:

![Action confusion matrices under the time-based split](pic/time_action_confusion.png)

![Pose estimation and action recognition under the time-based split](pic/time_pose_results.png)

Subject-based split:

![Action confusion matrices under the subject-based split](pic/subject_action_confusion.png)

![Pose estimation and action recognition under the subject-based split](pic/subject_pose_results.png)

### Code and Saved Results

The [baseline repository](https://github.com/Toby154/UWB-TWHP_baseline) provides training and testing code, checkpoints, exported predictions, and a MATLAB pose/action player. Saved results are in `results_under_the_time-based_split/` and `results_under_the_subject-based_split/`; the corresponding checkpoints use Git LFS. See its English and Chinese README files for setup and evaluation instructions.

To reproduce the results above, pool matched-person errors and perform class-wise action matching according to the evaluation protocol. The current Python `test.py` export `mean_mpjpe` averages per-sample means; its `matched_action_accuracy` measures classification accuracy among pose-matched persons and differs from class-wise action detection F1. These summary fields cannot directly substitute for the metrics in the tables.

## Dataset Tools

### Repartitioning and Adding Range Coordinates

Extract `UWB-TWHP_Dataset_Splitter.zip` and run `START.m` in MATLAB. Select Chinese or English, the source dataset directory, a separate output directory, and a generation speed. Higher speed settings generally require more memory.

The tool supports chronological splitting by a configurable frame ratio, splitting by selected subjects, splitting by complete trials, and retaining complete trials for custom partitions. It first combines the original training and test portions of each trial. For a subject-based split, every trial containing any selected test subject is assigned in full to testing.

Select **Add distance information (range_bin)** to include the applicable two-way propagation distance vector in each generated file. Review trial/frame counts and output settings before starting. **Overwrite source** replaces the source dataset without a backup; use a separate output directory to retain the original data.

![Dataset splitting tool](pic/dataset_splitter.png)

### Inspecting Dataset Files

Extract `UWB-TWHP_Dataset_Visualizer.zip` and run `START.m` in MATLAB. Use **Select MAT File** to load a dataset file. The tool displays radar real/imaginary components, 3D pose ground truth, a range-time map, range and Doppler spectra, action labels, and target confidence labels, with playback and frame navigation controls.

![Dataset visualization tool](pic/dataset_visualizer.png)

The source versions are also available as `dataset_splitter.m` and `dataset_visualizer.m` in the [post-processing repository](https://github.com/Toby154/UWB-TWHP_Post-processing).

## Usage Notes

- Select the time-based or H1 subject-based protocol according to the evaluation objective. The default within-trial split does not measure unseen-subject generalization.
- For custom generalization studies, rebuild splits by subjects, actions, scene person counts, or complete trials, and report the exact partition. Keep each trial intact when evaluating trial-independent performance.
- In multi-person scenes, the subject order, action order, and the person order in `action` and `joint` must be kept consistent.
- Walking sequences are longer and therefore contain more frames. Pay attention to class distribution when performing action classification or balanced training.
- All data were collected in one indoor environment; the dataset does not support direct evaluation across different acquisition environments.
- Subjects generally stayed at least 1 m apart, apart from some walking crossings and occlusions. Challenging inter-person occlusion and radar target mixing samples are limited.
- Participants were healthy adults. The dataset contains no patient recordings.
- Action labels describe trial-level activities; use additional annotations for frame-level action-state analysis.

## License

The dataset is shared under the CC BY 4.0 license. The related code is released under the MIT license.

## Citation

If you use this dataset, please cite the dataset entry and the accompanying paper, *Through-wall 3D Human Pose Estimation Dataset Using UWB MIMO Radar*. Cite the dataset as follows:

> Dai, Y. et al. UWB-TWHP: through-wall 3D human pose estimation dataset using UWB radar. Science Data Bank (2026). <https://doi.org/10.57760/sciencedb.41175>

The paper's final bibliographic details will be added when available.
