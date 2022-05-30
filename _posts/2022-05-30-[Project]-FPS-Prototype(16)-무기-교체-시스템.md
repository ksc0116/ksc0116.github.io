---
layout: post
title:  "[Project] FPS Prototype(16) 무기 교체 시스템"
subtitle:   ""
categories: Unity
comments: true
---

<br>

무기 교체 시스템을 만들어 보자. 

# 무기 관리를 위한 기반 클래스 WeaponBase

1.무기의 종류가 다양해지기 때문에 무기를 하나의 타입으로 통일하기 위해 모든 무기가 상속받는 기반클래스 WeaponBase스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public enum WeaponType { Main = 0, Sub, Melee, Throw }

[System.Serializable]
public class AmmoEvent : UnityEngine.Events.UnityEvent<int, int> { }
[System.Serializable]
public class MagazineEvent : UnityEngine.Events.UnityEvent<int> { }

public abstract class WeaponBase : MonoBehaviour
{
    [Header("[WeaponBase]")]
    [SerializeField]
    protected WeaponType weaponType;        // 무기 종류
    [SerializeField]
    protected WeaponSetting weaponSetting;      // 무기 설정

    protected   float lastAttackTime = 0;     // 마지막 발사시간 체크용
    protected   bool isReload = false;        // 재장전 중인지 체크
    protected   bool isAttack = false;        // 공격 여부 체크용
    protected   AudioSource audioSource;      // 사운드 재생 컴포넌트
    protected   PlayerAnimatorController anim;        // 애니메이션 재생 제어

    // 외부에서 이벤트 함수 등록을 할 수 있도록 public 선언
    [HideInInspector]
    public AmmoEvent onAmmoEvent=new AmmoEvent();
    [HideInInspector]
    public MagazineEvent onMagazineEvent=new MagazineEvent();

    // 외부에서 필요한 정보를 열람하기 위해 정의한 Get Property's
    public PlayerAnimatorController Animator => anim;
    public WeaponName WeaponName => weaponSetting.weaponName;
    public int CurrentMagazine=>weaponSetting.currentMagazine;
    public int MaxMagazine => weaponSetting.maxMagazine;

    public abstract void StartWeaponAction(int type = 0);
    public abstract void StopWeaponAction(int type = 0);
    public abstract void StartReload();

    protected void PlaySound(AudioClip clip)
    {
        audioSource.Stop();
        audioSource.clip = clip;
        audioSource.Play();
    }

    protected void Setup()
    {
        audioSource = GetComponent<AudioSource>();
        anim = GetComponent<PlayerAnimatorController>();
    }
}
```

<span style='background-color: #fff5b1'>여기서 중요한 것은 모든 무기가 추상 메소드로 선언한 함수들은 모든 무기에 정의되어야 하는 함수이지만 재정의가 필요하므로 추상 메소드로 만든것이고 그렇기 때문에 WeaponBase클래스가 추상 클래스가 된 것이다.</span>

# WeaponBase를 상속받는 파생 클래스 WeaponAssaultRifle

1.WeaponBase클래스를 상속받기 때문에 WeaponBase 클래스에 작성된 변수, 메소드는 삭제, WeaponBase 상속, WeaponBase에 있는 추상 메소드를 재정의(override 붙이기)

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.Events;

public class WeaponAssaultRifle : /*●*/WeaponBase
{
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

    [Header("Aim UI")]
    [SerializeField]
    private Image imageAim;     // default / aim 모드에 따라 Aim 이미지 활성 / 비활성

    private bool isModeChange = false;      // 모드 전환 여부 체크용
    private float defaultModeFOV = 60f;      // 기본모드에서의 카메라 FOV
    private float aimModeFOV = 30f;          // AIM모드에서의 카메라 FOV

    private CasingMemoryPool casingMemoryPool;      // 탄피 생성 후 활성/비활성 관리
   private ImpactMemoryPool impactMemoryPool;      // 공격 효과 생성 후 활성/비활성 관리
   private Camera mainCamera;      // 광선 발사

    private void Awake()
    {
        /*●*/ // 기반 클래스의 초기화를 위한 Setup() 메소드 호출
        /*●*/base.Setup();

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
    public override void StartWeaponAction(int type = 0)
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
    public override void StartReload()
    {
        // 현재 재장전 중이면 재장전 불가능
        if(isReload == true || weaponSetting.currentMagazine<=0) return;
    
        // 무기 액션 도중에 'R'키를 눌러 재장전을 시도하면 무기 액션 종료 후 재장전
        StopWeaponAction();
    
        StartCoroutine("OnReload");
    }
    // 외부에서 공격 종료할 때 StopWeaponAction(0) 메소드 호출
    public override void StopWeaponAction(int type = 0)
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

   public void IncreaseMagazine(int magazine)
   {
       weaponSetting.currentMagazine = weaponSetting.currentMagazine + magazine > MaxMagazine ? MaxMagazine : weaponSetting.currentMagazine + magazine;
   
       onMagazineEvent.Invoke(CurrentMagazine);
   }
}
```

