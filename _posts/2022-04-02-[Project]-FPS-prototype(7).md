---
layout: post
title:  "[Project] FPS prototype(7)"
subtitle:   ""
categories: Unity
comments: true
---

<br>

무기의 탄창을 설정해보자.

(1) WeaponSetting 스크립트를 수정해준다.(탄창 변수를 선언한다.)

```cs
// 무기의 종류가 여러 종류일 때 공용으로 사용하는 변수들은  구조체로 묶어서 정의하면
// 변수가 추가/삭제될 때 구조체에 선언하기 때문에 추가/삭제에 대한 관리가 용이함

public enum WeaponName {  AssaultRifle=0}

[System.Serializable]
public struct WeaponSetting
{
    public  WeaponName  weaponName;       // 무기 이름
    /*●*/public int currentMagazine;     // 현재 탄창 수
    /*●*/public int maxMagazine;         // 최대 탄창 수
    public int currentAmmo;     // 현재 탄약 수
    public int maxAmmo;     // 최대 탄약수
    public  float            attackRate;             // 공격 속도
    public  float            attackDistance;       // 공격 사거리
    public  bool            isAutomaticAttack;  // 연속 공격 여부
}
// TIP) 구조체는 스택 영역, 클래스는 힙 영역에 메모리 할당된다.
// TIP) [System.Serializable]을 이용해 직렬화 하지 않으면 다른 클래스의 변수로 생성되었을 때 인스펙터 창에 멤버 변수들의 목록이 뜨지 않는다.
```

(2) WeaponAssaultRifle의 maxMagazine을 8로 설정

(3) WeaponAssaultRifle 스크립트를 수정한다.(탄창 수 UI 업데이트)(이벤트 사용)

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Events;

[System.Serializable]
public class AmmoEvent : UnityEvent<int, int> { }

/*●*/[System.Serializable]
/*●*/public class MagazineEvent : UnityEvent<int> { }

public class WeaponAssaultRifle : MonoBehaviour
{
    [HideInInspector]
    public AmmoEvent onAmmoEvent=new AmmoEvent();

    /*●*/[HideInInspector]
    /*●*/public MagazineEvent onMagazineEvent=new MagazineEvent();

    [Header("Fire Effects")]
    [SerializeField]
    private GameObject muzzleFlashEffect;      // 총구 이펙트 (On/Off)

    [Header("Spawn Points")]
    [SerializeField]
    private Transform casingSpawnPoint;     // 탄피 생성 위치

    [Header("Audio Clips")]
    [SerializeField]
    private AudioClip audioClipTakeOutWeapon;           // 무기 장착 사운드
    [SerializeField]
    private AudioClip audioClipFire;        // 공격 사운드
    [SerializeField]
    private AudioClip audioClipReload;      // 재장전 사운드

    [Header("Weapon Setting")]
    [SerializeField]
    private WeaponSetting weaponSetting;              // 무기 설정

    private float lastAttackTime = 0;                       // 마지막 발사시간 체크용
    private bool isReload = false;          // 재장전 중인지 체크

    private AudioSource          audioSource;            // 사운드 재생 컴포넌트
    private PlayerAnimatorController anim;             // 애니메이션 재생 제어
    private CasingMemoryPool casingMemoryPool;      // 탄피 생성 후 활성/비활성 관리

    // 외부에서 필요한 정보를 열람하기 위해 정의한 Get 프로퍼티들
    public WeaponName WeaponName => weaponSetting.weaponName;
    /*●*/public int CurrentMagazine => weaponSetting.currentMagazine;
    /*●*/public int MaxMagazine=>weaponSetting.maxMagazine;

    private void Awake()
    {
        audioSource = GetComponent<AudioSource>();
        anim = GetComponentInParent<PlayerAnimatorController>();
        casingMemoryPool=GetComponent<CasingMemoryPool>();

        // 처음 탄창 수는 최대로 설정
        /*●*/weaponSetting.currentMagazine = weaponSetting.maxMagazine;

        // 처음 탄 수는 최대로 설정
        weaponSetting.currentAmmo=weaponSetting.maxAmmo;
    }

