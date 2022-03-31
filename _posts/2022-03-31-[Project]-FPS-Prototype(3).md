---
layout: post
title:  "[Project] FPS Prototype(3)"
subtitle:   ""
categories: Unity
comments: true
---

<br>

# 가시적 공격 효과 - 애니메이션

플레이어가 공격할 때 애니메이션이 재생되도록 공격 애니메이션을 등록한다.

(1) 애니메이터 뷰로 가서 fire애니메이션을 등록하고 이름을 Fire로 변경(코드에서 Animator.Play()로 재생)

(2) Fire에서 Movement로 이어지는 Transition 생성 (조건 설정 x)

(3) PlayerAnimatorController 스크립트를 수정한다(Play() 메소드를 외부에서 호출할 수 있도록 정의).

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
	
    /*●*/public void Play(string stateName,int layer, float normalizedTime)
    /*●*/{
    /*●*/    anim.Play(stateName,layer,normalizedTime);
    /*●*/}
}
```

(4) 무기의 정보를 관리하는 WeaponSetting스크립트를 만든다.

```cs
// 무기의 종류가 여러 종류일 때 공용으로 사용하는 변수들은  구조체로 묶어서 정의하면
// 변수가 추가/삭제될 때 구조체에 선언하기 때문에 추가/삭제에 대한 관리가 용이함
[System.Serializable]
public struct WeaponSetting
{
    public float attackRate;        // 공격 속도
    public float attackDistance;  // 공격 사거리
    public bool isAutomaticAttack;  // 연속 공격 여부
}
// TIP) 구조체는 스택 영역, 클래스는 힙 영역에 메모리 할당된다.
// TIP) [System.Serializable]을 이용해 직렬화 하지 않으면 다른 클래스의 변수로 생성되었을 때 인스펙터 창에 멤버 변수들의 목록이 뜨지 않는다.
```

(5) WeaponAssaultRifle스크립트를 수정한다. (무기 공격 구현을 위해)

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class WeaponAssaultRifle : MonoBehaviour
{
    [Header("Audio Clips")]
    [SerializeField]
    private AudioClip               audioClipTakeOutWeapon;     // 무기 장착 사운드

    /*●*/[Header("Weapon Setting")]
    /*●*/[SerializeField]
    /*●*/private WeaponSetting weaponSetting;           // 무기 설정

    /*●*/private float lastAttackTime = 0;               // 마지막 발사시간 체크용

    private AudioSource          audioSource;          // 사운드 재생 컴포넌트
    /*●*/private PlayerAnimatorController anim;        // 애니메이션 재생 제어

    private void Awake()
    {
        audioSource = GetComponent<AudioSource>();
        /*●*/anim = GetComponentInParent<PlayerAnimatorController>();
    }

    private void OnEnable()
    {
        // 무기 장착 사운드 재생
        PlaySound(audioClipTakeOutWeapon);
    }

            // 외부에서 공격 시작할 때 StartWeaponAction(0) 메소드 호출
    /*●*/public void StartWeaponAction(int type = 0)
    /*●*/{
    /*●*/    // 마우스 왼쪽 클릭 (공격 시작)
    /*●*/    if (type == 0)
    /*●*/    {
    /*●*/        // 연속 공격
    /*●*/        if (weaponSetting.isAutomaticAttack == true)
    /*●*/        {
    /*●*/            // OnAttack()을 매 프레임 실행
    /*●*/            StartCoroutine("OnAttackLoop");
    /*●*/        }
    /*●*/        // 단발 공격
    /*●*/        else
    /*●*/        {
    /*●*/            // 실제 공격 정의 함수
    /*●*/            OnAttack();
    /*●*/        }
    /*●*/    }
    /*●*/}

            // 외부에서 공격 종료할 때 StopWeaponAction(0) 메소드 호출
    /*●*/public void StopWeaponAction(int type = 0)
    /*●*/{
    /*●*/    // 마우스 왼쪽 해제 (공격 종료)
    /*●*/    if(type == 0)
    /*●*/    {
    /*●*/        StopCoroutine("OnAttackLoop");
    /*●*/    }
    /*●*/}

    /*●*/private IEnumerator OnAttackLoop()
    /*●*/{
    /*●*/    while (true)
    /*●*/    {
    /*●*/        OnAttack();
    /*●*/
    /*●*/        yield return null;
    /*●*/    }
    /*●*/}

    /*●*/public void OnAttack()
    /*●*/{
    /*●*/    if(Time.time - lastAttackTime > weaponSetting.attackRate)
    /*●*/    {
    /*●*/        // 뛰고있을 땐 공격x
    /*●*/        if (anim.MoveSpeed > 0.5f)
    /*●*/        {
    /*●*/            return;
    /*●*/        }
    /*●*/
    /*●*/        // 공격주기가 되어야 공격할 수 있도록 하기 위해 현재시간 정보 저장
    /*●*/        lastAttackTime = Time.time;
    /*●*/
    /*●*/        // 무기 애니메이션 재생
    /*●*/        anim.Play("Fire",-1,0);
    /*●*/        // TIP) anim.Play("Fire"); : 같은 애니메이션을 반복할 때 중간에 끊지 못하고 재생 완료 후 다시 재생
    /*●*/        // TIP) anim.Play("Fire",-1,0); : 같은 애니메이션을 반복할 때 애니메이션을 끊고 처음부터 다시 재생
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

(6) PlayerController스크립트를 수정한다.(무기 공격 실행을 위해)

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    [Header("Input KeyCodes")]
    [SerializeField]
    private KeyCode keyCodeRun = KeyCode.LeftShift;            // 달리기 키
    /*●*/[SerializeField]
    /*●*/private KeyCode keyCodeJump = KeyCode.Space;            // 점프 키


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
    /*●*/private WeaponAssaultRifle weapon;      // 무기를 이용한 공격 제어

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
        /*●*/weapon=GetComponentInChildren<WeaponAssaultRifle>();
    }
    private void Update()
    {
        UpdateRotate();
        UpdateMove();
        UpdateJump();
        /*●*/UpdateWeaponAction();
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
    /*●*/private void UpdateWeaponAction()
    /*●*/{
    /*●*/    if (Input.GetMouseButtonDown(0))
    /*●*/    {
    /*●*/        weapon.StartWeaponAction();
    /*●*/    }
    /*●*/    else if (Input.GetMouseButtonUp(0))
    /*●*/    {
    /*●*/        weapon.StopWeaponAction();
    /*●*/    }
    /*●*/}
}
```

