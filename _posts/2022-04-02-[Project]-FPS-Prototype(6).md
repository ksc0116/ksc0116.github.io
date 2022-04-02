---
layout: post
title:  "[Project] FPS Prototype(6)"
subtitle:   ""
categories: Unity
comments: true
---

<br>

(1) 애니메이터 뷰로 가서 재장전 애니메이션을 등록한다.

(2) 트리거 타입의 파라미터 onReload 생성한다.

(3) Movement -> reload로 향하는 트랜지션 하나 Fire -> reload로 가는 트랜지션, reload -> Movement로 가는 트랜지션을 만들고 조건을 알맞게 설정한다.

(4) PlayerAnimatorController를 수정한다.(재장전 애니메이션을 하도록 정의)

```cs
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
    /*●*/public void OnReload()
    /*●*/{
    /*●*/    anim.SetTrigger("onReload");
    /*●*/}

   public void Play(string stateName,int layer, float normalizedTime)
   {
       anim.Play(stateName,layer,normalizedTime);
   }

    /*●*/public bool CurrentAnimationIs(string name)
    /*●*/{
    /*●*/    return anim.GetCurrentAnimatorStateInfo(0).IsName(name);
    /*●*/}
}
```

(5) WeaponAssaultRifle 스크립트를 수정한다.(재장전 로직)

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Events;

[System.Serializable]
public class AmmoEvent : UnityEvent<int, int> { }

public class WeaponAssaultRifle : MonoBehaviour
{
    [HideInInspector]
    public AmmoEvent onAmmoEvent=new AmmoEvent();

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
    /*●*/[SerializeField]
    /*●*/private AudioClip audioClipReload;      // 재장전 사운드

    [Header("Weapon Setting")]
    [SerializeField]
    private WeaponSetting weaponSetting;              // 무기 설정

    private float lastAttackTime = 0;                       // 마지막 발사시간 체크용
    /*●*/private bool isReload = false;          // 재장전 중인지 체크

    private AudioSource          audioSource;            // 사운드 재생 컴포넌트
    private PlayerAnimatorController anim;             // 애니메이션 재생 제어
    private CasingMemoryPool casingMemoryPool;      // 탄피 생성 후 활성/비활성 관리

    // 외부에서 필요한 정보를 열람하기 위해 정의한 Get 프로퍼티
    public WeaponName WeaponName => weaponSetting.weaponName;

    private void Awake()
    {
        audioSource = GetComponent<AudioSource>();
        anim = GetComponentInParent<PlayerAnimatorController>();
        casingMemoryPool=GetComponent<CasingMemoryPool>();

        // 처음 탄 수는 최대로 설정
        weaponSetting.currentAmmo=weaponSetting.maxAmmo;
    }

    private void OnEnable()
    {
        // 무기 장착 사운드 재생
        PlaySound(audioClipTakeOutWeapon);
        // 총구 이펙트 오브젝트 비활성화
        muzzleFlashEffect.SetActive(false);

        // 무기가 활성화될 때 해당 무기의 탄 수 정보를 갱신한다.
        onAmmoEvent.Invoke(weaponSetting.currentAmmo,weaponSetting.maxAmmo);
    }

    // 외부에서 공격 시작할 때 StartWeaponAction(0) 메소드 호출
    public void StartWeaponAction(int type = 0)
    {
        // 재장전 중일 때는 무기 액션을 할 수 없다.
        /*●*/if (isReload == true) return;

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
    /*●*/public void StartReload()
    /*●*/{
    /*●*/    // 현재 재장전 중이면 재장전 불가능
    /*●*/    if(isReload == true) return;
    /*●*/
    /*●*/    // 무기 액션 도중에 'R'키를 눌러 재장전을 시도하면 무기 액션 종료 후 재장전
    /*●*/    StopWeaponAction();
    /*●*/
    /*●*/    StartCoroutine("OnReload");
    /*●*/}
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

    /*●*/private IEnumerator OnReload()
    /*●*/{
    /*●*/    isReload = true;
    /*●*/
    /*●*/    // 재장전 애니메이셔느 사운드 재생
    /*●*/    anim.OnReload();
	/*●*/    audioSource.PlayOneShot(audioClipReload,1.0f); // 사운드가 겹쳐도 재생이 되도록
    /*●*/
    /*●*/    while (true)
    /*●*/    {
    /*●*/        // 사운드가 재생중이 아니고, 현재 애니메이션 Movement이면
    /*●*/        // 재장전 애니메이션(, 사운드) 재생이 종료되었다는 뜻
    /*●*/        if(audioSource.isPlaying==false && anim.CurrentAnimationIs("Movement"))
    /*●*/        {
    /*●*/            isReload=false;
    /*●*/
    /*●*/            // 현재 탄 수를 최대로 설정하고, 바뀐 찬 수 정보를 Text UI에 업데이트
    /*●*/            weaponSetting.currentAmmo=weaponSetting.maxAmmo;
    /*●*/            onAmmoEvent.Invoke(weaponSetting.currentAmmo, weaponSetting.maxAmmo);
    /*●*/
    /*●*/            yield break;
    /*●*/        }
    /*●*/        yield return null;
    /*●*/    }
    /*●*/}
    private void PlaySound(AudioClip clip)
    {
        audioSource.Stop();             // 기존에 재생중인 사운드를 정지하고,
        audioSource.clip = clip;        // 새로운 사운드 clip으로 교체 후
        audioSource.Play();             // 사운드 재생
    }
}
```

(6) PlayerController스크립트를 수정한다.

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
    /*●*/[SerializeField]
    /*●*/private KeyCode keyCodeReload = KeyCode.R;          // 탄 재장전 키


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
        float hAxis = Input.GetAxis("Horizontal");
        float vAxis = Input.GetAxis("Vertical");

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

        /*●*/if (Input.GetKeyDown(keyCodeReload))
        /*●*/{
        /*●*/    weapon.StartReload();
        /*●*/}
    }
}
```

(7) WeaponAssaultRifle컴포넌트에 재장전 사운드를 넣어준다.

이제 게임을 실행하고 R키를 누르면 재장전 애니메이션과 사운드가 재생되며 탄 수가 채원지는 걸 볼 수 있다.