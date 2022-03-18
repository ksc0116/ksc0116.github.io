---
layout: post
title:  "[Basic] Navigation Mesh 응용(Off Mesh Link)"
subtitle:   ""
categories: Unity
comments: true
---

<br>

Navigation Mesh를 응용해보자. 사다리, 절벽과 같이 메시가 끊어진 곳을 이동하는 Off Mesh Link와

이동가능한 장애물을 설정하는 Nav Mesh Obstacle을 사용해보자.

프로젝트는 Navigation Mesh 기초에서 했던 것을 이어서 하겠다.

# Off Mesh Link

Off Mesh Link는 사다리, 암벽과 같이 수직으로 올라가거나 내려오는 길, 절벽 사이를 뛰어서 넘어가거나 낭떠러지 알래로 떨어지는 길과 같이 메시가 끊어져 있는 곳을 이동할 수 있게 설정하는 것이다.

## 자동(Auto) Off Mesh Link 설정 방법

(1) 자동으로 Off Mesh Link를 설정할 오브젝트들을 선택 (단일 or 복수)

(2) Navigation View - Object 탭의 "Generate OffMeshLinks" 체크

![image](https://user-images.githubusercontent.com/101051124/158814252-db85fb22-b104-4197-ab8e-0b4f835b3b3f.png)

(3) Navigation View - Bake 탭의 낙하 높이(Drop Height). 점프 거리(Jump Distance)를 설정하고, "Bake"를 눌러 데이터 저장

![image](https://user-images.githubusercontent.com/101051124/158814514-718a3306-08c4-41bd-921c-20b033aae99f.png)

그럼 다음과 같이 점프 거리가 4이하이고 낙하 높이가 4이하인 곳에 Off Mesh Link가 생성되는 걸 볼 수 있다.

![image](https://user-images.githubusercontent.com/101051124/158815867-38987e21-6c6f-4d5e-8465-e014a38aa541.png)

이제 게임 플레이를 해보면 원래는 갈 수 없고 뛰어내리지 못했던 곳을 플레이어가 이동하고 뛰어내린다.

![image](https://user-images.githubusercontent.com/101051124/158816422-a10586aa-d030-4c9e-b1e5-b23e718a6186.png)

위와 같이 점프를 하며

![image](https://user-images.githubusercontent.com/101051124/158816566-d71b57dc-72af-46ff-a5d1-45589a2e209a.png)

떨어지는 걸 볼 수 있다.

### 장점

게임월드에 배치된 많은 오브젝트의 Off Mesh Link를 한꺼번에 설정 가능

### 단점

-낙하 높이(Drop Height)와 점프 거리(Jump Distance)를 하나만 설정할 수 있기 때문에 다양한 지형을 세세하게 설정하는 것이 불가능(Off Mesh Link 데이터 소실의 위험)

-위로 올라가는 Off Mesh Link 설정 불가능

## 수동(Manual) Off Mesh Link 설정 방법

(1) 연결되는 두 지점으로 사용할 오브젝트 생성 및 배치

![image-20220317221939185](C:\Users\ksc52\AppData\Roaming\Typora\typora-user-images\image-20220317221939185.png)

사다리를 오르고 내리는걸 구현하기 위해 사다리 오브젝트의 자식 오브젝트로 스피어 오브젝트 2개를 생성하고 이름을 각각 Start,End로 바꾸고 Size를 0.4로 바꾼다. 그리고 Start의 위치를 (0, -1.4, -0.8)로설정한다. 그리고 End의 위치는 (0, 1.6, 0.8)로 설정한다.

![image](https://user-images.githubusercontent.com/101051124/158817664-c8a38bed-6423-4057-84ed-56f4ed348e45.png)

여기서 주의해야 할 점이 게임 오브젝트의 위치가 네비게이션 메시의 이동 경로 내에 포함되어야 정상적인 이동이 가능하다는 것이다. 이렇게 배치가 되었으면 Transform을 제외한 나머지 컴포넌트들을 끄거나 삭제한다.

(2) 사다리를 구성하고 있는 레더 오브젝트에 Off Mesh Link 컴포넌트를 생성하고, "Start", "End" 변수에 연결되는 두 지점을 설정한다.

![image](https://user-images.githubusercontent.com/101051124/158818536-fb186ab9-2bbd-4069-84bf-fd586b7e17ac.png)

그리고 Bidirectional을 체크하면 양방향으로 이동이 가능하지만, 체크가 안되어 있으면 Start->End로 단일방향으로만 이동할 수 있다. 이제 게임 실행을 해보면 다음과 같이 사다리를 타는 걸 볼 수 있다.

![image](https://user-images.githubusercontent.com/101051124/158818943-97d159e9-f3be-418d-8824-60b3862f910b.png)

### 장점

-지형에 따라 세세한 설정이 가능

-사다리/암벽과 같이 위로 올라가는 Off Mesh Link 설정 가능

### 단점

-Off Mesh Link로 연결이 필요한 모든 부분을 직접 설정해야 함

# 구역 설정

구역을 설정하는 것은 게임 월드에 다양한 지형이 존재할 때 어느 지형을 가로질러 가는 것이 더 빠르고 느린지 명시하기 위함이다.

구역은 Navigation View - Areas탭에서 설정 할 수 있다.

![image](https://user-images.githubusercontent.com/101051124/158819801-a8ccf9ab-b96a-4347-94c0-78cd45b15a2e.png)

다음과 같이 User3에 climb이라는 새로운 구역을 생성하고 Cost를 5로 설정한다.

## Navigation Areas에 등록된 구역을 게임오브젝트에 설정하는 방법

(1)  수동(Manual) Off Mesh Link 오브젝트는 Off Mesh Link 컴포넌트의 Navigation Area 변수 설정.

ex) 수동으로 오프 메시 링크를 설정했던 레더 오브젝트의 오프 메시 링크 컴포넌트를 다음과 같이 수정한다.



![image](https://user-images.githubusercontent.com/101051124/158820673-ac113360-c32e-4f50-a671-a9ae17f09018.png)

(2)  일반 게임오브젝트는 게임 오브젝트 선택 후 Navigation View - Object탭에서 Navigation Area 변수 설정

ex)

![image](https://user-images.githubusercontent.com/101051124/158821174-8e36cde5-e348-4f20-8549-c71a33318de7.png)

여기서 Tip) NavMeshAgent 컴포넌트를 사용하는 이동 에이전트의 Area Mask에서 이동 가능 / 불가능 구역 선택이 가능하다.

ex) Player 오브젝트를 봐보자.

![image](https://user-images.githubusercontent.com/101051124/158821749-aa6adbca-1fc0-4101-817b-9949cbcd864b.png)

저기서 체크가 되어 있는 구역은 이동이 가능하며, 체크가 해제되면 이동 불가능 구역이 된다.

이제 Off Mesh Link로 설정된 곳을 지나갈 때 코드를 이용해 애니메이션을 재생하거나 이동 방법을 선택할 수 있다.

**(TIP) NavMeshAgent 컴포넌트의 Auto Travers Off Mesh Link가 체크 해제되어 있으면 Off Mesh Link를 만나면 오브젝트가 멈추게 된다.**

![image](https://user-images.githubusercontent.com/101051124/159007802-d77684d2-7059-402f-adf9-27dc05d1df92.png)

지금 작성 할 스크립트는 수동으로 설정한 OffMeshLink에 대해 특정 이동방법을 부여한 스크립트이다. 이제 스크립트를 통해 제어를 해보자. 

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.AI;

public class OffMeshLinkClimb : MonoBehaviour
{
    [SerializeField]
    private int offMeshArea = 3;    //오프메시의 구역 (Climb)
    [SerializeField]
    private float climbSpeed = 1.5f;    //오르내리는 이동 속도
    private NavMeshAgent navMeshAgent;

    private void Awake()
    {
        navMeshAgent = GetComponent<NavMeshAgent>();
    }

    IEnumerator Start()
    {
        while (true)
        {
            //IsOnClimb() 함수의 반환 값이 true일 때 싸지 반복 호출

            yield return new WaitUntil(() => IsOnClimb());

            //올라가거나 내려오는 행동
            yield return StartCoroutine(ClimbOrDescend());
        }
    }

    public bool IsOnClimb()
    {
        //현재 오브젝트의 위치가 OffMeshLink에 있는지 (true / false)
        if (navMeshAgent.isOnOffMeshLink)
        {
            //현재 위치에 있는 OffMeshLink의 데이터
            OffMeshLinkData linkData = navMeshAgent.currentOffMeshLinkData;

            //설명: navMeshAgent.currentOffMeshLinkData.offMeshLink가
            //true이면 수동으로 생성한 OffMeshLink
            //false이면 자동으로 생성한 OffMeshLink

            //현재 위치에 있는 OffMeshLink가 수동으로 생성한 OffMeshLink이고, 장소 정보가 "Climb"이면
            if((linkData.offMeshLink != null) && (linkData.offMeshLink.area == offMeshArea))
            {
                return true;
            }
        }

        return false;
    }

    private IEnumerator ClimbOrDescend()
    {
        //네비게이션을 이용한 이동을 잠시 중지한다
        navMeshAgent.isStopped = true;

        // 현재 위치에 있는 OffMeshLink의 시작 / 종료 위치
        OffMeshLinkData linkData = navMeshAgent.currentOffMeshLinkData;
        Vector3 start = linkData.startPos;
        Vector3 end = linkData.endPos;

        // 오르내리는 시간 설정
        float climbTime = Mathf.Abs(end.y - start.y) / climbSpeed;
        float currentTime = 0.0f;
        float percent = 0.0f;

        while (percent < 1)
        {
            // 단순히 deltaTime만 더하면 무조건 1초 후에 percent가 1이 되기 때문에
            // climbTime 변수를 연산해서 시간을 조절한다
            currentTime += Time.deltaTime;
            percent=currentTime/climbTime;

            // 시간 경과(최대 1)에 따라 오브젝트의 위치를 바꿔준다.
            transform.position = Vector3.Lerp(start, end, percent);

            yield return null;
        }

        // OffMeshLink를 이용한 이동 완료
        navMeshAgent.CompleteOffMeshLink();
        // OffMeshLink 이동이 완료되었으니 네비게이션을 이용한 이동을 다시 시작한다
        navMeshAgent.isStopped=false;
    }
}
```

여기서 새로 알게 된 명령어 들을 정리해보자.

**yield return new WaitUntil(() => IsOnClimb());** : 코루틴에서 특정 작업이 완료될 때까지 기다리는 방법

**navMeshAgent.isOnOffMeshLink** : 현재 오브젝트의 위치가 OffMeshLink에 있는지 확인

**navMeshAgent.currentOffMeshLinkData** : OffMeshLinkData타입을 반환한다.(현재 위치에 있는 OffMeshLink의 데이터)

**navMeshAgent.currentOffMeshLinkData.offMeshLink** : 현재 위치에 있는 OffMeshLink가 수동으로 설치된거면 OffMeshLink가 반환되고, 자동으로 설치한거면 null을 반환한다.

나머지 새로운 기능들은 익히기에 어렵지 않다. 이제 다시 게임 월드로 돌아가 플레이어 오브젝트에 위의 스크립트를 컴포넌트로 부착해주면 사다리를 올라갈 때 속도가 느려지는 것을 확인할 수 있다.

이제 작성 할 스크립트는 자동으로 설정한 OffMeshLink에 대한 이동 방법을 결정할 것이다. 스크립트를 봐보자.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.AI;

public class OffMeshLinkJump : MonoBehaviour
{
    [SerializeField]
    private float jumpSpeed = 10.0f;    //점프 속도
    [SerializeField]
    private float gravity = -9.81f;         //중력 계수
    private NavMeshAgent navMeshAgent;

    private void Awake()
    {
        navMeshAgent = GetComponent<NavMeshAgent>();
    }

    IEnumerator Start()
    {
        while(true)
        {
            // IsOnJump() 함수의 반환값이 true일 때 까지 반복 호출
            yield return new WaitUntil(() => IsOnJump());

            // 점프 행동
            yield return StartCoroutine(JumpTo());
        }
    }

    public bool IsOnJump()
    {
        if (navMeshAgent.isOnOffMeshLink)
        {
            // 현재 위치에 있는 OffMeshLink의 데이터
            OffMeshLinkData linkData = navMeshAgent.currentOffMeshLinkData;

            // 설명: OffMeshLinkType은 Manual=0, DropDown=1, JumpAcross=2로
            // 자동으로 생성한 OffMeshLink의 속성 구분을 위해 사용(1,2)

            // 현재 위치에 있는 OffMeshLink의 OffMeshLinkType이 JumpAcross이면
            if(linkData.linkType == OffMeshLinkType.LinkTypeJumpAcross ||
                linkData.linkType == OffMeshLinkType.LinkTypeDropDown)
            {
                return true;
            }
        }
        return false;
    }

    IEnumerator JumpTo()
    {
        //네비게이션을 이욯안 이동을 잠시 중지
        navMeshAgent.isStopped = true;

        // 현재 위치에 있는 OffMeshLink의 시작/종료 위치
        OffMeshLinkData linkData=navMeshAgent.currentOffMeshLinkData;
        Vector3 start = linkData.startPos;
        Vector3 end = linkData.endPos;

        // 뛰어서 이동하는 시간 설정
        float jumpTime = Mathf.Max(0.3f,Vector3.Distance(start,end) / jumpSpeed);
        float currentTime = 0.0f;
        float percent = 0.0f;

        // y 방향의 초기 속도
        float v0 = (end - start).y - gravity;

        while(percent < 1)
        {
            // 단순히 deltaTime만 더하면 무조건 1초 후에 percent가 1이 되기 때문에
            // jumpTime 변수를 연산해서 시간을 조절한다.
            currentTime+=Time.deltaTime;
            percent=currentTime/jumpTime;

            // 시간 경과(최대 1)에 따라 오브젝트의 위치(x,z)를 바꿔준다
            Vector3 position = Vector3.Lerp(start,end,percent);

            // 시간 경과에 따라 오브젝트의 위치(y)를 바꿔준다
            // 포물선 운동 : 시작위치 + 초기속도*시간 +중력*시간제곱
            position.y=start.y +(v0*percent) + (gravity*percent*percent);

            // 위에서 게산한 x, y, z 위치 값을 실제 오브젝트에 대입
            transform.position = position;

            yield return null;
        }

        // OffMeshLink를 이용한 이동 완료
        navMeshAgent.CompleteOffMeshLink();

        // OffMeshLink 이동이 완료되었으니 네비게이션을 이용한 이동을 다시 시작한다.
        navMeshAgent.isStopped=false;
    }
}
```

여기서는 공식들이 나오는데 우선 초기속도 구하는 공식을 봐보자.

### 초기속도 구하는 공싱

y 방향의 초기속도 : float v0 = (end - start).y - gravity;

(초기 속도 v0)=목표 위치.y - 현재 위치.y - 중력 계수

### 포물선에서 현재 높이(y)

 position.y=start.y +(v0xpercent) + (gravity*percent*percent);

현재 높이 = 시작 위치.y +(초기 속도 x 시간) + (중력 계수 x 시간제곱)

### 새로 알게 된 함수

**Mathf.Max(값1,값2)** : 2개의 값 중 큰 값을 반환한다.

이제 위 스크립트를 플레이어 오브젝트에 컴포넌트로 추가하고 게임을 실행하면 절벽을 뛰어 내릴 때, 절벽 사이를 뛰어서 넘어 갈 때 포물선 운동을 하는걸 볼수 있다.