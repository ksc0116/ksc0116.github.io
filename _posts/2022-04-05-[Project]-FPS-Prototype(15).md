---
layout: post
title:  "[Project] FPS Prototype(15)"
subtitle:   ""
categories: Unity
comments: true
---

<br>

아이템 오브젝트를 만들어 보자.

# 체력 회복 아이템

(1) 체력 회복 아이템에 적용 할 Material(HealthItem)을 만든다.

(2) 체력 회복 아이템 제작을 위해 Cube(healthItem)을 만든다. 위치(0 / 0.3 / 3), 크기(0.5 / 0.5 / 0.5), IsTrigger(true), Materials(HealthItem)

(3) HealthItem 오브젝트를 프리팹으로 만든다. Item태그를 새로 만들고 HealthItem프리팹에 태그를 Item으로 바꾼다.

(4) 맵에 임의로 HealthItem프리팹을 2개 배치한다.

(5) 체력 회복 이펙트에 사용할 Material(HealthEffect)을 만든다.

* Shader : Particles / Standard Unlit
* Rendering Mode : Fade
* Texture : HealthEffect

(6) 체력 회복 이펙트 제작을 위해 파티클시스템(HealthEffect)을 하나 만든다.

* Main

  * Duration : 2

  * Looping : false

  * Start LifeTime : 2

  * Start Speed : 2

  * Start Size : 0.1 - 0.5

  * Start Color : (23 / 212 / 23 / 255)

  * Max Particles : 50

* Emission

  * Rate over Time : 50

* Shape

  * Radius : 0.1
  * Radius :  Thickness : 0.1
  * Rotation (270 / 0 / 0)

* Size over LifeTime
  * Size : 1.0 to 0.0 curve line graph

* Renderer
  * Material : HealthEffect

(7) 체력 회복 이펙트 오브젝트를 프리팹으로 만든다.

(8) HealthEffect프리팹에 ParticleAutoDestroyerByTime스크립트를 추가한다.

(9) Status스크립트를 수정한다.(HP 증가 함수를 추가한다.)

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Events;

[System.Serializable]
public class HPEvent : UnityEvent<int,int> {  }
public class Status : MonoBehaviour
{
    [HideInInspector]
    public HPEvent onHPEvent=new HPEvent();

    [Header("Walk, Run Speed")]
    [SerializeField]
    private float walkSpeed;
    [SerializeField]
    private float runSpeed;

    [Header("HP")]
    [SerializeField]
    private int maxHP = 100;
    private int currentHP;

    public float WalkSpeed => walkSpeed;
    public float RunSpeed => runSpeed;

    public int CurrentHP => currentHP;
    public int MaxHP => maxHP;
    
    private void Awake()
    {
        currentHP = maxHP;
    }

    public bool DecreaseHP(int damage)
    {
        int previousHP = currentHP;
    
        currentHP=currentHP- damage> 0 ? currentHP - damage : 0;
    
        onHPEvent.Invoke(previousHP,currentHP);
    
        if (currentHP == 0)
        {
            return true;
        }
    
        return false;
    }

