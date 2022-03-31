---
layout: post
title:  "[Project] FPS Prototype(4)"
subtitle:   ""
categories: Unity
comments: true
---

# 탄피

(1) 에셋파일에서 탄피를 하이어라키창에 드래그해서 생성해주고, 탄피를 위한 Physics Material(CasingPhysics)을 만들어준다.

### PhysicsMaterial

![image](https://user-images.githubusercontent.com/101051124/161003633-09715308-9d4a-4ecd-b0a6-502663250b34.png)

* Friction(마찰계수) : 미끄러지거나 미끄러지지 않도록 설정(0~1까지 설정하며, 0에 가까울수록 빙판과 같이 미끄러지는 정도가 심하다)
  * Dynamic Friction(동적 마찰계수) : 이미 움직이고 있을 경우에 적용되는 마찰 계수
  * Static Friction(정적 마찰계수) : 오브젝트가 표면에 붙어있을 때 적용되는 마찰 계수

* Bounciness(탄성도) : 공처럼 통통 튀는 정도를 나타낸다.(0~1까지 값을 설정하는데 이 값은 energy loss 정도를 나타내며, 0이면 loss가 100%로 튕기지 않고,                         1이면 loss가 0%로 이전 힘과 동일하게 튕긴다.) 
* Combine : 서로 다른 오브젝트에 Physics Material이 적용되어 있을 때의 처리 (일반적으로 충돌한다면 두 물체가 동일한 효과를 받게 되지만 두 오브젝트에 적용된 Combine이 다르면 우선순위에 따라 다르게 처리 된다) (우선순위 : Maximum > Mutiply > Minimum > Average)
  * Friction Combine : 2개의 콜라이더를 가진 오브젝트들이 충돌할 때 연산 방식
  * Bounce Combine : 2개의 탄성력을 지닌 오브젝트들이 충돌할 때 연산 방식
    * Average : 2개의 마찰(or 탄성) 값의 평균 값 사용
    * Minimum : 2개의 마찰(or 탄성) 값 중 최소값 사용
    * Maximum : 2개의 마찰(or 탄성) 값 중 최대값 사용
    * Multiply : 2개의 마찰(or 탄성) 값의 곱셈 값 사용

(2) CasingPhtsics의 설정을 다음과 같이 바꾼다, Dynamic Friction : 0.4, Static Friction : 0.4, Bounciness : 1

(3) 탄피 오브젝트에 Capsule콜라이더, RigidBody, AudioSource 컴포넌트를 추가한다.

(4) Capsule콜라이더의 설정을 다음과 같이 바꾼다, Material : CasingPhysics/ Center(0/ 0.001/ 0)/  Radius:0.007/ Height: 0.07

(5) 탄피 오브젝트를 프리펩으로 만든다.

(6) 탄피, 총알과 같이 총에서 생성되는 다양한 오브젝트들의 위치 선정을 위해 빈 오브젝트(SpawnPoints)를 weapon오브젝트의 자식으로 만든다.

(7) 탄피가 생성되는 위치 설정을 위해 SpawnPoints의 자식으로 빈 오브젝트(CasingSpawnPoint)를 만든다. 위치(0.018 / 0.17 / 0.09)

(8) 오브젝트를 파괴하지 않고 비활성화해서 관리하는 MemoryPool스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class MemoryPool 
{
    // 메모리 풀로 관리되는 오브젝트 정보
    private class PoolItem
    {
        public bool isActive;   // "gmaeObject"의 활성화/비활성화 정보
        public GameObject gameObject;       // 화면에 보이는 실제 게임 오브젝트
    }

    private int increaseCount = 5;  // 오브젝트가 부족할 때 Instantiate()로 추가 생성되는 오브젝트 개수
    private int maxCount;               // 현재 리스트에 등록되어 있는 오브젝트 개수
    private int activeCount;            // 현재 게임에 사용되고 있는(활성화) 오브젝트 개수

    private GameObject poolObject;  // 오브젝트 풀링에서 관리하는 게임 오브젝트 프리펩
    private List<PoolItem> poolItemList;    // 관리되는 모든 오브젝트를 저장하는 리스트

    public int MaxCount => maxCount;    // 외부에서 현재 리스트에 등록되어 있는 오브젝트 개수 확인을 위한 프로퍼티
    public int ActiveCount => activeCount; // 외부에서 현재 활성화 되어 있는 오브젝트 개수 확인을 위한 프로퍼티

    public MemoryPool(GameObject poolObject)
    {
        maxCount = 0;
        activeCount = 0;
        this.poolObject = poolObject;

        poolItemList = new List<PoolItem>();

        InstantiateObjects();
    }

    /// <summary>
    /// increaseCount 단위로 오브젝트 생성
    /// </summary>
    public void InstantiateObjects()
    {
        maxCount += increaseCount;
        for(int i=0;i<increaseCount;i++)
        {
            PoolItem poolItem=new PoolItem();

            poolItem.isActive = false;
            poolItem.gameObject=GameObject.Instantiate(poolObject);
            poolItem.gameObject.SetActive(false);

            poolItemList.Add(poolItem);
        }
    }

    /// <summary>
    /// 현재 관리중인(활성/비활성) 모든 오브젝트를 삭제
    /// </summary>
    public void DestroyObjects()
    {
        if (poolItemList == null) return;
        int count = poolItemList.Count;
        for(int i=0; i < count; i++)
        {
            GameObject.Destroy(poolItemList[i].gameObject);
        }
        poolItemList.Clear();
    }

    /// <summary>
    /// poolItemList에 저장되어 있는 오브젝트를 활성화해서 사용
    /// 현재 모든 오브젝트가 사용중이면 Instantiateobjects()로 추가 생성
    /// </summary>
    public GameObject ActivatePoolItem()
    {
        if(poolItemList == null) return null;

        // 현재 생성해서 관리하는 모든 오브젝트의 개수와 현재 활성화 상태인 오브젝트 개수 비교
        // 모든 오브젝트가 활성화 되어 있으면 새로운 오브젝트 필요
        if (maxCount == activeCount)
        {
            InstantiateObjects();
        }

        int count = poolItemList.Count;
        for(int i = 0; i < count; i++)
        {
            PoolItem poolItem = poolItemList[i];

            if (poolItem.isActive == false)
            {
                activeCount++;

                poolItem.isActive = true;
                poolItem.gameObject.SetActive(true);

                return poolItem.gameObject;
            }
        }
        return null;
    }

    /// <summary>
    /// 현재 사용이 완료된 오브젝트를 비활성화 상태로 설정
    /// 현재 활성화 상태인 오브젝트 removeObject를 비활성화 상태로 만들어 보이지 않게 한다.
    /// </summary>
    public void DeactivatePoolItem(GameObject removeObject)
    {
        if (poolItemList == null || removeObject==null) return;

        int count=poolItemList.Count;
        for(int i=0; i < count; i++)
        {
            PoolItem poolItem=poolItemList[i];

            if(poolItem.gameObject == removeObject)
            {
                activeCount--;

                poolItem.gameObject.SetActive(false);
                poolItem.isActive=false;

                return;
            }
        }
    }
    ///<summary>
    /// 게임에 사용중인 모든 오브젝트를 비활성화 상태로 설정
    ///</summary>
    public void DeactivateAllPoolItems()
    {
        if(poolItemList==null) return;

        int count = poolItemList.Count;
        for(int i=0;i< count; i++)
        {
            PoolItem poolItem = poolItemList[i];

            if(poolItem.gameObject!=null && poolItem.isActive == true)
            {
                poolItem.gameObject.SetActive(false);
                poolItem.isActive = false;
            }
        }
    }
}
```

(9) 탄피 오브젝트를 제어하는 Casing스크립트를 작성한다.

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Casing : MonoBehaviour
{
    [SerializeField]
    private float deactivateTime = 5.0f;     // 탄피 등장 후 비활성화 되는 시간
    [SerializeField]
    private float casingSpin = 1.0f;        // 탄피가 회전하는 속력 계수
    [SerializeField]
    private AudioClip[] audioClips;     // 탄피가 부딪혔을 때 재생되는 사운드

    private Rigidbody rigid;
    private AudioSource audioSource;
    private MemoryPool memoryPool;

    public void SetUp(MemoryPool pool,Vector3 direction)
    {
        rigid=GetComponent<Rigidbody>();
        audioSource=GetComponent<AudioSource>();
        memoryPool=pool;

        // 탄피의 이동 속력과 회전 속력 설정
        rigid.velocity = new Vector3(direction.x, 1.0f, direction.z);
        rigid.angularVelocity = new Vector3(Random.Range(-casingSpin, casingSpin),
                                                           Random.Range(-casingSpin, casingSpin),
                                                           Random.Range(-casingSpin, casingSpin));

        // 탄피 자동 비활성화를 위한 코루틴 실행
        StartCoroutine("DeactivateAfterTime");
    }
    private void OnCollisionEnter(Collision collision)
    {
        // 여러 개의 탄피 사운드 중 임의의 사운드 선택
        int index=Random.Range(0,audioClips.Length);
        audioSource.clip = audioClips[index];
        audioSource.Play();
    }
    private IEnumerator DeactivateAfterTime()
    {
        yield return new WaitForSeconds(deactivateTime);

        memoryPool.DeactivatePoolItem(this.gameObject);
    }
}
```

(10) 탄피 프리펩에 Casing스크립트를 추가한다. AudioClips변수에 casing-sound1~5번을 넣어준다.

(11) 메모리풀을 이용해 탄피 생성을 제어하는 CasingMemoryPool스크립트를 만든다.

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CasingMemoryPool : MonoBehaviour
{
    [SerializeField]
    private GameObject casingPrefab;    // 탄피 오브젝트
    private MemoryPool memoryPool;      // 탄피 메모리풀

    private void Awake()
    {
        memoryPool=new MemoryPool(casingPrefab);
    }

    public void SpawnCasing(Vector3 position,Vector3 direction)
    {
        GameObject item = memoryPool.ActivatePoolItem();
        item.transform.position = position;
        item.transform.rotation = Random.rotation;
        item.GetComponent<Casing>().SetUp(memoryPool, direction);
    }
}
```

(12) WeaponAssaultRifle스크립트를 수정한다.

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class WeaponAssaultRifle : MonoBehaviour
{
    [Header("Fire Effects")]
    [SerializeField]
    private GameObject muzzleFlashEffect;      // 총구 이펙트 (On/Off)

    /*●*/[Header("Spawn Points")]
    /*●*/[SerializeField]
    /*●*/private Transform casingSpawnPoint;     // 탄피 생성 위치

    [Header("Audio Clips")]
    [SerializeField]
    private AudioClip audioClipTakeOutWeapon;           // 무기 장착 사운드
    [SerializeField]
    private AudioClip audioClipFire;        // 공격 사운드

    [Header("Weapon Setting")]
    [SerializeField]
    private WeaponSetting weaponSetting;              // 무기 설정

    private float lastAttackTime = 0;                       // 마지막 발사시간 체크용

    private AudioSource          audioSource;            // 사운드 재생 컴포넌트
    private PlayerAnimatorController anim;             // 애니메이션 재생 제어
    private CasingMemoryPool casingMemoryPool;      // 탄피 생성 후 활성/비활성 관리

    private void Awake()
    {
        audioSource = GetComponent<AudioSource>();
        anim = GetComponentInParent<PlayerAnimatorController>();
        /*●*/casingMemoryPool=GetComponent<CasingMemoryPool>();
    }

    private void OnEnable()
    {
        // 무기 장착 사운드 재생
        PlaySound(audioClipTakeOutWeapon);
        // 총구 이펙트 오브젝트 비활성화
        muzzleFlashEffect.SetActive(false);
    }

    // 외부에서 공격 시작할 때 StartWeaponAction(0) 메소드 호출
    public void StartWeaponAction(int type = 0)
    {
        // 마우스 왼쪽 클릭 (공격 시작)
        if (type == 0)
        {
            // 연속 공격
            if (weaponSetting.isAutomaticAttack == true)
            {
                // OnAttack()을 매 프레임 실행
                StartCoroutine("OnAttackLoop");
            }
            // 단발 공격
            else
            {
                // 실제 공격 정의 함수
                OnAttack();
            }
        }
    }

    // 외부에서 공격 종료할 때 StopWeaponAction(0) 메소드 호출
    public void StopWeaponAction(int type = 0)
    {
        // 마우스 왼쪽 해제 (공격 종료)
        if(type == 0)
        {
            StopCoroutine("OnAttackLoop");
        }
    }

    private IEnumerator OnAttackLoop()
    {
        while (true)
        {
            OnAttack();
    
            yield return null;
        }
    }

    public void OnAttack()
    {
        if(Time.time - lastAttackTime > weaponSetting.attackRate)
        {
            // 뛰고있을 땐 공격x
            if (anim.MoveSpeed > 0.5f)
            {
                return;
            }
    
            // 공격주기가 되어야 공격할 수 있도록 하기 위해 현재시간 정보 저장
            lastAttackTime = Time.time;
    
            // 무기 애니메이션 재생
            anim.Play("Fire",-1,0);
            // TIP) anim.Play("Fire"); : 같은 애니메이션을 반복할 때 중간에 끊지 못하고 재생 완료 후 다시 재생
            // TIP) anim.Play("Fire",-1,0); : 같은 애니메이션을 반복할 때 애니메이션을 끊고 처음부터 다시 재생

            // 총구 이펙트 재생
            StartCoroutine("OnMuzzleFlashEffect");
            // 공격 사운드 재생
            PlaySound(audioClipFire);
            // 탄피 생성
            /*●*/casingMemoryPool.SpawnCasing(casingSpawnPoint.position, transform.right);
        }
    }
    private IEnumerator OnMuzzleFlashEffect()
    {
        muzzleFlashEffect.SetActive (true);
    
        // 무기의 공격속도보다 빠르게 잠깐 활성화 시켰다가 비활성화 한다
        yield return new WaitForSeconds(weaponSetting.attackRate*0.3f);
    
        muzzleFlashEffect.SetActive(false);
    }
    private void PlaySound(AudioClip clip)
    {
        audioSource.Stop();             // 기존에 재생중인 사운드를 정지하고,
        audioSource.clip = clip;        // 새로운 사운드 clip으로 교체 후
        audioSource.Play();             // 사운드 재생
    }
}
```

(13) 무기 오브젝트에 CasingMemoryPool스크립트를 넣는다.

(14) 생성된 탄피와 플레이어가 부딪힐 수 있기 때문에 탄피 프리펩의 Layer를 Casing으로 바꾸고 플레이어의 Layer를 Player로 바꾼 다음 프로젝트세팅 -> Physics에서 체크를 해제한다.

![image](https://user-images.githubusercontent.com/101051124/161023717-b31d9037-dabf-4a1e-b578-4bae9d26fc81.png)

이제 게임을 실행하고 공격을 해보면 탄피가 생성되는 모습을 볼 수 있다.