---
title: shapenet을 통한 학습결과를 category별로 출력하는 방법
author: kju
layout: post
use_math: True
categories:
 - deep learning (vision)
---
## shapenet을 통한 학습진행과정에서 chair, table, ... 처럼 category별로 결과를 출력하기
3D data에 관한 논문의 결과들을 읽어보면 다음과 같은 표를 볼 수 있다.

![table](/post_images/shapenet_category/category_table.PNG "pix2vox_table")
[reference - pix2vox](https://arxiv.org/abs/1901.11153)   
위와 같이 category별로 IoU를 계산해 비교한다.   
2D dataset이라면 label별로 구분을 해서 나누면 됐지만 shapenet의 경우 category와 label이 다르기때문에 다른 방법이 필요하다.

shapenet의 구조에 대해서는 이 포스팅 참고 [shapenet dataset](https://kju01.github.io/deep%20learning%20(vision)/2024/02/27/vision-shapenet.html)

### dataloader 과정에서 받아오기

```
def __getitem__(self, index):  
    # index indicates the model id (model id's are randomly shuffled)
    if self.cur_index_within_batch == self.batch_size:
        self.cur_index_within_batch = 0
        self.cur_n_views = random.randint(self.min_views, self.max_views+1)
    self.cur_index_within_batch += 1
    # the specific images within the chosen model are chosen at random
    filenames = random.choice(self.im_list[index], self.cur_n_views, replace=False)
    imgs = torch.zeros(self.cur_n_views, 3, 128, 128)  
    label = torch.zeros(32,32,32, dtype=torch.long)
    try:
        labeltmp = self.loader_label(os.path.join(self.label_root, self.cat_model_list[index][0], self.cat_model_list[index][1], 'model.binvox'))
        category = self.cat_model_list[index][0]
        label = torch.from_numpy(labeltmp.data.astype('uint8')).long()
        for view in range(self.cur_n_views):
            imgtmp = self.loader_image(os.path.join(self.image_root, self.cat_model_list[index][0], self.cat_model_list[index][1], 'rendering', filenames[view]))                
            if self.transform is not None:
                imgs[view,:,:,:] = self.transform(imgtmp)
            
    except:
        print('PROBLEM WITH LOADING A BATCH')
        pass

    return {'imgs': imgs, 'label': label, 'category': category}

    #return {'imgs': imgs, 'label': label}
```