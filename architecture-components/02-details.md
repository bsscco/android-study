#architecture

# 덜 짧은 소개
### 개발자들이 직면한 공통의 문제
- 안드로이드 앱은 독립적인 다양한 컴포넌트(Activity, Fragment 등)로 구성되어있습니다. 각 컴포넌트는 앱의 entry point가 될 수 있기 때문에 앱은 여러 개의 entry point를 가질 수 있습니다. 앱이 어떤 entry point에서 시작될지 모르기 때문에 **컴포넌트 간 데이터를 독립시켜야 합니다.**
- 또한 안드로이드 앱은 메모리 관리가 필요할 때 OS에 의해 강제종료 되었다가 필요할 때 재실행 될 수 있습니다. 앱이 강제종료 되었을 때 데이터를 보존시켜야 하기 때문에 **데이터를 컴포넌트에 저장해서는 안 됩니다.** 컴포넌트는 앱이 종료될 때 함께 메모리에서 해제됩니다.
- 'multi entry point 문제'와 'OS의 앱 강제종료-재실행 문제'는 **앱 컴포넌트의 특이한 라이프사이클에 의해 발생한 문제들입니다.**


### 이 문제들을 해결하기 위한 아키텍쳐 원칙
- 가장 중요한 원칙은 **관심사의 분리**를 하는 것입니다. 
	- 우리는 모든 코드를 액티비티나 프래그먼트에 작성하는 실수를 범하곤 합니다. 이것은 라이프사이클과 관련있는 여러 문제들을 야기시킵니다. 앱 컴포넌트에서 UI 핸들링이나 OS와의 인터렉션을 독립시킴으로써 관심사의 분리를 실현할 수 있습니다. **관심사의 분리를 통해 라이프사이클과 관련된 문제들을 피할 수 있습니다.**

- 그 다음 중요한 원칙은 **데이터를 영속적으로 관리**하는 것입니다. 
	- OS에 의해 앱이 강제종료 됐다가 다시 살아나는 등의 라이프사이클 문제가 발생해도 유저는 앱이 상태를 잃어버리기 원치 않습니다. **영속적으로 구현된 모델(MVVM에서 M)은 라이프사이클로부터 독립적으로 앱 데이터를 핸들링하는 책임을 지기 때문에 이러한 문제의 해결책이 됩니다.**
	
	
### 추천하는 앱 아키텍쳐
UI Building(MVVM)

- (1) **View**와 **ViewController(LifecycleOwner)** 준비
	- 유저 프로필을 보여주는 화면(UserProfileFragment.java, user_profile_layout.xml)이 있다고 가정합시다. UserProfileFragment는 LifecycleOwner를 implementing 한 클래스입니다.

- (2) 데이터 **Model** 구상
	- UI를 운용하기 위해서 데이터 모델을 두 가지 데이터 요소로 구성합니다.
		- User ID
			- 프래그먼트 인자를 사용하는 프래그먼트에 정보를 넘길 때 최선의 방법이 됩니다.
			- User ID로 유저 데이터를 식별할 수 있습니다.
		- User Object
			- 유저 데이터

- (3) **ViewModel** 준비
	- 우리는 UserProfileViewModel를 생성할 겁니다.
	- **ViewModel**
		- UI를 기술하는 데이터를 제공합니다.
		- 비지니스 파트 데이터를 핸들링합니다.
		- View를 알지 못하며 기기 설정 변경에 영향을 받지 않습니다.

