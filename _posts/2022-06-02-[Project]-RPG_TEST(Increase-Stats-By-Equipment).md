---
layout: post
title:  "[Project] RPG_TEST(Increase Stats By Equipment)"
subtitle:   ""
categories: Unity
comments: true
---

<br>

# 장비착용으로 플레이어의 스텟 변경하기

(1) Equip 스크립트를 수정한다.(플레이어의 스텟 정보를 가져오고 장착중인 무기에 따라 스텟 변화 하기 위해서)

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Equip : MonoBehaviour
{
    [Header("[Player]")]
    /*●*/public PlayerState player;
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

            /*●*/ReduceStats(cur_Equip[item.equipNum]);
        }

        GameObject item_Slot = Instantiate(item.gameObject, slot_Equip[item.equipNum]);
        item_Slot.GetComponent<Items_Action>().enabled = false;

        cur_Equip[item.equipNum] = item;    
        item.equipped = true;
        item.transform.GetChild(0).gameObject.SetActive(true);
        parts_Player[item.equipNum].sharedMesh = item.mesh;

        /*●*/IncreaseStats(cur_Equip[item.equipNum]);

        Manager.instance.manager_Inven.itemInfoFrame.SetActive(false);
        Manager.instance.manager_Inven.itemInfoFrame.SetActive(true);
    }

    public void ReleaseBtn()
    {
        Items_Info item = Manager.instance.manager_Inven.selectedItem.GetComponent<Items_Info>();

        Destroy(slot_Equip[item.equipNum].GetChild(1).gameObject);

        item.equipped = false;
        item.transform.GetChild(0).gameObject.SetActive(false);

        /*●*/ReduceStats(cur_Equip[item.equipNum]);

        cur_Equip[item.equipNum] = null;
        parts_Player[item.equipNum].sharedMesh = originMesh[item.equipNum];

        Manager.instance.manager_Inven.itemInfoFrame.SetActive(false);
        Manager.instance.manager_Inven.itemInfoFrame.SetActive(true);
    }

   /*●*/void IncreaseStats(Items_Info item)
   /*●*/{
   /*●*/    player.hp += item.hpBonus;
   /*●*/    player.hp_Cur+=item.hpBonus;
   /*●*/    player.atk+=item.atkBonus;
   /*●*/    player.def+=item.defBonus;
   /*●*/    player.cri+=item.criBonus;
   /*●*/    
   /*●*/     Manager.instance.manager_Inven.charInfoFrame.SetActive(false);
   /*●*/     Manager.instance.manager_Inven.charInfoFrame.SetActive(true);
   /*●*/}
    
    /*●*/void ReduceStats(Items_Info item)
    /*●*/{
    /*●*/    player.hp -= item.hpBonus;
    /*●*/    player.hp_Cur -= item.hpBonus;
    /*●*/    player.atk -= item.atkBonus;
    /*●*/    player.def -= item.defBonus;
    /*●*/    player.cri -= item.criBonus;
    /*●*/
    /*●*/    Manager.instance.manager_Inven.charInfoFrame.SetActive(false);
    /*●*/    Manager.instance.manager_Inven.charInfoFrame.SetActive(true);
    /*●*/}
}
```

(2) 에디터에서 Manager_Inven에 있는 Equip 컴포넌트에 Player 변수를 할당한다.

게임을 실행해보면 내가 장착중인 아이템에 맞게 스텟이 변화하는걸 볼 수 있다.