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
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    /*●*/[Header("Input KeyCodes")]
    /*●*/[SerializeField]
    /*●*/private KeyCode keyCodeRun = KeyCode.LeftShift;            // 달리기 키

    private RotateToMouse rotateToMouse;        // 마우스 이동으러 카메라 회전
    private MovementCharacterController movement;      // 키보드 입력으로 플레이어 이동, 점프
    /*●*/private Status status;      // 이동속도 등의 플레이어 정보
    private void Awake()
    {
        // 마우스 커서를 보이지 않게 설정하고, 현재 위치에 고정 시킨다.
        Cursor.visible = false;
        Cursor.lockState = CursorLockMode.Locked;

        rotateToMouse = GetComponent<RotateToMouse>();
        movement=GetComponent<MovementCharacterController>();
        /*●*/status=GetComponent<Status>();
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
    private void UpdateMove()
    {
        float hAxis = Input.GetAxis("Horizontal");
        float vAxis = Input.GetAxis("Vertical");

        // 이동중 일 때 (걷기 or 뛰기)
        /*●*/if(hAxis != 0 || vAxis != 0)
        {
            bool isRun = false;

            // 옆이나 뒤로 이동할 때는 달릴 수 없다.
            if(vAxis>0) isRun=Input.GetKey(keyCodeRun);

            movement.MoveSpeed = isRun ? status.RunSpeed : status.WalkSpeed;
        }

        movement.MoveTo(new Vector3(hAxis, 0, vAxis));
    }
}
```

이제 게임을 실행하면 앞으로 가는중 쉬프트를 누르면 달리는 모습을 볼 수 있다.

***

# 플레이어 캐릭터 이동 - 애니메이션, 사운드 재생

(1) WeaponAssaultRifle 애니메이터 뷰로 가서 기존에 등록한 Idle을 삭제하고 대기, 걷기, 뛰기를 한번에 관리하는 BlendTree(Movement)를 만든다.

(2) take_out_weapon에서 Movement로 이어지는 트랜지션 생성하고 같이 만들어진 파라미터의 이름을 movementSpeed로 바꿔준다.

(3) 블랜드 트리 내부로 이동해 Add Motion으로 3개의 모션을 만든다.

(4) 3개의 애니메이션 Idle, walk, run을 순서대로 넣어준다.

![image-20220331091319404](C:\Users\ksc52\AppData\Roaming\Typora\typora-user-images\image-20220331091319404.png)

(5) 애니메이터에 접근해 애니메이션을 제어하는 PlayerAnimatorController스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerAnimatorController : MonoBehaviour
{
    private Animator anim;
    private void Awake()
    {
        // "Player" 오브젝트 기준으로 자식 오브젝트인 
        // "arms_assault_rifle_01" 오브젝트에 Animator 컴포넌트 있음
        anim=GetComponentInChildren<Animator>();
    }

    public float MoveSpeed
    {
        set=>anim.SetFloat("movementSpeed",value);
        get => anim.GetFloat("movementSpeed");
    }
}
```

