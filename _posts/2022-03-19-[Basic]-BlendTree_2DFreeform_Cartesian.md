---
layout: post
title:  "[Basic] BlendTree_2DFreeform_Cartesian"
subtitle:   ""
categories: Unity
comments: true
---

<br>

BlendTree_2DFreeform_Cartesian을 실습해보자.

(1) 애니메이터 뷰로 가서 블랜드 트리를 생성하고 더블클릭 후 내부로 이동

(2) 파라미터스를 2개 생성하고, 이름을 각각 Horizintal, Vertical로 설정

(3) Blend Type을 2D Freeform Cartesian으로 설정

(4) Blend Tree의 파라미터에 Horizontal, Vertical 등록

(5) Add Motion으로 4개의 Motion 생성

(6) Asset - unity chan! - Art  - Animations 폴더의 애니메이션을 Motion에 등록

(7) Pos X, Pos Y에 알맞은 값 설정

![image](https://user-images.githubusercontent.com/101051124/159110670-67473cc3-9905-473e-83bd-08bcf38babd4.png)

이제 이 블랜드 트리를 제어 할 스크립트를 만들자.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerController2DFreeformC : MonoBehaviour
{
    private Animator anim;

    private void Awake()
    {
        anim = GetComponent<Animator>();
    }

    private void Update()
    {
        float horizontal = Input.GetAxis("Horizontal");
        float vertical = Input.GetAxis("Vertical");

        anim.SetFloat("Horizontal",horizontal);

        anim.SetFloat("Vertical", vertical);
    }
}
```

이제 앞으로 가면서 좌 / 우 회전이 자연스럽게 이어진다.