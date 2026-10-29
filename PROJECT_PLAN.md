# Thesis Project Plan

**Working title:** Predictive Multi-Person Tracking with Adaptive Identity Association under Occlusion  
**Version:** 0.1 — 29 October 2026  
**Current status:** GitHub repository created. Implementation and experiments have not yet been verified.

This is a working implementation and research plan. Model sizes, training settings, and the exact adaptive association rule will be selected through experiments.

## 1. Objective and research question

Build a system that processes a single-camera video, detects people, maintains their identities across frames, estimates future movement, and attempts to recover the correct identity after temporary occlusion.

Research question: Does combining learned trajectory prediction with association that adapts to motion uncertainty, appearance reliability, and time missing improve identity continuity compared with established trackers?

Success means a reproducible system and a careful evaluation of this question. Improvements and novelty are hypotheses to test, not assumed outcomes.

## 2. What the finished system should do

- Accept a recorded video or ordered image sequence.
- Detect people and assign track IDs.
- Display bounding boxes, IDs, observed movement trails, and predicted future positions.
- Keep track histories and manage new, active, temporarily lost, and removed tracks.
- Attempt identity recovery after a person reappears, while allowing uncertain matches to be rejected.
- Export tracking results, runtime measurements, configurations, and comparison summaries.
- Provide examples of successful recoveries and remaining failures.

Initial scope is single-camera, two-dimensional tracking. Multi-camera identity matching, facial recognition, 3D reconstruction, and action recognition are outside this initial plan. A live-camera interface is an optional extension.

## 3. Tracking comes before the learned predictor

Detection provides boxes for one frame. Tracking connects detections across time, maintains identities, and produces movement histories.

The first tracker will already use a simple motion model. A Kalman filter provides short-term position estimates; the later GRU is a learned alternative or extension to this motion component, not the mechanism that creates tracking identities.

At each frame, the eventual system will:

1. Detect people in the current image.
2. Predict where existing tracks should be at the current time, using their earlier histories.
3. Compare detections with predicted tracks using motion and, where available, appearance.
4. Assign compatible matches; allow tracks and detections to remain unmatched.
5. Update matched tracks, create new tracks, and maintain or retire lost tracks.
6. Save the updated histories and optionally forecast further into the future for display.

Steps 1 and 2 are conceptually independent inputs to association. Tracking, prediction, and association form a recurring loop.

## 4. Components and planned contributions

| Component | Starting point | Planned work |
| --- | --- | --- |
| Detector | Pretrained YOLO; small model for initial development | Integrate inference; run a pedestrian fine-tuning study; compare frozen-layer and broader fine-tuning strategies where useful |
| Tracking | ByteTrack, then BoT-SORT | Build the video pipeline, histories, exports, configuration, evaluation, and tracker integration; study matching and track lifecycle |
| Motion prediction | Existing tracker motion model; constant-velocity reference | Implement and train a GRU on annotated trajectories; compare forecasting and downstream tracking performance |
| Appearance/ReID | Pretrained person appearance model when needed | Integrate embeddings and appearance history; fine-tune only if evidence supports the additional work |
| Adaptive association | Fixed matching rule as a reference | Design adaptive cue weighting, gating, and rejection using uncertainty and occlusion duration; optionally compare a learned match scorer |
| Evaluation and application | TrackEval and a simple video runner | Reproducible experiments, overlays, saved outputs, measured runtime, failure analysis, and a demonstrable final system |

Using an established tracker initially means implementing the surrounding system and integrating its algorithm. It does not mean claiming the tracker itself as original work. We will modify or extend a clearly separated tracking/association module when testing our method.

Fine-tuning and training a standard GRU are substantial implementation and experimental work. The candidate original contribution is the specific adaptive association method and its demonstrated effects. Motion-plus-appearance matching already exists; related work must establish what our method changes.

## 5. Ordered milestones

### M0 — Record the scope and prepare the workspace

Tasks:
- Add this plan to the repository, for example as docs/PROJECT_PLAN.md.
- Add a short README describing the objective and current status.
- Confirm operating system, Python environment, GPU, and available memory.
- Create a dedicated environment and verify PyTorch GPU access if available.
- Record package versions; ignore datasets, weights, and generated outputs in Git.

Done when: the repository contains the plan and the selected environment can import the required packages.

### M1 — Person detection on a sequence

Tasks:
- Load one short development video or annotated image sequence.
- Run pretrained YOLO for the person class.
- Preserve frame ordering, timestamps or FPS, image size, boxes, and confidence scores.
- Save an annotated detection video.

Done when: one repeatable command produces a video with person detections.

### M2 — Working tracking baseline

Tasks:
- Integrate ByteTrack explicitly.
- Display IDs and store a movement history for each track.
- Save observed trails, annotated video, and MOT-compatible tracking results.
- Ensure tracker state persists within a sequence and resets between sequences.
- Inspect crossings, missed detections, disappearances, and reappearances.
- Add BoT-SORT as a comparison; record whether ReID is enabled.

Done when: the system maintains IDs across a complete sequence, saves its results, and documents concrete successes and failures.

### M3 — Evaluation and related-work foundation

Tasks:
- Evaluate annotated development/validation sequences with TrackEval.
- Record HOTA, IDF1, ID switches, and processing speed.
- Distinguish missing detections from incorrect identity association.
- Define training, validation, and final evaluation partitions before training.
- Read ByteTrack, BoT-SORT, and relevant prediction/occlusion association work; keep notes linking methods to our hypothesis.

Done when: we have a baseline results table, a reproducible evaluation command, a fixed data split, and a specific improvement hypothesis.

