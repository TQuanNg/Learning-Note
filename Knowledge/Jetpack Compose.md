
# Data Layer

### API
API, Data from Internet
**Retrofit** is a **type-safe HTTP client library** for Android that lets you call web APIs using **interfaces and annotations instead of manual network code**.
> **OkHttp performs the real network communication.**

OkHttp is the engine and Retrofit is an abstract interface.
So **when** do you need OkHttp? When i need a bit more advance feature like authentication, caching, logging, custom header

### Difference Between `launch` and `async`
Both create coroutines, but they are used **for different purposes**.
`launch` and `async` create a new coroutine and they are not suspend functions.

| Feature      | `launch`              | `async`               |
| ------------ | --------------------- | --------------------- |
| Return value | No result             | Returns result        |
| Return type  | `Job`                 | `Deferred<T>`         |
| Use case     | Fire-and-forget tasks | Parallel computations |

#### `launch`

Used when you **do not need a return value**.

#### `async - await`
You get the value using: await()
⭐: Common, modern way for handling asynchronous tasks
Returns: Deferred\<T>

Used when you need a result. Can be run in parallel
- With async, use **supervisorScope** with the individual `try-catch` for each task, when you want to continue with other tasks if one or some of them have failed.
- With `async`, use **coroutineScope** with the top-level `try-catch`, when you do **NOT** want to continue with other tasks if any of them have failed.

Benefits:
- prevents memory leaks
- automatic cancellation
- 🔥 easier error handling (try/catch)
```
textRecognizer  
	.process(image)  
	.await()
```

📌**suspendCancellableCoroutine**: convert **callback** API into a **suspend function**
Designed to convert legacy, callback-driven APIs (e.g., listeners, completion handlers, RxJava) into modern suspending functions.
used when some library doesn't provide await()

Ex: 
- BluetoothSDK, BLE SDK, location SDK, custom callback APIs
```
suspend fun awaitCallback(): T = suspendCancellableCoroutine { continuation ->
	recognizer.process(image)  
		.addOnSuccessListener { visionText ->
			continuation.resume(visionText)
		}  
		.addOnFailureListener { 
			continuation.resumeWithException(it)
		}
}
```
❗Manually requires `cont.invokeOnCancellation` cleanup logic.



### DataStore - SharedPreferences
**DataStore** is the **modern, recommended replacement** for `SharedPreferences`.
2 types: 
- Key–Value
- Proto DataStore (Typed objects, strongly recommended for larger data)
- message UserSettings {
  bool dark_mode = 1;
  string language = 2;
}

Google introduced it to fix:
- Thread safety issues
- Performance problems
- Poor API design of SharedPreferences

`SharedPreferences` is a **legacy key–value storage** mechanism in Android used to store **small, simple data**. such as Theme preference, language



### Annotation
@**Volatile**: The value of a volatile variable is never cached, and all reads and writes are to and from the main memory. These features help ensure the **value** of  is always up to date and is the same for all execution threads. It means that changes made by one thread to `Instance` are immediately visible to all other threads.
@**Singleton**: App wide. Lives until app process dies
@**Module**: 
@**ActivityScoped**: One per ActivityScoped
@**ViewModelScoped**: 
@**Serializer**: converting Kotlin objects to and from JSON.
@Serializable: Convert an object into a format that can be stored or transferred.

@**HiltAndroidApp** : to trigger code generation (by ksp or kapt)
@**AndroidEntryPoint**: Instruct Hilt to add that main class to DI, so sublass, smaller unit can apply Hilt for DI

If a single instance needs to be shared across various ViewModels, then it should be scoped using either `@ActivityRetainedScoped` or `@Singleton`
Basically:
- to share dependency: use`@ActivityRetainedScoped` or `@Singleton`
- to obtain dependency: use `@Inject` or `@Provide`



# UI Layer
use "=" for normal values
use by for compose values

## Navigation
A navigation is construct by route and argument(optional)

1/ NavController vs NavHostController
- `NavController` → **interface**
- `NavHostController` → **implementation**

