# 🧘 자세히봐

> **OpenCV 기반 측면 이미지를 활용한 VDT 취급 근로자의 자세·작업환경 통합 교정 솔루션**
사용자의 앉은 자세와 주변 작업 환경(책상, 의자, 모니터)을 통합적으로 분석하여 실시간 교정 피드백을 제공하는 컴퓨터 비전 프로젝트입니다.
---

## 📌 프로젝트 개요

VDT(Visual Display Terminal) 증후군으로 진료받은 환자 수는 2024년 기준 약 **705만 명**으로, 5년간 12.2% 증가했습니다. 전체 산업재해 중 근골격계 질환(신체부담 작업)은 **41.9%** 를 차지하며, 특히 50인 미만 소규모 사업장은 보건관리자 선임 의무에서 제외되어 안전보건 사각지대에 놓여 있습니다.

**자세히봐**는 물리치료사의 임상 노하우를 디지털 기술로 전환한 서비스입니다. 측면 사진 한 장으로 신체 자세와 작업 환경을 함께 분석하여, 전문가 없이도 스스로 지속 가능한 건강 관리를 실천할 수 있도록 돕습니다.

---

## ✨ 주요 기능

### 📸 AI 자세 분석 파이프라인
측면 사진을 업로드하면 3단계 AI 파이프라인이 순차적으로 실행됩니다.

| 단계 | 기술 | 역할 |
|------|------|------|
| Step 1 | MediaPipe Pose | 신체 33개 관절 좌표 추출 |
| Step 2 | MLP 분류기 | CVA·TIA 기반 자세 양호/불량 판정 |
| Step 3 | YOLOv8 | 모니터·책상·의자 객체 탐지 |
| Step 4 | NumPy 벡터 연산 | 나머지 6개 지표 각도 산출 |
| Step 5 | OpenCV | 결과 오버레이 시각화 생성 |

### 📐 8가지 인간공학 측정 지표

RULA(Rapid Upper Limb Assessment) 및 고용노동부 VDT 작업관리지침 기준으로 평가합니다.

| # | 지표 | 정상 범위 | 분류 |
|---|------|-----------|------|
| 1 | CVA 목굴곡각 | 0° ~ 20° | 자세 |
| 2 | TIA 몸통굴곡각 | 0° ~ 10° | 자세 |
| 3 | 팔꿈치 각도 | 90° ~ 120° | 자세 |
| 4 | 무릎 각도 | 85° ~ 100° | 자세 |
| 5 | 손목 편차 | ±15° 이내 | 자세 |
| 6 | 모니터 시선각 | 하방 10° ~ 15° | 환경 |
| 7 | 작업대 높이 | 팔꿈치 기준 ±10% | 환경 |
| 8 | 의자-등받이 거리 | 골반너비 20% 이내 | 환경 |

### 🖥️ 서비스 화면 구성

- **자세측정** — 측면 사진 업로드 → AI 분석 → 오버레이 이미지 + 8개 지표 결과 카드
- **측정이력** — 계정별 점수 추이 차트 및 전체 측정 기록
- **근골격계 리포트** — 상세 분석 리포트 + PDF 저장
- **예상 영수증** — 비급여 의료비 예상 비용 자동 산출 (도수치료·체외충격파·증식치료)
- **바른자세 챌린지** — 알림 설정 + 팀원 간 포인트 랭킹 레이스
- **제품 추천** — BAD 항목별 맞춤 인간공학 제품 큐레이션

---

## 🏗️ 시스템 아키텍처

```
측면 사진 입력
     │
     ▼
┌─────────────────────────────────────────┐
│           integrate.py (분석 파이프라인) │
│                                         │
│  Step 1: MediaPipe → 관절 좌표 추출      │
│  Step 2: MLP → CVA·TIA 자세 판정        │
│     ├─ BAD → 교정 오버레이만 출력        │
│     └─ GOOD → Step 3, 4 진행            │
│  Step 3: YOLOv8 → 환경 객체 탐지         │
│  Step 4: NumPy → 나머지 6개 지표 계산    │
│  Step 5: OpenCV → 결과 오버레이 합성     │
└──────────────────┬──────────────────────┘
                   │
                   ▼
        app.py (Streamlit UI)
   ┌───────────────────────────┐
   │  자세측정 / 측정이력       │
   │  근골격계 리포트 / 영수증  │
   │  챌린지 / 제품 추천        │
   └───────────────────────────┘
```