2.여러 종류의 무기를 소지하고 현재 활성화 된 무기에 따라 애니메이션을 재생하기 위해 Player 오브젝트에 있는 PlayerAnimatorController를 삭제한다.

![image](https://user-images.githubusercontent.com/101051124/170933980-8fbc04a0-5804-4a2c-86d9-9ae3efbd9737.png)

3.무기 오브젝트에 PlayerAnimatorController컴포넌트를 추가한다.

![image](https://user-images.githubusercontent.com/101051124/170934235-f6b7a088-6c6a-4925-bb01-983ce02628cc.png)

4.소총의 WeaponType을 Main으로 설정한다.

![image](https://user-images.githubusercontent.com/101051124/170934405-b0661954-664a-4f24-af48-54f7c6f6487d.png)

5.PlayerController스크립트를 수정한다.(기존에는 PlayerAnimatorController 컴포넌트가 Player 오브젝트의 컴포넌트로 존재했지만 무기의 종류가 다양해지면서 각 무기에 PlayerAnimatorController 컴포넌트를 추가하고, 무기에 접근해서 애니메이션을 제어한다.)

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    [Header("Input KeyCodes")]
    [SerializeField]
    private KeyCode keyCodeRun = KeyCode.LeftShift;            // 달리기 키
    [SerializeField]
    private KeyCode keyCodeJump = KeyCode.Space;            // 점프 키
    [SerializeField]
    private KeyCode keyCodeReload = KeyCode.R;          // 탄 재장전 키


    [Header("Audio Clips")]
    [SerializeField]
    private AudioClip audioClipWalk;           // 걷기 사운드
    [SerializeField]
    private AudioClip audioClipRun;            // 달리기 사운드

    private RotateToMouse rotateToMouse;        // 마우스 이동으러 카메라 회전
    private MovementCharacterController movement;      // 키보드 입력으로 플레이어 이동, 점프
    private Status status;      // 이동속도 등의 플레이어 정보
    private AudioSource audioSource;        // 사운드 재생 제어
    private WeaponAssaultRifle weapon;      // 무기를 이용한 공격 제어

    private void Awake()
    {
        // 마우스 커서를 보이지 않게 설정하고, 현재 위치에 고정 시킨다.
        Cursor.visible = false;
        Cursor.lockState = CursorLockMode.Locked;

        rotateToMouse = GetComponent<RotateToMouse>();
        movement=GetComponent<MovementCharacterController>();
        status=GetComponent<Status>();
        audioSource=GetComponent<AudioSource>();
        weapon=GetComponentInChildren<WeaponAssaultRifle>();
    }
    private void Update()
    {
        UpdateRotate();
        UpdateMove();
        UpdateJump();
        UpdateWeaponAction();
    }
    private void UpdateRotate()
    {
        float mouseX = Input.GetAxis("Mouse X");
        float mouseY = Input.GetAxis("Mouse Y");

        rotateToMouse.UpdateRotate(mouseX, mouseY);
    }
    private void UpdateMove()
    {
        float hAxis = Input.GetAxisRaw("Horizontal");
        float vAxis = Input.GetAxisRaw("Vertical");

        // 이동중 일 때 (걷기 or 뛰기)
        if(hAxis != 0 || vAxis != 0)
        {
            bool isRun = false;

            // 옆이나 뒤로 이동할 때는 달릴 수 없다.
            if(vAxis>0) isRun=Input.GetKey(keyCodeRun);

            movement.MoveSpeed = isRun ? status.RunSpeed : status.WalkSpeed;
            weapon.Animator.MoveSpeed = isRun == true ? 1 : 0.5f;
            audioSource.clip = isRun ? audioClipRun : audioClipWalk;

            // 방향키 입력 여부는 매 프레임 확인하기 때문데
            // 재생중일 때는 다시 재생하지 않도록 isPlaying으로 체크해서 재생ㅇ
            if (audioSource.isPlaying==false)
            {
                audioSource.loop = true;
                audioSource.Play();
            }
        }
        else
        {
            movement.MoveSpeed = 0;
            weapon.Animator.MoveSpeed = 0;

            // 멈췄을 때 사운드가 재생중이면 정지
            if (audioSource.isPlaying== true)
            {
                audioSource.Stop();
            }
        }

        movement.MoveTo(new Vector3(hAxis, 0, vAxis));
    }
   private void UpdateJump()
   {
       if (Input.GetKeyDown(keyCodeJump))
       {
           movement.Jump();
       }
   }
    private void UpdateWeaponAction()
    {
        if (Input.GetMouseButtonDown(0))
        {
            weapon.StartWeaponAction();
        }
        else if (Input.GetMouseButtonUp(0))
        {
            weapon.StopWeaponAction();
        }

        if (Input.GetMouseButtonDown(1))
        {
            weapon.StartWeaponAction(1);
        }
        else if (Input.GetMouseButtonUp(1))
        {
            weapon.StopWeaponAction(1);
        }

        if (Input.GetKeyDown(keyCodeReload))
        {
            weapon.StartReload();
        }
    }
    public void TakeDamage(int damage)
    {
        bool isDie = status.DecreaseHP(damage);
    
        if(isDie == true)
        {
            Debug.Log("GameOver");
        }
    }
}
```

# 무기 교체 시스템

1.WeaponSetting 스크립트를 수정한다.

```cs
// 무기의 종류가 여러 종류일 때 공용으로 사용하는 변수들은  구조체로 묶어서 정의하면
// 변수가 추가/삭제될 때 구조체에 선언하기 때문에 추가/삭제에 대한 관리가 용이함

public enum WeaponName {  AssaultRifle=0, /*●*/Revolver, /*●*/Combatknife, /*●*/HandGrenade }

[System.Serializable]
public struct WeaponSetting
{
    public  WeaponName  weaponName;       // 무기 이름
    public int damage;      // 무기 공격력
    public int currentMagazine;     // 현재 탄창 수
    public int maxMagazine;         // 최대 탄창 수
    public int currentAmmo;     // 현재 탄약 수
    public int maxAmmo;     // 최대 탄약수
    public  float            attackRate;             // 공격 속도
    public  float            attackDistance;       // 공격 사거리
    public  bool            isAutomaticAttack;  // 연속 공격 여부
}
// TIP) 구조체는 스택 영역, 클래스는 힙 영역에 메모리 할당된다.
// TIP) [System.Serializable]을 이용해 직렬화 하지 않으면 다른 클래스의 변수로 생성되었을 때 인스펙터 창에 멤버 변수들의 목록이 뜨지 않는다.
```

2.보조 무기 리볼버를 제어하는 WeaponRevolver스크립트를 만든다.(무기 교체만 테스트하기 위해 푀소한의 코드만 일단 넣음)

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class WeaponRevolver : WeaponBase
{
    public override void StartReload()
    {
    }

    public override void StartWeaponAction(int type = 0)
    {
    }

    public override void StopWeaponAction(int type = 0)
    {
    }
}
```

3.보조 무기 리볼버를 만든다.

(1) arms_handgun_01를 Player 오브젝트의 자식으로 만든다

(2) Position(0, -0.06, 0)

(3) WeaponRevolver 컴포넌트를 추가하고 다음과 같이 셋팅한다.

![image](https://user-images.githubusercontent.com/101051124/170942618-02092bda-44ef-4be7-a15b-f79e2178ffd2.png)

4.근접 무기 단검을 제어하는 WeaponKnife 스크립트를 만든다.(무기 교체만 테스트하기 위해 푀소한의 코드만 일단 넣음)

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class WeaponKnife : WeaponBase
{
    public override void StartReload()
    {
    }

    public override void StartWeaponAction(int type = 0)
    {
    }

    public override void StopWeaponAction(int type = 0)
    {
    }
}
```

+ arms_assault_rifle_01를 Player의 자식으로 넣고 이름을(arms_combat_knife)로 바꾼다.(나이프 모델이 없어 소총 오브젝트를 빌려쓴다.)
+ Position(0, -0.1, 0.1)
+ 화면에 총이 보이지 않게 총과 관련된 오브젝트는 비활성화 한다.
+ WeaponKnife 컴포넌트를 추가하고 아래와 같이 셋팅한다.

​	![image](https://user-images.githubusercontent.com/101051124/170944733-7d83337b-9ff5-421a-9fee-0379997de487.png)

5.투척 무기 수류탄을 제어하는 WeaponGrenade 스크립트를 만든다.(무기 교체만 테스트하기 위해 푀소한의 코드만 일단 넣음)

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class WeaponGrenade : WeaponBase
{
    public override void StartReload()
    {
    }

    public override void StartWeaponAction(int type = 0)
    {
    }

    public override void StopWeaponAction(int type = 0)
    {
    }
}
```

+ 단검 오브젝트를 복사하고 이름을 (arms_hand_grenade)로 바꾼다.
+ WeaponGrenage 컴포넌트를 추가하고 다음과 같이 셋팅한다.

![image](https://user-images.githubusercontent.com/101051124/170945739-d816e720-6890-42cc-8630-a5c3a9fbb080.png)

6.PlayerController스크립트를 수정한다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    [Header("Input KeyCodes")]
    [SerializeField]
    private KeyCode keyCodeRun = KeyCode.LeftShift;            // 달리기 키
    [SerializeField]
    private KeyCode keyCodeJump = KeyCode.Space;            // 점프 키
    [SerializeField]
    private KeyCode keyCodeReload = KeyCode.R;          // 탄 재장전 키


    [Header("Audio Clips")]
    [SerializeField]
    private AudioClip audioClipWalk;           // 걷기 사운드
    [SerializeField]
    private AudioClip audioClipRun;            // 달리기 사운드

    private RotateToMouse rotateToMouse;        // 마우스 이동으러 카메라 회전
    private MovementCharacterController movement;      // 키보드 입력으로 플레이어 이동, 점프
    private Status status;      // 이동속도 등의 플레이어 정보
    private AudioSource audioSource;        // 사운드 재생 제어
    /*●*/private WeaponBase weapon;      // 모든 무기가 상속받는 기반 클래스
    private void Awake()
    {
        // 마우스 커서를 보이지 않게 설정하고, 현재 위치에 고정 시킨다.
        Cursor.visible = false;
        Cursor.lockState = CursorLockMode.Locked;

        rotateToMouse = GetComponent<RotateToMouse>();
        movement=GetComponent<MovementCharacterController>();
        status=GetComponent<Status>();
        audioSource=GetComponent<AudioSource>();
    }
    private void Update()
    {
        UpdateRotate();
        UpdateMove();
        UpdateJump();
        UpdateWeaponAction();
    }
    private void UpdateRotate()
    {
        float mouseX = Input.GetAxis("Mouse X");
        float mouseY = Input.GetAxis("Mouse Y");

        rotateToMouse.UpdateRotate(mouseX, mouseY);
    }
    private void UpdateMove()
    {
        float hAxis = Input.GetAxisRaw("Horizontal");
        float vAxis = Input.GetAxisRaw("Vertical");

        // 이동중 일 때 (걷기 or 뛰기)
        if(hAxis != 0 || vAxis != 0)
        {
            bool isRun = false;

            // 옆이나 뒤로 이동할 때는 달릴 수 없다.
            if(vAxis>0) isRun=Input.GetKey(keyCodeRun);

            movement.MoveSpeed = isRun ? status.RunSpeed : status.WalkSpeed;
            weapon.Animator.MoveSpeed = isRun == true ? 1 : 0.5f;
            audioSource.clip = isRun ? audioClipRun : audioClipWalk;

            // 방향키 입력 여부는 매 프레임 확인하기 때문데
            // 재생중일 때는 다시 재생하지 않도록 isPlaying으로 체크해서 재생ㅇ
            if (audioSource.isPlaying==false)
            {
                audioSource.loop = true;
                audioSource.Play();
            }
        }
        else
        {
            movement.MoveSpeed = 0;
            weapon.Animator.MoveSpeed = 0;

            // 멈췄을 때 사운드가 재생중이면 정지
            if (audioSource.isPlaying== true)
            {
                audioSource.Stop();
            }
        }

        movement.MoveTo(new Vector3(hAxis, 0, vAxis));
    }
   private void UpdateJump()
   {
       if (Input.GetKeyDown(keyCodeJump))
       {
           movement.Jump();
       }
   }
    private void UpdateWeaponAction()
    {
        if (Input.GetMouseButtonDown(0))
        {
            weapon.StartWeaponAction();
        }
        else if (Input.GetMouseButtonUp(0))
        {
            weapon.StopWeaponAction();
        }

        if (Input.GetMouseButtonDown(1))
        {
            weapon.StartWeaponAction(1);
        }
        else if (Input.GetMouseButtonUp(1))
        {
            weapon.StopWeaponAction(1);
        }

        if (Input.GetKeyDown(keyCodeReload))
        {
            weapon.StartReload();
        }
    }
    public void TakeDamage(int damage)
    {
        bool isDie = status.DecreaseHP(damage);
    
        if(isDie == true)
        {
            Debug.Log("GameOver");
        }
    }
    /*●*/public void SwitchingWeapon(WeaponBase newWeapon)
    /*●*/{
    /*●*/    weapon = newWeapon;
    /*●*/}
}
```

7.PlayerHUD스크립트를 수정한다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using TMPro;

public class PlayerHUD : MonoBehaviour
{
    /*●*/private WeaponBase weapon;

    [Header("Components")]
    [SerializeField]
    private Status status;      // 플레이어의 상태 (이동속도, 체력)

    [Header("Weapon Base")]
    [SerializeField]
    private TextMeshProUGUI textWeaponName;     // 무기 이름
    [SerializeField]
    private Image imageWeaponIcon;      // 무기 아이콘
    [SerializeField]
    private Sprite[] spriteWeaponIcons;    // 무기 아이콘에 사용되는 sprite 배열
    /*●*/[SerializeField]
    /*●*/private Vector2[] sizeWeaponIcons;      // 무기 아이콘의 UI 크기 배열

    [Header("Ammo")]
    [SerializeField]
    private TextMeshProUGUI textAmmo;   // 현재/최대 탄 수 출력 text

    [Header("Magazine")]
    [SerializeField]
    private GameObject magazineUIPrefab;    // 탄창 UI 프리펩
    [SerializeField]
    private Transform magazineParent;       // 탄창 UI가 배치되는 Panel
    /*●*/[SerializeField]
    /*●*/private int maxMagazineCount;
    
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
        // 메소드가 등록되어 있는 이벤트 클래스 (weapon.xx)의
        // Invoke() 메소드가 호출될 때 등록된 메소드(매개변수)가 실행된다.
        status.onHPEvent.AddListener(UpdateHPHUD);
    }
    /*●*/public void SetupAllWeapons(WeaponBase[] weapons)
    /*●*/{
    /*●*/    SetupMagazine();
    /*●*/
    /*●*/    // 사용 가능한 모든 무기의 이벤트 등록
    /*●*/    for(int i = 0; i < weapons.Length; i++)
    /*●*/    {
    /*●*/        weapons[i].onAmmoEvent.AddListener (UpdateAmmoHUD);
    /*●*/        weapons[i].onMagazineEvent.AddListener(UpdateMagazineHUD);
    /*●*/    }
    /*●*/}
    /*●*/public void SwitchingWeapon(WeaponBase newWeapon)
    /*●*/{
    /*●*/    weapon = newWeapon;
    /*●*/
    /*●*/    SetupWeapon();
    /*●*/}
    private void SetupWeapon()
    {
        textWeaponName.text=weapon.WeaponName.ToString();
        imageWeaponIcon.sprite=spriteWeaponIcons[(int)weapon.WeaponName];
        /*●*/imageWeaponIcon.rectTransform.sizeDelta=sizeWeaponIcons[(int)weapon.WeaponName];
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
       for(int i=0;i<maxMagazineCount;i++)
       {
           GameObject clone = Instantiate(magazineUIPrefab);
           clone.transform.SetParent(magazineParent);
           clone.SetActive(false);
   
           magazineList.Add(clone);
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
        if (previous <= current) return;

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

8.PlayerHUD오브젝트의 PlayerHUD스크립트의 Sprite Weapon Icons, Size Weapon Icons배열에 값을 할당한다.

![image](https://user-images.githubusercontent.com/101051124/170967004-5a3f34ca-b999-4401-ac10-23a45a39ca8c.png)

9.무기 교체 시스템을 제어하는 WeaponSwitchSystem 스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class WeaponSwitchSystem : MonoBehaviour
{
    [SerializeField]
    private PlayerController playerController;
    [SerializeField]
    private PlayerHUD playerHUD;

    [SerializeField]
    private WeaponBase[] weapons;       // 소지중인 무기 4종류

    private WeaponBase currentWeapon;       // 현재 사용중인 무기
    private WeaponBase previousWeapon;      // 직전에 사용했던 무기

    private void Awake()
    {
        // 무기 정보 출력을 위해 현재 소지중인 모든 무기 이벤트 등록
        playerHUD.SetupAllWeapons(weapons);

        // 현재 소지중인 모든 무기를 보이지 않게 설정
        for(int i = 0; i < weapons.Length; i++)
        {
            if (weapons[i].gameObject != null)
            {
                weapons[i].gameObject.SetActive(false);
            }
        }

        // Main 무기를 현재 사용 무기로 설정
        SwitchingWeapon(WeaponType.Main);
    }

    private void Update()
    {
        UpdateSwitch();
    }
    private void UpdateSwitch()
    {
        if (!Input.anyKeyDown) return;

        // 1~4 숫자키를 누르면 무기 교체
        int inputIndex = 0;
        if(int.TryParse(Input.inputString,out inputIndex) && (inputIndex>0 && inputIndex < 5))
        {
            SwitchingWeapon((WeaponType)(inputIndex-1));
        }
    }
    private void SwitchingWeapon(WeaponType weaponType)
    {
        // 교체 가능한 무기가 없으면 종료
        if (weapons[(int)weaponType] == null)
        {
            return;
        }

        // 현재 사용중인 무기가 있으면 이전 무기 정보에 저장
        if (currentWeapon != null)
        {
            previousWeapon = currentWeapon;
        }

        // 무기 교체
        currentWeapon=weapons[(int)weaponType];

        // 현재 사용중인 무기로 교체하려고 할 때 종료
        if (currentWeapon == previousWeapon)
        {
            return;
        }

        // 무기를 사용하는 PlayerController, PlayerHUD에 현재 무기 정보 전달
        playerController .SwitchingWeapon(currentWeapon);
        playerHUD.SwitchingWeapon(currentWeapon);

        if(previousWeapon != null)
        {
            previousWeapon.gameObject.SetActive(false);
        }
        // 현재 사용하는 무기 활성화
        currentWeapon.gameObject.SetActive(true);
    }
}
```

10.Player 오브젝트에 WeaponSwitchSystem을 컴포넌트로 추가하고 다음과 같이 값을 할당해준다.

![image](https://user-images.githubusercontent.com/101051124/170972868-feead4f8-b8d4-464a-8fc0-aa716752b3e3.png)

이제 게임을 실행하면 내가 누른 숫자키에 맞게 무기가 교체 되는 것을 볼 수 있다.