---
layout: post
title:  "[Project] FPS Prototype(2)"
subtitle:   ""
categories: Unity
comments: true
---

<br>

# 플레이어 캐릭터 이동

(1) CharacterController컴포넌트를 통해 움직임을 제어하는 MovementCharacterController스크립트를 만든다.

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[RequireComponent(typeof(CharacterController))]
public class MovementCharacterController : MonoBehaviour
{
    [SerializeField]
    private float moveSpeed;        // 이동속도
    private Vector3 moveForce;      // 이동 힘 (x,z와 y축을 별도로 계산해 실제 이동에 적용)

    private CharacterController characterController;        // 플레이어 이동 제어를 위한 컴포넌트

    private void Awake()
    {
        characterController = GetComponent<CharacterController>();
    }

    private void Update()
    {
        // 1초당 moveForce 속력으로 이동
        characterController.Move(moveForce * Time.deltaTime);
    }

    public void MoveTo(Vector3 direction)
    {
        // 이동 방향 = 캐릭터 회전 값 * 방향 값
        direction=transform.rotation*new Vector3(direction.x, 0,direction.z);

        // 이동 힘 = 이동방향 * 속도
        // 카메라 회전으로 전방 방향이 계속 변하기 때문에 회전 값을 곱해서 연산해야 한다.
        moveForce =new Vector3(direction.x*moveSpeed,moveForce.y,direction.z*moveSpeed);
    }
}
```

**[RequireComponent(typeof(CharacterController))]** : 이 명령이 포함된 스크립트를 게임 오브젝트에 컴포넌트로 적용하면 해당 컴포넌트도 같이 추가된다.

**TIP) 위나 아래를 바라보고 이동할 경우 캐릭터가 공중으로 뜨거나 아래로 가라앉으려하기 때문에 direction을 그대로 사용하지 않고, moveForce 변수에 x,z값만 넣어서 사용**

(2) PlyerController스크립트를 수정한다.

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    private RotateToMouse rotateToMouse;        // 마우스 이동으러 카메라 회전
    /*●*/private MovementCharacterController movement;      // 키보드 입력으로 플레이어 이동, 점프
    private void Awake()
    {
        // 마우스 커서를 보이지 않게 설정하고, 현재 위치에 고정 시킨다.
        Cursor.visible = false;
        Cursor.lockState = CursorLockMode.Locked;

        rotateToMouse = GetComponent<RotateToMouse>();
        /*●*/movement=GetComponent<MovementCharacterController>();
    }
    private void Update()
    {
        UpdateRotate();
        UpdateMove();
    }
    private void UpdateRotate()
    {
        float mouseX = Input.GetAxis("Mouse X");
        float mouseY = Input.GetAxis("Mouse Y");

        rotateToMouse.UpdateRotate(mouseX, mouseY);
    }
    /*●*/private void UpdateMove()
    {
        float hAxis = Input.GetAxis("Horizontal");
        float vAxis = Input.GetAxis("Vertical");

        movement.MoveTo(new Vector3(hAxis, 0, vAxis));
    }
}
```

(3) Player오브젝트에 MovementCharacterController스크립트를 추가해준다.

이제 게임을 실행하면 내가 입력한 키보드 값에 따라 플레이어 오브젝트가 움직이는 것을 볼 수 있다.

***

# 플레이어 캐릭터 이동 - 걷기, 뛰기 설정

(1) 이동속도, 체력등의 데이터를 관리하는 Status스크립트를 만든다.

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Status : MonoBehaviour
{
    [Header("Walk, Run Speed")]
    [SerializeField]
    private float walkSpeed;
    [SerializeField]
    private float runSpeed;

    public float WalkSpeed => walkSpeed;
    public float RunSpeed => runSpeed;  
}
```

(2) MovementCharacterController스크립트를 수정한다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[RequireComponent(typeof(CharacterController))]
public class MovementCharacterController : MonoBehaviour
{
    [SerializeField]
    private float moveSpeed;        // 이동속도
    private Vector3 moveForce;      // 이동 힘 (x,z와 y축을 별도로 계산해 실제 이동에 적용)

    /*●*/public float MoveSpeed
    {
        set => moveSpeed = Mathf.Max(0, value);
        get=> moveSpeed;
    }

    private CharacterController characterController;        // 플레이어 이동 제어를 위한 컴포넌트

    private void Awake()
    {
        characterController = GetComponent<CharacterController>();
    }

    private void Update()
    {
        // 1초당 moveForce 속력으로 이동
        characterController.Move(moveForce * Time.deltaTime);
    }

    public void MoveTo(Vector3 direction)
    {
        // 이동 방향 = 캐릭터 회전 값 * 방향 값
        direction=transform.rotation*new Vector3(direction.x, 0,direction.z);

        // 이동 힘 = 이동방향 * 속도
        moveForce = new Vector3(direction.x*moveSpeed,moveForce.y,direction.z*moveSpeed);
    }
}
```

(3) PlayerController스크립트를 수정해준다.

```cs
```

