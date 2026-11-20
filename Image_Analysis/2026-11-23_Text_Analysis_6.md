# TIL | 2026-11-23 — 객체 탐지·세그멘테이션 평가

오늘은 이미지 분류의 클래스 평가에서 확장하여, 객체 탐지의 위치 정확성과 검출 성능, 세그멘테이션의 픽셀 영역 일치도를 평가하는 방법을 배웠다.

## 1. 태스크마다 평가 대상이 달라진다

| 태스크 | 확인할 내용 | 대표 지표 |
| --- | --- | --- |
| 이미지 분류 | 이미지별 클래스 예측과 정답의 일치 | Accuracy, Precision, Recall, F1-score |
| 객체 탐지 | 객체 클래스, 위치, 누락과 오검출 | Box IoU, Precision, Recall, AP·mAP |
| 세그멘테이션 | 정답과 예측 픽셀 영역의 위치·형태 | Mask IoU, Dice Score |
| Instance Segmentation | 개별 객체 검출·구분과 객체별 영역 | Mask IoU, Mask AP·mAP |

예측 클래스가 맞아도 Box가 실제 결함 위치에서 벗어날 수 있다. 또한 정확한 위치의 객체 몇 개만 찾았다면 나머지 객체의 누락까지 평가해야 한다.

## 2. IoU로 위치의 겹침 평가

IoU(Intersection over Union)는 예측 영역과 정답 영역의 교집합을 합집합으로 나눈 값이다.

```text
Union = 정답 면적 + 예측 면적 - Intersection
IoU = Intersection / Union
```

IoU는 0~1이며, 0은 겹침 없음, 1은 영역의 완전한 일치를 나타낸다. 클래스의 정확성은 별도로 확인한다.

학습자료의 XYXY Box 예시에서 정답은 `(50, 50, 150, 150)`, 예측은 `(70, 50, 170, 150)`이다.

| 항목 | 계산 결과 |
| --- | --- |
| 정답 면적 | 100 × 100 = 10,000 |
| 예측 면적 | 100 × 100 = 10,000 |
| 교집합 | 80 × 100 = 8,000 |
| 합집합 | 10,000 + 10,000 - 8,000 = 12,000 |
| IoU | 8,000 / 12,000 ≈ 0.667 |

```python
true_box = (50, 50, 150, 150)
pred_box = (70, 50, 170, 150)

left = max(true_box[0], pred_box[0])
top = max(true_box[1], pred_box[1])
right = min(true_box[2], pred_box[2])
bottom = min(true_box[3], pred_box[3])

intersection = max(0, right - left) * max(0, bottom - top)
true_area = (true_box[2] - true_box[0]) * (true_box[3] - true_box[1])
pred_area = (pred_box[2] - pred_box[0]) * (pred_box[3] - pred_box[1])
union = true_area + pred_area - intersection

print(f"IoU = {intersection / union:.3f}")  # 0.667
```

위 코드는 유효한 면적을 가진 예시 Box를 사용한다. 실제 평가에서는 잘못된 좌표와 면적 0인 Box의 처리 기준도 정한다.

## 3. TP·FP·FN과 Precision·Recall

일반적인 탐지 평가에서는 클래스가 일치하고 IoU 기준을 충족하며, 아직 대응되지 않은 정답 객체와 연결된 예측을 TP로 인정한다. 하나의 정답에 여러 예측이 대응하려 하면 중복 검출을 평가 규칙에 따라 처리한다.

| 항목 | 의미 |
| --- | --- |
| TP | 클래스·위치 조건을 충족해 정답 객체와 대응된 검출 |
| FP | 클래스 오류, 위치 기준 미달, 중복 등으로 올바르게 대응되지 못한 검출 |
| FN | 올바른 검출과 대응되지 못한 정답 객체 |

객체 탐지는 배경의 모든 가능한 영역을 TN으로 세기 어려워 TP·FP·FN을 중심으로 평가한다.

정답 객체 4개에 예측 5개가 있고 TP 2개, FP 3개라면 FN은 2개다.

```text
Precision = TP / (TP + FP) = 2 / 5 = 0.40
Recall = TP / (TP + FN) = 2 / 4 = 0.50
```

Precision은 검출 결과 중 올바른 비율, Recall은 실제 객체 중 찾아낸 비율이다. 클래스가 틀린 검출은 FP가 되고, 해당 정답이 다른 올바른 예측과도 연결되지 않으면 FN이 남는다.

### 두 Threshold의 역할

| 기준 | 용도 |
| --- | --- |
| Confidence Threshold | 어떤 예측을 유지할지 결정 |
| IoU Threshold | 정답 위치와 충분히 일치하는지 판정 |

