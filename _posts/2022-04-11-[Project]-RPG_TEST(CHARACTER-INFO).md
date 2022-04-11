---
layout: post
title:  "[Project] RPG_TEST(CHARACTER INFO)"
subtitle:   ""
categories: Unity
comments: true
---

<br>

착용중인 장비와 캐릭터의 스탯을 보여주는 정보창을 구성해보자.

# 정보창 구성

(1) Canvas의 자식 오브젝트로 Panel(CharacterInfoFrame)을 만든다.

(2) CharacterInfoFrame안에 Image(CharacterInfoWindow)를 만든다. 너비(500), 높이(500) ,SourceImage(Tool_Bag)

(3) CharacterInfoFrame안에 Image(CharacterStatsWindow)를 만든다. 너비(300), 높이(500) ,SourceImage(Tool_Bag), 위치(385 / 0 / 0)

# 스탯창 구성

### Level

(1) CharacterStatsWindow오브젝트안에 TMP(Lev)를 만들고 text(Lev), 너비(115), 높이(50), 색상(주황)

(2) Lev오브젝트 안에 Image(ExpBar_Bg)를 만들어서 경험치 Bar 뒷 배경을 만든다. SourceImage(Bar), 너비(250), 높이(20) , 색상(180 / 180 / 180 / 150)

(3) ExpBar_Bg오브젝트를 복사하고 이름을 ExpBar_Cur로 바꾼다. ImageType(Filled), FillMethod(Horizontal)

(4) Lev오브젝트 안에 TMP(Lev_Cur)를 만들고 알맞게 설정한다.

## H P

(1) CharacterStatsWindow안에 TMP(Hp)를 만들고 알맞게 설정한다.

(2) Hp오브젝트안에 현재 Hp를 출력 할 TMP(Hp_Cur)를 만들고 알맞게 설정한다.

## ATK

(1) Hp를 복사하여 Atk를 만들어준다.

## DEF

(1) Hp를 복사하여 Def를 만들어준다.

## CRI

(1) Hp를 복사하여 Cri를 만들어준다.

![image](https://user-images.githubusercontent.com/101051124/162667633-4d1a84e6-e74c-4456-afb5-5d8b7f169660.png)

![image](https://user-images.githubusercontent.com/101051124/162667673-cdde5dea-2edd-48c4-8acb-d7c35e6e35b0.png)

여태 잘 따라했으면 위와 같은 상황이 된다.

# 캐릭터 정보창 구성

## Head

(1) 착용한 장비를 보여 줄 Image(Slot_Head)를 CharacterInfoWindow안에 생성 SourceImage(Tool_Slot)

(2) 부위를 표시해 줄 TMP(Head)를 만들고 알맞게 설정

## Body

(1) Slot_Head을 복사해서 Body도 만들어준다.

## Arm

(1) Slot_Head을 복사해서 Arm도 만들어준다.

![image](https://user-images.githubusercontent.com/101051124/162669404-4ba7b3b4-4821-4c00-bc4c-b94777fa0ab7.png)

![image](https://user-images.githubusercontent.com/101051124/162669458-198e3741-28d0-42e2-bbb8-7afd136f15cf.png)

위와 같은 상황이 되어야 한다.

# 캐릭터 모습 나타나게 하기

(1) CharacterInfoFrame오브젝트 안에 RawImage(CharView)를 생성한다. 너비(500), 높이(500)

**TIP)** RawImage는 실시간 렌더링에 사용하는 Render Texture, 조작, 변형 없이 이미지를 사용하는 경우에 사용한다. Render Texture는 씬에서 카메라가 렌더링 하는 이미지를 보여주는 텍스쳐이다.실시간 렌더링에 사용된다.

(2) Asset폴더에 Texture폴더를 만들고 우클릭 - Create - RenderTexture를 눌러 RenderTecture(Cam_CharView)를 만든다.

(3) CharView오브젝트에 Cam_CharView텍스쳐를 넣어준다.

(4) Player오브젝트 안에 Camera(Cam_CharView)를 만들어 준다. 반드시 AudioListener를 삭제해야한다.

(5) Cam_CharView오브젝트에 TargetTextuer변수에 Cam_CharView텍스쳐를 넣어준다.

(6) Camera오브젝트: 위치(0 / 3.3 / 5), 회전(12 / 180 / 0)

![image](https://user-images.githubusercontent.com/101051124/162670804-f81517a3-7b96-4b5c-a17e-8329a1c27ce1.png)

그럼 위와 같은 상황이 된다.

(7) CharacterInfoFrame오브젝트의 자식 오브젝트에 있는 CharView오브젝트를 자식 중 맨 위로 올리면 다음과 같은 모습이 된다.

![image](https://user-images.githubusercontent.com/101051124/162671189-0dc921d0-3466-4ab9-96b1-0436ed7684e3.png)

![image](https://user-images.githubusercontent.com/101051124/162671258-481281be-f59c-4116-92a5-366ff5bcdec1.png)

(8) 프레임에 이름을 붙여 줄 TMP(Name_Frame)를 CharacterInfoFrame의 자식으로 만들고 알맞게 설정한다.

![image](https://user-images.githubusercontent.com/101051124/162671682-bda1a85b-b8b4-487e-b0e6-7e069c50ca65.png)

(9) 최종적으로 위치를 조정해준다.

![image](https://user-images.githubusercontent.com/101051124/162671886-105d8c44-19a5-4d7c-a556-f1ac448ed4d1.png)

(10) 정보창에 하늘이 보이니까 Cam_CharView오브젝트로 가서 Camera컴포넌트의 Clear Flags를 SolidColor로 바꿔준다.

이제 게임을 실행해보면 플레이어의 모습도 보이면서 움직이는 모습까지 그려지는 것을 볼 수 있다. 이건 RawImage에 RenderTexture를 사용했기 때문에 가능하다.