# 라이프사이클 핸들링하기

### android.arch.lifecycle 패키지
- 하는 일
	- 이 패키지는 우리가 lifecycle-aware 컴포넌트를 만들 수 있도록 해주는 클래스와 인터페이스를 제공합니다.
	- lifecycle-arware 컴포넌트는 현재 LifecycleOwner(액티비티, 프래그먼트 등)의 라이프사이클에 맞추어 자동으로 적절히 동작합니다.

- 라이프사이클과 관련된 문제
	```java
	class MyLocationListener {
	    public MyLocationListener(Context context, Callback callback) {
		// ...
	    }

	    void start() {
		// connect to system location service
	    }

	    void stop() {
		// disconnect from system location service
	    }
	}

	class MyActivity extends AppCompatActivity {
	    private MyLocationListener myLocationListener;

	    public void onCreate(...) {
		myLocationListener = new MyLocationListener(this, (location) -> {
		    // update UI
		});
	  }

	    public void onStart() {
		super.onStart();
		myLocationListener.start();
	    }

	    public void onStop() {
		super.onStop();
		myLocationListener.stop();
	    }
	}
	```

- 위 코드는 앱에서 실제로 잘 동작하지만, onStop() 없이 onStart()가 다시 호출되는 상황이 발생할 수 있습니다. 그래서 아래와 같은 코드로 수정합니다.
	```java
	class MyActivity extends AppCompatActivity {
	    private MyLocationListener myLocationListener;

	    public void onCreate(...) {
		myLocationListener = new MyLocationListener(this, location -> {
		    // update UI
		});
	    }

	    public void onStart() {
		super.onStart();
		Util.checkUserStatus(result -> {
		    // what if this callback is invoked AFTER activity is stopped?
		    if (result) {
			myLocationListener.start();
		    }
		});
	    }

	    public void onStop() {
		super.onStop();
		myLocationListener.stop();
	    }
	}
	```

- android.arch.lifecycle 패키지는 이러한 라이프사이클과 관련된 문제들을 탄력적이고 독립적으로 해결할 수 있도록 돕습니다.

<br>
<br>

