---
title: 그래픽스 파이프라인 (Graphics Pipeline)
date: 2026-06-23 09:00:00 +0900
categories: [Unity, Graphics]
tags: [graphics, pipeline]
---

# 그래픽스 파이프라인 (Graphics Pipeline)
3D 장면을 화면의 2D 이미지로 변환하는 일련의 절차다. Direct3D, OpenGL, Vulkan 같은 그래픽스 API들이 이 파이프라인을 표준화해서, 개발자가 AMD, Nvidia 등 각 하드웨어에 맞는 코드를 따로 짜지 않아도 되게 해준다. 각 단계는 병렬로 실행되며 대부분 GPU에서 처리된다.

---

### 전체 흐름

![그래픽스 파이프라인 전체 흐름](https://github.com/user-attachments/assets/ccf5b036-f926-4495-a3b0-99deaaea454a)

파이프라인은 크게 세 파트로 나눌 수 있다.

1. **Application**
2. **Geometry**
3. **Rasterization**

---

### 1. Application

CPU에서 실행되는 단계다. 사용자 입력, 애니메이션, 충돌 감지 같은 처리가 여기서 일어나고, 수정된 오브젝트 데이터(점, 선, 삼각형 등의 프리미티브)를 다음 단계로 넘긴다.

---

### 2. Geometry

![Geometry 단계](https://github.com/user-attachments/assets/90f65a50-0886-4eb0-be8f-c81dda49a962)

폴리곤과 정점(Vertex)을 다루는 단계다. 크게 다섯 가지 작업으로 구성된다.

- 모델 / 카메라 변환
- 라이팅
- 투영 (Z-buffering)
- 클리핑
- 뷰포트 변환

**좌표 변환**

오브젝트는 자기 기준의 로컬 좌표계(모델 좌표계)로 설계된다. 이걸 씬 전체의 월드 좌표계로 변환하고, 다시 카메라 기준 좌표계로 변환한 다음, 최종적으로 화면 좌표계로 변환한다. 이 과정은 변환 행렬(Transformation Matrix)을 곱하는 방식으로 처리된다. 하나의 오브젝트에서 다르게 변환된 복사본을 여러 개 만들 수도 있는데, 예를 들어 나무 하나로 숲을 만드는 것 — 이를 **인스턴싱(Instancing)** 이라고 한다.

씬에는 오브젝트 외에 카메라도 정의된다. 카메라의 위치와 방향을 기준으로 씬 전체가 변환되며, 이를 카메라 변환(View Transformation)이라고 한다.

**투영 (Projection)**

3D 시야 볼륨을 정규화된 좌표로 변환한다. 원근 투영(Perspective)은 멀리 있을수록 작게 보이는 일반적인 3D 시점이고, 직교 투영(Orthographic)은 거리와 상관없이 크기가 동일해서 기술 도면이나 지도 등에 사용된다.

**클리핑 (Clipping)**

시야 볼륨은 절두체(Frustum) 형태로 정의된다. 절두체 완전히 밖에 있는 프리미티브는 버려지는데 — 이를 **Frustum Culling** 이라고 한다. 일부만 걸쳐 있는 프리미티브는 경계면에서 잘린다.

**라이팅**

씬에 배치된 광원에 따라 각 정점마다 텍스처 게인 계수(gain factor)를 계산한다. 이후 래스터화 단계에서 삼각형 표면 전체에 걸쳐 정점값이 보간된다. 주변광(Ambient Light)은 모든 표면에 적용되는 방향 독립적인 밝기다. 태양과 같은 방향성 광원은 광원 방향 벡터와 표면의 법선 벡터의 내적(Dot Product)으로 조명 효과를 계산한다.

**뷰포트 변환**

최종적으로 화면의 특정 영역(뷰포트)에 이미지를 출력하기 위해 Window-Viewport 변환이 적용된다. 결과 좌표가 출력 장치의 디바이스 좌표가 된다.

현대 GPU에서는 이 단계의 대부분이 **Vertex Shader** 에서 실행된다. DirectX 10부터는 커스텀 Vertex Shader 사용이 필수다.

---

### 3. Rasterization

연속적인 삼각형 면을 이산적인 픽셀 조각(Fragment)으로 변환하는 단계다. 각 Fragment는 프레임버퍼의 픽셀 하나에 대응된다. 겹치는 폴리곤 처리는 **Z-buffer(깊이 버퍼)** 로 판단한다.

**Fragment Shader** 가 각 Fragment마다 실행되며, 조명, 텍스처, 재질 속성에 따라 픽셀의 최종 색상을 결정한다. 반투명이거나 멀티샘플링이 적용된 경우 이미 있는 색상값과 혼합(Blending)된다.

렌더링이 진행되는 동안 화면이 그려지는 과정이 사용자에게 보이지 않도록, 별도의 메모리 영역에서 렌더링을 수행한다. 이미지가 완전히 완성되면 그때 화면에 복사한다 — 이를 **더블 버퍼링(Double Buffering)** 이라고 한다.

---

*출처: [Wikipedia - Graphics pipeline](https://en.wikipedia.org/wiki/Graphics_pipeline)*
