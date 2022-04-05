---
layout: post
title:  "[Project] FPSPrototype(13)"
subtitle:   ""
categories: Unity
comments: true
---

<br>

상호작용 오브젝트 제작을 해보자.

# 기본 드럼통

게임 맵에 상호작용이 가능한 오브젝트들을 배치하고 플레이어가 오브젝트를 공격했을 때 오브젝트의 종류에 따라 다른 반응을 하게 된다.

(1) 기본 드럼통 오브젝트에 적용 할 Material(NormalBarrel)을 만든다. Color(24 / 65 / 24 / 255), Metallic(1)

(2) 기본 드럼통 제작을 위해 Cylinder(NormalBarrel)오브젝트를 만든다. NormalBarrel머테리얼 추가하고, 중력, 물리처리를 위해 Rigidbody추가한다.

(3) NormalBarrel오브젝트를 프리팹으로 만들고 맵에 임의로 NormalBarrel프리팹을 2개 배치해준다.

(4) InteractionObject태그를 만들고 NormalBarrel 프리팹에 태그를 바꿔준다.

(5) 상호작용 오브젝트를 타격했을 때 타격 이펙트를 생성하도록 ImpactNormal프리팹을 복제해서 이름을 ImpactInteractionObject로 바꾼다.

(6) ImpactmemoryPool스크립트를 수정한다.(상호작용 오브젝트에서 생성되는 이펙트 설정)

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public enum ImpactType { Normal=0, Obstacle, Enemy, /*●*/InteractionObject,}

public class ImpactMemoryPool : MonoBehaviour
{
    [SerializeField]
    private GameObject[] impactPrefab;      // 피격 이펙트
    private MemoryPool[] memoryPool;        // 피격 이펙트 메모리풀

    private void Awake()
    {
        // 피격 이펙트가 여러 종류 이면 종류별로 메모리풀 생성
        memoryPool = new MemoryPool[impactPrefab.Length];
        for(int i = 0; i < impactPrefab.Length; i++)
        {
            memoryPool[i]=new MemoryPool(impactPrefab[i]);
        }
    }

    public void SpawnImpact(RaycastHit hit)
    {
        // 부딪힌 오브젝트의 Tag 정보에 따라 다르게 처리
        if(hit.transform.tag == "ImpactNormal")
        {
            OnSpawnImpact(ImpactType.Normal, hit.point, Quaternion.LookRotation(hit.normal));
        }
        else if(hit.transform.tag == "ImpactObstacle")
        {
            OnSpawnImpact(ImpactType.Obstacle, hit.point, Quaternion.LookRotation(hit.normal));
        }
       else if (hit.transform.tag == "ImpactEnemy")
       {
           OnSpawnImpact(ImpactType.Enemy,hit.point,Quaternion.LookRotation(hit.normal));
       }
       /*●*/else if (hit.transform.tag == "InteractionObject")
       /*●*/{
       /*●*/    Color color=hit.transform.GetComponentInChildren<MeshRenderer>().material.color;
       /*●*/    OnSpawnImpact(ImpactType.InteractionObject, hit.point, Quaternion.LookRotation(hit.normal),color);
       /*●*/}
    }

    public void OnSpawnImpact(ImpactType type, Vector3 position, Quaternion rotation,/*●*/Color color = new Color())
    {
        GameObject item=memoryPool[(int)type].ActivatePoolItem();
        item.transform.position=position;
        item.transform.rotation=rotation;
        item.GetComponent<Impact>().Setup(memoryPool[(int)type]);

        /*●*/if (type == ImpactType.InteractionObject)
        /*●*/{
        /*●*/    ParticleSystem.MainModule main= item.GetComponent<ParticleSystem>().main;
        /*●*/    main.startColor=color;
        /*●*/}
    }
}

