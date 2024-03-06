---
title: shapenet dataset
author: kju
layout: post
use_math: True
categories:
 - deep learning (vision)
---
## shapenet data의 구조를 알아보자

보통 shapenet 데이터를 받게 되면 구조가 다음과 같다.

```
Shapenet
|--ShapeNetRendering
    |--category (chair, table, ... 등등)
        |--세부 category (둥근 테이블, 네모난 테이블, ... 등등)
            |--rendering
                |--각 viewpoint에서의 이미지들
                |--rendering_metadata.txt
                |--renderings.txt
|--ShapeNetVox32
    |--category (chair, table, ... 등등)
        |--세부 category (둥근 테이블, 네모난 테이블, ... 등등)
            |--model.binvox
|--rendering_only.tgz
```

하나하나 자세히 알아보자.

### shapenet

![shapenet](/post_images/shapenet/folder1.PNG "shapenet")

### category

![category](/post_images/shapenet/renderingfolder.PNG "category")

image data에서는 label이 1이면 cat, 2이면 dog라고 하듯이 label안에 사진이 어떤 category인지 같이 표현되었으나, shapenet에서는 이것이 분리되어 있다.

category의 경우 사진과 같이 넘버링이 되어 있으며 이런 넘버링이 어떤 카테고리인지는 이 링크[https://gist.github.com/tejaskhot/15ae62827d6e43b91a4b0c5c850c168e](https://gist.github.com/tejaskhot/15ae62827d6e43b91a4b0c5c850c168e)에 가면 자세히 나와있다.
예를 들어 02691156은 airplane이라는 category를 가진 데이터들이 들어있는 폴더이다.

![세부 category](/post_images/shapenet/renderingfolder2.PNG "세부category")

예시로 02691156폴더에 들어가면 위와 같은 폴더명으로 된 폴더들이 들어있다. 이 폴더들은 같은 category이지만 다른 모양을 가진(ex) 둥근 table과 네모난 table)
이 폴더부터는 ShapeNetRendering 폴더와 ShapeNetVox32와 다른 파일이 들어있다.

### ShapeNetRendering

![rendering1](/post_images/shapenet/renderingfolder4.PNG "rendering1")
![rendering2](/post_images/shapenet/renderingfolder4_1.PNG " rendering2")

rendering폴더의 경우 위와 같이 여러 viewpoint에서의 2D이미지가 들어있다.

### ShapeNetVox32

![Voxel](/post_images/shapenet/voxelfolder3.PNG "Voxel")
Voxel폴더의 경우 위와 같은 binvox파일하나만 들어있다. 이는 3D data를 32X32X32크기의 tensor에 저장한 파일이다. 이 파일이 label의 역할을 한다.