    private void OnEnable()
    {
        // 무기 장착 사운드 재생
        PlaySound(audioClipTakeOutWeapon);
        // 총구 이펙트 오브젝트 비활성화
        muzzleFlashEffect.SetActive(false);

        // 무기가 활성화될 때 해당 무시의 탄창 정보를 갱신한다.
        /*●*/onMagazineEvent.Invoke(weaponSetting.currentMagazine);

        // 무기가 활성화될 때 해당 무기의 탄 수 정보를 갱신한다.
        onAmmoEvent.Invoke(weaponSetting.currentAmmo,weaponSetting.maxAmmo);
    }

    // 외부에서 공격 시작할 때 StartWeaponAction(0) 메소드 호출
    public void StartWeaponAction(int type = 0)
    {
        // 재장전 중일 때는 무기 액션을 할 수 없다.
        if (isReload == true) return;

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
    public void StartReload()
    {
        // 현재 재장전 중이면 재장전 불가능
        /*●*/if(isReload == true || weaponSetting.currentMagazine<=0) return;
    
        // 무기 액션 도중에 'R'키를 눌러 재장전을 시도하면 무기 액션 종료 후 재장전
        StopWeaponAction();
    
        StartCoroutine("OnReload");
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

            // 탄 수가 없으면 공격 불가능
            if (weaponSetting.currentAmmo <= 0)
            {
                return;
            }
            // 공격 시 currentAmmo 1 감소, 탄 수 UI 업데이트
            weaponSetting.currentAmmo--;
            onAmmoEvent.Invoke(weaponSetting.currentAmmo,weaponSetting.maxAmmo);

            // 무기 애니메이션 재생
            anim.Play("Fire",-1,0);
            // TIP) anim.Play("Fire"); : 같은 애니메이션을 반복할 때 중간에 끊지 못하고 재생 완료 후 다시 재생
            // TIP) anim.Play("Fire",-1,0); : 같은 애니메이션을 반복할 때 애니메이션을 끊고 처음부터 다시 재생

            // 총구 이펙트 재생
            StartCoroutine("OnMuzzleFlashEffect");
            // 공격 사운드 재생
            PlaySound(audioClipFire);
            // 탄피 생성
            casingMemoryPool.SpawnCasing(casingSpawnPoint.position, transform.right);
        }
    }
    private IEnumerator OnMuzzleFlashEffect()
    {
        muzzleFlashEffect.SetActive (true);
    
        // 무기의 공격속도보다 빠르게 잠깐 활성화 시켰다가 비활성화 한다
        yield return new WaitForSeconds(weaponSetting.attackRate*0.3f);
    
        muzzleFlashEffect.SetActive(false);
    }

