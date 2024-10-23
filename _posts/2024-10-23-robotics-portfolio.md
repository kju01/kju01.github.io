---
title: 로봇기구학 텀프로젝트
author: kju
layout: post
use_math: True
categories: 
 - robotics

---

### 부산대학교 2022년 1학기 로봇기구학 수업 텀프로젝트

![drawing-robot](/images/robot_drawing.gif)

### 수행내용
<hr>

다음과 같은 조건이 있었으며 해결법은 다음과 같았습니다.


#### 로봇팔은 벽면에 one path drawing을 하도록 함.
<hr>
6축 로봇팔을 통해 구현하였습니다. 로봇팔의 구동은 Jacobian을 이용한 forward kin으로 계산하였습니다.
<br>

#### 단 로봇의 다리는 총 4개 이상이여야 함. (multi-legged robot with a serial arm(number of legs >= 4))
<hr>

![parallel](/post_images/로봇기구학텀프/parallel.png)
(출처 : wikipedia)
대표적인 parallel manipulator 구조를 이용하였습니다.

#### 바닥은 특정 파형으로 진동하는 상태
<hr>
로봇 몸통의 밑 부분을 기준(base)으로 하고 진동하는 바닥을 계산하는 방식으로 하였습니다. parallel 모형을 뒤집었다고 생각하면 될 것 같습니다.

#### 디자인을 로봇처럼 보이게 꾸밀것
<hr>

로봇 joint들을 시각화하였으며 몸통을 꾸몄습니다. 조원끼리 빅맥모양이라고 이름을 지었습니다.


#### 모터에 랜덤한 노이즈를 부여하는 제약조건이 있음.(각도에 대한 노이즈만 있음)
<hr>

각도에 증폭계수(amp)를 적용하여 노이즈의 영향을 줄였습니다.    
twist_end값을 hyperparameter로 정의한 epsilon의 값과 비교하여 증폭계수를 조정하였습니다.

### 역할 및 결과
<hr>

3인1조로 진행하였으며 평가 기준은 크게 robot vilusalization, Code optimization, Position error(least drawing error)였습니다.   
저는 이 중에서 Code optimization 파트를 담당하였으며, matlab의 Object를 활용하여 코드를 정리하고 반복된 형태의 계산이나 코드를 최소화 하였습니다. 다른 조원분들도 각자의 파트에서 다 잘 해주셔서 만점을 받았습니다.     
같이 했던 팀5 조원분들께 감사하며 과제를 통해 많은 걸 배우게 해주신 손동훈 교수님께도 감사드립니다.

