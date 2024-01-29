---
title: 논문리뷰 - vision - Learning a Multi-View Stereo Machine (2017)
subtitle: 3D Vison 생성모델 분야
author: kju
layout: post
categories:
 - deep learning (vision)
---
본 논문은 2D 이미지들을 모아 3D로 복원하는 multi-view base의 모델에 관한 연구이다.    
main idea는 1D data를 2D로 복원하는 backprojection것과 같이 2D data를 3D로 backprojection하는 부분을 설계함으로서 3D reconstruction을 하는 것이다.

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
![model structure](/post_images/Multi-View-stereo-machine/model-structure.PNG "LSM 모델 구조")

## 4. Experiments
<hr>

## 5. Discussion
<hr>

## 6. summary
<hr>