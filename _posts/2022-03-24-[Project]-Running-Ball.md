---
layout: post
title:  "[Project] Running Ball"
subtitle:   ""
categories: Unity
comments: true
---

<br>

# 프로젝트 기본 설정 

(1) MainCamera의 위치:(0 5 -13) 회전:(30 0 0) Camera컴포넌트 Clear Flags:Solid Color / 배경: 검은색

(2) 게임 화면의 해상도를 10 : 16으로 설정

(3) 임시 바닥으로 사용 할 큐브 오브젝트를 생성하고 이름을 Ground로 설정하고 크기:(5 /0.1 /20), MeshRenderer - materials : Ground를 넣어준다

# 터치 - 드래그를 이용한 플레이어 이동

(1) 플레이어로 사용 할 스피어 오브젝트를 만들고 이름을 Player로 바꾼다.

(2) 플레이어 오브젝트에 물리 처리를 위해 RigidBody컴포넌트를 추가하고 Constraints를 열어 플레이어가 바닥에 있을 때 낭떠러지를 만나면 아래로 떨어지도록 Freeze Position Y만 물리/중력에 의해 이동이 되도록 나머지를 다 체크해준다.

![image](https://user-images.githubusercontent.com/101051124/159831844-9bcd4c1d-98fa-44b4-927b-0ad8ee69496d.png)

(3) 플레이어 오브젝트의 위치: (0/ 0.55/ -9)로 설정해준다.

(4) 플레이어 이동을 제어하는 Movement스크립트를 만든다.

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Movement : MonoBehaviour
{
    // x축 이동
    private float moveXWidth = 1.5f;        // 1회 이동 시 이동 거리 (x축)
    private float moveTimeX = 0.1f;         // 1회 이동에 소모되는 시간 (x축)
    private bool isXMove = false;             // true : 이동 중 , false : 이동 가능
    // y축 이동
    private float originY = 0.55f;              // 점프 및 착지하는 y축 값
    private float gravity = -9.81f;             // 중력
    private float moveTimeY = 0.3f;         // 1회 이동에 소요되는 시간 (y축)
    private bool isJump = false;               //true : 점프 중 , false : 점프 가능
    // z축 이동
    [SerializeField]
    private float moveSpeed = 20.0f;    // 이동 속도 (z축)
    // 회전 
    private float rotateSpeed = 300.0f;       // 회전 속도

    private float limitY = -1.0f;       //플레이어가 사망하는 y 위치

    private Rigidbody rigidbody;

    private void Awake()
    {
        rigidbody = GetComponent<Rigidbody>();
    }
    private void Update()
    {
        // z축 이동 
        transform.position+=Vector3.forward*moveSpeed*Time.deltaTime;

        // 오브젝트 회전 (x축)
        transform.Rotate(Vector3.right*rotateSpeed*Time.deltaTime);

        // 낭떠러지로 떨어지면 플레이어 사망
        if (transform.position.y < limitY)
        {
            Debug.Log("사망");
        }
    }
    public void MoveToX(int x)  // x는 1 아니면 -1
    {
        // 현재 x축 이동 중으로 이동 불가능
        if (isXMove == true) return;

        if( x>0 && transform.position.x<moveXWidth)
        {
            StartCoroutine(OnMoveToX(x));
        }
        else if(x<0 && transform.position.x> -moveTimeX)
        {
            StartCoroutine(OnMoveToX(x));
        }
    }
    public void MoveToY()
    {
        // 현재 점프 중으로 점프 불가능
        if(isJump == true) return;

        StartCoroutine(OnMoveToY());
    }
    private IEnumerator OnMoveToX(int direction)
    {
        float current = 0;
        float percent = 0;
        float start=transform.position.x;
        float end=transform.position.x +direction*moveXWidth;

        isXMove = true;
        while (percent < 1)
        {
            current += Time.deltaTime;
            percent = current / moveTimeX;

            float x=Mathf.Lerp(start, end, percent);
            transform.position=new Vector3(x,transform.position.y,transform.position.z);

            yield return null;
        }
        isXMove = false;
    }
    private IEnumerator OnMoveToY()
    {
        float current = 0;
        float percent = 0;
        float v0 = -gravity;    // y방향의 초기 속도
        isJump = true;
        rigidbody.useGravity = false;
        while(percent < 1)
        {
            current += Time.deltaTime;
            percent=current / moveTimeY;

            // 시간 경과에 따라 오브젝트의 y 위치를 바꿔준다.
            // 포물선 운동 : (시작 위치) +(초기속도 * 시간) + (중력 * 시간제곱) 
            float y = originY =(originY) + (v0*percent)+(gravity*percent*percent);
            transform.position=new Vector3(transform.position.x,y,transform.position.z);

            yield return null;
        }
        isJump = false;
        rigidbody.useGravity=true;
    }
}
```

(5) 터치-드래그로 플레이어 행동을 제어하는 PlayerController스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    [SerializeField]
    private float dragdistance = 50.0f;     // 드래그 거리
    private Vector3 touchStart;                // 터치 시작 위치
    private Vector3 touchEnd;                  // 터치 종료 위치

    private Movement movement;

    private void Awake()
    {
        movement = GetComponent<Movement>();
    }

    private void Update()
    {
        if (Application.isMobilePlatform)
        {
            OnMobilePlatform();
        }
        else
        {
            OnPCPlatform();
        }
    }
    private void OnMobilePlatform()
    {
        // 현재 화면을 터치하고 있지 않으면 메소드 종료
        if (Input.touchCount == 0) return;

        // 첫 번째 터치 정보 불러오기
        Touch touch=Input.GetTouch(0);

        // 터치 시작
        if(touch.phase == TouchPhase.Began)
        {
            touchStart = touch.position;
        }
        // 터치 & 드래그
        else if(touch.phase == TouchPhase.Moved)
        {
            touchEnd=touch.position;

            OnDragXY();
        }
    }
    private void OnPCPlatform()
    {
        // 터치 시작
        if (Input.GetMouseButtonDown(0))
        {
            touchStart = Input.mousePosition;
        }
        // 터치 & 드래그
        else if (Input.GetMouseButton(0))
        {
            touchEnd = Input.mousePosition;

            OnDragXY();
        }
    }
    private void OnDragXY()
    {
        // 터치 상태로 x축 들래그 범위가 dragDistance보다 클 때
        if (Mathf.Abs(touchEnd.x - touchStart.x) >= dragdistance)
        {
            // Mathf.Sign() -> 매개변수로 넘긴 값이 0 or 양수 : 1 , 음수 : -1을 반환
            movement.MoveToX((int)Mathf.Sign(touchEnd.x - touchStart.x));
        }

        // 터치 상태로 y축 양의 방향으로 드래그 범위가 dragDistance보다 클 때
        if (touchEnd.y - touchStart.y >= dragdistance)
        {
            movement.MoveToY();
        }
    }
}
```

(6) 플레이어 오브젝트에 Movement, PlayerController스크립트를 컴포넌트로 추가한다.

이제 플레이어가 위로 드래그하면 점프를하고 좌,우로 드래그하면  드래그한 방향으로 움직이는 걸 볼 수 있다.

# 카메라 오브젝트

(1) 플레이어 오브젝트를 추적하는 카메라 이동을 위해 CameraController 스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CameraController : MonoBehaviour
{
    [SerializeField]
    private Transform target;
    private float zDistance;

    private void Awake()
    {
        zDistance=target.position.z-transform.position.z;
    }
    private void LateUpdate()
    {
        Vector3 position = transform.position;
        position.z=target.position.z-zDistance;
        transform.position=position;
    }
}
```

메인 카메라 오브젝트에 컴포넌트로 추가하고 실행하면 카메라가 플레이어 오브젝트를 쫒아가는 모습을 볼 수 있다.

# 구역 오브젝트

(1) 프리펩으로 만들어 둔 구역 중 임의의 구역을 생성하는 AreaSpawner스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class AreaSpawner : MonoBehaviour
{
    [SerializeField]
    private GameObject[] areaPrefabs;        // 구역 프리펩
    [SerializeField]
    private int spawnAreaCountAtStart = 3;     // 게임 시작 시 최초 생성되는 구역 개수
    [SerializeField]
    private float zDistance = 20;         // 구역 사이의 거리
    private int areaIndex = 0;          // 구역 인덱스 (배치되는 구역의 z 위치 연산에 사용)

    private void Awake()
    {
        // spawnAreaCountAtStart에 저장된 개수 만큼 최초 구역 생성
        for(int i = 0; i < spawnAreaCountAtStart; i++)
        {
            // 첫 번째 구역은 항상 0번 구역 프리펩으로 설정
            if (i == 0)
            {
                SpawnArea(false);
            }
            else
            {
                SpawnArea();
            }
        }
    }
    public void SpawnArea(bool isRandom = true)
    {
        GameObject clone = null;
        if (isRandom == false)
        {
            clone = Instantiate(areaPrefabs[0]);
        }
        else
        {
            int index=Random.Range(0,areaPrefabs.Length);
            clone = Instantiate(areaPrefabs[index]);
        }
        // 구역이 배치되는 위치 설정 (z축은 현재 구역 인덱스 * zDistance)
        clone.transform.position =new Vector3(0,0, areaIndex * zDistance);
        areaIndex++;
    }
}
```

(2) 빈 오브젝트를 만드러 이름을 AreaSpawner로 바꾸고 AreaSpawner스크립트를 컴포넌트로 추가한다.

(3) AreaSpawner컴포넌트 변수에 미리 만들어둔 구역셋을 등록해준다.

(4) 미리 만들어둔 Ground오브젝트를 삭제하고 게임을 실행한다.

![image](https://user-images.githubusercontent.com/101051124/159844905-9ecbeb7d-960a-4a8a-a074-ffa186a2f4d7.png)

![image](https://user-images.githubusercontent.com/101051124/159844985-1ee16d98-062c-436a-b256-b19727dbad99.png)

구역이 3개 생성되어 이어진 것을 볼 수 있다.

# 화면에서 사라진 구역 삭제, 새로운 구역 생성

(1) 구역 오브젝트를 제어하는 Area스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Area : MonoBehaviour
{
    [SerializeField]
    private float destroyDistance = 15f;
    private AreaSpawner areaSpawner;
    private Transform playerTransform;

    public void SetUp(AreaSpawner arearSpawner, Transform playerTransform)
    {
        this.areaSpawner = arearSpawner;
        this.playerTransform = playerTransform;
    }
    private void Update()
    {
        if(playerTransform.position.z-transform.position.z >= destroyDistance)
        {
            // 새로운 구역 생성
            areaSpawner.SpawnArea();
            // 현재 구역 삭제
            Destroy(gameObject);
        }
    }
}
```

(2) 구역을 사용되는 AreaSet을 모두 선택하고 Area스크립트를 컴포넌트로 추가해준다.

(3) AreaSpawner 스크립트를 수정해준다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class AreaSpawner : MonoBehaviour
{
    [SerializeField]
    private GameObject[] areaPrefabs;        // 구역 프리펩
    [SerializeField]
    private int spawnAreaCountAtStart = 3;     // 게임 시작 시 최초 생성되는 구역 개수
    [SerializeField]
    private float zDistance = 20;         // 구역 사이의 거리
    private int areaIndex = 0;          // 구역 인덱스 (배치되는 구역의 z 위치 연산에 사용)

    //============================================
    [SerializeField]
    private Transform playerTransform;
    //============================================
    private void Awake()
    {
        // spawnAreaCountAtStart에 저장된 개수 만큼 최초 구역 생성
        for(int i = 0; i < spawnAreaCountAtStart; i++)
        {
            // 첫 번째 구역은 항상 0번 구역 프리펩으로 설정
            if (i == 0)
            {
                SpawnArea(false);
            }
            else
            {
                SpawnArea();
            }
        }
    }
    public void SpawnArea(bool isRandom = true)
    {
        GameObject clone = null;
        if (isRandom == false)
        {
            clone = Instantiate(areaPrefabs[0]);
        }
        else
        {
            int index=Random.Range(0,areaPrefabs.Length);
            clone = Instantiate(areaPrefabs[index]);
        }
        // 구역이 배치되는 위치 설정 (z축은 현재 구역 인덱스 * zDistance)
        clone.transform.position =new Vector3(0,0, areaIndex * zDistance);

        //===================================================
        // 구역이 삭제될 때 새로운 구역을 생성할 수 있도록 this와 플레이어의 Transform 정보 전달
        clone.GetComponent<Area>().SetUp(this,playerTransform);
        //===================================================

        areaIndex++;
    }
}
```

이제 게임을 실행해보면 구역이 화면 밖으로 사라지면 새로운 구역이 생기는 걸 볼 수 있다.

# 코인 오브젝트 획득

(1) 코인 오브젝트를 획득했을 때 이펙트가 재생되기 때문에 이펙트 재생이 종료되면 이펙트를 삭제하는 ParticleAutoDestroyer스크립트를 만든다.

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

(2) 코인이펙트 프리펩에 ParticleAutoDestroyer스크립트를 컴포넌트로 추가해준다.

(3) 코인오브젝트 회전과 충돌 시 처리를 제어하는 Coin스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Coin : MonoBehaviour
{
    [SerializeField]
    private GameObject coinEffectPrefab;
    private float rotateSpeed;

    private void Awake()
    {
        rotateSpeed = Random.Range(0, 360);
    }

    private void Update()
    {
        transform.Rotate(Vector3.right*rotateSpeed*Time.deltaTime);
    }

    private void OnTriggerEnter(Collider other)
    {
        // 코인 이펙트 오브젝트 생성
        GameObject clone=Instantiate(coinEffectPrefab);
        clone.transform.position = transform.position;

        Destroy(gameObject);
    }
}
```

(4) 코인 프리펩에 Coin스크립트를 컴포넌트로 추가하고 코인이펙트 프리펩을 할당해준다.

![image](https://user-images.githubusercontent.com/101051124/159849724-14f534f0-dc35-4b0e-b379-1147c16dfe2c.png)

이제 플레이어가 코인을 먹으면 코인 획득 이펙트가 발생하고 코인 오브젝트는 사라진다.

# 게임 내 UI 출력

(1) Game Title 텍스트 출력을 위해 TextMeshPro를 생성하고 이름을 TextGameTitle로 바꾸고 알맞게 설정해준다.

(2) Tap To Play 텍스트 출력을 위해 TextGameTitle를 복제하고 이름을 TextTapToPlay로 바꾸고 알맞게 설정한다.

(3) 코인 획득 개수를 출력하기 위해 TextTapToPlay를 복제하고 이름을 TextCoinCount로 바꾸고 알맞게 설정한다.

# Tap To Play 페이드 효과

(1) 페이드 효과를 제어하는 FadeEffect스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using TMPro;

public class FadeEffect : MonoBehaviour
{
    [SerializeField]
    private float fadeTime;     // 페이드 되는 시간
    private TextMeshProUGUI textFade;       // 페이드 효과에 사용되는 TMPro

    private void Awake()
    {
        textFade = GetComponent<TextMeshProUGUI>();

        // 페이드 효과 In -> Out 무한 반복한다
        StartCoroutine(FadeInOut());
    }
    private IEnumerator FadeInOut()
    {
        while (true)
        {
            yield return StartCoroutine(Fade(1, 0));   // Fade In

            yield return StartCoroutine(Fade(0, 1));    // Fade Out
        }
    }
    private IEnumerator Fade(float start,float end)
    {
        float current = 0;
        float percent = 0;

        while (percent < 1)
        {
            current+=Time.deltaTime;
            percent = current / fadeTime;

            Color color=textFade.color;
            color.a=Mathf.Lerp(start, end, percent);
            textFade.color=color;

            yield return null;
        }
    }
}
```

(2) TextTapToPlay오브젝트에 FadeEffect를 컴포넌트로 추가해주고 FadeTime을 0.5로 설정한다.

# 게임 시작 전, 후 보여지는 UI 설정

(1) 코인 획득 개수 출력과 게임 진행을 제어하는 GameController스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;
using TMPro;
public class GameController : MonoBehaviour
{
    [SerializeField]
    private TextMeshProUGUI textTitle;
    [SerializeField]
    private TextMeshProUGUI textTapToPlay;

    [SerializeField]
    private TextMeshProUGUI textCoinCount;
    private int coinCount = 0;

    public bool IsGameStar { private set; get; }

    private void Awake()
    {
        IsGameStar = false;

        textTitle.enabled = true;
        textTapToPlay.enabled = true;
        textCoinCount.enabled = false;
    }
    private IEnumerator Start()
    {
        // 마우스 왼쪽 클릭 하기 전까지 시작하지 않고 대기
        while (true)
        {
            if (Input.GetMouseButtonDown(0))
            {
                IsGameStar = true;

                textTitle.enabled = false;
                textTapToPlay.enabled =false;
                textCoinCount.enabled = true;
                break;
            }
            yield return null;
        }
    }
    public void IncreaseCoinCount()
    {
        coinCount++;
        textCoinCount.text = coinCount.ToString();
    }

    public void GameOver()
    {
        // 현재 씬을 다시 로드
        SceneManager.LoadScene(SceneManager.GetActiveScene().name);
    }
}
```

(2) GameController스크립트를 실행하기 위해 빈 오브젝트를 만들고 이름을 GamController로 바꾼 다음 GameController스크립트를 컴포넌트로 추가해준다. 그리고 각 변수에 알맞는 텍스트 오브젝트를 할당해준다.

(3) IsGameStart가 false일 때 움직이지 않게 설정하기 위해 Movement스크립트를 수정한다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Movement : MonoBehaviour
{
    //==================================================
    [SerializeField]
    private GameController gameController;  // IsGameStart가 true일 때 움직이기 위해
    //==================================================

    // x축 이동
    private float moveXWidth = 1.5f;        // 1회 이동 시 이동 거리 (x축)
    private float moveTimeX = 0.1f;         // 1회 이동에 소모되는 시간 (x축)
    private bool isXMove = false;             // true : 이동 중 , false : 이동 가능
    // y축 이동
    private float originY = 0.55f;              // 점프 및 착지하는 y축 값
    private float gravity = -9.81f;             // 중력
    private float moveTimeY = 0.3f;         // 1회 이동에 소요되는 시간 (y축)
    private bool isJump = false;               //true : 점프 중 , false : 점프 가능
    // z축 이동
    [SerializeField]
    private float moveSpeed = 20.0f;    // 이동 속도 (z축)
    // 회전 
    private float rotateSpeed = 300.0f;       // 회전 속도

    private float limitY = -1.0f;       //플레이어가 사망하는 y 위치

    private Rigidbody rigidbody;

    private void Awake()
    {
        rigidbody = GetComponent<Rigidbody>();
    }
    private void Update()
    {

        //==========================================
        // 현재 상태가 게임 시작이 아니면 Update()를 실행하지 않는다
        if (gameController.IsGameStart == false) return;
        //==========================================

        // z축 이동 
        transform.position+=Vector3.forward*moveSpeed*Time.deltaTime;

        // 오브젝트 회전 (x축)
        transform.Rotate(Vector3.right*rotateSpeed*Time.deltaTime);

        // 낭떠러지로 떨어지면 플레이어 사망
        if (transform.position.y < limitY)
        {
            Debug.Log("사망");
        }
    }
    public void MoveToX(int x)  // x는 1 아니면 -1
    {
        // 현재 x축 이동 중으로 이동 불가능
        if (isXMove == true) return;

        if( x>0 && transform.position.x<moveXWidth)
        {
            StartCoroutine(OnMoveToX(x));
        }
        else if(x<0 && transform.position.x> -moveTimeX)
        {
            StartCoroutine(OnMoveToX(x));
        }
    }
    public void MoveToY()
    {
        // 현재 점프 중으로 점프 불가능
        if(isJump == true) return;

        StartCoroutine(OnMoveToY());
    }
    private IEnumerator OnMoveToX(int direction)
    {
        float current = 0;
        float percent = 0;
        float start=transform.position.x;
        float end=transform.position.x +direction*moveXWidth;

        isXMove = true;
        while (percent < 1)
        {
            current += Time.deltaTime;
            percent = current / moveTimeX;

            float x=Mathf.Lerp(start, end, percent);
            transform.position=new Vector3(x,transform.position.y,transform.position.z);

            yield return null;
        }
        isXMove = false;
    }
    private IEnumerator OnMoveToY()
    {
        float current = 0;
        float percent = 0;
        float v0 = -gravity;    // y방향의 초기 속도
        isJump = true;
        rigidbody.useGravity = false;
        while(percent < 1)
        {
            current += Time.deltaTime;
            percent=current / moveTimeY;

            // 시간 경과에 따라 오브젝트의 y 위치를 바꿔준다.
            // 포물선 운동 : (시작 위치) +(초기속도 * 시간) + (중력 * 시간제곱) 
            float y =(originY) + (v0*percent)+(gravity*percent*percent);
            transform.position=new Vector3(transform.position.x,y,transform.position.z);

            yield return null;
        }
        isJump = false;
        rigidbody.useGravity=true;
    }
}
```

이제 게임을 실행해보면 마우스 좌클릭을 하기 전까지는 움직이지 않고 UI들이 설정대로 나타나거나 사라진다.

# 플레이어와 코인,장애물 충돌 처리

(1) 플레이어와 코인, 플레이어와 장애물의 충돌 처리를 위해 플레이어의 자식으로 빈 오브젝트를 생성하고 이름을 PlayerCollision으로 바꾼다. 충돌 처리를 위해 스피어 콜라이더와 리지드바디를 컴포넌트로 추가한다.

(2) 스피어 콜라이더의 is Trigger를 체크하고 리지드바디의 use Gravity를 체크 해제한다.

(3) 플레이어의 충돌을 처리하는 PlayerCollision스크립트를 만든다.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerCollision : MonoBehaviour
{
    [SerializeField]
    private GameController gameController;

    private void OnTriggerEnter(Collider other)
    {
        if (other.gameObject.tag == "Obstacle")
        {
            gameController.GameOver();
        }
        else if(other.gameObject.tag == "Coin")
        {
            gameController.IncreaseCoinCount();
        }
    }
}
```

(4) PlayerCollision오브젝트에 PlayerCollision스크립트를 컴포넌트로 추가해주고 GameController변수에 GameController오브젝트를 할당해준다.

이제 게임을 실행하면 코인을 획득했을 때는 코인의 갯수가 증가하고 장애물에 부딪히면 게임이 종료 된다.