Confidence 기준을 높이면 낮은 점수의 후보를 제외하지만 실제 객체도 놓칠 수 있다. Precision의 변화 방향은 실제 검출 순위에 따라 달라질 수 있다. 결함 누락의 영향과 오검출 비용을 고려하여 검증 데이터에서 기준을 선택한다.

## 4. AP와 mAP

AP(Average Precision)는 클래스별로 IoU 기준을 정하고 예측을 Confidence 순으로 평가하여 얻은 Precision-Recall 관계를 종합한다. 구체적인 곡선 보간과 평균 방식은 평가 규약을 따른다.

mAP는 클래스별 AP의 평균이다. 균열 AP가 0.80, 흠집 AP가 0.60이면 mAP는 0.70이다.

| 표기 | 의미 |
| --- | --- |
| mAP@0.5 | IoU 0.5 기준의 클래스별 AP 평균 |
| mAP@0.5:0.95 | IoU 0.50~0.95를 0.05 간격으로 적용해 얻은 결과의 평균 |

단일 Confidence 기준의 TP·FP·FN 합계만으로는 AP를 계산하기 어렵다. 클래스별 예측 점수와 정답 대응을 바탕으로 순위별 Precision-Recall을 구성해야 한다.

mAP@0.5가 0.82이고 mAP@0.5:0.95가 0.54라는 가상 예시는 더 엄격한 위치 기준을 포함할 때 점수가 낮아지는 상황을 보여 준다. 모델을 비교할 때는 평가 데이터와 IoU 기준, 평가 규약을 맞춘다.

## 5. Mask IoU와 Dice Score

세그멘테이션은 Box 면적 대신 대상 픽셀 집합을 비교한다. 정답 픽셀이 4개, 예측 픽셀이 4개이고 겹치는 픽셀이 2개라면 다음과 같다.

```text
Mask IoU = 교집합 / 합집합 = 2 / (4 + 4 - 2) ≈ 0.333
Dice = 2 × 교집합 / (정답 픽셀 수 + 예측 픽셀 수)
     = 4 / 8 = 0.500
```

```python
import numpy as np

true_mask = np.array([
    [0, 0, 0, 0],
    [0, 1, 1, 0],
    [0, 1, 1, 0],
    [0, 0, 0, 0],
], dtype=bool)

pred_mask = np.array([
    [0, 0, 0, 0],
    [0, 0, 1, 1],
    [0, 0, 1, 1],
    [0, 0, 0, 0],
], dtype=bool)

intersection = np.logical_and(true_mask, pred_mask).sum()
union = np.logical_or(true_mask, pred_mask).sum()
total = true_mask.sum() + pred_mask.sum()

print(f"Mask IoU = {intersection / union:.3f}")
print(f"Dice = {2 * intersection / total:.3f}")
```

이 예시는 두 Mask에 대상 픽셀이 있는 경우다. 실제 평가에서는 양쪽이 모두 비어 분모가 0이 되는 상황의 점수 정책을 정하고 일관되게 적용한다.

같은 두 영역에서 Dice는 IoU보다 크거나 같으므로 서로 다른 지표의 숫자만으로 모델을 비교하지 않는다. 같은 지표와 같은 집계 조건으로 비교한다.

## 6. 실제 예측과 함께 오류 분석

| 오류 유형 | 이미지에서 확인할 내용 |
| --- | --- |
| 영역 누락 | 실제 결함 일부가 빠졌는가? |
| 영역 과다 예측 | 정상 영역까지 포함했는가? |
| 위치 오류 | 다른 위치에 Mask가 생겼는가? |
| 객체 구분 오류 | 서로 다른 객체가 연결되거나 잘못 나뉘었는가? |

정답과 예측 Mask를 이미지 위에 겹쳐 보면 오류의 위치와 형태를 확인할 수 있다. Instance Segmentation은 개별 객체의 대응과 누락도 평가한다. 실제 균열 세 개 중 두 개의 Mask만 정확하게 예측했다면 나머지 균열의 누락도 성능에 반영해야 한다.

## 배운 점

Box와 Mask의 겹침 점수가 높아도 놓친 객체가 많으면 검출 목적을 충족하기 어렵다. 위치 일치도, 오검출, 객체 누락을 함께 살피고, 제조 공정에서 어떤 오류의 영향이 큰지에 따라 Precision과 Recall을 해석해야 한다.

## 학습자료

2026년 11월 23일 제공된 이미지 분류와 객체 탐지 평가의 차이 및 IoU, 객체 탐지 모델의 Precision·Recall·mAP, 세그멘테이션 모델의 평가 자료를 바탕으로 작성했다.