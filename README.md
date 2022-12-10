## Unbiased-Teacher-with-Adaptive-EMA

- 2022-2 소프트웨어 캡스톤 디자인
- 지도 교수: 이대호 교수님
- 프로젝트 진행자: 배제언

## 요약

 - 준 지도 학습은 라벨링 되지 않은 많은 양의 데이터를 이용하여 라벨링의 어려움을 해결했을 뿐 아니라 지도 학습 못지않은 우수한 성능을 보였다. 그러나 객체 탐지에서 준 지도 학습은 클래스의 레이블뿐 아니라 바운딩 박스까지 라벨링 해야 하는 어려움 때문에, 이미지 분류보다 비교적 연구가 덜 되었다. 최근 연구들은 선생 모델은 Pseudo-bounding box와 Pseudo-label을 만들고, 학생 모델은 이를 학습하는 방식으로 진행되었다. 준 지도 객체 탐지에서 state-of-the art 알고리즘인 'unbiased-teacher'는 학생-선생 학습 프레임워크에서 EMA를 사용하여, Pseudo-label을 학습한 학생 모델의 가중치를 아주 낮은 비중으로 선생 모델의 가중치와 가중평균함으로써 선생 모델이 따로 학습을 하지 않고 Pseudo-label을 만들 수 있도록 했다. 그러나 본 프로젝트 진행자는 해당 방식에 두 가지 한계점이 있다고 보았다. 첫째, 선생 모델이 학생의 가중치를 사용하기 때문에, 학생 모델이 잘못된 Pseudo-label과 Pseudo-bounding box를 학습할 경우, 학생의 잘못된 가중치를 그대로 사용하여 선생 모델이 잘못된 Pseudo-label을 만드는 가능성이 더욱 증가하는 문제가 있다. 둘째, 학생 모델의 낮은 비중의 고정된 가중치만 가중평균에 사용하기 때문에 선생 모델은 Pseudo-label과 Pseudo-bounding box를 만드는 수준이 학생 모델의 학습 속도를 따라가지 못할 가능성이 있다.이러한 문제점을 해결하기 위해, 본 프로젝트 진행자는 레이블이 있는 데이터에서 학생 모델의 손실에 비례하여 전달받는 학생 모델의 가중치의 비중을 적응적으로 조절하는 EMA 방법을 제안했다. 제안 방법은 Voc2007과 VOC2012에서 앞선 한계점을 개선하여 기존 방법 대비 최대 1AP의 성능향상을 보였다.

## 프로젝트 설명

- 준 지도 객체 탐지는 Unlabeled data에 대해 Pseudo-label을 생성 하고 이를 학습하기 위해 Teacher-Student의 학습 프레임워크를 주로 사용해왔다. 이때 Teacher Model은 Pseudo-label을 생성하는 역할을 하며, Student Modeld은 Pseudo-label을 학습하는 역할을 한다. 'unbiased teacher'는  학습 프레임워크를 아래의 사진과 같이 구성하여 준 지도 객체 탐지에서 State-of-the art를 달성했다. 

![image](https://user-images.githubusercontent.com/70766134/206851220-1bd69e52-a46c-4729-a464-e4535e2713c5.png)
github: https://github.com/facebookresearch/unbiased-teacher  
paper: https://arxiv.org/abs/2102.09480

![image](https://user-images.githubusercontent.com/70766134/206851619-cf17f3f4-d968-49c8-b321-d786e4aef2e0.png)
- 이때, Student model을 update 하는 가중치의 수식은 위와 같이 진행되며, 


![image](https://user-images.githubusercontent.com/70766134/206851620-9033983a-04e9-48f2-96e4-0bf42e9b8b0d.png)
- teacher model의 가중치는 매 iteration 마다 위와 같이 결정 된다. 

- 그러나 본 프로젝트 진행자는 위와 같은 학습 프레임워크는 두 가지 문제점이 있다고 보았다. 
- 첫째, 선생 모델이 학생의 가중치를 사용하기 때문에, 학생 모델이 잘못된 Pseudo-label과 Pseudo-bounding box를 학습할 경우, 학생의 잘못된 가중치를 그대로 사용하여 선생 모델이 잘못된 Pseudo-label을 만드는 가능성이 더욱 증가하는 문제가 있다. 
- 둘째, 학생 모델의 낮은 비중의 고정된 가중치만 가중평균에 사용하기 때문에 선생 모델은 Pseudo-label과 Pseudo-bounding box를 만드는 수준이 학생 모델의 학습 속도를 따라가지 못할 가능성이 있다.
- 따라서 본 프로젝트 진행자는 이러한 문제를 해결하기 위해, Student Model이 올바르게 학습한 경우에 알파를 작게 주고 학습을 잘못한 경우에는 알파를 크게 주어 Student Model의 학습 수준에 맞게 동적으로 설정하는 알파를 제안했다. 올바르게 학습을 한 경우와 올바르게 학습을 하지 않은 경우는 정답이 있는 데이터에 대해 손실을 계산한 Supervised Loss를 기준으로 설정했다. 해당 알파를 동적으로 설정하는 과정의 핵심은 아래와 같다.

![image](https://user-images.githubusercontent.com/70766134/206852196-c7577b21-a4ea-406f-a216-7a84afa2de7f.png)




## 실험 결과

- 데이터셋: Voc2007 + VOC2012

- 사용한 준지도 학습 알고리즘: unbiased-teacher

- 모델: Faster r-cnn with fpn + ResNet-50 backbone

- 갸중치: ImageNet pretrain-weight 사용

- Confidnence Threshold : 0.7

- Batch_size: 4(labeled data) + 4(unlabeled data)

- 기타: 다른 모든 실험 세부사항은 "UNBIASED TEACHER FOR SEMI-SUPERVISED OBJECT DETECTION" 논문에서 사용한 조건을 그대로 사용하였다.  

- 논문과 제안 방법의 성능 비교

|          |      Model     |    Labeled dataset  |     Unlabeled dataset    |     Batchsize       |     AP 50     |    AP     |
|:-------: |:--------------:|:------------------: | :----------------------: | :-----------------: | :-----------: | :-------: |
|  Papers  |    R50-FPN     |      VOC 2007       |        VOC 2012          |        4            |     75.74     |   46.58   |
|  Ours    |    R50-FPN     |      VOC 2007       |        VOC 2012          |        4            |     76.42     |   47.61   |



## 실행 방법
아래 레퍼런스에 있는 https://github.com/facebookresearch/unbiased-teacher에 들어가서 git clone을 한 뒤, unbiased-teacher/ubteacher/engine/trainer.py
를 업로드한 코드로 변경한 뒤 사용하시면 됩니다.  
```
python train_net.py \
      --num-gpus 1 \
      --config configs/voc/voc07_voc12.yaml \
       SOLVER.IMG_PER_BATCH_LABEL 4 SOLVER.IMG_PER_BATCH_UNLABEL 4
```
## 레퍼 런스
- https://github.com/facebookresearch/unbiased-teacher

