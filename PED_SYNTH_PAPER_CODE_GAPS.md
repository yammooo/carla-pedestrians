# PedSynth++: paper claims versus this checkout

This note compares the [ARCANE-PedSynth paper](https://arxiv.org/abs/2605.24950) with the pinned `pedestrians-scenarios` checkout (`2e47ab3`), mainly [`free_drive_front_cam_v2.py`](pedestrians-scenarios/src/pedestrians_scenarios/scenarios/free_drive_front_cam_v2.py). It describes current output, not a dataset specification.

| Paper describes | Current code and one 4-second smoke clip | Effort to match the claim |
| --- | --- | --- |
| RGB, LiDAR, and DVS | Generated 120 RGB PNGs, 120 LiDAR `.bin` files, 120 DVS `.npz` files, and an MP4. | Already generates these files. Exact sensor alignment still needs checking. |
| Synchronized sensors | Files share output frame numbers, but the exporter does not save CARLA sensor frame IDs, timestamps, or poses; it takes the next item from each queue. | Needs frame matching and saved timing metadata. |
| 2D boxes, pedestrian IDs, distance, crossing | Present in CSV/JSON. The smoke clip writes every pedestrian-frame row twice. | Duplicate-row fix is small. Crossing semantics need review: code also labels some shoulder, parking, and near-road sidewalk positions as crossing. |
| Frame-wise, 12-state pedestrian behavior labels | **Not exported.** The active path uses only `WALKING_SIDEWALK`, `CROSSING_ROAD`, and `FINISHED_CROSSING`. | Exporting those three values is easy; producing the paper's 12 meaningful states requires substantial behavior logic and validation. |
| Time to first crossing | `crossing_point` stores a frame number (or first visible frame), not frames remaining until crossing. | Needs a defined onset and per-frame calculation. |
| COCO-17 pose | No pose keypoints in generator output; `SKELETON_KEYPOINTS` is empty. The paper describes separate AlphaPose post-processing. | Separate pipeline work; not validated here. |

## The important behavior gap

The paper says pedestrians can **look around, check traffic, and hesitate** before crossing. Those names exist in the code's enum, but the active generator never enters them. Fields such as `attention_span` and `hesitation_duration` are stored but not used to control these stages. Instead, a potential crosser switches directly from `WALKING_SIDEWALK` to `CROSSING_ROAD` when the ego vehicle is 10–40 m away and a random check succeeds. The paper's pre-crossing traffic-gap/TTC decision is not implemented in this path.

The enums differ too: the paper includes `RETREAT`, while the code has `NORMAL_CROSSING` instead. A retreat is recorded only by an internal flag and remains `CROSSING_ROAD`. Stopping for a vehicle does not enter `PAUSING_MID_CROSS`; faster crossing does not enter `RUNNING_ACROSS`. `FINISHED_CROSSING` may remain set while a pedestrian continues walking or after a retreat. Thus, simply adding an FSM column would **not** create the behavior labels described in the paper.

## Other limits

- CARLA supplies ego pose and pedestrian positions during generation, but this exporter saves neither as per-frame trajectories. The paper describes a moving ego vehicle; it does not clearly promise an ego-trajectory file.
- The inspected Hugging Face clip has the same CSV columns and duplicate rows, but its metadata disables LiDAR/DVS and it contains no such files. This is one inspected clip, not a claim about every released clip.
- The paper's documented `scenarios generate` command is not registered in this checkout. The smoke test called the scenario class directly without changing generator code.

**Conclusion:** This checkout can generate the raw sensor modalities. It cannot currently reproduce the paper's advertised frame-wise behavior annotations. The missing pre-crossing states need real implementation and a small-run validation before they can be treated as research labels.
