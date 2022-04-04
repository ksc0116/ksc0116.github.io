---
layout: post
title:  "[Project] FPS Prototype(12)"
subtitle:   ""
categories: Unity
comments: true
---

<br>

적 캐릭터의 공격 행동과, 플레이어의 적 공격을 구현해보자.

# 적 캐릭터 FSM - 공격하기

(1) 적 발사체로 사용 할 sphere(EnemyProjectile)를 만든다. 크기(0.1 / 0.1 / 0.1), IsTrigger(true)

(2) Enemyprojectile오브젝트를 프리팹으로 만든다.

(3) Transform으로 이동을 제어하는 MovementTransform스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class MovementTransform : MonoBehaviour
{
    [SerializeField]
    private float moveSpeed = 0.0f;
    [SerializeField]
    private Vector3 moveDirection = Vector3.zero;

    /// <summary>
    /// 이동 방향이 설정되면 알아서 이동하도록 함
    /// </summary>
    private void Update()
    {
        transform.position+=moveDirection*moveSpeed*Time.deltaTime;
    }

    /// <summary>
    /// 외부에서 매개변수로 이동 방향을 설정
    /// </summary>
    public void MoveTo(Vector3 direction)
    {
        moveDirection = direction;
    }
}
```

(4) 적 발사체를 제어하는 EnemyProjectile스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class EnemyProjectile : MonoBehaviour
{
    private MovementTransform movement;
    private float projectileDistance = 30;      // 발사체 최대 발사거리

    public void Setup(Vector3 position)
    {
        movement = GetComponent<MovementTransform>();

        StartCoroutine("OnMove", position);
    }

    private IEnumerator OnMove(Vector3 targetPosition)
    {
        Vector3 start = transform.position;

        movement.MoveTo((targetPosition-transform.position).normalized);

        while (true)
        {
            if (Vector3.Distance(transform.position, start) >= projectileDistance)
            {
                Destroy(gameObject);

                yield break;
            }
            yield return null;
        }
    }

    private void OnTriggerEnter(Collider other)
    {
        if (other.gameObject.tag == "Player")
        {
            Debug.Log("Player Hit");

            Destroy(gameObject);
        }
    }
}
```

(5) EnemyProjectile프리팹에 MovementTransform,EnemyProjectile스크립트를 넣어준다.

(6) EnemyFSM스크립트를 수정한다.(공격 상태 추가)

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.AI;

public enum EnemyState {  None=-1,Idle = 0, Wander, Pursuit, /*●*/Attack,}

public class EnemyFSM : MonoBehaviour
{
    [Header("Pursuit")]
    [SerializeField]
    private float targetRecognitionRange = 8;       // 인식 범위 (이 범위 안에 들어오면 "PurSuit" 상태로 변경)
    [SerializeField]
    private float pursuitLimitRange = 10;       // 추적 범위 (이 범위 바깥으로 나가면 "Wander" 상태로 변경)

    /*●*/[Header("Attack")]
    /*●*/[SerializeField]
    /*●*/private GameObject projectilePrefab;        // 발사체 프리팹
    /*●*/[SerializeField]
    /*●*/private Transform projectileSpawnPoint;     // 발사체 생성 위치
    /*●*/[SerializeField]
    /*●*/private float attackRange = 5;      // 공격 범위 (이 범위 안에 들어오면 "Attack" 상태로 변경)
    /*●*/[SerializeField]        
    /*●*/private float attackRate = 1;       // 공격 속도

    private EnemyState enemyState = EnemyState.None;        // 현재 적 행동
    /*●*/private float lastAttackTime = 0;       // 공격 주기 계산용 변수

    private Status status;      // 이동속도 등의 정보
    private NavMeshAgent navMeshAgent;      // 이동 제어를 위한 NavMeshAgent
    private Transform target;       // 적의 공격 대상 (플레이어)

    //private void Awake()
    public void Setup(Transform target)
    {
        status=GetComponent<Status>();
        navMeshAgent=GetComponent<NavMeshAgent>();
        this.target=target;

        // NavMeshAgent 컴포넌트에서 회전을 업데이트하지 않도록 설정
        navMeshAgent.updateRotation = false;
    }

    private void OnEnable()
    {
        // 적이 활성화될 떄 적의 상태를 "대기"로 설정
        ChangeState(EnemyState.Idle);
    }

