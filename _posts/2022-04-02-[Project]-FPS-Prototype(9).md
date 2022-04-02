---
layout: post
title:  "[Project] FPS Prototype(9)"
subtitle:   ""
categories: Unity
comments: true
---

<br>

# 에임 모드 설정 (Default / Aim Mode)

타겟을 좀 더 가깝게 보는 Aim Mode를 만든다. Aim Mide는 Aim 애니메이션 재생과 함께 목표물이 가깝게 보이도록 카메라의 FOV를 설정한다.

(1) 에임 모드 애니메이션을 관리하기 위한 Sub-State Machine(AimMode)을 만든다.

(2) AimMode내부로 이동해 aim_in / aim_fire_pose / aim_fire / aim_out 애니메이션을 추가한다.

(3) aim_fire의 이름을 AimFire로 설정 ("Fire" 애니메이션과 마찬가지로 코드에서 Animator.Play()로 재생)

(4) bool 타입의 파라미터 isAimMode를 만든다.

(5) BaseLayer에서 Movement에서 AimMode로 가는 Transition을 생성 및 설정한다.

* Movement -> States::aim_in
* Has Exit Time : false
* Conditions : isAimMode - true

(6) AimMode로 들어가서 Transition을 생성 및 설정 해준다.

* aim_in -> aim_fire_pose
* AimFire -> aim_fire_pose
* aim_out -> States::Movement
* Has Exit Time : true
* Condition : -

(7) AimMode Transition 생성 및 설정

* aim_in -> aim_out
* aim_fire_pose -> aim_out
* Has Exit Time : false
* Condition : isAimMode - false

(8) 에임 모드 도중에 재장전을 할 수 있으므로 Transition 생성 및 설정

* aim_fire_pose -> States::reload_out_of_ammo
* Has Exit Time : false
* Conditions : onReload

