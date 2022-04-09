---
layout: post
title:  "[Project] 3D Action(Camera, Player Move)"
subtitle:   ""
categories: Unity
comments: true
---

<br>

# 카메라 회전

(1) Camera의 중심축으로 사용 할 빈 오브젝트(Cam_CentralAxis)를 만든다. 위치(0 / 0 / 0)

(2) Main camera오브젝트를 Cam_centralAxis의 자식으로 넣어준다.

**중심축이 회전을 담당하고 Main Camera오브젝트는 줌 인, 줌 아웃을 담당한다.**

(3) Controller 스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Controller : MonoBehaviour
{
    [Header("Camera")]
    [SerializeField]
    private Transform camAxis_Central;
    [SerializeField]
    private Transform cam;
    [SerializeField]
    private float camSpeed;
    private float rotX;
    private float rotY;
    private float wheel;

    private void CamMove()
    {
        rotX -= Input.GetAxis("Mouse Y")*camSpeed*Time.deltaTime;
        rotY += Input.GetAxis("Mouse X") * camSpeed * Time.deltaTime;

        Quaternion rot = Quaternion.Euler(rotX, rotY, 0);

        if(rotX > 80)
            rotX = 80;
        if(rotX < 0)
            rotX = 0;

        camAxis_Central.rotation = rot;
    }
    private void Update()
    {
        CamMove();
    }
}
```

(4) Controller스크립트를 컴포넌트로 추가해 줄 빈 오브젝트(Controller)를 만들고 Controller스크립트를 추가해준 다음 변수들을 할당해준다.

# 줌 기능

(1) Controller스크립트를 수정한다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Controller : MonoBehaviour
{
    [Header("Camera")]
    [SerializeField]
    private Transform camAxis_Central;
    [SerializeField]
    private Transform cam;
    [SerializeField]
    private float camSpeed;
    private float rotX;
    private float rotY;
    private float wheel;
    [SerializeField]
    private float wheelSpeed;
    [SerializeField]
    private float zoomMin;
    [SerializeField]
    private float zoomMax;
    private void CamMove()
    {
        rotX -= Input.GetAxis("Mouse Y")*camSpeed*Time.deltaTime;
        rotY += Input.GetAxis("Mouse X") * camSpeed * Time.deltaTime;

        Quaternion rot = Quaternion.Euler(rotX, rotY, 0);

        if(rotX > 80)
            rotX = 80;
        if(rotX < 0)
            rotX = 0;

        camAxis_Central.rotation = rot;
    }
    private void Zoom()
    {
        wheel -= Input.GetAxis("Mouse ScrollWheel")* wheelSpeed*Time.deltaTime;

        wheel=Mathf.Clamp(wheel, zoomMin, zoomMax);

        cam.localPosition = new Vector3(0, 0, -wheel);
    }
    private void Update()
    {
        CamMove();

        Zoom();
    }
}
```

(2) 에디터에서 Controll오브젝트에 새로 생긴 변수들에 값을 넣어주고 플레이해보면 줌 기능이 구현된 걸 확인할 수 있다.

(3) Cam_CentralAxis 위치(0 / 4 / 0)

# 플레이어 이동