---

## 🛠️ 기술 스택

| 구분 | 기술 |
|------|------|
| Language | Python 3.11 |
| AI / CV | MediaPipe, YOLOv8 (Ultralytics), TensorFlow/Keras, OpenCV |
| Web UI | Streamlit |
| 데이터 처리 | NumPy, Pandas, scikit-learn (StandardScaler) |
| 시각화 | Altair, ReportLab |
| MLOps | Roboflow |

---

## 📂 프로젝트 구조

```text
Fit_me_up/
├── MediaPipe/                # 자세 분석 모듈 (Postural Analysis)
│   ├── code/                 # 추론 API, 테스트 UI, 학습 스크립트
│   ├── models/               # CNN(MobileNetV2), MLP 모델 파일
│   └── data/                 # 학습용 관절 좌표 데이터셋
├── YOLO/                     # 환경 탐지 모듈 (Environment Detection)
│   ├── fit_me_up/            # 학습 결과 저장소
│   │   └── combined_gpu/
│   │       └── weights/
│   │           └── best.pt   # ← [공유 모델] 클론 후 즉시 테스트 가능
│   ├── run_all.py            # 자동 학습 및 전략 비교 스크립트
│   └── yolo_test.py          # YOLO 추론 테스트 UI (best.pt 사용)
├── integrate/                # 통합 파이프라인
└── requirements.txt          # 프로젝트 공통 의존성 관리

---

## ⚙️ 설치 및 실행

### 1. 저장소 클론

```bash
git clone https://github.com/your-org/fit-me-up.git
cd fit-me-up
```

### 2. 의존성 설치

```bash
pip install -r requirements.txt
```

주요 패키지:
```
streamlit
opencv-python
mediapipe
ultralytics
tensorflow
numpy
pandas
altair
scikit-learn
reportlab
Pillow
joblib
```

### 3. 모델 파일 준비

아래 경로에 학습된 모델 파일을 배치합니다.

```
MediaPipe/models/pose_landmarker_full.task  # MediaPipe Tasks API 모델
MediaPipe/models/posture_mlp_final.keras    # MLP 분류기
MediaPipe/models/posture_scaler.pkl         # StandardScaler
YOLO/fit_me_up/combined_gpu/weights/best.pt # YOLOv8 가중치
```

### 4. Streamlit 앱 실행

```bash
streamlit run app.py
```

### 5. (선택) 데스크톱 UI 실행

Tkinter 기반 독립 실행형 분석 툴:

```bash
python integrate.py
```

---

## 📊 데이터셋

| 구분 | 수량 | 비고 |
|------|------|------|
| Roboflow 원본 | 1,503장 | good_pose / bad_pose 라벨 |
| MediaPipe 필터링 후 | 400장 | full_body + ankle_visible |
| 검수 완료 최종 | 347장 | RULA 기준 라벨 확정 |

**데이터 분할**

| 세트 | 비율 | 수량 |
|------|------|------|
| Train | 80% | 약 279장 |
| Validation | 10% | 약 34장 (StratifiedKFold K=5) |
| Test | 10% | 약 34장 |

**데이터 증강:** 좌우반전(Horizontal Flip), 밝기 조절(0.8~1.2)

---

## 🔍 판정 로직

### CVA/TIA 게이트

1. MLP가 CVA·TIA를 기반으로 자세를 분류합니다.
2. **둘 중 하나라도 BAD**이면 → 교정 화살표 오버레이만 출력하고 이후 지표 분석을 중단합니다.
3. **둘 다 GOOD**이면 → YOLO 환경 탐지 및 나머지 6개 지표 분석을 진행합니다.

### 종합 점수 산출

| 판정 | 점수 |
|------|------|
| 정상 | 10점 |
| 주의 (CVA·TIA만 해당) | 6점 |
| 위험 | 2점 |

`종합 점수 = 측정된 지표 점수의 평균`

---

## ⚠️ 주의사항

- 본 서비스는 자세 위험도 참고용이며, **의료적 진단을 대체하지 않습니다.**
- **측면 전신 사진** (의자·책상·모니터 포함)을 사용할 때 정확도가 가장 높습니다.
- 발목·무릎·골반·어깨·귀가 모두 보이는 사진을 권장합니다.
- YOLO 객체 탐지는 모니터·책상·의자 3가지 클래스만 지원합니다.