(9) PlayerAnimator스크립트를 수정한다. 

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerAnimatorController : MonoBehaviour
{
    private Animator anim;
    private void Awake()
    {
        // "Player" 오브젝트 기준으로 자식 오브젝트인 
        // "arms_assault_rifle_01" 오브젝트에 Animator 컴포넌트 있음
        anim=GetComponentInChildren<Animator>();
    }

    public float MoveSpeed
    {
        set=>anim.SetFloat("movementSpeed",value);
        get => anim.GetFloat("movementSpeed");
    }
    public void OnReload()
    {
        anim.SetTrigger("onReload");
    }

    // Assault Rifle 마우스 오른쪽 클릭 액션 (default / aim mode)
    /*●*/public bool AimModeIs
    /*●*/{
    /*●*/    set => anim.SetBool("isAimMode", value);
    /*●*/    get => anim.GetBool("isAimMode");
    /*●*/}

   public void Play(string stateName,int layer, float normalizedTime)
   {
       anim.Play(stateName,layer,normalizedTime);
   }

    public bool CurrentAnimationIs(string name)
    {
        return anim.GetCurrentAnimatorStateInfo(0).IsName(name);
    }
}
```

(10) WeaponAssaultRifle스크립트를 수정한다.

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

    /*●*/[Header("Aim UI")]
    /*●*/[SerializeField]
    /*●*/private Image imageAim;     // default / aim 모드에 따라 Aim 이미지 활성 / 비활성

    private float lastAttackTime = 0;                       // 마지막 발사시간 체크용
    private bool isReload = false;          // 재장전 중인지 체크
    /*●*/private bool isAttack=false;        // 공격 여부 체크용
    /*●*/private bool isModeChange = false;      // 모드 전환 여부 체크용
    /*●*/private float defaultModeFOV = 60;      // 기본모드에서의 카메라 FOV
    /*●*/private float aimModeFOV = 30;          // AIM모드에서의 카메라 FOV

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

        /*●*/ResetVariables();
    }

    // 외부에서 공격 시작할 때 StartWeaponAction(0) 메소드 호출
    public void StartWeaponAction(int type = 0)
    {
        // 재장전 중일 때는 무기 액션을 할 수 없다.
        if (isReload == true) return;

        // 모드 전환중이면 무기 액션을 할 수 없다.
        /*●*/if (isModeChange == true) return;

        // 마우스 왼쪽 클릭 (공격 시작)
        if (type == 0)
        {
            // 연속 공격
            if (weaponSetting.isAutomaticAttack == true)
            {
                /*●*/isAttack = true;
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
        /*●*/else
        /*●*/{
        /*●*/    // 공격 중일 때는 모드 전환을 할 수 없다
        /*●*/    if (isAttack == true) return;
        /*●*/
        /*●*/    StartCoroutine("OnModeChange");
        /*●*/}
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
            /*●*/isAttack=false;
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
            /*●*/string animation = anim.AimModeIs == true ? "AimFire" : "Fire";
            /*●*/anim.Play(animation, - 1, 0);
            // TIP) anim.Play("Fire"); : 같은 애니메이션을 반복할 때 중간에 끊지 못하고 재생 완료 후 다시 재생
            // TIP) anim.Play("Fire",-1,0); : 같은 애니메이션을 반복할 때 애니메이션을 끊고 처음부터 다시 재생

            // 총구 이펙트 재생 (default 모드 일 때만 재생)
            /*●*/if(anim.AimModeIs==false) StartCoroutine("OnMuzzleFlashEffect");
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
        }
        Debug.DrawRay(bulletSpawnPoint.position, attackDirection * weaponSetting.attackDistance, Color.blue);
    }
    /*●*/private IEnumerator OnModeChange()
    /*●*/{
    /*●*/    float current = 0;
    /*●*/    float percent = 0;
    /*●*/    float time = 0.35f;
    /*●*/
    /*●*/    anim.AimModeIs= !anim.AimModeIs;
    /*●*/    imageAim.enabled = !imageAim.enabled;
    /*●*/
    /*●*/    float start = mainCamera.fieldOfView;
    /*●*/    float end = anim.AimModeIs==true? aimModeFOV : defaultModeFOV;
    /*●*/
    /*●*/    isModeChange = true;
    /*●*/
    /*●*/    while (percent < 1)
    /*●*/    {
    /*●*/        current+=Time.deltaTime;
    /*●*/        percent=current/time;
    /*●*/
    /*●*/        // mode에 따라 카메라의 시야각을 변경
    /*●*/        mainCamera.fieldOfView = Mathf.Lerp(start,end,percent);
    /*●*/
    /*●*/        yield return null;
    /*●*/    }
    /*●*/    isModeChange=false;
    /*●*/}

    /*●*/private void ResetVariables()
    /*●*/{
    /*●*/    isReload = false;
    /*●*/    isAttack = false;   
    /*●*/    isModeChange=false;
    /*●*/}
    private void PlaySound(AudioClip clip)
    {
        audioSource.Stop();             // 기존에 재생중인 사운드를 정지하고,
        audioSource.clip = clip;        // 새로운 사운드 clip으로 교체 후
        audioSource.Play();             // 사운드 재생
    }
}
```

(11) WeaponAssaultRifle컴포넌트의 Aim Image에 AimImage오브젝트를 넣어준다.

(12) PlayerController를 수정한다.

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
    private PlayerAnimatorController anim;      // 애니메이션 재생 제어
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
        anim=GetComponent<PlayerAnimatorController>();
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
            anim.MoveSpeed = isRun ? 1.0f : 0.5f;
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
            anim.MoveSpeed = 0;

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

        /*●*/if (Input.GetMouseButtonDown(1))
        /*●*/{
        /*●*/    weapon.StartWeaponAction(1);
        /*●*/}
        /*●*/else if (Input.GetMouseButtonUp(1))
        /*●*/{
        /*●*/    weapon.StopWeaponAction(1);
        /*●*/}

        if (Input.GetKeyDown(keyCodeReload))
        {
            weapon.StartReload();
        }
    }
}
```

