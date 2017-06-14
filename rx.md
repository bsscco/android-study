# ReactiveX

### ReactiveX
데이터를 흐름으로 만들고, 흐름에 변형을 가하여 데이터를 조작합니다. 이 흐름을 Observable이라고 부릅니다. Observable은 함수와 같아서 바로 사용하지 않고 가지고 있다가 필요한 시점에 사용할 수 있습니다. 데이터를 Observable에 흘려보내면 Observable에 정의된 변형을 통해 데이터가 변형되어 반환됩니다.

<br>

### map
흐름에서 차례대로 흘러오는 데이터마다 변형을 가합니다.

<br>

### flatMap
흐름에서 차례대로 흘러오는 데이터마다 변형을 거쳐 Observable이 된다면, 모든 Observable을 하나의 흐름으로 만들어줍니다.

<br>

### filter
흐름에서 차례대로 흘러오는 데이터마다 담을지, 안 담을지를 체크하고 담아야 하는 데이터들만 모아서 하나의 흐름으로 만들어줍니다.

<br>

### groupBy
흐름에서 차례대로 흘러오는 데이터마다 그룹 key를 물어서 그룹별로 GroupObservable을 만들고 GroupObservable들을 하나의 흐름으로 만들어줍니다.

<br>

### reduce
흐름에서 첫 데이터와 두번 째 데이터를 묶음으로 처리하게 하고, 그 반환 값과 세번 째 데이터를 묶음으로 처리하게 하고, 그 반환 값과 네번 째 데이터를 묶음으로 처리하게 하고, ... 그 반환 값과 마지막 데이터까지 묶음으로 처리하여 최종 하나의 반환 값을 반환하는 흐름을 만들어줍니다. 

<br>

### publish
연결 가능한 ConnectableObservable을 반환합니다. ConnectableObservable에 subscribe()를 여러 번 호출하면, 반환된 여러 개의 Subscriber들은 하나의 데이터 흐름을 구독할 수 있습니다. 데이터 흐름을 한 개만 사용하기 때문에 낭비가 없습니다.

<br>

### 팁
Subscriber들을 한 곳에 담아 관리할 수 있는 CompositSubscription이 있습니다. CompositSubscription.clear()로 한꺼번에 구독을 해지할 수 있습니다. unsubscribe()는 액티비티가 강제종료됐다가 재실행될 때 잘못된 동작을 일으킬 수 있으니 사용하지 말라고 합니다.
<br>

그리고 에러 핸들러도 꼭 정의해야 합니다. 에러 발생 시 스택트레이스에 아무것도 나오지 않을 수 있으므로 에러 핸들러에서 로그를 남기는 일을 반드시 합시다.

<br>

### 참고
- [Realm 뉴스 : 누구나 할 수 있는 RxJava, 지금 시작하기!](https://news.realm.io/kr/news/mobilization-hugo-visser-rxjava-for-rest-of-us/)
- [[Realm 뉴스 : Rx와 Kotlin으로 간결하고 명료하게 모든 것을 조립해보세요!](https://news.realm.io/kr/news/compose-everything-rx-kotlin/)
