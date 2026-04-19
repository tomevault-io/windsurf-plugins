---
trigger: always_on
description: 수박 소리 데이터를 활용하여 **그래디언트 부스팅 트리(GBT)**, **서포트 벡터 머신(SVM)**, **랜덤 포레스트** 모델로 당도를 예측하는 머신러닝 프로젝트의 상세 작업 리스트
---

# 🍉 전통적인 ML 모델 기반 수박 당도 예측 프로젝트 - Todo List

## 📋 프로젝트 개요

수박 소리 데이터를 활용하여 **그래디언트 부스팅 트리(GBT)**, **서포트 벡터 머신(SVM)**, **랜덤 포레스트** 모델로 당도를 예측하는 머신러닝 프로젝트의 상세 작업 리스트

---

## ✅ Phase 1: 환경 설정 및 데이터 준비

### 🔧 1.1 프로젝트 환경 구성

- [x] **1.1.1** Python 3.8+ 설치 확인 ✅ _완료: Python 3.13.5 확인_
- [x] **1.1.2** 가상환경 생성 및 활성화 ✅ _완료: watermelon_ml_env 생성 및 활성화_
  ```bash
  python -m venv watermelon_ml_env
  source watermelon_ml_env/bin/activate  # Windows: watermelon_ml_env\Scripts\activate
  ```
- [x] **1.1.3** 필수 라이브러리 설치 ✅ _완료: 모든 핵심 라이브러리 설치 성공_
  ```bash
  pip install scikit-learn>=1.3.0 pandas>=1.5.0 numpy>=1.21.0
  pip install librosa>=0.9.0 soundfile>=0.12.0
  pip install matplotlib>=3.5.0 seaborn>=0.11.0 plotly>=5.0.0
  pip install joblib>=1.2.0 PyYAML>=6.0 tqdm>=4.64.0 scipy>=1.9.0
  pip install skl2onnx>=1.15.0 onnx>=1.12.0 coremltools>=7.0
  ```
  **설치된 주요 버전:**
  - scikit-learn: 1.7.0
  - pandas: 2.3.1
  - numpy: 2.2.6
  - librosa: 0.11.0
  - matplotlib: 3.10.3
  - seaborn: 0.13.2
  - plotly: 6.2.0
  - PyYAML: 6.0.2
  - skl2onnx: 1.19.1
  - onnx: 1.18.0
  - coremltools: 8.3.0
- [x] **1.1.4** `requirements.txt` 파일 생성 ✅ _완료: 139개 패키지 의존성 저장_
- [x] **1.1.5** Jupyter Notebook 환경 설정 (선택사항) ✅ _완료: jupyter 1.1.1 설치_

### 📁 1.2 프로젝트 디렉토리 구조 생성

- [x] **1.2.1** 기본 디렉토리 구조 생성 ✅ _완료: 표준 ML 프로젝트 구조_
  ```
  mkdir -p src/{data,models,training,evaluation,conversion,utils}
  mkdir -p {configs,data/{raw,processed,splits},models/{saved,mobile},experiments,notebooks,tests,scripts}
  ```
- [x] **1.2.2** `__init__.py` 파일 생성 (모든 패키지 디렉토리) ✅ _완료: 모든 패키지 디렉토리 초기화_
- [x] **1.2.3** `.gitignore` 파일 생성 (Python, 데이터 파일, 모델 파일 제외) ✅ _완료: Git 버전 관리 설정_

### 🍉 1.3 데이터 수집 및 정리

- [x] **1.3.1** 수박 오디오 데이터 수집 ✅ _완료: 50개 수박, 146개 오디오 파일 생성_
  - 익은 수박 소리 (.wav, .m4a, .mp3 형식)
  - 안 익은 수박 소리
  - 다양한 당도의 수박 소리 (8.1~12.9 Brix)
- [x] **1.3.2** 데이터를 `data/raw/` 폴더에 정리 ✅ _완료: 001_10.1 ~ 050_11.5 형식_
  - 폴더명: `{번호}_{당도값}` (예: `1_10.5`, `2_9.7`)
  - 각 폴더에 해당 수박의 여러 오디오 파일 저장
- [x] **1.3.3** 데이터 메타정보 CSV 파일 생성 ✅ _완료: watermelon_metadata.csv (146행)_
  - 컬럼: `file_path`, `watermelon_id`, `sweetness`, `recording_session`
- [x] **1.3.4** 데이터 품질 검사 ✅ _완료: 모든 파일 정상, 평균 길이 2.2초_
  - 파일 손상 여부 확인
  - 오디오 길이 확인 (최소 0.5초 이상)
  - 샘플링 레이트 확인

### 🔍 1.4 탐색적 데이터 분석 (EDA)

- [x] **1.4.1** Jupyter Notebook 생성: `notebooks/01_EDA.ipynb` ✅ _완료: 포괄적 EDA 노트북_
- [x] **1.4.2** 데이터셋 통계 분석 ✅ _완료: 당도 범위 8.1~12.9 Brix, 평균 10.57±1.20_
  - 총 파일 수, 당도 분포, 파일 형식 분석
  - 당도 구간별 샘플 수 확인
