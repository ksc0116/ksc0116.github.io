---
layout: post
title:  "[Basic] Animation Layer"
subtitle:   ""
categories: Unity
comments: true
---

<br>

게임을 만들 때 캐릭터가 걸으면서 공격을 한다던가, 걸으면서 총을 장전한다고 치면 하체는 걷기 애니메이션 상체는 공격 혹은 장전 이런식으로 나눠서 애니메이션 재생을 할 때 사용한다.

애니메이터 뷰를 살펴보면

![image](https://user-images.githubusercontent.com/101051124/159106804-96c46a04-a9d2-43d0-a84a-07de9831f3be.png)

기본적으로 Base Layer가 존재하고 현재 여기는 Idle과 Run 애니메이션을 넣어놨다.

여기서 "+" 버튼을 눌러서 레이어를 추가할 수 있다. 그럼 다음과 같이 되는데 

![image](https://user-images.githubusercontent.com/101051124/159106848-0b832722-6a94-432b-9eb1-67c5da692538.png)

이렇게 레이어를 여러개 만들어 부위별로 다른 애니메이션을 재생하거나 같은 부위도 비율에 따라 애니메이션 연산을 통해 혼합하여 사용할 수 있다.

각 레이어는 레이어 이름 옆에 톱니바퀴 모양이 있는데 이 버튼을 누르면 현재 레이어에 전체적인 설정을 할 수 있다.

![image](https://user-images.githubusercontent.com/101051124/159106984-402e3346-3591-49b3-b03a-5150f55c49b9.png)

**Weight** : 가중치를 설정해서 합성하는 정도를 선택

**Mask** : 현재 레이어에서 사용할 Avatar Mask

**Blending** : 앞 순서의 애니메이션 계산에 덮어쓸 것인지(Override), 가산(Additive)할 것인지 선택

**Sync** : 다른 레이어와의 동기화 설정

**IK Pass** : IK(역 운동학) 애니메이션 사용 여부

(참고) 순방향 운동학(FK) 허리를 기준으로 잡았을 때 왼쪽 상체 부위는 손끝이 마지막으로 계산된다.

​			역방향 운동학(IK) 손 끝이 움직이면 그에 맞춰 다른 부위의 위치도 함께 맞추는 방식(계단 오르내리는 애니)

이제 애니메이션 레이어를 활용해보자

우선 Base Layer의 기본 스테이트를 걷기로 해두고 레이러를 추가해서 상체는 다른 움직임을 하도록 해보겠다.

(1) Avatar Mask 생성하고 이름을 UpperBodyAvatar로 설정

(2) 생성한 아바타 마스크의 휴머노이드 탭에서 하체만 연동되지 않도록 설정해준다.

![image](https://user-images.githubusercontent.com/101051124/159108295-22ad034e-8f48-4375-9a9a-5209bf9c8ff5.png)

(3) 애니메이터 뷰에서 새로운 레이어를 생성해주고 이름을 UpperLayer로 바꾼다.

(4) UpperLayer의 설정 버튼을 눌러 가중치(Weight)를 1로 설정하고 Mask에 방금 만든 UpperBodyAvatar를 넣어준다.

(5) UpperLayer의 기본 스테이트를 Wait01(대기 자세)로 해준다.

이제 게임을 실행하면 하체는 걷기 상체는 대기 애니메이션을 실행한다.

![image](https://user-images.githubusercontent.com/101051124/159108441-6fe7f73f-4c02-431f-9833-3e30443e66d9.png)

