# TIL | 2026-11-20 — 이미지 분류와 예측 확률 해석

상품 이미지를 신발·가방·의류로 분류하는 사례를 통해 모델의 입력·출력 구조와 사전학습 모델의 추론 과정을 배웠다. 클래스별 점수인 Logit을 Softmax로 변환하고, 예측 결과를 실제 정답과 비교하는 방법도 살펴봤다.

## 1. 이미지 분류의 입력과 출력

이미지 분류는 이미지 전체가 속하는 클래스를 예측하는 태스크다. 이번 학습에서는 이미지 한 장에 정답 클래스 하나가 있다고 가정했다.

| 문제 | 클래스 구성 | 예시 |
| --- | --- | --- |
| 이진 분류 | 두 클래스 | 정상·불량 |
| 다중 클래스 분류 | 세 개 이상의 클래스 | 신발·가방·의류 |

일반적인 PyTorch CNN은 `(N, C, H, W)` 형태의 입력을 받는다. RGB 이미지 4장을 224 × 224로 준비하고 클래스가 3개라면 입력은 `(4, 3, 224, 224)`, 출력은 `(4, 3)`이다. 출력의 각 행은 이미지 하나, 각 열은 클래스 하나의 점수다.

```text
입력 이미지 → 전처리 → CNN 특징 추출 → 분류기
→ 클래스별 Logit → 가장 높은 점수의 클래스 선택
```

```python
import torch
from torch import nn

model = nn.Sequential(
    nn.Conv2d(3, 8, kernel_size=3, padding=1),
    nn.ReLU(),
    nn.AdaptiveAvgPool2d((1, 1)),
    nn.Flatten(),
    nn.Linear(8, 3),
)

images = torch.randn(4, 3, 224, 224)
logits = model(images)
predictions = logits.argmax(dim=1)

print(logits.shape)       # torch.Size([4, 3])
print(predictions.shape)  # torch.Size([4])
```

`AdaptiveAvgPool2d((1, 1))`은 채널별 공간 정보를 1 × 1로 요약하고, `Flatten`은 이를 분류기에 전달할 형태로 만든다. `Linear(8, 3)`은 8개 특징으로 클래스 점수 3개를 출력한다. 위 예시는 무작위 입력과 초기화된 모델로 구조를 확인한다.

이미지 분류 결과는 이미지 전체의 범주를 나타낸다. 이미지 안의 여러 물체를 각각 찾아 위치까지 표시해야 한다면 객체 탐지와 같은 태스크를 검토한다.

## 2. 사전학습 모델로 추론하기

추론(Inference)은 학습된 모델로 새로운 입력의 예측값을 얻는 과정이다. ImageNet으로 사전학습된 ResNet18은 기존 1,000개 클래스에 대한 점수를 출력하므로 이미지 한 장의 출력은 `(1, 1000)`이다.

| 단계 | 사용할 정보와 기능 |
| --- | --- |
| 모델 준비 | 가중치를 불러오고 `model.eval()`로 평가 모드 설정 |
| 입력 준비 | RGB 변환과 `weights.transforms()`로 권장 전처리 적용 |
| 배치 구성 | `unsqueeze(0)`으로 배치 차원 추가 |
| 추론 | `torch.inference_mode()`에서 점수 계산 |
| 클래스 매핑 | 인덱스를 `weights.meta["categories"]`의 이름과 연결 |

```python
import torch
from PIL import Image
from torchvision.models import resnet18, ResNet18_Weights

weights = ResNet18_Weights.DEFAULT
model = resnet18(weights=weights)
model.eval()

image = Image.open("sample.jpg").convert("RGB")
images = weights.transforms()(image).unsqueeze(0)

with torch.inference_mode():
    logits = model(images)

index = logits.argmax(dim=1).item()
label = weights.meta["categories"][index]
print(label)
```

이 학습 예시에는 `sample.jpg`가 필요하고, 최초 사용 시 가중치가 다운로드될 수 있다. `eval()`은 모델의 평가 동작을 설정하고, `inference_mode()`는 추론 중 기울기 기록 등 학습용 처리를 줄인다.

기존 클래스가 서비스의 분류 기준과 맞으면 결과를 활용할 수 있다. 정상·불량처럼 새 클래스가 필요하면 분류기를 교체하고 새 데이터로 학습한다. 이때 Feature Extraction이나 Fine-tuning으로 학습 범위를 선택한다.

## 3. Logit과 Softmax

Logit은 클래스별 정규화되지 않은 점수이며 음수와 1보다 큰 값도 가능하다. Softmax는 한 이미지의 클래스 점수를 합계가 1인 분포로 변환한다.

| 클래스 | Logit | Softmax 결과 |
| --- | ---: | ---: |
| 신발 | 2.0 | 약 0.659 |
| 가방 | 1.0 | 약 0.242 |
| 의류 | 0.1 | 약 0.099 |

```python
import torch

classes = ["신발", "가방", "의류"]
logits = torch.tensor([
    [2.0, 1.0, 0.1],
    [0.2, 2.5, 0.5],
    [1.5, 1.3, 1.0],
])

probabilities = torch.softmax(logits, dim=1)
predictions = probabilities.argmax(dim=1)

for row, index in enumerate(predictions.tolist()):
    print(classes[index], f"{probabilities[row, index].item():.1%}")
```

위 예시의 예측 클래스와 확률은 각각 신발 약 65.9%, 가방 약 80.9%, 신발 약 41.2%다. 세 번째 입력은 다른 클래스에도 점수가 분산되어 있다. `dim=1`은 각 이미지의 클래스 방향으로 계산한다는 의미다.

Softmax는 클래스 점수의 순서를 유지하므로 최종 클래스만 필요하면 Logit에 바로 `argmax(dim=1)`을 적용할 수 있다. 확률 분포를 함께 확인하면 다른 후보와의 점수 차이도 살펴볼 수 있다.

## 4. 예측값 검증과 활용

예측 확률은 모델이 부여한 값이다. 실제 정답 빈도와 얼마나 잘 일치하는지는 별도로 평가해야 한다. 실제로 의류인 이미지에 신발 확률 97%를 부여하는 오분류가 가능하며, 학습하지 않은 종류의 입력에도 기존 클래스에 높은 값을 줄 수 있다.

| 확인 항목 | 검토할 내용 |
| --- | --- |
| 정답 비교 | 실제 클래스와 예측 클래스가 일치하는가? |
| 클래스 매핑 | 인덱스와 이름의 대응이 올바른가? |
| 입력 조건 | 크기·RGB 채널·정규화가 가중치의 기준에 맞는가? |
| 오류 사례 | 특정 클래스나 촬영 조건에서 오분류가 반복되는가? |
| 자동 처리 기준 | 검증 데이터에서 확인한 성능과 오류 영향을 반영했는가? |

4개 중 3개를 맞히면 Accuracy는 75%다. 전체 정확도와 함께 클래스별 오류와 원본 이미지를 확인한다. 자동 분류와 확인 요청을 나누는 확률 기준도 검증 데이터를 사용해 결정한다.

## 배운 점

배치의 이미지 수와 클래스 수를 알면 출력 텐서의 구조를 예상할 수 있게 됐다. 추론에서는 전처리와 클래스 매핑까지 확인해야 의미 있는 결과를 얻을 수 있고, 높은 Softmax 값이 나온 사례도 실제 정답과 비교해 오류를 기록해야 한다.