- (4) **View**, **ViewController**, **ViewModel** 준비 결과
	- 이제 우리는 3개의 파일을 가지게 됐습니다.
		- **View** : user_profile_layout.xml
			- UI가 정의된 파일
		- **ViewModel** : UserProfileViewModel.java
			- UI를 위한 데이터를 준비하는 클래스
				```java
				public class UserProfileViewModel extends ViewModel {
				    private String userId;
				    private User user;

				    public void init(String userId) {
					this.userId = userId;
				    }
				    public User getUser() {
					return user;
				    }
				}
				```
		- **ViewController** : UserProfileFragment.java
			- ViewModel에 있는 데이터를 보여주고 유저 인터렉션에 반응하는 UI 컨트롤러
				```java
				public class UserProfileFragment extends LifecycleFragment {
				    private static final String UID_KEY = "uid";
				    private UserProfileViewModel viewModel;

				    @Override
				    public void onActivityCreated(@Nullable Bundle savedInstanceState) {
					super.onActivityCreated(savedInstanceState);
					String userId = getArguments().getString(UID_KEY);
					viewModel = ViewModelProviders.of(this).get(UserProfileViewModel.class); // 이 부분에서 UserProfileViewModel에 포함된 LiveData들이 UserProfileFragment의 라이프사이클을 알 수 있도록 연결해줍니다.
					viewModel.init(userId);
				    }

				    @Override
				    public View onCreateView(LayoutInflater inflater,
						@Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
					return inflater.inflate(R.layout.user_profile, container, false);
				    }
				}
				```
	- UserProfileFragment(ViewController)는 user_profile_layout.xml(View)을 inflating 하여 가지고 있고, UserProfileViewModel(ViewModel) 객체를 생성하여 가지고 있습니다.

- (5) 데이터의 갱신을 **View**에게 알리려면?
	- 이제 이 3가지 어떻게 연결할까요? ViewModel의 유저 필드가 세팅됐을 때 우리는 UI에게 유저 필드의 정보를 알려야 합니다. **LiveData** 클래스를 통해 알릴 수 있습니다.
	- **LiveData**
		- 관찰될 수 있는 데이터 홀더입니다. 이 기능을 사용하면 우리의 앱 컴포넌트가 LiveData 객체의 변화를 명시적이고 의존적인 경로 없이 관찰 할 수 있게 됩니다.
		- 앱 컴포넌트들의 라이프사이클을 이해하며, 앱이 메모리를 더 사용하지 않도록 메모리 누수를 방지합니다.

- (6) 데이터 갱신에 대한 알림기능을 **LiveData**를 사용해 추가하기
	- 이제 우리는 UserProfileViewModel 안에 있는 유저 필드를 ```LiveData<User>```로 대체할 겁니다. 그러면 프래그먼트는 데이터가 갱신됐을 때 알림 받을 수 있습니다. **LiveData**가 뛰어난 이유는, 그것이 라이프사이클을 잘 알고 있기 때문에 참조들을 clean up 시키는 작업을 라이프사이클에 따라서 자동으로 해주기 때문입니다.
	```java
	public class UserProfileViewModel extends ViewModel {
	    ...
	    // private User user;
	    private LiveData<User> user;
	    public LiveData<User> getUser() {
		return user;
	    }
	}
	```

- (7) 데이터 갱신 시 알림받을 관찰자를 **LiveData**에 등록하기
	- 이제 우리는 데이터를 관찰해서 UI를 갱신할 수 있도록 프래그먼트를 수정할 겁니다.
	```java
	@Override
	public void onActivityCreated(@Nullable Bundle savedInstanceState) {
	    super.onActivityCreated(savedInstanceState);
	    viewModel.getUser().observe(this, user -> {
	      // update UI
	    });
	}
	```
	- LiveData는 자동으로 프래그먼트의 ```onStart()```에서 데이터를 받고, ```onStop()```에선 데이터를 받지 않고, ```onDestroy()```에서는 관찰자를 제거합니다. 그밖에 기기 설정이 바뀌었을 때도 우리가 특별히 할 일은 없습니다. 기기 설정이 바뀔 때 LiveData는 자동으로 데이터를 저장하고, 새로운 ViewModel 인스턴스에서 데이터를 불러옵니다. 이것이 LiveData가 View의 참조를 직접적으로 가지지 않아야 하는 이유입니다.

데이터 fetching

- (8) 데이터 fetching API 준비하기
	- ViewModel이 어떻게 유저 데이터를 fetching 할까요? 우리의 백엔드가 REST API를 제공한다고 가정해서 우리는 Retrofit 라이브러리를 사용할 겁니다.
	```java
	public interface Webservice {
		/**
		 * @GET declares an HTTP GET request
		 * @Path("user") annotation on the userId parameter marks it as a
		 * replacement for the {user} placeholder in the @GET path
		 */
		@GET("/users/{user}")
		Call<User> getUser(@Path("user") String userId);
	}
	```
	
