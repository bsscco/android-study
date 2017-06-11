# LiveData

### LiveData
- 하는 일
	- LiveData는 LifecycleObserver를 더 추상화한 클래스라고 생각할 수 있습니다.
	- LiveData는 데이터를 가지고 있는 클래스입니다. LiveData는 이 데이터를 관찰할 수 있도록 해줍니다. 그리고 일반적인 Observable과 달리 Lifecycle에 따른 적절한 처리가 들어있습니다.
	- LiveData는 만약 Observer의 라이프사이클 상태가 STARTED 또는 RESUMED라면, Observer가 활성화된 상태에 있다고 간주합니다.
	
- LiveData 만들기
	```java
	public class LocationLiveData extends LiveData<Location> {
			private LocationManager locationManager;

			private SimpleLocationListener listener = new SimpleLocationListener() {
					@Override
					public void onLocationChanged(Location location) {
							setValue(location);
					}
			};

			public LocationLiveData(Context context) {
					locationManager = (LocationManager) context.getSystemService(
									Context.LOCATION_SERVICE);
			}

			@Override
			protected void onActive() {
					locationManager.requestLocationUpdates(LocationManager.GPS_PROVIDER, 0, 0, listener);
			}

			@Override
			protected void onInactive() {
					locationManager.removeUpdates(listener);
			}
	}
	```

	- 위 구현체 안에서는 3가지 중요한 점이 있습니다.
		- onActive()
			- 이 메소드는 LiveData가 활성화된 Observer를 가질 때 호출됩니다.
		- onInactive()
			- 이 메소드는 LiveData가 활성화되지 않은 Observer를 가질 때 호출됩니다.
		- setValue()
			- 이 메소드를 호출하면 LiveData를 갱신되며, Observer에게 변화를 통지합니다.

- LiveData 사용하기		
	```java
	public class MyFragment extends LifecycleFragment {
			public void onActivityCreated (Bundle savedInstanceState) {
					LiveData<Location> myLocationListener = ...;
					Util.checkUserStatus(result -> {
							if (result) {
									myLocationListener.addObserver(this, location -> {
											// update UI
									});
							}
					});
			}
	}
	```
	- addObserver() 메소드의 첫번 째 인자로 LifecycleOwner(액티비티 프래그먼트 등)를 넘기세요. 이로써 Observer가 LifecyclerOwner의 라이프사이클에 얽메이게 됩니다. 그 의미는 아래와 같습니다.
		- 만약 라이프사이클이 활성화(STARTED, RESUMED) 상태가 아니라면, Observer는 데이터가 바뀌어도 그것을 알리지 않습니다.
		- 만약 라이프사이클이 파괴되면, Observer는 스스로를 자동으로 제거합니다.

	- 여러 액티비티, 프래그먼트에서 사용할 수 있습니다. 그러기 위해서 Singleton으로 만들 수 있습니다.
		```java
		public class LocationLiveData extends LiveData<Location> {
				private static LocationLiveData sInstance;
				private LocationManager locationManager;

				@MainThread
				public static LocationLiveData get(Context context) {
						if (sInstance == null) {
								sInstance = new LocationLiveData(context.getApplicationContext());
						}
						return sInstance;
				}

				private SimpleLocationListener listener = new SimpleLocationListener() {
						@Override
						public void onLocationChanged(Location location) {
								setValue(location);
						}
				};

				private LocationLiveData(Context context) {
						locationManager = (LocationManager) context.getSystemService(
										Context.LOCATION_SERVICE);
				}

				@Override
				protected void onActive() {
						locationManager.requestLocationUpdates(LocationManager.GPS_PROVIDER, 0, 0, listener);
				}

				@Override
				protected void onInactive() {
						locationManager.removeUpdates(listener);
				}
		}
		
		public class MyFragment extends LifecycleFragment {
				public void onActivityCreated (Bundle savedInstanceState) {
						Util.checkUserStatus(result -> {
								if (result) {
										LocationLiveData.get(getActivity()).observe(this, location -> {
											 // update UI
										});
								}
						});
			}
		}
		```
	- 이렇게 함으로써 여러 액티비티, 프래그먼트는 MyLocationListener를 관찰하게 될 것입니다. 그리고 LiveData는 오직 옵저버가 활성화 상태일 때만 시스템 서비스에 접근합니다.
	
