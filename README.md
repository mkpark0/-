## 프로젝트 소개

이 프로젝트는 초음파 화질 개선에 DiConvAE(Dilated Convolutional Autoencoder) 모델을 적용하여 기존의 화질개선 과정이었던 BM3D 사용 또는 여러 필터를 쓰는 방법이 아닌 새로운 방법을 도입해 보았다.

## 프로젝트 구조

- `딥러닝 5조.ipynb`: 주요 코드와 설명이 포함된 ipynb 파일
-  `README.md`: 프로젝트에 대한 개요와 설명을 포함한 파일

## 활용 데이터

Zenodo 초음파 이미지 데이터
모델 학습, 검증 및 테스트에 이용할 데이터, 총 3,832건 분량의 태아 머리 초음파 이미지 – [그림 1]참고

(1) 이미지 영역분할와 관련된 4개의 class를 갖는 PNG이미지 데이터 세트 3,862개

(2) 픽셀 크기 : 959 x 661

데이터 파일은 4가지 태아 평면(시상, 심실, 소뇌, 태아 머리 다양한 이미지)이 각각의 zip폴더 형태로 
세심하게 이루어져 있다. 
각 폴더는 11개의 하위 zip 폴더를 포함하고 이는 데이터 세트에서 지원하는 11가지 형식 중 하나를 나타낸다. – [그림 2] 참고 

![image](https://github.com/mkpark0/-/assets/127065297/3477b121-799a-4e7e-ac07-90db3b403cdd)



## 데이터 전처리

원본이미지와 원본 Label 이미지 Shape:(540,800,3) ,(661,959,3)

Merge하면서 이미지 크기 통일, 시간 감소 위해 (256,256,3) 형태로 reshape

## 모델 학습

- Autoencoder 구조로써 인코더 부분에 Conv2d레이어 4개를 적용

- 활성화 함수는모두 LeLU를 적용

- 모델의 안정성과 학습향상 효과등을 기대해 배치 정규화를 실행하였고, 이로인한 과적합 방지 위해 Dropout 실행

- 디코더 부분도 인코더와 같은 형태이고 마지막에 sigmoid함수를 활성화 함수로 활용하여 값을 반환 하였다.

- 옵티마이저 설정 (Adam, 학습률 0.001)

- 에포크 수 설정 (예: 50 에포크)

- 학습 손실 및 검증 손실 계산

## 손실 함수

다양한 손실 함수를 사용하여 모델을 학습합니다. 사용된 손실 함수는 다음과 같습니다.

- L1: 예측 값과 실제 값 사이의 절대 오차를 합산
- L2: 예측 값과 실제 값 사이의 제곱 오차를 합산
- LM SSIM: 멀티스케일(Multi-scale)에서의 SSIM을 측정하고 환경에 따라 가중치를 합하여 화질을 측정할 수 있도록 만든 지표
- Perceptual Loss: 예측 이미지와 실제 이미지 사이의 차이를 고수준의 특징 공간에서 계산하는 손실 함수. 주로 사전 학습된 신경망(예: VGG 네트워크)의 중간 레이어 출력을 사용하여 특징을 추출

손실 함수의 조합은 다음과 같습니다.
- L1 + MS-SSIM
- L1 + Perceptual Loss
- L2 + Perceptual Loss

## 결과 및 해석

* 예상 결과 : 스페클 노이즈를 Di-Conv-AE-Net으로 개선한 뒤 SegNet에서 더 높은 성능을 보일 것(영역분할 성능이 높을 것)으로 예상


* 실행 결과 : 대조군으로 SegNet을 실행한 결과의 성능이 더 높게 나왔음.


* 예상 원인 : Di-Conv-AE-Net 알고리즘을 적용한 denoised 이미지들은 시각적으로는 훨씬 깔끔한 결과를 보여줬으나, \
SegNet의 결과물을 보면 정답 영역 범위보다 더 많이 영역의 범위를 잡고 있음
    - 픽셀값도 추출해보면 0이나 255로 이루어져있어야 하는데 그 사이 값((255,0,0) 이런식인데 출력해보면 (54,65,23))들로 많이 이루어져 있음
    1. 이는 Di-Conv-AE-Net 알고리즘을 사용하여 스페클 노이즈를 제거하는 과정에서 중요한 세부 정보가 손실되었을 가능성 존재
    2. Denoising 과정에서 이미지의 픽셀 값이 변하면서 SegNet이 훈련된 데이터 분포와 다른 데이터가 입력되었을 가능성 존재
     > SegNet은 특정 데이터 분포에 대해 훈련되기 때문에, denoised 이미지가 원본 이미지의 특성을 잃어버린다면 성능
    

* 예상 해결 방안
1. 시각적 손실 정도 보다 SegNet이 픽셀 단위로 인식 할 수 있는 지표를 활용하여 Denoise의 적당한 강도 설정
2. Di-Conv-AE-Net의 네트워크 구조나 하이퍼파라미터를 조정하여 보다 나은 denoising 성능을 도출



## 사용 방법
1. Google Drive에 데이터셋 업로드
2. `딥러닝 5조.ipynb` 파일을 Colab에서 열기
3. Google Drive를 마운트하고 데이터셋 로드
4. 노트북의 셀을 실행하여 모델 학습 및 평가

## 참고 논문
- A Perceptually Inspired New Blind Image Denoising Method Using L1 and Perceptual Loss
- Removal of speckle noises from ultrasound images using five different deep learning networks
- 이진아, 곽근창. (2023-06-01). 『딥러닝기반 심장 구조 분할을 위한 성능 비교 분석』.
- Proceedings of KIIT Conference, 제주
- 황보민서, 김호중, 전영은, 박제현, 손가현, 이재준, 원동옥. (2023-12-20). 『초음파 영상에서의 태아 머리 영역 분할 문제 해결을 위한 딥러닝 기반의 최적 모델 탐색』. 한국정보과학회 학술발표논문집, 부산.
- Karaoğlu, O., Bilge, H. Ş., & Uluer, I. (2022). Removal of speckle noises from ultrasound images using five different deep learning networks. Engineering Science and Technology, an International Journal, 29, 101030.



