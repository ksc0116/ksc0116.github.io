---
layout: post
title:  "[Project] Jumping Ball"
subtitle:   ""
categories: Unity
comments: true
---

<br>

# 프로젝트 기본 설정

(1) MainCamera 위치:0 /10/ -10 회전:30/ 0/ 0 Camera컴포넌트 Clear Flags:Solid Color 색상:검은색

(2) 화면 해상도를 10:16으로 설정한다.

# 발판 오브젝트

(1) 플레이어가 밟는 발판은 실제로 밟는 발판과 코인을 한 세트로 관리하기 위해 빈 오브젝트를 만들고 이름을 Platform으로 바꾼다.

(2) 실제로 밟는 발판을 만들기 위해 Cube오브젝트를 만들어서 이름을 Platform으로 바꾸고 아까 만든 Platform의 자식으로 넣어준다. 그리고 발판의 머테리얼을 만들어서 입혀준다. 크기도 2/ 0.1/ 2로 바꾼다.

(3) 코인에게 입힐 머테리얼을 생성하고 이름을 Coin으로 바꾼 다음 적절히 설정해준다.

(4) 빈 오브젝트의 Platform의 자식으로 코인으로 사용 할 실린더 오브젝트를 만들고 이름을 Coin으로 바꾼 다음 위치:0/ 0.55/ 0 회전:0/ 0/ 90 크기:1/ 0.05/ 1로 설정하고 Coin머테리얼을 입힌다.

(5) Coin오브젝트를 비활성화하고 Platform오브젝트를 프리펩으로 만든다.

(6) 프리펩으로 만든 발판을 생성하는 PlatformSpawner스크립트를 만든다.

```sharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlatfornSpawner : MonoBehaviour
{
    [SerializeField]
    private GameObject platformPrefab;
    [SerializeField]
    private int spawnPlatformCountAtStart = 8;  // 게임 시작 시 최초 생성되는 발판 개수
    [SerializeField]
    private float xRange = 4;       // 발판이 배치될 수 있는 x 위치 범위(+-xRange)
    [SerializeField]
    private float zDistance = 5;    // 발판 사이의 거리 (z)
    private int platformIndex = 0;

    private void Awake()
    {
        // spawnPlatformAtStart에 저장된 개수만큼 최초 플랫폼 생성
        for (int i = 0; i < spawnPlatformCountAtStart; i++)
        {
            SpawnPlatform();
        }
    }

    public void SpawnPlatform()
    {
        // 새로운 발판 생성
        GameObject clone = Instantiate(platformPrefab);
        // 발판 위치 설정
        ResetPlatform(clone.transform);
    }

    public void ResetPlatform(Transform transform,float y = 0)
    {
        platformIndex++;

        // 발판이 배치되는 x 위치를 임의로 설정(-Range ~ Range)
        float x=Random.Range(-xRange,xRange);
        // 발판이 배치되는 위치 설정 (z축은 현재 발판 인덱스*zDistance)
        transform.position = new Vector3(x, y, platformIndex * zDistance);
    }
}
```

여기서 핵심은 발판끼리의 거리를 구하는데 platformIndex &#42; zDistance를 했다는 것이다.

(7) 빈 오브젝트를 만들고 이름을 PlatformSpawner로 바꾼다음 PlatformSpawner스크립트를 할당한다.

이제 게임을 실행하면 최초에 0 0 0에 위치한 플랫폼을 포함해 총 9개의 발판이 생성되는 걸 볼 수 있다.