- [x] **1.4.3** 오디오 파일 기본 정보 분석 ✅ _완료: 평균 길이 2.2초, 16000Hz 샘플링_
  - 길이, 샘플링 레이트, 채널 수
  - 평균 파일 크기, 최대/최소 길이
- [x] **1.4.4** 샘플 오디오 시각화 ✅ _완료: 파형 및 스펙트로그램 분석_
  - 파형(waveform) 플롯
  - 스펙트로그램 시각화
  - 당도별 차이점 관찰

---

## ✅ Phase 2: 전처리 및 특징 추출

### 🎵 2.1 오디오 전처리 모듈 개발

- [x] **2.1.1** `src/data/audio_loader.py` 생성 ✅ _완료: 다중 형식 지원 AudioLoader 구현_
  - `AudioLoader` 클래스 구현
  - 다양한 형식 (.wav, .m4a, .mp3, .flac, .aiff, .ogg) 지원
  - 오류 처리 및 로깅 기능
- [x] **2.1.2** `src/data/preprocessor.py` 생성 ✅ _완료: 포괄적 전처리 파이프라인 구현_
  - `AudioPreprocessor` 클래스 구현
  - 세그멘테이션 (묵음 구간 제거)
  - 정규화 (-1~1 범위)
  - 노이즈 필터링 (선택사항)
- [x] **2.1.3** 전처리 설정 파일 생성: `configs/preprocessing.yaml` ✅ _완료: 127줄 포괄적 설정_
  ```yaml
  audio:
    sample_rate: 16000
    trim_top_db: 20
    normalize: true
    filter_noise: false
  ```

### 🔧 2.2 특징 추출 모듈 개발

- [x] **2.2.1** `src/data/feature_extractor.py` 생성 ✅ _완료: 299줄 포괄적 특징 추출기_
- [x] **2.2.2** `AudioFeatureExtractor` 클래스 구현 ✅ _완료: 정확히 51개 특징 추출_
  - **MFCC 특성** (13개): `extract_mfcc_features()`
  - **스펙트럴 특성** (7개): `extract_spectral_features()`
  - **에너지 특성** (4개): `extract_energy_features()`
  - **리듬 특성** (3개): `extract_rhythm_features()`
  - **수박 전용 특성** (8개): `extract_watermelon_specific_features()`
  - **통계적 특성** (16개): `extract_statistical_features()`
- [x] **2.2.3** 특징 추출 통합 함수 구현 ✅ _완료: 51차원 벡터 및 특징명 반환_
  - `extract_all_features()`: 모든 특징을 1D 벡터로 반환
  - `get_feature_names()`: 특징 이름 리스트 반환
- [x] **2.2.4** 특징 추출 테스트 ✅ _완료: 0 NaN/Inf 값, 100% 성공률_
  - 샘플 오디오로 특징 추출 테스트
  - 특징 벡터 크기 확인 (51개)
  - NaN/Inf 값 처리

### 📊 2.3 데이터셋 구축

- [x] **2.3.1** `src/data/dataset_builder.py` 생성 ✅ _완료: 배치 처리 및 메모리 관리 포함_
- [x] **2.3.2** 전체 데이터셋 특징 추출 실행 ✅ _완료: 146개 파일, 평균 0.1초/파일_
  ```python
  # 모든 오디오 파일에서 특징 추출
  # DataFrame 형태로 저장: [features..., sweetness]
  ```
- [x] **2.3.3** 특징 데이터셋 저장 ✅ _완료: features.csv (0.13MB), feature_names.txt_
  - `data/processed/features.csv`
  - `data/processed/feature_names.txt`
- [x] **2.3.4** 데이터 품질 검증 ✅ _완료: 0 결측값, 0 무한값, 완벽한 데이터 품질_
  - 누락값(NaN) 확인 및 처리
  - 이상치(outlier) 탐지 및 분석
  - 특징간 상관관계 분석

### ✂️ 2.4 데이터 분할

- [x] **2.4.1** `src/data/data_splitter.py` 생성 ✅ _완료: 층화 샘플링 및 검증 포함_
- [x] **2.4.2** 데이터 분할 구현 ✅ _완료: Train(69.9%) / Val(15.1%) / Test(15.1%)_
  - Train(70%) / Validation(15%) / Test(15%)
  - 층화 샘플링 (당도 구간별 균등 분할)
  - random_state=42 (재현성)
- [x] **2.4.3** 분할된 데이터 저장 ✅ _완료: train.csv(91.7KB), val.csv(20.3KB), test.csv(20.3KB)_
  - `data/splits/train.csv`
  - `data/splits/val.csv`
  - `data/splits/test.csv`
- [x] **2.4.4** 분할 정보 검증 ✅ _완료: EXCELLENT 품질, 완벽한 층화 샘플링_
  - 각 세트별 당도 분포 확인
  - 샘플 수 균형 검증

---

## ✅ Phase 3: 모델 학습 및 평가 - **100% 완료** (2025-01-15) 🎉**대성공!**

### 🤖 3.1 모델 클래스 개발

- [x] **3.1.1** `src/models/traditional_ml.py` 생성 ✅ _완료: 500+ 줄 객체지향 프레임워크_
- [x] **3.1.2** 모델 클래스 구현 ✅ _완료: 특화된 3개 모델 클래스_

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sing-u8) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
