---
layout: post
title:  "[Project] RPG_TEST(Item Info)"
subtitle:   ""
categories: Unity
comments: true
---

<br>

# 아이템 정보 창 만들기

(1) Canavas오브젝트의 자식으로 빈 오브젝트(ItemInfoFrame)을 만든다.

(2) ItemInfoFrame오브젝트 안에 Image(ItemInfoWindow)를 만든다. SourceImage(Tool_Slot) 너비(800), 높이(250)

(3) ItemInfoFrame오브젝트 안에 TMP(Name_Item)을 만들고 알맞게 설정한다.

(4) Name_item오브젝트를 복사하고 이름을 (Info_Item)으로 바꾼 다음 알맞게 설정한다.

(5) Info_Item오브젝트를 복사하고 이름을 (ResalePrice)로 바꾼 다음 알맞게 설정한다.

(6) ItemInfoFrame오브젝트 안에 빈 오브젝트(StatBonus)를 만든다.

(7) StatBonus오브젝트 안에 TMP(HP)를 만들고 알맞게 설정한다.

(8) HP오브젝트를 복사하여 이름을 (Atk)로 바꾸고 알맞게 설정한다.

(8) HP오브젝트를 복사하여 이름을 (Def)로 바꾸고 알맞게 설정한다.

(8) HP오브젝트를 복사하여 이름을 (Cri)로 바꾸고 알맞게 설정한다.

![image](https://user-images.githubusercontent.com/101051124/162894290-51e059b9-0fa4-4a87-b6e5-fa93e8cc9b80.png)

그럼 위와 같은 모습이 된다. 이제 최종적인 위치를 조정하자.

(9) 아이템들을 관리하는 Items_Info스크립트를 작성한다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Items_Info : MonoBehaviour
{
    [Header("Common Info")]
    public string type;
    public string name_Item;
    public string info_Item;
    public int resalePrice;
    public int count;

    [Header("Equipment Info")]
    public int hpBonus;
    public int atkBonus;
    public int defBonus;
    public float criBonus;
}
```

(10) Weapon_SahmonsHammer오브젝트에 Items_Info스크립트를 넣어준다.

![image](https://user-images.githubusercontent.com/101051124/162896364-60498a1c-eb7d-437d-95fb-42e9fa0137e0.png)

(11) Head_SkullCap오브젝트에 Items_Info 스크립트를 넣어준다.

![image-20220412154059112](C:\Users\ksc52\AppData\Roaming\Typora\typora-user-images\image-20220412154059112.png)

(12) 위와 같이 데이터들을 할당해준다.

(13) Skull Cap오브젝트를 복사하여 이름을 PebbleStome으로 바꾸고 Slot(2)안에 넣어준다. Sourceimage(PebbleStone)

![image](https://user-images.githubusercontent.com/101051124/162897708-3f9ec58c-c5f6-4126-a9a1-1f9092492437.png)

(14) ItemInfoFrame을 관리하는 ItemInfoFrame스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using TMPro;

public class ItemInfoFrame : MonoBehaviour
{
    public Items_Info item;


    public GameObject statBonus;
    public TextMeshProUGUI name_Item;
    public TextMeshProUGUI info_Item;
    public TextMeshProUGUI resalePrice;
    public TextMeshProUGUI hpBonus;
    public TextMeshProUGUI atkBonus;
    public TextMeshProUGUI defBonus;
    public TextMeshProUGUI criBonus;

    private void OnEnable()
    {
        statBonus.SetActive(false);
        name_Item.text = item.name_Item;
        info_Item.text=item.info_Item;
        resalePrice.text = string.Format("Used Price : {0}",item.resalePrice);

        if (item.type == "Equipment")
        {
            hpBonus.text =string.Format("HP + {0}", item.hpBonus);
            atkBonus.text = string.Format("ATK + {0}", item.atkBonus);
            defBonus.text = string.Format("DEF + {0}", item.defBonus);
            criBonus.text = string.Format("CRI + {0:0.00}", item.criBonus);
            statBonus.SetActive(true);
        }
    }
}
```

(15) ItemInfoFrame오브젝트에 ItemInfoFrame스크립트를 넣어주고 변수들을 할당한다.

(16) Manager_Inven스크립트를 수정한다.(itemInfoFrmae 설정)

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
    public GameObject itemInfoFrame;

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

(17) Items_Action스크립트를 수정한다.

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
                Manager.instance.manager_Inven.itemInfoFrame.SetActive(false);

                Manager.instance.manager_Inven.itemInfoFrame.
                    GetComponent<ItemInfoFrame>().item=
                    GetComponent<Items_Info>();

                Manager.instance.manager_Inven.itemInfoFrame.SetActive(true);
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

(18) Items_Action스크립트를 수정한다.

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
                Manager.instance.manager_Inven.itemInfoFrame.SetActive(false);

                Manager.instance.manager_Inven.itemInfoFrame.
                    GetComponent<ItemInfoFrame>().item =
                    GetComponent<Items_Info>();

                Manager.instance.manager_Inven.itemInfoFrame.SetActive(true);
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
                    Manager.instance.manager_Inven.itemInfoFrame.SetActive(false);

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

(19) CharInfoFrame오브젝트에서 EventTrigger컴포넌트에 ItemInfoFrame오브젝트를 비활성화하는 함수를 추가한다.

(20) 효과음을 주기 위해 Manager_SE스크립트를 수정한다.

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
    public AudioClip btnB;
}
```

(21) Items_Action 스크립트를 수정한다.

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
                Manager.instance.manager_SE.seAudio.PlayOneShot(
                    Manager.instance.manager_SE.btnB);

                Manager.instance.manager_Inven.itemInfoFrame.SetActive(false);

                Manager.instance.manager_Inven.itemInfoFrame.
                    GetComponent<ItemInfoFrame>().item =
                    GetComponent<Items_Info>();

                Manager.instance.manager_Inven.itemInfoFrame.SetActive(true);
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
                    Manager.instance.manager_Inven.itemInfoFrame.SetActive(false);

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

이제 게임을 실행해서 아이템을 클릭하면 정보창이 뜨는걸 볼 수 있다.