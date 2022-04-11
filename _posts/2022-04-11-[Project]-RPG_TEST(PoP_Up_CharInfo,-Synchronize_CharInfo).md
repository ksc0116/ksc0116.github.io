---
layout: post
title:  "[Project] RPG_TEST(PoP_Up_CharInfo, Synchronize_CharInfo)"
subtitle:   ""
categories: Unity
comments: true
---

<br>

# 정보창 팝업 시키기

플레이어를 누르면 정보창이 활성화되도록 만들 것이다.

(1) Manager오브젝트에 빈 오브젝트(Manager_Inven)를 만든다.

(2) 정보창들에 대한 관리를 담당 할 Manager_Inven스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Manager_Inven : MonoBehaviour
{
    [Header("Frame")]
    public GameObject charInfoFrame;
}
```

(3) Manger스크립트를 수정한다.(추가된 Manager_Inven 등록)

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Manager : MonoBehaviour
{
    public static Manager instance;

    [Header("Manager")]
    public Manager_SE manager_SE;
    public Manager_Inven manager_Inven;

    private void Awake()
    {
        if(instance != this)
            instance = this;
    }
}
```

(4) Manager_Inven 오브젝트에 Manager_Inven 스크립트를 넣어주고 변수를 할당한다.

(5) Manager오브젝트에 추가된 변수에 데이터를 할당한다.

(6) Manager_SE스크립트를 수정한다.(버튼효과음 추가)

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Manager_SE : MonoBehaviour
{
    public AudioSource seAudio;

    [Header("Clip")]
    public AudioClip step;
    public AudioClip btnA;
}
```

(7) Manager_SE오브젝트에 새로 추가된 변수에 데이터를 할당한다.

(8) PlayerController스크립트를 수정한다.(플레이어를 클릭하면 정보창이 뜨도록)

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

            if (hit.transform.gameObject.tag == "Player")
            {
                Manager.instance.manager_SE.seAudio.PlayOneShot(
                    Manager.instance.manager_SE.btnA);

                Manager.instance.manager_Inven.charInfoFrame.SetActive(true);
            }
        }
    }
    private void Update()
    {
        playerAnim.SetBool("Walk", playerNav.velocity != Vector3.zero);
    }
}
```

(9) Player오브젝트의 태그를 Player로 바꾸고 BoxCollider를 추가한다.

이제 게임을 실행하고 플레이어를 누르면 정보창이 나오는걸 볼 수 있다.

# 정보창 닫기

화면에 빈 곳을 누르면 정보창이 사라지도록 만든다.

(1) CharacterInfoFrame오브젝트에 PlaySound스크립트를 넣어준다.

(2) CharacterInfoFrame오브젝트에 Event Trigger 컴포넌트를 추가한다.

(3) Event Type을 Pointer Down으로 하고 항목 2개를 추가한 다음 하나는 정보창을 닫을 때 나는 소리 재생 함수, 하나는 정보창 비활성화하는 함수를 등록한다.

![image](https://user-images.githubusercontent.com/101051124/162687579-557fe0b4-a509-4d72-a151-24896d1782f9.png)

이제 게임을 실행하고 정보창을 활성화 시킨 뒤 화면을 클릭하면 정보창이 비활성화 되는 모습을 볼 수 있다.

***

***

# 정보창 동기화

정보창이 활성화 되었을 때 실제 캐릭터의 스탯이 반영되도록 만들어보자.

(1) Player의 스탯을 관리하는 PlayerState스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerState : MonoBehaviour
{
    public int lev;
    public float hp;
    public int atk;
    public int def;
    public float cri;
    public float exp_Max;

    public float exp_Cur;
    public float hp_Cur;

    private void OnEnable()
    {
        hp_Cur = hp;
    }
}
```

(2) Player오브젝트에 PlayerState스크립트를 넣어주고 능력치를 정해준다.

(3) CharInfoFrame 스크립트를 만든다.(플레이어의 실제 정보와 동기화 해줌)

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using TMPro;

public class CharInfoFrame : MonoBehaviour
{
    public PlayerState player;

    // 정보창이 활성화 될 때 동기화할 요소들을 선언한다.
    public TextMeshProUGUI lev;
    public TextMeshProUGUI hp;
    public TextMeshProUGUI atk;
    public TextMeshProUGUI def;
    public TextMeshProUGUI cri;
    public Image expBar;

    private void OnEnable()
    {
        lev.text = player.lev.ToString();
        hp.text = player.hp.ToString();
        atk.text = player.atk.ToString();
        def.text = player.def.ToString();
        cri.text = string.Format("{0:0.00}",player.cri);

        expBar.fillAmount = player.exp_Cur / player.exp_Max;
    }
}
```

(4) CharacterInfoFrame오브젝트에 CharInfoFrame스크립트를 넣어주고 새로 생긴 변수들에 데이터를 할당한다.

이제 게임을 실행시키고 Player의 능력치를 바꾼 다음 정보창을 활성화 시키면 동기화가 되는 걸 볼 수 있다.