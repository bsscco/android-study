# 짧은 소개
### 아키텍쳐 컴포넌트란?
- 라이브러리
- 테스트가 쉽고 유지보수가 쉬운 앱을 설계할 수 있도록 도와주는 컴포넌트 모음
- 라이프사이클과 관련된 문제를 해결합니다.

### 아키텍처 컴포넌트 종류
- 라이프사이클 컴포넌트
	- 정의
		- 기기 설정이 바뀌었을 때 오류가 없게 해줍니다. (라이프 사이클 문제 방지)
		- 라이프사이클과 관련된 문제를 처리하지 않아서 발생하는 메모리 누수를 피하게 해줍니다. (라이프 사이클 문제 방지)
		- 데이터를 쉽게 로딩하게 해줍니다. (라이프 사이클 문제 방지)
	- 클래스 목록
		- LiveData(라이프사이클에 맞춰 **데이터를 자동 관리**해주는 클래스)
		- ViewModel(뷰를 위한 **데이터를 담고 있고, 데이터를 핸들링하는** 클래스) 
			- Repository(아키텍쳐 컴포넌트는 아니지만 MVVM에서 M의 역할. **데이터를 연산**하는 클래스)
		- LifecycleObserver(라이프사이클을 관찰하는 인터페이스)
		- LifecycleOwner(라이프사이클을 가지는 인터페이스)
	- 링크
		- [코드랩](https://codelabs.developers.google.com/codelabs/android-lifecycles/#0)
		- [닥스](https://developer.android.com/topic/libraries/architecture/lifecycle.html)
		- [샘플](https://github.com/googlesamples/android-architecture-components)

- Room
	- 정의
		- SQLite ORM 라이브러리
		- 컴파일 타임에 쿼리를 체크해줍니다.
	- 클래스 목록
		- Room
	- 링크
		- [코드랩](https://codelabs.developers.google.com/codelabs/android-persistence/#0)
		- [닥스](https://developer.android.com/topic/libraries/architecture/room.html)
		- [샘플](https://github.com/googlesamples/android-architecture-components)