    private void OnDisable()
    {
        // 적이 비활성화될 때 현재 재생중인 상태를 종료하고, 상태를 "None"으로 설정
        StopCoroutine(enemyState.ToString());

        enemyState=EnemyState.None;
    }

    public void ChangeState(EnemyState newState)
    {
        // 현재 재생중인 상태와 바꾸려고 하는 상태가 같으면 바꿀 필요가 없기 때문에 return
        if (enemyState == newState) return;

        // 이전에 재생중이던 상태 종료
        StopCoroutine(enemyState.ToString());
        // 현재 적의 상태를 newState로 설정
        enemyState = newState;
        // 새로운 상태 재생
        StartCoroutine(enemyState.ToString());
    }

    private IEnumerator Idle()
    {
        // n초 후에  "배회" 상태로 변경하는 코루틴 실행
        StartCoroutine("AutoChangeFromIdleToWander");

        while (true)
        {
            // "대기" 상태일 떄 하는 행동
            // 타겟과의 거리에 따라 행동 선택 (배회, 추격, 원거리 공격)
            CalculateDistanceToTargetAndSelectState();

            yield return null;
        }
    }
    private IEnumerator AutoChangeFromIdleToWander()
    {
        // 1~4초 시간 대기
        int changeTime = Random.Range(1, 5);

        yield return new WaitForSeconds(changeTime);

        // 상태를 "배회"로 변경
        ChangeState(EnemyState.Wander);
    }
    private IEnumerator Wander()
    {
        float currentTime = 0;
        float maxTime = 10;

        // 이동 속도 설정
        navMeshAgent.speed = status.WalkSpeed;

        // 목표 위치 설정
        navMeshAgent.SetDestination(CalculateWanderPosition());

        // 목표 위치로 회전
        Vector3 to = new Vector3(navMeshAgent.destination.x,0,navMeshAgent.destination.z);
        Vector3 from = new Vector3(transform.position.x,0,transform.position.z);
        transform.rotation = Quaternion.LookRotation(to - from);

        while (true)
        {
            currentTime += Time.deltaTime;

            // 목표위치에 근접하게 도달하거나 너무 오랜시간 배회하기 상태에 머물러 있으면
            to = new Vector3(navMeshAgent.destination.x, 0, navMeshAgent.destination.z);
            from = new Vector3(transform.position.x, 0, transform.position.z);
            if((to-from).sqrMagnitude<0.01f || currentTime >= maxTime)
            {
                // 상태를 "대기"로 변경
                ChangeState(EnemyState.Idle);
            }

            // 타겟과의 거리에 따라 행동 선택 (배회, 추격, 원거리 공격)
            CalculateDistanceToTargetAndSelectState();

            yield return null;
        }
    }

    private Vector3 CalculateWanderPosition()
    {
        float wanderRadius = 10;        // 현재 위치를 원점으로 하는 원의 반지름
        int wanderJitter = 0;       // 선택된 각도 (wanderJitterMin ~ wanderJitterMax)
        int wanderJitterMin = 0;    // 최소 각도
        int wanderJitterMax = 360;

        // 현재 적 캐릭터가 있는 월드의 중심 위치와 크기 (구역을 벗어난 행동을 하지 않도록)
        Vector3 rangePosition = Vector3.zero;
        Vector3 rangeScale=Vector3.one*100.0f;

        // 자신의 위치를 중심으로 반지름(wanderRadius) 거리, 선택된 각도(wanderJitter)에 위치한 좌표를 목표지점으로 설정
        wanderJitter=Random.Range(wanderJitterMin,wanderJitterMax);
        Vector3 targetPosition=transform.position+SetAngle(wanderRadius,wanderJitter);

        // 생성된 목표위치가 자신의 이동구역을 벗어나지 않게 조절
        targetPosition.x = Mathf.Clamp(targetPosition.x, rangePosition.x - rangeScale.x * 0.5f, rangePosition.x + rangeScale.x * 0.5f);
        targetPosition.y = 0.0f;
        targetPosition.z = Mathf.Clamp(targetPosition.z, rangePosition.z - rangeScale.z * 0.5f, rangePosition.z + rangeScale.z * 0.5f);

        return targetPosition;
    }
    private Vector3 SetAngle(float radius,int angle)
    {
        Vector3 position = Vector3.zero;

        position.x = Mathf.Cos(angle) * radius;
        position.z = Mathf.Sin(angle) * radius;

        return position;
    }

