
What make Kotlin noticeable different from other function
Null safety
val vs var
Type inference: give value and kotlin knows its type
Higher-order functions / lambdas: Kotlin treat function as value
Data classes
Extension function
Couroutines
Smart casts: Kotlin can automatically understand the type after checking it.
Properties instead of getters/setters

**Uppercase** is for class type
**Lowercase** is for function

#Kotlin 

|Concept|Kotlin|What it is|
|---|---|---|
|**List**|`List<T>` / `MutableList<T>`|Ordered collection|
|**Set**|`Set<T>` / `MutableSet<T>`|Unique elements|
|**Dictionary / Map**|`Map<K, V>` / `MutableMap<K, V>`|Key → value|
|**Deque**|`ArrayDeque<T>`|Add/remove from both ends|
|**Stack**|`ArrayDeque<T>`|Usually use `ArrayDeque`|
|**Queue**|`ArrayDeque<T>`|Usually use `ArrayDeque`|
|**String**|`String`|Sequence of characters|


#Kotlin 
`it` is **Kotlin’s default name for a lambda parameter**  
**when there is exactly ONE parameter**

#Kotin **DataType? - String?** - allow data to be null
Safe null call: 
```
val name: String? = null

println(name?.length)
```
⚠️when calling its property/function/member like **length**: 
**MUST have ?** to avoid crash null ->>> **name?.length**

#Kotlin let{}: execute code only if object is not null
```
name?.let {
 println("The name length is ${it.length}")
}
```


#Kotlin **Elvis operator ?:**  - concise way to provide default value for null variable
🔥Usually used with nullable data to provide default value - avoid null

#Kotlin **onQueryChange: (String) -> Unit**
- Take a String as input
- Return Unit (nothing)

#Kotlin **mutable**: can modify orginial data structure

| Type              | Can add/remove elements? |
| ----------------- | ------------------------ |
| `listOf()`        | ❌ No                     |
| `mutableListOf()` | ✅ Yes                    |

#Kotlin **object**: Only 1 instance exist.
Ex: object Screen
#Kotlin **companion object**: Static memebers in Java


#Kotlin **sealed**: used to create restricted class hierarchies where all direct subclasses are **known at compile time**

**sealed class**: when to use? when all children share data or behavior. `sealed class Screen`
**sealed interface**: when to use? when you have mixed of state instead of one normal data class `sealed interface UiState`

⭐using `when` with `sealed` ensures you handle **every** state


`lazy initialization` is when object creation is purposely delayed, until you actually need that object, to avoid unnecessary computation or use of other computing resources
Android allows local http connect, but not http from public DNS(like ec2)

#Kotlin **when**: similar to switch statement but without the need of break, used to execute different code blocks based on a matching condition and can function as either a **statement** (performing an action) or an **expression** (returning a value).
***When used as an **expression**, the compiler requires you to handle **every possible** case, often necessitating an `else` branch.

#Kotlin l**istOf():** return read-only list containing only specific object relement(immutable)

#Kotlin Result<\T>: return Sucess or Failure, simple and typically used on Repos layer, but not recommended because you have to unwrap it everywhere
⭐ Better approach is not use it, instead use try/catch to handle error if occur

#Kotin You can add extension funtions to almost any class in Kotlin, Android or third-party libraries.
Ex:
```
fun NavBackStackEntry.expenseFlowViewModel( navController: NavHostController )
text.uppercase()
list.map { }
```

#Kotlin operator fun invoke()
Commonly used in UseCase
Kotlin translate object() into object.invoke() -> this make calling UseCase class more concise
```
class GetUserUseCase( 
	private val repository: UserRepository
) { 
	suspend operator fun invoke(id: String): User { 
		return repository.getUser(id) 
	}
}

// usage
val user = getUserUseCase(id)
```

`operator fun invoke()` allows a UseCase object to be called like a function, making the calling code more concise and readable. Instead of `loginUseCase.execute()`, you can write `loginUseCase()`. It doesn't change the behavior of the UseCase—it simply improves the API while still allowing the UseCase to encapsulate business logic and receive dependencies through dependency injection."

#Kotlin `operator fun` works only with specific names
```
operator fun plus(...)
operator fun minus(...)
operator fun times(...)
operator fun div(...) 
operator fun get(...) 
operator fun set(...) 
operator fun contains(...)
operator fun compareTo(...)
operator fun iterator(...)
operator fun invoke(...)
```

