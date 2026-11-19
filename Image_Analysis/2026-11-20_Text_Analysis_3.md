# TIL | 2026-11-20 — CNN 특징 추출과 전이학습

제품의 정상·불량 분류를 예로 들어 합성곱과 풀링이 특징 맵을 어떻게 바꾸는지 배웠다. 이어서 사전학습 CNN의 분류기를 교체하고, 새로운 데이터에 맞춰 학습할 파라미터의 범위를 정하는 방법을 살펴봤다.

## 1. 합성곱과 특징 맵

CNN(Convolutional Neural Network)은 합성곱 필터를 이용해 이미지의 공간적 특징을 학습한다. 필터는 작은 영역을 따라 이동하며 값을 계산하고 **특징 맵(Feature Map)**을 만든다. 학습 과정에서는 분류에 유용한 패턴을 추출하도록 필터의 가중치를 조정한다.

일반적인 합성곱 레이어에서 **필터 수는 출력 특징 맵의 채널 수**에 해당한다. RGB 3채널 입력에 필터 8개를 적용하면 출력은 8채널이다. 가로·세로 크기는 커널, 패딩, 스트라이드 등의 설정에 따라 달라진다.

Max Pooling은 영역 안의 최댓값을 선택해 공간 정보를 요약한다. 2 × 2 영역의 값이 1, 3, 2, 5라면 출력은 5다. 크기 2와 간격 2로 32 × 32 특징 맵을 풀링하면 16 × 16이 된다.

| 단계 | Shape | 변화 |
| --- | --- | --- |
| RGB 이미지 1장 | `(1, 3, 32, 32)` | 배치·채널·세로·가로 |
| 합성곱 3 → 8채널 | `(1, 8, 32, 32)` | 패딩으로 공간 크기 유지 |
| Max Pooling | `(1, 8, 16, 16)` | 공간 크기 축소 |
| 합성곱 8 → 16채널 | `(1, 16, 16, 16)` | 출력 채널 증가 |

```python
import torch
from torch import nn

x = torch.randn(1, 3, 32, 32)
conv1 = nn.Conv2d(3, 8, kernel_size=3, padding=1)
pool = nn.MaxPool2d(kernel_size=2)
conv2 = nn.Conv2d(8, 16, kernel_size=3, padding=1)

feature1 = torch.relu(conv1(x))
feature2 = pool(feature1)
feature3 = torch.relu(conv2(feature2))

for feature in [x, feature1, feature2, feature3]:
    print(feature.shape)
```

이 예시는 무작위 입력과 초기화된 필터로 Shape 변화를 확인한다. 실제로 학습되는 특징은 데이터와 모델 구조에 따라 달라진다.

## 2. 레이어 깊이와 정보 손실

초기 레이어는 선·모서리·밝기 변화, 중간 레이어는 패턴과 부분 형태, 깊은 레이어는 복합적인 분류 특징을 학습하는 경향이 있다. 깊은 모델은 표현력이 커질 수 있지만 연산량과 과적합 가능성도 함께 고려해야 한다.

풀링으로 특징 맵을 줄이면 이후 연산 부담이 줄어든다. 작은 균열이나 흠집이 분류 근거인 제조 이미지에서는 축소 과정에서 세부 형태와 위치가 얼마나 유지되는지 확인해야 한다.

## 3. 사전학습 모델과 새로운 분류기

| 방식 | 시작 가중치 | 확인할 점 |
| --- | --- | --- |
| Scratch Learning | 무작위 초기화 | 데이터 양과 다양성, 모델 크기, 학습 자원이 충분한지 |
| 전이학습 | 기존 데이터로 학습한 가중치 | 사전학습 특징이 새 도메인에도 유효한지 |

전이학습에서는 이미지 특징을 만드는 **Backbone**과 특징에서 클래스를 예측하는 **Classifier**를 구분한다. ImageNet 등으로 사전학습한 모델의 기존 분류기를 정상·불량의 2개 출력으로 교체할 수 있다. 이때 새 분류기의 가중치는 초기화되므로 새 데이터로 학습해야 한다.

```python
from torch import nn
from torchvision.models import resnet18, ResNet18_Weights

scratch_model = resnet18(weights=None)
pretrained_model = resnet18(weights=ResNet18_Weights.DEFAULT)

for model in [scratch_model, pretrained_model]:
    model.fc = nn.Linear(model.fc.in_features, 2)
```

위 구성에서 두 모델의 최종 분류기는 512개 특징을 받아 2개 출력을 만든다. 사전학습 가중치는 처음 사용할 때 다운로드가 필요할 수 있다. 데이터가 적을 때 기존 특징을 활용할 수 있지만, 특수 제조 이미지처럼 도메인이 다르면 실제 Validation 성능으로 적합성을 확인한다.

## 4. Feature Extraction과 Fine-tuning

| 학습 방식 | 기존 특징 추출 부분 | 새 분류기 |
| --- | --- | --- |
| Feature Extraction | 전체 고정 | 학습 |
| Partial Fine-tuning | 일부 학습 | 학습 |
| Full Fine-tuning | 전체 학습 | 학습 |

PyTorch에서는 `requires_grad`로 파라미터의 기울기 계산 대상을 정한다. 아래는 마지막 특징 추출 단계인 `layer4`와 분류기만 학습 대상으로 설정하는 예시다.

```python
for parameter in pretrained_model.parameters():
    parameter.requires_grad = False

for parameter in pretrained_model.layer4.parameters():
    parameter.requires_grad = True

for parameter in pretrained_model.fc.parameters():
    parameter.requires_grad = True

trainable_count = sum(
    parameter.numel()
    for parameter in pretrained_model.parameters()
    if parameter.requires_grad
)
print(trainable_count)
```

학습자료의 2클래스 ResNet18에서 분류기만 학습하면 1,026개, `layer4`와 분류기를 학습하면 8,394,754개, 전체를 학습하면 11,177,538개 파라미터가 대상이 된다. 이는 학습 범위 비교이며 성능 수치는 아니다.

## 5. 학습 방식 선택과 배운 점

같은 검증 조건에서 Validation 성능, Train과 Validation의 차이, 학습 시간과 연산 부담을 비교한다. Train 정확도가 높고 Validation 성능이 낮다면 과적합을 검토한다. 기존 특징이 충분하면 분류기만 학습하는 방식으로 시작하고, 필요한 경우 학습 범위를 늘려 성능 변화를 확인할 수 있다.

이번 학습으로 입력 채널과 특징 맵의 채널을 구분하게 됐다. 입력의 3채널은 RGB 정보이며 이후 8채널이나 16채널은 학습 가능한 필터가 만든 특징이다. 또 분류기 교체와 파라미터 고정은 학습을 준비하는 단계이므로, 최종 선택은 새 데이터에서의 검증 결과를 보고 결정해야 한다.