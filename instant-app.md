# 링크
- [공식 가이드](https://developer.android.com/topic/instant-apps/index.html)
- [공식 코드랩](https://codelabs.developers.google.com/codelabs/android-instant-apps/index.html)

# 정의
### 인스턴트앱이란?
- 7MB 짜리 앱이 있다.
- 이 앱은 상품 목록 보기 기능(A)과 상품 정보 기능(B)을 가지고 있다.
- A가 4MB, B가 3MB를 차지한다.
- 이 앱을 인스턴트앱으로 만들었다.
- 이 앱을 설치하지 않은 유저는 A에 대한 다운로드 URL을 통해 A의 코드만 내려받을 수 있다. 그리고 A를 사용한다.
- 앱을 설치하지 않고 기능 일부만 내려받았을 경우엔 기능을 종료함과 동시에 관련 코드가 흔적없이 사라진다.
- 우리는 인스턴트앱을 통해 일부 핵심 기능만 사용시켜주고, 앱 설치를 유도할 수 있다.
- 주의사항: 인스턴트앱을 만들 때 내려 받을 수 있는 하나의 기능의 크기는 최대 4MB이다.
- 인스턴트앱은 마시멜로우 버전부터 지원한다.
