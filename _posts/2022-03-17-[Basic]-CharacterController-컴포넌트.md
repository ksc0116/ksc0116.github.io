---
layout: post
title:  "[Basic] CharacterController 컴포넌트"
subtitle:   ""
categories: Unity
comments: true
---

<br>

# characterController 

characterController 컴포넌트에 대해 알아보자.

![image](https://user-images.githubusercontent.com/101051124/158762411-e1947781-beb9-4af2-9142-610cf8e1ca32.png)

characterController는 기본적으로 3D 게임에서 사람 형태의 캐릭터 움직임과 관련된 제어를 위해 사용하며 캡슐 모양의 콜라이더가 포함되어 있다.

**Slope Limit** : 올라갈 수 있는 경사 한계 각

**Step Offset** : 설정값보다 낮은 높이의 계단(그 외 오브젝트)을 오를 수 있다.

**Center** : 캡슐 콜라이더 범위의 중심점

**Radius** : 캡슐 콜라이더 범위의 반지름 (x,z)

**Height** : 캡슐 콜라이더 범위의 높이(y)

이제 스크립트를 이용하여 플레이어 캐릭터의 이동을 구현해보자

우선 characterController 클래스의 메소드에서 움직임과 관련된 함수를 봐보자

**Move(이동방향 x 속도 x Time.deltaTime);** : 매개변수로 이동방향,속도,Time.deltaTime등의 세부적인 이동 방법을 설정하면 이동을 수행한다.

**SimpleMove(Vector3 speed);** : 매개변수로 이동 속도를 넣어 호출 하면 된다.

우선 플레이어 캐릭터의 움직임을 담당할 스크립트를 하나 만든다.

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Movement3D : MonoBehaviour
{
    [SerializeField]
    private float moveSpeed = 5.0f;     //이동 속도
    private Vector3 moveDir;               //이동 방향

    private CharacterController characterController;
    void Awake()
    {
        characterController=GetComponent<CharacterController>();
    }
    void Update()
    {
        characterController.Move(moveDir*moveSpeed*Time.deltaTime);
    }
    
    //PlayerController에서 호출하기 위해 public 설정
    public void MoveTo(Vector3 dir)
    {
        moveDir=dir;
    }
}
```

그리고 플레이어를 제어 할 스크립트를 하나 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    private Movement3D movement3D;
    void Start()
    {
        movement3D=GetComponent<Movement3D>();
    }
    void Update()
    {
        float hAxis = Input.GetAxisRaw("Horizontal");
        float vAxis = Input.GetAxisRaw("Vertical");

        movement3D.MoveTo(new Vector3(hAxis, 0 ,vAxis));
    }
}
```

이제 저장하고 프로젝트로 돌아가 Player오브젝트에 두 스크립트들을 추가해준다. 그러면 우리가 입력한대로 플레이어 오브젝트가 움직이는걸 확인 할 수 있다.

하지만 현재까지 작업한걸로는 중력이 적용되지 않는다. 그래서 스크립트에 중력을 적용하는 코드를 넣어줘야한다. Movement3D 스크립트를 다음과 같이 변경하자.

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Movement3D : MonoBehaviour
{
    [SerializeField]
    private float moveSpeed = 5.0f;     //이동 속도
    private float gravity = -9.81f;         //중력 계수
    private Vector3 moveDir;               //이동 방향

    private CharacterController characterController;
    void Awake()
    {
        characterController=GetComponent<CharacterController>();
    }
    void Update()
    {
        //characterController.isGrounded 
        //발 위치의 충돌을 체크해 충돌이 되면 true,충돌이 되지 않으면 false값을 나타내는 변수.
        if (characterController.isGrounded == false)
        {
            moveDir.y += gravity * Time.deltaTime;
        }
        characterController.Move(moveDir*moveSpeed*Time.deltaTime);
    }
    
    //PlayerController에서 호출하기 위해 public 설정
    public void MoveTo(Vector3 dir)
    {
        //앞, 뒤, 좌, 우 움직임은 매개변수로 넘어온 값을 사용하고
        //아래방향은 Update에서 계속 계산 되는 moveDir.y를 사용한다.
        moveDir = new Vector3(dir.x, moveDir.y, dir.z);
    }
}
```

이제 저장하고 게임 플레이를 해보면 플레이어 오브젝트가 높은 곳에 있다가 발판이 없으면 떨어지는걸 확인할 수 있다.

이제 플레이어 점프를 구현해보자 Movement3D 스크립트를 다음과 같이 변경해주자

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Movement3D : MonoBehaviour
{
    [SerializeField]
    private float moveSpeed = 5.0f;     //이동 속도
    [SerializeField]
    private float jumpForce = 3.0f;       //점프하는 힘
    private float gravity = -9.81f;         //중력 계수
    private Vector3 moveDir;               //이동 방향

    private CharacterController characterController;
    void Awake()
    {
        characterController=GetComponent<CharacterController>();
    }
    void Update()
    {
        //characterController.isGrounded 
        //발 위치의 충돌을 체크해 충돌이 되면 true,충돌이 되지 않으면 false값을 나타내는 변수.
        if (characterController.isGrounded == false)
        {
            moveDir.y += gravity * Time.deltaTime;
        }
        characterController.Move(moveDir*moveSpeed*Time.deltaTime);
    }
    
    //PlayerController에서 호출하기 위해 public 설정
    public void MoveTo(Vector3 dir)
    {
        //앞, 뒤, 좌, 우 움직임은 매개변수로 넘어온 값을 사용하고
        //아래방향은 Update에서 계속 계산 되는 moveDir.y를 사용한다.
        moveDir = new Vector3(dir.x, moveDir.y, dir.z);
    }

    //PlayerController에서 호출하기 위해 public 설정
    public void JumpTo()
    {
        if(characterController.isGrounded == true)//플레이어가 바닥에 붙어있으면
        {
            moveDir.y = jumpForce;
        }
    }
}
```

그리고 PlayerController스크립트도 다음과 같이 수정하자.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    private Movement3D movement3D;
    void Awake()
    {
        movement3D=GetComponent<Movement3D>();
    }
    void Update()
    {
        float hAxis = Input.GetAxisRaw("Horizontal");
        float vAxis = Input.GetAxisRaw("Vertical");
        Debug.Log(Input.GetAxisRaw("Horizontal"));
        movement3D.MoveTo(new Vector3(hAxis, 0 ,vAxis));

        if (Input.GetKeyDown(KeyCode.Space))
        {
            movement3D.JumpTo();
        }
    }
}
```

이제 게임 실행을 해보면 스페이스바를 누르면 플레이어가 점프하는걸 볼 수 있다.

이제 경사면 이동과 계단 이동에 대해 살펴보자.

![image](https://user-images.githubusercontent.com/101051124/158762411-e1947781-beb9-4af2-9142-610cf8e1ca32.png)

우선 characterController 컴포넌트에 Slope Limit 프로퍼티의 값을 보면 45로 되어있다. 그렇기 때문에 플레이어는 45도가 넘는 경사면을 오를 수 없다.

![image](https://user-images.githubusercontent.com/101051124/158777330-12bf48bb-2fbb-454b-a541-c40ebb4c18c8.png)

30도인 곳은 올라가지만 50도인곳은 걸어서 올라갈 수 없다.

그리고 CharacterController컴포넌트에서 Step Offset의 값을 보면 0.3이라고 되어 있는데 이건 계단 사이에 높이 차이가 0.3이하여야지 플레이어가 올라갈 수 있다는 뜻이다. 현재 우리 프로젝트의 계단 사이에 높이는 0.4이기 때문에 플레이어가 올라갈 수 없다.

![image](https://user-images.githubusercontent.com/101051124/158777757-d7434f8f-6ee8-45f1-a456-c65acf1ca425.png)

