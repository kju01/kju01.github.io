---
title: Pix2Vox++ review
author: kju
layout: post
use_math: True
categories: 
 - deep learning (vision)

---
### summary    
본 논문은 2D 이미지들을 모아 3D로 복원하는 multi-view base의 모델 중에서 pix2vox([논문 리뷰 링크](https://kju01.github.io/deep%20learning%20(vision)/2024/08/30/vision-pix2vox.html "Pix2Vox review"))를 향상시킨 모델에 대한 것이다. 크게 세가지가 추가되었는데, VGG 구조가 아닌 resnet구조를 이용한 것, single view를 통해 encoder, decoder, refiner를 먼저 training한다는 것과 context-aware-fusion구조가 바뀌었다는 점이다. sota기준 이미지를 입력으로 받는 컨볼루션 네트워크 구조 중에서는 제일 성능이 좋은 모델이다.

[-논문 링크-](https://arxiv.org/abs/2006.12250, "Pix2Vox++: Multi-scale Context-aware 3D Object Reconstruction from Single and Multiple Images")      [-source code-](https://gitlab.com/hzxie/Pix2Vox "github code")

## 모델구조
<hr>

모델 구조에 대해서는 pix2vox와 비교하여 변경된 부분만 설명한다. 기존의 모델은 pix2vox논문 또는 블로그에 있는 논문리뷰를 참고바람.

### encoder 구조
<hr>

![model-structure](/post_images/pix2vox++/model_structure.PNG "pix2vox++ 모델 구조") 

pix2vox와 비교해서 바뀐부분은 encoder부분에 있는데 기존의 pretrain된 VGG모델을 쓰지 않고 Pix2Vox++/F 모델의 경우 ResNet18에서 일부 layer를, Pix2Vox++/A 모델의 경우 ResNet50에서의 일부를 이용하여 학습을 진행한다.

### merger 구조
<hr>

![model-structure](/post_images/pix2vox++/merger.PNG "pix2vox++ merger")

또 다른 변경점은 context-aware-fusino(merger)에 있는데 scoring을 하는 network의 구조가 바뀌었다. 기존의 경우 9,16,8,4,1 출력 채널을 가진 convolution layer를 통과시켰지만 pix2vox++의 경우 9,9,9,9,9,1 출력 채널을 가진 convolution layer로 기존의 layer에 비해 하나가 추가되었다. 또한 첫번째부터 네번째 layer까지의 출력을 concat을 하여 다섯번째 layer를 통과시킨다. resnet의 skip connection과 같은 구조를 이용하였다.

## Implementation Details
<hr>

 학습과정도 변화가 있었는데, 먼저 context-aware fusion을 제외하고 250 epoch만큼 single-view로 학습을 진행한다. learning rate는 0.001로 하고 150 epoch을 넘어가면 1/2배만큼 감소시킨다. optimizer는 Adam을 사용하고 Adam의 parameter는 pytorch기준 default값을 이용한다.
 이렇게 학습된 encoder, decoder, refiner를 이용하여 100 epoch만큼 multi-view 로 학습을 진행한다. 이 때는 context-aware fusion(merger)를 활성화하고 학습을 진행한다.

 ## Results
<hr>


![model-structure](/post_images/pix2vox++/result1.PNG "pix2vox++ result1")

${32^3}$ 으로 resoultion 기준으로 single view로 학습한 결과는 figure6과 같다.

![model-structure](/post_images/pix2vox++/result2.PNG "pix2vox++ result2")

여러 시점으로 학습한 IoU와 F-score는 Table3과 같다.

![model-structure](/post_images/pix2vox++/result3.PNG "pix2vox++ result3")

ShapeNet-Cars 데이터 기준으로 ${128^3}$ resolution한 결과는 figure13과 같다.

이 외의 실험 결과는 논문을 직접 참고하길 바란다.


## etc.
<hr>

 3D reconstruction 모델 중에서 pix2vox++모델의 경우 이미지를 입력으로 받는 다중 시점 3D convolution 모델 중에서 제일 좋은 성능을 보인다.
 이 외에 transformer구조를 이용한 다중 시점 모델이 있다. 이 모델의 경우 시점이 적을 때는 pix2vox++모델이 더 좋은 성능을 보이지만 시점이 많을 수록 transformer구조가 더 좋은 성능을 보인다.
 또한 이미지를 입력으로 받지 않고 point cloud를 입력으로 받는 3D모델의 경우 이미지를 입력으로 받는 다중 시점 모델보다 더 좋은 성능을 보인다.