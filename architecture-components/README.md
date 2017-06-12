# 이곳 문서들은?
### [원문](https://developer.android.com/topic/libraries/architecture/index.html)
- 제멋대로 의역한 문서들입니다.

<br>

# 기타 도움이 되는 링크
### 코드랩
- [라이프사이클 컴포넌트로 라이프사이클 문제 반쯤 해결하기](https://codelabs.developers.google.com/codelabs/android-lifecycles/#0)
	- 기기 회전 시 ViewModel로 데이터 보존하기
	- 이전 예제에서 LiveData를 추가해서 비지니스 로직을 ViewModel쪽으로 완전히 이관하기
	- 액티비티 라이프사이클에 따라 GPS리스너 등록/해제 하기
	- 두 프래그먼트 간에 ViewModel 공유하기
	
- [Room으로 데이터에 영속성을 줘서 라이프사이클 문제 완전 해결하기](https://codelabs.developers.google.com/codelabs/android-persistence/#0)
	- Room을 통해 데이터 캐시를 구현해서 앱 컴포넌트가 종료돼도 데이터를 보존하기
	- ViewModel과 LiveData를 추가해서 비동기로 DB 데이터를 불러오기(without 라이프사이클 문제)
	- TypeConverter를 사용해서 Entity의 POJO 멤버변수와 컬럼 사이의 전환을 커스텀하기

### 샘플 프로젝트
- [샘플 프로젝트](https://github.com/googlesamples/android-architecture-components)