    private IEnumerator Pursuit()
    {
        while (true)
        {
            // 이동 속도 설정 (배회할 때는 걷는 속도로 이동, 추적할 때는 뛰는 속도로 이동)
            navMeshAgent.speed = status.RunSpeed;
    
            // 목표위치를 현재 플레이어의 위치로 설정
            navMeshAgent.SetDestination(target.position);
    
            // 타겟 방향을 주시하도록 함
            LookRotationToTarget();
    
            // 타겟과의 거리에 따라 행동 선택 (배회, 추격, 원거리 공격)
            CalculateDistanceToTargetAndSelectState();
    
            yield return null;
        }
    }

    /*●*/private IEnumerator Attack()
    /*●*/{
    /*●*/    // 공격할 때는 이동을 멈추도록 설정
    /*●*/    navMeshAgent.ResetPath();
    /*●*/
    /*●*/    while (true)
    /*●*/    {
    /*●*/        // 타겟 방향 주시
    /*●*/        LookRotationToTarget();
    /*●*/
    /*●*/        // 타겟과의 거리에 따라 행동 선택 (배회, 추격, 원거리 공격)
    /*●*/        CalculateDistanceToTargetAndSelectState();
    /*●*/
    /*●*/        if (Time.time - lastAttackTime > attackRate)
    /*●*/        {
    /*●*/            // 공격주지가 되어야 공격할 수 있도록 하기 위해 연재 시간 저장
    /*●*/            lastAttackTime = Time.time;
    /*●*/
    /*●*/            // 발사체 생성
    /*●*/            GameObject clone = Instantiate(projectilePrefab,projectileSpawnPoint.position,projectileSpawnPoint.rotation);
    /*●*/            clone.GetComponent<EnemyProjectile>().Setup(target.position);
    /*●*/        }
    /*●*/
    /*●*/        yield return null;
    /*●*/    }
    /*●*/}

    private void LookRotationToTarget()
    {
        // 목표 위치
        Vector3 to = new Vector3(target.position.x,0,target.position.z);
        // 내 위치
        Vector3 from = new Vector3(transform.position.x,0,transform.position.z);
    
        // 바로 돌기
        transform.rotation = Quaternion.LookRotation(to-from);
    
        // 서서히 돌기
        //Quaternion rotation = Quaternion.LookRotation(to - from);
        //transform.rotation = Quaternion.Slerp(transform.rotation, rotation, 0.01f);
    }

    private void CalculateDistanceToTargetAndSelectState()
    {
        if (target == null) return;
    
        // 플레이어(target)와 적의 거리 계산 후 거리에 따라 행동 선택
        float distance = Vector3.Distance(target.position, transform.position);

        /*●*/if (distance <= attackRange)
        /*●*/{
        /*●*/    ChangeState(EnemyState.Attack);
        /*●*/}
        else if (distance <= targetRecognitionRange)
        {
            ChangeState(EnemyState.Pursuit);
        }
        else if(distance >= pursuitLimitRange)
        {
            ChangeState(EnemyState.Wander);
        }
    }