`NavHostController` is a Compose-specific implementation of `NavController`.  
In Jetpack Compose, we create a `NavHostController` using `rememberNavController()` and pass it to `NavHost`.  
For screens, we typically depend on `NavController` to keep navigation loosely coupled and testable.

Ways to navigate and when to use
- Navigation Arguments
- Shared ViewModel

### Passing data to screen
#### Navigation Arguments (for primitives and IDs)
Doc: https://developer.android.com/guide/navigation/type-safe-destinations
**The Problems**: 
First we need to understand that navigation argument is **not** type-safe - meaning it does not check wether argument passed match the data type defined.

**Route strings** (old way)
You need to manually build the route, read arguments, convert types
Example:
```
navController.navigate(  
	"expense_review?merchant=Amazon&amount=25"  
)

// Usage
val merchant = backStackEntry.arguments?.getString("merchent") // misspell the key - but no error
- Or missing argument, argument name mistmatch, unmatch data type -> crash
```
✅**Typed Navigation (modern, recommended)**
- fixes it by verifying type, compiler knows what the destination expects
- Cleanner approach
How to implement it? ✅**Serializable** + **Navigation Compose**
#### ✅ Serializable
⚠️⚠️ Android Navigation **can't** put Kotlin objects directly into a route or `Bundle`. -> Serialization is used con make object be **transferable** between screen navigation

Convert an object into a format that can be stored or transferred.
**Deserialization**: Convert that stored format back into the original object.

When to use:
- **Navigation arguments**
- Saving objects to disk (JSON)
- Sending or receiving JSON from an API (when using Kotlinx Serialization)
- Persisting objects in preferences or files

##### Kotlin Serialization
- Navigation Compose **2.8+**
- Required plugin and dependency
```
alias(libs.plugins.kotlin.serialization)

implementation(libs.kotlinx.serialization.json)
```
- How to use:
```
@Serializable
object Home

@Serializable
data class Profile(val userId: String)


composable<Profile> { backStackEntry ->
	// The library automatically handles argument extraction
	val profile: Profile = backStackEntry.toRoute()
	ProfileScreen(profile.userId)
}


// pass
navController.navigate(Profile(userId = "user123"))


// retrieve
savedStateHandle.toRoute<ExpenseReview>()
```
##### Serializable Limitation
⚠️Only support primitive type
⚠️ `LocalDate` is **not** serializable by default.
✅Solve
 - Store the date as an ISO string in data class
`val date: String = LocalDate.now().toString()`
- Then use: 
`val localDate = LocalDate.parse(expenseDraft.date)`

##### Navigation Compose Limitation
⚠️⚠️Serializable **only** make data to be transferable - NOT participate in Type Safety
Navigation Compose only handles Primitives and IDs, NOT Custom Types (ie: data class)
**✅Solve: (Advanced) Handling Custom Types**


If you need to pass complex data classes (not just primitives), you must define a custom `NavType`.
1. **Create the Custom Type**: 
```
kotlin val SearchFilterType = object : NavType(isNullableAllowed = false) {

override fun get(bundle: Bundle, key: String): SearchFilter? = Json.decodeFromString(bundle.getString(key) ?: return null)


override fun parseValue(value: String): SearchFilter =
    Json.decodeFromString(Uri.decode(value))

override fun put(bundle: Bundle, key: String, value: SearchFilter) {
    bundle.putString(key, Json.encodeToString(value))
}
```