### Lifecycle과 LifecycleObserver
- 하는 일
	- Lifecycle과 LifecycleObserver 클래스는 다른 객체들이 LifecycleOwner(액티비티, 프래그먼트 등)의 라이프사이클 상태를 관찰할 수 있도록 돕는 클래스입니다.
	- 라이프사이클 상태를 관찰하기 위해 두 가지 enumaration을 사용합니다. 
		- Event
			- 라이프사이클 이벤트들은 프레임워크와 Lifecycle 클래스로부터 디스패치 됩니다. 이러한 이벤트들은 액티비티나 프래그먼트의 이벤트 콜백과 매치됩니다.
		- State
			- 앱 컴포넌트의 현재 상태는 Lifecycle 객체에 의해 추적됩니다.
		- ![lifecycle-states.png](https://developer.android.com/images/topic/libraries/architecture/lifecycle-states.png)

- LifecycleObserver 만들기
	- 아래 클래스는 어노테이션을 통해 라이프사이클 상태를 관찰할 수 있습니다.
		```java
		public class MyObserver implements LifecycleObserver {
		    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
		    public void onResume() {
		    }

		    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
		    public void onPause() {
		    }
		}
		aLifecycleOwner.getLifecycle().addObserver(new MyObserver()); // 여기가 중요.
		```	

<br>
<br>

### LifecycleOwner
- 하는 일
	- LifecycleOwner는 라이프사이클을 가지는 싱글 메소드 인터페이스입니다. getLifecycle()이라는 메소드만 가집니다.

- 알아두기
	- 아키텍쳐 컴포넌트들이 안정화될 때가지 Fragment와 AppCompatActivity 대신에 LifecycleFragment와 LifecycleActivity를 사용해주세요. 
	- 나중에 LifecycleFragment와 LifecycleActivity는 deprecated가 될 예정입니다.

- LifecycleObserver 사용하기 
	- 위의 예로부터 우리는 LifecycleObserver로써 MyLocationListener를 만들 수 있습니다. 그리고 LifecycleOwner의 onCreate()에서 그것을 초기화할 수 있습니다. 이것은 LifecycleObserver가 스스로 필요할 때 클린업 되도록 할 수 있게 만듭니다. 
	- 아래 코드를 봅시다.
		```java
		class MyLocationListener implements LifecycleObserver {
		    private boolean enabled = false;
		    public MyLocationListener(Context context, Lifecycle lifecycle, Callback callback) {
		       ...
		    }

		    @OnLifecycleEvent(Lifecycle.Event.ON_START)
		    void start() {
			if (enabled) {
			   // connect
			}
		    }

		    public void enable() {
			enabled = true;
			if (lifecycle.getState().isAtLeast(STARTED)) {
			    // connect if not connected
			}
		    }

		    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
		    void stop() {
			// disconnect if connected
		    }
		}

		class MyActivity extends LifecycleActivity {
		    private MyLocationListener myLocationListener;

		    public void onCreate(...) {
			myLocationListener = new MyLocationListener(this, getLifecycle(), location -> {
			    // update UI
			});
			Util.checkUserStatus(result -> {
			    if (result) {
				myLocationListener.enable();
			    }
			});
		  }
		}
		```
- 좋은 사용 예
	- 좋은 사용 예는 LifecycleOwner의 라이프사이클 상태가 좋지 않은 경우일 때 callback 호출을 피하는 것입니다. 예를 들어 액티비티가 상태를 저장한 후에 callback이 프래그먼트 트랜잭션을 시작하는 경우가 좋지 않은 경우입니다. Lifecycle 객체를 통해 현재 라이프사이클 상태를 체크할 수 있습니다. ```if (lifecycle.getState().isAtLeast(STARTED))```
	- 이제 MyLocationListener 클래스는 액티비티의 관리 없이 스스로 초기화할 수 있고, 필요할 때 클린업 할 수 있습니다. 따라서 MyLocationListener는 다른 액티비티나 클래스에서도 독립적으로 재활용할 수 있습니다.
	- 또 다른 좋은 예는 LiveData과 ViewModel입니다. 이 클래스들은 lifecycle-ware 컴포넌트입니다.

<br>
<br>

### 라이프사이클 컴포넌트 Best practices
- ViewController(액티비티, 프래그먼트 등)를 가능한 한 lean하게 만듭니다. ViewController가 자신의 데이터를 가지게 하지 마세요. 대신에 ViewModel과 LiveData를 써서 데이터를 처리하게 하세요.
	- VeiwController는 데이터를 가지지 않는다.

- ViewController가 변경된 데이터로 View를 갱신하는 책임만 지게 하세요. 유저 액션은 ViewModel에게 넘기세요.
	- ViewController는 UI 렌더링만 책임진다.

- 데이터 로직은 ViewModel의 책임입니다. ViewModel은 ViewController와 다른 요소들 간의 연결자로서 동작합니다. 단, ViewModel은 데이터 fetching의 책임은 지지 않습니다. ViewModel은 적절한 다른 컴포넌트로부터 fetching한 데이터를 ViewController로 보내줍니다.
	- ViewModel은 데이터 로직은 갖지만, 데이터 fetching은 다른 컴포넌트에게 맡긴다.

- ViewController와 View 사이에 클린코드를 위해 데이터 바인딩 라이브러리를 사용하세요. 이것은 ViewController에 써야 하는 UI 갱신 코드들을 최소화하는 데 도움을 줍니다.
	- 데이터 바인딩은 데이터 바인딩 라이브러리를 사용한다.

- UI가 복잡하다면, Presenter를 만드는 것을 추천하지만, 과도한 작업일 수 있습니다. 하지만 UI를 쉽게 테스트하게 해줍니다.
	- UI가 복잡하면 Presenter를 만들자.

- 결코 View 또는 ViewController의 참조를 ViewModel이 가지게 하지 마세요. 기기설정 변경 같은 상황에서 액티비티가 메모리 해제할 수 없게 될 수 있습니다.
	- View, ViewController의 참조를 ViewModel이 가지게 하지 말자.

<br>
<br>

### 추가적인 내용
- 커스텀 액티비티, 프래그먼트에서 LifecycleOwner 구현하기
	```java
	public class MyFragment extends Fragment implements LifecycleRegistryOwner {
	    LifecycleRegistry lifecycleRegistry = new LifecycleRegistry(this);

	    @Override
	    public LifecycleRegistry getLifecycle() {
		return lifecycleRegistry;
	    }
	}
	```
