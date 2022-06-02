---
layout: post
title:  "[Project] RPG_TEST(Equip The Equipment)"
subtitle:   ""
categories: Unity
comments: true
---

<br>

# 아이템 장착하기

각 부위별 Mesh를 바꿔서 이 기능을 구현할 것이다.

(1) Items_info 스크립트를 수정한다.(Mesh 정보와 착용 번호 변수 생성)

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

    public bool equipped;
    /*●*/public Mesh mesh;
    /*●*/public int equipNum;
}
```

(2) 가방안에 있는 아이템 중 착용가능한 아이템의 Mesh를 할당해주고 hammer는 2번, cap은 0번으로 설정한다.

(3) ShamansHammer오브젝트의 자식으로 Text(Equipped)를 만들고 text를 Equipped로 바꾼다.

(4) Equipped오브젝트를 비활성화 하고 복사 한 다음 SkullCap의 자식으러 넣어준다.

(5) Hammer오브젝트를 복사하여 slot3 오브젝트의 자식으로 넣고 이름을 (ShamansStaff)로 바꾸고 컴포넌트의 프로퍼티를 재설정한다.

(6) SkullCap을 복사하여 slot4로 넣고 이름을(ShamanHeadband)로 바꾸고 컴포넌트의 프로퍼티를 재설정한다.

(7) ShamanHeadband를 복사하여 slot5로 넣고 이름을(ShamanArmor)로 바꾸고 컴포넌트의 프로퍼티를 재설정한다.

(8) ShamanArmor를 복사하여 slot6로 넣고 이름을(ShanmanClothesA)로 바꾸고 컴포넌트의 프로퍼티를 재설정한다.

(9) 장착 장비를 관리하는 Equip 스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Equip : MonoBehaviour
{
    [Header("[Player]")]
    public Mesh[] originMesh;
    public SkinnedMeshRenderer[] parts_Player;

    [Header("[Character Info]")]
    public Transform[] slot_Equip;
    public Items_Info[] cur_Equip;

    public void EquipBtn()
    {
        Items_Info item=Manager.instance.manager_Inven.selectedItem.GetComponent<Items_Info>();

        if (slot_Equip[item.equipNum].childCount == 2)
        {
            cur_Equip[item.equipNum].equipped = false;
            cur_Equip[item.equipNum].transform.GetChild(0).gameObject.SetActive(false);
            Destroy(slot_Equip[item.equipNum].GetChild(1).gameObject);
        }

        GameObject item_Slot = Instantiate(item.gameObject, slot_Equip[item.equipNum]);
        item_Slot.GetComponent<Items_Action>().enabled = false;

        cur_Equip[item.equipNum] = item;    
        item.equipped = true;
        item.transform.GetChild(0).gameObject.SetActive(true);
        parts_Player[item.equipNum].sharedMesh = item.mesh;

        Manager.instance.manager_Inven.itemInfoFrame.SetActive(false);
        Manager.instance.manager_Inven.itemInfoFrame.SetActive(true);
    }

    public void ReleaseBtn()
    {
        Items_Info item = Manager.instance.manager_Inven.selectedItem.GetComponent<Items_Info>();

        Destroy(slot_Equip[item.equipNum].GetChild(1).gameObject);

        item.equipped = false;
        item.transform.GetChild(0).gameObject.SetActive(false);
        cur_Equip[item.equipNum] = null;
        parts_Player[item.equipNum].sharedMesh = originMesh[item.equipNum];

        Manager.instance.manager_Inven.itemInfoFrame.SetActive(false);
        Manager.instance.manager_Inven.itemInfoFrame.SetActive(true);
    }
}
```

(10) 위 스크립트를 Manager_Inven 오브젝트의 컴포넌트로 넣어준다.

![image](https://user-images.githubusercontent.com/101051124/171602304-33a479f3-89b4-4fc2-bc61-eb987c9300e3.png)

위와 같이 프로퍼티를 할당한다.

(11) 장착 버튼과 해제 버튼에 이벤트를 넣어준다.

이제 게임을 실행하면 이번 챕터에 맞는 기능을 한다.