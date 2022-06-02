---
layout: post
title:  "[Project] RPG_TEST(Activate The Equip BTN)"
subtitle:   ""
categories: Unity
comments: true
---

<br>

# 내가 선택한 아이템 강조하기

(1) Canvas 오브젝트의 자식으로 Image(Rect)를 만들고 Image Source를 Tool_Bag으로 바꿔준 다음 색상을 노란색으로 바꾼다.

![image](https://user-images.githubusercontent.com/101051124/171572455-4f604fd8-ad7d-4358-802c-9222e529c3aa.png)

(2) Manager_Inven 스크립트를 수정한다.(방금 만든 Rect 이미지의 공간을 만들기 위해)

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using TMPro;

public class Manager_Inven : MonoBehaviour
{
    [Header("[Frame]")]
    public GameObject charInfoFrame;
    public GameObject bagFrame;
    public GameObject itemInfoFrame;

    [Header("[Bag]")]
    public int gold;
    public TextMeshProUGUI goldAmount;
    /*●*/public Transform rect;


    [Header("[Drag&Drop]")]
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

(3) Items_Action 스크립트를 수정한다.(내가 아이템을 클리했을 때 Manager_Inven에 있는 Rect의 Position값을 내가 클릭한 곳으로 옮기기 위해)

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.EventSystems;
using UnityEngine.UI;

public class Items_Action : MonoBehaviour,IPointerUpHandler,IPointerDownHandler,IDragHandler
{
    public Image img;

    [Header("[Location]")]
    public bool inBag;

    float releaseTime;
    bool dragging;

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

            if (releaseTime >= 0.1f)
            {
                transform.SetParent(Manager.instance.manager_Inven.curParent);
                transform.localPosition = Vector3.zero;
                img.raycastTarget = true;
                return;
            }

            if (releaseTime < 0.1f)
            {
                Manager.instance.manager_SE.seAudio.PlayOneShot(Manager.instance.manager_SE.btnB);

                Manager.instance.manager_Inven.itemInfoFrame.SetActive(false);

                Manager.instance.manager_Inven.itemInfoFrame.GetComponent<ItemInfoFrame>().item = GetComponent<Items_Info>();

                Manager.instance.manager_Inven.itemInfoFrame.SetActive(true);

                /*●*/Manager.instance.manager_Inven.rect.position=transform.position;
                /*●*/Manager.instance.manager_Inven.rect.gameObject.SetActive(true);
            }
        }
    }

    public void OnDrag(PointerEventData eventData)
    {
        if (inBag && dragging)
        {
            transform.position = eventData.position;
        }
    }

    IEnumerator ReleaseTime()
    {
        releaseTime = 0;
        dragging = false;

        while (true)
        {
            releaseTime += Time.deltaTime;

            if (releaseTime >= 0.1f)
            {
                transform.localScale = new Vector3(1.3f,1.3f,1);
                if (!dragging)
                {
                    Manager.instance.manager_Inven.itemInfoFrame.SetActive(false);

                    Manager.instance.manager_SE.seAudio.PlayOneShot(Manager.instance.manager_SE.drag);

                    dragging = true;
                    Manager.instance.manager_Inven.curParent = transform.parent;
                    transform.SetParent(Manager.instance.manager_Inven.parentOnDrag);

                    img.raycastTarget = false;

                    /*●*/Manager.instance.manager_Inven.rect.gameObject.SetActive(false);
                }
            }
            yield return null;
        }
    }
}
```

(4) Rect 오브젝트의 Image 컴포넌트에서 Raycast target을 꺼준다.

(5) Rect 오브젝트도 다른 Frame처럼 Off 할 수 있데 CharInfoFrame에 있는 Event Trigger컴포넌트에 추가해서 SetActive(false) 해준다.

# 장착 버튼 활성화 하기

(1) ItemInfoFrame오브젝트의 자식으로 빈 오브젝트(ButtonBox)를 만든다.

(2) Button Box의 자식으로 Button(EquipBtn)을 만든다.

* Width(200) / Height(70) / Source Image(Tool_Slot)

(3) EquipBtn의 Text를 Equip로 바꾼다.

(4) 장착중인 아이템은 해제하기 버튼을 만들어준다. EquipBtn을 복사(ReleaseBtn)한다. Text를 Release로 바꿔준다.

(5) 2개의 버튼을 비활성화 한다.

(6) Items_Info 스크립트를 수정한다.(현재 아이템이 장착중인지 여부를 bool 자료형으로 판단)

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Items_Info : MonoBehaviour
{
    [Header("[Commom Info]")]
    public string type;
    public string name_Item;
    public string info_Item;
    public int resalePrice;
    public int count;

    [Header("[Equipment Info]")]
    public int hpBonus;
    public int atkBonus;
    public int defBonus;
    public float criBonus;

    /*●*/public bool equipped;
}
```

(7) ItemInfoFrame 스크립트를 수정한다.(Item의 장착여부에 따라 활성화 시키는 버튼을 달리 함)

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

    /*●*/[Header("[Button]")]
    /*●*/public GameObject equipBtn;
    /*●*/public GameObject releaseBtn;

    private void OnEnable()
    {
        statBonus.SetActive(false);
        name_Item.text = item.name_Item;
        info_Item.text = item.info_Item;
        resalePrice.text = string.Format("Used Price : {0}",item.resalePrice);

        if(item.type == "Equipment")
        {
            hpBonus.text = string.Format("HP +{0}",item.hpBonus);
            atkBonus.text = string.Format("Atk +{0}", item.atkBonus);
            defBonus.text = string.Format("Def +{0}", item.defBonus);
            criBonus.text = string.Format("Cri +{0}", item.criBonus);
            statBonus.SetActive(true);

            /*●*/if (item.equipped == true)
            /*●*/{
            /*●*/    releaseBtn.SetActive(true);
            /*●*/}
            /*●*/else if(item.equipped == false)
            /*●*/{
            /*●*/    equipBtn.SetActive(true);
            /*●*/}
        }
    }
    /*●*/private void OnDisable()
    /*●*/{
    /*●*/    releaseBtn.SetActive(false);
    /*●*/    equipBtn.SetActive(false);
    /*●*/}
}
```

이제 게임을 실행해보면 장착이 가능한 아이템 중 작창하고 있으면 해제하기 버튼, 장찯하지 않고 있으면 장착 버튼이 활성화 되는 모습을 볼 수 있다.