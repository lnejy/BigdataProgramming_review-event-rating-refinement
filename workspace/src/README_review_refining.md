# 한성대학교 빅데이터 프로그래밍 프로젝트

배달의민족 리뷰 데이터에서 리뷰 이벤트 참여 여부를 판별하고, 모델 예측 확률을 활용해 별점을 정제하는 것을 목표로 하는 프로젝트입니다.

## 로드맵

```text
1. 데이터 수집 및 라벨링
   - 배달의민족 리뷰 엑셀 데이터 통합
   - MENU 컬럼의 리뷰 이벤트 관련 표현을 기준으로 이벤트 리뷰 label 생성

2. 데이터 전처리
   - 리뷰 원문 정리
   - KcBERT 입력용 cleaned_review_text 생성
   - rating은 별점 정제 대상 값으로 보존
   - text_length, emoji_count, photo_count 메타데이터 생성
   - has_photo는 생성하지 않고, rating은 이벤트 판별 입력 feature에서 제외
   - DATE, OWNER_REPLY, created_at 등 미사용 원본 컬럼 제외

3. KcBERT 특징 추출
   - beomi/kcbert-base를 특징 추출기로 사용
   - 리뷰 텍스트별 768차원 CLS 임베딩 생성

4. PCA 차원 축소 및 Feature Fusion
   - train 데이터 기준으로 PCA fit
   - 누적 설명 분산 90% 수준으로 임베딩 차원 축소
   - 정규화한 메타데이터와 PCA 임베딩 결합
   - train / validation / test를 70% / 15% / 15%로 분할
   - 이 노트북은 구조 확인용이며 final_hybrid_*.csv를 저장하지 않음

5. 5-Fold 교차 검증 및 SMOTE
   - train 데이터 안에서 5-Fold 구성
   - 각 fold의 학습 데이터에만 SMOTE 적용
   - 검증 fold, validation, test에는 SMOTE 미적용
   - hidden layer, solver, learning rate, L2 alpha, activation, batch size grid 비교

6. 모델 학습 및 평가
   - 베이스라인: TF-IDF + Random Forest
   - 비교 실험: 메타데이터 only, KcBERT only, hybrid feature
   - 제안 모델: Scikit-learn MLP Classifier
   - 주요 지표: F1-Score, PR-AUC

7. 별점 정제
   - 제안 모델과 validation 기준 선택 모델의 이벤트 리뷰 확률을 비교
   - 중간 발표에서 채택한 후보 2: 이벤트 확률 기반 가중 평균만 적용
   - 후보 1 제외 방식은 정보 손실 위험이 있어 최종 코드에서는 사용하지 않음

8. End-to-End 데모
   - 저장된 프로젝트 제안 모델을 실제 리뷰 입력에 적용
   - 리뷰 입력에서 이벤트 확률, 이벤트 여부, 후보 2 별점 정제 가중치 출력
   - 학습이 아니라 최종 모델 적용 과정을 확인하는 발표용 데모
```

## 프로젝트 구조

```text
.
├── 01data_process.ipynb            # 원본 엑셀 통합, 라벨링, 텍스트/메타데이터 전처리
├── 02KcBERT_extract.ipynb          # KcBERT CLS 임베딩 추출
├── 03PCA_Feature_Fusion.ipynb      # PCA 차원 축소, 메타데이터 정규화, feature fusion 확인
├── 04SMOTE_5-fold_model_train.ipynb # 5-Fold CV, SMOTE, MLP grid 탐색
├── 05baseline_compare_metadata_effect.ipynb # baseline 및 ablation 비교
├── 06model_selection_error_analysis.ipynb   # validation 기준 모델 선택 및 오답 분석
├── 07rating_refinement_comparison.ipynb     # 후보 2 기반 별점 정제 비교
├── 08end_to_end_demo.ipynb       # 저장된 제안 모델 기반 입력-예측-정제 데모
├── custom/
│   └── review_v1.md                # 로컬 리뷰 문서
├── csv/
│   ├── preprocessed_reviews.csv
│   └── reviews_embeddings_extract.csv
├── outputs/
│   ├── proposed_mlp_*.csv/json/joblib
│   ├── baseline_*.joblib
│   ├── ablation_*.joblib
│   ├── final_selected_model.joblib
│   ├── final_proposed_model.joblib
│   └── rating_refinement_*.csv
├── reviews/                        # 원본 리뷰 엑셀 데이터
├── requirements.txt
├── 7_빅데이터프로그래밍_수행계획서_오남.pdf
└── 7분반_중간발표_오남.pdf
```

