---
title: Pix2Vox review
author: kju
layout: post
categories: 
 - deep learning (vision)

---
### summary    
본 논문은 2D 이미지들을 모아 3D로 복원하는 multi-view base의 모델에 관한 연구이다.    
main idea는 기존의 다중 시점 모델의 경우 2D feature 단계에서 각 시점의 특징을 종합하는 반면 pix2vox모델의 경우 3D feature단계에서 각 시점의 특징을 종합하였다는 특징이 있다. 그 외에 refiner라는 3D encoder-decoder를 추가하여 최종 3D객체를 생성한다.

[-논문 링크-](https://arxiv.org/abs/1901.11153, "Pix2Vox: Context-aware 3D Reconstruction from Single and Multi-view Images")      [-source code-](https://github.com/hzxie/Pix2Vox "github code")

## 모델 구조
<hr>

![model-structure](/post_images/pix2vox/pix2vox_architecture.PNG "pix2vox 모델 구조") 

 **pix2vox** 모델의 전체적인 구조는 **figure 2**와 같다.
 먼저 각 시점에서의 이미지를 2D encoder를 통해 2D feature를 생성하게 된다. 생성된 2D feature는 reshape과정을 거친 후 3D feature가 된 후 3D decoder를 거쳐 **coarse Volumes**를 생성하게 된다. 이렇게 생성된 coarse Volumes, 즉 3D feature들은 **context-aware Fusion**을 거쳐 각 시점의 feature들이 종합된 **Fused Volume**이 된다. 이 Fused Volume을 최종 결과물로 쓰는 경우를 Pix2Vox-F모델이라고 하고, 이를 **Refiner**를 거쳐 최종 Volume을 생성한다면 Pix2Vox-A모델이 된다.

![model-network](/post_images/pix2vox/pix2vox_architecture2.PNG "pix2vox 네트워크 구조") 

 각 네트워크의 상세적인 convolution layer 정보의 경우 **figure3**와 같다. 

### context aware fusion 
<hr>

 pix2vox모델에서 핵심인 부분으로 볼 수 있는 **context aware fusion**에 대해 자세히 설명하고자 한다.

![model-context aware fusion](/post_images/pix2vox/context_aware_fusion.PNG "context aware fusion") 

context aware fusion의 아이디어는 **figure4**와 같다. 한 시점에서 3D 객체를 생성하기 위한 정보의 양은 차이가 있을 것이다. 위 그림에서 첫번째 시점과 두번째 시점은 3D 객체에 대한 정보가 많아 coarse volume의 생성이 잘 되었으며 이를 heat map으로 표현한 score map에서도 1에 가까운 voxel이 많이 있다. 하지만 세번째 시점과 같이 밑에서 본 시점의 경우 책상의 윗부분에 대한 정보가 부족하기 때문에 coarse volume의 생성이 잘 안되었으며 score map 또한 낮은 점수로 분포되어 있다. 그러므로 context aware fusion 구조를 이용하여 각 시점마다 부족한 부분을 서로 보완해준다는 아이디어이다.

![model-context aware fusion2](/post_images/pix2vox/context_aware_fusion2.PNG "context aware fusion") 

 context aware fusion의 네트워크 구조는 **figure5**와 같다. 3D decoder에서 최종적으로 출력되는 3D feature maps와 layer하나를 통과하기 전의 3D feature maps를 concat하여 context volume을 생성하게 된다.

 이렇게 생성된 context volume은 context scoring 네트워크를 통과하여 점수를 계산하게 된다. context scoreing 네트워크의 경우 3D convolutional layers로 구성되어 있으며 각각 9,16,8,4,1의 출력 채널로 모든 layer는 kernel 크기는 3, padding은 1이며 각 레이어마다 batch normalization과 leaky ReLU를 거치게 된다. 


 ![model-score-normalization](/post_images/pix2vox/score_normalization.png "score-normalization") 

 이렇게 계산된 learned score인 m은 모든 시점에 대해 score normalization을 계산하게 된다. score normalization의 경우 위의 이미지와 같이 각 feature 차원을 기준으로 softmax를 취하게 된다. 사진의 예시는 2D feature이지만 본 모델에서는 3D feature이다.

![model-scoring](/post_images/pix2vox/scoring.PNG "scoring") 

score normalization의 식은 위와 같다.

### Refiner
<hr>

refiner의 경우 **figure3**과 같이 3D encoder-decoder 구조에 U-Net에서 이용되는 skip-connection을 추가한 구조이다.
 encoder의 각 layer의 경우 filter의 크기는 ${4^3}$이며 padding은 2이고 batch normalizaition과 leaky ReLU를 지나게된다. 그리고 max pooling layer의 경우 kernel의 크기가 ${2^3}$이다. 출력채널은 순서대로 32,64,128이다.
 이렇게 생성된 feature는 두개의 fully connected layer를 지나게 된다. 각 layer의 차원은 2048, 8192이다.
 Decoder의 각 layer의 경우 filter의 크기는 ${4^3}$ 이며 padding은 2, stride는 1이다. 각 layer마다 batch normalization과 ReLU activation을 지나게 된다. 출력채널은 encoder와 반대로 64,32,1이다. 최종 출력값에 sigmoid를 취하여 0~1사이의 확률분포값을 형성하고 threshold에 따라 voxel을 생성하게 된다.


## loss function, Metrics and Dataset
<hr>

![loss](/post_images/pix2vox/loss.PNG "loss") 

loss function의 경우 위의 식과 같다. voxel-wise binary cross entropy 이며 ${N}$ 은 ground truth에서의 voxel갯수이며 ${p_i}$는 sigmoid를 통과한 확률분포값, ${gt_i}$는 ground truth의 값이다.

![iou](/post_images/pix2vox/iou.PNG "iou") 

metric의 경우 위의 식과 같이 IoU를 사용하였으며, ${p}$와 ${gt}$는 loss function과 같으며 ${I(p_(i,j,k) > t)}$는 각 확률분포에서 t(threshold)보다 크다면 0, 아니면 1의 값을 부여한다는 의미이다. 본 논문에서는 t를 0.3으로 설정하였다.

**Dataset**의 경우 ShaepNet을 사용하였다.

## 결과
<hr>

![result1](/post_images/pix2vox/result1.PNG "result1") 

왼쪽의 경우 단일시점에서의 생성을 비교한 것이고 오른쪽의 경우 3개의 시점에서의 생성을 비교한 것이다.

![result2](/post_images/pix2vox/result2.PNG "result2") 

Pix3D에서의 단일시점 생성을 pix2vox와 비교한 것이다.

![result3](/post_images/pix2vox/result3.PNG "result3") 

**figure7**의 경우 Pix3D와 단일시점 생성을 비교한 것이고, **figure8**의 경우 5개의 시점에서 ShapeNet에 없는 물체에 대한 생성을 보여준다.

![result4](/post_images/pix2vox/result4.PNG "result4") 

**Table4**의 경우 batchsize 1을 기준으로 단일시점에서 ShapeNet dataset을 이용하였을때 모델의 메모리 사용량을 비교한 것이다. Pix2Vox-F의 경우 비교대상보다 memory를 덜 사용함에도 불구하고 성능이 좋았으며, Pix2Vox의 경우 학습시간이 전체적으로 적게 걸림을 확인할 수 있다.