![image](https://user-images.githubusercontent.com/101051124/159597277-8efe29f8-7226-4998-bab7-44c52c8ed9b4.png)

# 화면에서 사라진 발판 설정

(1) 발판 오브젝트를 제어하는 Platform 스크립트를 만든다.

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Platform : MonoBehaviour
{
    private PlatformSpawner platformSpawner;
    private Camera mainCamera;
    private float yMoveTime = 0.5f;

    public void SetUp(PlatformSpawner platformSpawner)
    {
        this.platformSpawner = platformSpawner;
        mainCamera = Camera.main;
    }

    private void Update()
    {
        // 플랫폼이 카메라 뒤로 가서 보이지 않게 되면 플랫폼 재배치
        if (mainCamera.transform.position.z - transform.position.z > 0)
        {
            // 플랫폼 위치 재설정
            platformSpawner.ResetPlatform(this.transform);
            // 새로 등할 때는 위에서 떨어지는 효과 적용
            StartCoroutine(MoveY(10, 0));
        }
    }
    private IEnumerator MoveY(float start,float end)
    {
        float current = 0;
        float percent = 0;
        while (percent < 1)
        {
            current += Time.deltaTime;
            percent = current / yMoveTime;

            // 시간 경과 (최대 1)에 따라 오브젝트릐 y 위치를 바꿔준다
            float y = Mathf.Lerp(start, end, percent);
            transform.position = new Vector3(transform.position.x, y, transform.position.z);
            yield return null;
        }
    }
}
```

(2) PlatformSpawner스크립트에서 발판이 생성되자마자 Setup()메소드를 호출 할 수 있게 스크립트를 수정한다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlatformSpawner : MonoBehaviour
{
    [SerializeField]
    private GameObject platformPrefab;
    [SerializeField]
    private int spawnPlatformCountAtStart = 8;  // 게임 시작 시 최초 생성되는 발판 개수
    [SerializeField]
    private float xRange = 4;       // 발판이 배치될 수 있는 x 위치 범위(+-xRange)
    [SerializeField]
    private float zDistance = 5;    // 발판 사이의 거리 (z)
    private int platformIndex = 0;

    private void Awake()
    {
        // spawnPlatformAtStart에 저장된 개수만큼 최초 플랫폼 생성
        for (int i = 0; i < spawnPlatformCountAtStart; i++)
        {
            SpawnPlatform();
        }
    }

    public void SpawnPlatform()
    {
        // 새로운 발판 생성
        GameObject clone = Instantiate(platformPrefab);
        // 발판 Setup() 메소드 호출
        clone.GetComponent<Platform>().SetUp(this);
        // 발판 위치 설정
        ResetPlatform(clone.transform);
    }

    public void ResetPlatform(Transform transform, float y = 0)
    {
        platformIndex++;

        // 발판이 배치되는 x 위치를 임의로 설정(-Range ~ Range)
        float x = Random.Range(-xRange, xRange);
        // 발판이 배치되는 위치 설정 (z축은 현재 발판 인덱스*zDistance)
        transform.position = new Vector3(x, y, platformIndex * zDistance);
    }
}
```

(3) 플랫폼 프리펩에 Platform스크립트를 컴포넌트로 추가해주고 하이어라키창에 있는 Platform오브젝트의 Platform컴포넌트를 삭제해준다.

(4) 게임을 실행하고 카메라의 z위치를 천천히 키워보면 화면 밖으로 사라졌던 발판들은 위치가 재설정 된다음 위에서 떨어지게 된다.

# 플레이어 오브젝트

플레이어 오브젝트는 보여지는 플레이어 오브젝트와 현재 위치를 보여주기 위해 바닥에 표시되는 그림자 오브젝트를 한 세트로 관리한다.

(1) 빈 오브젝트를 만들고 이름을 Player로 설정한다. 위치:0/ 0.55/ 0로 설정한다. 충돌 처리를 위해 스피어 콜라이더와 리지드바디를 컴포넌트로 추가해준다. 스피어 콜라이더는 is Trigger를 체크하고 리지드바디는 Use Gravity를 비활성화한다.

(2) 실제로 보이게 될 스피어 오브젝트를 Player오브젝트의 자식으로 배치하고 이름을 Player로 바꾼 다음 그냥 보여지는 용도이기 때문에 콜라이더는 삭제해준다.

(3) Player오브젝트의 그림자를 표현을 위해 머테리얼을 만들고 이름을 PlayerShadow로 설정한다.

(4) PlayerShadow머테리얼의 세부 설정은 Shader-> particles -> Standard Unit으로 바꾸고,Rendering Mode:Fade로 Albedo:Defailt-particle로 설정한다.

(5) 플레이어의 그림자로 사용 할 Plane오브젝트를 생성하고 이름을 PlayerShadow로 바꾸고 Player오브젝트의 자식으로 배치한다. 트랜스폼은 위치:0/ -0.45/ 0 크기: 0.2/ 0.2/ 0.2로 해준다. 그리고 만들어둔  PlayerShadow머테리얼을 입혀준다. 그럼 다음과 같은 상태가 된다.

![image](https://user-images.githubusercontent.com/101051124/159603718-71ea3e00-b25a-42e2-acda-13967cf99303.png)

(6) 그림자도 보여지는 용도이기 때문에 콜라이더를 삭제한다.

(7) 플레이어를 제어하는 PlayerController스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    [SerializeField]
    private float xSensitivity = 10.0f;

    private void Update()
    {
        // 마우스 왼쪽 버튼 (or 모바일 터치)으로 플레이어의 x 위치 제어
        if (Input.GetMouseButton(0))
        {
            MoveToX();
        }
    }
    private void MoveToX()
    {
        float x = 0.0f;
        Vector3 position = transform.position;

        // 현재 플랫폼이 모바일 일 때
        if (Application.isMobilePlatform)
        {
            if (Input.touchCount > 0)
            {
                Touch touch = Input.GetTouch(0);
                // 0.0f ~ 1.0f의 값으로 만들고 -0.5f를 하기 때문에
                // -0.5f ~ 0.5f 사이의 값이 나온다.
                x=(touch.position.x / Screen.width)-0.5f;
            }
        }
        // 현재 플랫폼이 PC 일 때
        else
        {
            // 참고) 화면의 왼쪽 아래쪽이 0.0임
            // 0.0f~1.0f의 값으로 만들고 -0.5f를 하기 때문에 
            // -0.5f~0.5f 사이의 값이 나온다.
            x=(Input.mousePosition.x / Screen.width)-0.5f;
        }

        // 화면 밖을 터치해 x 값이 -0.5~0.5의 범위를 넘어가지 않도록
        x= Mathf.Clamp(x,-0.5f,0.5f);
        // 플레이어의 x 위치 설정
        position.x=Mathf.Lerp(position.x,x*xSensitivity, xSensitivity * Time.deltaTime);

        transform.position=position;
    }
}
```

이제 게임을 실행하고 마우스 좌클릭을 한 상태로 마우스를 움직이면 플레이어 오브젝트도 따라 움직인다.

(8) PlatformSpawner스크립트를 외부에서 발판사이의 거리의 정보를 Get할 수 있게 바꿔준다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlatformSpawner : MonoBehaviour
{
    [SerializeField]
    private GameObject platformPrefab;
    [SerializeField]
    private int spawnPlatformCountAtStart = 8;  // 게임 시작 시 최초 생성되는 발판 개수
    [SerializeField]
    private float xRange = 4;       // 발판이 배치될 수 있는 x 위치 범위(+-xRange)
    [SerializeField]
    private float zDistance = 5;    // 발판 사이의 거리 (z)
    private int platformIndex = 0;

    public float ZDistance
    {
        get { return zDistance; }
    }
    private void Awake()
    {
        // spawnPlatformAtStart에 저장된 개수만큼 최초 플랫폼 생성
        for (int i = 0; i < spawnPlatformCountAtStart; i++)
        {
            SpawnPlatform();
        }
    }

    public void SpawnPlatform()
    {
        // 새로운 발판 생성
        GameObject clone = Instantiate(platformPrefab);
        // 발판 Setup() 메소드 호출
        clone.GetComponent<Platform>().SetUp(this);
        // 발판 위치 설정
        ResetPlatform(clone.transform);
    }

    public void ResetPlatform(Transform transform, float y = 0)
    {
        platformIndex++;

        // 발판이 배치되는 x 위치를 임의로 설정(-Range ~ Range)
        float x = Random.Range(-xRange, xRange);
        // 발판이 배치되는 위치 설정 (z축은 현재 발판 인덱스*zDistance)
        transform.position = new Vector3(x, y, platformIndex * zDistance);
    }
}
```

(9) x,z축 이동은 Player부모오브젝트가 하고 y축 이동은 보여지는 Player오브젝트만 하게 PlayerController를 수정한다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    [SerializeField]
    private PlatformSpawner platformSpawner;

    [SerializeField]
    private Transform playerObject;             // 보여지는 Sphere 오브젝트 (y 이동)
    [SerializeField]
    private float xSensitivity = 10.0f;
    [SerializeField]
    private float moveTime = 1.0f;      // y,z이동 시간
    [SerializeField]
    private float minPositionY = 0.55f;     // y축 이동을 위해 플레이어의 최소 y 위치 설정
    private float gravity = -9.81f;         //중력
    private int platformIndex = 0;          // 플레이어가 다음으로 이동할 플랫폼 인덱스

    private IEnumerator Start()
    {
        // 마우스 왼쪽 클릭을 누르기 전까지 시작하지 않고 대기
        while (true)
        {
            if (Input.GetMouseButtonDown(0))
            {
                // 플레이어의 y, z이동 제어
                StartCoroutine("MoveLoop");
                break;
            }
            yield return null;
        }
    }
    private IEnumerator MoveLoop()
    {
        while (true)
        {
            platformIndex++;

            float current = (platformIndex - 1) * platformSpawner.ZDistance;
            float next = platformIndex * platformSpawner.ZDistance;

            // 플레이어의 y, z위치 제어
            yield return StartCoroutine(MoveToZ(current, next));
        }
    }
    private void Update()
    {
        // 마우스 왼쪽 버튼 (or 모바일 터치)으로 플레이어의 x 위치 제어
        if (Input.GetMouseButton(0))
        {
            MoveToX();
        }
    }
    private void MoveToX()
    {
        float x = 0.0f;
        Vector3 position = transform.position;

        // 현재 플랫폼이 모바일 일 때
        if (Application.isMobilePlatform)
        {
            if (Input.touchCount > 0)
            {
                Touch touch = Input.GetTouch(0);
                // 0.0f ~ 1.0f의 값으로 만들고 -0.5f를 하기 때문에
                // -0.5f ~ 0.5f 사이의 값이 나온다.
                x=(touch.position.x / Screen.width)-0.5f;
            }
        }
        // 현재 플랫폼이 PC 일 때
        else
        {
            // 참고 화면의 왼쪽 아래쪽이 0.0임
            // 0.0f~1.0f의 값으로 만들고 -0.5f를 하기 때문에 
            // -0.5f~0.5f 사이의 값이 나온다.
            x=(Input.mousePosition.x / Screen.width)-0.5f;
        }

        // 화면 밖을 터치해 x 값이 -0.5~0.5의 범위를 넘어가지 않도록
        x= Mathf.Clamp(x,-0.5f,0.5f);
        // 플레이어의 x 위치 설정
        position.x=Mathf.Lerp(position.x,x*xSensitivity, xSensitivity * Time.deltaTime);

        transform.position=position;
    }
    private IEnumerator MoveToZ(float start,float end)
    {
        float current = 0;
        float percent = 0;
        float v0 = -gravity;        // y 방향의 초기속도

        while(percent < 1)
        {
            current += Time.deltaTime;
            percent = current / moveTime;

            // 시간 경과에 따라 오브젝트의 y 위치를 바꿔준다.
            // 포물선 운동 : 시작 위치 + 초기속도*시간 + 중력*시간제곱
            float y=minPositionY + (v0 * percent)+ (gravity * percent*percent);
            playerObject.position=new Vector3(playerObject.position.x, y, playerObject.position.z);

            // 시간 경과(최대 1)에 따라 오브젝트의 z위치를 바꿔준다
            float z = Mathf.Lerp(start, end, percent);
            transform.position=new Vector3(transform.position.x,transform.position.y,z);

            yield return null;
        }
    }
}
```

에디터에서 PlayerController스크립트의 변수에 할당할 것을 할당해주고 게임을 실행하고 마우스 좌클릭을 하면 플레이어 오브젝트가 움직인다.

# 카메라 오브젝트

(1) 플레이어를 추적하는 카메라 이동을 위해 CameraController스크립트를 만들어준다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CameraController : MonoBehaviour
{
    [SerializeField]
    private Transform target;
    private float distance;
    private void Awake()
    {
        distance = transform.position.z;
    }
    private void LateUpdate()
    {
        if (target == null) return;
        
        Vector3 position=transform.position;
        position.z=target.position.z+ distance;
        transform.position=position;
    }
}
```

