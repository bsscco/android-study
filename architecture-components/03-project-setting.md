# 아키텍쳐 컴포넌트를 프로젝트에 추가하기
### 구글 메이븐 저장소 추가하기
- 프로젝트 build.gragle에 추가합니다.
```
allprojects {
    repositories {
        jcenter()
        maven { url 'https://maven.google.com' }
    }
}
```

### 아키텍쳐 컴포넌트 추가하기
- 앱 build.gradle에 추가합니다.
- Lifecycle, LiveData, ViewModel
	- compile "android.arch.lifecycle:runtime:1.0.0-alpha1"
	- compile "android.arch.lifecycle:extensions:1.0.0-alpha1"
	- annotationProcessor "android.arch.lifecycle:compiler:1.0.0-alpha1"
- Room
	- compile "android.arch.persistence.room:runtime:1.0.0-alpha1"
	- annotationProcessor "android.arch.persistence.room:compiler:1.0.0-alpha1"
	- Room 마이그레이션 테스트
		- testCompile "android.arch.persistence.room:testing:1.0.0-alpha1"
	- Room RxJava 지원
		- compile "android.arch.persistence.room:rxjava2:1.0.0-alpha1"