    private void OnDrawGizmos()
    {
        // "배회" 상태일 때 이동할 경로 표시
        Gizmos.color = Color.black;
        Gizmos.DrawRay(transform.position,navMeshAgent.destination-transform.position);

        // 목표 인식 범위
        Gizmos.color = Color.red;
        Gizmos.DrawWireSphere(transform.position, targetRecognitionRange);
        
        // 추적 범위
        Gizmos.color = Color.green;
        Gizmos.DrawWireSphere(transform.position, pursuitLimitRange);

        // 공격 범위
        /*●*/Gizmos.color =new Color(0.39f, 0.04f, 0.04f);
        /*●*/Gizmos.DrawWireSphere(transform.position, attackRange);
    }
}
```

(7) Enemy프리팹에 EnemyProjectile프리팹, EnemyGun프리팹을 할당해준다.

이제 플레이어가 공격 범위 안에 들어가면 공격하는 모습을 볼 수 있다.

***

# 플레이어 체력 감소 설정

(1) 플레이어 체력 출력을 위해 TextMeshPro(TextHP)를 만들고 알맞게 설정한다.

(2) 화면에 플레이어의 피격을 표시하기 위해 Image(ImageBloodScreen)를 만들고 알맞게 설정한다. (색상의 알파값만 0으로 설정)

(3) status스크립트를 수정한다.(현재 체력이 바뀌면 호출 되는 이벤트 선언, 체력 감소 함수, 체력 변수들)

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Events;

/*●*/[System.Serializable]
/*●*/public class HPEvent : UnityEvent<int,int> {  }
public class Status : MonoBehaviour
{
    /*●*/[HideInInspector]
    /*●*/public HPEvent onHPEvent=new HPEvent();

    [Header("Walk, Run Speed")]
    [SerializeField]
    private float walkSpeed;
    [SerializeField]
    private float runSpeed;

    /*●*/[Header("HP")]
    /*●*/[SerializeField]
    /*●*/private int maxHP = 100;
    /*●*/private int currentHP;

    public float WalkSpeed => walkSpeed;
    public float RunSpeed => runSpeed;

    /*●*/public int CurrentHP => currentHP;
    /*●*/public int MaxHP => maxHP;
    /*●*/
    /*●*/private void Awake()
    /*●*/{
    /*●*/    currentHP = maxHP;
    /*●*/}

    /*●*/public bool DecreaseHP(int damage)
    /*●*/{
    /*●*/    int previousHP = currentHP;
    /*●*/
    /*●*/    currentHP=currentHP- damage> 0 ? currentHP - damage : 0;
    /*●*/
    /*●*/    onHPEvent.Invoke(previousHP,currentHP);
    /*●*/
    /*●*/    if (currentHP == 0)
    /*●*/    {
    /*●*/        return true;
    /*●*/    }
    /*●*/
    /*●*/    return false;
    /*●*/}
}
```

(4) PlayerHUD스크립트를 수정한다.(체력 UI가 생신되도록, 피격 당했을 때 나오는 이미지 설정)

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
    /*●*/[SerializeField]
    /*●*/private Status status;      // 플레이어의 상태 (이동속도, 체력)

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

    /*●*/[Header("HP & BloodScreen UI")]
    /*●*/[SerializeField]
    /*●*/private TextMeshProUGUI textHP;     // 플레이어의 체력을 출력하는 text
    /*●*/[SerializeField]
    /*●*/private Image imageBloodScreen;     // 플레이어가 공격받았을 때 화면에 표시되는 Image
    /*●*/[SerializeField]
    /*●*/private AnimationCurve curveBloodScreen;

    private void Awake()
    {
        SetupWeapon();
        SetupMagazine();

        // 메소드가 등록되어 있는 이벤트 클래스 (weapon.xx)의
        // Invoke() 메소드가 호출될 때 등록된 메소드(매개변수)가 실행된다.
        weapon.onAmmoEvent.AddListener(UpdateAmmoHUD);
        weapon.onMagazineEvent.AddListener(UpdateMagazineHUD);
        /*●*/status.onHPEvent.AddListener(UpdateHPHUD);
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

    /*●*/private void UpdateHPHUD(int previous,int current)
    /*●*/{
    /*●*/    textHP.text = "HP " + current;
    /*●*/
    /*●*/    if(previous - current > 0)
    /*●*/    {
    /*●*/        StopCoroutine("OnBloodScreen");
    /*●*/        StartCoroutine("OnBloodScreen");
    /*●*/    }
    /*●*/}
    /*●*/private IEnumerator OnBloodScreen()
    /*●*/{
    /*●*/    float percent = 0;
    /*●*/
    /*●*/    while (percent < 1)
    /*●*/    {
    /*●*/        percent+=Time.deltaTime;
    /*●*/
    /*●*/        Color color = imageBloodScreen.color;
    /*●*/        color.a=Mathf.Lerp(1,0,curveBloodScreen.Evaluate(percent));
    /*●*/        imageBloodScreen.color=color;
    /*●*/
    /*●*/        yield return null;
    /*●*/    }
    /*●*/}
}
```

(5) PlayerHUD오브젝트의 status(Player오브젝트), Text HP(TextHP), ImageBloodScreen(ImageBloodScreen), CurveBloodScreen(상승곡선)을 설정해준다.

(6) PlayerController스크립트를 수정한다.(플레이거 데미지 입는 함수 만듬)

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
    /*●*/public void TakeDamage(int damage)
    /*●*/{
    /*●*/    bool isDie = status.DecreaseHP(damage);
    /*●*/
    /*●*/    if(isDie == true)
    /*●*/    {
    /*●*/        Debug.Log("GameOver");
    /*●*/    }
    /*●*/}
}
```