(6) PlayerController스크립트를 수정한다.

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    [Header("Input KeyCodes")]
    [SerializeField]
    private KeyCode keyCodeRun = KeyCode.LeftShift;            // 달리기 키

    private RotateToMouse rotateToMouse;        // 마우스 이동으러 카메라 회전
    private MovementCharacterController movement;      // 키보드 입력으로 플레이어 이동, 점프
    private Status status;      // 이동속도 등의 플레이어 정보
    /*●*/private PlayerAnimatorController anim;      // 애니메이션 재생 제어
    private void Awake()
    {
        // 마우스 커서를 보이지 않게 설정하고, 현재 위치에 고정 시킨다.
        Cursor.visible = false;
        Cursor.lockState = CursorLockMode.Locked;

        rotateToMouse = GetComponent<RotateToMouse>();
        movement=GetComponent<MovementCharacterController>();
        status=GetComponent<Status>();
        /*●*/anim=GetComponent<PlayerAnimatorController>();
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
    private void UpdateMove()
    {
        float hAxis = Input.GetAxis("Horizontal");
        float vAxis = Input.GetAxis("Vertical");

        // 이동중 일 때 (걷기 or 뛰기)
        if(hAxis != 0 || vAxis != 0)
        {
            bool isRun = false;

            // 옆이나 뒤로 이동할 때는 달릴 수 없다.
            if(vAxis>0) isRun=Input.GetKey(keyCodeRun);

            movement.MoveSpeed = isRun ? status.RunSpeed : status.WalkSpeed;
            /*●*/anim.MoveSpeed = isRun ? 1.0f : 0.5f;
        }
        else
        {
            /*●*/movement.MoveSpeed = 0;
            /*●*/anim.MoveSpeed = 0;
        }

        movement.MoveTo(new Vector3(hAxis, 0, vAxis));
    }
}
```

(7) Player오브젝트에 PlayerAnimatorController 스크립트를 넣어준다.

(8) PlayerController스크립트에 사운드 재생을 위한 코드를 넣어준다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    [Header("Input KeyCodes")]
    [SerializeField]
    private KeyCode keyCodeRun = KeyCode.LeftShift;            // 달리기 키


    /*●*/[Header("Audio Clips")]
    /*●*/[SerializeField]
    /*●*/private AudioClip audioClipWalk;           // 걷기 사운드
    /*●*/[SerializeField]
    /*●*/private AudioClip audioClipRun;            // 달리기 사운드

    private RotateToMouse rotateToMouse;        // 마우스 이동으러 카메라 회전
    private MovementCharacterController movement;      // 키보드 입력으로 플레이어 이동, 점프
    private Status status;      // 이동속도 등의 플레이어 정보
    private PlayerAnimatorController anim;      // 애니메이션 재생 제어
    /*●*/private AudioSource audioSource;        // 사운드 재생 제어

    private void Awake()
    {
        // 마우스 커서를 보이지 않게 설정하고, 현재 위치에 고정 시킨다.
        Cursor.visible = false;
        Cursor.lockState = CursorLockMode.Locked;

        rotateToMouse = GetComponent<RotateToMouse>();
        movement=GetComponent<MovementCharacterController>();
        status=GetComponent<Status>();
        anim=GetComponent<PlayerAnimatorController>();
        /*●*/audioSource=GetComponent<AudioSource>();
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
    private void UpdateMove()
    {
        float hAxis = Input.GetAxis("Horizontal");
        float vAxis = Input.GetAxis("Vertical");

        // 이동중 일 때 (걷기 or 뛰기)
        if(hAxis != 0 || vAxis != 0)
        {
            bool isRun = false;

            // 옆이나 뒤로 이동할 때는 달릴 수 없다.
            if(vAxis>0) isRun=Input.GetKey(keyCodeRun);

            movement.MoveSpeed = isRun ? status.RunSpeed : status.WalkSpeed;
            anim.MoveSpeed = isRun ? 1.0f : 0.5f;
            /*●*/audioSource.clip = isRun ? audioClipRun : audioClipWalk;

            // 방향키 입력 여부는 매 프레임 확인하기 때문데
            // 재생중일 때는 다시 재생하지 않도록 isPlaying으로 체크해서 재생ㅇ
            /*●*/if (audioSource.isPlaying==false)
            {
                audioSource.loop = true;
                audioSource.Play();
            }
        }
        else
        {
            movement.MoveSpeed = 0;
            anim.MoveSpeed = 0;

            // 멈췄을 때 사운드가 재생중이면 정지
            /*●*/if (audioSource.isPlaying== true)
            {
                audioSource.Stop();
            }
        }

        movement.MoveTo(new Vector3(hAxis, 0, vAxis));
    }
}
```

(9) Player오브젝트에 AudioSource컴포넌트를 추가하고, PlayerController컴포넌트에 Clip변수들에 사운드를 넣어준다.

이제 게임을 실행하면 걸을 때는 걷는 사운드가, 달리면 달리기 사운드가 재생된다.

***

# 플레이어 캐릭터 중력과 점프

