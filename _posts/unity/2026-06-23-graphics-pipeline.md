---
title: 렌더링 파이프라인
date: 2026-06-23 09:00:00 +0900
categories: [Unity, Graphics]
tags: [graphics, rendering]
---

# 렌더링 파이프라인 (Rendering Pipeline)

렌더링 파이프라인은 **Draw Call 하나가 발생할 때 GPU 내부에서 처리되는 단계별 흐름**이다. 3D 버텍스 데이터를 받아 최종적으로 화면의 픽셀 색상을 결정하기까지의 과정이다.

![Shader Pipeline](https://www.cyanilux.com/tutorials/intro-to-the-shader-pipeline/Pipeline.png){: .bg-gray }
*출처: [Cyanilux - Intro to the Shader Pipeline](https://www.cyanilux.com/tutorials/intro-to-the-shader-pipeline/)*

---

## 메시(Mesh)와 버텍스(Vertex)

화면에 보이는 모든 3D 오브젝트는 **버텍스(Vertex, 점)** 에서 시작한

![Mesh Overview](https://upload.wikimedia.org/wikipedia/commons/6/6d/Mesh_overview.svg)
*출처: [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Mesh_overview.svg) (CC BY-SA 3.0)*

점들이 모여 선이 되고, 선들이 모여 폴리곤이 되고, 폴리곤들이 모여 메시가 된다. **메시는 결국 버텍스로 이루어져 있다.**

---

## 입력 데이터 (GPU에 전달되는 것들)

파이프라인이 시작되기 전, CPU는 아래 데이터를 GPU에 전달한다.

- **Model Space** (Local Space) — 오브젝트 자체 기준 좌표. 원점이 오브젝트 중심
- **World Space** — 씬 전체 기준 좌표. World Transform 행렬로 변환
- **View Space** — 카메라 기준 좌표. View Transform 행렬로 변환
- **Clip Space** — 화면에 보일 영역을 기준으로 자르기 위한 좌표. Projection Transform으로 변환

버텍스 하나에는 아래 데이터가 담겨 있다:

```
Vertex Buffer
└── Vertex 1
│   ├── Position (Attribute)
│   ├── UV (Attribute)
│   ├── Normal (Attribute)
│   └── ID (Attribute)
└── Vertex 2
    ├── Position
    ├── ...
```

---

## Draw Call

CPU가 데이터 준비를 마치면 GPU에 **"이 오브젝트 그려!"** 하고 명령을 내린다. 이 명령 하나하나를 **Draw Call**이라 한다.

Draw Call이 발생하는 순간 GPU가 렌더링 파이프라인을 실행하기 시작한다.

<img width="790" height="329" alt="Image" src="https://github.com/user-attachments/assets/c7e37831-d8ca-47c0-9337-d9a74f7e3b5b" />

씬에 오브젝트가 여러 개면 Draw Call도 여러 번 발생한다:

```
CPU → "큐브 그려!"     → Draw Call 1 → 파이프라인 실행
CPU → "캐릭터 그려!"   → Draw Call 2 → 파이프라인 실행
CPU → "나무 그려!"     → Draw Call 3 → 파이프라인 실행
```

> Draw Call이 많을수록 CPU → GPU 통신 비용이 커져 성능에 영향을 준다. 이를 줄이기 위해 여러 오브젝트를 하나의 Draw Call로 묶는 **배칭(Batching)** 최적화를 사용한다.
> 

---

## 1. Vertex Shader (버텍스 쉐이더)

메시의 **각 버텍스마다 실행**된다.

주요 역할은 버텍스의 **좌표 공간 변환**이다. 아래 순서로 변환이 일어난다

![Coordinate Systems](https://learnopengl.com/img/getting-started/coordinate_systems.png)
*출처: [LearnOpenGL - Coordinate Systems](https://learnopengl.com/Getting-started/Coordinate-Systems)*

- **Model Space** (Local Space) — 오브젝트 자체 기준 좌표. 원점이 오브젝트 중심
- **World Space** — 씬 전체 기준 좌표. World Transform 행렬로 변환
- **View Space** — 카메라 기준 좌표. View Transform 행렬로 변환
- **Clip Space** — 화면에 보일 영역을 기준으로 자르기 위한 좌표. Projection Transform으로 변환

이 세 변환을 하나로 합친 것이 **WVP(World-View-Projection) 행렬**이다.

> Shader Graph에서 Position 포트(Vertex 단계)가 바로 이 Vertex Shader의 출력을 제어하는 곳이다.
> 

---

## 2. Tessellation (테셀레이션) — 선택적 단계

삼각형을 더 작은 삼각형으로 **세분화**해 메시의 디테일을 높이는 단계. 카메라 거리에 따라 디테일을 조절하는 LOD(Level of Detail)에 활용된다.

Hull Shader → Tessellator → Domain Shader 순으로 처리된다.

> Shader Graph에서는 아직 지원하지 않는다.
> 

---

## 3. Geometry Shader (지오메트리 쉐이더) — 선택적 단계

버텍스/삼각형으로부터 **새로운 지오메트리(점, 선, 삼각형)를 생성**할 수 있는 단계. 풀(grass), 파티클 등 구현에 사용되나 성능 비용이 크다.

> Shader Graph에서는 아직 지원하지 않는다.
> 

---

## 4. Rasterisation (래스터화)

**고정 단계** — 프로그래밍 불가, GPU가 자동으로 처리.

Clip Space의 3D 버텍스 좌표를 2D 화면의 **픽셀(Fragment)들로 변환**하는 과정이다.

주요 처리 내용:

- **클리핑** : 화면 밖 영역(절두체 외부)의 폴리곤 제거
- **Face Culling** : 카메라를 등진 뒷면 폴리곤 제거 (Counter Clockwise 기준)
- **Fragment 생성** : 삼각형이 차지하는 픽셀마다 Fragment 생성
- **보간(Interpolation)** : 버텍스 데이터(UV, 색상, 노멀 등)를 Fragment 전체에 보간

---

## 5. Fragment Shader (프래그먼트 쉐이더)

화면의 **각 픽셀(Fragment)마다 실행**된다. DirectX에서는 **Pixel Shader**라고 부른다.

최종 픽셀의 **색상**을 결정하는 단계로, 쉐이더 프로그래밍의 핵심이다.

주요 역할:

- 텍스처 샘플링 (UV를 이용해 텍스처 색상 가져오기)
- 라이팅 계산
- 알파 클리핑 (특정 픽셀 제거)

> "쉐이더(Shader)"라는 이름 자체가 이 단계의 음영(Shading) 계산에서 유래했다.
> 

---

## Material과 Shader의 구분

머테리얼(Material)과 쉐이더(Shader)는 다르다.

유니티에서 **머테리얼**은 텍스처, 컬러, 쉐이더를 모두 포함할 수 있는 컨테이너다. **쉐이더**는 머테리얼의 구성요소 중 하나일 뿐이다.

> Material = { Texture, Color, Shader, ... }
{: .prompt-info }

즉, 머테리얼 ≠ 쉐이더이며, 쉐이더는 머테리얼 안에 속하는 개념이다.

---

## 출처

- [Render Hell – Book I](https://simonschreibt.de/gat/renderhell-book1/) — Simon Schreibt
- [유튜브 강의](https://www.youtube.com/watch?v=r_eatgPFQYg)
- [Coordinate Systems](https://learnopengl.com/Getting-started/Coordinate-Systems) — LearnOpenGL
- [Polygon mesh](https://en.wikipedia.org/wiki/Polygon_mesh) — Wikipedia