## 실행 순서

노트북은 아래 순서대로 실행합니다.

```text
01data_process.ipynb
-> 02KcBERT_extract.ipynb
-> 03PCA_Feature_Fusion.ipynb
-> 04SMOTE_5-fold_model_train.ipynb
-> 05baseline_compare_metadata_effect.ipynb
-> 06model_selection_error_analysis.ipynb
-> 07rating_refinement_comparison.ipynb
-> 08end_to_end_demo.ipynb
```

산출물 흐름은 다음과 같습니다.

```text
reviews/*.xlsx
-> csv/preprocessed_reviews.csv
-> csv/reviews_embeddings_extract.csv
-> outputs/proposed_mlp_final_model.joblib
-> outputs/baseline_1_tfidf_random_forest_model.joblib
-> outputs/baseline_2_metadata_only_mlp_model.joblib
-> outputs/baseline_3_kcbert_only_mlp_model.joblib
-> outputs/ablation_metadata_best_model.joblib
-> outputs/final_selected_model.joblib
-> outputs/final_proposed_model.joblib
-> outputs/rating_refinement_all_model_summary.csv
-> outputs/rating_refinement_error_samples.csv
-> outputs/rating_refinement_top_changed_stores.csv
```

`03PCA_Feature_Fusion.ipynb`는 PCA와 feature fusion 구조를 확인하지만 `final_hybrid_train.csv`, `final_hybrid_val.csv`, `final_hybrid_test.csv`를 저장하지 않습니다. 04번 이후 모델 학습과 07번 별점 정제는 `csv/reviews_embeddings_extract.csv`의 raw KcBERT feature에서 split을 재현하고, PCA/scaler/SMOTE를 pipeline 내부에서 fit합니다.

06번은 validation F1 기준 선택 모델을 `outputs/final_selected_model.joblib`로 저장하고, 프로젝트 제안 모델인 hybrid MLP를 `outputs/final_proposed_model.joblib`로 별도 저장합니다. 07번은 두 bundle만 로드해 후보 2 방식으로 별점을 정제합니다.

08번은 `outputs/final_proposed_model.joblib`만 사용해 실제 리뷰 입력부터 이벤트 확률과 후보 2 별점 정제 결과까지 확인하는 end-to-end 데모입니다. 모델을 새로 학습하지 않으며, 최종 발표에서 프로젝트 제안 구조의 적용 흐름을 보여주기 위한 선택 실행 단계입니다.

현재 생성된 전처리 데이터는 `8,841`개이며, 라벨 분포는 일반 리뷰 `5,691`개, 이벤트 리뷰 `3,150`개입니다.

`*.csv`, `*.joblib`, `custom/`은 `.gitignore`에 포함되어 있어 기본적으로 Git에 추적되지 않습니다. `outputs/*.json`은 재실행 시 긴 CV 탐색을 건너뛸 수 있도록 추적 대상입니다. 새 환경에서는 위 실행 순서대로 노트북을 실행해 CSV/joblib 산출물을 재생성해야 합니다.

## 환경 설정

Python 3.10 이상 사용을 권장합니다. `02KcBERT_extract.ipynb`과 `08end_to_end_demo.ipynb`는 Hugging Face에서 KcBERT 모델을 내려받기 때문에 최초 실행 시 인터넷 연결이 필요합니다. 필요한 패키지는 `requirements.txt`에 정리되어 있습니다.

### macOS: uv 사용

```bash
cd /Users/hyunseop/Developer/uni/big-data
uv python install 3.11
uv add -r project/requirements.txt
uv run jupyter lab project
```

### Windows: pip 사용

Git Bash 또는 Bash 터미널 기준입니다.

```bash
cd path/to/big-data/project
python -m pip install -r requirements.txt
jupyter lab
```

노트북 셀에서 아래 명령으로 현재 커널에 패키지를 설치할 수 있습니다.

```python
%pip install -r requirements.txt
```

설치 후에는 커널을 재시작해야 새 패키지가 반영됩니다.
