Instead of creating objects manually, dependencies are **provided externally**.

Benefits:
- loose coupling  
- easier testing  
- cleaner architecture

**❌** BAD Example: 
```
class StockViewModel(
	private val repo: StockRepositoryImpl = StockRepositoryImpl()  
) : ViewModel() {}
```
This is **❌**BAD because:
- ViewModel is tightly coupled to implementation ❌
- harder to test ❌
- not true dependency injection ❌

- When a class A require a fun in other class, it means that class A has constructor dependency(like viewmodel need repository), so to provide it
- declare dependency inside constructor
- inject it(using factory or Hilt)

## Hilt way
### With ViewModel
#### 🔥This strictly follow **clean** code, adhering the purpose of **Interface**

**Step 1**: Use **@Inject constructor**. It tells Hilt to inject depenencies into constructor -> no manually calling

```
// Here use case, and repos is automatically injected WHEREVER ViewModel is used
@HiltViewModel
class DashboardViewModel @Inject constructor(
    private val getDashboardDataUseCase: GetDashboardDataUseCase,
    private val userPreferencesRepository: UserPreferencesRepository //⭐ Not Impl
) : ViewModel() {

class GetDashboardDataUseCase @Inject constructor(
    private val dashboardRepository: DashboardRepository,
)

class UserPreferencesRepositoryImpl @Inject constructor(
    private val dataStore: UserPreferencesDataStore
) : UserPreferencesRepository
```

**Step 2**: Create Binding: @Binds
```
@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {

	@Binds
	@Singleton
	abstract fun bindUserPreferenceRepository(
		impl: UserPreferencesRepositoryImpl
	): UserPreferencesRepository
}


```
#### 💡Why Binding? Because now ViewModel depend on abstraction (interface) and not concrete(Impl) - which following SOLID principles

- 💡**Why @Singleton**? It tells Hilt to create the class Impl once and resuse that instance everywhere -> but **why** just need once? Basically you **dont** need mutiple instances of Impl
- 💡What is this?
```
@Module  
@InstallIn(SingletonComponent::class)  
abstract class RepositoryModule{}
```
 - This tell Hilt: this class contains dependency definitions, and those dependencies belong to the application's singleton dependency graph.
 - Break down:
	 -  **@Module**: This class provides or binds dependencies, **without** it, Hilt **ignores** the class
	 - **@InstallIn(SingletonComponent::class)**: Install these bindings into application-wide scope.
	 - **SingletonComponent** is the top/largest level -> Binds apply eveywhere in the app(including VIewModel)
	 ```
	 SingletonComponent  //⭐ top level - app-wide
		└── ActivityRetainedComponent  
			└── ViewModelComponent  
				└── ActivityComponent  
					└── FragmentComponent
	 ```


-> So for **optional small scale app**, when Interface only have 1 Impl
```
@HiltViewModel
class DashboardViewModel @Inject constructor(
    private val getDashboardDataUseCase: GetDashboardDataUseCase,
    private val userPreferencesRepository: UserPreferencesRepositoryImpl // ⭐
) : ViewModel() {
```

## No Hilt way
A clean DI(No Hilt is)
Step 1: Depend on an interface - implementation(like repository)
Step 2: Then ViewModel declare it 
```
class StockViewModel(  
	private val repo: StockRepository  
) : ViewModel() {
```
Step 3: Manual Injection
```
val viewModel: StockViewModel = viewModel(  
	factory = object : ViewModelProvider.Factory {
		override fun T : ViewModel> create(modelClass: Class T): T {
			val repo = StockRepositoryImpl()  
			return StockViewModel(repo) as T
		  }
		}
	)
```

Another way, **unrecommened**: 
🔥**but useful for interview**
If you you class depent on something, simply create an instance of that class to use it
Example:

```
class StockViewModel(  
	private val repo: StockRepository  
) : ViewModel() {
```
```

fun StockScreen() {
	val repository = StockRepository()
	val stockViewModel: StockViewModel(repository)
}
```