(7) WeaponAssaultRifle 컴포넌트 설정 

WeaponSetting

* attackRate : 0.1
* attackDistance : 30
* isAutomaticAttack : true

이제 게임을 실행하고 마우스 좌클릭을하면 공격 애니메이션이 재생되는 걸 볼 수 있다. 그리고 마우스 클릭을 그만두면 애니메이션이 종료되고, isAutomaticAttack이 false면 단발 공격이 되는 걸 볼 수 있다.

***

# 가시적 공격 효과 - 총구 이펙트

플레이어가 공격할 때 총구 이펙트를 재생한다. 총구 이펙트는 공격할 때 총구에서 뿜어져 나오는 불꽃과 같은 효과로 이미지, 파티클, 조명등을 이용해 제작한다.

(1) Toon Muzzleflash 1번을 weapon오브젝트의 자식으로 넣어준다. (weapon오브젝트는 애니메이션이 재생될 때 현재 무기 위치를 나타내기 때문에 weapon의 자식으로 총구 이펙트를 배치)

![image](https://user-images.githubusercontent.com/101051124/160986658-26b9bcdf-5158-428d-a553-b98e3d992e7b.png)

(2)Toon Muzzleflash 1번의 Transform을 다음과 같이 설정한다. 

* 위치(0 / 0.6 / 0.78)
* 회전(0 / 270 / 33)
* 크기(0.05 / 0.05 / 0.05)

![image](https://user-images.githubusercontent.com/101051124/160987014-bca52ab4-3426-4645-8372-034833c9448f.png)

그럼 위와 같은 모습이 된다. 평소에는 보이지 않다가 공격할 때마다 잠깐 등장하게 설정할 것이다.

(3) WeaponAssaultRifle스크립트를 수정한다. (총구 이펙트 On/Off를 위해)

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class WeaponAssaultRifle : MonoBehaviour
{
    [Header("Fire Effects")]
    [SerializeField]
    private GameObject muzzleFlashEffect;      // 총구 이펙트 (On/Off)

    [Header("Audio Clips")]
    [SerializeField]
    private AudioClip               audioClipTakeOutWeapon;           // 무기 장착 사운드

    [Header("Weapon Setting")]
    [SerializeField]
    private WeaponSetting weaponSetting;              // 무기 설정

    private float lastAttackTime = 0;                       // 마지막 발사시간 체크용

    private AudioSource          audioSource;            // 사운드 재생 컴포넌트
    private PlayerAnimatorController anim;             // 애니메이션 재생 제어

    private void Awake()
    {
        audioSource = GetComponent<AudioSource>();
        anim = GetComponentInParent<PlayerAnimatorController>();
    }

    private void OnEnable()
    {
        // 무기 장착 사운드 재생
        PlaySound(audioClipTakeOutWeapon);
        // 총구 이펙트 오브젝트 비활성화
        /*●*/muzzleFlashEffect.SetActive(false);
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
            /*●*/StartCoroutine("OnMuzzleFlashEffect");
        }
    }
    /*●*/private IEnumerator OnMuzzleFlashEffect()
    /*●*/{
    /*●*/    muzzleFlashEffect.SetActive (true);
    /*●*/
    /*●*/    // 무기의 공격속도보다 빠르게 잠깐 활성화 시켰다가 비활성화 한다
    /*●*/    yield return new WaitForSeconds(weaponSetting.attackRate*0.3f);
    /*●*/
    /*●*/    muzzleFlashEffect.SetActive(false);
    /*●*/}
    private void PlaySound(AudioClip clip)
    {
        audioSource.Stop();             // 기존에 재생중인 사운드를 정지하고,
        audioSource.clip = clip;        // 새로운 사운드 clip으로 교체 후
        audioSource.Play();             // 사운드 재생
    }
}
```

이제 게임을 실행해서 공격해보면 이펙트가 잠시동안만 활성화했다가 다시 비활성화되는 모습을 볼 수 있다.

***

# 가시적 공격 효과 - 사운드

(1) WeaponAssaultRifle을 수정한다. (공격 사운드 재생을 위해)

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class WeaponAssaultRifle : MonoBehaviour
{
    [Header("Fire Effects")]
    [SerializeField]
    private GameObject muzzleFlashEffect;      // 총구 이펙트 (On/Off)

    [Header("Audio Clips")]
    [SerializeField]
    private AudioClip audioClipTakeOutWeapon;           // 무기 장착 사운드
    /*●*/[SerializeField]
    /*●*/private AudioClip audioClipFire;        // 공격 사운드

    [Header("Weapon Setting")]
    [SerializeField]
    private WeaponSetting weaponSetting;              // 무기 설정

    private float lastAttackTime = 0;                       // 마지막 발사시간 체크용

    private AudioSource          audioSource;            // 사운드 재생 컴포넌트
    private PlayerAnimatorController anim;             // 애니메이션 재생 제어

    private void Awake()
    {
        audioSource = GetComponent<AudioSource>();
        anim = GetComponentInParent<PlayerAnimatorController>();
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
            /*●*/PlaySound(audioClipFire);
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

(2) Weapon Assault Rifle컴포넌트의 Audio Clip Fire에 shoot사운드를 넣어준다.

이제 게임을 실행하면 공격할 때 총구 이펙트와 공격 사운드가 나오는 걸 볼 수 있다.