# Oral Disease Image Classification Project

사전학습 CNN을 fine-tuning하여 **구강질환 5개 클래스**를 분류하는 프로젝트 정리본입니다.

## 프로젝트 목표
데이터셋이 미리 정리되어 있지 않은 특수 도메인 환경에서 구강질환 이미지를 직접 수집하고,
데이터셋을 구축한 뒤, pre-trained CNN을 fine-tuning하여 실제로 동작하는 분류 모델을 만드는 것이 목표입니다.

대상 클래스는 다음 5개입니다.

- calculus (치석)
- caries (치아우식증/충치)
- discoloration (치아 변색)
- hypodontia (치아 결손)
- ulcers (입안 궤양/구내염)

## 데이터셋 개요
수집된 전체 이미지는 총 **3269장**이며, train / val / test로 분할되어 있습니다.

- Train: 2410
- Val: 547
- Test: 312

클래스별 분포는 불균형하며, 특히 discoloration이 소수 클래스입니다.  
이 때문에 평가 지표는 accuracy 단독이 아니라 **Macro-F1 / Balanced Accuracy / per-class recall / confusion matrix** 중심으로 해석합니다.

## 핵심 파이프라인
- normalization: pre-trained 모델 입력 분포 정렬
- augmentation: crop / flip / rotation / color jitter
- imbalance 대응: `WeightedRandomSampler`
- overfitting 대응: watch-on + metric 기반 early stopping
- fine-tuning 전략:
  - VGG19_BN: backbone 고정 + BN affine + 마지막 FC
  - ResNet50: layer4 + fc + BN affine
  - DenseNet121: denseblock4(+norm5) + classifier + BN affine
- optimizer: SGD

## 핵심 결과 정리
### 1) 원본 데이터셋에서의 문제
원본 데이터셋 기준 평가에서는 특히 **discoloration 클래스의 recall 붕괴**가 크게 나타났습니다.  
즉, 모델 구조를 바꾸더라도 특정 클래스가 test split에서 거의 학습되지 않는 현상이 확인되었습니다.

### 2) 외부 직접 선별 평가셋 검증
직접 선별한 외부 평가셋으로 다시 확인했을 때, 동일 모델에서도 discoloration recall이 크게 개선되었습니다.  
이 결과는 **모델 자체보다 데이터 분포 문제**가 더 근본 원인일 가능성을 지지합니다.

### 3) 재분할(resplit) 이후 성능 개선
no-reference 품질 proxy(선명도, 밝기, 대비, 채도, 노이즈, 파일 크기 등)로 split 간 분포 차이를 정량화한 뒤,
sharpness 기준으로 재분할한 데이터셋에서 성능이 크게 개선되었습니다.

대표적으로 ResNet50(resplit)은 다음 수준까지 향상되었습니다.

- Test Accuracy: **91.35%**
- Test Macro-F1: **0.8536**
- Test Balanced Accuracy: **85.62%**

### 4) segmentation ROI 확장
segmentation ROI crop 실험도 수행했지만, 현재 저장된 결과 기준으로는
**sharpness 기반 재분할 baseline을 유의미하게 넘지 못했습니다.**
특히 치아 자체보다 주변 문맥이 더 중요한 클래스(예: ulcers, hypodontia)에서는 불리한 사례가 있었습니다.

## 제공 파일
- `oral_disease_project_github_ready.ipynb`
  - GitHub 업로드용 메인 노트북
  - outputs 제거
  - 프로젝트 흐름 기준으로 재배치
- `oral_disease_project_reordered_exact.ipynb`
  - 원본 visible 셀 번호를 유지한 exact reorder 버전
  - 모든 원본 셀을 project-flow 기준 순서로 재배치
- `source_cell_mapping.md`
  - 새 노트북 섹션과 원본 visible 셀 번호 매핑
- `requirements.txt`
  - 재현을 위한 최소 패키지 목록

## 실행 환경 메모
현재 코드에는 Google Colab / Google Drive 경로가 직접 들어가 있으므로,
재실행 전에는 다음을 먼저 점검해야 합니다.

1. 데이터셋 경로
2. 결과 저장 경로
3. 외부 평가셋 path / target class 설정
4. segmentation ROI 생성 경로

