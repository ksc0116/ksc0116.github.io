---
layout: post
title:  "[Basic] BlendTree_1D"
subtitle:   ""
categories: Unity
comments: true
---

<br>

 BlendTree_1D를 실습해보자.

(1) 애니메이터 뷰로 가서 블랜드 트리를 생성하고 더블클릭 후 내부로 이동

(2) 파라미터스에 생성되어 있는 파라미터 이름을 moveSpeed로 설정

(3) Blend Type을 1D로 설정

(4) Blend Tree의 파라미터에 moveSpeed 등록

(5) Add Motion으로 3개의 Motion 생성

(6) Asset - unity chan! - Art  - Animations 폴더의 애니메이션을 Motion에 등록

![image](https://user-images.githubusercontent.com/101051124/159108725-6750d1f6-0294-4d75-8b74-67ecc1788d7f.png)

위의 과정을 그대로 하면 위와 같이 된다. 이제 moveSpeed값에 따라 애니메이션이 혼합되어 재생된다. 이제 moveSpeed값을 코드에서 변화시켜보자.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerController1D : MonoBehaviour
{
    private Animator anim;
    //private float walkSpeed=4.0f;
    //private float runSpeed=8.0f;

    private void Awake()
    {
        anim = GetComponent<Animator>();
    }

    private void Update()
    {
        float vertical = Input.GetAxis("Vertical");

       //Shift키를 안누르면 최대 0.5, Shift키를 누르면 최대 1까지 값이 바뀌게 된다
        float offset = 0.5f + Input.GetAxis("Sprint") * 0.5f;

        //오른쪽 방향키를 누르면 forward가 +지만 왼쪽 방향키를 누르면 forward가 -이기 때문에
        //애니메이션 파라미터를 설정할 땐 절대값으로 적용한다
        float moveParameter=Mathf.Abs(vertical*offset);

        //moveParameter 값에 따라 애니메이션 재생 (0: 대기, 0.5: 걷기, 1: 뛰기)
        anim.SetFloat("moveSpeed",moveParameter);

        //이동 속도 : Shift키를 안눌렀을 땐 walkSpeed, Shift키를 눌렀을 땐 runSpeed값이 moveSpeed에 저장
        //flaot moveSpeed = Mathf.Lerp(walkSpeed, runSpeed, Input.GetAxis("Sprint"));
        //실제 이동
        //transform.position += new Vector3(vertical, 0, 0) * moveSpeed * Time.deltaTime;
    }
}
```

코드해석은 어려울 게 없다. 위와 같이 작성하고 에디터로 돌아가 플레이어 컴포넌트에 추가하고 실행시키면 방향키를 누르면 걷고, 방향키를 누른 상태로 쉬프트키를 누르면 뛴다.

