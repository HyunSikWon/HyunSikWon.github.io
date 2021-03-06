---
title: iOS - setNeedsLayout과 layoutIfNeeded
layout: single
comments: true
share: true
categories: 
- iOS
tag:
- UIView
last_modified_at: 2021-06-14 T15:45:00+08:00
---

`UIView`의 view-layout 메소드인 `setNeedsLayout`과 `layoutIfNeeded`에 대해 알아보려 합니다. `setNeedsLayout`과 `layoutIfNeeded`을 알아보기 전에 Main Run Loop 개념을 먼저 살펴보자. 

## Main Run Loop

앱의 메인 런 루프는 모든 사용자 관련 이벤트를 처리한다. `UIApplication` 객체는 앱 시작시 기본 실행 루프를 설정하고 이를 사용하여 이벤트를 처리하여 뷰 기반 인터페이스에 대한 업데이트를 수행한다. 이름에서 알 수 있듯이 메인 런 루프는 앱의 메인 스레드에서 실행된다. 이 동작은 사용자 관련 이벤트가 수신 된 순서대로 순차적으로 처리된다. 

![1*oDckiqvUj_95hNrFliDisw](https://user-images.githubusercontent.com/48352065/122876308-53c2b800-d370-11eb-8741-6e8e9e1317d3.png)

이벤트는 앱에서 내부적으로 대기열(Event queue)에 추가되고 실행을 위해 메인 런 루프에 하나씩 전달된다. `UIApplication` 객체는 큐로부터 이벤트를 수신하고 적절한 타켓 객체에 전달한다. 이러한 과정에서 각 이벤트들에 알맞는 핸들러를 찾아 그들에게 처리 권한을 위임하며 진행된다. 각 핸들러는 개발자가 작성한 코드를 호출한다. 

## Update Cycle

업데이트 사이클은 앱이 모든 이벤트 처리 코드 실행을 마친 후 컨트롤이 기본 실행 루프로 반환되는 지점이다. 이 시점에서 시스템이 layout, display, constraint를 갱신한다. 만약, 이벤트 핸들러를 처리하는 동안 뷰에서 변경을 요청하면 시스템은 뷰를 다시 그리기가 필요한 것으로 표시한다(예약). 이후, 다음 업데이트주기에서 시스템은 이러한뷰에 대한 모든 변경 사항을 실행한다. 

여기서 알 수 있는 점은 유저 인터랙션이 일어나는 시점과 유저 인터랙션을 통해 발생하는 뷰의 수정사항이 실제로 갱신되는 시점에 시간차가 존재한다는 것이다. 물론 이 런 루프가 갱신되는 데에는 1/60초 밖에 걸리지 않기 때문에 유저는 이러한 시간차를 인지하지 못하고 애플리케이션을 사용하게된다. 

그러나 이벤트가 처리되는 시점과 해당 뷰가 다시 그려지는 시점 사이의 간격으로 인해 실행 루프 중 특정 지점에서 원하는 방식으로 뷰가 업데이트되지 않을 수 있다. 뷰의 최신 콘텐츠 또는 레이아웃에 의존하는 계산이있는 경우 뷰에 대한 이전 정보로 작업 할 위험이 있다. 따라서 개발자가 런 루프, 업데이트주기 및 특정 `UIView` 메서드를 이해하고 있으면 이러한 문제를 방지하거나 디버깅하는 데 도움이 될 수 있다.

## Laying Out Subviews

레이아웃이란 스크린에서의 뷰의 크기와 위치를 의미한다. `UIView`의 메소드를 통해서 AutoLayout을 사용하지 않고 수동으로 뷰의 레이아웃을 조정할 수 있다. 이 메소드들은 시스템에게 뷰의 레이아웃이 바뀌었다고 알리기 때문에 레이아웃이 갱신되었을 때 수행할 적절한 동작을 메소드 내부에 구현할 수 있다.

### 1. layoutSubviews

`layoutSubviews`는 뷰와 자신의 모든 서브뷰들의 위치와 크기를 재조정을 담당하는 메소드이다. 시스템은 뷰(혹은 서브뷰)의 프레임을 다시 계산해야할 때 이 메소드를 호출한다. 서브클래스는 서브뷰의 정확한 레이아웃을 수행하기 위해 이 메소드를 재정의 할 수 있다.  이 메소드는 연쇄적으로 서브뷰들의 `layoutSubviews` 또한 호출하므로 성능에 큰 부담을 줄 수 있는 메소드이다. 따라서 이 메소드를 직접 호출해서는 안되고, 이 메소드의 트리거인 `setNeedsLayout()`, `layoutIfNeeded()` 를 통해 간접적으로 호출해야 한다. 

### 2. setNeedsLayout

바로 다음 업데이트 사이클에 레이아웃이 갱신될 수 있도록 마크해두는 것이다. 즉, layoutSubviews를 예약하는 메소드이다.  이 메소드는 어플리케이션의 메인 스레드에서 호출해야한다. 다음 업데이트 사이클에 갱신을 예약해두고 바로 반환되는 비동기적인 메소드이다. 즉시 레이아웃을 업데이트 시키지 않는대신 다음 업데이트 사이클을 기다리므로 몇몇 뷰들이 갱신되기 전에 레이아웃을 무효화할 수 있다. 이러한 동작을 통해 여러 레이아웃 업데이트를 하나로 묶어 한번의 업데이트 사이클에서 갱신할 수 있어서 성능상의 이점이 있다.

### 3. layoutIfNeeded

서브 뷰의 레이아웃을 즉시 갱신하는 메소드이다. 즉 업데이트 사이클이 올 때까지 기다려 `layoutSubviews`를 호출시키는 것이 아니라 그 즉시 `layoutSubviews`를 발동시키는 메소드이다. 오토 레이아웃을 사용할 때 레이아웃 엔진은 제약 조건의 변경을 충족하기 위해 필요에 따라 뷰의 위치를 업데이트한다. 이 메소드는 메시지를 수신하는 뷰를 루트 뷰로 사용하여 해당 루트 뷰의 서브 뷰들의 레이아웃을 갱신한다. 

그런데 어차피 다음 업데이트 싸이클까지 기다리는 시간차가 유저에게 영향을 미치지 않을정도로 짧은데 굳이 즉시 뷰 갱신을 하는 이유는 뭘까? 이 메소드는 레이아웃에 의존해야하거나 다음 업데이트 싸이클까지 기다릴 수 없을 때 유용하다. 특히 애니메이션에서 자주 사용된다. 그 이유는 애니메이션이 시작되기 전에 모든 레이아웃이 업데이트 된 상황이어야하기 때문이다.

## Reference

[https://developer.apple.com/documentation/uikit/uiview](https://developer.apple.com/documentation/uikit/uiview)

[https://tech.gc.com/demystifying-ios-layout](https://tech.gc.com/demystifying-ios-layout/)

[https://sihyungyou.github.io/iOS-setNeedslayout-layoutIfNeeded/](https://sihyungyou.github.io/iOS-setNeedslayout-layoutIfNeeded/)

[https://baked-corn.tistory.com/105](https://baked-corn.tistory.com/105)
