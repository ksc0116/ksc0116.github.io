---
layout: post
title:  "[Basic] Navigation Mesh 기초"
subtitle:   ""
categories: Unity
comments: true
---

<br>

경로 탐색 AI, Navigation Mesh에 대해 알아보자.

우선 기초지식을 차근차근 알아가 보자.

# NavMeshAgent

**Navigation Mesh** : 게임월드에 이동 가능, 이동 불가능 등의 경로를 데이터로 저장

**NavMeshAgent**는 네비게이션 메시 정보를 바탕으로 이동하는 **오브젝트**이다.

NavMeshAgent의 컴포넌트는 아래와 같이 생겼다.

![image](https://user-images.githubusercontent.com/101051124/158793979-a4e609f4-8466-4a16-91fd-942c46fddb8f.png)

# OffMeshLink

**OffMeshLink** : 연결이 끊어져 있는 절벽, 낭떠러지, 사다리 등을 이동 가능하게 설정

OffMeshLink 컴포넌트는 아래와 같이 생겼다.

![image](https://user-images.githubusercontent.com/101051124/158794526-e6b1a071-00ae-4afb-95b2-cfcb43698c50.png)

# NavMeshObstacle

**NavMeshObstacle** : 이동하는 장애물의 네비게이션 메시 정보를 실시간으로 설정

NavMeshObstacle 컴포넌트는 아래와 같이 생겼다.

![image](https://user-images.githubusercontent.com/101051124/158794815-7b3bcda5-6290-4613-b077-37249efc39d0.png)

# 네비게이션 메시 설정방법

게임 월드에 네비게이션 월드 설정을 하려면 상단에 Window - AI - Navigation을 누르면 Navigation View가 나온다. 

![image](https://user-images.githubusercontent.com/101051124/158795977-dfdfe7ef-10a4-4cb0-92b8-57c396dbf61f.png)

4개의 탭이 있는데 차근차근 살펴보자.

# Navigation View - Agents

**Navigation View- Agents** : 네비게이션 메시 정보를 바탕으로 움직이른 에이전트에 대한 설정(NavMeshAgent 컴포넌트)

![image](https://user-images.githubusercontent.com/101051124/158796312-64d8fc23-dd1d-4925-a5ba-07d887a8409e.png)

**Agent Type** : (에이전트 속성), +버튼을 눌러서 새로운 에이전트 속성을 추가할 수 있다.

**Name** : Agent Type에 보여지는 이름

**Radius** : 에이전트의 반지름

**Height** : 에이전트의 높이 (키)

**Step Height** : 오르내릴 수 있는 계단의 높이

**Max Slope** : 올라갈 수 있는 경사 각도

# Navigation View - Areas

**Navigation View - Areas** : 네비게이션 메시로 사용되는 오브젝트들의 구영 설정

![image](https://user-images.githubusercontent.com/101051124/158797144-5cd8a484-dec5-4146-896e-5e5d84cf9148.png)

**Name** : 구역 이름으로 기본으로 Walkable(이동가능), Not Walkable(이동 불가능), Jump(뛰기)가 제공되고, User3부터 원하는 구역을 추가로 설정할 수 있다.

**Cost** : 구역과 함께 등록, 이동하는데 소요되는 비용 (1이상) 경로를 탐색할 때 Cost 정보를 기준으로 최단거리를 찾게 된다. Cost가 2인 Jump는 Cost가 1인 Walkable을 지나갈 때 보다 2배의 시간이 걸린다는 뜻으로 사용된다. 

(실제 오브젝트의 물리적인 이동 속도가 느려지는 것이 아닌 경로를 게산할 때 활용된다)

# Navigation View - Bake

 **Navigation View - Bake** : 네비게이션 메시 데이터를 생성

![image](https://user-images.githubusercontent.com/101051124/158798022-cbb41082-6d0c-477f-8b61-1859982d4117.png)

### Baked Agent Size

**Agent Radius** : 에이전트가 지나갈 수 있는 반지름

**Agent Height** : 에이전트가 아래로 지나갈 수 있는 높이

**Agent Slope** : 에이전트가 올라갈 수 있는 경사 각도

**Step Height** : 에이전트가 오르내릴 수 있는 계단의 높이

### Generated Off Mesh Links

오프 메시 링크는 올라가기 힘든 언덕, 사다리, 절벽 등을 연결해서 이동 가능하게 만드는 옵션이다

**Drop Height** : 이동할 수 있는 절벽 아래의 높이

**Jump Distance** : 뛰어서 넘을 수 있는 절벽 거리

### Bake 버튼

Navigation에 설정된 옵션들을 바탕으로 네비게이션 정보를 데이터로 굽는다.

# Navigation View - Object

현재 씬에 있는 오브젝트 설정 (하나 or 다수)

![image](https://user-images.githubusercontent.com/101051124/158799189-f3136714-b4bf-4486-9b01-b22b36687acd.png)

네비게이션 메시 데이터로 사용하려면 오브젝트에 네비게이션 스태틱이 체크 되어 있어야함

### Scene Filter

현재 씬에서 원하는 오브젝트만 선택해서 볼 수 있다.(Mesh Renderer컴포넌트, Terrain 컴포넌트 선택 가능)

### 선택된 오브젝트

**Navigation Static** : 네비게이션 메시로 사용할지 설정

**Generate OffmeshLinks** : 자동으로 Off Mesh를 생성할지 설정

**Navigation Area** : 해당 오브젝트의 구역 설정 (설정되는 구역에 따라 해당 오브젝트의 Cost가 설정된다)

# 게임 오브젝트의 "Navigation Static" 설정 방법

원하는 게임 오브젝트 선택 후

방법 1. Inspector View의 Static - Navigation Static 선택

방법 2. Navigation View의 Object 탭에 있는 "Navigation Static" 변수 체크



이제 이렇게 스태틱 설정까지 했으면 네비게이션뷰의 베이크탭으로 가서 베이크 버튼을 누른다. 그럼 다음과 같은 화면이 된다.

![image](https://user-images.githubusercontent.com/101051124/158800453-831e1906-ff30-4a54-b576-68c9d1c306f8.png)

(참고) : 맵에 새로운 바닥, 장애물을 배티하였다면 Navigation Static 설정과 Bake를 다시 진행해야 한다.

씬 뷰에서 하늘색으로 표시 된 부분이 이동 가능한 부분이다.(네비게이션 뷰가 활성화 되어 있어야 네비게이션 메시가 출력 됨)

이제 웬만큼 설명은 다 한 것 같다. 

# NavMeshAgent 기반의 캐릭터 이동

이동 할 플레이어 오브젝트를 다음과 같이 생성해주자.

![image](https://user-images.githubusercontent.com/101051124/158801260-f41e9f1d-3d33-4e41-a49a-132ddf858aef.png)

네비게이션 메시 정보를 바탕으로 이동하려면 Nav Mesh Agent 컴포넌트가 필요하다. 플레이어 오브젝트의 컴포넌트에 추가해주자.

![image](https://user-images.githubusercontent.com/101051124/158801590-e1a5a26d-3268-439b-8efe-45a18585553c.png)

# Nav Mesh Agent

네비게이션 메시 정보를 기반으로 이동하는 에이전트

### Steering (에이전트 이동)

**Speed** : 이동 속도

**Angular Speed** : 방향을 바꿀 때의 회전 속도

**Acceleration** : 가속도 (정지 상태에서 이동속도가 될 때까지 적용)

**Stopping Distance** : 목적지가 이 값까지 가까워지면 이동을 멈춘다.

**Auto Breaking** : 목적지에 가까워지면 멈추는 기능(목적지에 도착해도 에이전트를 멈추지 않을 때 사용)

(여러 목적지를 계속 탐색하는 순찰 오브젝트에 주로 사용)

### Obstacle Avoidance (장애물 회피)

**Radius** : 장애물을 회피할 때 에이전트의 반지름

**Height** : 에이전트의 높이

**Quality** : 장애물과 충돌 수준 (None이면 뚫고 지나간다)

**Priority** : 장애물과 충돌했을 때의 우선순의(낮을수록 높다)

(이동 중인 두 에이전트의 모든 조건이 동일할 때 Priority가 낮은 에이전트가 더 우선권을 가지고 경로를 탐색한다)

### Path Finding (경로 탐색)

**Auto Traverse Off Mesh Link** : 오프 메시 링크가 있을 경우 자동으로 탐색해서 찾아갈지 설정

**Auto Repath** : 이동 중에 경로 탐색을 다시 할지 설정

(true : 이동 중에 장애물 등ㅇ로 막혔을 때 자동으로 재 계산)

**Area Mask** : 해당 에이전트의 이동 가능한 구역 지정



이제 네비 메시 에이전트 정보를 기반으로 이동하는 Movement3D 스크립트를 작성해보자.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.AI; //유니티에서 만든 AI를 사용하려면 선언해야됨

public class Movement3D : MonoBehaviour
{
    [SerializeField]
    private float moveSpeed = 5.0f;
    private NavMeshAgent navMeshAgent;
    void Awake()
    {
        navMeshAgent=GetComponent<NavMeshAgent>();
    }

    public void MoveTo(Vector3 goalPosition)
    {
        //기존에 이동 행동을 하고 있었다면 코루틴 중지
        StopCoroutine("OnMove");

        //이동 속도 설정
        navMeshAgent.speed = moveSpeed;

        //목표지점 설정 (목표까지의 경로 계산 후 알아서 움직인다)
        navMeshAgent.SetDestination(goalPosition);

        //이동 행동에 대한 코루틴 시작
        StartCoroutine("OnMove");
    }
    
    IEnumerator OnMove()
    {
        while (true)
        {
            //목표 위치(navMeshAgent.destination)와 내 위치(transform.position)의 거리가 0.1미만일 때
            //즉, 목표 위치에 거의 도착 했을 때
            if(Vector3.Distance(navMeshAgent.destination,transform.position) < 0.1f)
            {
                //내 위치를 목표 위치로 설정
                transform.position = navMeshAgent.destination;

                //SetDestination()에 의해 설정된 경로를 초기화. 이동을 멈춘다
                navMeshAgent.ResetPath();

                break;
            }
            yield return null;
        }
    }
}
```

**navMeshAgent.SetDestination(Vector3 Position);** : position을 목표지점으로 설정

(목표지점만 설정되면 경로를 탐색해서 자동으로 이동한다)

**float distance = Vector3.Distance(Vector3 a, Vector3 b);** : a와 b 두 벡터 사이의 거리 값을 구한다.

**NavMeshAgent.destination** : SetDestination()으로 설정한 목표지점 Vector3 정보가 저장되어 있다.

**navMeshAgent.Resetpath();** : 현재 설정되어 있는 이동 경로를 초기화하여 이동을 멈추게 한다.

이제 Movement3D를 활용한 PlayerController 스크립트를 작성해보자

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    private Movement3D movement3D;
    private void Awake()
    {
        movement3D=GetComponent<Movement3D>(); 
    }
    void Update()
    {
        //마우스 왼쪽 클릭을 했을 때
        if (Input.GetMouseButtonDown(0))
        {
            RaycastHit hit;

            //Camera.main : 태그가 "Camera"인 오브젝트 = "Main Camera"
            //카메라로부터 마우스 좌표(Input.mousePosition) 위치를 관통하는 광선 생성
            //ray.origin : 광선의 시작 위치
            //ray.direction : 광선의 진행 방향
            Ray ray= Camera.main.ScreenPointToRay(Input.mousePosition);

            //Physics.Raycast() : 광선을 발사해서 부딪히는 오브젝트 검출
            //(광선에 부딪히는 오브젝트가 있으면 true 반환)
            //ray.origin 위치로부터 ray.direction방향으로 무한한 길이 Mathf.Infinity)의 광선 발사
            //광선에 부딪히는 오브젝트의 정보를 hit에 저장
            if(Physics.Raycast(ray, out hit, Mathf.Infinity))
            {
                //hit.transform.position : 부짇힌 오브젝트의 위치
                //hit.point : 광선과 오브젝트가 부딪힌 세부 위치

                //hit.point를 목표 위치로 이동
                movement3D.MoveTo(hit.point);
            }
        }
    }
}
```

이제 게임 실행을 해보면 내가 클릭한 위치로 플레이어 오브젝트가 이동하는것을 볼 수 있다. 하지만 현재 이동 경로가 없는 사다리,경사면,절벽을 클릭하면 근처까지 간 다음 멈추게 된다.

![image](https://user-images.githubusercontent.com/101051124/158808457-402047df-f008-4bbe-a56d-d23963874507.png)

빨간원이 내가 클릭한 위치인데 올라가지 못하고 근처에서 멈춰있다.