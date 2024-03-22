---
title: 논문리뷰-vision-Occupancy networks_learning 3D reconstruction in Function space(작성중)
author: kju
layout: post
use_math: True
categories:
 - deep learning (vision)
---
### summary    
 본 논문은 point cloud 기법을 제시하여 이를 이용해 3D reconstruction을 진행하는 방법을 제시한다. 이 방법은 3D 객체의 표면을 나타내는 3차원 좌표(point)를 생성하는 방식으로 복원을 진행한다. 입력 이미지를 인코딩하여 얻어진 특징으로 3D point cloud 후보를 생성하고, 후보 중에서 조건부 샘플링을 통해 3D reconstruction에 적절한 point cloud를 선정한다. 이를 Mesh의 연결구조로 후처리를 하여 최종 3D 객체를 생성한다.

 ## Introduction   

 