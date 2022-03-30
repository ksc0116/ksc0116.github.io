---
layout: post
title:  "[Project] FPS Prototype(1)"
subtitle:   ""
categories: Unity
comments: true
---

<br>

# 프로젝트 기본 설정

(1) 해상도를 Full HD (1920 x 1080)으로 설정하고 Directional Light 회전(45 / 45 / 0)으로 설정한다.

(2) 월드에 배치 되는 오브젝트들을 관리하는 빈 오브젝트(WorldMap)를 만든다.

(3) 바닥 오브젝트에 적용 할 머테리얼(Ground100x100)을 생성한다.

(4) 바닥으로 사용할 Cube오브젝트(Ground)를 WorldMap오브젝트의 자식으로 만들고 방금 만든 머테리얼을 입힌 다음 크기(100 / 0.1 / 100)으로 설정한다.

(5) Ground100x100머테리얼을 복제하여 이름을 Wall100x50으로 바꾸고 Tilling (100 / 50)으로 설정한다.

(6) 벽으로 사용할 Cube오브젝트(Wall)를 WorldMap오브젝트의 자식으로 생성하고 방금 만든 머테리얼을 입힌다. 위치와 크기 정보는 아래와 같다.

Wall 위치 (-50 / 25 / 0), 크기 (0.1 / 50 / 100)

Wall(1) 위치 (50 / 25 / 0), 크기 (0.1 / 50 / 100)

Wall(2) 위치 (0 / 25 / 50), 크기 (100 / 50 / 0.1)

Wall(3) 위치 (0 / 25 / -50), 크기 (100 / 50 / 0.1)

(7) Ground100x100머테리얼을 복제하여 이름을 Obstacle2x2으로 바꾸고 Tilling (2 / 2)으로 설정한다.

(8) 장애물로 사용 할 Cube오브젝트(Obstacle)를 WorldMap오브젝트의 자식으로 생성하고 방금 만든 머테리얼을 입힌다. 위치 (0 / 1 / 5) 크기 (2 / 2 / 2)로 설정한다.

![image](https://user-images.githubusercontent.com/101051124/160748397-c09d3993-3016-4449-bcc3-fe8a92e8c748.png)

현재까지 만든 것은 위와 같다.

***

# 플레이어 캐릭터 오브젝트

(1) 플레이어에게 필요한 오브젝트들을 묶어서 관리하는 빈 오브젝트(Player)로 설정한다.위치 (0 / 1 / 0)

(2) MainCamera오브젝트를 Player 오브젝트 자식으로 배치하고 위치를 초기화 한 뒤 Camera 컴포넌트의 Clipping Planes::Near의 값을 0.01로 설정한다.(카메라가 볼 수 있는 시작지점을 좀 더 가까이 해서 내 눈앞에 있는 사물을 보기 위함)

(3) 다운 받은 에셋에서 Assault_Rifle_01/을 Player오브젝트 자식으로 넣어준다. 위치 (0/ -0.1 / 0.1)

**TIP) 플레이어 Mesh에 따라 배치되는 위치가 조금씩 다를 수 있으며, 손, 무기를 화면에 나타내고 싶은 범위, 각도에 따라 위치/회전을 다르게 설정**

(4) 바닥에 손 무기의 그림자가 생성되지 않도록 SkinnedMeshRenderer Component에서 Lighting::Cast Shadows를 꺼준다.

(5) 애니메이션 재생 및 제어를 위해 애니메이터 컨트롤러(WeaponAssaultRifle)를 하나 생성하고 무기 오브젝트에 넣어준다.

(6) WeaponAssaultRifle컨트롤러에 take_out_weapon과 idle 애니메이션을 넣어준다.

**TIP)** take_out_weapon은 총을 장전하고 조준하는 동작으로 게임을 시작하거나 총이 바뀔 때 무조건 1회 재생하도록 default로 설정

(7) take_out_weapon에서 idle로 가는 트랜지션을 하나 만들고 take_out_weapon을 1회 재생하고, idle로 넘어가기 때문에 condition은 설정하지 않는다.

