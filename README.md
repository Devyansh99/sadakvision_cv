# SadakVision — Road Defect & Garbage Detection

Object detection for Indian roads: **crack / pothole / manhole / garbage**.
Benchmarks **YOLOv8m vs YOLOv11n vs YOLOv26m vs RF-DETR** on the same merged dataset, with test-set prediction samples in-repo.

## Dataset

**Merged_Final_Data — [Roboflow](https://app.roboflow.com/ai-lisrp/merged_final_data/train)**

- Classes (4): `crack, garbage, manhole, pothole`
- Splits: train / valid / test (YOLO format from Roboflow)
- Not in git (too large). Download it yourself:
  1. Open link → Download → YOLOv8 format
  2. Unzip as `data/` next to `data.yaml`
  3. Train / val / predict

See `data.yaml` for the expected layout.

## Results (validation split)

| Model | mAP50 | mAP50-95 | Precision | Recall |
|---|---|---|---|---|
| YOLOv8m (100e, imgsz 640, batch 32) | **0.773** | **0.489** | 0.858 | 0.736 |
| YOLOv26m (100e, batch 16) | 0.757 | 0.386 | 0.824 | 0.737 |
| YOLOv11n (100e, imgsz 512, batch 64) | 0.633 | 0.294 | 0.721 | 0.594 |
| RF-DETR (medium) | **0.796** | **0.550** | 0.881 | 0.768 |

RF-DETR val also: F1 0.817, mAP75 0.586, mAR 0.660. Per-class AP: crack 0.730, manhole 0.686, pothole 0.525, garbage 0.259 — garbage is the hardest class.

Test split holds the same ranking: YOLOv8m mAP50 0.742, YOLOv26m 0.741, YOLOv11n 0.644.

Per-class mAP50-95 (YOLOv8m val): crack 0.675, manhole 0.579, pothole 0.466, garbage 0.235 — garbage is the hardest class.

Full curves, confusion matrices and per-split summaries: `yolov8m_res_100/`, `yolo26_results/`, `yolov11n_res/`, `yolov8m_runs/` (each has `train/`, `val/`, `test/` with `results.csv`, `Box*_curve.png`, `confusion_matrix*.png`, `metrics_summary.json`).

## Samples

`assets/samples/<model>/` — a handful of test-set predictions per model (full 2000-image dumps excluded from git).

## Weights (Hugging Face — live)

`*.pt / *.pth` are git-ignored. Download from **[devyansh99/sadakvision-weights](https://huggingface.co/devyansh99/sadakvision-weights)**; file links in `weights/README.md`.

## Reproduce

```bash
pip install -r requirements.txt

# YOLO — train
yolo detect train model=yolov8m.pt data=data.yaml epochs=100 imgsz=640 batch=32 device=0

# YOLO — val / test
yolo detect val model=weights/yolov8m_best.pt data=data.yaml split=test save_json=True

# YOLO — predict on samples
yolo detect predict model=weights/yolov8m_best.pt source=assets/samples/yolov8m/ save=True

# RF-DETR — `rfdetr_best.pth` from [HF weights repo](https://huggingface.co/devyansh99/sadakvision-weights)
```

Training configs preserved in each `*/train/args.yaml` (original absolute server paths sanitized — point `data:` at your local `data.yaml`).

## Repo structure

```text
├── README.md
├── requirements.txt
├── data.yaml
├── .gitignore
├── weights/README.md        # HF links (coming soon, no binaries in git)
├── assets/samples/          # curated test predictions per model
├── yolov8m_res_100/         # train/val/test summaries + curves (bulk preds excluded)
├── yolov8m_runs/
├── yolo26_results/
├── yolov11n_res/
├── rfdetr/                  # checkpoints excluded from git, via HF later
└── results_videos/          # excluded from git for now
```

Excluded from git: `*.pt/*.pth`, `*.mp4`, `*_test_predictions/` (8000+ images), `predictions.json`, `data/`. Anything reproducible from weights + dataset link stays out.