(1) MovementCharacterController를 중력 적용과 점프가 가능하게 수정한다.

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

    /*●*/[SerializeField]
    /*●*/private float jumpForce;        // 점프 힘
    /*●*/[SerializeField]
    // TIP) Physics.gravity 변수를 사용해 rigidbody와 동일한 중력을 설정할 수도 있지만
    // 좀 더 큰 중력 값을 적용하기 위해 변수를 따로 선언
    /*●*/private float gravity;             // 중력 계수

    public float MoveSpeed
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
        // 공중에 떠 있으면 중력만큼 y축 이동속도 감소
        /*●*/if (!characterController.isGrounded)
        /*●*/{
        /*●*/    moveForce.y+=gravity*Time.deltaTime;
        /*●*/}

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
    /*●*/public void Jump()
    /*●*/{
    /*●*/    // 플레이어가 바닥에 있을 때만 점프 가능
    /*●*/    if (characterController.isGrounded)
    /*●*/    {
    /*●*/        moveForce.y = jumpForce;
    /*●*/    }
    /*●*/}
}
```

(2) PlayerController스크립트를 수정한다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    [Header("Input KeyCodes")]
    [SerializeField]
    private KeyCode keyCodeRun = KeyCode.LeftShift;            // 달리기 키
    /*●*/[SerializeField]
    /*●*/private KeyCode keyCodeJump = KeyCode.Space;            // 점프 키


    [Header("Audio Clips")]
    [SerializeField]
    private AudioClip audioClipWalk;           // 걷기 사운드
    [SerializeField]
    private AudioClip audioClipRun;            // 달리기 사운드

    private RotateToMouse rotateToMouse;        // 마우스 이동으러 카메라 회전
    private MovementCharacterController movement;      // 키보드 입력으로 플레이어 이동, 점프
    private Status status;      // 이동속도 등의 플레이어 정보
    private PlayerAnimatorController anim;      // 애니메이션 재생 제어
    private AudioSource audioSource;        // 사운드 재생 제어

    private void Awake()
    {
        // 마우스 커서를 보이지 않게 설정하고, 현재 위치에 고정 시킨다.
        Cursor.visible = false;
        Cursor.lockState = CursorLockMode.Locked;

        rotateToMouse = GetComponent<RotateToMouse>();
        movement=GetComponent<MovementCharacterController>();
        status=GetComponent<Status>();
        anim=GetComponent<PlayerAnimatorController>();
        audioSource=GetComponent<AudioSource>();
    }
    private void Update()
    {
        UpdateRotate();
        UpdateMove();
        /*●*/UpdateJump();
    }
    private void UpdateRotate()
    {
        float mouseX = Input.GetAxis("Mouse X");
        float mouseY = Input.GetAxis("Mouse Y");

        rotateToMouse.UpdateRotate(mouseX, mouseY);
    }
    private void UpdateMove()
    {
        float hAxis = Input.GetAxis("Horizontal");
        float vAxis = Input.GetAxis("Vertical");

        // 이동중 일 때 (걷기 or 뛰기)
        if(hAxis != 0 || vAxis != 0)
        {
            bool isRun = false;

            // 옆이나 뒤로 이동할 때는 달릴 수 없다.
            if(vAxis>0) isRun=Input.GetKey(keyCodeRun);

            movement.MoveSpeed = isRun ? status.RunSpeed : status.WalkSpeed;
            anim.MoveSpeed = isRun ? 1.0f : 0.5f;
            audioSource.clip = isRun ? audioClipRun : audioClipWalk;

            // 방향키 입력 여부는 매 프레임 확인하기 때문데
            // 재생중일 때는 다시 재생하지 않도록 isPlaying으로 체크해서 재생ㅇ
            if (audioSource.isPlaying==false)
            {
                audioSource.loop = true;
                audioSource.Play();
            }
        }
        else
        {
            movement.MoveSpeed = 0;
            anim.MoveSpeed = 0;

            // 멈췄을 때 사운드가 재생중이면 정지
            if (audioSource.isPlaying== true)
            {
                audioSource.Stop();
            }
        }

        movement.MoveTo(new Vector3(hAxis, 0, vAxis));
    }
    /*●*/private void UpdateJump()
    /*●*/{
    /*●*/    if (Input.GetKeyDown(keyCodeJump))
    /*●*/    {
    /*●*/        movement.Jump();
    /*●*/    }
    /*●*/}
}
```

이제 게임을 실행하고 스페이스바를 누르면 점프하는 모습을 볼 수 있다.