```

(7) 무기 오브젝트의 ImpactMemoryPool컴포넌트의 ImpactPrefab배열에 ImpactInteractionObject 프리팹을 할당한다.

이제 게임을 시작해서 드럼통을 공격하면 드럼통 색상에 맞게 이펙트가 생성되는 걸 볼 수 있다.

***

# 폭발 드럼통

(1) 폭발 드럼통에 적용할 Material(ExplosionBarrel)을 생성한다. Color(255 / 0 / 0 / 255), Metallic(1)

(2) 폭발 드럼통 제작을 위해 Cylinder(ExplosionBarrel)오브젝트를 만든다.위치(0 / 1 / 0), ExplosionBarrel머테리얼을 입힌다. 중력,물리 처리를 위해 RogodBody추가한다.

(3) ExplosionBarrel오브젝트를 프리팹으로 만들고 태그를 InteractionObject로 바꿔준다.

(4) 맵에 임의로 EcplosionBarrel프리팹을 배치한다.

(5) 폭발 효과에 적용할 Material(Explosion)을 생성한다.

* Shader : particles/Standard Unlit

(6) 폭발 효과를 재생하는 파티클시스템(Explosion)을 하나 만든다.

* particle System Component [Main]
  * Duration : 4
  * Looping : false
  * Start Speed : 8
  * Start Size : 0.1 - 0.3
  * Start Rotation : 0 - 360
  * Start Color(255 / 255 / 0 / 255) - (255 / 123 / 0 / 255)
  * Gravity Modifier : 1
  * Max particles : 200

**Tip)** Duration은 파티클을 방사하는 시간(수명, 재생시간)이고, LifeTime은 파티클의 생존시간으로 Explosion의 파티클은 1초만 재생하기 때문에 LifeTime은 1로 설정하고, Explosion 하위로 오게 될 파티클까지 모두 재생되는 시간이 4초기 때문에 duration은 4로 설정하였다.

* Emission
  * Rate over Time : 0
  * Bursts => Count : 200
* Shape
  * shape : sphere
  * Arc : 180
* Size over LifeTime
  * Size : 1.0 to 0.0 straight line graph
* Renderer
  * Renderer Mode : Mesh
  * Material : Explosion

(7) 폭발효과 이후 등장하는 연기를 재생하는 이펙트 제작을 위해 파티클시스템(Smoke)을 Explosion오브젝트의 자식으로 생성한다.

* Main
  * Duration : 2
  * Looping : false
  * Start Delay : 0.2
  * Start LifeTime : 2
  * Start Speed : 2
  * Start Size : 0.1 - 0.5
  * Start Rotation : 0 - 360
  * Start Color (103 / 103 / 103 / 255)
  * Max Particles : 100

* Emission
  * Rate over Time : 100
* Shape
  * Shape : Sphere
  * Radius Thickness : 0.1
  * Arc : 180
* Velocity over Lifetime
  * Linear(Curve)
  * y축 : 0.0 to 1.0 curve line graph
* Size over Lifetime
  * Size : 1.0 to 0.0 curve line graph
* Renderer
  * Render Mode : Mesh
  * material : Explosion

(8) Explosion오브젝트를 프리팹으로 만들어준다.

(9) 폭발효과가 재생될 때 사운드가 재생되도록 AudioSource컴포넌트를 추가한다.

(10) 파티클 재생이 완료되면 자동 삭제하는 ParticleAutoDestroyerByTime스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ParticleAutoDestroyerByTime : MonoBehaviour
{
    private ParticleSystem particle;

    private void Awake()
    {
        particle = GetComponent<ParticleSystem>();
    }

    private void Update()
    {
        // 파티클 재생중이 아니면 삭제
        if (particle.isPlaying == false)
        {
            Destroy(gameObject);
        }
    }
}
```

**폭발 드럼통을 임시로 2개만 배치했기 때문에 메모리 출을 적용하지 않았지만 맵에 더 많은 개수가 등장할 예정이면 메모리 풀을 적용**

(11) Explosion프리팹에 ParticleAutoDestroyerByTime를 추가해준다.