(2) 메인 카메라 오브젝트에 넣어주고 변수를 할당해준다.

이제 게임을 실행하면 카메라가 플레이어를 추적하는 걸 확인할 수 있다.

# 발판 충돌 처리

플레이어가 발판을 밟았는지 검사한다. PlayerController 스크립트를 수정한다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    [SerializeField]
    private PlatformSpawner platformSpawner;

    [SerializeField]
    private Transform playerObject;             // 보여지는 Sphere 오브젝트 (y 이동)
    [SerializeField]
    private float xSensitivity = 10.0f;
    [SerializeField]
    private float moveTime = 1.0f;      // y,z이동 시간
    [SerializeField]
    private float minPositionY = 0.55f;     // y축 이동을 위해 플레이어의 최소 y 위치 설정
    private float gravity = -9.81f;         //중력
    private int platformIndex = 0;          // 플레이어가 다음으로 이동할 플랫폼 인덱스

    private RaycastHit hit;     // 광선에 부딪힌 오브젝트 정보 저장

    private IEnumerator Start()
    {
        // 마우스 왼쪽 클릭을 누르기 전까지 시작하지 않고 대기
        while (true)
        {
            if (Input.GetMouseButtonDown(0))
            {
                // 플레이어의 y, z이동 제어
                StartCoroutine("MoveLoop");
                break;
            }
            yield return null;
        }
    }
    private IEnumerator MoveLoop()
    {
        while (true)
        {
            platformIndex++;

            float current = (platformIndex - 1) * platformSpawner.ZDistance;
            float next = platformIndex * platformSpawner.ZDistance;

            // 플레이어의 y, z위치 제어
            yield return StartCoroutine(MoveToYZ(current, next));


            // 플레이어가 다음 플랫폼에 도착했을 때
            // 플레이어의 도착 위치가 플랫폼이면
            if (hit.transform != null)
            {
                Debug.Log("Hit");
            }
            // 플레이어의 위치가 낭떠러지이면
            else
            {
                Debug.Log("GameOver");
                break;
            }
        }
    }
    private void Update()
    {
        // 아래쪽 방향으로 광선을 발사해 관선에 부딪히는 발판 정보를 hit에 저장
        Physics.Raycast(transform.position,Vector3.down, out hit, 1.0f);

        // 마우스 왼쪽 버튼 (or 모바일 터치)으로 플레이어의 x 위치 제어
        if (Input.GetMouseButton(0))
        {
            MoveToX();
        }
    }
    private void MoveToX()
    {
        float x = 0.0f;
        Vector3 position = transform.position;

        // 현재 플랫폼이 모바일 일 때
        if (Application.isMobilePlatform)
        {
            if (Input.touchCount > 0)
            {
                Touch touch = Input.GetTouch(0);
                // 0.0f ~ 1.0f의 값으로 만들고 -0.5f를 하기 때문에
                // -0.5f ~ 0.5f 사이의 값이 나온다.
                x=(touch.position.x / Screen.width)-0.5f;
            }
        }
        // 현재 플랫폼이 PC 일 때
        else
        {
            // 참고 화면의 왼쪽 아래쪽이 0.0임
            // 0.0f~1.0f의 값으로 만들고 -0.5f를 하기 때문에 
            // -0.5f~0.5f 사이의 값이 나온다.
            x=(Input.mousePosition.x / Screen.width)-0.5f;
        }

        // 화면 밖을 터치해 x 값이 -0.5~0.5의 범위를 넘어가지 않도록
        x= Mathf.Clamp(x,-0.5f,0.5f);
        // 플레이어의 x 위치 설정
        position.x=Mathf.Lerp(position.x,x*xSensitivity, xSensitivity * Time.deltaTime);

        transform.position=position;
    }
    private IEnumerator MoveToYZ(float start,float end)
    {
        float current = 0;
        float percent = 0;
        float v0 = -gravity;        // y 방향의 초기속도

        while(percent < 1)
        {
            current += Time.deltaTime;
            percent = current / moveTime;

            // 시간 경과에 따라 오브젝트의 y 위치를 바꿔준다.
            // 포물선 운동 : 시작 위치 + 초기속도*시간 + 중력*시간제곱
            float y=minPositionY + (v0 * percent)+ (gravity * percent*percent);
            playerObject.position=new Vector3(playerObject.position.x, y, playerObject.position.z);

            // 시간 경과(최대 1)에 따라 오브젝트의 z위치를 바꿔준다
            float z = Mathf.Lerp(start, end, percent);
            transform.position=new Vector3(transform.position.x,transform.position.y,z);

            yield return null;
        }
    }
}
```

계속해서 플레이어의 아래쪽 방향으로 광선을 쏘고 플레이어가 MoveToYZ()를 끝냈을 때 광선에 맞은 놈이 있으면 발판을 밟은 것이고, 아니면 낭떠러지에 떨어진 것이므로 게임오버이다.

# 현재 획득 점수 출력

(1) TapToPlay텍스트 출력을 위해 TextMeshPro를 하나 만들고 이름을 TextTapToPlay로 바꾼다. 그리고 알맞게 설정해준다.

(2)  현재 획득 점수를 출력하기 위해 TextMeshPro를 만들고 이름을 TextScore로 바꾼 다음 알맞게 설정해준다.

![image](https://user-images.githubusercontent.com/101051124/159614816-ef36e11c-a532-4573-be2e-ecdb9a94467e.png)

(3) 점수 출력과 게임 진행을 관리하는 GameController스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;
using TMPro;
public class GameController : MonoBehaviour
{
    [SerializeField]
    private TextMeshProUGUI textTapToPlay;

    [SerializeField]
    private TextMeshProUGUI textScore;
    private int score = 0;

    private void Awake()
    {
        // 마지막 플레이에서 획득했던 점수 불러오가
        int score = PlayerPrefs.GetInt("LastScore");
        textScore.text = score.ToString();
    }

    public void GameStart()
    {
        // Tap To Play 텍스트를 보이지 않게함
        textTapToPlay.enabled = false;

        // 게임 시작 전에는 마지막 획득 점수가 출력되기 때문에 게임 시작 시 점수 텍스트 갱신
        textScore.text=score.ToString();
    }

    public void IncreaseScore()
    {
        score++;
        textScore.text=score.ToString();
    }

    public void GameOver()
    {
        // 마지막에 획득한 점수 저장 (씬을 다시 로드 했을 때 마지막 점수 출력)
        PlayerPrefs.SetInt("LastScore", score);

        // 현재 씬을 다시 로드
        SceneManager.LoadScene(SceneManager.GetActiveScene().name);
    }

    private void OnApplicationQuit()
    {
        // 프로그램을 종료할 때 LastScore를 0으로 설정
        PlayerPrefs.SetInt("LastScore", 0);
    }
}
```

