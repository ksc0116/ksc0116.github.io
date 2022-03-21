---
layout: post
title:  "[Project] Roll A Ball"
subtitle:   ""
categories: Unity
comments: true
---

# 게임판 생성

(1) 게임의 바닥으로 사용 할 Cube오브제트를 생성하고 이름을 Ground로 바꿔준다. 그리고 Ground의 material을 위해 Material을 하나 생성해주고 Rendering Mode를 Fade로 해준다.

**Rendering Mode : Fade** : png 파일에 알파 정보가 있을 때 알파값 처리

![image](https://user-images.githubusercontent.com/101051124/159198741-639f03a5-44c4-4406-8595-252c132a16a3.png)

(2) Ground 머테리얼의 Albedo에 그라운드 소스를 입히고 Tilling을 X:4, Y:4로 설정한다.

**Tilling** : 모델링에 텍스처를 몇 번 반복해서 표현 할 것인지 설정

![image](https://user-images.githubusercontent.com/101051124/159198883-2d039266-b4e3-4dd6-afd4-fec5bef2219c.png)

(3) 만든 머테리얼을 Ground 오브젝트에 입히고 Ground의 크기를 20, 0.1, 20으로 바꾸고 MainCamera의 위치를 0, 10, -10으로 회전을 45, 0, 0으로 설정한다.

(4) MainCamera의 Clear Flags를 Solid Color로 배경색상을 검은색으로 설정

![image](https://user-images.githubusercontent.com/101051124/159199065-2159e401-311c-479e-b381-e45d2985b07d.png)

그럼 위 그림의 왼쪽과 같은 판이 만들어진다.

# 플레이어 설정

(1) 플레이어로 사용할 스피어 오브젝트를 생성하고 이름을 Player로 설정한다.

(2) 오브젝트의 이동을 제어하는 Movement3D 스크립트를 생성한다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Movement3D : MonoBehaviour
{
    [SerializeField]
    private float moveSpeed;
    private Rigidbody rigid;

    private void Awake()
    {
        rigid = GetComponent<Rigidbody>();
    }

    public void MoveTo(Vector3 direction)
    {
        Vector3 moveForce = Vector3.zero;

        direction.y = 0;
        moveForce = direction.normalized * moveSpeed;

        rigid.AddForce(moveForce,ForceMode.Impulse);
    }
}
```

(3) 플레이어를 제어하는 PlayerController스크립트를 만들어준다,

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    private Movement3D movement;

    private void Awake()
    {
        movement = GetComponent<Movement3D>();
    }

    private void Update()
    {
        float hAxis = Input.GetAxisRaw("Horizontal");
        float vAxis = Input.GetAxisRaw("Vertical");

        if(hAxis!= 0 || vAxis != 0)
        {
            movement.MoveTo(new Vector3(hAxis, 0, vAxis));
        }
    }
}
```

(4) Movement3D스크립트와 PlayerController스크립트를 Player오브젝트에 컴포넌트로 추가해주고 RigidBody컴포넌트도 추가해준다.

(5) 카메라가 플레이러를 따라다니고 회전,스크롤링이 가능하도록 CameraController스크립트를 만들어준다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CameraController : MonoBehaviour
{
    [SerializeField]
    private Transform target;
    [SerializeField]
    private float minDistance = 3;
    [SerializeField]
    private float maxDistance = 30;
    [SerializeField]
    private float wheelSpeed = 500;
    [SerializeField]
    private float xMoveSpeed=500;   // 카메라의 y축 회전 속도
    [SerializeField] 
    private float yMoveSpeed=250;  // 카메라의 x축 회전 속도
    private float yMinLimit = 5;        // 카메라 x축 외전 제한 최소값
    private float yMaxLimit = 80;        // 카메라 x축 외전 제한 최대값
    private float x, y;
    private float distance;

    private void Awake()
    {
        distance=Vector3.Distance(transform.position, target.position);

        Vector3 angle = transform.eulerAngles;
        x = angle.y;
        y = angle.x;
    }
    private void Update()
    {
        if(target==null) return;

        if (Input.GetMouseButton(1))
        {
            x+=Input.GetAxis("Mouse X")*xMoveSpeed*Time.deltaTime;
            y-=Input.GetAxis("Mouse Y")*yMoveSpeed*Time.deltaTime;

            y = ClampAngle(y, yMinLimit, yMaxLimit);

            transform.rotation = Quaternion.Euler(y,x,0);
        }
        distance-=Input.GetAxis("Mouse ScrollWheel")*wheelSpeed*Time.deltaTime;
        distance=Mathf.Clamp(distance, minDistance, maxDistance);
    }
    private void LateUpdate()
    {
        if (target == null) return; 

        transform.position=transform.rotation*new Vector3(0,0,-distance)+target.position;
    }
    private float ClampAngle(float angle,float min,float max)
    {
        if (angle < -360) angle += 360;
        if (angle > 360) angle -= 360;

        return Mathf.Clamp(angle, min, max);
    }
}
```

# 코인 오브젝트와 코인 획득 이펙트

(1) 코인 오브젝트로 사용할 실린더 오브젝트를 생성하고 이름을 Coin으로 바꿔준다.

(2) Coin을 위한 머테리얼을 생성하고 이름을 Coin으로 바꾼다음 색상을 노란색으로 설정한다.

(3) Coin 머테리얼의 Emission을 체크하고 색사을 각각 180, 18, 18로 설정하고 Coin오브젝트에 입혀준다.

(4) Coin오브젝트의 트랜스폼을 위치 : 0, 0.55, 0 회전 : 0, 0, 90 크기 : 1, 0.05, 1로 바꿔준다.

(5) Coin오브젝트를 프리펩으로 만들어주고 하이어라키창에 있는 Coin오브젝트는 삭제한다.

(6) Coin을 획득하였을 때 나타는 이펙트 제작을 위해 파티클 시스템을 생성하고 이름을 CoinEffect로 저장한다.

(7) 제작이 완료된 CoinEffect를 프리펩으로 저장해주고 재생이 완료된 CoinEffect는 자동삭제해주는 ParticleAutoDestroyer스크립트를 만들어준다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ParticleAutoDestroyer : MonoBehaviour
{
    private ParticleSystem particle;

    private void Awake()
    {
        particle = GetComponent<ParticleSystem>();
    }

    private void Update()
    {
        // 파티클이 재생중이 아니면
        if (particle.isPlaying == false)
        {
            Destroy(gameObject);
        }
    }
}
```

(8) 작성이 완료된 스크립트를 CoinEffect프리펩에 컴포넌트로 추가해준다.

# 플레이어와 코인 충돌 처리

(1) Coin오브젝트의 제어와 플레이어와 Coin오브젝트의 충돌 처리를 위해 CoinController스크립트를 만들어준다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CoinController : MonoBehaviour
{
    [SerializeField]
    private GameObject coinEffectPrefab;
    private float rotateSpeed = 100.0f;

    private void Update()
    {
        // 코인 오브젝트 회전
        transform.Rotate(Vector3.right*rotateSpeed*Time.deltaTime);
    }

    private void OnTriggerEnter(Collider other)
    {
        // 코인 오브젝트 획득 효과 생성
        GameObject clone = Instantiate(coinEffectPrefab);
        clone.transform.position = transform.position;

        Destroy(gameObject);
    }
}
```

(2) 작성이 완료된 스크립트를 Coin프리펩에 컴포넌트로 추가해주고 Coin Effect Prefab에 미리 만들어둔 CoinEffect프립펩을 할당하고 Coin오브젝트의 콜라이더에서 is Trigger를 체크해준다.

(3) CoinEffect 프리펩을 선택하고 파티클이 재생될 때 소리가 나도록 Audio Source 컴포넌트를 추가해준다.

(4) 오디오 소스 컴포넌트에 있는 오디오 클립에 45번 사운드를 넣어준다.

(5) 각 스테이지에 맞게 만들어둔 코인 프리펩을 미리 배치해준다.

Coin : 5, 0.55, 0

Coin(1) : 0, 0.55, 5

Coin(2) : -5, 0.55, 0

Coin(3) : 0, 0.55, -5

(6) CoinEffect의 파티클 시스템 컴포넌트의 항목 중 Renderer에 가서 Order In Layer를 1로 바꿔준다.

# 가속 발판 만들기

가속 발판은 발판에 부딪힌 플레이어를 특정 힘과 방향으로 밀어 멀리 밀어내는 오브젝트이다.

(1) 가속 발판으로 사용 할 Cube오브젝트를 하나 생성하고 이름을 AccelerationPlatform으로 설정해준다.

(2) 가속 발판을 위한 머테리얼을 만들고 이름을 Platform으로 바꾸고 Albedo에 Groud이미지를 넣어주고 가속 발판에 입혀준다.

(3) 가속 발판의 트랜스폼을 각각 위치 : 12.5, 1, -2.5 회전 : 0, 0, 25 크기 : 5, 0.1, 5로 설정해주면 다음과 같이 셋팅이 된다.

![image](https://user-images.githubusercontent.com/101051124/159206595-58380443-d22c-4abd-8936-67a0f8ed9a88.png)

(4) 발판을 밟았을 때 플레이어를 해당방향으로 이동시키기 위해 Movement3D스크립트를 수정한다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Movement3D : MonoBehaviour
{
    [SerializeField]
    private float moveSpeed;
    private Rigidbody rigid;

    private void Awake()
    {
        rigid = GetComponent<Rigidbody>();
    }

    public void MoveTo(Vector3 direction,float force=0)
    {
        Vector3 moveForce = Vector3.zero;

        if (force == 0)
        {
            direction.y = 0;
            moveForce = direction.normalized * moveSpeed;
        }
        else
        {
            moveForce = direction * force;
        }

        //moveForce의 힘으로 오브젝트를 민다.
        rigid.AddForce(moveForce);
    }
}
```

(5) 플레이어를 가속해서 이동시키는 AcclerationPlatform을 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class AccelerationPlatform : MonoBehaviour
{
    [SerializeField]
    private float accelForce;
    [SerializeField]
    private Vector3 direction;
    private AudioSource audio;

    private void Awake()
    {
        audio = GetComponent<AudioSource>();
    }

    private void OnCollisionEnter(Collision collision)
    {
        if (collision.gameObject.tag == "Player")
        {
            // 오디오 재생
            audio.Play();

            //발판에 부딪힌 오브젝트(플레이어)에게 방향(direction)과 힘(accelForce)전달
            collision.gameObject.GetComponent<Movement3D>().MoveTo(direction,accelForce);
        }
    }
}
```

(6) 작성된 스크립트를 가속 발판에 추가하고 AccelForce를 1000, direction을 1, 0, 0으로 설정해주고, 소리재생을 위해 AudioSource컴포넌트를 추가해준 다음 11번 사운드를 클립에 넣어주고 Play On Awake를 해제한다.

![image](https://user-images.githubusercontent.com/101051124/159207956-e688f986-0563-447c-8b4c-f1c3f630bfee.png)

그럼 다음과 같이 가속 발판 방향으로 플레이어가 날아간다.

# 점프 발판 만들기

(1) 점프 발판으로 사용 할 Cube 오브젝트를 생성하고 이름을 JumpPlatform으로 바꿔준다.

(2) 가속되는 방향을 위로 설정하면 점프가 되기 때문에 AccelerationPlatform스크립트를 컴포넌트로 추가하고 AccelForce를 1000, direction을 0, 1, 0으로 설정한다.

(3) 소리재생을 위해 AudioSource컴포넌트를 추가해준 다음 43번 사운드를 클립에 넣어주고 Play On Awake를 해제한다.

(4) 점프 발판의 트랜스폼을 각각 위치 : 12.5, 0, -7.5 회전 : 0, 0, 0 크기 : 5, 0.1, 5로 설정해준다.

(5) 머테리얼을 Platform 머테리얼을 추가해주면 다음과 같은 모습이 된다.

![image](https://user-images.githubusercontent.com/101051124/159208500-4340406d-ad97-4461-86cb-de81cca30d06.png)

그리고 플레이어가 점프 발판을 밟으면 위로 뛰어 오르는 모습을 볼 수 있다.

![image](https://user-images.githubusercontent.com/101051124/159208549-35c53ead-000e-4dac-a645-26579d38ca8d.png)

# 이동 발판 만들기

이동 발판은 설정해둔 좌표를 계속해서 움직에 만든다.

(1) 이동발판과 이동지점을 관리하는 빈 오브젝트를 만들어서 이름을 MovePlatform으로 바꾸고 위치는 리셋한다.

(2) 실제 이동 하는 오브젝트로 사용할 오브젝트를 MovePlatform오브젝트의 자식으로 만들고 이름을 MpvingPlatform으로 바꾼다.

(3) MovingPlatform의 위치를 12.5, 0, 7.5로 설정한다.

(4) 화면에 발판으로 보여지는 Cube 오브젝트를 생성하고 이름을 Platform으로 바꾸고 MovingPlatform의 자식으로 넣어주고 위치 : 0, 0, 0 크기 : 5, 0.1, 5로 설정해준다.

(5) 머테리얼을 Platform머테리얼을 넣어준다.

(6) 단순히 보여주는 역할만 하도록 박스 콜라이더 컴포넌트르 지워준다.

그럼 다음과 같은 모습이 된다.

![image](https://user-images.githubusercontent.com/101051124/159209795-ba5d94e4-7ad7-475b-940d-5f2912836365.png)

(7) 이제 MovingPlatform에 이동 지점으로 사용 할 빈 오브젝트를 생성하고(경로에 따라 갯수가 달라짐 현재 프로젝트는 3개만 생성) 이름을 각각 WayPoint01,WayPoint02,WayPoint03으로 바꾼다.

(8)각각의 위치를 WayPoint01 : 12.5, 0 ,7.5 / WayPoint02 : 22.5, 0, 7.5 / WayPoint03 : 22.5, 0, -2.5로 설정해주고 MovePlatform의 자식으로 넣어준다.

(9) 이동 발판의 이동제어를 위해 MovingPlatform스크립트를 만들어 준다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class MovingPlatform : MonoBehaviour
{
    [SerializeField]
    private Transform[] wayPoints;      // 이동 가능한 지점
    [SerializeField]
    private float waitTime;                 //WayPoint 도착 후 대기시간
    [SerializeField]
    private float moveSpeed;            //이동 속도
    private int wayPointCount;          //이동 가능한 wayPoint 개수
    private int currentIndex = 0;       //현재 wayPoint 인덱스

    private void Awake()
    {
        wayPointCount = wayPoints.Length;

        //최초 오브젝트 위치 설정
        transform.position = wayPoints[currentIndex].position;

        currentIndex++;

        StartCoroutine("MoveTo");
    }

    private IEnumerator MoveTo()
    {
        while (true)
        {
            //wayPoints[currentIndex].position까지 이동
            yield return StartCoroutine("Movement");

            //waitTime시간 동안 잠시 대기
            yield return new WaitForSeconds(waitTime);

            //다음 웨이포인트 설정
            if (currentIndex < wayPointCount - 1)
            {
                currentIndex++;
            }
            else
            {
                currentIndex = 0;
            }
        }
    }

    private IEnumerator Movement()
    {
        while (true)
        {
            //목표 위치로 가는 이동 방향 = 목표위치 - 내 위치(normalized를 해주지 않으면 멀수록 속도가 빠름)
            Vector3 direction=(wayPoints[currentIndex].position - transform.position).normalized;
            //오브젝트 이동
            transform.position+=direction*moveSpeed*Time.deltaTime;

            //목표 위치에 거의 도달했을 때 (현재 위치와 목표 위치와의 거리값이 0.1 미만일 때)
            if (Vector3.Distance(transform.position, wayPoints[currentIndex].position) < 0.1f)
            {
                //현재 위치를 목표 위치와 동일하게 설정
                transform.position=wayPoints[currentIndex].position;
                // 이동 중지
                break;
            }
            yield return null;
        }
    }

    private void OnCollisionEnter(Collision collision)
    {
        if (collision.gameObject.tag == "Player")
        {
            //플레이어 오브젝트를 플랫폼의 자식으로 설정
            collision.transform.SetParent(transform);
        }
    }

    private void OnCollisionExit(Collision collision)
    {
        if (collision.gameObject.tag == "Player")
        {
            //플레이어 오브젝트를 플랫폼의 자식에서 해제
            collision.transform.SetParent(null);
        }
    }
}
```

작성이 완료된 스크립트를 MovingPlatform의 컴포넌트로 추가해주고 충돌 처리를 위해 박스 콜라이더 컴포넌트를 추가해준다. 그리고 필드들은 다음과 같이 설정한다.

![image](https://user-images.githubusercontent.com/101051124/159211701-456dd968-18f6-43cd-9368-88a5ef0718b8.png)

게임을 실행하면 이동 발판은 설정해둔 지점을 반복해서 이동하며 플레이어가 발판위에 올라가면 플레이어 오브젝트는 이동 발판의 자식으로 설정된다.

# 순간이동 포탈 만들기

순간이동 포탈은 플레이어를 특정 위치로 이동시킨다.

(1) 먼저 포탈로 보이는 이펙트를 제작한다. 두 개의 이펙트를 사용해 하나의 포탈로 사용하기 때문에 빈 오브젝트를 만들고 이름을 Portal로 설정한다.

(2) 포탈의 가장자리 이펙트를 위해 파티클 오브젝트를 하나 생성하고 이름을 PortalBorder로 바꾼 뒤 Portal오브젝트의 자식으로 넣어준다.

(3) PortalBorder오브젝트의 위치를 리셋하고 회전 : -90, 0, 0 크기 : 3, 5, 1로 설정한다.

(4) 머테리얼을 새로 하나 생성하고 이름을 Portal로 바꾼 뒤에 셰이더를 Legacy에 있는 Particles로 설정해주고 텍스처에 미리 준비해둔 포탈 텍스처를 등록해준다.

(5) 포탈 이펙트 제작 방법 : Renderer탭 - Render Mode:Mesh - Mesh:Quad - Material:Portal - Render Alignment:World - Order in Layer:1 - Shape 체크 해제 -ParticleBorder탭- 색상 바꾸기 - StartSpeed:0 - Simulation Speed:10 이렇게 설정하면 다음과 같은 모습이 된다.

![image](https://user-images.githubusercontent.com/101051124/159213549-8ae73b0f-3f1e-464a-bdac-2c866400b330.png)

(6) 포탈 내부 이펙트 제작을 위해 PortalBorder를 복제하고 이름을 PortalInside로 바꾼다.

(7) 포탈 내부 이펙트 만드는 방법 : PortalInside탭 - StartSize를 랜덤으로 0.1~1사리 값으로 변경 - Emission탭 - Rate Over Time:5 이렇게 설정하면 다음과 같은 모습이 된다.

![image](https://user-images.githubusercontent.com/101051124/159213857-5435f80a-19de-42fd-b1d2-01611aac3333.png)

(8) 플레이어가 포탈에 충돌했을 때 플레이어의 위치를 옮기기 위해 Teleporter스크립트를 만들어 준다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Teleporter : MonoBehaviour
{
    [SerializeField]
    private Transform arrivePoint;
    private AudioSource audio;

    private void Awake()
    {
        audio = GetComponent<AudioSource>();    
    }

    private void OnTriggerEnter(Collider other)
    {
        if (other.gameObject.tag == "Player")
        {
            audio.Play();

            //other(플레이어)의 위치를 arrivePoint로 설정
            other.transform.position = arrivePoint.position;
        }
    }
}
```

(9) Portal오브젝트에 충돌처리를 위한 박스 콜라이더 컴포넌트,사운드 재생을 위한 AudioSource 컴포넌트, 방금 작성한 스크립트를 컴포넌트로 추가 해준다.

(10) 박스 콜라이더의 크기를 2.2, 3.6, 0.1로 바꾸고 is Trigger를 체크하고 오디오 소스의 클립은 9번 사운드로 넣어주고 Play On Awake를 해제한다.

(11) 포탈은 순간이동 포탈과 도착지점을 하나로 묶어 관리하기 위해 빈 오브젝트를 생성하고 이름을 Teleporter로 바꾸고 위치를 리셋한 다음 Portal오브젝트를 Teleporter의 자식으로 넣어준다.

(12) 도착지점으로 사용 할 빈 오브젝트를 하나 생성하고 이름을 ArrivePoint로 바꾸고 TelePorter의 자식으로 넣어준다.

(13) Portal의 위치 : -7.5, 2, 7.5 / ArrivePoint의 위치 : 7.5, 3, 0으로 설정

(14) Portal오브젝트의 Teleporter스크립트의 필드인 ArrivePoint에 우리가 만든 ArrivePoint오브젝트를 할당해준다.

이제 게임을 실행하여 플레이어가 포탈에 닿는 순간 설정한 위치로 순간이동하는 걸 볼 수 있다.

![image](https://user-images.githubusercontent.com/101051124/159215006-682b0372-3959-4d92-b5f9-309764cf7b34.png)

# 코인 관리 및 획득 처리

(1) Coin프리펩에 tag를 Coin으로 설정한다.

(2) 맵에 배치된 코인의 갯수, 스테이지 관리를 위해 빈 오브젝트를 생성하고 이름을 StageController로 바꾼다.

(3) StageController 스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class StageController : MonoBehaviour
{
    private int maxCoinCount;
    private int currentCoinCount;

    private void Awake()
    {
        // 태그가 "Coin"인 오브젝트의 개수를 maxCoinCount에 저장 (맵에 배치된 코인의 갯수)
        maxCoinCount = GameObject.FindGameObjectsWithTag("Coin").Length;
        currentCoinCount = maxCoinCount;
    }
}
```

(4) 게임화면에 남은 코인의 개수를 표시하기 위해 UI - Text-TectMeshPro를 선택하고 Import TMP Essentials를 눌러 사용가능하게 만들어준다.

(5) 캔버스 오브젝트의 캔버스 컴포넌트의 UI Scale Mode를 Scale With Screen Size로 설정하고 Reference Resolution을 1600,1000으로 설정하고 Match를 0.5로 설정해준다.

(6) 캔버스 오브제트의 자식으로 있는 텍스트 오브젝트의 이름을 TextCoinCount로 바꾸고 앵커를 이용해 위치를 조정한다.

(7) TextCoinCount의 설정을 알맞게 변경한다.

(8) StageController 스크립트를 수정한다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class StageController : MonoBehaviour
{
    private int maxCoinCount;
    private int currentCoinCount;

    //외부에서 maxCoinCount와 currentCoinCount를 수정 가능하도록 프로퍼티를 만들어 준다.
    public int MaxCoinCount => maxCoinCount;
    public int CurrentCoinCount => currentCoinCount;

    private void Awake()
    {
        // 태그가 "Coin"인 오브젝트의 개수를 maxCoinCount에 저장 (맵에 배치된 코인의 갯수)
        maxCoinCount = GameObject.FindGameObjectsWithTag("Coin").Length;
        currentCoinCount = maxCoinCount;
    }
}
```

(9) 스테이지 컨트롤러에 있는 코인 정보를 텍스트코인UI에 보여주기 위해 CoinCountViewer스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using TMPro;        //TextMeshProUGUI클래스를 사용하기 위해
public class CoinCountViewer : MonoBehaviour
{
    [SerializeField]
    private StageController stageController;
    private TextMeshProUGUI textCoinCount;

    private void Awake()
    {
        textCoinCount = GetComponent<TextMeshProUGUI>();
    }

    private void Update()
    {
        textCoinCount.text = "Coins : " + stageController.CurrentCoinCount + "/" + stageController.MaxCoinCount;
    }
}
```

(10) StageController 오브젝트에 StageController스크립트를 적용하고, TextCoinCount오브젝트에 CoinCountViewer를 적용해주고 CoinCountViewer의 필드인 StageController변수에 StageController  오브젝트를 할당해준다.

(11) StageController 스크립트를 코인 획득시 남은 코인의 개수를 변경하는 GetCoin함수를 만들어 준다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class StageController : MonoBehaviour
{
    private int maxCoinCount;
    private int currentCoinCount;

    //외부에서 maxCoinCount와 currentCoinCount를 수정 가능하도록 프로퍼티를 만들어 준다.
    public int MaxCoinCount => maxCoinCount;
    public int CurrentCoinCount => currentCoinCount;

    private void Awake()
    {
        // 태그가 "Coin"인 오브젝트의 개수를 maxCoinCount에 저장 (맵에 배치된 코인의 갯수)
        maxCoinCount = GameObject.FindGameObjectsWithTag("Coin").Length;
        currentCoinCount = maxCoinCount;
    }

    public void GetCoin()
    {
        currentCoinCount--;

        if (currentCoinCount == 0)
        {
            //스테이지 클리어 처리
        }
    }
}
```

(12) 이에 맞게 CoinController스크립트를 수정해준다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CoinController : MonoBehaviour
{
    [SerializeField]
    private StageController stageController;
    [SerializeField]
    private GameObject coinEffectPrefab;
    private float rotateSpeed = 100.0f;

    private void Update()
    {
        // 코인 오브젝트 회전
        transform.Rotate(Vector3.right*rotateSpeed*Time.deltaTime);
    }

    private void OnTriggerEnter(Collider other)
    {
        // 코인 오브젝트 획득 효과 생성
        GameObject clone = Instantiate(coinEffectPrefab);
        clone.transform.position = transform.position;

        //코인 획득 처리
        stageController.GetCoin();

        Destroy(gameObject);
    }
}
```

(13)  맵에 배치된 코인 오브젝트들을 선택하고 CoinController컴포넌트에 StageController를 할당해주고 게임을 실행하면 현재 남은 코인의 개수와 최대 코인의 개수가 나오는걸 볼 수 있다.

![image](https://user-images.githubusercontent.com/101051124/159218120-a8ba2d2f-819b-45ee-b9ef-4403e89cb4b2.png)

# 스테이지 클리어

모든 코인을 획득 하였을 때 스테이지 클리어를 알리는 UI를 등장시킬 것이다.

(1) UI - Panel오브젝트를 생성하고 이름을 StageClear로 바꾼다.

(2) 패널의 여백을 Left:400 Right:400 Top:300 Bottom:300으로 설정해준다.

(3) 텍스트 출력을 위해 텍스트 메시 프로를 하나 생성하고 StageClear의 자식으로 넣어준다.

(4) 텍스트 메시 프로를 다음과 같이 설정한다.

![image](https://user-images.githubusercontent.com/101051124/159218831-be0255e0-e2e1-481a-9135-6a3ab76855b5.png)

그럼 다음과 같은 화면이 된다.

![image](https://user-images.githubusercontent.com/101051124/159218862-75c7d458-2b7d-468e-aa0d-17335fa28d7f.png)

(5) 모든 코인을 획득했을 때 UI가 활성화되게 하고 엔터키를 눌렀을 때 다음 씬을 불러오기 위해 StageController스크립트를 수정한다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

public class StageController : MonoBehaviour
{
    [SerializeField]
    private string nextSceneName;       //다음 씬 이름
    [SerializeField]
    private GameObject panelStageClear;     //스테이지 클리어 시 나타나는 Panel UI
    private int maxCoinCount;
    private int currentCoinCount;
    private bool getAllCoins=false;     //모든 코인 획득 시 true

    //외부에서 maxCoinCount와 currentCoinCount를 수정 가능하도록 프로퍼티를 만들어 준다.
    public int MaxCoinCount => maxCoinCount;
    public int CurrentCoinCount => currentCoinCount;

    private void Awake()
    {
        //시간배율을 1로 설정하여 정상속도로 재생
        Time.timeScale = 1.0f;
        //모든 코인을 획득했을 때 등장 (panelStageClear 오브젝트 비활성화)
        panelStageClear.SetActive(false);

        // 태그가 "Coin"인 오브젝트의 개수를 maxCoinCount에 저장 (맵에 배치된 코인의 갯수)
        maxCoinCount = GameObject.FindGameObjectsWithTag("Coin").Length;
        currentCoinCount = maxCoinCount;
    }
    private void Update()
    {
        //모든 코인을 획득했을 때
        if (getAllCoins == true)
        {
            //엔터키를 누르면
            if (Input.GetKeyDown(KeyCode.Return))
            {
                //다음 씬으로 이동
                SceneManager.LoadScene(nextSceneName);
            }
        }
    }
    public void GetCoin()
    {
        currentCoinCount--;

        if (currentCoinCount == 0)
        {
            //getAllCoin true
            getAllCoins = true;
            //시간 배율을 0으로 설정해 일시정지
            Time.timeScale = 0.0f;
            //panelStageClear 오브젝트 활성화
            panelStageClear.SetActive (true);
        }
    }
}
```

(6) StageController오브젝트에 StageController컴포넌트에 Next Scene Name에 Stage01을 입력하고 Panel Stage Clear에 StageClear오브젝트를 넣어준다.

(7) 새로운 씬을 만들고 이름을 Stage01로 지정해준다.

(8) 빌드 셋팅에 들어가 현재 만든 씬들을 등록해준다.(그래야 LoadScene()가능)

이제 게임을 실행하고 코인을 다 먹으면 스테이지 클리어 UI가 보이고 엔터키를 누르면 다음 스테이지로 넘어가는걸 볼 수 있다.

# 플레이어 사망

플레이어가 맵에 아래에 떨어졌을 때 사망해서 재시작 하도록 PlayerController스크립트를 수정한다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

public class PlayerController : MonoBehaviour
{
    [SerializeField]
    private float minHeight;
    private Movement3D movement;

    private void Awake()
    {
        movement = GetComponent<Movement3D>();
    }

    private void Update()
    {
        float hAxis = Input.GetAxisRaw("Horizontal");
        float vAxis = Input.GetAxisRaw("Vertical");

        if(hAxis!= 0 || vAxis != 0)
        {
            movement.MoveTo(new Vector3(hAxis, 0, vAxis));
        }

        //낭떠러지로 플레이어가 떨어지는지 체크
        ChangeSceneFallDown();
    }
    private void ChangeSceneFallDown()
    {
        if(transform.position.y < minHeight)
        {
            //현재 씬 로드 (현재 씬 재시작)
            SceneManager.LoadScene(SceneManager.GetActiveScene().name);
        }
    }
}
```

에디터로 돌아가서 Player오브젝트에 PlayerController컴포넌트의 MinHeight를 -1로 설정한다.

이제 게임을 실행하면 플레이어가 낭떠러지로 떨어지면 게임을 재시작하는 걸 볼 수 있다.