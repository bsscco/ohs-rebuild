# 짧은 소개
### 아키텍쳐 컴포넌트
- 테스트가 쉽게 유지보수가 쉽게 앱을 디자인할 수 있는 라이브러리 모음

### 컴포넌트:라이프사이클 컴포넌트
- 뭔가요?
	- 기기 설정이 바뀌었을 때 오류가 없게 해줍니다.
	- 메모리 누수를 피하게 해줍니다.
	- 데이터를 쉽게 로딩하게 해줍니다.
- 클래스 목록
	- LiveData
	- ViewModel
	- LifecycleObserver
	- LifecycleOwner
- 링크
	- [코드랩](https://codelabs.developers.google.com/codelabs/android-lifecycles/#0)
	- [닥스](https://developer.android.com/topic/libraries/architecture/lifecycle.html)
	- [샘플](https://github.com/googlesamples/android-architecture-components)
	
### 컴포넌트:Room
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

### 아키텍쳐 공통 원칙
- 가장 중요한 원칙은 **관심사의 분리**를 하는 것입니다. 
	- 우리는 모든 코드를 액티비티나 프래그먼트에 작성하는 실수를 범하곤 합니다. 이것은 라이프사이클과 관련있는 여러 문제들을 야기시킵니다. 앱 컴포넌트에서 UI 핸들링이나 OS와의 인터렉션을 독립시킴으로써 관심사의 분리를 실현할 수 있습니다. **관심사의 분리를 통해 라이프사이클과 관련된 문제들을 피할 수 있습니다.**

- 그 다음 중요한 원칙은 **영속적인 모델로부터 UI를 운용**하는 것입니다. 
	- OS에 의해 앱이 강제종료 됐다가 다시 살아나는 등의 라이프사이클 문제가 발생해도 유저는 앱이 상태를 잃어버리기 원치 않습니다. **영속적인 모델은 라이프사이클로부터 독립적으로 앱 데이터를 핸들링하는 책임을 지기 때문에 이러한 문제의 해결책이 됩니다.** 그것은 모델이 앱 컴포넌트나 View로부터 독립되어야 한다는 뜻입니다. 
	
### 추천하는 앱 아키텍쳐
- UI Building
	- 1) View와 ViewController 준비 : 유저 프로필을 보여주는 화면(UserProfileFragment.java, user_profile_layout.xml)이 있다고 가정합시다.

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
		- UserProfileFragment는 View를 
		
	- 5) View와 ViewModel을 독립적으로 연결하기 : 이제 이 3가지 어떻게 연결할까요?
		- ViewModel의 유저 필드가 세팅됐을 때 우리는 UI에게 유저 필드의 정보를 알려야 합니다. LiveData 클래스를 통해 알릴 수 있습니다.
		- **LiveData**
			- 관찰될 수 있는 데이터 홀더입니다. 이 기능을 사용하면 당신의 앱 컴포넌트가 LiveData 객체의 변화를 명시적이고 의존적인 경로 없이 관찰 할 수 있게 됩니다.
			- 앱 컴포넌트들의 라이프사이클을 존중하며 앱이 메모리를 더 사용하지 않도록 메모리 누수를 방지합니다.
	- 6) 이제 우리는 UserProfileViewModel 안에 있는 유저 필드를 LiveData<User>로 대체할 거에요.
		
		
	
- 데이터 fetching
- ViewModel과 Repository 연결하기
- 데이터 캐싱
- 데이터를 영속적으로 보존하기
- 테스팅
- 최종 아키텍

### 원칙 가이드

### 추가적인 내용