- (9) **ViewModel**에서 직접 fetching 하면 안 되요. **Model**에서 해야 되요.
	- **ViewModel**은 Webservice에 직접 접근해서 데이터를 fetching할 수 있고, 데이터를 User 객체에 할당할 수 있습니다. 이것은 **ViewModel**에게 너무 많은 책임감을 쥐어줍니다. 이것은 곧 관심사의 분리라는 원칙에 어긋나게 됩니다. 추가적으로 **ViewModel**의 범위는 프래그먼트나 액티비티의 라이프사이클에 얽메이게 되는데요, 그렇기 때문에 라이프사이클이 종료될 때 데이터도 함께 잃어버리게 됩니다. 이것은 유저 경험에 좋지 않습니다. 대신에 **ViewModel**은 **Repository**에게 이 작업을 위임합니다.
	- **Repository**
		- 데이터 연산의 책임을 집니다.
		- 깔끔한 REST API를 제공합니다.
		- Repository는 어디서 데이터를 가져와야 할지, 데이터가 갱신될 때 어느 API를 호출해야 할지 압니다.
		- 우리는 Repository를 캐싱, 영속적 모델, 웹 서비스 등의 사이에서의 중재자로 여길 수 있습니다.

- (10) 데이터 fetching 기능을 가지는 **Model(Repository)** 준비하기
	```java
	public class UserRepository {
	    private Webservice webservice;
	    // ...
	    public LiveData<User> getUser(int userId) {
		// This is not an optimal implementation, we'll fix it below
		final MutableLiveData<User> data = new MutableLiveData<>();
		webservice.getUser(userId).enqueue(new Callback<User>() {
		    @Override
		    public void onResponse(Call<User> call, Response<User> response) {
			// error case is left out for brevity
			data.setValue(response.body());
		    }
		});
		return data;
	    }
	}
	```
	- **Repository**는 쓸모 없어 보일지 모르지만 중요한 목적이 있습니다. **Repository**는 앱으로부터 데이터 소스를 추상화 합니다. 이제 **ViewModel**은 데이터가 Webservice에 의해서 fetching 되는지 알지 못합니다. 이것은 우리가 필요할 때 다른 구현체로 교체할 수 있다는 것을 의미합니다.

- (11) **Repository**들 사이의 의존성 관리하기
	- UserRepository는 자신의 일을 하기 위해서 Webservice 인스턴스가 필요합니다. 만약 여러 개의 **Repository** 클래스들이 Webservice 인스턴스를 생성해서 가지고 있다면 이는 리소스가 매우 무거워지는 일입니다. 이 문제를 해결하기 위한 2가지 패턴이 있습니다.
		- Dependency Injection
			- DI는 생성자 없이 의존성을 정의할 수 있게 해줍니다. Google의 Dagger2를 추천합니다.
		- Service Locator
			- Service Locator는 생성자 대신에 의존성을 obtain할 수 있는 등록자를 제공합니다. 만약 DI에 친숙하지 않다면 상대적으로 더 쉬운 Dagger2를 추천합니다.
	- 이러한 패턴들은 중복코드나 복잡성 없이 의존성을 관리할 수 있게 해줍니다. 또한 이들은 테스팅을 위해 구현체를 교체할 수 있게도 해줍니다.

- (12) **ViewModel**과 **Model** 연결하기
	- 이제 UserProfileViewModel을 수정할 겁니다.
	```java
	public class UserProfileViewModel extends ViewModel {
	    private LiveData<User> user;
	    private UserRepository userRepo;

	    @Inject // UserRepository parameter is provided by Dagger 2
	    public UserProfileViewModel(UserRepository userRepo) {
		this.userRepo = userRepo;
	    }

	    public void init(String userId) {
		if (this.user != null) {
		    // ViewModel is created per Fragment so
		    // we know the userId won't change
		    return;
		}
		user = userRepo.getUser(userId);
	    }

	    public LiveData<User> getUser() {
		return this.user;
	    }
	}

	```
	
