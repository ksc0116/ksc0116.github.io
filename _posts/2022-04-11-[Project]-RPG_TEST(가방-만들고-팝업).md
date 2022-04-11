---
layout: post
title:  "[Project] RPG_TEST(가방 만들고 팝업)"
subtitle:   ""
categories: Unity
comments: true
---

<br>

# 가방 창 만들기

(1) Canvas오브젝트의 자식으로 빈 오브젝트(BagFrame)을 만든다.

(2) BagFrame오브젝트의 자식으로 Image(Bag)를 만들고 SourceImage(Tool_Bag), 너비(500), 높이(500)

(3) BagFrame오브젝트의 자식으로 TMP(Name_Bag)을 만들고 알맞게 설정한다.

(4) BagFrame오브젝트의 자식으로 Image(GoleBox)를 만들고  SourceImage(Tool_Slot), 너비(250), 높이(60)

(5) GoleBox오브젝트의 자식으로 TMP(G)를 만들고 알맞게 설정한다.

(6) G오브젝트를 복사하고 이름을 (Amount)로 바꾸고 알맞게 설정한다.

(7) BagFrame오브젝트의 자식으로 빈 오브젝트(SlotBox)를 만든다.

(8)  SlotBox오브젝의 자식으로 Image(Slot)을 만들고 알맞게 설정한 다음 계속 복사하여 다음과 같은 모양을 만들면 된다.

![image](https://user-images.githubusercontent.com/101051124/162772428-2de15a1e-7e8d-4ba4-a688-5bd134d7721a.png)

(9) 최종적인 위치 선정을 해주고 비활성화 시킨다.

(10) Manager_Inven스크립트를 수정한다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using TMPro;

public class Manager_Inven : MonoBehaviour
{
    [Header("Frame")]
    public GameObject charInfoFrame;
    public GameObject bagFrame;

    [Header("Bag")]
    public int gold;
    public TextMeshProUGUI goldAmount;

    public void OpenBag()
    {
        goldAmount.text=gold.ToString();
        bagFrame.SetActive(true);
    }
}
```

(11) Manager_Inven오브젝트에 새로 생긴 변수에 데이터를 할당한다.

(12) PlayerController스크립트를 수정한다.

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

                Manager.instance.manager_Inven.OpenBag();
            }
        }
    }
    private void Update()
    {
        playerAnim.SetBool("Walk", playerNav.velocity != Vector3.zero);
    }
}
```

이제 게임을 실행하여 플레이어를 클릭하면 정보창과 함께 가방창도 나타나는걸 볼 수 있다.

(13) CharInfoFrame오브젝트에 있는 EventTrigger컴포넌트에 가방창이 사라지는 SetActive()함수를 추가한다.

![image](https://user-images.githubusercontent.com/101051124/162776170-295f8048-4f54-4446-92dd-5d4041159fb4.png)