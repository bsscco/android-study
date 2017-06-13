# MVVM

### MVVM
MVVM MVC의 파생이며, 따라서 엄밀히 말하면 MVC입니다. 

<br>

### Model
Model은 데이터 상태를 대표하며, 데이터를 처리하는 비지니스 로직도 가지고 있습니다. 서버로 데이터 요청하기, 데이터 캐시 등의 작업을 합니다.

<br>

### View
View는 사용자에게 보여지는 UI입니다. 사용자로부터 들어온 입력들을 관찰할 수 있는 메소드들을 제공합니다.

<br>

### ViewModel
ViewModel은 View와 Model을 알고 있습니다. View의 메소드들을 사용하여 사용자의 입력을 관찰합니다. 입력이 들어왔을 땐 Model에게 데이터 처리를 요청합니다. 데이터가 갱신되면 그에 대응되는 UI도 갱신되어야 합니다. ViewModel은 데이터를 통해 View의 UI를 갱신하지만 이렇게 하면 ViewModel과 View가 강한 결합을 하게 됩니다. 따라서 Model의 데이터가 갱신되면 자동으로 대응되는 View의 UI도 갱신되도록 데이터 바인딩을 통해 해결합니다.

- 데이터 바인딩이란?
데이터 제공자와 소비자를 묶어서 동기화 시키는 기법입니다. 데이터가 수정되면 데이터에 묶여있는 UI 요소도 자동으로 갱신됩니다.

<br>

### 참고
- [위키 MVVM](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93viewmodel)
- [위키 Data binding](https://en.wikipedia.org/wiki/Data_binding)
- [이규원님의 mvvm-architecture](https://justhackem.wordpress.com/2015/03/19/rmvvm-architecture/)