    /*●*/public void IncreaseHP(int hp)
    /*●*/{
    /*●*/    int previousHP=currentHP;
    /*●*/    currentHP=currentHP+hp > maxHP ? maxHP : currentHP + hp;
    /*●*/
    /*●*/    onHPEvent.Invoke(previousHP,currentHP);
    /*●*/}
}
```

(10) PlayerHUD스크립트를 수정한다.(체력이 증가하면 빨간색 화면이 나오지 않도록 수정)

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
    [SerializeField]
    private Status status;      // 플레이어의 상태 (이동속도, 체력)

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

    [Header("Magazine")]
    [SerializeField]
    private GameObject magazineUIPrefab;    // 탄창 UI 프리펩
    [SerializeField]
    private Transform magazineParent;       // 탄창 UI가 배치되는 Panel
    
    private List<GameObject> magazineList;      // 탄창 UI 리스트

    [Header("HP & BloodScreen UI")]
    [SerializeField]
    private TextMeshProUGUI textHP;     // 플레이어의 체력을 출력하는 text
    [SerializeField]
    private Image imageBloodScreen;     // 플레이어가 공격받았을 때 화면에 표시되는 Image
    [SerializeField]
    private AnimationCurve curveBloodScreen;

    private void Awake()
    {
        SetupWeapon();
        SetupMagazine();

        // 메소드가 등록되어 있는 이벤트 클래스 (weapon.xx)의
        // Invoke() 메소드가 호출될 때 등록된 메소드(매개변수)가 실행된다.
        weapon.onAmmoEvent.AddListener(UpdateAmmoHUD);
        weapon.onMagazineEvent.AddListener(UpdateMagazineHUD);
        status.onHPEvent.AddListener(UpdateHPHUD);
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
    
   private void SetupMagazine()
   {
       // weapon애 등록되어 있는 최대 탄창 개수만큼 Image Icon을 생성
       // magazineParent 오브젝트의 자식으로 등록 후 모두 비활성화/리스트에 저장
       magazineList=new List<GameObject>(); 
       for(int i=0;i<weapon.MaxMagazine;i++)
       {
           GameObject clone = Instantiate(magazineUIPrefab);
           clone.transform.SetParent(magazineParent);
           clone.SetActive(false);
   
           magazineList.Add(clone);
       }
       // weapon에 등록되어 있는 현재 탄창 개수만큼 오브젝트 활성화
       for(int i=0;i<weapon.CurrentMagazine;i++)
       {
           magazineList[i].SetActive(true);
       }
   }

    private void UpdateMagazineHUD(int currentmagazine)
    {
        // 전부 비활성화하고, currentMagazine 개수만큼 활성화
        for(int i=0; magazineList.Count > i; i++)
        {
            magazineList[i].SetActive(false);
        }
        for(int i = 0; i < currentmagazine; i++)
        {
            magazineList[i].SetActive(true);
        }
    }

    private void UpdateHPHUD(int previous,int current)
    {
        textHP.text = "HP " + current;

        // 체력이 증가했을 때는 화면에 빨간색 이미지를 출력하지 않도록 return
        /*●*/if (previous <= current) return;

        if(previous - current > 0)
        {
            StopCoroutine("OnBloodScreen");
            StartCoroutine("OnBloodScreen");
        }
    }
    private IEnumerator OnBloodScreen()
    {
        float percent = 0;
    
        while (percent < 1)
        {
            percent+=Time.deltaTime;
    
            Color color = imageBloodScreen.color;
            color.a=Mathf.Lerp(1,0,curveBloodScreen.Evaluate(percent));
            imageBloodScreen.color=color;
    
            yield return null;
        }
    }
}
```

(11) 아이템 오브젝트가 상속받는 기반 클래스 ItemBase스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public abstract class ItemBase : MonoBehaviour
{
    public abstract void Use(GameObject entity);
}
```

(12) 체력 회복 아이템을 제어하는 ItemMedicBag스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ItemMeicBag : ItemBase
{
    [SerializeField]
    private GameObject hpEffectPrefab;
    [SerializeField]
    private int increaseHP = 50;
    [SerializeField]
    private float moveDistance = 0.2f;
    [SerializeField]
    private float pingpongSpeed = 0.5f;
    [SerializeField]
    private float rotateSpeed = 50;

    private IEnumerator Start()
    {
        float y = transform.position.y;

        while (true)
        {
            // y축을 기준으로 회전
            transform.Rotate(Vector3.up*rotateSpeed*Time.deltaTime);

            // 처음 배치된 위치를 기준으로 y 위치를 위, 아래로 이동
            Vector3 position= transform.position;
            position.y = Mathf.Lerp(y,y+moveDistance,Mathf.PingPong(Time.time*pingpongSpeed,1));
            transform.position=position;    

            yield return null;
        }
    }

    public override void Use(GameObject entity)
    {
        entity.GetComponent<Status>().IncreaseHP(increaseHP);

        Instantiate(hpEffectPrefab,transform.position,Quaternion.identity);

        Destroy(gameObject);
    }
}
```

(13) HealthItem프리팹에 ItemMedicBag컴포넌트를 추가하고, 컴포넌트 변수들을 설정한다.

(14) 플레이어와 다른 오브젝트의 충돌을 처리하기 위해 Player오브젝트 자식으로 빈오브젝트(PlayerCollider)를 만든다.

(15) 충돌처리를 위해 CapsuleCollider,Rigidbody를 추가 및 설정 해준다. IsTrigger(true), Height(2), Use Gravity(false) Constraints(모든 항목 true)