(4) PlayerController스크립트를 수정한다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    [SerializeField]
    private PlatformSpawner platformSpawner;
    [SerializeField]
    private GameController gameController;

    [SerializeField]
    private Transform playerObject;             // 보여지는 Sphere 오브젝트 (y 이동)
    [SerializeField]
    private float xSensitivity = 10.0f;
    [SerializeField]
    private float moveTime = 1.0f;      // y,z이동 시간
    [SerializeField]
    private float minPositionY = 0.55f;     // y축 이동을 위해 플레이어의 최소 y 위치 설정
    private float gravity = -9.81f;         //중력
    private int platformIndex = 0;          // 플레이어가 다음으로 이동할 플랫폼 인덱스

    private RaycastHit hit;     // 광선에 부딪힌 오브젝트 정보 저장

    private IEnumerator Start()
    {
        // 마우스 왼쪽 클릭을 누르기 전까지 시작하지 않고 대기
        while (true)
        {
            if (Input.GetMouseButtonDown(0))
            {
                // 게임 시작
                gameController.GameStart();
                // 플레이어의 y, z이동 제어
                StartCoroutine("MoveLoop");
                break;
            }
            yield return null;
        }
    }
    private IEnumerator MoveLoop()
    {
        while (true)
        {
            platformIndex++;

            float current = (platformIndex - 1) * platformSpawner.ZDistance;
            float next = platformIndex * platformSpawner.ZDistance;

            // 플레이어의 y, z위치 제어
            yield return StartCoroutine(MoveToYZ(current, next));


            // 플레이어가 다음 플랫폼에 도착했을 때
            // 플레이어의 도착 위치가 플랫폼이면
            if (hit.transform != null)
            {
                //Debug.Log("Hit");
                gameController.IncreaseScore();
            }
            // 플레이어의 위치가 낭떠러지이면
            else
            {
                // Debug.Log("GameOver");
                gameController.GameOver();
                break;
            }
        }
    }
    private void Update()
    {
        // 아래쪽 방향으로 광선을 발사해 관선에 부딪히는 발판 정보를 hit에 저장
        Physics.Raycast(transform.position,Vector3.down, out hit, 1.0f);

        // 마우스 왼쪽 버튼 (or 모바일 터치)으로 플레이어의 x 위치 제어
        if (Input.GetMouseButton(0))
        {
            MoveToX();
        }
    }
    private void MoveToX()
    {
        float x = 0.0f;
        Vector3 position = transform.position;

        // 현재 플랫폼이 모바일 일 때
        if (Application.isMobilePlatform)
        {
            if (Input.touchCount > 0)
            {
                Touch touch = Input.GetTouch(0);
                // 0.0f ~ 1.0f의 값으로 만들고 -0.5f를 하기 때문에
                // -0.5f ~ 0.5f 사이의 값이 나온다.
                x=(touch.position.x / Screen.width)-0.5f;
            }
        }
        // 현재 플랫폼이 PC 일 때
        else
        {
            // 참고 화면의 왼쪽 아래쪽이 0.0임
            // 0.0f~1.0f의 값으로 만들고 -0.5f를 하기 때문에 
            // -0.5f~0.5f 사이의 값이 나온다.
            x=(Input.mousePosition.x / Screen.width)-0.5f;
        }

        // 화면 밖을 터치해 x 값이 -0.5~0.5의 범위를 넘어가지 않도록
        x= Mathf.Clamp(x,-0.5f,0.5f);
        // 플레이어의 x 위치 설정
        position.x=Mathf.Lerp(position.x,x*xSensitivity, xSensitivity * Time.deltaTime);

        transform.position=position;
    }
    private IEnumerator MoveToYZ(float start,float end)
    {
        float current = 0;
        float percent = 0;
        float v0 = -gravity;        // y 방향의 초기속도

        while(percent < 1)
        {
            current += Time.deltaTime;
            percent = current / moveTime;

            // 시간 경과에 따라 오브젝트의 y 위치를 바꿔준다.
            // 포물선 운동 : 시작 위치 + 초기속도*시간 + 중력*시간제곱
            float y=minPositionY + (v0 * percent)+ (gravity * percent*percent);
            playerObject.position=new Vector3(playerObject.position.x, y, playerObject.position.z);

            // 시간 경과(최대 1)에 따라 오브젝트의 z위치를 바꿔준다
            float z = Mathf.Lerp(start, end, percent);
            transform.position=new Vector3(transform.position.x,transform.position.y,z);

            yield return null;
        }
    }
}
```

(5) GameController실행을 위해 빈 오브젝트 하나 만들고 이름을 GameController로 바꾼 다음 GameController스크립트를 추가해준다.

# 최고 점수 출력

(1) 최고 점수 출력을 위해 TextmeshPro를 생성하고 이름을 TextHighScore로 바꾼 다음 알맞게 설정해준다.

(2) GameControll 스크립트를 수정해준다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;
using TMPro;
public class GameController : MonoBehaviour
{
    [SerializeField]
    private TextMeshProUGUI textTapToPlay;

    [SerializeField]
    private TextMeshProUGUI textScore;
    private int score = 0;

    [SerializeField]
    private TextMeshProUGUI textHighScore;

    private void Awake()
    {
        // 마지막 플레이에서 획득했던 점수 불러오가
        int score = PlayerPrefs.GetInt("LastScore");
        textScore.text = score.ToString();

        // 기존에 등록된 최고 점수 불러오기
        int highScore = PlayerPrefs.GetInt("highScore");
        textHighScore.text = highScore.ToString();
    }

    public void GameStart()
    {
        // Tap To Play 텍스트를 보이지 않게함
        textTapToPlay.enabled = false;
        // 최고 점수 보이지 않게 설정
        textHighScore.enabled = false;

        // 게임 시작 전에는 마지막 획득 점수가 출력되기 때문에 게임 시작 시 점수 텍스트 갱신
        textScore.text=score.ToString();
    }

    public void IncreaseScore()
    {
        score++;
        textScore.text=score.ToString();
    }

    public void GameOver()
    {
        // 기존의 등록된 최고 점수 불러오기
        int highScore = PlayerPrefs.GetInt("HighScore");
        if(score> highScore)
        {
            PlayerPrefs.SetInt("HighScore", score);
        }
        // 마지막에 획득한 점수 저장 (씬을 다시 로드 했을 때 마지막 점수 출력)
        PlayerPrefs.SetInt("LastScore", score);

        // 현재 씬을 다시 로드
        SceneManager.LoadScene(SceneManager.GetActiveScene().name);
    }

    private void OnApplicationQuit()
    {
        // 프로그램을 종료할 때 LastScore를 0으로 설정
        PlayerPrefs.SetInt("LastScore", 0);
    }
}
```

