# Test in Android

kotlinx-coroutines-test

Mockito
MockK - MockK does the same thing as Mockito but is designed specifically for Kotlin
Give complete control over dependencies of a class(ex: ViewModel)
You control every return value, exception, and can verify interaction

Compose UI Test - Instead of checking state directly, you interact with the UI like a user.
## 1/ Android Test - Instrumentation Test
Run on emulator
Tests that **need Android framework**
- UI testing
- Context, Activities, Views
- Navigation, database, etc.
Types of androidTest:
- UI Testing (most common) - Espresso
- Jetpack Compose Testing - 
- Integration Testing - test multiple layers together (UI + ViewModel + Repository)

**@HiltAndroidTest**: enable Hilt in Test - allow inject fake dependencies
**@RunWith**(AndroidJUnit4::class): Run test on emulator
#### 🧩 `HiltAndroidRule`
- Initializes dependency injection before tests
- After initialize, make sure to inject before each test
#### 🧩 `createAndroidComposeRule<MainActivity>()`
- Launches your **MainActivity**
- Lets you interact with Compose UI

⚠️ Should not use thread.sleep() because its slow down testing time. better use compose test sync
or waitUntil with UI condition
- add fake api, fake db to make the test more stable, valuable

### Compose Testing Structure
#### Start with
```
@get:Rule  
val composeTestRule = createAndroidComposeRule<MainActivity>()
```


## 2/ Unit Test
No emulator, UI needed, just Kotlin
Test business logic
Run on JVM

**@Test**: marks function as a test
**@Before**: runs before every test

#### Stub behavior – define how the mocked dependency should behave when called.
- **What a function should do/return**

**every {}**: used for normal (**non-suspend**) functions
```
every {  
	repository.getUserName()  
} returns "Alice"
```

**coEvery {}**: used for **suspend** functions ⭐⭐⭐
```
coEvery { 
	repository.getProblemDetail(problemId) 
} returns sampleProblemDetai
```

**returnsMany**: sometimes a function is called multiple times - returnsMany can capture all that
```
every {  
	repository.getValue()  
} returnsMany listOf(  
	1,  
	2,  
	3  
)
```

**throws**: simulates an exception
```
coEvery {
	repository.getProblemDetail(problemId)
} throws RuntimeException(errorMsg) // Simulate API/database/network failure.
```

#### Verify interaction - did my code call function as expected?
**verify {}**: checks that a normal (**non-suspend**) function was called
```
verify {  
	analytics.logScreen()  
}
```

**coVerify {}**: check that **suspend** function was called ⭐⭐⭐
❗**Notes**: This does **not** check the return value
```
coVerify {
	repository.getProblemDetail(problemId)
}
```
**coVerify(exactly = 1) {}**: Must be called exactly once

#### Assert - check result or state
- **assertEquals**(): checks that 2 values are equal
```
assertEquals(
    expected = "Hello",
    actual = uiState.joke
)   
```

- **assertTrue**() - **assertFalse**(): check value == true/false
`assertTrue(uiState.isLoading)`

- **assertNull**() - **assertNotNull**():  check value == null
```
assertNotNull(uiState.joke)
```


**runTest {}**: Runs coroutine-based tests.
```
@Test  
fun load() = runTest {  // can run suspend funs and coroutines
  
}
```

**advanceUntilIdle**(): Run **all pending** coroutine work until there's nothing left.


#### Argument test
**slot()**: capture what argument passed to a mock - > Useful when you care about **what argument** was passed.
```
// Actual code
repository.search("Username")

// Test
val querySlot = slot<String>()  

// When
coEvery {  
	repository.search(capture(querySlot))  
} returns emptyList()

// Then
assertEquals(  
	"Username",  
	querySlot.captured  
)
```

**any()**: match any argument - **don't care** what string/value is passed
```
coEvery {  
	repository.search(any())  
}
```

#### Order of function call test
**verifyOrder {}**: Check call order - can have extra call in between
```
verifyOrder {  
	repository.clearCache()  
	repository.load()  
	analytics.log()  
}
```

**verifySequence {}**: Check call order - **no extra calls** are allowed.
**confirmVerified()**: Verify there were no additional interactions beyond those already verified.
### ViewModel Test

**Step 1**: 
- Create the mock repository using **MockK** (not a real repository)
- Create ViewModel and pass fake repository
Ex:
```
@Before  
fun setup() {  
    civicsRepository = mockk()  
    statsRepository = mockk()  
    scheduleFlashcards = mockk()  
    
    viewModel = FlashCardViewModel(civicsRepository, statsRepository, scheduleFlashcards)  
}
```

**Step 2**:
- Write Test following pattern: Given -> When -> Then or Arrange -> Act -> Assert 
```
// Given - Arrange
val questions = listOf()
val stats = emptyList<FlashcardStats>()

coEvery { civicsRepository.getQuestionsByVersion(CivicVersion.VERSION_2025) } returns questions  
coEvery { statsRepository.getAllStats(CivicVersion.VERSION_2025) } returns stats  
every { scheduleFlashcards(any(), any()) } returns expectedFlashcards

// When - Act
viewModel.getFlashCard(CivicVersion.VERSION_2025, LanguageCode.EN)

// Then - Assert
assertEquals(expected = "Question 1", actual = result.first().question)
coVerify(exactly = 1) { civicsRepository.getQuestionsByVersion(CivicVersion.VERSION_2025) }
```


# 3/ Integration Test
Test multiple classes working together
Ex:
Example:

```
StockPriceFeed
↓
StockAlertManager
↓
AlertSubscriber
```

Question:
> When the feed publishes a new price, is the subscriber notified?