데이터 캐싱

- (13) 인메모리에 데이터 캐싱하기
	- 만약 유저가 UserProfileFragment를 떠났다가 다시 들어오면 유저는 re-fetching을 기다려야 합니다. 이것은 좋은 유저경험이 아닙니다. 이것을 다루기 위해 새로운 데이터 소스를 UserRepository에 추가할 겁니다. 이 데이터 소스는 캐시용 데이터 소스입니다.
	```java
	@Singleton  // informs Dagger that this class should be constructed once
	public class UserRepository {
	    private Webservice webservice;
	    // simple in memory cache, details omitted for brevity
	    private UserCache userCache;
	    public LiveData<User> getUser(String userId) {
		LiveData<User> cached = userCache.get(userId);
		if (cached != null) {
		    return cached;
		}

		final MutableLiveData<User> data = new MutableLiveData<>();
		userCache.put(userId, data);
		// this is still suboptimal but better than before.
		// a complete implementation must also handle the error cases.
		webservice.getUser(userId).enqueue(new Callback<User>() {
		    @Override
		    public void onResponse(Call<User> call, Response<User> response) {
			data.setValue(response.body());
		    }
		});
		return data;
	    }
	}

	```
	
- (14) 데이터를 영속적으로 보존하려면?
	- 만약 유저가 오랜 시간 앱을 떠났다가 돌아오거나, OS가 앱을 강제로 종료시켰다가 재실행 시키는 상황에선 무슨 일이 벌어질까요? 데이터를 re-fetching 해야 합니다. 이것은 caching을 통해 간단히 해결할 수 있습니다. 그러나 이 해결법은 또 다른 문제를 야기합니다. 다른 시간대에 유저 데이터를 요청했을 때 데이터들이 서로 다르다면 유저 경험에 매우 좋지 않습니다. 같은 요청의 데이터들이 일관성을 가질 수 없는 문제는 영속적인 모델을 사용함으로써 해결할 수 있습니다. 이것의 구현체가 **Room** 라이브러리입니다.
	- **Room**
		- ORM 라이브러리입니다. 보일러 플레이트 코드를 최소화합니다.
		- 컴파일타임에 쿼리가 잘못됐는지 체크합니다.
		- 이것은 그밖에 DB 데이터가 바뀌었을 때를 관찰해서 LiveData에게 알립니다.
		- 추가적으로 백그라운드 스레드에서만 동작시켜야 합니다.

- (15) 데이터 영속 보존을 위해 **Room** 준비하기
	- **Room**을 사용하기 위해서는 로컬 스키마를 정의해야 합니다. 먼저 User 클래스에 @Entity를 마크합니다.
	```java
	@Entity
	class User {
	  @PrimaryKey
	  private int id;
	  private String name;
	  private String lastName;
	  // getters and setters for fields
	}
	```
	- 그리고 RoomDatabase를 상속해서 데이터베이스 클래스를 만듭니다.
	```java
	@Database(entities = {User.class}, version = 1)
	public abstract class MyDatabase extends RoomDatabase {
	}
	```

- (16) 데이터 저장 API를 위한 DAO 준비
	- 이제 우리는 유저 데이터를 DB에 삽입하기 위한 방법이 필요합니다. DAO를 만들 겁니다.
	```java
	@Dao
	public interface UserDao {
	    @Insert(onConflict = REPLACE)
	    void save(User user);
	    @Query("SELECT * FROM user WHERE id = :userId")
	    LiveData<User> load(String userId);
	}
	```

- (17) **Room**과 DAO 연결하기
	- 그리고 DB로부터 DAO를 참조합니다.
	```java
	@Database(entities = {User.class}, version = 1)
	public abstract class MyDatabase extends RoomDatabase {
	    public abstract UserDao userDao();
	}
	```
	- MyDatabase는 추상화 클래스입니다. Room은 자동으로 구현체를 제공합니다. 자세한 내용은 닥스를 참고하세요.