# 플레이어 Y,Z  이동 시간 감소(점점 빠르게)

실제 발판의 생성거리나 플레이어의 이동거리는 같지만 Mathf.Lerp()로 플레이어가 이동하기 때문에 Lerp가 실행되는 시간을 감소 시켜 점점 이동속도가 빨라지는 것처럼 보이게 한다.

(1) PlayerController스크립트를 수정한다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    [SerializeField]
    private PlatformSpawner platformSpawner;
    [SerializeField]
    private GameController gameController;

    [SerializeField]
    private Transform playerObject;             // 보여지는 Sphere 오브젝트 (y 이동)
    [SerializeField]
    private float xSensitivity = 10.0f;
    [SerializeField]
    private float moveTime = 1.0f;      // y,z이동 시간
    [SerializeField]
    private float minPositionY = 0.55f;     // y축 이동을 위해 플레이어의 최소 y 위치 설정
    private float gravity = -9.81f;         //중력
    private int platformIndex = 0;          // 플레이어가 다음으로 이동할 플랫폼 인덱스

    private RaycastHit hit;     // 광선에 부딪힌 오브젝트 정보 저장

    private IEnumerator Start()
    {
        // 마우스 왼쪽 클릭을 누르기 전까지 시작하지 않고 대기
        while (true)
        {
            if (Input.GetMouseButtonDown(0))
            {
                // 게임 시작
                gameController.GameStart();
                // 플레이어의 y, z이동 제어
                StartCoroutine("MoveLoop");
                // 플레이어의 이동 시간 감소
                StartCoroutine("DecreaseMoveTime");
                break;
            }
            yield return null;
        }
    }
    private IEnumerator MoveLoop()
    {
        while (true)
        {
            platformIndex++;

            float current = (platformIndex - 1) * platformSpawner.ZDistance;
            float next = platformIndex * platformSpawner.ZDistance;

            // 플레이어의 y, z위치 제어
            yield return StartCoroutine(MoveToYZ(current, next));


            // 플레이어가 다음 플랫폼에 도착했을 때
            // 플레이어의 도착 위치가 플랫폼이면
            if (hit.transform != null)
            {
                //Debug.Log("Hit");
                gameController.IncreaseScore();
            }
            // 플레이어의 위치가 낭떠러지이면
            else
            {
                // Debug.Log("GameOver");
                gameController.GameOver();
                break;
            }
        }
    }
    private void Update()
    {
        // 아래쪽 방향으로 광선을 발사해 관선에 부딪히는 발판 정보를 hit에 저장
        Physics.Raycast(transform.position, Vector3.down, out hit, 1.0f);

        // 마우스 왼쪽 버튼 (or 모바일 터치)으로 플레이어의 x 위치 제어
        if (Input.GetMouseButton(0))
        {
            MoveToX();
        }
    }
    private void MoveToX()
    {
        float x = 0.0f;
        Vector3 position = transform.position;

        // 현재 플랫폼이 모바일 일 때
        if (Application.isMobilePlatform)
        {
            if (Input.touchCount > 0)
            {
                Touch touch = Input.GetTouch(0);
                // 0.0f ~ 1.0f의 값으로 만들고 -0.5f를 하기 때문에
                // -0.5f ~ 0.5f 사이의 값이 나온다.
                x = (touch.position.x / Screen.width) - 0.5f;
            }
        }
        // 현재 플랫폼이 PC 일 때
        else
        {
            // 참고 화면의 왼쪽 아래쪽이 0.0임
            // 0.0f~1.0f의 값으로 만들고 -0.5f를 하기 때문에 
            // -0.5f~0.5f 사이의 값이 나온다.
            x = (Input.mousePosition.x / Screen.width) - 0.5f;
        }

        // 화면 밖을 터치해 x 값이 -0.5~0.5의 범위를 넘어가지 않도록
        x = Mathf.Clamp(x, -0.5f, 0.5f);
        // 플레이어의 x 위치 설정
        position.x = Mathf.Lerp(position.x, x * xSensitivity, xSensitivity * Time.deltaTime);

        transform.position = position;
    }
    private IEnumerator MoveToYZ(float start, float end)
    {
        float current = 0;
        float percent = 0;
        float v0 = -gravity;        // y 방향의 초기속도

        while (percent < 1)
        {
            current += Time.deltaTime;
            percent = current / moveTime;

            // 시간 경과에 따라 오브젝트의 y 위치를 바꿔준다.
            // 포물선 운동 : 시작 위치 + 초기속도*시간 + 중력*시간제곱
            float y = minPositionY + (v0 * percent) + (gravity * percent * percent);
            playerObject.position = new Vector3(playerObject.position.x, y, playerObject.position.z);

            // 시간 경과(최대 1)에 따라 오브젝트의 z위치를 바꿔준다
            float z = Mathf.Lerp(start, end, percent);
            transform.position = new Vector3(transform.position.x, transform.position.y, z);

            yield return null;
        }
    }
    private IEnumerator DecreaseMoveTime()
    {
        while (true)
        {
            yield return new WaitForSeconds(1.0f);

            // 플레이어의 Y, Z 이동 시간 감소 (점점 빠르게 이동)
            moveTime -= 0.02f;

            // 이동 시간이 0.2f 이하이면 더이상 줄이지 않는다
            if (moveTime <= 0.2f)
            {
                break;
            }
        }
    }
}
```

# 발판 색상 설정

발판을 밟을 때 화면에 있는 전체 발판이 임의로 색상이 바뀌도록 설정한다.

(1) PlatformSpawner스크립트를 수정한다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlatformSpawner : MonoBehaviour
{
    [SerializeField]
    private Color[] platformColors;     // 발판 색상 배열
    [SerializeField]
    private Material platformMaterial;      // 발판 메터리얼 (색상 설정)

    [SerializeField]
    private GameObject platformPrefab;
    [SerializeField]
    private int spawnPlatformCountAtStart = 8;  // 게임 시작 시 최초 생성되는 발판 개수
    [SerializeField]
    private float xRange = 4;       // 발판이 배치될 수 있는 x 위치 범위(+-xRange)
    [SerializeField]
    private float zDistance = 5;    // 발판 사이의 거리 (z)
    private int platformIndex = 0;

    public float ZDistance
    {
        get { return zDistance; }
    }
    private void Awake()
    {
        // 최초 발판 색상은 platformColors[0] 색상으로 설정
        platformMaterial.color = platformColors[0];

        // spawnPlatformAtStart에 저장된 개수만큼 최초 플랫폼 생성
        for (int i = 0; i < spawnPlatformCountAtStart; i++)
        {
            SpawnPlatform();
        }
    }

    public void SpawnPlatform()
    {
        // 새로운 발판 생성
        GameObject clone = Instantiate(platformPrefab);
        // 발판 Setup() 메소드 호출
        clone.GetComponent<Platform>().SetUp(this);
        // 발판 위치 설정
        ResetPlatform(clone.transform);
    }

    public void ResetPlatform(Transform transform, float y = 0)
    {
        platformIndex++;

        // 발판이 배치되는 x 위치를 임의로 설정(-Range ~ Range)
        float x = Random.Range(-xRange, xRange);
        // 발판이 배치되는 위치 설정 (z축은 현재 발판 인덱스*zDistance)
        transform.position = new Vector3(x, y, platformIndex * zDistance);
    }
    public void SetPlatformColor()
    {
        // 화면에 등장하는 모든 플랫폼의 색상 설정
        int index=Random.Range(0,platformColors.Length);
        platformMaterial.color=platformColors[index];
    }
}
```

