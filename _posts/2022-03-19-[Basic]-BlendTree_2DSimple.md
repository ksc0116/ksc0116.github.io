---
layout: post
title:  "[Basic] BlendTree_2DSimple"
subtitle:   ""
categories: Unity
comments: true
---

<br>

BlendTree_2DSimple를 실습해보자.

(1) 애니메이터 뷰로 가서 블랜드 트리를 생성하고 더블클릭 후 내부로 이동

(2) 파라미터스를 2개 생성하고, 이름을 각각 Horizintal, Vertical로 설정

(3) Blend Type을 2D Simple Directional로 설정

(4) Blend Tree의 파라미터에 Horizontal, Vertical 등록

(5) Add Motion으로 5개의 Motion 생성

(6) Asset - unity chan! - Art  - Animations 폴더의 애니메이션을 Motion에 등록

(7) Pos X, Pos Y에 알맞은 값 설정

![image](https://user-images.githubusercontent.com/101051124/159109443-be7ea3ed-9738-488d-95f4-f7be467fa691.png)

이 블랜드 트리를 제어하는 스크립트를 만들어보자.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerController2DSimple : MonoBehaviour
{
    private Animator anim;

    private void Awake()
    {
        anim = GetComponent<Animator>();    
    }

    private void Update()
    {
        float vertical = Input.GetAxis("Vertical");
        float horizontal = Input.GetAxis("Horizontal");

        anim.SetFloat("Horizontal", horizontal);

        anim.SetFloat("Vertical", vertical);

        //이동 속도
        //float moveSpeed=5.0f
        //실제 이동
        //transform.position+=new Vector3(horizontal,0,vertical)*moveSpeed*Time.deltaTime;
    }
}
```

에디터로 돌아가서 플레이어의 컴포넌트로 추가해주고 실행하면 내가 누른 방향키대로 애니메이션이 동작하는 걸 확인가능하다.