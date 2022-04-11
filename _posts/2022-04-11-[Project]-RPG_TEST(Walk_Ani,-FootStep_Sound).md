---
layout: post
title:  "[Project] RPG_TEST(Walk_Ani, FootStep_Sound)"
subtitle:   ""
categories: Unity
comments: true
---

<br>

# 걷기 애니메이션

(1) Asset - Animations - Player에서 Walk 애니메이션을 Player오브젝트에 넣어준다.

(2) bool타입 파라미터(Walk)를 만든다.

(3) Idle -> Walk로 가는 Transition: Walk(true), duration(0), Has Exit Time(false)

​	Walk-> Idle로 가는 Transition: Walk(false), duration(0), Has Exit Time(false)

(4) PlayerController스크립트를 수정한다.(걷기 애니메이션 재생 설정)

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
    private RaycastHit hit;

    [Header("Player")]
    public Transform player;
    public float moveSpeed;
    public float ratateSpeed;
    private NavMeshAgent playerNav;
    private Animator playerAnim;

    private void Start()
    {
        playerNav = player.GetComponent<NavMeshAgent>();
        playerNav.speed = moveSpeed;
        playerNav.angularSpeed = ratateSpeed;
        playerAnim = player.GetComponent<Animator>();
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
    private void Update()
    {
        playerAnim.SetBool("Walk", playerNav.velocity != Vector3.zero);
    }
}
```

이제 게임을 실행하면 처음에는 가만히 있다가 내가 클리하는 순간에 걷기 애니메이션을 재생하고 목적지에 도착하면 재생하지 않는다.

***

# 발자국 사운드

(1) Manager들을 묶어서 관리하는 빈 오브젝트(Manager)를 만든다.

(2) Manager오브젝트의 자식으로 효과음을 관리 할 빈 오브젝트(Manager_SE)를 만든다.

(3) Manager_SE효과음 재생에 필요한 AudioSource컴포넌트를 추가한다. Play On Awake(false)

(4) 효과음을 관리 하는 Manager_SE스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Manager_SE : MonoBehaviour
{
    public AudioSource seAudio;

    [Header("Clip")]
    public AudioClip step;
}
```

(5) Manager_SE스크립트를 Manager_SE오브젝트에 넣어주고 변수들을 할당한다.

(6) Manager 오브젝트에 부착 할 Manager스크립트를 만든다(모든 Manager를 관리하는 스크립트)

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Manager : MonoBehaviour
{
    // 싱글톤 패턴
    public static Manager Instance;

    [Header("Manager")]
    public Manager_SE manager_SE;

    private void Awake()
    {
        if(Instance!=this)
            Instance = this;
    }
}
```

(7) Manager 오브젝트에 Manager스크립트를 넣어주고 변수를 할당한다.

(8) 스스로 효과음 재생이 필요한 오브젝트들에게 넣어 줄 PlaySound스크립트를 만든다.(발자국 사운드 재생 설정)

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlaySound : MonoBehaviour
{
    public void PlaySE(AudioClip clip)
    {
        Manager.Instance.manager_SE.seAudio.PlayOneShot(clip);
    }
}
```

(9) Player오브젝트에 PlaySound 스크립트를 넣어준다.

(10) 애니메이션 이벤트를 통해 사운드 재생 함수를 실행 시킨다.

(11) Walk 애니메이션에 발이 땅에 닿는 순간에 이벤트를 호출한다.

![image](https://user-images.githubusercontent.com/101051124/162658340-4fb24747-63e1-4e51-8f8d-fee8f5ae9719.png)

![image](https://user-images.githubusercontent.com/101051124/162658391-c60638e6-6f8a-4c29-9008-67da61b596d1.png)

이제 게임을 실행하고 클릭해서 움직일 때 발자국 효과음이 나오는걸 볼 수 있다.