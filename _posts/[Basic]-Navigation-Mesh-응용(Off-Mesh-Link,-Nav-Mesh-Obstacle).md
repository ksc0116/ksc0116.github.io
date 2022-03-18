---
layout: post
title:  "[Basic] Navigation Mesh 응용(Off Mesh Link, Nav Mesh Obstacle)"
subtitle:   ""
categories: Unity
comments: true
---

<br>

Navigation Mesh를 응용해보자. 사다리, 절벽과 같이 메시가 끊어진 곳을 이동하는 Off Mesh Link와

이동가능한 장애물을 설정하는 Nav Mesh Obstacle을 사용해보자.

프로젝트는 Navigation Mesh 기초에서 했던 것을 이어서 하겠다.

# Off Mesh Link

Off Mesh Link는 사다리, 암벽과 같이 수직으로 올라가거나 내려오는 길, 절벽 사이를 뛰어서 넘어가거나 낭떠러지 알래로 떨어지는 길과 같이 메시가 끊어져 있는 곳을 이동할 수 있게 설정하는 것이다.

## 자동(Auto) Off Mesh Link 설정 방법

(1) 자동으로 Off Mesh Link를 설정할 오브젝트들을 선택 (단일 or 복수)

(2) Navigation View - Object 탭의 "Generate OffMeshLinks" 체크

![image](https://user-images.githubusercontent.com/101051124/158814252-db85fb22-b104-4197-ab8e-0b4f835b3b3f.png)

(3) Navigation View - Bake 탭의 낙하 높이(Drop Height). 점프 거리(Jump Distance)를 설정하고, "Bake"를 눌러 데이터 저장

![image](https://user-images.githubusercontent.com/101051124/158814514-718a3306-08c4-41bd-921c-20b033aae99f.png)

그럼 다음과 같이 점프 거리가 4이하이고 낙하 높이가 4이하인 곳에 Off Mesh Link가 생성되는 걸 볼 수 있다.

![image](https://user-images.githubusercontent.com/101051124/158815867-38987e21-6c6f-4d5e-8465-e014a38aa541.png)

이제 게임 플레이를 해보면 원래는 갈 수 없고 뛰어내리지 못했던 곳을 플레이어가 이동하고 뛰어내린다.

![image](https://user-images.githubusercontent.com/101051124/158816422-a10586aa-d030-4c9e-b1e5-b23e718a6186.png)

위와 같이 점프를 하며

![image](https://user-images.githubusercontent.com/101051124/158816566-d71b57dc-72af-46ff-a5d1-45589a2e209a.png)

떨어지는 걸 볼 수 있다.

### 장점

게임월드에 배치된 많은 오브젝트의 Off Mesh Link를 한꺼번에 설정 가능

### 단점

-낙하 높이(Drop Height)와 점프 거리(Jump Distance)를 하나만 설정할 수 있기 때문에 다양한 지형을 세세하게 설정하는 것이 불가능(Off Mesh Link 데이터 소실의 위험)

-위로 올라가는 Off Mesh Link 설정 불가능

## 수동(Manual) Off Mesh Link 설정 방법

(1) 연결되는 두 지점으로 사용할 오브젝트 생성 및 배치

![image-20220317221939185](C:\Users\ksc52\AppData\Roaming\Typora\typora-user-images\image-20220317221939185.png)

사다리를 오르고 내리는걸 구현하기 위해 사다리 오브젝트의 자식 오브젝트로 스피어 오브젝트 2개를 생성하고 이름을 각각 Start,End로 바꾸고 Size를 0.4로 바꾼다. 그리고 Start의 위치를 (0, -1.4, -0.8)로설정한다. 그리고 End의 위치는 (0, 1.6, 0.8)로 설정한다.

![image](https://user-images.githubusercontent.com/101051124/158817664-c8a38bed-6423-4057-84ed-56f4ed348e45.png)

여기서 주의해야 할 점이 게임 오브젝트의 위치가 네비게이션 메시의 이동 경로 내에 포함되어야 정상적인 이동이 가능하다는 것이다. 이렇게 배치가 되었으면 Transform을 제외한 나머지 컴포넌트들을 끄거나 삭제한다.

(2) 사다리를 구성하고 있는 레더 오브젝트에 Off Mesh Link 컴포넌트를 생성하고, "Start", "End" 변수에 연결되는 두 지점을 설정한다.

![image](https://user-images.githubusercontent.com/101051124/158818536-fb186ab9-2bbd-4069-84bf-fd586b7e17ac.png)

그리고 Bidirectional을 체크하면 양방향으로 이동이 가능하지만, 체크가 안되어 있으면 Start->End로 단일방향으로만 이동할 수 있다. 이제 게임 실행을 해보면 다음과 같이 사다리를 타는 걸 볼 수 있다.

![image](https://user-images.githubusercontent.com/101051124/158818943-97d159e9-f3be-418d-8824-60b3862f910b.png)

### 장점

-지형에 따라 세세한 설정이 가능

-사다리/암벽과 같이 위로 올라가는 Off Mesh Link 설정 가능

### 단점

-Off Mesh Link로 연결이 필요한 모든 부분을 직접 설정해야 함

# 구역 설정

구역을 설정하는 것은 게임 월드에 다양한 지형이 존재할 때 어느 지형을 가로질러 가는 것이 더 빠르고 느린지 명시하기 위함이다.

구역은 Navigation View - Areas탭에서 설정 할 수 있다.

![image](https://user-images.githubusercontent.com/101051124/158819801-a8ccf9ab-b96a-4347-94c0-78cd45b15a2e.png)

다음과 같이 User3에 climb이라는 새로운 구역을 생성하고 Cost를 5로 설정한다.

## Navigation Areas에 등록된 구역을 게임오브젝트에 설정하는 방법

(1)  수동(Manual) Off Mesh Link 오브젝트는 Off Mesh Link 컴포넌트의 Navigation Area 변수 설정.

ex) 수동으로 오프 메시 링크를 설정했던 레더 오브젝트의 오프 메시 링크 컴포넌트를 다음과 같이 수정한다.



![image](https://user-images.githubusercontent.com/101051124/158820673-ac113360-c32e-4f50-a671-a9ae17f09018.png)

(2)  일반 게임오브젝트는 게임 오브젝트 선택 후 Navigation View - Object탭에서 Navigation Area 변수 설정

ex)

![image](https://user-images.githubusercontent.com/101051124/158821174-8e36cde5-e348-4f20-8549-c71a33318de7.png)

여기서 Tip) NavMeshAgent 컴포넌트를 사용하는 이동 에이전트의 Area Mask에서 이동 가능 / 불가능 구역 선택이 가능하다.

ex) Player 오브젝트를 봐보자.

![image](https://user-images.githubusercontent.com/101051124/158821749-aa6adbca-1fc0-4101-817b-9949cbcd864b.png)

저기서 체크가 되어 있는 구역은 이동이 가능하며, 체크가 해제되면 이동 불가능 구역이 된다.

이제 Off Mesh Link로 설정된 곳을 지나갈 때 코드를 이용해 애니메이션을 재생하거나 이동 방법을 선택할 수 있다.

**(TIP) NavMeshAgent 컴포넌트의 Auto Travers Off Mesh Link가 체크 해제되어 있으면 Off Mesh Link를 만나면 오브젝트가 멈추게 된다.**

![image](https://user-images.githubusercontent.com/101051124/159007802-d77684d2-7059-402f-adf9-27dc05d1df92.png)

이제 스크립트를 통해 제어를 해보자. 