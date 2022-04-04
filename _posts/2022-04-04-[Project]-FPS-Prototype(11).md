---
layout: post
title:  "[Project] FPS Prototype(11)"
subtitle:   ""
categories: Unity
comments: true
---

<br>

적 캐릭터의 행동 중 대기, 배회, 추적하기에 대해 만들어보자.

# 적 캐릭터 이동 경로 설정 (Navigation Mesh)

**맵 정보가 바뀌면 바뀐 맵에 대해 Navigation 설정을 다시하면 된다.**

(1) Navigation 뷰를 열고 경로, 장애물로 사용되는 오브젝트들을 선택한 후 Object탭에 있는 Navigation Static을 체크해준다.

![image](https://user-images.githubusercontent.com/101051124/161471545-9278591e-44de-4fa9-9149-a6aacefc9501.png)

(2) Bake탭으로 가서 Bake버튼을 눌러서 맵 데이터를 생성해준다.

![image](https://user-images.githubusercontent.com/101051124/161471674-aa70dc05-3890-497b-b3d9-544d4562acea.png)

이제 이동가능, 이동불가능 구역이 나타난다.

# 적 캐릭터 FSM - 배회하기

(1) 적 캐릭터의 행동을 제어하는 EnemyFSM스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.AI;

public enum EnemyState {  None=-1,Idle = 0, Wander,}

public class EnemyFSM : MonoBehaviour
{
    private EnemyState enemyState = EnemyState.None;        // 현재 적 행동

    private Status status;      // 이동속도 등의 정보
    private NavMeshAgent navMeshAgent;      // 이동 제어를 위한 NavMeshAgent

    private void Awake()
    {
        status=GetComponent<Status>();
        navMeshAgent=GetComponent<NavMeshAgent>();

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

    private void OnDrawGizmos()
    {
        // "배회" 상태일 때 이동할 경로 표시
        Gizmos.color = Color.black;
        Gizmos.DrawRay(transform.position,navMeshAgent.destination-transform.position);
    }
}
```

(2) Enemy프리팹에 status, EnemyFSM스크립트와 NavMeshAgent컴포넌트를 추가해준다.

![image](https://user-images.githubusercontent.com/101051124/161477490-3611d1f9-e622-4262-b7d9-f08fe2539063.png)

이제 위와 같이 처음에는 가만히 있다가 몇 초후에 배회상태가 되는 걸 볼 수 있다.

***

# 적 캐릭터 FSM - 추적하기

(1) EnemyFSM스크립트를 수정한다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.AI;

public enum EnemyState {  None=-1,Idle = 0, Wander, /*●*/Pursuit,}

public class EnemyFSM : MonoBehaviour
{
    /*●*/[Header("Pursuit")]
    /*●*/[SerializeField]
    /*●*/private float targetRecognitionRange = 8;       // 인식 범위 (이 범위 안에 들어오면 "PurSuit" 상태로 변경)
    /*●*/[SerializeField]
    /*●*/private float pursuitLimitRange = 10;       // 추적 범위 (이 범위 바깥으로 나가면 "Wander" 상태로 변경)

    private EnemyState enemyState = EnemyState.None;        // 현재 적 행동

    private Status status;      // 이동속도 등의 정보
    private NavMeshAgent navMeshAgent;      // 이동 제어를 위한 NavMeshAgent
    /*●*/private Transform target;       // 적의 공격 대상 (플레이어)

    //private void Awake()
    public void Setup(Transform target)
    {
        status=GetComponent<Status>();
        navMeshAgent=GetComponent<NavMeshAgent>();
        /*●*/this.target=target;

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
            /*●*/// 타겟과의 거리에 따라 행동 선택 (배회, 추격, 원거리 공격)
            /*●*/CalculateDistanceToTargetAndSelectState();

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

            /*●*/// 타겟과의 거리에 따라 행동 선택 (배회, 추격, 원거리 공격)
            /*●*/CalculateDistanceToTargetAndSelectState();

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

    /*●*/private IEnumerator Pursuit()
    /*●*/{
    /*●*/    while (true)
    /*●*/    {
    /*●*/        // 이동 속도 설정 (배회할 때는 걷는 속도로 이동, 추적할 때는 뛰는 속도로 이동)
    /*●*/        navMeshAgent.speed = status.RunSpeed;
    /*●*/
    /*●*/        // 목표위치를 현재 플레이어의 위치로 설정
    /*●*/        navMeshAgent.SetDestination(target.position);
    /*●*/
    /*●*/        // 타겟 방향을 주시하도록 함
    /*●*/        LookRotationToTarget();
    /*●*/
    /*●*/        // 타겟과의 거리에 따라 행동 선택 (배회, 추격, 원거리 공격)
    /*●*/        CalculateDistanceToTargetAndSelectState();
    /*●*/
    /*●*/        yield return null;
    /*●*/    }
    /*●*/}

    /*●*/private void LookRotationToTarget()
    /*●*/{
    /*●*/    // 목표 위치
    /*●*/    Vector3 to = new Vector3(target.position.x,0,target.position.z);
    /*●*/    // 내 위치
    /*●*/    Vector3 from = new Vector3(transform.position.x,0,transform.position.z);
    /*●*/
    /*●*/    // 바로 돌기
    /*●*/    transform.rotation = Quaternion.LookRotation(to-from);
    /*●*/
    /*●*/    // 서서히 돌기
    /*●*/    //Quaternion rotation = Quaternion.LookRotation(to - from);
    /*●*/    //transform.rotation = Quaternion.Slerp(transform.rotation, rotation, 0.01f);
    /*●*/}

    /*●*/private void CalculateDistanceToTargetAndSelectState()
    /*●*/{
    /*●*/    if (target == null) return;
    /*●*/
    /*●*/    // 플레이어(target)와 적의 거리 계산 후 거리에 따라 행동 선택
    /*●*/    float distance = Vector3.Distance(target.position, transform.position);
    /*●*/
    /*●*/    if (distance <= targetRecognitionRange)
    /*●*/    {
    /*●*/        ChangeState(EnemyState.Pursuit);
    /*●*/    }
    /*●*/    else if(distance >= pursuitLimitRange)
    /*●*/    {
    /*●*/        ChangeState(EnemyState.Wander);
    /*●*/    }
    /*●*/}

    private void OnDrawGizmos()
    {
        // "배회" 상태일 때 이동할 경로 표시
        Gizmos.color = Color.black;
        Gizmos.DrawRay(transform.position,navMeshAgent.destination-transform.position);

        /*●*/// 목표 인식 범위
        /*●*/Gizmos.color = Color.red;
        /*●*/Gizmos.DrawWireSphere(transform.position, targetRecognitionRange);
        /*●*/
        /*●*/// 추적 범위
        /*●*/Gizmos.color = Color.green;
        /*●*/Gizmos.DrawWireSphere(transform.position, pursuitLimitRange);
    }
}
```

(2) EnemyMemoryPool 스크립트를 수정한다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class EnemyMemoryPool : MonoBehaviour
{
    /*●*/[SerializeField]
    /*●*/private Transform target;       // 적의 목표 (플레이어)
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

                item.transform.position = new Vector3(Random.Range(-mapSize.x * 0.49f, mapSize.x * 0.49f), 0,
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

        /*●*/item.GetComponent<EnemyFSM>().Setup(target);

        // 타일 오브젝트 비활성화
        spawnPointMemoryPool.DeactivatePoolItem(point);
    }
}
```

(3) EnemyMemoryPool오브젝트의 target변수에 Player오브젝트를 넣어준다.

![image](https://user-images.githubusercontent.com/101051124/161481294-e2450f7b-d2b4-4f05-94a3-b4068e6b231a.png)

이제 추적 범위가 나오고 플레이어가 빨간선 안으로 들어가면 추적을 하고 초록선 밖으로 나가면 배회하는 모습을 볼 수 있다.