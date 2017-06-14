#architecture

# ViewModel
### ViewModel
- 하는 일
	- UI와 관련된 데이터를 기기설정 변경과 같은 상황에서도 적절하게 저장하고 관리하기 위해 설계된 클래스입니다.

- 문제
	- 액티비티가 재실행 될 때는 데이터를 잃어버리게 되는데, ```onSaveInstanceState()```를 통해 저장할 수 있습니다. 그러나 이 접근법은 저장할 데이터가 적을 때만 유용하며, 데이터가 많을 때는 적절하지 않습니다.
	- 또 다른 문제는, ViewController가 자주 비동기 호출이 필요하다는 것입니다. 이러한 호출들은 메모리 누수를 피하기 위해 파괴되었을 때 관리하고 클린업 하는 게 필요합니다. 이런 경우는 많은 유지보수가 필요합니다.
	- 다른 문제는, ViewController가 유저액션에 반응하는 너무 많은 로직을 담고 있다는 것입니다. 이것은 god activity가 되는 문제점을 초래합니다.

- 해결책
	- ViewModel을 사용해서 UI와 연관된 데이터는 ViewController로부터 분리하게 할 수 있습니다. ViewModel은 기기설정 변경에 따라 자동으로 관리됩니다. 다음 액티비티와 프래그먼트에서도 즉시 사용가능합니다.
		```java
		public class MyViewModel extends ViewModel {
				private MutableLiveData<List<User>> users;
				public LiveData<List<User>> getUsers() {
						if (users == null) {
								users = new MutableLiveData<List<Users>>();
								loadUsers();
						}
						return users;
				}

				private void loadUsers() {
						// do async operation to fetch users
				}
		}

		public class MyActivity extends AppCompatActivity {
				public void onCreate(Bundle savedInstanceState) {
						MyViewModel model = ViewModelProviders.of(this).get(MyViewModel.class);
						model.getUsers().observe(this, users -> {
								// update UI
						});
				}
		}
		```

	- 만약 액티비티가 재생성 된다면, 이전 액티비티에 의해 생성된 MyViewModel 인스턴스를 받습니다. Owner 액티비티가 종료될 때 프레임워크는 ViewModel의 onCleared() 메소드를 호출합니다.

- 알아두기
	- ViewModel은 액티비티나 프래그먼트보다 오래 살아야 하기 때문에 액티비티나 View를 참조하면 안 됩니다. Context를 참조하지 말아야 한다는 뜻입니다.
	- 만약 ViewModel에 Application Context가 필요하다면, AndroidViewModel 클래스를 확장시켜야 합니다.

<br>
<br>

### 프래그먼트들 사이에서 데이터 공유하기
- 만약 액티비티에 여러 개의 프래그먼트가 있다면, 프래그먼트마다 데이터를 공유하기 위해 여러 가지 작업을 해야 합니다. 이러한 불편함을 ViewModel로 해결할 수 있습니다.
	```java
	public class SharedViewModel extends ViewModel {
			private final MutableLiveData<Item> selected = new MutableLiveData<Item>();

			public void select(Item item) {
					selected.setValue(item);
			}

			public LiveData<Item> getSelected() {
					return selected;
			}
	}

	public class MasterFragment extends Fragment {
			private SharedViewModel model;
			public void onActivityCreated() {
					model = ViewModelProviders.of(getActivity()).get(SharedViewModel.class);
					itemSelector.setOnClickListener(item -> {
							model.select(item);
					});
			}
	}

	public class DetailFragment extends LifecycleFragment {
			public void onActivityCreated() {
					SharedViewModel model = ViewModelProviders.of(getActivity()).get(SharedViewModel.class);
					model.getSelected().observe(this, { item ->
						 // update UI
					});
			}
	}
	```
- 두 프래그먼트가 같은 액티비티(getActivitiy())를 사용한다는 것을 알아두세요. 이것은 두 프래그먼트가 같은 SharedViewModel 인스턴스를 사용한다는 것을 의미합니다. 아래와 같은 이점이 있습니다.
	- 두 프래그먼트 간의 통신을 위해 액티비티에서 따로 작업할 필요가 없다.
	- 프래그먼트들은 서로를 알 필요가 없다.
	- 프래그먼트들은 서로의 라이프사이클을 가지며, 서로 영향을 주지 않는다.

<br>
<br>

### ViewModel의 라이프사이클
- 정의
	- ViewModel 객체들은 ViewModelProvider에 넘겨진 Lifecycle의 범위를 갖습니다.
	![ViewModel's lifecycle](https://developer.android.com/images/topic/libraries/architecture/viewmodel-lifecycle.png)
