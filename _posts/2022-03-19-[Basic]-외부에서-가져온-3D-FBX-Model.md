---
layout: post
title:  "[Basic] 외부에서 가져온 3D FBX Model"
subtitle:   ""
categories: Unity
comments: true
---

<br>

외부에서 가져온 3D FBX Model의 정보를 봐보자.

![image](https://user-images.githubusercontent.com/101051124/159121676-864f9270-8c6d-4e04-a368-211d0685d9a8.png)

4개의 탭이 보이는데 하나하나 살펴보자.

**Model** : 외부에서 가져온 FBX 모델의 기본적인 정보

**Rig** : FBX 모델의 리깅 정보

**Animation** : FBX 모델의 애니메이션 정보

**Materials** : FBX 모델에 적용되는 재질(material) 정보

# Model

## Scene 

광원과 카메라 임포트 여부, 모델 크기 등 씬과 관련된 옵션 설정

### Scale Factor

원본 모델의 크기 설정 (Unity Scale 1일 때 크기)

### Import BlendShapes 

블렌드 셰이프를 메시와 함께 Import



## Meshes

메시 압축, 최적화 등 메시와 관련된 옵션 설정

### Mesh Compression

메시 파일 크기를 줄일 압축 레벨 설정

### Generate Colliders

메시 콜라이더를 사용할지 여부 (충돌 처리)



## Geometry

UV와 노말 처리 등을 위한 지오메트리 관련 옵션 설정

### Normals

노말 벡터를 계산할지, 어떻게 계산할지 설정

# Rigig

모델의 뼈대를 만들어 심거나 뼈대를 할당하여, 모델이 애니메이션 될 수 있는 상태로 만드는 것

![image](https://user-images.githubusercontent.com/101051124/159121970-80d7aa66-785f-430f-889b-a9a0813dfbad.png)

## Animation Type

### None

애니메이션이 없을 때 (지형, 건물, 아이템 등)

### Legacy

유니티 3.x 이전 버전에서 사용 (Animation 컴포넌트)

### Generic

정점 애니메이션 (Animator 컴포넌트)

![image](https://user-images.githubusercontent.com/101051124/159122081-07011f08-32dd-48c9-8fdc-bac54c0803e2.png)

**Avatar Definition** :아바타 정의를 가져올 위치 선택

**Root node** :아바타의 루트 노드로 사용할 뼈대 선택

**Skin Weight** : 하나의 버텍스에 영향을 줄 수 있는 최대 뼈대 수

**Optimize Game Object** : 게임 오브젝트의 Transform 계층 구조를 Avatar와 Animator 컴포넌트에서 제거(성능 향상)

### Humanoid

본 애니메이션 (Ani,ator 컴포넌트)

![image](https://user-images.githubusercontent.com/101051124/159122153-78a59efa-7dbb-4d9a-a60d-c433cda4720f.png)

**Avatar Definition** :아바타 정의를 가져올 위치 선택

**Skin Weight** : 하나의 버텍스에 영향을 줄 수 있는 최대 뼈대 수

**Optimize Game Object** : 게임 오브젝트의 Transform 계층 구조를 Avatar와 Animator 컴포넌트에서 제거(성능 향상)

### 참고 Optimize Game Object

![image](https://user-images.githubusercontent.com/101051124/159122307-dfacd095-07c1-4902-b557-3445dc61730a.png)

**Extra Transform to Expose** : Optimize Game Object에 체크 되어있을 때만 활성화 되는 메뉴(체크되는  Transform은 게임오브젝트의 자식 오브젝트로 출력 된다)

예를들어 오른손에 무기를 쥐어 캐릭터가 애니메이션 할 때 무기가 오른손 위치를 따라다니게 하려면 오른손에 해당하는 Transform은 최적화 하지 않고 남겨 둔다.

# Animation

Animation은 모델링의 애니메이션을 설정하는 탭이다.

![image](https://user-images.githubusercontent.com/101051124/159122530-465475c5-ec08-43fb-95c6-8305c4dd137a.png)

위 사진은 애니메이션이 없을 때의 모습이다.

애니메이션이 있을 때의 모습은 아래와 같다.

![image](https://user-images.githubusercontent.com/101051124/159122670-241bac12-18fd-48ff-985b-6572618e1b8d.png)

# Materials

Materials탭은 Material과 Texture 접근 방법을 정의할 수 있으며, 모델에 사용되는 Material을 설정할수도 있다.

![image](https://user-images.githubusercontent.com/101051124/159122793-7a13c72d-2adf-4079-90cd-879c25b1ddca.png)

**Location** : Materials 및 텍스처 접근 방법 저의

-Use External Materials (Legacy)는 2017.1 이하 버전에서 사용

-Use Embedded materials는 2017.2 버전부터 기본 옵션