- LiveData의 이점
	- 메모리 누수가 없어요.
		- Observer들은 LifecycleOwner에 묶이기 때문에 라이프사이클이 파괴되면, Observer는 자동으로 클린업 됩니다.
	- 액티비티가 정지됬을 때 충돌이 없어요.
		- 만약 Observer의 라이프사이클이 비활성화 상태라면, 데이터 변화 이벤트를 받지 않게 됩니다. 
	- 항상 최신 데이터를 유지합니다.
		- 만약 라이프사이클이 다시 시작된다면, 가장 최신의 데이터를 받게 될 거에요.
	- 기기설정의 변경에 적절하게 대응합니다.
		- 만약 화면방향 전환 같은 기기설정 변경으로 액티비티나 프래그먼트가 재실행 된다면, 즉각 가장 마지막 데이터를 받게 됩니다.
	- 자원을 공유합니다.
		- Singleton MyLocationListener를 통해 시스템서비스와의 단 한 번의 연결만 필요합니다.
	- 더 이상 수동적인 라이프사이클 핸들링이 필요하지 않습니다.
		- LifecyclerOwner는 데이터를 관찰하기만 하면 되며, 더 이상 라이프사이클에 따라서 데이터를 관리해줄 필요가 없습니다.

<br>
<br>

### Transformations
- 하는 일
	- 당신은 아마도 LiveData의 값을 Observer로 디스패칭 하기 전에 변형시키고 싶을 수도 있습니다. lifecycle 패키지는 이러한 연산들을 헬퍼 메소드로 담고 있는 Transformations 클래스를 제공합니다.

- Transformation 종류
	- ransformations.map()
		- LiveData의 데이터 위에서 function을 적용합니다. 그리고 새로운 LiveData를 생성해서 반환합니다.
		```java
		LiveData<User> userLiveData = ...;
		LiveData<String> userName = Transformations.map(userLiveData, user -> {
		    user.name + " " + user.lastName
		});
		```
	- Transformations.switchMap()	
		- 아... map이랑 뭔 차이지????
		```java
		private LiveData<User> getUser(String id) {
		  ...;
		}

		LiveData<String> userId = ...;
		LiveData<User> user = Transformations.switchMap(userId, id -> getUser(id) );
		```
		
- 사용 예
	- 사용자의 주소입력에 따라 우편번호를 보여주는 UI가 있다고 가정합시다.
		```java
		class MyViewModel extends ViewModel {
		    private final PostalCodeRepository repository;
		    public MyViewModel(PostalCodeRepository repository) {
		       this.repository = repository;
		    }

		    private LiveData<String> getPostalCode(String address) {
		       // DON'T DO THIS
		       return repository.getPostCode(address);
		    }
		}
		```
	- 이 UI에선 ```getPostalCode()```를 호출할 때마다 이전 LiveData를 제거하고, 새로운 LiveData를 다시 등록하는 작업이 필요할 겁니다. 
	- 대신에 아래와 같이 해봅시다.
		```java
		class MyViewModel extends ViewModel {
		    private final PostalCodeRepository repository;
		    private final MutableLiveData<String> addressInput = new MutableLiveData();
		    public final LiveData<String> postalCode =
			    Transformations.switchMap(addressInput, (address) -> {
				return repository.getPostCode(address);
			     });

		  public MyViewModel(PostalCodeRepository repository) {
		      this.repository = repository
		  }

		  private void setInput(String address) {
		      addressInput.setValue(address);
		  }
		}
		```
	- ```postalCode```가 ```public final```로 선언되었음에 집중해봅시다. 이것은 ```postalCode```가 결코 바뀌지 않을 거라는 뜻입니다. 이것은 ```addressInput```이 바뀔 때마다 ```addressInput```으로 동작하는 Transformation으로 정의됐습니다.

<br>
<br>

### 새로운 Transformations 생성하기
- 다른 Trasformation이 필요할 땐 MediatorLiveData를 사용하면 됩니다. MediatorLiveData는 LiveData를 듣는 특별한 클래스입니다.