(8)  AssaultRifle무기를 제어하는 WeaponAssaultRifle스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class WeaponAssaultRifle : MonoBehaviour
{
    [Header("Audio Clips")]
    [SerializeField]
    private AudioClip               audioClipTakeOutWeapon;           // 무기 장착 사운드

    private AudioSource          audioSource;                             // 사운드 재생 컴포넌트

    private void Awake()
    {
        audioSource = GetComponent<AudioSource>();
    }

    private void OnEnable()
    {
        // 무기 장착 사운드 재생
        PlaySound(audioClipTakeOutWeapon);
    }
    private void PlaySound(AudioClip clip)
    {
        audioSource.Stop();             // 기존에 재생중인 사운드를 정지하고,
        audioSource.clip = clip;        // 새로운 사운드 clip으로 교체 후
        audioSource.Play();             // 사운드 재생
    }
}
```

**[Header("Audio Clips")] : 인스펙터창에 Audio Clips가 보이게 되며 관련있는 필드들을 그룹화 할 수 있다.**

![image](https://user-images.githubusercontent.com/101051124/160753853-8a394413-d935-440c-a910-3528258a2051.png)

(9) 무기 오브젝트에 WeaponAssaultRifle스크립트, AudioSource컴포넌트를 를 추가해준다.

(10) WeaponAssaultRifle컴포넌트의 변수 Audio Clip Take Out weapon에 take_out_weapon클립을 넣어준다.

이제 게임을 실행하면 최초 1회 무기 꺼내는 소리가 재생되고 대기 상태가 된다.

***

# 카메라 회전

(1) 마우스를 이용해 카메라 회전을 제어하는 RotateToMouse스크립트를 만든다.

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class RotateToMouse : MonoBehaviour
{
    [SerializeField]
    private float rotCamXAXisSpeed = 5;     // 카메라 x축 회전속도
    [SerializeField]
    private float rotCamYAXisSpeed = 3;     // 카메라 y축 회전속도

    private float limitMinX = -80;          // 카메라 x축 회전 범위 (최소)
    private float limitMaxX = -80;          // 카메라 x축 회전 범위 (최대)
    private float eulerAngleX;
    private float eulerAngleY;

    public void UpdateRotate(float mouseX, float mouseY)
    {
        eulerAngleY += mouseX * rotCamYAXisSpeed;       // 마우스 좌/우 이동으로 카메라 y축 회전
        eulerAngleX-=mouseY * rotCamXAXisSpeed;         // 마우스 위/아래 이동으로 카메라 x축 회전

        // 카메라 x축 회전의 경우 회전 범위를 설정
        eulerAngleX = ClampAngle(eulerAngleX, limitMinX, limitMaxX);

        transform.rotation = Quaternion.Euler(eulerAngleX, eulerAngleY, 0);
    }
    private float ClampAngle(float angle,float min,float max)
    {
        if (angle < -360) angle += 360;
        if (angle > 360) angle -= 360;

        return Mathf.Lerp(angle, min, max);
    }
}
```

(2) 플레이어를 제어하는 PlayerController스크립트를 만든다.

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    private RotateToMouse rotateToMouse;        // 마우스 이동으러 카메라 회전
    private void Awake()
    {
        // 마우스 커서를 보이지 않게 설정하고, 현재 위치에 고정 시킨다.
        Cursor.visible = false;
        Cursor.lockState = CursorLockMode.Locked;

        rotateToMouse = GetComponent<RotateToMouse>();
    }
    private void Update()
    {
        UpdateRotate();
    }
    private void UpdateRotate()
    {
        float mouseX = Input.GetAxis("Mouse X");
        float mouseY = Input.GetAxis("Mouse Y");

        rotateToMouse.UpdateRotate(mouseX, mouseY);
    }
}
```

(3) 플레이어 오브젝트에 PlayerController, RotateToMouse스크립트를 넣어준다.

이제 카메라의 움직임에 따라 시점이 바뀌는 걸 볼 수 있다.