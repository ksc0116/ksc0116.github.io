---
layout: post
title:  "[Basic] BlendTree_2DFreeform_Directional"
subtitle:   ""
categories: Unity
comments: true
---

<br>

BlendTree_2DFreeform_Directional을 실습해보자.

(1) 애니메이터 뷰로 가서 블랜드 트리를 생성하고 더블클릭 후 내부로 이동

(2) 파라미터스를 2개 생성하고, 이름을 각각 Horizintal, Vertical로 설정

(3) Blend Type을 2D Freeform Directional로 설정

(4) Blend Tree의 파라미터에 Horizontal, Vertical 등록

(5) Add Motion으로 9개의 Motion 생성

(6) Asset - unity chan! - Art  - Animations 폴더의 애니메이션을 Motion에 등록

(7) Pos X, Pos Y에 알맞은 값 설정

![image](https://user-images.githubusercontent.com/101051124/159110117-39b7f3eb-c933-4c49-88e7-b95227835f60.png)

이 블랜드 트리를 스크립트를 이용해서 제어해보자.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerController2DFreeformD : MonoBehaviour
{
    private Animator anim;
    // private float walkSpeed = 4.0f;
    // private float runSpeed = 8.0f;

    private void Awake()
    {
        anim = GetComponent<Animator>();
    }

    private void Update()
    {
        float horizontal = Input.GetAxis("Horizontal");
        float vertical = Input.GetAxis("Vertical");

        float offset = 0.5f + Input.GetAxis("Sprint") * 0.5f;

        anim.SetFloat("Horizontal",horizontal*offset);

        anim.SetFloat("Vertical",vertical*offset);

        //이동 속도 : Shift키를 안눌렀을 땐 walkSpeed, Shift키를 눌렀을 땐 runSpeed값이 moveSpeed에 저장
        // float moveSpeed = Mathf.Lerp(walkSpeed, runSpeed, Input.GetAxis("Sprint"));
        //실제 이동
        // transform.position+=new Vector3(horizontal,0,vertical)*moveSpeed*Time.deltaTime;
    }
}
```

에디터로 돌아가서 플레이어 오브젝트에 컴포넌트로 추가하고 실행해보면 같은 방향의 애니메이션이더라고 파라미터 값에 따라 재생되는 애니메이션이 다르고 다른 방향의 애니까지 재생이 가능하다.