- (18) **Repository**와 **Room** 연결해서 데이터 연속성 완성하기
	- 이제 우리는 **Room**의 data source와 협동하기 위해 UserRepository를 수정할 겁니다.
	```java
	@Singleton
	public class UserRepository {
	    private final Webservice webservice;
	    private final UserDao userDao;
	    private final Executor executor;

	    @Inject
	    public UserRepository(Webservice webservice, UserDao userDao, Executor executor) {
		this.webservice = webservice;
		this.userDao = userDao;
		this.executor = executor;
	    }

	    public LiveData<User> getUser(String userId) {
		refreshUser(userId); // 없으면 서버에서 가져와서 DB에 저장한다.
		// return a LiveData directly from the database.
		return userDao.load(userId); // DB에서 불러온다.
	    }

	    private void refreshUser(final String userId) {
		executor.execute(() -> {
		    // running in a background thread
		    // check if user was fetched recently
		    boolean userExists = userDao.hasUser(FRESH_TIMEOUT);
		    if (!userExists) {
			// refresh the data
			Response response = webservice.getUser(userId).execute();
			// TODO check for error etc.
			// Update the database.The LiveData will automatically refresh so
			// we don't need to do anything else here besides updating the database
			userDao.save(response.body());
		    }
		});
	    }
	}
	```
	- UserRepository로부터 오는 데이터의 장소를 바꾸어도 우리는 UserProfileFragment나 UserProfileViewModel을 수정할 필요가 없습니다. 그 이유는 추상화를 통한 유연성을 제공하기 때문입니다. 이것이 또한 놀라운 이유는 우리가 UserProfileViewModel을 테스트하기 위해 fake UserRepository를 제공할 수도 있기 때문입니다.