(1) controller스크립트를 수정한다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Controller : MonoBehaviour
{
    [Header("Camera")]
    [SerializeField]
    private Transform camAxis_Central;
    [SerializeField]
    private Transform cam;
    [SerializeField]
    private float camSpeed;
    private float rotX;
    private float rotY;
    private float wheel;
    [SerializeField]
    private float wheelSpeed;
    [SerializeField]
    private float zoomMin;
    [SerializeField]
    private float zoomMax;

    [Header("Player")]
    [SerializeField]
    private Transform playerAxis;
    [SerializeField]
    private Transform player;
    [SerializeField]
    private float playerSpeed;
    [SerializeField]
    private Vector3 movement;

    private void Awake()
    {
        wheel = -10;
        rotX = 4;
    }
    private void CamMove()
    {
        rotX -= Input.GetAxis("Mouse Y")*camSpeed*Time.deltaTime;
        rotY += Input.GetAxis("Mouse X") * camSpeed * Time.deltaTime;

        Quaternion rot = Quaternion.Euler(rotX, rotY, 0);

        if(rotX > 80)
            rotX = 80;
        if(rotX < 0)
            rotX = 0;

        camAxis_Central.rotation = rot;
    }
    private void Zoom()
    {
        wheel -= Input.GetAxis("Mouse ScrollWheel")* wheelSpeed*Time.deltaTime;

        wheel=Mathf.Clamp(wheel, zoomMin, zoomMax);

        cam.localPosition = new Vector3(0, 0, -wheel);
    }
    private void PlayerMove()
    {
        movement = new Vector3(Input.GetAxis("Horizontal"), 0, Input.GetAxis("Vertical"));
        if (movement != Vector3.zero)
        {
            playerAxis.rotation = Quaternion.Euler(0, rotY, 0);

            playerAxis.Translate(movement*playerSpeed*Time.deltaTime);

            player.localRotation = Quaternion.Slerp(player.localRotation,Quaternion.LookRotation(movement),5*Time.deltaTime);
        }
        camAxis_Central.position = new Vector3(player.position.x,player.position.y+4,player.position.z);
    }
    private void Update()
    {
        CamMove();

        Zoom();

        PlayerMove();
    }
}
```

(2) 빈 오브젝트(PlayerAxis)를 만들고 Player오브젝트를 자식으로 넣어준다.

(3) Controller오브젝트의 Controller컴포넌트 변수에 할당해준다.

이제 게임 실행을 하면 Player가 움직이는 모습을 볼 수 있다.

# 플레이어 애니메이션 설정

Player에 Stand와 Run 애니메이션을 추가할 것이다.

(1) animator뷰에 Stand와 Walk애니메이션을 추가한다.

(2) Stand를 기본 애니메이션으로 설정하고 아래와 같이 Transition을 만든다.

![image](https://user-images.githubusercontent.com/101051124/162553835-1980dd36-a6bf-49f3-82ae-c54c7f5d26b4.png)

(3) controller스크립트를 수정한다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Controller : MonoBehaviour
{
    [Header("Camera")]
    [SerializeField]
    private Transform camAxis_Central;
    [SerializeField]
    private Transform cam;
    [SerializeField]
    private float camSpeed;
    private float rotX;
    private float rotY;
    private float wheel;
    [SerializeField]
    private float wheelSpeed;
    [SerializeField]
    private float zoomMin;
    [SerializeField]
    private float zoomMax;

    [Header("Player")]
    [SerializeField]
    private Transform playerAxis;
    [SerializeField]
    private Transform player;
    [SerializeField]
    private float playerSpeed;
    [SerializeField]
    private Vector3 movement;

    private void Awake()
    {
        wheel = -10;
        rotX = 4;
    }
    private void CamMove()
    {
        rotX -= Input.GetAxis("Mouse Y")*camSpeed*Time.deltaTime;
        rotY += Input.GetAxis("Mouse X") * camSpeed * Time.deltaTime;

        Quaternion rot = Quaternion.Euler(rotX, rotY, 0);

        if(rotX > 80)
            rotX = 80;
        if(rotX < 0)
            rotX = 0;

        camAxis_Central.rotation = rot;
    }
    private void Zoom()
    {
        wheel -= Input.GetAxis("Mouse ScrollWheel")* wheelSpeed*Time.deltaTime;

        wheel=Mathf.Clamp(wheel, zoomMin, zoomMax);

        cam.localPosition = new Vector3(0, 0, -wheel);
    }
    private void PlayerMove()
    {
        movement = new Vector3(Input.GetAxis("Horizontal"), 0, Input.GetAxis("Vertical"));
        if (movement != Vector3.zero)
        {
            playerAxis.rotation = Quaternion.Euler(0, rotY, 0);

            playerAxis.Translate(movement*playerSpeed*Time.deltaTime);

            player.localRotation = Quaternion.Slerp(player.localRotation,Quaternion.LookRotation(movement),5*Time.deltaTime);

            player.GetComponent<Animator>().SetBool("Walk", true);
        }
        if (movement == Vector3.zero)
        {
            player.GetComponent<Animator>().SetBool("Walk", false);
        }
        camAxis_Central.position = new Vector3(player.position.x,player.position.y+4,player.position.z);
    }
    private void Update()
    {
        CamMove();

        Zoom();

        PlayerMove();
    }
}
```

이제 게임을 실행하면 Player가 움직일 때 애니메이션이 재생되는 걸 확인할 수 있다.