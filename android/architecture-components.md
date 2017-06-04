# 짧은 소개
### 아키텍쳐 컴포넌트
- 테스트가 쉽게 유지보수가 쉽게 앱을 디자인할 수 있는 라이브러리 모음

### 컴포넌트 : 라이프사이클 컴포넌트
- 뭔가요?
	- 기기 설정이 바뀌었을 때 오류가 없게 해줍니다.
	- 메모리 누수를 피하게 해줍니다.
	- 데이터를 쉽게 로딩하게 해줍니다.
- 클래스 목록
	- LiveData(라이프사이클에 맞춰 *데이터를 자동 관리*해주는 클래스)
	- Repository(*데이터 연산*을 담고 있는 클래스)
	- ViewModel(뷰를 위한 *데이터를 담고 있고, 핸들링하는* 클래스) 
	- LifecycleObserver(LiveData의 자동화를 위한 기능이 들어있는 클래스)
	- LifecycleOwner(LiveData의 자동화를 위한 기능이 들어있는 클래스)
- 링크
	- [코드랩](https://codelabs.developers.google.com/codelabs/android-lifecycles/#0)
	- [닥스](https://developer.android.com/topic/libraries/architecture/lifecycle.html)
	- [샘플](https://github.com/googlesamples/android-architecture-components)
	
### 컴포넌트 : Room
- 뭔가요?
	- SQLite ORM 라이브러리
	- 컴파일타임에 SQL을 체크해줘요.
- 클래스 목록
	- Room
- 링크
	- [코드랩](https://codelabs.developers.google.com/codelabs/android-persistence/#0)
	- [닥스](https://developer.android.com/topic/libraries/architecture/room.html)
	- [샘플](https://github.com/googlesamples/android-architecture-components)
	
# 덜 짧은 소개
### 개발자들이 직면한 공통의 문제
- 안드로이드 앱은 독립적인 다양한 컴포넌트(Activity, Fragment 등)로 구성되어있습니다. 각 컴포넌트는 앱의 entry point가 될 수 있기 때문에 안드로이드 앱은 여러 개의 entry point를 가질 수 있습니다. 안드로이드 앱은 어떤 entry point에서 시작될지 모르기 때문에 **컴포넌트 간 데이터를 독립시켜야 합니다.**
- 또한 안드로이드 앱은 메모리 관리가 필요할 때 OS에 의해 강제종료 되었다가 필요할 때 재실행 될 수 있습니다. 앱이 강제종료 되었을 때 데이터를 보존시켜야 하기 때문에 **데이터를 컴포넌트에 저장해서는 안 됩니다.** 컴포넌트는 앱이 종료될 때 함께 메모리에서 해제됩니다.
- multi entry point와 OS의 앱 강제종료-재실행 문제는 **앱 컴포넌트의 특이한 라이프사이클에 의해 발생한 문제들입니다.**

### 이 문제들을 방지하기 위한 아키텍쳐 공통 원칙
- 가장 중요한 원칙은 **관심사의 분리**를 하는 것입니다. 
	- 우리는 모든 코드를 액티비티나 프래그먼트에 작성하는 실수를 범하곤 합니다. 이것은 라이프사이클과 관련있는 여러 문제들을 야기시킵니다. 앱 컴포넌트에서 UI 핸들링이나 OS와의 인터렉션을 독립시킴으로써 관심사의 분리를 실현할 수 있습니다. **관심사의 분리를 통해 라이프사이클과 관련된 문제들을 피할 수 있습니다.**

- 그 다음 중요한 원칙은 **영속적인 모델로부터 UI를 운용**하는 것입니다. 
	- OS에 의해 앱이 강제종료 됐다가 다시 살아나는 등의 라이프사이클 문제가 발생해도 유저는 앱이 상태를 잃어버리기 원치 않습니다. **영속적인 모델은 라이프사이클로부터 독립적으로 앱 데이터를 핸들링하는 책임을 지기 때문에 이러한 문제의 해결책이 됩니다.** 그것은 모델이 앱 컴포넌트나 View로부터 독립되어야 한다는 뜻입니다. 
	
### 추천하는 앱 아키텍쳐
- UI Building
	- 1) View와 ViewController 준비 : 유저 프로필을 보여주는 화면(UserProfileFragment.java, user_profile_layout.xml)이 있다고 가정합시다.
		- UserProfileFragment는 LifecycleOwner를 implementing 한 class입니다.

	- 2) 데이터 모델 구상 : UI를 운용하기 위해서 데이터 모델을 두 가지 데이터 요소로 구성합니다.
		- User ID
		 	- 프래그먼트 인자를 사용하는 프래그먼트에 정보를 넘길 때 최선의 방법이 됩니다.
			- User ID로 유저 데이터를 식별할 수 있습니다.
		- User Object
		 	- 유저 데이터
	
	- 3) ViewModel 준비 : 우리는 UserProfileViewModel를 생성할 겁니다.
		- **ViewModel**
			- 프래그먼트, 액티비티 같은 UI를 기술하는 데이터를 제공합니다.
			- 비지니스 파트 데이터를 핸들링합니다.
			- View를 알지 못하며 기기 설정 변경에 영향을 받지 않습니다.
	
	- 4) 준비 결과 : 이제 우리는 3개의 파일을 가지게 됐습니다.
		- user_profile_layout.xml
			- UI가 정의된 파일
		- UserProfileViewModel.java
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
		- UserProfileFragment.java
			- ViewModel에 있는 데이터를 보여주고 유저 인터렉션에 반응하는 UI 컨트롤러
			```java
			public class UserProfileFragment extends LifecycleFragment {
			    private static final String UID_KEY = "uid";
			    private UserProfileViewModel viewModel;

			    @Override
			    public void onActivityCreated(@Nullable Bundle savedInstanceState) {
				super.onActivityCreated(savedInstanceState);
				String userId = getArguments().getString(UID_KEY);
				viewModel = ViewModelProviders.of(this).get(UserProfileViewModel.class);
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
		
	- 5) View와 ViewModel을 연결하는 방법 : 이제 이 3가지 어떻게 연결할까요?
		- ViewModel의 유저 필드가 세팅됐을 때 우리는 UI에게 유저 필드의 정보를 알려야 합니다. LiveData 클래스를 통해 알릴 수 있습니다.
		- **LiveData**
			- 관찰될 수 있는 데이터 홀더입니다. 이 기능을 사용하면 우리의 앱 컴포넌트가 LiveData 객체의 변화를 명시적이고 의존적인 경로 없이 관찰 할 수 있게 됩니다.
			- 앱 컴포넌트들의 라이프사이클을 존중하며 앱이 메모리를 더 사용하지 않도록 메모리 누수를 방지합니다.
	
	- 6) View와 ViewModel을 LiveData로 연결하기
		- 이제 우리는 UserProfileViewModel 안에 있는 유저 필드를 LiveData<User>로 대체할 겁니다. 그러면 프래그먼트는 데이터가 갱신됐을 때 알림 받을 수 있습니다.
		- LiveData가 뛰어난 이유는, 그것이 라이프사이클을 잘 알고 있기 때문에 참조들을 clean up 시키는 작업을 라이프사이클에 따라서 자동으로 해주기 때문입니다.
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
		
	- 7) ViewController에서 LiveData 사용하기 : 이제 우리는 데이터를 관찰해서 UI를 갱신할 수 있도록 프래그먼트를 수정할 겁니다.
		```java
		@Override
		public void onActivityCreated(@Nullable Bundle savedInstanceState) {
		    super.onActivityCreated(savedInstanceState);
		    viewModel.getUser().observe(this, user -> {
		      // update UI
		    });
		}
		```
		- LiveData는 자동으로 프래그먼트의 ```onStart()```에서 데이터를 받고, ```onStop()```에선 데이터를 받지 않고, ```onDestroy()```에서는 Observer를 제거합니다. 그밖에 기기 설정이 바뀌었을 때도 우리가 특별히 할 일은 없습니다. 기기 설정이 바뀔 때 LiveData는 자동으로 데이터를 저장하고, 새로운 ViewModel 인스턴스에서 데이터를 불러옵니다. 이것이 LiveData가 View의 참조를 직접적으로 가지지 않아야 하는 이유입니다.

- 데이터 fetching
	- 8) 데이터 fetching API 준비하기 : ViewModel이 어떻게 유저 데이터를 fetching 할까요?
		- 우리의 백엔드가 REST API를 제공한다고 가정해서 우리는 Retrofit 라이브러리를 사용할 겁니다.
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
	- 9) ViewModel에서 직접 fetching 하면 안 되는 이유
		- ViewModel은 Webservice에 직접 접근해서 데이터를 fetching할 수 있고, 데이터를 User 객체에 할당할 수 있습니다. 이것은 ViewModel에게 너무 많은 책임감을 쥐어줍니다. 이것은 곧 관심사의 분리라는 원칙에 어긋나게 됩니다. 추가적으로 ViewModel의 범위는 프래그먼트나 액티비티의 라이프사이클에 얽메이게 됩니다. 그래서 라이프사이클이 종료될 때 데이터도 함께 잃어버리게 됩니다. 이것은 유저 경험에 좋지 않습니다. 대신에 ViewModel은 Repository 모듈에게 이 작업을 위임합니다.
		- **Repository**
			- 데이터 연산의 책임을 집니다.
			- 깔끔한 REST API를 제공합니다.
			- Repository는 어디서 데이터를 가져와야 할지, 데이터가 갱신될 때 어느 API를 호출해야 할지 압니다.
			- 우리는 Repository를 캐싱, 영속적 모델, 웹 서비스 등의 사이에서의 중재자로 여길 수 있습니다.

	- 10) Repository 준비
		```java
		public class UserRepository {
		    private Webservice webservice;
		    // ...
		    public LiveData<User> (int userId) {
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
		- Repository는 쓸모 없어 보일지 모르지만, 그것은 중요한 목적이 있습니다. Repository는 앱으로부터 데이터 소스를 추상화 합니다. 이제 우리의 ViewModel은 데이터가 Webservice에 의해서 fetching 되는지 알지 못합니다. 이것은 우리가 필요할 때 다른 구현체로 교체할 수 있다는 것을 의미합니다.
		
	- 11) 컴포넌트 사이의 의존성 관리하기
		- 
		
- ViewModel과 Repository 연결하기
- 데이터 캐싱
- 데이터를 영속적으로 보존하기
- 테스팅
- 최종 아키텍

### 원칙 가이드

### 추가적인 내용

