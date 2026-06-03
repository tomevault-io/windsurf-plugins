---
trigger: always_on
description: - `src/lib.rs` is the library entry point with CLI command dispatch.
---

# Repository Guidelines

## Project Structure & Module Organization
- `src/lib.rs` is the library entry point with CLI command dispatch.
- `src/main.rs` is a thin CLI wrapper that calls into the library.
- `src/ir/` contains the Intermediate Representation module (model, bbox, converters), including `src/ir/io_yolo.rs` for YOLO TXT directory format (flat Darknet-style and split-aware layouts, with optional confidence token), `src/ir/io_yolo_keras_txt.rs` for shared YOLO Keras / YOLOv4 PyTorch absolute-coordinate TXT, `src/ir/io_voc_xml.rs` for Pascal VOC XML, `src/ir/io_label_studio_json.rs` for Label Studio JSON, `src/ir/io_labelbox_json.rs` for Labelbox JSON/NDJSON, `src/ir/io_scale_ai_json.rs` for Scale AI JSON, `src/ir/io_unity_perception_json.rs` for Unity Perception/SOLO JSON, `src/ir/io_labelme_json.rs` for LabelMe JSON (per-image, file + directory), `src/ir/io_createml_json.rs` for Apple CreateML JSON, `src/ir/io_cloud_annotations_json.rs` for IBM Cloud Annotations JSON, `src/ir/io_vott_csv.rs` for VoTT CSV, `src/ir/io_vott_json.rs` for VoTT JSON, `src/ir/io_kitti.rs` for KITTI object detection labels, `src/ir/io_via_json.rs` for VGG Image Annotator (VIA) JSON, `src/ir/io_via_csv.rs` for VGG Image Annotator (VIA) CSV, `src/ir/io_retinanet_csv.rs` for RetinaNet Keras CSV, `src/ir/io_tfrecord.rs` for TFRecord (single-file uncompressed TFOD-style Example records), `src/ir/io_sagemaker_manifest.rs` for AWS SageMaker Ground Truth manifests, `src/ir/io_superannotate_json.rs` for SuperAnnotate JSON, `src/ir/io_supervisely_json.rs` for Supervisely JSON, `src/ir/io_cityscapes_json.rs` for Cityscapes polygon JSON, `src/ir/io_marmot_xml.rs` for Marmot XML, `src/ir/io_datumaro_json.rs` for Datumaro JSON, `src/ir/io_wider_face_txt.rs` for WIDER Face TXT, `src/ir/io_oidv4_txt.rs` for OIDv4 TXT (`Label/` layout), `src/ir/io_bdd100k_json.rs` for BDD100K/Scalabel JSON, `src/ir/io_v7_darwin_json.rs` for V7 Darwin JSON, `src/ir/io_edge_impulse_labels.rs` for Edge Impulse `bounding_boxes.labels`, `src/ir/io_openlabel_json.rs` for OpenLABEL JSON bbox subset, and `src/ir/io_super_json_common.rs` shared helpers for the SuperAnnotate/Supervisely adapters.
- `src/conversion/` contains conversion lossiness analysis and stable report issue codes.
- `src/stats/` contains dataset statistics logic and HTML/text/JSON reporting.
- `src/validation/` contains dataset validation logic.
- `tests/cli.rs` contains CLI integration tests using `assert_cmd`.
- `tests/tfod_csv_roundtrip.rs`, `tests/tfrecord_roundtrip.rs`, `tests/yolo_roundtrip.rs`, `tests/yolo_keras_roundtrip.rs`, `tests/voc_roundtrip.rs`, `tests/label_studio_roundtrip.rs`, `tests/labelbox_roundtrip.rs`, `tests/scale_ai_roundtrip.rs`, `tests/unity_perception_roundtrip.rs`, `tests/labelme_roundtrip.rs`, `tests/createml_roundtrip.rs`, `tests/cloud_annotations_roundtrip.rs`, `tests/vott_csv_roundtrip.rs`, `tests/vott_json_roundtrip.rs`, `tests/kitti_roundtrip.rs`, `tests/via_roundtrip.rs`, `tests/via_csv_roundtrip.rs`, `tests/retinanet_csv_roundtrip.rs`, `tests/sagemaker_manifest_roundtrip.rs`, `tests/superannotate_roundtrip.rs`, `tests/supervisely_roundtrip.rs`, `tests/cityscapes_roundtrip.rs`, `tests/marmot_roundtrip.rs`, `tests/datumaro_roundtrip.rs`, `tests/wider_face_roundtrip.rs`, `tests/oidv4_roundtrip.rs`, `tests/bdd100k_roundtrip.rs`, `tests/v7_darwin_roundtrip.rs`, `tests/edge_impulse_roundtrip.rs`, and `tests/openlabel_roundtrip.rs` cover format-specific integration behavior.
- `tests/proptest_*.rs` add property-based roundtrip/idempotency/subset checks; shared helpers live in `tests/proptest_helpers/mod.rs` and `tests/common/mod.rs`.
- `docs/` is the durable documentation home for users and contributors.
- `benches/` contains Criterion benchmarks.
- `fuzz/` contains cargo-fuzz targets for parser fuzzing.
- `scripts/dataset_generator.py` generates synthetic datasets for testing.
- `assets/` is for generated test data and is gitignored.

## Build, Test, and Development Commands
```bash
cargo build              # Build debug version
cargo build --release    # Build optimized release version
cargo run                # Run the CLI
cargo run -- -V          # Run with CLI args
cargo check              # Fast type checking
cargo fmt                # Format code
cargo clippy             # Lint
cargo test               # All tests (unit + integration + proptests)
cargo test --test cli    # CLI integration tests only
cargo test --test proptest_ir_json
cargo test --test proptest_coco
cargo test --test proptest_tfod
cargo test --test proptest_label_studio
cargo test --test proptest_voc
cargo test --test proptest_yolo
cargo test --test proptest_labelme
cargo test --test proptest_createml
cargo test --test proptest_cross_format
cargo test runs          # Run a single test by name

PROPTEST_CASES=1000 cargo test --test proptest_ir_json  # deeper local run
cargo doc --open         # Build and view docs
cargo bench              # Run Criterion benchmarks
cargo bench -- --test    # Smoke test benchmarks
cargo +nightly fuzz list                          # List all fuzz targets (requires cargo-fuzz + nightly)
cargo +nightly fuzz build coco_json_parse         # Build a target without running
cargo +nightly fuzz run coco_json_parse -- -runs=256  # Short smoke run

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [strickvl/panlabel](https://github.com/strickvl/panlabel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