(2) GameController스크립트를 수정해준다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;
using TMPro;
public class GameController : MonoBehaviour
{
    [SerializeField]
    private PlatformSpawner platformSpawner;

    [SerializeField]
    private TextMeshProUGUI textTapToPlay;

    [SerializeField]
    private TextMeshProUGUI textScore;
    private int score = 0;

    [SerializeField]
    private TextMeshProUGUI textHighScore;

    private void Awake()
    {
        // 마지막 플레이에서 획득했던 점수 불러오가
        int score = PlayerPrefs.GetInt("LastScore");
        textScore.text = score.ToString();

        // 기존에 등록된 최고 점수 불러오기
        int highScore = PlayerPrefs.GetInt("HighScore");
        textHighScore.text ="High Score "+ highScore.ToString();
    }

    public void GameStart()
    {
        // Tap To Play 텍스트를 보이지 않게함
        textTapToPlay.enabled = false;
        // 최고 점수 보이지 않게 설정
        textHighScore.enabled = false;

        // 게임 시작 전에는 마지막 획득 점수가 출력되기 때문에 게임 시작 시 점수 텍스트 갱신
        textScore.text=score.ToString();
    }

    public void IncreaseScore()
    {
        score++;
        textScore.text=score.ToString();

        // score가 짝수 일 때 마다 모든 발판의 색상 변경
        if (score % 2 == 0)
        {
            platformSpawner.SetPlatformColor();
        }
    }

