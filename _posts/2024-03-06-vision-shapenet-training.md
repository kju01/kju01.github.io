---
title: shapenet을 통한 학습결과를 category별로 출력하는 방법
author: kju
layout: post
use_math: True
categories:
 - deep learning (vision)
---
### shapenet을 통한 학습진행과정에서 chair, table, ... 처럼 category별로 결과를 출력하기

     
3D data에 관한 논문의 결과들을 읽어보면 다음과 같은 표를 볼 수 있다.

![table](/post_images/shapenet_category/category_table.PNG "pix2vox_table")     
[reference-pix2vox](https://arxiv.org/abs/1901.11153)     

위와 같이 category별로 IoU를 계산해 비교한다.   
2D dataset이라면 label별로 구분을 해서 나누면 됐지만 shapenet의 경우 category와 label이 다르기때문에 다른 방법이 필요하다.

shapenet의 구조에 대해서는 이 포스팅 참고 [shapenet dataset](https://kju01.github.io/deep%20learning%20(vision)/2024/02/27/vision-shapenet.html)

### Shapenet dataloader 구조

```python
class Dataset(data.Dataset):

    def __init__(self, root, transform=None, loader_image=loader_image, loader_label=loader_label, model_portion=[0, 0.8], min_views=1, max_views=5, batch_size=24):
        image_dict = {}        
        image_list = []   
        cat_model_list = []
        im_list = []
        image_root = os.path.join(root, 'ShapeNetRendering')
        label_root = os.path.join(root, 'ShapeNetVox32')
        main_dir_list = os.listdir(image_root)
        for directory in main_dir_list: # loop over model-categories
            image_dict[directory] = {}
            model_list = portion_models(model_portion, os.path.join(image_root,directory))
            print('Directory: ' + directory + ', # of models: ' + str(len(model_list)))
            for subdirectory in model_list: # loop over models
                image_dict[directory][subdirectory] = []        
                cat_model_list.append([directory, subdirectory])
                im_list_cur = []
                sub_dir_list = [f for f in os.listdir(os.path.join(image_root,directory,subdirectory,'rendering')) if is_image_file(f)]
                for filename in sub_dir_list: # loop over image files
                    image_list.append('{}'.format(os.path.join(directory,subdirectory,'rendering',filename)))
                    image_dict[directory][subdirectory].append('{}'.format(filename))
                    im_list_cur.append(filename)
                im_list.append(im_list_cur)
## Simple image folder case:        
#        for filename in os.listdir(root):
#            if is_image_file(filename):
#                images.append('{}'.format(filename))        
        #combined = list(zip(cat_model_list, im_list))
        #random.shuffle(combined)        
        #cat_model_list[:], im_list[:] = zip(*combined)
        
        self.min_views = min_views
        self.max_views = max_views
        #self.cur_idx = 0
        self.image_root = image_root
        self.label_root = label_root
        self.image_list = image_list
        self.image_dict = image_dict
        self.cat_model_list = cat_model_list
        self.im_list = im_list
        self.transform = transform
        self.loader_image = loader_image
        self.loader_label = loader_label
        self.batch_size = batch_size
        self.cur_index_within_batch = self.batch_size
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
            label = torch.from_numpy(labeltmp.data.astype('uint8')).long()
            for view in range(self.cur_n_views):
                imgtmp = self.loader_image(os.path.join(self.image_root, self.cat_model_list[index][0], self.cat_model_list[index][1], 'rendering', filenames[view]))                
                if self.transform is not None:
                    imgs[view,:,:,:] = self.transform(imgtmp)
                
        except:
            print('PROBLEM WITH LOADING A BATCH')
            pass

        return {'imgs': imgs, 'label': label}
```
alex-golts/pytorch-3D-R2N2/dataset.py 내의 코드를 들고 옴. [github](https://github.com/alex-golts/Pytorch-3D-R2N2)

일반적으로 shapenet 데이터를 불러오는 과정에서는 dict구조를 통해 받아오는 경우가 많다. 2D 데이터와 다르게 camera pose, light 등등 추가적인 정보를 사용할 수도 있기 때문에 dict 구조가 편리하다.  

'imgs' key의 경우 일반적인 image data에서 view라는 차원이 추가되어있다. 즉 [view, channel, height, width] 로 4차원 data로 되어있다.     
view의 경우 보통 dataloader과정에서 몇개의 view를 임의로 뽑을지 지정한다.

코드를 보면 'label' 이라는 key가 있으니 이를 이용하면 될 것 같지만 56 line을 보면 알 수 있듯이 (32,32,32) tensor이다. 2D data에서는 한 자리 숫자여서 구별이 가능했지만 shapenet의 label로는 구별이 어렵다. 또한 label의 경우 세부 category에 대한 정보를 담고 있기 때문에 더욱 category 구별이 불가능하다.

### dataload 과정에서 category 정보를 같이 받기. (getitem 함수 수정)

```python
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
```

나는 이러한 문제를 폴더를 탐색하는 과정에서 체크를 하도록 하였다.   
[shapenet dataset](https://kju01.github.io/deep%20learning%20(vision)/2024/02/27/vision-shapenet.html) 포스팅에서 소개했듯이 shapenet의 dataset의 경우 category-세부category 순으로 이루어져있다.      
그러므로 img나 label을 불러오기 위한 폴더 경로의 이름을 이용하였다.    
13 line에서와 같이 category folder의 이름에 해당하는 ```self.cat_model_list[index][0]```를 category로 저장하고 return하는 dict에 이를 추가하였다.   
주의할 점은 저렇게 return한 category는 02691156 처럼 넘버링되어있기 때문에 정확한 category 명으로 바꿔줘야한다.   
category 이름은 다음 링크를 참고하면 된다. [https://gist.github.com/tejaskhot/15ae62827d6e43b91a4b0c5c850c168e](https://gist.github.com/tejaskhot/15ae62827d6e43b91a4b0c5c850c168e)