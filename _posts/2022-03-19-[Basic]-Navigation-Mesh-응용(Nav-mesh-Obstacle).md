---
layout: post
title:  "[Basic] Navigation Mesh 응용(Nav mesh Obstacle)"
subtitle:   ""
categories: Unity
comments: true
---

<br>

이제 이동 가능한 장애물 설정에 대해 알아보자. 네비게이션 메시는 게임 시작전에 미리 Bake해서 생성한다. 그래서 게임 안에서 오브젝트가 이동 했을 때 이동한 오브젝트의 좌표를 기반으로 네비게이션 메시가 새로 생성되지 않는다. 그래서 게임에서 이동이 가능한 오브젝트는 Nav Mesh Obstacle 컴포넌트를 추가해서 실시간으로 네비게이션 메시 데이터를 관리한다. Nav Mesh Obstacle 컴포넌트(이동 오브젝트의 네비게이션 메시 설정에 사용되는 컴포넌트)는 아래와 같이 생겼다. 기능을 하나하나 설명하겠다.

![image](https://user-images.githubusercontent.com/101051124/159094047-8961ba8b-4116-4ff8-9a3b-958cf9c92821.png)

**Shape** : 장애물의 모양 (Box or Capsule)

**Center** : Shape의 중심 위치

**Carve** : Navigation Mesh에 공간을 비울지 (true : 비운다)

**Move Threshold** : 설정된 거리를 이동하면 오브젝트의 Navigation Mesh 데이터 갱신

**Time To Staionary** : 설정된 시간만큼 움직임이 업으면  "멈춰 있다"라고 인식

**Carve Only Stationary** : 멈춰 있을 때만 공간을 비울지 (true : 멈춰있을 때만 공간을 비움, false : 실시간으로 비움)

(참고)

Shape가 Box일 때 => Size : Box의 크기

Shape가 Capsule일 때 => Radius : Capsule의 반지름 / Height : Capsule의 높이

# 이동하는 장애물 만들기

(1) 우선 큐브 오브젝트 하나를 만들고 이름을 PatrolCube로 바꾸고 포지션을 (0, 1, -6)으로 바꾼다.

(2) 이동 경로로 사용할 빈 오브젝트 2개를 만들고 이름을 각각 Path01, Path02로 바꾼다음 포지션을 각각 (5.5, 1, -8), (-6.1, 1, -5)로 설정한다.

(3) 지정된 지점을 이동하는 SimplePatrol 스크립트를 만들어 보겠다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class SimplePatrol : MonoBehaviour
{
    [SerializeField]
    private Transform[] paths;      //순찰 경로
    private int currentPath = 0;    //현재 목표지점 인덱스
    private float moveSpeed = 3.0f;  //이동 속도

    private void Update()
    {
        // 이동 방향 설정 : (목표위치 - 내위치).정규화
        Vector3 direction=(paths[currentPath].position - transform.position).normalized;

        // 오브젝트 이동
        transform.position+=direction*moveSpeed*Time.deltaTime;

        // 목표 위치에 거의 도달했을 때
        if((paths[currentPath].position - transform.position).sqrMagnitude < 0.1f)
        {
            // 목표 위치 변경
            if (currentPath < paths.Length - 1) currentPath++;
            else currentPath = 0;
        }
    }
}
```

(참고) (a-b).sqrMagnitude : 계산된 a-b 거리 값을 제곱한 결과 값! 정확한 거리 값은 구할 수 없지만 둘 사이가 얼마나 가깝거나 먼지 대략적으로 구할 수 있다. Vector3.Distance()보다 연산속도가 빠르다.

(4) PatrolCube 오브젝트에 SimplePatrol 스크립트를 부착하고 경로들을 등록해준다.

(5) 게임을 실행하고 네비게이션뷰를 보면 다음과 같이 PatrolCube가 움직이는곳을 이동 불가능 구역으로 치지 않는 모습을 볼 수 있다.

![image](https://user-images.githubusercontent.com/101051124/159095692-79181edc-36c9-43de-b71e-8d8660275dca.png)

이걸 개선하기 위해서

(6) PatrolCube에 Nav Mesh Obstacle을 추가하고 Carve를 체크해주고 Carve Only Stationary는 체크 해제해준다.

![image](https://user-images.githubusercontent.com/101051124/159095875-61e1d73c-197c-4045-bf24-37ab9a743856.png)

(TIP) Move Threshold의 값을 낮게 설정할수록 정밀한 위치 계산이 되지만 게임이 무거워질 수 있기 때문에 적당한 거리를 설정해야 한다.

다시 게임을 실행하고 네비게이션 뷰를 보면

![image](https://user-images.githubusercontent.com/101051124/159095920-81cba4dd-eca7-4829-aa53-db31bce4a804.png)

PatrolCube의 움직임에 맞게 네비게이션 메시가 갱신되는 것을 볼 수 있다.

(TIP) 실시간으로 장애물의 위치가 갱신되는 오브젝트가 있을 경우 NavMeshAgent 컴포넌트의 Auto Repath가 체크되어 있는지 확인!!!