    private IEnumerator OnReload()
    {
        isReload = true;
    
        // 재장전 애니메이션 사운드 재생
        anim.OnReload();
        audioSource.PlayOneShot(audioClipReload,1.0f); // 사운드가 겹쳐도 재생이 되도록
    
        while (true)
        {
            // 사운드가 재생중이 아니고, 현재 애니메이션 Movement이면
            // 재장전 애니메이션(, 사운드) 재생이 종료되었다는 뜻
            if(audioSource.isPlaying==false && anim.CurrentAnimationIs("Movement"))
            {
                isReload=false;

                // 현재 탄창 수를 1 감소시키고, 바뀐 탄창 정보를 Text UI에 업데이트
                /*●*/weaponSetting.currentMagazine--;
                /*●*/onMagazineEvent.Invoke(weaponSetting.currentMagazine);

                // 현재 탄 수를 최대로 설정하고, 바뀐 찬 수 정보를 Text UI에 업데이트
                weaponSetting.currentAmmo=weaponSetting.maxAmmo;
                onAmmoEvent.Invoke(weaponSetting.currentAmmo, weaponSetting.maxAmmo);
    
                yield break;
            }
            yield return null;
        }
    }
    private void PlaySound(AudioClip clip)
    {
        audioSource.Stop();             // 기존에 재생중인 사운드를 정지하고,
        audioSource.clip = clip;        // 새로운 사운드 clip으로 교체 후
        audioSource.Play();             // 사운드 재생
    }
}
```

(4) PlayerHUD스크립트를 수정한다.(탄창 UI가 갱신되도록)

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using TMPro;

public class PlayerHUD : MonoBehaviour
{
    [Header("Components")]
    [SerializeField]
    private WeaponAssaultRifle weapon;      // 현재 정보가 출력되는 무기

    [Header("Weapon Base")]
    [SerializeField]
    private TextMeshProUGUI textWeaponName;     // 무기 이름
    [SerializeField]
    private Image imageWeaponIcon;      // 무기 아이콘
    [SerializeField]
    private Sprite[] spriteWeaponIcons;    // 무기 아이콘에 사용되는 sprite 배열

    [Header("Ammo")]
    [SerializeField]
    private TextMeshProUGUI textAmmo;   // 현재/최대 탄 수 출력 text

    /*●*/[Header("Magazine")]
    /*●*/[SerializeField]
    /*●*/private GameObject magazineUIPrefab;    // 탄창 UI 프리펩
    /*●*/[SerializeField]
    /*●*/private Transform magazineParent;       // 탄창 UI가 배치되는 Panel
    /*●*/
    /*●*/private List<GameObject> magazineList;      // 탄창 UI 리스트

    private void Awake()
    {
        SetupWeapon();
        /*●*/SetupMagazine();

        // 메소드가 등록되어 있는 이벤트 클래스 (weapon.xx)의
        // Invoke() 메소드가 호출될 때 등록된 메소드(매개변수)가 실행된다.
        weapon.onAmmoEvent.AddListener(UpdateAmmoHUD);
        /*●*/weapon.onMagazineEvent.AddListener(UpdateMagazineHUD);
    }
    private void SetupWeapon()
    {
        textWeaponName.text=weapon.WeaponName.ToString();
        imageWeaponIcon.sprite=spriteWeaponIcons[(int)weapon.WeaponName];
    }
    private void UpdateAmmoHUD(int currentAmmo,int maxAmmo)
    {
         textAmmo.text=$"<size=40>{currentAmmo}/</size>{maxAmmo}";
    }
    
    /*●*/private void SetupMagazine()
    /*●*/{
    /*●*/    // weapon애 등록되어 있는 최대 탄창 개수만큼 Image Icon을 생성
    /*●*/    // magazineParent 오브젝트의 자식으로 등록 후 모두 비활성화/리스트에 저장
    /*●*/    magazineList=new List<GameObject>(); 
    /*●*/    for(int i=0;i<weapon.MaxMagazine;i++)
    /*●*/    {
    /*●*/        GameObject clone = Instantiate(magazineUIPrefab);
    /*●*/        clone.transform.SetParent(magazineParent);
    /*●*/        clone.SetActive(false);
    /*●*/
    /*●*/        magazineList.Add(clone);
    /*●*/    }
    /*●*/    // weapon에 등록되어 있는 현재 탄창 개수만큼 오브젝트 활성화
    /*●*/    for(int i=0;i<weapon.CurrentMagazine;i++)
    /*●*/    {
    /*●*/        magazineList[i].SetActive(true);
    /*●*/    }
    /*●*/}

    /*●*/private void UpdateMagazineHUD(int currentmagazine)
    /*●*/{
    /*●*/    // 전부 비활성화하고, currentMagazine 개수만큼 활성화
    /*●*/    for(int i=0; magazineList.Count > i; i++)
    /*●*/    {
    /*●*/        magazineList[i].SetActive(false);
    /*●*/    }
    /*●*/    for(int i = 0; i < currentmagazine; i++)
    /*●*/    {
    /*●*/        magazineList[i].SetActive(true);
    /*●*/    }
    /*●*/}
}
```

(5) 탄창 Icon관리를 위해 Panel(PanelMainMagazine)오브젝트를 PanelWeapon오브젝트의 자식으로 만들고 알맞게 설정한다.

(6) 탄창 Icon들의 위치, 크기를 정렬하기 위해 GridLayOut 컴포넌트를 추가하고 설정을 다음과 같이 한다.

* Cell Size : 10 / 30
* Spacing : 10 / 0
* Child Alignment : Middle Center
* Constraint : Fixed Low Count
  * Constraint Count : 1

(7) 탄창 Icon출력을 위해 Image(ImageMagazine) UI를 만들고 탕창 스프라이트를 입히고 프리펩으로 만든다.

(8) PlayerHUD오브젝트에 ImageMagazine프리펩이랑 PanelMainMagazine을 넣어준다.

이제 게임을 시작하면 다음과 같이 탄창 수가 출력되는 모습을 볼 수 있다.

![image](https://user-images.githubusercontent.com/101051124/161361658-158cb584-110f-49d5-967d-46562de6f676.png)