2. **Register it in the Graph**:
```kotlin
composable<Search>(
    typeMap = mapOf(typeOf<SearchFilter>() to SearchFilterType)
) { ... }
````


##### Parcelable (old way, still common)
Parcelable is Android's way of putting objects into: Intent, Bundle, or SavedStateHandle
It solves a different problem compared to Serialization
```
@Parcelize  
data class ExpenseDraft(  
	val merchant: String,  
	val amount: String,  
	val date: LocalDate  
) : Parcelable
```


#### Shared ViewModel (for complex object)


## Screen
### Screen **should** creates/gets ViewModel from DI + lifecycle
#ViewModel
❌**Bad**: `viewModel: StockViewModel = StockViewModel()`
-> This creates a **new ViewModel on every recomposition**

✅Use the Compose ViewModel helper: 
`val viewModel: StockViewModel = viewModel()`
which require implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.8.0")
viewModel() = tied to Activity lifecycle  
		= survives recomposition  
		= proper architecture
#### What this does
- Asks **Hilt + Navigation** for a `HomeViewModel`
- Scopes it to the **current NavBackStackEntry**
- Handles:
    - lifecycle
    - configuration changes
    - SavedStateHandle
- Ensures **one ViewModel instance per destination**
This is **CREATION / OWNERSHIP**

❌❌It **SHOULD** **NOT** receive ViewModel elsewhere(like in navgraph from what i did). **WHY NOT?**
#### Problems
❌ UI now depends on ViewModel type  
❌ Harder to preview  
❌ Harder to reuse  
❌ Couples UI to state source

##### `SavedStateHandle`: a small key-value map attached to a ViewModel.
It lets your ViewModel:
1. **Receive navigation arguments**
2. **Save small pieces of UI state** that can survive process death
⭐ Use it for navigation arguments

```
composable(  
	route = "expense_detail/{expenseId}"  
)