- 클래스 관계도
	- ![class-relations.png](https://github.com/bsscco/ohs-reproject/blob/master/android/blob/class-relations.png)

- 데이터 셋팅의 흐름
	- UserProfileFragment는 ```onViewCreated()```에서 멤버변수 UserProfileViewModel의 ```init()``` 호출 ->
		- UserProfileViewModel은 싱글톤 UserRepository의 ```getUser()``` 호출 ->
			- UserRepository는 멤버변수 UserDao에게 User데이터를 물어서 User데이터가 없으면 멤버변수 Webservice를 통해 서버로부터 User데이터를 가져와서 UserDao를 통해 User데이터를 저장  
			- UserRepository는 멤버변수 UserDao를 통해 User데이터를 불러와서 반환 ->
		- UserProfileViewModel은 반환받은 User데이터를 멤버변수 ```LiveData<User>```에 담기 -> 
	- UserProfileFragment는 멤버변수 UserProfileViewModel의 ```getUser()```로부터 ```LiveData<User>```를 받아 ```observe()```를 호출

- 데이터 갱신의 흐름
	- ```LiveData<User>```가 갱신됨 ->
		- UserProfileFragment는 ```LiveData<User>```의 ```observe()```를 호출하면서 넣었던 콜백을 통해 갱신됨을 통보받고 핸들링

테스팅

- 우리는 테스팅을 분리시키는 것의 이점을 언급해왔습니다. 각 코드모듈마다 어떻게 테스트할 수 있을까요?
	- UI, 인터렉션
		- [Android UI Instrumentation test](https://developer.android.com/training/testing/unit-testing/instrumented-unit-tests.html) 참고, [Espresso](https://developer.android.com/training/testing/ui-testing/espresso-testing.html)를 생성하는 것입니다. fragment는 오직 ViewModel과 이야기하기 때문에 테스트가 편리합니다.
		- Fragment code + Espresso code
	- ViewModel
		- [JUnit test](https://developer.android.com/training/testing/unit-testing/local-unit-tests.html)로 테스트될 수 있습니다. 우리는 오직 UserRepository의 mock만 필요합니다.
		- ViewModel code + UserRespository mock code
	- UserRepository
		- JUnit test로 테스트할 수 있습니다. Webservice mock과 dao mock이 필요합니다. Webserivce와 dao 둘 다 인터페이스이기 때문에 더 복잡한 테스트 케이스를 위한 fake 구현체를 만들기 쉽습니다.
		- UserRepository code + Webservice mock + dao mock
	- UserDao
		- instrumentation test를 추천합니다. 어떤 UI도 필요하지 않습니다. 각 테스트를 위해 어떤 사이드 이펙트도 없는 in-memory db를 만들어 사용할 수도 있습니다.
		- UserDao code + instrumentation test code
	- Webservice
		- fake 로컬 서버를 만들 수 있도록 돕는 [MockWebServer](https://github.com/square/okhttp/tree/master/mockwebserver) 같은 라이브러리들을 이용하는 것이 좋습니다.
		- Webservice code + MockWebServer code
	- 인공적인 흐름 테스트 하기
		- 아키텍처 컴포넌트는 백그라운드 스레드를 제어하기 위해 maven artifact(android.arch.core:core-testing)를 제공합니다. 두 가지 JUnit rule이 있습니다.
			- InstantTaskExecuteRule : 아키텍처 컴포넌트가 즉각 백그라운드 작업을 실행할 수 있게 하는 룰입니다.
			- CountingTaskExecuteRule : instrumentation test를 위해서 아키텍처 컴포넌트의 백그라운드 작업을 기다리게 하기 위해 사용되거나 리소스를 대기하는 espresso를 연결하기 위해 사용될 수 있습니다.

최종 아키텍쳐
![final-architecture.png](https://developer.android.com/topic/libraries/architecture/images/final-architecture.png)
- Activity/Fragment : ViewModel = 1 : 1
- ViewModel has many LiveDatas
- ViewModel references singleton Repository


### 원칙으로 돌아와서, 원칙에 대한 가이드
- enty point들(앱 컴포넌트)은 데이터를 가지고 있어서는 안 됩니다. 각 컴포넌트의 라이프사이클 문제 때문입니다. 
- 각 모듈 간 책임지는 경계를 칼 같이 나누세요. 어떤 모듈에 관련없는 기능들이 모여 있으면 테스트가 어렵고 유지보수가 곤란해지기 때문입니다.
- 각 모듈을 가능한한 작게 만드세요. 어떤 모듈에 관련없는 기능들이 모여 있으면 테스트가 어렵고 유지보수가 곤란해지기 때문입니다.
- 각 모듈을 정의할 때 어떻게 테스터블하게 정의할지 고민하세요. mock을 만들어 테스트하기 쉽기 때문입니다.
- 중복 코드를 다시 작성하는 데에 시간을 뺏기지 말고 아키텍처 컴포넌트나 기타 다른 라이브러리를 통해 정신적 에너지를 다른 창의적인 영역에 사용하세요.
- 데이터를 되도록 많이 영속적으로 보관하세요. 기기를 껐다 켰을 때 사용경험을 유지시킬 수 있고, 연결 속도가 빨라집니다.
- 데이터 저장소는 항상 the single source of truth 원칙을 지켜야 합니다. 다른 화면에서의 데이터 싱크를 맞추고, 여러 곳에서의 불필요한 새로고침 작업을 없애기 위함입니다.


### 추가적인 내용
- 우리는 데이터와 네트워크 상태를 캡슐화하기 위해 Resource 클래스를 사용할 겁니다.
```java
//a generic class that describes a data with a status
public class Resource<T> {
    @NonNull public final Status status;
    @Nullable public final T data;
    @Nullable public final String message;
    private Resource(@NonNull Status status, @Nullable T data, @Nullable String message) {
        this.status = status;
        this.data = data;
        this.message = message;
    }

    public static <T> Resource<T> success(@NonNull T data) {
        return new Resource<>(SUCCESS, data, null);
    }

    public static <T> Resource<T> error(String msg, @Nullable T data) {
        return new Resource<>(ERROR, data, msg);
    }

    public static <T> Resource<T> loading(@Nullable T data) {
        return new Resource<>(LOADING, data, null);
    }
}
```
