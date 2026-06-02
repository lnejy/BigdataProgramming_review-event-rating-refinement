# 리뷰 이벤트 기반 별점 정제 프로젝트

2026 한성대학교 빅데이터프로그래밍 프로젝트이다.  
배달앱 리뷰에서 리뷰 이벤트성 리뷰를 판별하고, 그 확률을 이용해 가게 별점을 정제하는 파이프라인을 구현한다.

## 프로젝트 목표

- 리뷰 이벤트로 인해 높아질 수 있는 별점 왜곡을 줄인다.
- `KcBERT + 메타데이터 + MLP` 구조로 이벤트 리뷰를 분류한다.
- 분류 결과의 확률값을 사용해 가게별 정제 별점을 계산한다.

## 주요 파일

- `workspace/src/01data_process.ipynb`
  - 원본 리뷰 엑셀을 불러오고 전처리와 라벨 생성을 수행한다.
- `workspace/src/02KcBERT_extract.ipynb`
  - 정제된 리뷰 텍스트에서 KcBERT 임베딩을 추출한다.
- `workspace/src/03PCA_Feature_Fusion.ipynb`
  - KcBERT 임베딩 PCA 축소와 메타데이터 결합 구조를 확인한다.
- `workspace/src/04SMOTE_5-fold_model_train.ipynb`
  - 제안 모델 학습, 5-fold CV, SMOTE, threshold 조정을 수행한다.
- `workspace/src/05baseline_compare_metadata_effect.ipynb`
  - 베이스라인 비교와 메타데이터 조합 비교를 수행한다.
- `workspace/src/06model_selection_error_analysis.ipynb`
  - 모델 선택, 오답 분석, 최종 bundle 저장을 수행한다.
- `workspace/src/07rating_refinement_comparison.ipynb`
  - 후보 2 방식 별점 정제 비교를 수행한다.
- `workspace/src/08end_to_end_demo.ipynb`
  - 단일 리뷰 및 가게 리뷰 예시 데모를 수행한다.

## 데이터 위치

- 원본 리뷰 데이터: `workspace/src/reviews`
- 중간 CSV 산출물: `workspace/src/csv`
- 모델 및 평가 결과: `workspace/src/outputs`

## 실행 순서

아래 순서대로 실행한다.

1. `01data_process.ipynb`
2. `02KcBERT_extract.ipynb`
3. `03PCA_Feature_Fusion.ipynb`
4. `04SMOTE_5-fold_model_train.ipynb`
5. `05baseline_compare_metadata_effect.ipynb`
6. `06model_selection_error_analysis.ipynb`
7. `07rating_refinement_comparison.ipynb`
8. `08end_to_end_demo.ipynb`

## 현재 반영된 핵심 개선

- 전체 리뷰 데이터 기준으로 파이프라인을 다시 정렬하였다.
- `cleaned_text_length`, `is_very_short_review`, `has_emoji`, `has_photo` 메타데이터를 반영하였다.
- `TF-IDF + Linear SVM` 베이스라인을 추가하였다.
- 제안 모델과 베이스라인에 확률 보정(`IsotonicRegression`)을 추가하였다.
- 보정된 확률이 모델 선택, 별점 정제, 데모 추론까지 동일하게 적용되도록 연결하였다.

## 참고

- 최종 제안 모델은 `KcBERT + 메타데이터 + MLP` 방향을 유지한다.
- 별점 정제는 `후보 2: 1 - event_prob` 가중 평균 방식을 사용한다.