(7) EnemyProjectile스크립트를 수정한다.(발사체의 공격력을 정하고, 발사체가 플레이어랑 부딪히면 플레이어 데미지 입음)

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class EnemyProjectile : MonoBehaviour
{
    private MovementTransform movement;
    private float projectileDistance = 30;      // 발사체 최대 발사거리
    /*●*/private int damage = 5;     // 발사체 공격력

    public void Setup(Vector3 position)
    {
        movement = GetComponent<MovementTransform>();

        StartCoroutine("OnMove", position);
    }

    private IEnumerator OnMove(Vector3 targetPosition)
    {
        Vector3 start = transform.position;

        movement.MoveTo((targetPosition-transform.position).normalized);

        while (true)
        {
            if (Vector3.Distance(transform.position, start) >= projectileDistance)
            {
                Destroy(gameObject);

                yield break;
            }
            yield return null;
        }
    }

    private void OnTriggerEnter(Collider other)
    {
        if (other.gameObject.tag == "Player")
        {
            /*Debug.Log("Player Hit");*/
            /*●*/other.GetComponent<PlayerController>().TakeDamage(damage);

            Destroy(gameObject);
        }
    }
}
```

이제 게임을 실행하고 적에게 데미지를 입으면 화면이 잠깐 빨개지고 체력 UI가 갱신되는 모습을 볼 수 있다.

***

# 플레이어의 적 캐릭터 공격

(1) Enemy프리팹에 RigidBody를 추가한다. UseGravity(false), Constraints(모든 항목 true)

(2) ImpactNormal 파티클을 복제하고 이름을 ImpactBlood로 변경한다. StartColor(255 / 0 / 0 / 255)

(3) EnemyMemoryPool스크립트를 수정한다.(외부에서 매개변수로 받은 오브젝트를 비활성화하는 함수, 적이 생성될 때 EnemyMemoryPool를보냄(Setup 메소드로))

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class EnemyMemoryPool : MonoBehaviour
{
    [SerializeField]
    private Transform target;       // 적의 목표 (플레이어)
    [SerializeField]
    private GameObject enemySpawnPointPrefab;           // 적이 등장하기 전 적의 등장 위치를 알려주는 프리팹
    [SerializeField]
    private GameObject enemyPrefab;     // 생성되는 적 프리팹
    [SerializeField]
    private float enemySpawnTime = 1;       // 적 생성 주기
    [SerializeField]
    private float enemySpawnLatency = 1;    // 타일 생성 후 적이 등장하기까지 대기 시간

    private MemoryPool spawnPointMemoryPool;        // 적 등장 위치를 알려주는 오브젝트 생성, 활성/비활성화 관리
    private MemoryPool enemyMemoryPool;      // 적 생성, 활성/비활성화 관리

    private int numberOfEnemiesSpawnedAtOne = 1;        // 동시에 생성되는 적의 숫자
    private Vector2Int mapSize = new Vector2Int(100, 100);

    private void Awake()
    {
        spawnPointMemoryPool = new MemoryPool(enemySpawnPointPrefab);
        enemyMemoryPool = new MemoryPool(enemyPrefab);

        StartCoroutine("SpawnTile");
    }

    private IEnumerator SpawnTile()
    {
        int currentNumber = 0;
        int maximumNumber = 50;

        while (true)
        {
            // 동시에 numberOfEnemiesSpawnAtOne 숫자만큼 생성되도록 반복문 사용
            for(int i = 0; i < numberOfEnemiesSpawnedAtOne; i++)
            {
                GameObject item = spawnPointMemoryPool.ActivatePoolItem();

                item.transform.position = new Vector3(Random.Range(-mapSize.x * 0.49f, mapSize.x * 0.49f), 1,
                                                                       Random.Range(-mapSize.y * 0.49f, mapSize.y * 0.49f));

                StartCoroutine("SpawnEnemy", item);
            }
            currentNumber++;
            if(currentNumber >= maximumNumber)
            {
                currentNumber = 0;
                numberOfEnemiesSpawnedAtOne++;
            }

            yield return new WaitForSeconds(enemySpawnTime);
        }
    }

    private IEnumerator SpawnEnemy(GameObject point)
    {
        yield return new WaitForSeconds(enemySpawnLatency);

        // 적 오브젝트를 생성하고, 적의 위치를 point의 위치로 설정
        GameObject item = enemyMemoryPool.ActivatePoolItem();
        item.transform.position=point.transform.position;

        /*●*/item.GetComponent<EnemyFSM>().Setup(target,this);

        // 타일 오브젝트 비활성화
        spawnPointMemoryPool.DeactivatePoolItem(point);
    }

    /*●*/public void DeactiveEnemy(GameObject enemy)
    /*●*/{
    /*●*/    enemyMemoryPool.DeactivatePoolItem(enemy);
    /*●*/}
}
```

