---
layout: post
title:  "[Project] RPG_TEST(Nav, 클릭한 곳에 오브젝트 생성)"
subtitle:   ""
categories: Unity
comments: true
---

<br>

클릭한 곳에 Touch글자가 나타나게 하고, Player가 터치한 곳으로 이동하게 하기(NavMesh 이용)

# Touch 텍스트 만들기

(1) UI - TMP(TouchEffect) 만들기 text(Touch), 크기(50), 정렬(가운데 중간)

(2) TouchEffect에 애니메이션 적용을 위해 Animation폴더에 UI폴더를 만들고 그 안에 애니메이션(TouchEffect) 하나 생성한다.

(3) TouchEffect애니메이션을 TouchEffect오브젝트에 넣어준다.

(4) 애니메이션을 제작하는데 Animation창을 열고 TouchEffect를 클릭한 다음 녹화버튼을 눌러서 나타날 때 커지는 애니메이션을 만든다.

![image](https://user-images.githubusercontent.com/101051124/162625613-c76ce64f-c379-46ad-8d2d-e5fc035fd738.png)

(5) 크기(0.5 / 0.5 / 1), 크기(2 / 2 / 1) 녹화 버튼을 다시 눌러 녹화를 끝낸다.

(6) Canvas의 Canvas Scaler: Sclae Mode(Scale With ScreenSize), Reference Resolution(1920 / 1080)

(7) Touch텍스트가 나타났다가 자동으로 사라지는 DisableObj스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class DisableObj : MonoBehaviour
{
    public float disableTime;       // 비활성화 될 시간

    private void OnEnable()
    {
        // 오브젝트가 활성화되면 실행중인 Invoke를 취소
        CancelInvoke();

        Invoke("Disable", disableTime);
    }

    // 오브젝트를 끄는 기능
    void Disable()
    {
        gameObject.SetActive(false);
    }
}
```

(8) TouchEffect 오브젝트에 DisableObj를 넣어주고 disableTime를 0.5로 설정하고 비활성화 한다.

게임을 실행시키고 활성화 시키면 TouchEffect가 커졌다가 자동으로 사라지는 모습을 볼 수 있다.

(9) 터치를 감지해 줄 Panel(PlayerController)을 만든다. 색상(알파값 0)

(10) Player오브젝트를 제어 하고 TouchEffect를 제어하는 PlayerController스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.EventSystems;

public class PlayerController : MonoBehaviour, IPointerClickHandler
{
    [Header("Camera")]
    public Camera cam;
    public GameObject touchEffect;   
    RaycastHit hit;

    public void OnPointerClick(PointerEventData eventData)
    {
        Ray ray = cam.ScreenPointToRay(eventData.position);
        Physics.Raycast(ray, out hit);

        if (hit.transform != null)
        {
            touchEffect.SetActive(false);
            touchEffect.transform.position=cam.WorldToScreenPoint(hit.point);
            touchEffect.SetActive(true);
        }
    }
}
```

(11) PlayerController오브젝트에 PlayerController스크립트를 넣어주고 변수들을 할당한다.

(12) 마을 오브젝트에 있는 Pane을 선택하고 BoxCollider를 추가한다.

**TIP)** 콜라이더가 있어야 Ray에 맞을 수 있다. 우리는 PlayerController스크립트에서 Ray를 사용하기 때문에 추가한다.

(13) TouchEffect오브젝트를 회전시킨다.(0 / 0 / 14)

이제 게임을 실행시키면 내가 클릭한 곳에 Touch텍스트가 생기는 모습을 볼 수 있다.

# 플레이어 이동

(1) NavMesh를 사용하기 위해서 마을 오브젝트에서 Plane과 Props를 선택하고 인스펙터 창의 오른쪽 상단에 있는 static을 눌러 Navigation Static을 선택한다.

![image](https://user-images.githubusercontent.com/101051124/162627362-2f2ef3b2-964f-410f-8d0a-f6269e78a6a6.png)

(2) Window - AI -Navigation를 눌러서 Navigation창을 열고 Bake탭으로 가서 Bake해준다.

![image](https://user-images.githubusercontent.com/101051124/162627428-df734cfa-58a3-4475-8450-5b4871b8c616.png)

(3) Player 오브젝트에 Nav Mesh Agent컴포넌트를 넣어주고 다음과 같이 설정한다.

![image](https://user-images.githubusercontent.com/101051124/162627517-6ce9a0be-f2d5-4ea6-859a-7ca77dac3cae.png)

(4) PlayerController스크립트를 수정한다.(플레이어 움직임 추가)

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.EventSystems;
using UnityEngine.AI;

public class PlayerController : MonoBehaviour, IPointerClickHandler
{
    [Header("Camera")]
    public Camera cam;
    public GameObject touchEffect;   
    RaycastHit hit;

    [Header("Player")]
    public Transform player;
    public float moveSpeed;
    public float ratateSpeed;
    NavMeshAgent playerNav;

    private void Start()
    {
        playerNav = player.GetComponent<NavMeshAgent>();
        playerNav.speed = moveSpeed;
        playerNav.angularSpeed = ratateSpeed;
    }

    public void OnPointerClick(PointerEventData eventData)
    {
        Ray ray = cam.ScreenPointToRay(eventData.position);
        Physics.Raycast(ray, out hit);

        if (hit.transform != null)
        {
            touchEffect.SetActive(false);
            touchEffect.transform.position=cam.WorldToScreenPoint(hit.point);
            touchEffect.SetActive(true);

            if (hit.transform.gameObject.tag == "Land")
                playerNav.SetDestination(hit.point);
        }
    }
}
```

(5) PlayerContoller컴포넌트에 변수를 할당한다. moveSpeed(5), RotateSpeed(200)

(6) 마을 오브젝트에 있는 plane에 Land태그를 생성하고 태그를 Land로 바꿔준다.

이제 게임을 실행하면 내가 클릭한 곳으로 Player 오브젝트가 움직이는 모습을 볼 수 있다.