    public void GameOver()
    {
        // 기존의 등록된 최고 점수 불러오기
        int highScore = PlayerPrefs.GetInt("HighScore");
        if(score> highScore)
        {
            PlayerPrefs.SetInt("HighScore", score);
        }
        // 마지막에 획득한 점수 저장 (씬을 다시 로드 했을 때 마지막 점수 출력)
        PlayerPrefs.SetInt("LastScore", score);

        // 현재 씬을 다시 로드
        SceneManager.LoadScene(SceneManager.GetActiveScene().name);
    }

    private void OnApplicationQuit()
    {
        // 프로그램을 종료할 때 LastScore를 0으로 설정
        PlayerPrefs.SetInt("LastScore", 0);
    }
}
```

# 코인 활성화 및 획득 처리

(1) 코인 등장 확률을 설정하고 실제 코인이 등장하게 하기 위해 Platform스크립트를 수정한다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Platform : MonoBehaviour
{
    [SerializeField]
    private GameObject coinObject;      // 현재 발판에 소속된 코인 오브젝트
    [SerializeField]
    private int coinSpawnPercent = 50;      // 코인 등장 확률

    private PlatformSpawner platformSpawner;
    private Camera mainCamera;
    private float yMoveTime = 0.5f;

    public void SetUp(PlatformSpawner platformSpawner)
    {
        this.platformSpawner = platformSpawner;
        mainCamera = Camera.main;
    }

    private void Update()
    {
        // 플랫폼이 카메라 뒤로 가서 보이지 않게 되면 플랫폼 재배치
        if (mainCamera.transform.position.z - transform.position.z > 0)
        {
            // 플랫폼 위에 코인 활성/비활성화
            SpawnCoin();
            // 플랫폼 위치 재설정
            platformSpawner.ResetPlatform(this.transform);
            // 새로 등할 때는 위에서 떨어지는 효과 적용
            StartCoroutine(MoveY(10, 0));
        }
    }
    private IEnumerator MoveY(float start,float end)
    {
        float current = 0;
        float percent = 0;
        while (percent < 1)
        {
            current += Time.deltaTime;
            percent = current / yMoveTime;

            // 시간 경과 (최대 1)에 따라 오브젝트릐 y 위치를 바꿔준다
            float y = Mathf.Lerp(start, end, percent);
            transform.position = new Vector3(transform.position.x, y, transform.position.z);
            yield return null;
        }
    }
    private void SpawnCoin()
    {
        // 임의의 값 percent가 platformSpawner.SpawnCoinPercent보다 작으면 코인 생성
        int percent = Random.Range(0, 100);

        if (percent < coinSpawnPercent)
        {
            coinObject.SetActive(true);
        }
        else
        {
            coinObject.SetActive(false);
        }
    }
}
```

(2) 코인을 획득했을 때 이펙트 재생을 위해 파티클 시스템을 하나 만들고 이름을 CoinEffect로 바꾸고 알맞게 설정한다음 프리펩으로 만든다.

(3) 이펙트 재생이 완료되면 자동으로 삭제되는 ParticleAutoDestroyer스크립트를 만든다.

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
        if (particle.isPlaying == false)
        {
            Destroy(gameObject);
        }
    }
}
```

파티클 프리펩에 넣어준다.

(4) 코인 오브젝트를 제어하는 CoinController스크립트를 만든다.

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
        transform.Rotate(Vector3.right*rotateSpeed*Time.deltaTime);
    }

    private void OnTriggerEnter(Collider other)
    {
        GameObject clone =Instantiate(coinEffectPrefab);
        clone.transform.position = transform.position;

        gameObject.SetActive(false);
    }
}
```