(4) EnemyFSM스크립트를 수정한다(Setup메소드 수정, 적의 체력이 0이되면 비활성화하는 함수 추가)

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.AI;

public enum EnemyState {  None=-1,Idle = 0, Wander, Pursuit, /*●*/Attack,}

public class EnemyFSM : MonoBehaviour
{
    [Header("Pursuit")]
    [SerializeField]
    private float targetRecognitionRange = 8;       // 인식 범위 (이 범위 안에 들어오면 "PurSuit" 상태로 변경)
    [SerializeField]
    private float pursuitLimitRange = 10;       // 추적 범위 (이 범위 바깥으로 나가면 "Wander" 상태로 변경)

    [Header("Attack")]
    [SerializeField]
    private GameObject projectilePrefab;        // 발사체 프리팹
    [SerializeField]
    private Transform projectileSpawnPoint;     // 발사체 생성 위치
    [SerializeField]
    private float attackRange = 5;      // 공격 범위 (이 범위 안에 들어오면 "Attack" 상태로 변경)
    [SerializeField]        
    private float attackRate = 1;       // 공격 속도

    private EnemyState enemyState = EnemyState.None;        // 현재 적 행동
    private float lastAttackTime = 0;       // 공격 주기 계산용 변수

    private Status status;      // 이동속도 등의 정보
    private NavMeshAgent navMeshAgent;      // 이동 제어를 위한 NavMeshAgent
    private Transform target;       // 적의 공격 대상 (플레이어)
    /*●*/private EnemyMemoryPool enemyMemoryPool;        // 적 메모리 풀 (적 오브젝트 비활성화에 사용)

    //private void Awake()
    public void Setup(Transform target, /*●*/EnemyMemoryPool enemyMemoryPool)
    {
        status=GetComponent<Status>();
        navMeshAgent=GetComponent<NavMeshAgent>();
        this.target=target;
        /*●*/this.enemyMemoryPool=enemyMemoryPool;

        // NavMeshAgent 컴포넌트에서 회전을 업데이트하지 않도록 설정
        navMeshAgent.updateRotation = false;
    }

    private void OnEnable()
    {
        // 적이 활성화될 떄 적의 상태를 "대기"로 설정
        ChangeState(EnemyState.Idle);
    }

    private void OnDisable()
    {
        // 적이 비활성화될 때 현재 재생중인 상태를 종료하고, 상태를 "None"으로 설정
        StopCoroutine(enemyState.ToString());

        enemyState=EnemyState.None;
    }

    public void ChangeState(EnemyState newState)
    {
        // 현재 재생중인 상태와 바꾸려고 하는 상태가 같으면 바꿀 필요가 없기 때문에 return
        if (enemyState == newState) return;

        // 이전에 재생중이던 상태 종료
        StopCoroutine(enemyState.ToString());
        // 현재 적의 상태를 newState로 설정
        enemyState = newState;
        // 새로운 상태 재생
        StartCoroutine(enemyState.ToString());
    }

