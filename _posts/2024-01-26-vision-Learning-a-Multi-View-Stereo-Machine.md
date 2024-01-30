---
title: 논문리뷰 - vision - Learning a Multi-View Stereo Machine (2017)
subtitle: 3D Vison 생성모델 분야
author: kju
layout: post
use_math: True
categories:
 - deep learning (vision)
---
본 논문은 2D 이미지들을 모아 3D로 복원하는 multi-view base의 모델에 관한 연구이다.    
main idea는 1D data를 2D로 복원하는 unprojection것과 같이 2D data를 3D로 unprojection하는 부분을 설계함으로서 3D reconstruction을 하는 것이다.

[논문 링크](https://arxiv.org/abs/1708.05375, "Learning a Multi-View Stereo Machine")

## 1. Abstract
<hr>
본 논문에서는 learnt system for multi-view stereopsis를 발표합니다. 최근에 3d reconstruction을 위한 학습 방법과 다르게 우리는 viewing rays 에 따른 feature projection 과 unprojection을 통해 기본적인 3D geometry을 활용한다고 합니다. 이러한 작업을 differentiable manner로 정의하여 3D reconstruction에서 end to end로 학습(입력부터 출력까지의 전체 과정을 학습)이 가능하다고 합니다. 이를 통해 고전적인 방식보다 더 적은 이미지(single image 까지도) reconstruction이 가능하다고 합니다.

## 2. Introduction
<hr>
Multi-view stereopsis(MVS)는 주어진 이미지들과 카메라 정보가 있을 때 이를 통해 3D 공간을 표현합니다. 이 표현은 disparity maps, voxel을 통한 3D volumn, signed distance field... 등이 될 수 있습니다. 최근에는 voxel을 통한 표현이나 다각형 메쉬를 통해 reconstruction을 하는데 중점을 둡니다.      

이 논문에서는 Learnt Stereo Machines(LSM)을 제안합니다. LSM은 projective geometry를 갖추고 있어 3D공간에서의 추론이 가능하고 MVS에서의 기하학적 구조를 효과적으로 활용할 수 있습니다. 실루엣이나 이미지의 일관성과 같은 특정 단서를 활용하는 고전적인 방법과 달리 본 논문의 시스템은 특정 인스턴스에 관련된 단서를 활용하는 방법을 학습하면서도 보이지 않는 영역의 geometry를 예측하기 위해 모양에 대한 prior(사전 정보)를 사용합니다.    

3D-R2N2의 경우 cnn을 통한 학습을 통해 예측하므로 semantic cues(의미론적 단서)에만 의존하였으나, 본 방법은 geometric cues(기하학적 단서)또한 활용합니다. 이를 통해 더 적은 이미지를 통해 reconstruction이 가능하며 성공적인 일반화를 보여주었다고 합니다.

## 3. Learnt Stereo Machines
<hr>

![model-structure](/post_images/Multi-View-stereo-machine/model-structure.PNG "LSM 모델 구조")   

먼저 dense features를 image space에서 계산합니다. 그리고 이러한 특징들은 camera pose를 기반으로 matching volume으로 변환됩니다. 이 과정에서 unprojection을 통해 volume으로 변환됩니다. 이 matching volume의 optimum(최적값)dms 3D volume/surface/dispartiy maps의 추정치로 제공됩니다.

본 모델은 figure 1과 같은 과정을 따릅니다. 1. 입력이미지 ${I_i}$ 는 먼저 image encoder를 통해 처리됩니다. 이는 각 이미지에 대해 하나의 dense feature maps ${F_i}$ 를 생성합니다  2. 그런 다음 features는 camera pose ${P_i}$ 를 통한 unprojection을 통해 3D feature grids ${G_i}$ 를 생성합니다. 이 unprojection 작업은 features를 epipola line을 따라 정렬하여 효율적인 local matching이 가능하게 합니다. 3. 이러한 matching은 neural network를 통해 모델링되며 이 network는 unprojection된 grid를 순차적으로 처리하여 local matching costs인 ${G^p}$ 를 생성합니다. 이 cost volume은 일반적으로 noisy하므로 이를 smooth하게 처리해야합니다. 4. 이를 위하여 본 논문에서는 feedforward 3d convolution-deconvolution cnn사용을 제안합니다. 이 cnn은 $G^p$ 는 smoothed 3D grid $G^0$로 변환합니다. 5. 원하는 출력에 따라 최종 grid를 volumetric occupancy map 또는 다시 2d feature maps ${O_i}$ 로 투영합니다. 이러한 2D maps은 이후 view depth/disparity map과 같은 모양에 대한 view specific representation으로 매핑됩니다.    

본 시스템의 주요 구성 요소는 미분 가능한 projection 및 unprojection 작업입니다. 이러한 과정을 통해 시스템을 end-to-end 학습이 가능하면서도 3D geometry를 metrically하게 정확히 주입하였다고 합니다.     

이 LSM에 대한 변형으로 volume occupancy maps을 생성하는 Voxel LSM, 입력 이미지당 depth map(Depth LSM)을 출력하는 것을 제시하였습니다.

### 1) 2D Image Encoder    
stereo algorithm 의 첫 step은 이미지들을 매칭하기위한 좋은 features를 계산하는 것입니다. 입력이미지 ${I_i}$ 는 encoder를 거쳐 2D space의 dense feature maps ${F_i}$가 됩니다. 이 features는 camera parameters를 이용한 unprojection module을 통과하여 metric 3D space이 됩니다.

![unprojection](/post_images/Multi-View-stereo-machine/backprojection.PNG "unprojection")

### 2) Differentiable Unprojection   
unprojection 과정의 목표는 2D image frame의 정보를 3D world frame의 정보로 옮기는 것이다. 2D point ${p}$ ,feature 표현 ${F(p)}$, 및 global 3D grid 표현이 주어졌을때 p에 대한 viewing ray를 따라 ${p}$위치의 metric 3D grid에 ${F(p)}$를 복제합니다.(Figure 2에 자세히 나와있음) 카메라 자체 특성인 내부 camera matrix ${K}$ 와 카메라 외부 특성인 외부 camera matrix ${[R|t]}$가 특정되었을 때 unprojection 과정은 이 camera pose를 이용하여 world에서의 viewing rays를 추적하고 image features를 3D world grid의 voxels로 복사합니다. viewing rays를 해석적으로 추적하는 대신 3D grid의 블록 중심 ${X_w^k}$이 주어졌을때, 각 블록의 특징을 계산합니다. 이를 위해 camera projection equations ${p_k' = K[R|t]X_w^k}$를 이용하여 블록을 이미지 공간으로 투영합니다. 이때 ${p_k'}$는 continuous한 양이지만 ${F}$는 이산적인 2D 위치에서 정의됩니다. 따라서 이산 grid에서 샘플링하기 위해 differentiable bilinear sampling을 사용합니다. 이로서 ${X_w^k}$에서 특징을 얻습니다.   
이를 통해 epipolar 제약 조건을 쉽게 강제하게 됩니다. 따라서 이러한 unprojected grid에서의 추가적인 처리는 이미지 공간에서의 특징 매칭을 위한 장거리 이미지 연결의 필요성을 줄여주어 이미지 연결이 필요없게 됩니다. 또한 feature maps에서 bilinearly sampling하는 것은 3D grid의 voxel이 카메라의 거리에 대한 영향을 감소시켜줍니다. 본 논문에서의 접근 방식을 통해 모든 voxel은 "soft" 특징을 가지게 되어 feature grids ${G^f}$를 부드럽고 stable gradient를 가지도록 합니다. 이를 통해 네트위크가 기하학을 학습할 필요가 없게 됩니다. 이 과정은 이미지 광간의 특징맵 ${F_i}$을 metric 3D space에 있는 feature grid ${G^f_i}$ 로 다시 투영할 때 사용됩니다.    
single image prediction의 경우 ray에 기하학적 특징(depth and ray direction)을 추가하여 예측을 용이하게 합니다.

### 3) Recurrent Grid Fusion (3D-R2n2의 GRU와 유사)    
Gated Recurrent Unit(GRU)의 3D convolution 변형을 사용하여 grids ${[{G^f_i}]^n_{i=1}}$를 단일 grid ${G^p}$로 결합합니다. 주의할 점은 입력 이미지에 대한 순서에 따라 출력이 영향을 받아 훈련중에는 이미지 순서를 무작위로 섞으면서 진행해야 합니다.

### 4) 3D grid Reasoning     
fused grid ${G^p}$을 모델링한 3D U-Net을 사용하여 사진의 일관성을 직접 평가하고 이미지가 일치하는 곳의 표면을 추출합니다. 이를 통해 ${G^p}$는 ${G^o}$로 변환됩니다. 이 네트워크를 통해 ${G^p}$에 있는 형태 정보를 활용하여 부분 정보가 가시적일 때에도 완전한 형태를 생성할 수 있도록 합니다. 이렇게 생성된 ${G^o}$는 full 3D supervision (Voxel LSM)에서는 voxel 점유맵으로 될 수도 있습니다. 또한 ${G^o}$는 3D world의 최종 표현을 포함하는 feature grid로도 볼 수 있으며, 이를 사용해 projection을 진행하여 rendering될 수도 있습니다.

### 5) Differentiable Projection    
본 논문에서는 ray를 따라 균일한 간격의 z value에서 depth planes의 위치에서 sampling하는 plane sweeping 접근 방식을 채택하였습니다.

### 6) Architecture Details    

- Voxel LSM(V-LSM)   
최종 grid ${G^o}$를 3D convolution을 거쳐 softmax연산을 적용하여 확률적인 voxel occupancy map으로 변환합니다. 이를 ground truth와 voxel occpancy map간의 binary cross entropy loss를 사용합니다. 최종 출력은 voxel occupancy grid입니다.    

- Depth-LSM(D-LSM)    
먼저 grid ${G^o}$를 2d feature map ${O_i}$로 투영하고, 이를 ray에 따른 reduction function을 학습하기 위해 1x1 convolution을 사용한 후 deconvolution layers를 통해 feature map을 입력이미지의 크기로 upsampling하여 metric depth maps ${d_i}$로 변환합니다. train과정에서는 L1 Loss를 사용합니다. 또한 이미지 encoder의 초기 layers와 depth maps를 생성하는 마지막 deconvolution layer 사이에 skip connection을 추가하여 이미지의 high frequency정보를 얻을 수 있도록 합니다. 최종 출력은 input view당 depth map입니다.


## 4. Experiments
<hr>

## 5. Discussion
<hr>

## 6. summary
<hr>