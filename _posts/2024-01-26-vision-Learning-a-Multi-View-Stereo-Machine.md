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

본 모델은 figure 1과 같은 과정을 따릅니다. 1. 입력이미지 ${I_i}$ 는 먼저 image encoder를 통해 처리됩니다. 이는 각 이미지에 대해 하나의 dense feature maps $\{F_i\}_{i=1}^{n}$ 를 생성합니다  2. 그런 다음 features는 camera pose $\{P_i\}_{i=1}^{n}$ 를 통한 unprojection을 통해 3D feature grids $\{G_i\}_{i=1}^{n}$ 를 생성합니다. 이 unprojection 작업은 features를 epipola line을 따라 정렬하여 효율적인 local matching이 가능하게 합니다. 3. 이러한 matching은 neural network를 통해 모델링되며 이 network는 unprojection된 grid를 순차적으로 처리하여 local matching costs인 $G^p$ 를 생성합니다. 이 cost volume은 일반적으로 noisy하므로 이를 smooth하게 처리해야합니다. 4. 이를 위하여 본 논문에서는 feedforward 3d convolution-deconvolution cnn사용을 제안합니다. 이 cnn은 $G^p$ 는 smoothed 3D grid $G^0$로 변환합니다. 5. 원하는 출력에 따라 최종 grid를 volumetric occupancy map 또는 다시 2d feature maps $\{O_i\}_{i=1}^{n}$ 로 투영합니다. 이러한 2D maps은 이후 view depth/disparity map과 같은 모양에 대한 view specific representation으로 매핑됩니다.    

본 시스템의 주요 구성 요소는 미분 가능한 projection 및 unprojection 작업입니다. 이러한 과정을 통해 시스템을 end-to-end 학습이 가능하면서도 3D geometry를 metrically하게 정확히 주입하였다고 합니다.     

이 LSM에 대한 변형으로 volume occupancy maps을 생성하는 Voxel LSM, 입력 이미지당 depth map(Depth LSM)을 출력하는 것을 제시하였습니다.

1) 2D Image Encoder

2) Differentiable Unprojection

3) Recurrent Grid Fusion

4) 3D grid Reasoning

5) Differentiable Projection

6) Architecture Details

![unprojection](/post_images/Multi-View-stereo-machine/backprojection.PNG "unprojection")


## 4. Experiments
<hr>

## 5. Discussion
<hr>

## 6. summary
<hr>