(12) 모든 상호작용 오브젝트가 상속받는 기반 클래스 InteractionObject클래스를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public abstract class InteractionObject : MonoBehaviour
{
    [Header("Interaction Object")]
    [SerializeField]
    protected int maxHP = 100;
    protected int currentHP;

    private void Awake()
    {
        currentHP = maxHP;
    }

    public abstract void TakeDamage(int damage);
}
```

(13) 폭발 드럼통을 제어하는 ExplosionBarrel스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ExplosionBarrel : InteractionObject
{
    [Header("Explosion Barrel")]
    [SerializeField]
    private GameObject explosionPrefab;
    [SerializeField]
    private float explosionDelayTime = 0.3f;
    [SerializeField]
    private float explosionRadius = 10.0f;
    [SerializeField]
    private float explosionForce = 100.0f;

    private bool isExplode = false;

    public override void TakeDamage(int damage)
    {
        currentHP -= damage;

        if(currentHP<=0 && isExplode == false)
        {
            StartCoroutine("ExplodeBarrel");
        }
    }
    private IEnumerator ExplodeBarrel()
    {
        yield return new WaitForSeconds(explosionDelayTime);

        // 근처의 베럴이 터져서 다시 현재 베럴을 터트리려고 할 때(StackOverflow 방지)
        isExplode = true;

        // 폭발 이펙트 생성
        Bounds bounds=GetComponent<Collider>().bounds;
        Instantiate(explosionPrefab, new Vector3(bounds.center.x, bounds.min.y, bounds.center.z), transform.rotation);

        // 폭발 범위에 있는 모든 오브젝트의 Collider 정보를 받아와 폭발 효과 처리
        Collider[] colliders = Physics.OverlapSphere(transform.position, explosionRadius);
        foreach(Collider hit in colliders)
        {
            // 폭발 범위에 부딪힌 오브젝트가 플레이어일 때 처리
            PlayerController player = hit.GetComponent<PlayerController>();
            if(player != null)
            {
                player.TakeDamage(50);
                continue;
            }

            // 폭발 범위에 부딪힌 오브젝트가 적 캐릭터일 때 처리
            EnemyFSM enemy =hit.GetComponentInParent<EnemyFSM>();
            if (enemy != null)
            {
                enemy.TakeDamage(300);
                continue;
            }

            // 폭발 범위에 부딪힌 오브젝트가 상호작용하는 오브젝트이면 TakeDamage()로 피해를 줌
            InteractionObject interaction =hit.GetComponent<InteractionObject>();
            if(interaction != null)
            {
                interaction.TakeDamage(300);
            }

            // 중력을 가지고 있는 오브젝트이면 힘을 받아 밀려나도록
            Rigidbody rigidbody = hit.GetComponent<Rigidbody>();    
            if(rigidbody != null)
            {
                rigidbody.AddExplosionForce(explosionForce,transform.position,explosionRadius);
            }
        }
        // 베럴 오브젝트 삭제
        Destroy(gameObject);
    }
}
```

(14) ExplosionBarrel프리팹에 ExplosionBarrel스크립트를 넣어준다.

(15) 기본 드럼통을 제어하는 NormalBarrel스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class NormalBarrel : InteractionObject
{
    public override void TakeDamage(int damage)
    {
       
    }
    // TIP) 태그가 "InteractionObject"인 모든 상호작용 오브젝트는 
    // InteractionObject.TakeDamage();를 호출하기 때문에
    // 기본 드럼통도 InteractionObject 클래스를 상속받는 클래스를 제작한다.
    // 기본 드럼통은 설정상 체력이 무한해서 부서지지 않기 때문에 체력이 닳지 않도록 TakeDamage() 내부가 비어있다.
}
```

(16) NormalBarrel프리팹에 NormalBarrel스크립트를 넣어준다.

(17) WeaponAssaultRifle스크립트를 수정한다.(플레이어의 공격에 맞은 상대의 태그가 InteractionObject이면 데미지를 주게끔 수정)

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
            /*●*/else if (hit.transform.tag == "Interactionobject")
            /*●*/{
            /*●*/    hit.transform.GetComponent<InteractionObject>().TakeDamage(weaponSetting.damage);
            /*●*/}
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
}
```

이제 게임을 실행해서 폭발 드럼통을 공격하면 터지면서 폭발 이펙트가 재생되고 주변에 상호작용 오브젝트에게 데미지를 주고 리지드바디를 가지고 있으면 날려보내는 모습을 볼 수 있다.