    private IEnumerator Idle()
    {
        // n초 후에  "배회" 상태로 변경하는 코루틴 실행
        StartCoroutine("AutoChangeFromIdleToWander");

        while (true)
        {
            // "대기" 상태일 떄 하는 행동
            // 타겟과의 거리에 따라 행동 선택 (배회, 추격, 원거리 공격)
            CalculateDistanceToTargetAndSelectState();

            yield return null;
        }
    }
    private IEnumerator AutoChangeFromIdleToWander()
    {
        // 1~4초 시간 대기
        int changeTime = Random.Range(1, 5);

        yield return new WaitForSeconds(changeTime);

        // 상태를 "배회"로 변경
        ChangeState(EnemyState.Wander);
    }
    private IEnumerator Wander()
    {
        float currentTime = 0;
        float maxTime = 10;

        // 이동 속도 설정
        navMeshAgent.speed = status.WalkSpeed;

        // 목표 위치 설정
        navMeshAgent.SetDestination(CalculateWanderPosition());

        // 목표 위치로 회전
        Vector3 to = new Vector3(navMeshAgent.destination.x,0,navMeshAgent.destination.z);
        Vector3 from = new Vector3(transform.position.x,0,transform.position.z);
        transform.rotation = Quaternion.LookRotation(to - from);

        while (true)
        {
            currentTime += Time.deltaTime;

            // 목표위치에 근접하게 도달하거나 너무 오랜시간 배회하기 상태에 머물러 있으면
            to = new Vector3(navMeshAgent.destination.x, 0, navMeshAgent.destination.z);
            from = new Vector3(transform.position.x, 0, transform.position.z);
            if((to-from).sqrMagnitude<0.01f || currentTime >= maxTime)
            {
                // 상태를 "대기"로 변경
                ChangeState(EnemyState.Idle);
            }

            // 타겟과의 거리에 따라 행동 선택 (배회, 추격, 원거리 공격)
            CalculateDistanceToTargetAndSelectState();

            yield return null;
        }
    }

    private Vector3 CalculateWanderPosition()
    {
        float wanderRadius = 10;        // 현재 위치를 원점으로 하는 원의 반지름
        int wanderJitter = 0;       // 선택된 각도 (wanderJitterMin ~ wanderJitterMax)
        int wanderJitterMin = 0;    // 최소 각도
        int wanderJitterMax = 360;

        // 현재 적 캐릭터가 있는 월드의 중심 위치와 크기 (구역을 벗어난 행동을 하지 않도록)
        Vector3 rangePosition = Vector3.zero;
        Vector3 rangeScale=Vector3.one*100.0f;

        // 자신의 위치를 중심으로 반지름(wanderRadius) 거리, 선택된 각도(wanderJitter)에 위치한 좌표를 목표지점으로 설정
        wanderJitter=Random.Range(wanderJitterMin,wanderJitterMax);
        Vector3 targetPosition=transform.position+SetAngle(wanderRadius,wanderJitter);

        // 생성된 목표위치가 자신의 이동구역을 벗어나지 않게 조절
        targetPosition.x = Mathf.Clamp(targetPosition.x, rangePosition.x - rangeScale.x * 0.5f, rangePosition.x + rangeScale.x * 0.5f);
        targetPosition.y = 0.0f;
        targetPosition.z = Mathf.Clamp(targetPosition.z, rangePosition.z - rangeScale.z * 0.5f, rangePosition.z + rangeScale.z * 0.5f);

        return targetPosition;
    }
    private Vector3 SetAngle(float radius,int angle)
    {
        Vector3 position = Vector3.zero;

        position.x = Mathf.Cos(angle) * radius;
        position.z = Mathf.Sin(angle) * radius;

        return position;
    }

    private IEnumerator Pursuit()
    {
        while (true)
        {
            // 이동 속도 설정 (배회할 때는 걷는 속도로 이동, 추적할 때는 뛰는 속도로 이동)
            navMeshAgent.speed = status.RunSpeed;
    
            // 목표위치를 현재 플레이어의 위치로 설정
            navMeshAgent.SetDestination(target.position);
    
            // 타겟 방향을 주시하도록 함
            LookRotationToTarget();
    
            // 타겟과의 거리에 따라 행동 선택 (배회, 추격, 원거리 공격)
            CalculateDistanceToTargetAndSelectState();
    
            yield return null;
        }
    }

    private IEnumerator Attack()
    {
        // 공격할 때는 이동을 멈추도록 설정
        navMeshAgent.ResetPath();
    
        while (true)
        {
            // 타겟 방향 주시
            LookRotationToTarget();
    
            // 타겟과의 거리에 따라 행동 선택 (배회, 추격, 원거리 공격)
            CalculateDistanceToTargetAndSelectState();
    
            if (Time.time - lastAttackTime > attackRate)
            {
                // 공격주지가 되어야 공격할 수 있도록 하기 위해 연재 시간 저장
                lastAttackTime = Time.time;
    
                // 발사체 생성
                GameObject clone = Instantiate(projectilePrefab,projectileSpawnPoint.position,projectileSpawnPoint.rotation);
                clone.GetComponent<EnemyProjectile>().Setup(target.position);
            }
    
            yield return null;
        }
    }

