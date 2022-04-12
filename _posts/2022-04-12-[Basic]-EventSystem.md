---
layout: post
title:  "[Basic] EventSystem"
subtitle:   ""
categories: Unity
comments: true
---

<br>

Eventsystem은 많은 이벤트를 지원하고 있으며, InputModule을 작성하여 사용자 정의 할 수 있습니다.

이벤트는 인터페이스가 제공하는 StandaloneInputModule과 TouchInputModule에 지원되고 있으며, 인터페이스를 구현하여 MonoBehaviour에 구현할 수 있습니다. 이벤트를 설정한 유효한 EventSystem이 있으면 적절한 타이밍에 호출됩니다.

- IPointerEnterHandler - OnPointerEnter - 포인터가 오브젝트에 들어왔을 때 호출됩니다.
- IPointerExitHandler - OnPointerExit - 포인터가 오브젝트로부터 멀어졌을 때에 호출됩니다.
- IPointerDownHandler - OnPointerDown - 포인터가 오브젝트를 눌렀을 때 호출됩니다.
- IPointerUpHandler - OnPointerUp - 포인터를 뗐을 때 호출됩니다(눌려 있는 오브젝트에서 호출됩니다).
- IPointerClickHandler - OnPointerClick - 오브젝트에서 포인터를 누르고 동일한 오브젝트에서 뗄 때 호출됩니다.
- IInitializePotentialDragHandler - OnInitializePotentialDrag - 드래그 대상이 발견되었을 때에 호출됩니다. 값을 초기화하는 데 사용할 수 있습니다.
- IBeginDragHandler - OnBeginDrag - 드래그 시작 직전에 드래그 대상 오브젝트에서 호출됩니다.
- IDragHandler - OnDrag - 드래그 대상이 드래그되는 동안 호출됩니다.
- IEndDragHandler - OnEndDrag - 드래그가 종료했을 때, 드래그 대상 오브젝트에서 호출됩니다.
- IDropHandler - OnDrop - 드래그를 멈춘 위치에 있는 오브젝트에서 호출됩니다.
- IScrollHandler - OnScroll - 마우스 휠 스크롤 했을 때에 호출됩니다.
- IUpdateSelectedHandler - OnUpdateSelected - 선택중인 오브젝트에서 매 프레임 호출됩니다.
- ISelectHandler - OnSelect - 오브젝트가 선택된 순간, 그 오브젝트에서 호출됩니다.
- IDeselectHandler - OnDeselect - 선택중인 오브젝트에서 선택 상태가 해제될 때 호출됩니다.
- IMoveHandler - OnMove - 키 입력에 의한 이동 이벤트(왼쪽, 오른쪽, 위, 아래 등)가 발생했을 때 호출됩니다.
- ISubmitHandler - OnSubmit - Submit 버튼을 눌렀을 때 호출됩니다.
- ICancelHandler - OnCancel - 취소 버튼을 눌렀을 때 호출됩니다.