### M4 — Detector fine-tuning study

Tasks:
- Prepare pedestrian detection labels from training data, respecting annotation and ignore-region rules.
- Train from pretrained weights; compare a frozen-backbone strategy with broader fine-tuning if resources allow.
- Select checkpoints and settings using validation data.
- Evaluate detection quality and downstream tracking using the same tracker settings.
- Keep the original detector and record a negative result if fine-tuning does not improve generalization.

Done when: a documented comparison shows the effect of detector adaptation.

### M5 — Trajectory prediction

Tasks:
- Extract identity-consistent movement sequences from ground-truth training annotations.
- Define the position representation, temporal sampling, history duration, and prediction horizon.
- Start with image-plane coordinates on stationary-camera footage. Normalized image coordinates are not real-world metres.
- Use constant-velocity and Kalman-based references.
- Implement and train a GRU; compare prediction errors on held-out sequences.
- Integrate predictions into tracking and evaluate with noisy, incomplete observed histories.

Done when: the learned predictor is compared against simple alternatives both as a forecaster and as part of the tracker.

### M6 — Adaptive identity association

Tasks:
- Integrate appearance embeddings and a consistent track appearance history.
- Implement a fixed motion/appearance matching reference.
- Define uncertainty explicitly: initially use the tracker covariance; a GRU needs its own justified uncertainty estimate rather than assuming its point prediction supplies one.
- Design adaptive weights or gates using motion uncertainty, appearance reliability, and time missing.
- Handle unmatched observations, lost-track lifetime, and rejection of implausible matches.
- Compare fixed and adaptive association with identical detector outputs and motion settings.

Done when: the method has a clear mathematical description, an implementation, and a controlled evaluation of identity recovery.

### M7 — Final experiments, demonstration, and thesis evidence

Tasks:
- Evaluate the selected methods on held-out footage.
- Use MOT17 and extend to DanceTrack as planned for more varied motion and similar appearances.
- Test real occlusions and separately report controlled missing-detection experiments. Dropping detections is a proxy, not a full simulation of visual occlusion.
- Run component comparisons to isolate what each change contributes.
- Create a demonstration showing IDs, trails, forecasts, and recovery cases.
- Document limitations, runtime conditions, training details, and reproducible commands.

Done when: the results and demonstration support a defensible thesis conclusion, including any conditions where the proposed method fails.

Optional extensions after the main system works: ReID fine-tuning, a small neural match scorer, and a live-camera interface.

## 6. Data and experimental rules

- Begin development with one MOT17 training sequence, such as MOT17-02.
- Do not treat MOT17 DPM/FRCNN/SDP copies of the same base video as independent scenes. All copies of a base sequence must remain in the same partition.
- Split by base video/scene for this project; do not randomly mix overlapping trajectory windows or adjacent video frames across partitions.
- Use training data for fitting weights, validation data for selecting settings, and untouched evaluation data for the final comparison.
- If ReID crops are used, preserve sequence-scoped person identities and prevent evaluation crops from entering training.
- Start trajectory learning with ground-truth histories, then test robustness to detector/tracker errors.
- Preserve or explicitly resample timestamps: a fixed number of frames represents different durations at different FPS.
- Report image-plane prediction units and forecast horizons consistently.
- Reuse identical detections when comparing association methods. Evaluate detector changes separately.
- Record implementation version, detector checkpoint, input resolution, thresholds, tracker settings, random seeds, data split, and runtime hardware.
- Report local comparisons under our own protocol. Do not imply reproduction of published leaderboard results with different models or data.

## 7. Planned evidence

| Question | Controlled comparison | Evidence |
| --- | --- | --- |
| Does detector fine-tuning help? | Original vs fine-tuned detector with the same tracker | Detection precision/recall and AP; downstream HOTA, IDF1, ID switches |
| Does learned prediction help? | Constant velocity/Kalman vs GRU under a fixed protocol | Average/final displacement error and downstream tracking metrics |
| Does adaptation help? | Fixed vs adaptive association with identical detections and predictor | HOTA, IDF1, ID switches; recovery examples and results by missing duration |
| What is the cost? | Same hardware and input settings | Runtime and throughput; separate processing from visualization/export where possible |

Any custom recovery metric must specify which occlusions count, the allowed recovery window, and how incorrect identity assignments are penalized. It should supplement standard metrics.

## 8. Next work session

The immediate next step after saving the plan is environment setup, followed by M1.

Required setup details still to confirm:
- Windows with Anaconda, WSL/Linux, or another environment.
- GPU model and available VRAM.
- Repository URL if direct repository assistance is wanted.

First session target:
- [ ] Plan saved in the repository.
- [ ] Dedicated environment created and checked.
- [ ] One input sequence accessible.
- [ ] First detection run saved.

Next target: add ByteTrack IDs and movement histories. No trajectory-network training is required to reach this baseline.

## 9. Primary references

- Ultralytics tracking documentation: https://docs.ultralytics.com/modes/track/
- Ultralytics training documentation: https://docs.ultralytics.com/modes/train/
- ByteTrack official repository and paper links: https://github.com/FoundationVision/ByteTrack
- BoT-SORT official repository and paper links: https://github.com/NirAharon/BoT-SORT
- MOT17 dataset: https://motchallenge.net/data/MOT17/
- DanceTrack dataset: https://github.com/DanceTrack/DanceTrack
- TrackEval: https://github.com/JonathonLuiten/TrackEval
- Torchreid fine-tuning documentation: https://github.com/KaiyangZhou/deep-person-reid/blob/master/docs/user_guide.rst

