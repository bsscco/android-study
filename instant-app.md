# 링크
- [공식 가이드](https://developer.android.com/topic/instant-apps/index.html)
- [공식 코드랩](https://codelabs.developers.google.com/codelabs/android-instant-apps/index.html)
- [플레이 콘솔 인스턴트앱 가이드](https://support.google.com/googleplay/android-developer/answer/7381861#production)


# 소개
### 인스턴트앱이란?
- 7MB 짜리 앱이 있다.
- 이 앱은 상품 목록 보기 기능(A)과 상품 정보 기능(B)을 가지고 있다.
- A가 4MB, B가 3MB를 차지한다.
- 이 앱을 인스턴트앱으로 만들었다.
- 이 앱을 설치하지 않은 유저는 A에 대한 다운로드 URL을 통해 A의 코드만 내려받을 수 있다. 그리고 A를 사용한다.
- 앱을 설치하지 않고 기능 일부만 내려받았을 경우엔 기능을 종료함과 동시에 관련 코드가 흔적없이 사라진다.
- 우리는 인스턴트앱을 통해 일부 핵심 기능만 사용시켜주고, 앱 설치를 유도할 수 있다.
- 주의사항: 인스턴트앱을 만들 때 내려 받을 수 있는 하나의 기능의 크기는 최대 4MB이다.

### 인스턴트앱의 목적
- 유저가 빠르고 만족스럽게 작업을 처리할 수 있도록 돕는 것이다.

### 지원 범위
- 인스턴트앱은 마시멜로우 버전부터 지원한다.
- Android Studio 3.0에서 빌드 가능하다.

### 인스턴트앱을 지원하는 앱의 구성
- 구성
	- base feature apk(공통 모듈이 들어간다.)
	- feature1 apk
	- app apk
- 설치앱에서 app은 base feature를 사용한다. 
- 인스턴트앱에서 feature1은 base feature를 사용한다.

### 기능(feature) 다운로드 하기
- 구글플레이로부터 feature를 다운로드할 수 있다.
- 유저가 feature에 대응하는 link를 클릭하면, 다운로드 및 실행은 구글플레이가 알아서 한다.
- 1개 feature가 여러 개의 entry-point 액티비티들을 가진다면, 여러 개의 url을 가질 수 있는데, path가 겹쳐서 우선순위가 필요한 경우를 생각해서 매니페이스트에서 priority를 줄 수 있다.


# 인스턴트앱 가이드
### 제거하기
- 필요없는 서드파티 의존성, 컴포넌트, 퍼미션들을 지웁니다.
- 이러한 작업은 앱 사이즈를 줄이는 데 큰 도움이 됩니다.

### AppLinks 걸기
- 인스턴트앱은 AppLinks를 사용합니다.
- 유저가 인스턴트앱의 link를 클릭했을 때 앱 내 액티비티를 실행합니다. 그것이 실패하거나 지원하지 않는 기기에서 클릭한 것이면, 웹사이트로 이동시킵니다. 

### 리팩토링 하기
- 필요하다면 URL addressable 구조로 리팩토링해야 합니다. 각 피처는 4MB 이하여야 합니다.

### 런타임 퍼미션을 적용하기
- 피처는 런타임 퍼미션만을 사용해야 합니다.

### 호환되는 라이브러리들
- Google Analytics나 Firebase Analytics 등 몇 개 안 됨.
- [참고](https://developer.android.com/topic/instant-apps/prepare.html#identify_tested_compatible_libraries)

### 허락되는 암시적 브로드캐스트들
- Intent.ACTION_SCREEN_ON 등 몇 개 받을 수 있음.
- [참고](https://developer.android.com/topic/instant-apps/prepare.html#identify_tested_compatible_libraries)

### 체크리스트
- 인스턴트앱의 모든 인텐트 필터들은 https와 http 프로토콜을 지원해야 합니다.
- 모든 인텐트 필터들은 CATEGORY_BROWABLE과 CATEGORY_DEFAULT를 가져야 합니다.
- 모든 웹 인텐트 필터들은 ACTION_VIEW를 가져야 합니다.
- 설치 버전은 무료여야 하며, 업무용 프로필에 답길 수 없고, 키즈앱으로 설계되면 안 됩니다.
- apk signature scheme v2로 사인이 되어있어야 합니다.

### 제약사항
- 푸시 미지원
- 일시적으로 앱의 private 저장소만 사용할 수 있음.
- 웰페이퍼 같은 기기 세팅을 바꾸는 작업은 하면 안 됨.
- 증명되지 않은 소프트웨어나 네이티브 코드, 동적 코드를 실행하면 안 됨.
- 매니페스트에 등록된 브로드캐스트 리시버는 사용할 수 없음.
- 한국에서 지원하지 않음(17.06.01 기준) [참고 링크](https://support.google.com/googleplay/android-developer/answer/7381861#production)