(16) 레이어 충돌 설정을 위해 Physics탭으로 가서 Player와 Player의 충돌을 해제한다.

(17) 플레이어의 충돌을 제어하는 PlayerCollider스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerCollider : MonoBehaviour
{
    private void OnTriggerEnter(Collider other)
    {
        if (other.transform.tag == "Item")
        {
            other.transform.GetComponent<ItemBase>().Use(transform.parent.gameObject);
        }
    }
}
```

(18) PlayerCollider오브젝트에 PlayerCollider스크립트를 추가한다.

이제 게임을 실행하고 체력이 단 상태에서 체력회복 아이템을 먹으면 체력이 회복되고, 체력 회복 이펙트가 생기는 걸 볼 수 있다.

***

# 탄창 회복 아이템

(1) 탄창 회복 아이템 제작을 위해 assault_rifle_01_mag_full FBX를 하이어라키창에  넣고 이름을 MagazineItem으로 바꾼다.

(2) 위치(0 / 0.4 / 3), 크기(3 / 3 / 3), IsTrigger(true)

(3) 탄창 회복 아이템 오브젝트를 프리팹으로 만들고 태그를 Item으로 바꾼다.

(4) 맵에 임의로 2개의 탄창 회복 아이템을 배치한다.

(5) 탄창 회복 이펙트 제작을 위해 파티클시스템(MagazineEffect)을 만든다.

* Main
  * Duration : 1
  * Looping : false
  * Start LifeTime : 1
  * Start Speed : 3
  * max Particles : 20
* Emission
  * Rate over Time : 20
* Shape
  * Shpae : Box
  * Rotation : 270 / 0 / 0
* Renderer
  * Render Mode : Mesh
  * Mesh : big_bullet
  * Material : Guns Material

(6) 탄창 회복 이펙트 오브젝트를 프리팹으로 만든다.

(7) WeaponAssaultRifle스크립트를 수정한다.(탄창 증가 함수 추가)

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.Events;

[System.Serializable]
public class AmmoEvent : UnityEvent<int, int> { }

[System.Serializable]
public class MagazineEvent : UnityEvent<int> { }

public class WeaponAssaultRifle : MonoBehaviour
{
    [HideInInspector]
    public AmmoEvent onAmmoEvent=new AmmoEvent();

    [HideInInspector]
    public MagazineEvent onMagazineEvent=new MagazineEvent();

    [Header("Fire Effects")]
    [SerializeField]
    private GameObject muzzleFlashEffect;      // 총구 이펙트 (On/Off)

    [Header("Spawn Points")]
    [SerializeField]
    private Transform casingSpawnPoint;     // 탄피 생성 위치
    [SerializeField]
    private Transform bulletSpawnPoint;    // 총알 생성 위치

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

    [Header("Aim UI")]
    [SerializeField]
    private Image imageAim;     // default / aim 모드에 따라 Aim 이미지 활성 / 비활성

    private float lastAttackTime = 0;                       // 마지막 발사시간 체크용
    private bool isReload = false;          // 재장전 중인지 체크
    private bool isAttack=false;        // 공격 여부 체크용
    private bool isModeChange = false;      // 모드 전환 여부 체크용
    private float defaultModeFOV = 60f;      // 기본모드에서의 카메라 FOV
    private float aimModeFOV = 30f;          // AIM모드에서의 카메라 FOV

    private AudioSource          audioSource;            // 사운드 재생 컴포넌트
    private PlayerAnimatorController anim;             // 애니메이션 재생 제어
    private CasingMemoryPool casingMemoryPool;      // 탄피 생성 후 활성/비활성 관리
   private ImpactMemoryPool impactMemoryPool;      // 공격 효과 생성 후 활성/비활성 관리
   private Camera mainCamera;      // 광선 발사

    // 외부에서 필요한 정보를 열람하기 위해 정의한 Get 프로퍼티들
    public WeaponName WeaponName => weaponSetting.weaponName;
    public int CurrentMagazine => weaponSetting.currentMagazine;
    public int MaxMagazine=>weaponSetting.maxMagazine;

    private void Awake()
    {
        audioSource = GetComponent<AudioSource>();
        anim = GetComponentInParent<PlayerAnimatorController>();
        casingMemoryPool=GetComponent<CasingMemoryPool>();
        impactMemoryPool=GetComponent<ImpactMemoryPool>();
        mainCamera = Camera.main;

        // 처음 탄창 수는 최대로 설정
        weaponSetting.currentMagazine = weaponSetting.maxMagazine;

        // 처음 탄 수는 최대로 설정
        weaponSetting.currentAmmo=weaponSetting.maxAmmo;
    }

    private void OnEnable()
    {
        // 무기 장착 사운드 재생
        PlaySound(audioClipTakeOutWeapon);
        // 총구 이펙트 오브젝트 비활성화
        muzzleFlashEffect.SetActive(false);

        // 무기가 활성화될 때 해당 무기의 탄창 정보를 갱신한다.
        onMagazineEvent.Invoke(weaponSetting.currentMagazine);

        // 무기가 활성화될 때 해당 무기의 탄 수 정보를 갱신한다.
        onAmmoEvent.Invoke(weaponSetting.currentAmmo,weaponSetting.maxAmmo);

        ResetVariables();
    }

    // 외부에서 공격 시작할 때 StartWeaponAction(0) 메소드 호출
    public void StartWeaponAction(int type = 0)
    {
        // 재장전 중일 때는 무기 액션을 할 수 없다.
        if (isReload == true) return;

        // 모드 전환중이면 무기 액션을 할 수 없다.
        if (isModeChange == true) return;

        // 마우스 왼쪽 클릭 (공격 시작)
        if (type == 0)
        {
            // 연속 공격
            if (weaponSetting.isAutomaticAttack == true)
            {
                isAttack = true;
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
        // 마우스 오른쪽 클릭 (모드 전환)
        else
        {
            // 공격 중일 때는 모드 전환을 할 수 없다
            if (isAttack == true) return;
        
            StartCoroutine("OnModeChange");
        }
    }
    public void StartReload()
    {
        // 현재 재장전 중이면 재장전 불가능
        if(isReload == true || weaponSetting.currentMagazine<=0) return;
    
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
            isAttack=false;
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
            //anim.Play("Fire",-1,0);
            // 무기 애니메이션 재생 (모드에 따라 AimFire or Fire 애니메이션 재생)
            string animation = anim.AimModeIs == true ? "AimFire" : "Fire";
            anim.Play(animation, - 1, 0);
            // TIP) anim.Play("Fire"); : 같은 애니메이션을 반복할 때 중간에 끊지 못하고 재생 완료 후 다시 재생
            // TIP) anim.Play("Fire",-1,0); : 같은 애니메이션을 반복할 때 애니메이션을 끊고 처음부터 다시 재생

            // 총구 이펙트 재생 (default 모드 일 때만 재생)
            if(anim.AimModeIs==false) StartCoroutine("OnMuzzleFlashEffect");
            // 공격 사운드 재생
            PlaySound(audioClipFire);
            // 탄피 생성
            casingMemoryPool.SpawnCasing(casingSpawnPoint.position, transform.right);

            // 광선을 발사해 원하는 위치 공격 (+Impact Effect)
            TwoStepRaycast();
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
            if(audioSource.isPlaying==false && (anim.CurrentAnimationIs("Movement") || anim.CurrentAnimationIs("AimFirePose")))
            {
                isReload=false;

                // 현재 탄창 수를 1 감소시키고, 바뀐 탄창 정보를 Text UI에 업데이트
                weaponSetting.currentMagazine--;
                onMagazineEvent.Invoke(weaponSetting.currentMagazine);

                // 현재 탄 수를 최대로 설정하고, 바뀐 찬 수 정보를 Text UI에 업데이트
                weaponSetting.currentAmmo=weaponSetting.maxAmmo;
                onAmmoEvent.Invoke(weaponSetting.currentAmmo, weaponSetting.maxAmmo);
    
                yield break;
            }
            yield return null;
        }
    }
    private void TwoStepRaycast()
    {
        Ray ray;
        RaycastHit hit;
        Vector3 targetPoint = Vector3.zero;
    
        // 화면 중앙 좌표 (Aim 기준으로 Raycast연산)
        ray = mainCamera.ViewportPointToRay(Vector2.one * 0.5f);
        // 공격 사거리(attackDistance) 안에 부딪히는 오브젝트가 있으면 targetPoint는 광선에 부딪힌 위치
        if(Physics.Raycast(ray,out hit, weaponSetting.attackDistance))
        {
            targetPoint = hit.point;
            impactMemoryPool.SpawnImpact(hit);
    
        }
        // 공격 사거리 안에  부딪히는 오브젝트가 없으면 targetPoint는 최대 사거리 위치
        else
        {
            targetPoint=ray.origin+ray.direction*weaponSetting.attackDistance;
        }
        Debug.DrawRay(ray.origin, ray.direction * weaponSetting.attackDistance, Color.red);
    
        // 첫번째 Raycast연산으로 얻어진 targetPoint를 목표지점으로 설정하고,
        // 총구를 시작지점으로 하여 Raycast 연산
        Vector3 attackDirection = (targetPoint - bulletSpawnPoint.position).normalized;
        if (Physics.Raycast(bulletSpawnPoint.position, attackDirection, out hit, weaponSetting.attackDistance))
        {
            impactMemoryPool.SpawnImpact(hit);

            if (hit.transform.tag == "ImpactEnemy")
            {
                hit.transform.GetComponent<EnemyFSM>().TakeDamage(weaponSetting.damage);
            }
            else if (hit.transform.tag == "InteractionObject")
            {
                hit.transform.GetComponent<InteractionObject>().TakeDamage(weaponSetting.damage);
            }
        }
        Debug.DrawRay(bulletSpawnPoint.position, attackDirection * weaponSetting.attackDistance, Color.blue);
    }
    private IEnumerator OnModeChange()
    {
        float current = 0;
        float percent = 0;
        float time = 0.35f;
    
        anim.AimModeIs = !anim.AimModeIs;
        imageAim.enabled = !imageAim.enabled;
    
        float start = mainCamera.fieldOfView;
        float end = anim.AimModeIs==true? aimModeFOV : defaultModeFOV;
    
        isModeChange = true;
    
        while (percent < 1)
        {
            current+=Time.deltaTime;
            percent=current/time;
    
            // mode에 따라 카메라의 시야각을 변경
            mainCamera.fieldOfView = Mathf.Lerp(start,end,percent);
    
            yield return null;
        }
        isModeChange=false;
    }

    private void ResetVariables()
    {
        isReload = false;
        isAttack = false;   
        isModeChange=false;
    }
    private void PlaySound(AudioClip clip)
    {
        audioSource.Stop();             // 기존에 재생중인 사운드를 정지하고,
        audioSource.clip = clip;        // 새로운 사운드 clip으로 교체 후
        audioSource.Play();             // 사운드 재생
    }
    /*●*/public void IncreaseMagazine(int magazine)
    /*●*/{
    /*●*/    weaponSetting.currentMagazine = weaponSetting.currentMagazine + magazine > MaxMagazine ? MaxMagazine : weaponSetting.currentMagazine + magazine;
    /*●*/
    /*●*/    onMagazineEvent.Invoke(CurrentMagazine);
    /*●*/}
}
```

(8) 탄창 회복 아이템 오브젝트를 제어하는 ItemMagazine스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ItemMagazine : ItemBase
{
    [SerializeField]
    private GameObject magazineEffectPrefab;
    [SerializeField]
    private int increaseMagazine = 2;
    [SerializeField]
    private float rotateSpeed = 50;

    private IEnumerator Start()
    {
        while (true)
        {
            // y축을 기준으로 회전
            transform.Rotate(Vector3.up* rotateSpeed*Time.deltaTime);

            yield return null;
        }
    }

    public override void Use(GameObject entity)
    {
        entity.GetComponentInChildren<WeaponAssaultRifle>().IncreaseMagazine(increaseMagazine);

        Instantiate(magazineEffectPrefab,transform.position,Quaternion.identity);

        Destroy(gameObject);
    }
}
```

(9) MagazineItem프리팹에 ItemMagazine스크립트를  추가한다.

이제 게임을 실행하고 탄창을 소모한 다음 탄창 회복 아이템을 먹으면 탄창 수가 증가하는 모습과 이펙트 재생이 되는 모습을 볼 수 있다. 그리고 최대 탄창 수 이상으로는 증가하지 않는 모습을 볼 수 있다.