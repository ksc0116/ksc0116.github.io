---
layout: post
title:  "[Skill] 마우스 줌 인, 줌 아웃"
subtitle:   ""
categories: Unity
comments: true
---

<br>

마우스 줌 인, 줌 아웃 구현하기

======

마우스 줌 인, 줌 아웃을 쉽게 구현하는 방법을 알아보자.

우선, 코드를 먼저 봐보자!

```csharp
 var mouseScroll = Input.mouseScrollDelta;
        Camera.main.fieldOfView = Mathf.Clamp(Camera.main.fieldOfView - mouseScroll.y,30f,70f);
```

보는거와 같이 엄청 간단하다.

여기서 알아가야 할 것들이 있다.

**1)Input.mouseScrollDelta**

mouseScrollDelta는 Vector2.y에 저장이 된다. Vector2.x값은 무시된다. Vector2.y값이 음수면 휠을 올리는거고, 양수면 휠을 내리는 것이다.

**2)Camera.main.fieldOfView**

카메라의 시야각 프로퍼티다. 인스텍터창에서 다음에 위치한다.

![image](https://user-images.githubusercontent.com/101051124/157414661-22208d43-b499-4c4b-bfab-9c06f394ae75.png)

값이 커지면 시야각이 넓어지고 작아지면 시야각이 좁아진다.

**3)Mathf.Clamp()**

Mathf.Clamp(제한을 두고 싶은 변수,최소값,최대값);

위와 같이 사용하면 된다.