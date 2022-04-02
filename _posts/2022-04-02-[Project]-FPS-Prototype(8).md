---
layout: post
title:  "[Project] FPS Prototype(8)"
subtitle:   ""
categories: Unity
comments: true
---

<br>

# 물리적 공격 설정 (Two Step Raycast)

(1) 화면 중앙에 Aim을 출력하기 위해 Image(ImageAim) UI를 만들고 aim01을 입힌다, 그럼 다음과 같이 화면 중앙에 에임이 생긴다.

![image](https://user-images.githubusercontent.com/101051124/161362525-6922e76e-85e8-4d69-bff2-84076ef61490.png)

FPS에서 공격을 처리하는 방식은 다양한데 주로 공격을할 때 탄을 실제로 발사하거나 탄이 발사되지 않고 공격하는 순간에 광선을 발사해 공격처리를 한다. 우리는 광선을 쏴서 공격처리를 할 것이다. 탄을 실제로 발사하지 않기 때문에 공격이 적중되었음을 나타내기 위해 타격 이펙트를 재생한다.

(2) 타격 이펙트에 적용할 Mterial(Impact)을 만든다.

* Shader : Particles / Standard Unlit

(3) 일반 타격 이펙트 제작을 위해 파티클(ImpactNormal)을 만든다.

* ParticleSystem Component [Main]
  * Duration : 1
  * Looping : false
  * Start LifeTime : 1
  * Start Size : 0.01 ~ 0.1
  * StartColor (255, 255, 255, 255) , (0, 0, 0, 255)
  * Gravity Modifier: 1
  * Simulation Space : World
  * Max Particles : 100

* ParticleSystem Component [Emission]
  * Rate Over Time : 0
  * Bursts :Count ->50

* ParticleSystem Component [Shape]
  * Shape : Sphere
  * Radius : 0.1
  * Radius Thickness : 0

* ParticleSystem Component [Velocity over LifeTime]
  * Linear (5, 0, 0), (1, 0, 0)

* ParticleSystem Component [Size over LifeTime]
  * 1 - 0으로 점차 감소하는 그래프
* ParticleSystem Component [Renderer]
  * Render Mode : Mesh
  * Mesh : Cube
  * Material : Impact

(4) ImpactNormal을 프리펩으로 만든다.

(5) 장애물 타격 이펙트를 만들기 위해 ImpactNormal 프립펩을 복제한 후 이름을 ImpactObstacle로 바꾼다.

* ParticleSystem Component[Main]
  * Start Color (0, 0, 255, 255), (0, 0, 92, 255)

(6) 타격 이펙트를 제어하는 Impact스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Impact : MonoBehaviour
{
    private ParticleSystem particle;
    private MemoryPool memoryPool;

    private void Awake()
    {
        particle = GetComponent<ParticleSystem>();
    }

    public void Setup(MemoryPool pool)
    {
        memoryPool = pool;
    }

    private void Update()
    {
        // 파티클이 재생중이 아니면 삭제
        if (particle.isPlaying == false)
        {
            memoryPool.DeactivatePoolItem(gameObject);
        }
    }
}
```

(7) ImpactNormal, ImpactObstacle 프리펩에 Impact스크립트를 추가

(8) 타격 이펙트를 메모리풀로 관리하는 ImpactMemoryPool스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public enum ImpactType { Normal=0, Obstacle,}

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
    }

    public void OnSpawnImpact(ImpactType type, Vector3 position, Quaternion rotation)
    {
        GameObject item=memoryPool[(int)type].ActivatePoolItem();
        item.transform.position=position;
        item.transform.rotation=rotation;
        item.GetComponent<Impact>().Setup(memoryPool[(int)type]);
    }
}
```

(9) 무기 오브젝트에 ImpactMemortPool스크립트를 추가하고 Impact프리펩을 넣어준다.

(10) WeaponAssaultRifle스크립트를 수정한다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
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
    /*●*/[SerializeField]
    /*●*/private Transform bulletSpawnPoint;    // 총알 생성 위치

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
    /*●*/private ImpactMemoryPool impactMemoryPool;      // 공격 효과 생성 후 활성/비활성 관리
    /*●*/private Camera mainCamera;      // 광선 발사

    // 외부에서 필요한 정보를 열람하기 위해 정의한 Get 프로퍼티들
    public WeaponName WeaponName => weaponSetting.weaponName;
    public int CurrentMagazine => weaponSetting.currentMagazine;
    public int MaxMagazine=>weaponSetting.maxMagazine;

    private void Awake()
    {
        audioSource = GetComponent<AudioSource>();
        anim = GetComponentInParent<PlayerAnimatorController>();
        casingMemoryPool=GetComponent<CasingMemoryPool>();
        /*●*/impactMemoryPool=GetComponent<ImpactMemoryPool>();
        /*●*/mainCamera = Camera.main;

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

            // 광선을 발사해 원하는 위치 공격 (+Impact Effect)
            /*●*/TwoStepRaycast();
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
    /*●*/private void TwoStepRaycast()
    /*●*/{
    /*●*/    Ray ray;
    /*●*/    RaycastHit hit;
    /*●*/    Vector3 targetPoint = Vector3.zero;
    /*●*/
    /*●*/    // 화면 중앙 좌표 (Aim 기준으로 Raycast연산)
    /*●*/    ray = mainCamera.ViewportPointToRay(Vector2.one * 0.5f);
    /*●*/    // 공격 사거리(attackDistance) 안에 부딪히는 오브젝트가 있으면 targetPoint는 광선에 부딪힌 위치
    /*●*/    if(Physics.Raycast(ray,out hit, weaponSetting.attackDistance))
    /*●*/    {
    /*●*/        targetPoint = hit.point;
    /*●*/    }
    /*●*/    // 공격 사거리 안에  부딪히는 오브젝트가 없으면 targetPoint는 최대 사거리 위치
    /*●*/    else
    /*●*/    {
    /*●*/        targetPoint=ray.origin+ray.direction*weaponSetting.attackDistance;
    /*●*/    }
    /*●*/    Debug.DrawRay(ray.origin, ray.direction * weaponSetting.attackDistance, Color.red);
    /*●*/
    /*●*/    // 첫번째 Raycast연산으로 얻어진 targetPoint를 목표지점으로 설정하고,
    /*●*/    // 총구를 시작지점으로 하여 Raycast 연산
    /*●*/    Vector3 attackDirection = (targetPoint - bulletSpawnPoint.position).normalized;
    /*●*/    if(Physics.Raycast(bulletSpawnPoint.position,attackDirection,out hit, weaponSetting.attackDistance))
    /*●*/    {
    /*●*/        impactMemoryPool.SpawnImpact(hit);
    /*●*/    }
    /*●*/    Debug.DrawRay(bulletSpawnPoint.position,attackDirection*weaponSetting.attackDistance, Color.blue);
    /*●*/}
    private void PlaySound(AudioClip clip)
    {
        audioSource.Stop();             // 기존에 재생중인 사운드를 정지하고,
        audioSource.clip = clip;        // 새로운 사운드 clip으로 교체 후
        audioSource.Play();             // 사운드 재생
    }
}
```

여기서 TwoStepRaycast() 메소드가 광선을 두 번 나눠서 쓰는 이유는 우리가 화면상에서 보고 있는 Aim과 총구가 겨누고 있는 방향에 차이가 있기 때문에 **mainCamera.ViewportPointToRay(Vector2.one * 0.5f);**이렇게 화면 중앙을 관통하는 광선을 발사해 에임 위치를 얻어내서 총구에서 얻어낸 에임 위치를 관통하는 광선을 하나 더 만든다.

(11) 총알이 생성되는 위치 선정을 위해 SpawnPoints의 자식으로 빈 오브젝트(BulletSpawnPoint)를 생성한다. 위치(0 / 0.4 / 0.5) 회전 (-33 / 0 / 0)

(12) WeaponAssaultRifle컴포넌트에 bullet spawn point를 할당해준다.

(13) 광선에 부딪힌 오브젝트는 태그로 구분 ImpactNormal, ImpactObstacle 태그를 만들고 장애물에는 ImpactObstacle태그를 벽, 바닥에는 ImpactNormal로 태그를 바꿔준다.

이제 게임을 실행하고 공격했을 때 공격 사거리 안에 오브젝트가 있으면 타격 이펙트가 오브젝트에 맞게 나온다.