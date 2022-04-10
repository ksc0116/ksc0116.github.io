---
layout: post
title:  "[Project] RPG_TEST(WorldMap제작)"
subtitle:   ""
categories: Unity
comments: true
---

<br>

# WorldMap 생성

(1) Asset 파일 중 Stage폴더에서 마을 오브젝트를 하이어라키 창에 넣어준다. 회전(0 / 180 / 0)

(2) 마을 오브젝트의 자식 오브젝트를 선택하고 Mesh에 Material(BaseMat)을 적용한다.

(3) Light오브젝트에서 Color(흰색), ShadowType(Hard Shadows), Strength(0.2)

(4) Windows - Rendering - Light - Environment - Source(Color) - AmbientColor(black)

(5) Camera 오브젝트의 Camera컴포넌트 -> ClearFlags(SolidColor), BackGround(gray)

![image](https://user-images.githubusercontent.com/101051124/162622144-abccde58-1525-4acd-a12d-96ef9b6e24d6.png)

그럼 위와 같은 상태가 된다.

# Player 제작

(1) Asset에서 Player파일에서 Player모델을 하이어라키 창에 드래그한다. 회전(0 / 180 / 0)

(2) Player 오브젝트의 자식 오브젝트를 선택하고 Mesh에 Material(BaseMat)을 적용한다.

(3) MainCamera 오브젝트 위치(0 / 12 / -6), 회전(60 / 0 / 0)

(4) Animation폴더에 있는 Player폴더에서 Idle 애니메이션을 Player 오브젝트에 넣어준다.

(5) Mayor 오브젝트도 Animation폴더에 있는 Mayor 폴더에서 Idle 애니메이션을 Mayor 오브젝트에 넣어준다.

(6) Mayor 오브젝트의 Animator로 가서 Idle 애니메이션의 속도를 0.6으로 바꿔준다.

# NPC 제작

(1) Asset 파일 중 NPC폴더에서 PeopleA 오브젝트를 하이어라키 창에 넣어준다. 회전(0 / 205 / 0), 위치(-5 / 0 / -3.5)

(2) PeopleA 오브젝트의 자식 오브젝트를 선택하고 Mesh에 Material(BaseMat)을 적용한다.

(3) Asset 파일 중 NPC폴더에서 Mayor 오브젝트를 하이어라키 창에 넣어준다. 회전(0 / 180 / 0), 위치(1 / 0 / 7)

(4) Mayor 오브젝트의 자식 오브젝트를 선택하고 Mesh에 Material(BaseMat)을 적용한다.

(5) Asset 파일 중 NPC폴더에서 Collector 오브젝트를 하이어라키 창에 넣어준다. 회전(0 / 140 / 0), 위치(-7 / 0 / 10)

(6) Collector 오브젝트의 자식 오브젝트를 선택하고 Mesh에 Material(BaseMat)을 적용한다.

(5) Asset 파일 중 NPC폴더에서 MerchantA, MerchantB 오브젝트를 하이어라키 창에 넣어준다. 각각 회전(0 / 180/ 0), 위치(14 / 0 / 12.5) / 

회전(0 / 180/ 0), 위치(18.6/ 0 / 15) 

(6) MerchantA, MerchantB 오브젝트의 자식 오브젝트를 선택하고 Mesh에 Material(BaseMat)을 적용한다.

(7) NPC들을 담을 빈 오브젝트(NPC)를 만들고 NPC들을 자식으로 넣어준다.