    private void LookRotationToTarget()
    {
        // 목표 위치
        Vector3 to = new Vector3(target.position.x,0,target.position.z);
        // 내 위치
        Vector3 from = new Vector3(transform.position.x,0,transform.position.z);
    
        // 바로 돌기
        transform.rotation = Quaternion.LookRotation(to-from);
    
        // 서서히 돌기
        //Quaternion rotation = Quaternion.LookRotation(to - from);
        //transform.rotation = Quaternion.Slerp(transform.rotation, rotation, 0.01f);
    }

    private void CalculateDistanceToTargetAndSelectState()
    {
        if (target == null) return;
    
        // 플레이어(target)와 적의 거리 계산 후 거리에 따라 행동 선택
        float distance = Vector3.Distance(target.position, transform.position);

        if (distance <= attackRange)
        {
            ChangeState(EnemyState.Attack);
        }
        else if (distance <= targetRecognitionRange)
        {
            ChangeState(EnemyState.Pursuit);
        }
        else if(distance >= pursuitLimitRange)
        {
            ChangeState(EnemyState.Wander);
        }
    }

    private void OnDrawGizmos()
    {
        // "배회" 상태일 때 이동할 경로 표시
        Gizmos.color = Color.black;
        Gizmos.DrawRay(transform.position,navMeshAgent.destination-transform.position);

        // 목표 인식 범위
        Gizmos.color = Color.red;
        Gizmos.DrawWireSphere(transform.position, targetRecognitionRange);
        
        // 추적 범위
        Gizmos.color = Color.green;
        Gizmos.DrawWireSphere(transform.position, pursuitLimitRange);

        // 공격 범위
        Gizmos.color =new Color(0.39f, 0.04f, 0.04f);
        Gizmos.DrawWireSphere(transform.position, attackRange);
    }
    
    /*●*/public void TakeDamage(int damage)
    /*●*/{
    /*●*/    bool isDie=status.DecreaseHP(damage);
    /*●*/
    /*●*/    if(isDie == true)
    /*●*/    {
    /*●*/        enemyMemoryPool.DeactiveEnemy(gameObject);
    /*●*/    }
    /*●*/}
}
```

(5) ImpactMemoryPool스크립트를 수정한다.(광선에 맞은 놈의 태그가 적이면 ImpactEnemy생성)

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public enum ImpactType { Normal=0, Obstacle, /*●*/Enemy,}

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
        /*●*/else if (hit.transform.tag == "ImpactEnemy")
        /*●*/{
        /*●*/    OnSpawnImpact(ImpactType.Enemy,hit.point,Quaternion.LookRotation(hit.normal));
        /*●*/}
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

(6) 무기 오브젝트의 ImpactMemoryPool 프로퍼티에 ImpactBlood프리팹을 할당한다.

(7) WeaponSetting스크립트를 수정한다.(무기의 공격력 설정)

```csharp
// 무기의 종류가 여러 종류일 때 공용으로 사용하는 변수들은  구조체로 묶어서 정의하면
// 변수가 추가/삭제될 때 구조체에 선언하기 때문에 추가/삭제에 대한 관리가 용이함

public enum WeaponName {  AssaultRifle=0}

[System.Serializable]
public struct WeaponSetting
{
    public  WeaponName  weaponName;       // 무기 이름
    /*●*/public int damage;      // 무기 공격력
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

(8) WeaponAssaultRifle스크립트를 수정한다.(무기가 쏘는 광선에 맞은 오브젝트의 태그가 적이면 적에게 데미지 주는 코드 추가)

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

            /*●*/if (hit.transform.tag == "ImpactEnemy")
            /*●*/{
            /*●*/    hit.transform.GetComponent<EnemyFSM>().TakeDamage(weaponSetting.damage);
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

(9) ImpactEnemy태그를 추가하고 Enemy프리팹의 태그를 바꿔준다.

이제 게임을 실행해서 적을 공격해보면 적의 몸에서 피가 튀는 것처럼 이펙트가 나오고 status의 HP는 기본적으로 100이고 소총의 공격력은 50이기 때문에 적은 2대 맞으면 비활성화 된다.