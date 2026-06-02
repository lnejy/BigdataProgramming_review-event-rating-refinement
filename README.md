# 리뷰 이벤트 기반 별점 정제 프로젝트

2026 한성대학교 빅데이터프로그래밍 프로젝트입니다.  
배달앱 리뷰에서 **리뷰 이벤트성 리뷰를 판별**하고, 그 확률을 이용해 **가게 별점을 정제**하는 파이프라인을 구현합니다.

## 프로젝트 목표

- 리뷰 이벤트로 인해 높아질 수 있는 별점 왜곡을 줄입니다.
- `KcBERT + 메타데이터 + MLP` 구조로 이벤트 리뷰를 분류합니다.
- 분류 결과의 확률값을 사용해 가게별 정제 별점을 계산합니다.

## 주요 파일

- `workspace/src/01data_process.ipynb`
  - 원본 리뷰 엑셀 로드, 전처리, 라벨 생성
- `workspace/src/02KcBERT_extract.ipynb`
  - 정제 리뷰 텍스트에서 KcBERT 임베딩 추출
- `workspace/src/03PCA_Feature_Fusion.ipynb`
  - KcBERT 임베딩 PCA 축소, 메타데이터 결합 구조 확인
- `workspace/src/04SMOTE_5-fold_model_train.ipynb`
  - 제안 모델 학습, 5-fold CV, SMOTE, threshold 조정
- `workspace/src/05baseline_compare_metadata_effect.ipynb`
  - 베이스라인 비교, 메타데이터 조합 비교
- `workspace/src/06model_selection_error_analysis.ipynb`
  - 모델 선택, 오답 분석, 최종 bundle 저장
- `workspace/src/07rating_refinement_comparison.ipynb`
  - 후보 2 방식 별점 정제 비교
- `workspace/src/08end_to_end_demo.ipynb`
  - 단일 리뷰/가게 리뷰 예시 데모

## 데이터 위치

- 원본 리뷰 데이터: `workspace/src/reviews`
- 중간 CSV 산출물: `workspace/src/csv`
- 모델/평가 결과: `workspace/src/outputs`

## 실행 순서

아래 순서대로 실행하면 됩니다.

1. `01data_process.ipynb`
2. `02KcBERT_extract.ipynb`
3. `03PCA_Feature_Fusion.ipynb`
4. `04SMOTE_5-fold_model_train.ipynb`
5. `05baseline_compare_metadata_effect.ipynb`
6. `06model_selection_error_analysis.ipynb`
7. `07rating_refinement_comparison.ipynb`
8. `08end_to_end_demo.ipynb`

## 현재 반영된 핵심 개선

- 전체 리뷰 데이터 기준으로 파이프라인을 다시 정렬했습니다.
- `cleaned_text_length`, `is_very_short_review`, `has_emoji`, `has_photo` 메타데이터를 반영했습니다.
- `TF-IDF + Linear SVM` 베이스라인을 추가했습니다.
- 제안 모델과 베이스라인에 **확률 보정(`IsotonicRegression`)** 을 추가했습니다.
- 보정된 확률이 모델 선택, 별점 정제, 데모 추론까지 동일하게 적용되도록 연결했습니다.

## 참고

- 최종 제안 모델은 `KcBERT + 메타데이터 + MLP` 방향을 유지합니다.
- 별점 정제는 `후보 2: 1 - event_prob` 가중 평균 방식을 사용합니다.
