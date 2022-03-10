---
layout: post
title:  "[Animation] 애니메이션 이벤트"
subtitle:   ""
categories: Unity
comments: true
---

<br>

게임을 만들다 보면 특정 애니메이션이 플레이 될 때 특정 부분에서 실행이 되었으면 기능들이 있다.

예를들면 플레이어의 움직임에 맞춰 발자국 소리가 난다 또는 공격 애니메이션 적을 가격하는 부분에만 실행되어야 하는 기능들 이때 유용한 것이 애니메이션 이벤트 기능이다.

애니메이션 이벤트를 적용하는 방법에는 2가지가 있다.

첫번째 방법부터 살펴보자.

1)FBX 파일에서 모델과 함께 임포트된 애니메이션에서 애니메이션 이벤트 사용하기

프로젝트뷰에서 임포트한 FBX파일을 선택하면

![image](https://user-images.githubusercontent.com/101051124/157337982-7a91f15a-7c77-4518-9e6f-35272e552dea.png)

인스펙터창이 이렇게 변한다 여기서 우리가 해야 할 일은 맨 위에 Animation탭을 누르고

우리가 애니메이션 이벤트를 만들 수 있게 하단에 Events탭을 눌러서 열어준다. 

![image](https://user-images.githubusercontent.com/101051124/157338269-1bce27ad-1fda-4919-bff2-fdcdede8099b.png)

자 그럼 이제 인스펙터창에서 제일 하단에 있는 내이메이션을 실행시켜서 볼 수 있는 창이 있다.

재생버튼을 눌러서 보면 Events항목에서 타임라인같이 생긴 놈도 같이 움직이는 것을 볼 수 있다.

제일 하단에 있는 곳에서 애니메이션 특정 지점을 선택하는 것이 먼저이다.

![image](https://user-images.githubusercontent.com/101051124/157338941-117a241b-6f72-4c16-97e3-0e2e2ee68cbd.png)

위 사진과 같이 특정 지점을 선택하고 Events항목에서 타임라인 옆에 무언가 추가할 수 있게 버튼이 있다.

![image](https://user-images.githubusercontent.com/101051124/157339097-94ba20d6-775c-40eb-a63f-b0a81969c38b.png)

저 버튼을 눌러보면 원래 내가 선택한 특정 부분에 파란색으로 책갈피 같은게 뜨는데 저게 애니메이션

이벤트이다. 

![image](https://user-images.githubusercontent.com/101051124/157339405-c1bd1ac6-1cde-4166-b1bc-8afc6dad5cad.png)

자 그럼 원래는 비활성화 되어있던 Events항목의 필드들이 활성화가 된다. 

Function필드에는 우리가 사용하고 싶은 함수의 이름(Attack)을 적고

나머지 밑의 필드들은 인스펙터창에서 함수의 매개변수에 특정 값을 넘길 수 있는 필드들이다.

Funtion필드에 우리가 사용 할 함수의 이름을 적었으면 스크립트에 Funtion필드에 적은 함수명과 같은 함수를 하나 만들고 필요한 기능을 구현하면 된다.

두번째 방법도 살펴보자.

2)애니메이션 클립에서 애니메이션 이벤트 사용하기

우선 FBX파일에서 애니메이션 클립을 추출해보자 그러기 위해서는 FBX 파일의 접힌 부분을 확장해서 애니메이션 클립을 선택하고 [Ctrl+D]를 누르면 추출이 된다.

그리고나서 [Ctrl + 6]을 누르면 애니메이션 뷰가 열리고 현재 선택한 애니메이션의 키 값들의 정보가 나온다.

![image](https://user-images.githubusercontent.com/101051124/157341409-2ad757fd-0cc3-4714-8890-224c95cb5f9b.png)

애니메이션 뷰를 보면 1번 방법에서 봤던 거 처럼 AddEvent키가 보이고 저 버튼을 누르면

인스팩터 창이 아래처럼 변하고 나머지는 1번 방법과 같다.

![image](https://user-images.githubusercontent.com/101051124/157341580-7f781be1-97d1-4961-9285-f85445a2af4d.png)

이렇게 편리 할 거 같은 애니메이션 이벤트에도 단점이 있다...

우선 레거시(Legacy)애니메이션을 사용할 때 프레임이 심하게 낮아지거나,메카님 애니메이션에서 트랜지션을 설정할 때 블랜드 되는 과정을 잘못 선정하면 애니메이션 이벤트가 실행되지 않을 수 있다.

그 다음으로는 함수 호출 구조를 파악하는게 어려워진다. 

스크립트에서 애니메이션 이벤트용으로 만든 함수는 참조 0개라는 글자가 보인다.

![image](https://user-images.githubusercontent.com/101051124/157342142-b6d45a58-4f0b-4567-a9cc-b775653e8ea2.png)

즉, 호출이 되는건지 판단하기 어려워진다.

여기서 이번 포스팅을 마치겠다.