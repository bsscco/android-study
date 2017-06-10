# 라이프사이클 핸들링하기

### android.arch.lifecycle 패키지
- 이 패키지는 우리가 lifecycle-aware 컴포넌트를 만들 수 있도록 해주는 클래스와 인터페이스를 제공합니다.
- lifecycle-arware 컴포넌트는 현재 LifecycleOwner(액티비티, 프래그먼트 등)의 라이프사이클에 맞추어 자동으로 적절히 동작합니다.
- 아래 코드를 봅시다.
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
- android.arch.lifecycle 패키지는 이러한 문제들을 탄력적이고 독립적으로 해결하는 것을 돕습니다.

### Lifecycle
- Lifecycle은 다른 객체들이 LifecycleOwner(액티비티, 프래그먼트 등)의 라이프사이클 상태를 관찰할 수 있도록 돕는 클래스입니다.
- Lifecycle은 라이프사이클 상태를 관찰하기 위해 두 가지 enumaration을 사용합니다. 
	- Event
	 	- 라이프사이클 이벤트들은 프레임워크와 Lifecycle 클래스로부터 디스패치 됩니다. 이러한 이벤트들은 액티비티나 프래그먼트의 이벤트 콜백과 매치됩니다.
	- State
		- 앱 컴포넌트의 현재 상태는 Lifecycle 객체에 의해 추적됩니다.
	- ![lifecycle-states.png](https://developer.android.com/images/topic/libraries/architecture/lifecycle-states.png)

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

### LifecycleOwner
- LifecycleOwner는 라이프사이클을 가지는 싱글 메소드 인터페이스입니다. getLifecycle()이라는 메소드만 가집니다.
- 알아두기
	- 아키텍쳐 컴포넌트들이 안정화될 때가지 Fragment와 AppCompatActivity 대신에 LifecycleFragment와 LifecycleActivity를 사용해주세요. 나중에 LifecycleFragment와 LifecycleActivity는 deprecated가 될 예정입니다.
- 

### Best practices

### 추가적인 내용
