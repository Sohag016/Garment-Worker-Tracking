# 🧵 Garment Worker Tracking: Pose, Hand Movement & Working Time

Track every worker in a garment-factory video using **YOLO11 Pose + ByteTrack**. The pipeline estimates each person's body pose, measures hand (wrist) movement, and calculates **working vs. idle time per person**.

Runs end-to-end in **Google Colab**. Upload a video, get back an annotated video and a JSON report.

---

## ✨ Features

- 👥 **Multi-person tracking** with persistent IDs (ByteTrack, tuned for occlusion)
- 🦴 **Body pose estimation** (17 keypoints, YOLO11 Pose)
- ✋ **Hand movement analysis**: left/right wrist distance, normalized by body size
- ⏱️ **Working / idle time per person** based on hand-motion activity
- 🎥 **Annotated output video**: skeleton, ID, WORKING/IDLE label, wrist trails, live work timer
- 📄 **JSON report** with per-person statistics and working segments

---

## 🚀 Quick Start (Google Colab)

1. Open `garment_worker_tracking.ipynb` in [Google Colab](https://colab.research.google.com/).
2. Go to **Runtime → Change runtime type → T4 GPU**.
3. Run the cells in order:
   1. Install dependencies
   2. Upload your video
   3. Run tracking and analysis
   4. Convert and download the results

Outputs:

| File | Description |
|------|-------------|
| `output_tracked.mp4` | Annotated video (H.264) |
| `output_report.json` | Per-person analytics |

---

## ⚙️ How It Works

```
Video ─► YOLO11 Pose ─► ByteTrack (IDs) ─► Wrist keypoints (L/R)
                                               │
                        Frame-to-frame wrist displacement ÷ body height
                                               │
                     1-second moving average > threshold ?
                                  ├── Yes ─► WORKING
                                  └── No  ─► IDLE
                                               │
                        Accumulate time per person ─► Video + JSON
```

1. **Detection & pose**: YOLO11 Pose detects people and 17 body keypoints.
2. **Tracking**: ByteTrack assigns a persistent ID to each person.
3. **Hand movement**: wrist displacement per frame is divided by the person's bounding-box height, so results don't depend on distance from the camera.
4. **Activity decision**: if the average movement over a short window (default 1 s) exceeds `MOVE_THRESH`, the person is counted as *working*.
5. **Reporting**: times and distances are accumulated per person and exported.

---

## 🔧 Configuration

All settings are at the top of the processing cell:

| Parameter | Default | Description |
|-----------|---------|-------------|
| `MODEL_NAME` | `yolo11m-pose.pt` | `yolo11n-pose.pt` (fast) → `yolo11l-pose.pt` (accurate) |
| `CONF` | `0.35` | Detection confidence |
| `IMGSZ` | `960` | Inference image size |
| `VID_STRIDE` | `1` | Process every N-th frame |
| `MOVE_THRESH` | `0.006` | Hand-movement threshold for "working" |
| `WINDOW_SEC` | `1.0` | Smoothing window in seconds |
| `MIN_SEEN_SEC` | `2.0` | Ignore IDs seen for less than this |
| `KP_CONF` | `0.5` | Wrist keypoint confidence |

> 💡 **Tuning tip:** if everyone appears idle, lower `MOVE_THRESH` (e.g. `0.003`). If everyone appears working, raise it. Seated sewing-machine operators usually need a lower value than standing workers.

---

## 📄 Sample JSON Output

```json
{
  "video": { "fps": 25.0, "width": 1920, "height": 1080, "duration_sec": 600.0 },
  "total_people": 1,
  "people": [
    {
      "person_id": 1,
      "present_time": "00:09:40",
      "working_time": "00:07:12",
      "idle_time": "00:02:28",
      "working_percent": 74.5,
      "hand_movement": {
        "left_total_px": 18250.4,
        "right_total_px": 24310.9,
        "avg_normalized_per_sec": 0.0312
      },
      "working_segments": [
        { "start": "00:00:05", "end": "00:03:20" }
      ]
    }
  ]
}
```

*(Values above are illustrative only.)*

---

## ⚠️ Limitations

- **ID switches** can happen when workers are heavily occluded or sit very close together. One person may then appear with multiple IDs.
- "Working" is inferred from **hand motion only**. It does not recognize the actual task or detect subtle work with very small hand movements.
- Accuracy depends on camera angle, resolution, and lighting. Top-down or frontal views work best.
- Work-time analytics on real people should be used responsibly and in line with local privacy and labor regulations.

---

## 🗺️ Roadmap

- [ ] Re-identification to merge split IDs
- [ ] Per-workstation zones
- [ ] Task/action classification (e.g. sewing, cutting, folding)
- [ ] CSV / dashboard export
- [ ] Real-time RTSP stream support

---

## 🧰 Tech Stack

[Ultralytics YOLO11](https://docs.ultralytics.com/) · ByteTrack · OpenCV · NumPy · Google Colab

---

## 📜 License

Add your preferred license here (e.g. MIT). Note that **Ultralytics YOLO is licensed under AGPL-3.0**, so check its terms for commercial use.

---

## 🙌 Acknowledgements

- [Ultralytics](https://github.com/ultralytics/ultralytics) for YOLO11
- [ByteTrack](https://github.com/ifzhang/ByteTrack) authors
