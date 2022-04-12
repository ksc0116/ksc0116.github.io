---
layout: post
title:  "[Project] RPG_TEST(Item Drag & Drop)"
subtitle:   ""
categories: Unity
comments: true
---

<br>

# 가방안에 있는 아이템 옮기기

(1) 우선 BagFrame오브젝트 안에 있는 Slot오브젝트에 Image(Weapon_ShamansHammer)를 만들고 SourceImage(Weapon_ShamansHammer)를 바꾼다.

(2) Manager_Inven 스크립트를 수정한다.(드래그 & 드롭에 필요한 변수들 선언)

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

    [Header("Drag&Drpo")]
    public Transform selectedItem;
    public Transform curParent;
    public Transform parentOnDrag;

    public void OpenBag()
    {
        goldAmount.text=gold.ToString();
        bagFrame.SetActive(true);
    }
}
```

(3) Items_Action스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.EventSystems;
using UnityEngine.UI;

public class Items_Action : MonoBehaviour,IPointerUpHandler, 
    IPointerDownHandler,IDragHandler
{
    public Image img;

    [Header("Location")]
    public bool inBag;

    private float releasTime;
    private bool dragging;

    public void OnPointerDown(PointerEventData eventData)
    {
        img=GetComponent<Image>();
        Manager.instance.manager_Inven.selectedItem = transform;

        if (inBag)
        {
            StartCoroutine("ReleaseTime");
        }
    }

    public void OnPointerUp(PointerEventData eventData)
    {
        if (inBag)
        {
            StopCoroutine("ReleaseTime");
            transform.localScale = new Vector3(1, 1, 1);

            if (releasTime >= 0.1f)
            {
                transform.SetParent(Manager.instance.manager_Inven.curParent);
                transform.localPosition = Vector3.zero;
                img.raycastTarget = true;
                return;
            }

            if (releasTime < 0.1f)
            {
                // click events
            }
        }
    }

    public void OnDrag(PointerEventData eventData)
    {
        if(inBag && dragging)
            transform.position = eventData.position;
    }

    private IEnumerator ReleaseTime()
    {
        releasTime = 0;
        dragging = false;

        while (true)
        {
            releasTime += Time.deltaTime;

            if(releasTime >= 0.1f)
            {
                transform.localScale = new Vector3(1.3f, 1.3f, 1);

                if (!dragging)
                {
                    dragging = true;
                    Manager.instance.manager_Inven.curParent = transform.parent;
                    transform.SetParent(Manager.instance.manager_Inven.parentOnDrag);

                    img.raycastTarget = false;
                }
            }

            yield return null;
        }
    }
}
```

(4) Weapon_ShamansHammer오브젝트에 Items_Action스크립트를 넣어준다. inBag(true)

(5) Manager_Inven:  ParentOnDrag(SlotBox)로 설정한다.

이제 게임을 실행해서 가방안에 있는 Weapon_ShamansHammer이미지를 클릭하면 크기가 조금 커지고 나의 마우스를 따라니는걸 볼 수 있다.

(6) Items_Drop스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.EventSystems;

public class Items_Drop : MonoBehaviour, IDropHandler
{
    public void OnDrop(PointerEventData eventData)
    {
        Manager_Inven inven = Manager.instance.manager_Inven;

        if (transform.childCount != 0)
        {
            Transform item = transform.GetChild(0);
            item.SetParent(inven.curParent);
            item.localPosition = Vector3.zero;
        }

        inven.selectedItem.SetParent(transform);
        inven.selectedItem.localPosition = Vector3.zero;
    }
}
```

(7) 모든 Slot 오브젝트에 Items_Drop스크립트를 넣어준다.

이제 게임을 실행하여 아이템을 클릭하고 다른 칸에 놓으면 놓아지는 것을 볼 수 있다.

# 아이템끼리 자리 바꾸기

(1) Slot(1)오브젝트에 Image(Head_SkullCap)를 추가한다. SourceImage(Head_SkullCap)

(2) Head_SkullCap오브젝트에 Items_Action스크립트를 넣어준다. InBag(true)

이제 게임을 실행하여 아이템을 드래그해서 다른 아이템의 위치로 바꾸면 저절로 바뀌는걸 볼 수 있다.

# 효과음 추가

(1) Manager_SE스크립트를 수정한다.

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
    public AudioClip drag;
}
```

(2) Manager)_SE에 새로 생긴 변수에 Drag사운드를 추가한다.

(3) 소리를 재생시키기 위해 Items_Action스크립트를 수정한다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.EventSystems;
using UnityEngine.UI;

public class Items_Action : MonoBehaviour,IPointerUpHandler, 
    IPointerDownHandler,IDragHandler
{
    public Image img;

    [Header("Location")]
    public bool inBag;

    private float releasTime;
    private bool dragging;

    public void OnPointerDown(PointerEventData eventData)
    {
        img=GetComponent<Image>();
        Manager.instance.manager_Inven.selectedItem = transform;

        if (inBag)
        {
            StartCoroutine("ReleaseTime");
        }
    }

    public void OnPointerUp(PointerEventData eventData)
    {
        if (inBag)
        {
            StopCoroutine("ReleaseTime");
            transform.localScale = new Vector3(1, 1, 1);

            if (releasTime >= 0.1f)
            {
                transform.SetParent(Manager.instance.manager_Inven.curParent);
                transform.localPosition = Vector3.zero;
                img.raycastTarget = true;
                return;
            }

            if (releasTime < 0.1f)
            {
                // click events
            }
        }
    }

    public void OnDrag(PointerEventData eventData)
    {
        if(inBag && dragging)
            transform.position = eventData.position;
    }

    private IEnumerator ReleaseTime()
    {
        releasTime = 0;
        dragging = false;

        while (true)
        {
            releasTime += Time.deltaTime;

            if(releasTime >= 0.1f)
            {
                transform.localScale = new Vector3(1.3f, 1.3f, 1);

                if (!dragging)
                {
                    Manager.instance.manager_SE.seAudio.PlayOneShot(
                        Manager.instance.manager_SE.drag);

                    dragging = true;
                    Manager.instance.manager_Inven.curParent = transform.parent;
                    transform.SetParent(Manager.instance.manager_Inven.parentOnDrag);

                    img.raycastTarget = false;
                }
            }

            yield return null;
        }
    }
}
```

(3) 소리를 재생시키기 위해 Items_Drop스크립트를 수정한다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.EventSystems;

public class Items_Drop : MonoBehaviour, IDropHandler
{
    public void OnDrop(PointerEventData eventData)
    {
        Manager_Inven inven = Manager.instance.manager_Inven;

        Manager.instance.manager_SE.seAudio.PlayOneShot(
                    Manager.instance.manager_SE.drag);

        if (transform.childCount != 0)
        {
            Transform item = transform.GetChild(0);
            item.SetParent(inven.curParent);
            item.localPosition = Vector3.zero;
        }

        inven.selectedItem.SetParent(transform);
        inven.selectedItem.localPosition = Vector3.zero;
    }
}
```

이제 게임을 실행하면 사운드가 들리는걸 볼 수 있다.