@HiltViewModel  
class ExpenseDetailViewModel @Inject constructor(  
savedStateHandle: SavedStateHandle  
) : ViewModel() {  
	val expenseId: String = checkNotNull(savedStateHandle["expenseId"])
```

## ViewModel
#ViewModel A **ViewModel** is a class designed to **store and manage UI-related data** in a lifecycle-aware way.
Key benefits:
- Survives configuration changes (e.g., screen rotation)
- Separates UI logic from UI (Activity/Composable)
- Works well with LiveData/StateFlow

ViewModel
- `_uiState` → **mutable**, type `MutableStateFlow` internal to ViewModel
- `MutableStateFlow` holds ScreenUiState
- `uiState` → **read-only**, observed by Compose

`update {}` is from **Kotlin StateFlow**.

#### MutableStateFlow functions

1/ **uiState.value** : read/set current state
- ```
  uiState.value = _uiState.value.copy(  // Replace state directly
		query = "Array"  
	)
  ```

2/ `_uiState.update { currentState ->   newState }`
`_uiState.update { it.copy(isLoading = true, error = null) }`
- Takes the **current value**
- Returns a **new value**
- Emits it atomically
No mutation. No side effects. Very safe.

3/ **asStateFlow**() : expose read-only state
`val uiState: StateFlow<HomeUiState> =  _uiState.asStateFlow()`

4/ **collect**() : observe values - observe mean it receives every emitted value
```
viewModel.uiState.collect { state ->  
	println(state.query)  
}

val uiState by viewModel.uiState.collectAsState()
```

5/ **map**(): Transform a Flow(from StateFlow)
```
val usernames = users.map { list ->
    list.map { user ->
            user.name
    }
}
```

6/ **combine()**: Recompute UI every time **ANY** of these changes -> This ensure Single source of truth
**combine** is also better than manually uiState.update because UI might update in wrong order(race condition) BUT **combine** ensure UI is recomputed ONLY when all latest values are available together
	- **Note**: combine produce **Flow** not StateFlow so **StateIn** should be used along with
```
combine(A, B, C) { a, b, c -> result }
```

7/ **stateIn**:
- This converts Flow → StateFlow -> so Ui can **collectAsState**()
- controls: lifecycle, caching, performance of upstreams flows

```
stateIn(
        scope = viewModelScope, // lifecycle owner of the stream
        
        started = SharingStarted.WhileSubscribed(5_000), // Only keep the flow active while UI is observing it AND wait 5 seconds before stopping
        
        initialValue = emptyList() // It defines: What UI sees before first real data arrives so UI not crash or null
    )
```
🧩combine → builds the data **AND** stateIn → turns it into UI-ready state holder

9/ filter(): filter emisions
```
flow.filter {  
	it.isNotEmpty()  // condition
}
```

10/ **debounce**(500): get/ run after 500ms -  avoid frequent run on change
- Used when: user typing, API calls
```
query  
	.debounce(500)  
	.collect {  
		search(it)  
	}
```

11/ **distinctUnilChanged**(): skip duplicate values
```
query  
	.distinctUntilChanged()
```

12/ **flatMapLatest**(): Switch to newest Flow and **cancel** old one.
- Usually used for search, or network call
- Cancel means it already run, but then cancel -> a bit inefficient so
- -> **debounce**() usually go along with to avoid that "already run"
```
query  
	.debounce(500)  
	.distinctUntilChanged()  
	.flatMapLatest {  // if there is old currently running Flow -> cancel it
		repository.search(it)  
	}
```

14/ **first**(): get first one value
- typical found in test
```
val state = uiState.first()
```

15/ **emit**(): emit value from a coroutine
- Mostly used with **MutableSharedFlow** / still can be used with StateFlow
- Less common with StateFlow beaucse it alreadu jas a cirremt value
#### data class functions
`copy()`: 
- Creates a **new UiState instance**
- Changes only the specified fields
- Keeps everything else the same
This is **immutability**

> What's the difference between `value = value.copy(...)` and `update { it.copy(...) }`?

You can answer:
> Both create a new immutable state object and emit it through StateFlow. However, `update` performs the modification atomically, making it safer when multiple coroutines may update the state concurrently. For that reason, `update {}` is generally preferred in modern ViewModels.

### 🔥 Common collection operations
**.map**: transform each item into something else, it **does not change** original list, but rather **create NEW** transformed list
- frequent used for creating **new** state as Compose prefer immutable state update

**.filter**: keep matching items
```
list.filter { it.isRunning }
```
**.filterNot**: removing matching items
```
list.filterNot { it.id == id }
```

**.forEach**: Loop
- Return Unit

**.find / firstOrNull**: get that First One item
```
list.firstOrNull { it.id == id }
```

.**any**: check existence
```
list.any { it.isRunning }
```

.**all**: check all match
```
list.all { it.completed }
```

# General Knowledge
## Flow
**Flow** is part of Kotlin Coroutines. It represents a **stream of asynchronous data**.
We use flow to update data that usually change over time.
Examples:
- database updates (Room)
- network polling
- user input
- real-time UI state
### Cold Flow
- Starts emitting **only when collected**
- Each collector gets fresh data
### Hot Flow
Emits **even if no one is collecting**

Types:
 1. **StateFlow**: holds **state**
- always has a value
- emits latest value to new collectors
 1. **SharedFlow**
 - does not need initial value
 - can emits many previous values using replay operator
 -> StateFlow is a type of SharedFlow


Error: 
"Skipped frames" error means too much work is happening on the main/UI thread.
Probable causes:

**mutableStateOf** vs **StateFlow**
StateFlow = default for real apps  
mutableStateOf = simple UI / quick logic
**StateFlow** is:  
✔ lifecycle-aware  
✔ works with coroutines  
✔ scalable  
✔ testable  
✔ consistent with backend async flow

For simplicity I can use mutableStateOf, but in production I’d typically use StateFlow and collect it in Compose using collectAsState.



## Work Manager
> Why remove WorkManagerInitializer?
	You can say:

> When using custom WorkManager configuration via Configuration.Provider, the default initializer must be removed to prevent duplicate initialization and conflicts during app startup.

how do you handle data syncing when the network isn’t stable?
#Interview_answer Work manager to schedule background sync task, it works bt automatically retrieves data when network is back
Or Offline approach, which store data locally, maybe using json file for Room, then sync with server by comparing data version
## Design Pattern
### 1/**MVVM**(Model-View_ViewModel) Clean Architecjtire Android
separation of concerns  
testability  
lifecycle awareness  
clean UI logic

MVVM separates UI logic from business logic. The View displays UI, the ViewModel manages UI-related data and business logic, and the Model represents the data layer.

View-ViewModel ---Domain(Usecase) --- Data(Repositori - Local - Remote)

### 2/ Singleton Pattern
**Singleton pattern**: ensures that one, and only one, instance of an object/class is created and has one global point of access to that object. Object initialization is thread-safe and done at first access.

Ex:
- Retrofit instance  
- Room database  
- Logger

**Singleton pattern** is not a recommended practice. Singletons represent global states that are hard to predict, particularly in tests. Objects should define which dependencies they need, instead of describing how to create them.
**Use** Dependency injection over singleton pattern.
### 3/ Dependency Injection
Instead of creating objects manually, dependencies are **provided externally**.

Benefits:
loose coupling  
easier testing  
cleaner architecture

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

#### 🔥 Why this is better

```
✔ decoupled (interface)
✔ testable (can mock repo)
✔ flexible (swap implementations)
✔ shows real architecture knowledge
```
#DI Hilt - Dependency Injection(DI)
https://developer.android.com/training/dependency-injection/hilt-android
- Hilt does not allow injecting ViewModels outside of the Android ViewModel scope.
### 4/ Observer Pattern
This pattern allows objects to react to data changes

LiveData  
Flow  
StateFlow  
Compose state

### 5/ Factory Pattern(Old Android)
Creates objects without exposing creation logic.

Android uses factories for:
ViewModels  
Fragments : A **Fragment is a reusable piece of UI/behavior that lives inside an Activity**.
Objects with parameters

### 6/ Builder Pattern
Used to construct complex objects step by step.
```
Retrofit.Builder()  
	.baseUrl(BASE_URL)  
	.addConverterFactory(...)  
	.build()
```


|Your Feature|Pattern|
|---|---|
|ViewModel|MVVM|
|Repository layer|Repository|
|Hilt|Dependency Injection|
|Flow / StateFlow|Observer|
|Retrofit instance|Singleton|

#Interview_answer Common Android design patterns include MVVM for architecture, the Repository pattern for data abstraction, Dependency Injection for managing dependencies, the Observer pattern for reactive data updates, and Singleton for shared resources like databases or network clients.

### Difference between ViewModel and Activity lifecycle?
What is Lifecycle?
**Lifecycle simply means the stages an Android component goes through during its existence**.

What is Activity?
An **Activity represents a screen/container that Android can launch and manage**.
You should not rely on Activity fields for important UI state

Android's system manages the Activity lifecycle:
onCreate(): Called when the Activity is being created.
   ↓
onStart(): The Activity is now becoming **visible** to the user.
   ↓
onResume(): The Activity is now in the **foreground and ready for user interaction**.
   ↓
   [user interacting]
   ↓
onPause(): The Activity is **losing focus**.
   ↓
onStop(): The Activity is no longer visible. Releaase resources that aren't needed
   ↓
onDestroy(): the user finishes the Activity, system destroyed it



The Activity lifecycle is tied to the UI screen and is controlled by the Android system. When **configuration changes** occur, such as screen rotation, the Activity is **destroyed** and recreated. 

A ViewModel is designed to store and manage UI-related data in a lifecycle-aware way and survives configuration changes, allowing the UI state to **persist** across Activity recreation.


Android has a **single UI thread called the Main Thread**.
Responsibilities:
- drawing UI
- handling user input
- animations
- If a **network request runs on the main thread**, it blocks the UI and often throw network thread exception.
Best practice:
Run network calls on **background threads**. Ex: Dispatchers.IO
- `Main`: UI thread
- `IO`: network, disk, DB operations
- `Default`: CPU-heavy tasks

application not responding: ANR

## What is Application class?
The Application class is a base class provided by the Android framework that serves as a global representation of an app's process.
Use Cases:
• Initializing Libraries.
• Maintaining global application state (Singleton instances).
• Setting up dependency injection frameworks like Dagger or Hilt.

## Context
### What is Context in Android?
**Context** is an interface that provides access to **application resources and system services**.
Through Context you can:
- access resources(getString, getDrawable)
- start activities
- access system services
- open databases
- read files
There are 2 types of context:
### Application Context
This context exists as long as the **app process is alive**.
Used for:
- database
- repositories
- services
- long-lived objects

### Activity Context
This context is tied to an **Activity lifecycle**.
Used for:
- UI related tasks
- launching activities
- dialogs
- layouts

#Important: Holding Activity Context in long-lived objects causes **memory leaks**. To prevent this, Application Context should be used for long-lived objects.

**Interview Answer**: Activity Context is tied to the lifecycle of an Activity, which is short-lived. If a long-lived object like a repository or singleton holds a reference to an Activity Context, the Activity cannot be garbage collected, causing a memory leak. Using `@ApplicationContext` ensures that the injected context has the same lifetime as the application, preventing lifecycle mismatch and avoiding leaks.
Because static variables live for the entire application lifetime. If an Activity Context is stored there, the Activity will never be garbage collected, leading to a memory leak.

## Coroutines?
It means when functions cooperate with each other, we call it Coroutines
A **coroutine in Kotlin** is a lightweight way to run **asynchronous or background tasks without blocking a thread**
-> the **Coroutines** make multitasking very easy.
Example:
- API calls
- database queries
- file operations
- image processing


**CoroutineScope**: A **CoroutineScope** defines the **lifecycle and context in which coroutines run**.
Without scope -> it can cause memory leak
CoroutineScope  
   ├── Job : Tracks coroutines lifecycle
   └── Dispatcher: Dispatcher: Decides which thread runs coroutine
   
 Difference between `coroutineScope` and `supervisorScope`**
- **coroutineScope:** If any child fails → parent fails → all children cancelled.
- **supervisorScope:** Child failures **do not cancel siblings**.

What Is **Structured Concurrency**?
Structured concurrency means:
> Coroutines must run inside a **defined scope**, and their lifecycle is tied to that scope. So children coroutines depend on parent coroutine, if parent coroutine is destroyed, then all children are also destroyed.
> Without structured concurrency, background work could continue even after UI disappears.

⭐**Job** = an object representing and controlling that coroutine's lifecycle.
```
val job = viewModelScope.launch {}

job.cancel()
job.isActive
```

### 1️⃣ viewModelScope: Used inside a ViewModel.
```
viewModelScope.launch {
```

creates a coroutine tied to the ViewModel lifecycle.
That means:
- async work happens off main thread
- UI stays responsive
- coroutine cancels automatically when ViewModel is destroyed

### 2️⃣ lifecycleScope: Used in Activity or Fragment.



### 3️⃣ GlobalScope (Not Recommended): coroutines lives forever - bad

**suspend** Function: A function that can **pause without blocking a thread**. Usually used for network call
**Dispatchers** decide **which thread the coroutine runs on**.



# Android Security
#security
How to manage API keys?

Sensitive user data?


# Paging 3
Provide a bunch of benefit: built-in error handling, loading states, retry, memory optimization, scroll restoration, DB intergration, cache invalidation, load cancellation, etc
### PagingSource<Key, Value> (Where data comes from)
Key: page identifier, a generic type defines what to use to load NEXT chunk of data. It can be: page number, cursor, token, offset
Value: Type of data or the data
prevKey, nextKey, page to determine HOW to load page
#### load(): LoadResult<Int, Value>
```
try {
      // Start refresh at page 1 if undefined.
      val nextPageNumber = params.key ?: 1
      val response = backend.searchUsers(query, nextPageNumber)
      return LoadResult.Page(
        data = response.users,
        prevKey = null, // Only paging forward.
        nextKey = nextPageNumber + 1
      )
    } catch (e: Exception) {
      // Handle errors in this block and return LoadResult.Error for
      // expected errors (such as a network failure).
    }
```
#### getRefreshKey(PagingState<Key, Value>)
Used to determine current position when Refresh(pull to refresh, screen recreated, database changes, filter, search) happen.

**anchorPosition**: the last accessed item positon
**closestPageToPosition**(position): page/key contain that item.
set prevKey + 1 = current page = nextKey - 1
Return the Key(which determined the position) to load.

**Basically**, when refresh happen, you don't want the scroll go up to the top

### Pager
Decide when to fetch(like pageSize, initiaLoadSize)
It also manage: caching, invalidation, etc
### Flow
**Pager(...).flow** which creates Flow<PagingData\<Stock>>
PagingData: a lazy container for paginated items
### LazyPagingItems (UI layer)
Again, Compose does NOT understand, handle Flow, so pagerFlow.collectAsLazyPagingItems() get StateFlow from Flow which is converted by .**collect**{} to **LazyPagingItems**\<Stock>
which Compose understands **LazyPagingItems** also **expose load states**: stocks.**loadState** -> cover: refresh, append, prepend, error