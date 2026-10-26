## Android Jetpack, Modern Architecture, MVVM, Room Database, Reactive Programming & Unit-IV Revision

**Technology focus:** Kotlin + XML Views  
**Sessions:** S_23 and S_24  
**Purpose:** University syllabus notes with practical examples, architecture intuition, revision points, and interview preparation.

---

# Session 23 — Android Jetpack, Modern Architecture & MVVM

## 1. Learning outcomes

By the end of this session, students should be able to:

- Explain how Android application architecture has evolved.
- Describe Android Jetpack and identify its important components.
- Explain Navigation, Data Binding, and WorkManager at a high level.
- Compare MVC, MVP, and MVVM.
- Explain Model, View, ViewModel, ViewModel, and LiveData responsibilities.
- Describe why modern Android architecture improves maintainability and testability.

## 2. Evolution of Android application architecture

Early Android apps often placed UI handling, business logic, and data operations together in an `Activity`. This can be acceptable for a tiny demo, but as the app grows, the Activity becomes difficult to understand, test, and maintain.

### A common progression

1. **UI and logic mixed together:** An Activity handles clicks, validation, database operations, and screen updates.
2. **Responsibilities separated:** Code is divided into UI, business logic, and data responsibilities.
3. **Architecture patterns adopted:** MVC, MVP, and MVVM provide clearer boundaries.
4. **Lifecycle-aware components and libraries:** Jetpack components help manage common Android tasks more consistently.

### Example: a login screen

Without separation, `LoginActivity` may validate input, query the database, call a network service, and update every UI element. With a clearer architecture:

- **View:** Displays fields, loading state, and messages.
- **ViewModel / presenter / controller:** Coordinates UI-related logic, depending on the architecture.
- **Model / data layer:** Represents and retrieves user data.

**Key idea:** Architecture is not just about folders. It is about assigning each responsibility to the right place.

## 3. Android Jetpack

Android Jetpack is a collection of libraries, tools, and guidance designed to help developers build robust Android apps with less repetitive code and more consistent lifecycle handling.

Jetpack is not one single library. It contains components that solve different problems.

### Important Jetpack components overview

| Component | Main purpose | Example use |
|---|---|---|
| ViewModel | Holds screen-related state across configuration changes | Keep search results when the screen rotates |
| LiveData | Lifecycle-aware observable data holder | Update UI when student data changes |
| Room | Abstraction over SQLite | Store student records locally |
| Navigation | Manage app navigation and back stack | Move from Home to Student Details |
| Data Binding | Bind UI layout views to data sources | Display a student's name in XML |
| WorkManager | Schedule deferrable, reliable background work | Upload pending data when constraints allow |
| Lifecycle | Help components react to lifecycle events | Avoid observing UI data after an Activity is stopped |
| Paging | Load large datasets in chunks | Display thousands of records efficiently |

**Scope note:** This session introduces these components conceptually. The practical depth is focused on Navigation, Data Binding, WorkManager, MVVM, ViewModel, and LiveData.

## 4. Navigation Component

The Navigation Component helps organize movement between screens or destinations. It can manage the navigation back stack and support a more structured navigation flow.

### Core terms

- **Destination:** A screen or location in the app.
- **Navigation graph:** Describes destinations and actions between them.
- **NavController:** Performs navigation actions.
- **NavHost:** A container that displays the current destination.

### Example flow

`StudentListFragment → StudentDetailsFragment`

The user taps a student row; the app navigates to the details screen.

### Typical setup concepts

1. Add the Navigation dependencies appropriate to the project.
2. Create a navigation graph resource, usually in `res/navigation/`.
3. Define destinations and actions in the graph.
4. Place a `NavHostFragment` in the host Activity layout.
5. Use a `NavController` to navigate.

Illustrative Kotlin call from a Fragment:

```kotlin
// Illustrative: requires a matching action or destination in nav_graph.xml
findNavController().navigate(R.id.action_studentList_to_studentDetails)
```

**Remember:** The ID must exist in the navigation graph, and the Fragment must be set up with the Navigation Component. This snippet alone is not a complete project.

### Why use it?

- Makes screen flow easier to understand.
- Centralizes navigation relationships.
- Helps manage back-stack behavior.
- Reduces scattered navigation code in larger apps.

## 5. Data Binding concept

Data Binding lets XML layouts bind UI elements to data objects and expressions. With it enabled, a layout can reference a variable declared in its `<data>` section.

### Illustrative XML

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data>
        <variable
            name="studentName"
            type="String" />
    </data>

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@{studentName}" />
</layout>
```

When the binding variable is assigned a value, the TextView can display it.

### Illustrative Kotlin

```kotlin
val binding = ActivityStudentBinding.inflate(layoutInflater)
setContentView(binding.root)

binding.studentName = "Aarav"
```

The generated binding class name depends on the XML filename (`activity_student.xml` produces `ActivityStudentBinding` when Data Binding is configured).

### Important points

- Data Binding must be enabled in the module's Gradle configuration.
- The XML root uses `<layout>`.
- Data Binding and View Binding are different:
  - **View Binding** generates type-safe references to views.
  - **Data Binding** additionally supports binding expressions and data variables in XML.
- Avoid placing complex business logic in XML expressions.

## 6. WorkManager overview

WorkManager is a Jetpack library for scheduling **deferrable, reliable background work** that should continue according to its constraints, including across app restarts and device reboots when supported by the system.

### Example use cases

- Upload queued feedback when a network connection becomes available.
- Synchronize pending app data.
- Perform periodic cleanup.
- Retry a background task after a temporary failure.

It is not designed for every kind of background task. Immediate, user-visible work may require a different API depending on the requirement and Android version.

### Core terms

- **Worker / CoroutineWorker:** Contains the work to perform.
- **WorkRequest:** Describes the work and its constraints.
- **WorkManager:** Enqueues and manages the request.
- **Constraints:** Conditions such as requiring network connectivity.
- **Result:** A worker can report success, failure, or retry.

### Illustrative Kotlin example

```kotlin
class UploadWorker(
    appContext: Context,
    params: WorkerParameters
) : Worker(appContext, params) {

    override fun doWork(): Result {
        // Perform a short, reliable background upload here.
        return Result.success()
    }
}
```

Enqueueing the worker:

```kotlin
val uploadRequest = OneTimeWorkRequestBuilder<UploadWorker>()
    .setConstraints(
        Constraints.Builder()
            .setRequiredNetworkType(NetworkType.CONNECTED)
            .build()
    )
    .build()

WorkManager.getInstance(applicationContext).enqueue(uploadRequest)
```

Required imports and Gradle dependencies are omitted for readability; add the WorkManager dependency before using this example.

**Remember:** WorkManager is for reliable, deferrable work—not a promise that work begins at an exact clock time.

## 7. Architecture patterns: MVC, MVP, and MVVM

Architecture patterns separate responsibilities so code is easier to maintain and test. Different patterns assign UI-related logic differently.

### MVC — Model, View, Controller

- **Model:** Data and domain rules.
- **View:** UI shown to the user.
- **Controller:** Receives user input and coordinates the Model and View.

**Android consideration:** In simple Android implementations, an Activity can become both the View and Controller, causing it to grow too large if responsibilities are not separated carefully.

### MVP — Model, View, Presenter

- **Model:** Data and business operations.
- **View:** UI contract and rendering.
- **Presenter:** Handles presentation logic and tells the View what to display.

The View generally delegates user actions to the Presenter. A Presenter can often be unit-tested without creating the actual UI.

### MVVM — Model, View, ViewModel

- **Model:** Data and domain operations.
- **View:** Activity, Fragment, or UI layout.
- **ViewModel:** Holds and prepares UI state; exposes data the View can observe.

### Comparison

| Aspect | MVC | MVP | MVVM |
|---|---|---|---|
| Main coordinator | Controller | Presenter | ViewModel |
| View communication | Controller coordinates View and Model | View calls Presenter; Presenter updates View contract | View observes ViewModel state |
| UI coupling | Can become high in simple Android implementations | Presenter often depends on a View interface | ViewModel should not hold Activity/View references |
| Testing | Depends on separation | Presenter logic can be tested separately | ViewModel logic can be tested separately |
| Typical challenge | Controller/Activity becomes large | Many View interfaces and manual update calls | State/lifecycle handling must be designed clearly |

No pattern automatically guarantees good code. Good responsibility boundaries matter more than the name of the pattern.

## 8. MVVM in detail

MVVM is a common architecture approach for Android applications. It separates screen rendering from the logic that prepares screen state.

### Responsibilities

**Model**
- Represents application data and business/domain operations.
- May include repositories, database access, and network data sources.
- Should not directly manipulate UI widgets.

**View**
- Usually an Activity or Fragment.
- Displays the current state.
- Sends user actions to the ViewModel.
- Observes state and updates UI elements.

**ViewModel**
- Stores and prepares UI-related state.
- Responds to user actions.
- Calls the data/repository layer when required.
- Should not store references to Activities, Fragments, or Views.

### Flow

```text
User action
    ↓
View (Activity / Fragment)
    ↓ calls
ViewModel
    ↓ requests data
Repository / Model
    ↓ returns result
ViewModel updates observable state
    ↓ observed by
View updates the screen
```

This is a conceptual flow. A repository is commonly used but is not a mandatory part of the acronym MVVM itself.

### Why ViewModel?

Android may recreate an Activity or Fragment during configuration changes such as rotation. A `ViewModel` is designed to retain screen-related state across these configuration changes, until its associated scope is cleared.

A ViewModel is **not** a replacement for persistent storage. It does not guarantee that state survives process death or that data is permanently saved.

### Simple ViewModel example

```kotlin
class CounterViewModel : ViewModel() {
    private val _count = MutableLiveData(0)
    val count: LiveData<Int> = _count

    fun increment() {
        _count.value = (_count.value ?: 0) + 1
    }
}
```

### Observing LiveData in an Activity

```kotlin
private lateinit var viewModel: CounterViewModel

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    // setContentView(...) should be called before accessing layout views

    viewModel = ViewModelProvider(this)[CounterViewModel::class.java]

    viewModel.count.observe(this) { newCount ->
        // Example: update a TextView using the newCount value
        countTextView.text = newCount.toString()
    }

    incrementButton.setOnClickListener {
        viewModel.increment()
    }
}
```

This is an illustrative snippet: define `countTextView` and `incrementButton` from the Activity's layout, and include the lifecycle/ViewModel dependencies used by the project.

## 9. LiveData concept

`LiveData` is an observable data holder that is lifecycle-aware. An Activity or Fragment can observe it, and updates are delivered when the observer is in an appropriate active lifecycle state.

### Terms

- `LiveData<T>`: Read-only observable value exposed to observers.
- `MutableLiveData<T>`: Mutable version that the ViewModel can update.
- `observe(owner) { ... }`: Registers an observer tied to a lifecycle owner.

### Why expose LiveData as read-only?

```kotlin
private val _name = MutableLiveData("Aarav")
val name: LiveData<String> = _name
```

The ViewModel controls changes through `_name`, while the View can observe `name` without directly modifying it. This helps maintain a clear boundary.

### Benefits

- Lifecycle-aware observation.
- Less manual UI refresh wiring.
- Helps the UI react to state changes.
- Pairs naturally with ViewModel in traditional XML-based apps.

**Note:** LiveData is not the only option for observable state. Kotlin Flow is also widely used in modern Android development, but this session's syllabus specifically includes LiveData and RxJava/RxAndroid.

## 10. Benefits of modern Android architecture

- **Separation of concerns:** UI rendering and data operations have clearer roles.
- **Testability:** ViewModel or Presenter logic can be tested independently.
- **Maintainability:** Smaller classes are easier to change.
- **Lifecycle awareness:** Components help handle Activity/Fragment lifecycle changes.
- **Reusability:** Data and business logic can be reused across screens.
- **Scalability:** New features can be added without putting every responsibility into one Activity.

### Common mistakes to avoid

- Putting all database and network code in an Activity.
- Keeping a reference to an Activity inside a ViewModel.
- Treating ViewModel as permanent storage.
- Putting complex business logic inside XML binding expressions.
- Assuming every app needs every Jetpack library.
- Adding architecture layers without a real reason; keep the design proportionate to the app.

---

# Session 24 — Room Database, Reactive Programming & Unit-IV Revision

## 11. Limitations of direct SQLite handling

Android provides SQLite for local relational data. Direct SQLite APIs are useful, but they require developers to handle many details manually.

Common difficulties include:

- Writing SQL strings and mapping `Cursor` columns to Kotlin objects.
- Managing database helper code and schema changes.
- Keeping table and column names consistent.
- Closing cursors and handling resources correctly.
- Avoiding errors caused by typos in SQL or column names.
- Repeating CRUD boilerplate.
- Testing data access logic cleanly.

Room does not replace SQLite as the underlying database engine. Room is an abstraction layer that generates and checks much of the database-access code.

## 12. Introduction to Room Persistence Library

Room is an Android Jetpack persistence library that provides an abstraction layer over SQLite. It uses annotations and compile-time code generation to help define tables and database operations.

### Main Room components

1. **Entity:** Defines a database table.
2. **DAO (Data Access Object):** Defines methods for database operations.
3. **RoomDatabase:** Main database holder and access point.

### Room architecture

```text
Activity / Fragment
        ↓
    ViewModel
        ↓
      DAO  ← defined by Room
        ↓
   RoomDatabase
        ↓
      SQLite
```

In a larger app, a Repository is commonly placed between ViewModel and DAO to separate data sources from UI logic. The diagram is a simplified learning model.

## 13. Entity

An Entity represents a table. Each object represents a row, and its properties represent columns.

### Example: `Student.kt`

```kotlin
import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity(tableName = "students")
data class Student(
    @PrimaryKey(autoGenerate = true)
    val id: Int = 0,
    val name: String,
    val course: String
)
```

**Explanation**
- `@Entity` tells Room to create a table for this class.
- `tableName` sets the database table name.
- `@PrimaryKey` marks the unique identifier.
- `autoGenerate = true` lets Room generate integer IDs for new rows.

Room supports common Kotlin/Java field types; special types may need a type converter.

## 14. DAO — Data Access Object

A DAO defines the operations used to access a database. Room generates the implementation based on its annotations.

### Example: `StudentDao.kt`

```kotlin
import androidx.room.Dao
import androidx.room.Insert
import androidx.room.Query
import androidx.room.Delete
import androidx.room.Update

@Dao
interface StudentDao {

    @Insert
    suspend fun insertStudent(student: Student)

    @Query("SELECT * FROM students ORDER BY name")
    suspend fun getAllStudents(): List<Student>

    @Update
    suspend fun updateStudent(student: Student)

    @Delete
    suspend fun deleteStudent(student: Student)
}
```

### What the annotations mean

- `@Dao`: Marks an interface or abstract class as a DAO.
- `@Insert`: Inserts a row.
- `@Query`: Runs a defined SQL query.
- `@Update`: Updates a matching row, typically identified by its primary key.
- `@Delete`: Deletes a matching row.

The `suspend` functions should be called from a coroutine. Database operations should not block the main/UI thread.

## 15. Room Database

The database class connects entities and DAOs.

### Example: `AppDatabase.kt`

```kotlin
import android.content.Context
import androidx.room.Database
import androidx.room.Room
import androidx.room.RoomDatabase

@Database(
    entities = [Student::class],
    version = 1,
    exportSchema = false
)
abstract class AppDatabase : RoomDatabase() {

    abstract fun studentDao(): StudentDao

    companion object {
        @Volatile
        private var INSTANCE: AppDatabase? = null

        fun getDatabase(context: Context): AppDatabase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext,
                    AppDatabase::class.java,
                    "college_database"
                ).build()

                INSTANCE = instance
                instance
            }
        }
    }
}
```

### Key points

- `@Database` lists the entities and database version.
- `studentDao()` gives access to the DAO.
- `Room.databaseBuilder()` constructs the database.
- `applicationContext` avoids holding an Activity context for the lifetime of the database.
- The singleton pattern shown here helps avoid creating multiple database instances in a simple app.
- In a production app, configure schema export and migrations thoughtfully. Do not casually increase the version without a migration strategy.

## 16. Calling Room from a coroutine

Illustrative usage from a ViewModel:

```kotlin
class StudentViewModel(
    private val studentDao: StudentDao
) : ViewModel() {

    fun addStudent(name: String, course: String) {
        viewModelScope.launch {
            studentDao.insertStudent(
                Student(name = name, course = course)
            )
        }
    }
}
```

This example shows the idea, not a complete app setup. A real project needs a suitable ViewModel factory or dependency-injection setup to provide `StudentDao`. Add the `lifecycle-viewmodel-ktx`/coroutines support appropriate to your project and import `viewModelScope` and `launch`.

For XML-based screens, the Activity or Fragment can observe ViewModel state and render the list. Keep the database work out of the UI class.

## 17. Advantages of Room

- Compile-time checking for many SQL queries.
- Less repetitive cursor and mapping code.
- Clear entities and DAO contracts.
- Integrates with coroutines and observable data patterns.
- Supports database migrations.
- Easier to test data-access components.
- Fits naturally into MVVM-style applications.

### Direct SQLite vs Room

| Feature | Direct SQLite APIs | Room |
|---|---|---|
| SQL execution | Mostly written and managed manually | SQL queries declared in annotated methods |
| Row mapping | Often manual `Cursor` mapping | Room maps results to declared types |
| Query checking | Many errors appear at runtime | Many query issues can be caught at compile time |
| Boilerplate | More manual helper and cursor code | Less repetitive code |
| Database engine | SQLite | SQLite underneath |
| Schema migration | Developer-managed | Supported through migration APIs; strategy still required |

**Important:** Room does not eliminate the need to understand SQL, table design, keys, indexes, or migrations.

## 18. Introduction to reactive programming

Reactive programming focuses on working with values or events that arrive over time. Instead of only asking for a value once, a program can react when a new value or event is emitted.

### Everyday examples

- Search suggestions update as the user types.
- A download progress indicator changes over time.
- A data stream emits new results when local data changes.
- A button click or network response triggers the next operation.

### Why reactive programming?

- Handles asynchronous event streams.
- Supports composing operations on streams of data.
- Can help coordinate background work and UI updates.
- Reduces some kinds of callback nesting when used carefully.

Reactive code can still be hard to understand if streams, errors, threading, and disposal are not managed clearly.

## 19. RxJava fundamentals

RxJava is a library for composing asynchronous and event-based programs using observable sequences. RxAndroid provides Android-specific support, including a scheduler that can post work to the Android main thread.

### Core concepts

- **Observable:** Produces a sequence of values or events.
- **Observer:** Receives values and terminal notifications.
- **Operators:** Transform, filter, combine, or otherwise compose emissions.
- **Scheduler:** Controls where work is scheduled.
- **Subscription / Disposable:** Represents a connection that may need to be disposed to stop observing and release resources.

### Observable and Observer

An Observable may emit:
1. Zero or more values.
2. Either a completion notification or an error notification.

An Observer can receive:
- `onNext(value)` — a new value.
- `onError(error)` — the stream failed.
- `onComplete()` — the stream completed successfully.

### Illustrative RxJava example

```kotlin
Observable.just("Android", "Kotlin", "Room")
    .subscribe(
        { value -> println(value) },
        { error -> println("Error: ${error.message}") },
        { println("Completed") }
    )
```

This emits three values and then completes. It is a learning example; `println` is not a UI update mechanism.

## 20. Operators overview

Operators transform or control a stream.

| Operator | Purpose | Example idea |
|---|---|---|
| `map` | Transform each emitted value | Convert a name to uppercase |
| `filter` | Keep values matching a condition | Keep scores above 50 |
| `take` | Limit number of emissions | Take first three results |
| `distinctUntilChanged` | Ignore consecutive duplicate values | Avoid repeating the same search query |
| `flatMap` | Map each item to another stream and merge results | For each user, request related data |
| `debounce` | Wait for a pause before emitting the latest value | Avoid a search request on every keystroke |

Operators have precise behavior; for example, `flatMap` can interleave results, so it should not be treated as a guaranteed ordered sequence.

### Example

```kotlin
Observable.just(10, 20, 30, 40)
    .filter { it >= 20 }
    .map { it / 10 }
    .subscribe { result ->
        println(result)
    }
```

Output:

```text
2
3
4
```

First, values below 20 are filtered out; then the remaining values are divided by 10.

## 21. Scheduler concept

A Scheduler determines where or when work is executed. It is similar in purpose to choosing an execution context for part of a stream.

Common RxJava schedulers include:

- `Schedulers.io()` — commonly used for I/O-bound work.
- `Schedulers.computation()` — commonly used for CPU-intensive computation.
- `Schedulers.single()` — uses a single-threaded scheduler.
- `AndroidSchedulers.mainThread()` — RxAndroid scheduler for Android's main thread.

### Illustrative example

```kotlin
Observable.fromCallable {
    // Perform a background calculation or data operation
    "Data loaded"
}
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe { result ->
        // Update a UI element here
    }
```

- `subscribeOn(...)` controls where the upstream work begins.
- `observeOn(...)` changes the scheduler used for downstream notifications from that point.
- UI updates should happen on the main thread.

**Important:** In a real app, keep the returned `Disposable` and dispose it at the appropriate lifecycle point. Do not assume a subscription automatically ends when an Activity is destroyed.

## 22. RxAndroid overview

RxAndroid extends RxJava with Android-focused integrations. Its best-known feature is `AndroidSchedulers.mainThread()`, which schedules notifications on the main/UI thread.

### Typical flow

```text
Source Observable
      ↓
subscribeOn(IO)
      ↓
Background work
      ↓
observeOn(Main Thread)
      ↓
Update UI
```

Use RxAndroid when a project uses RxJava and needs Android scheduler integration. It is not a mandatory part of every Android app. Kotlin Coroutines and Flow are other common asynchronous/reactive tools in modern Android projects.

### Common reactive-programming mistakes

- Doing long-running work on the main thread.
- Updating views from a background thread.
- Ignoring stream errors.
- Forgetting to dispose subscriptions when needed.
- Creating streams with no clear owner or lifecycle.
- Using complex operators without explaining why they are needed.

---

# Unit-IV Revision

## 23. Security revision

Review these concepts from Session 21:

- Android app security and secure coding.
- Normal vs dangerous permissions.
- Runtime permission requests for applicable permissions.
- Authentication vs authorization.
- Safe handling of credentials and sensitive data.
- Least-privilege principle.
- Avoid logging secrets or tokens.
- Validate inputs and handle failures safely.

**Quick distinction:** Authentication asks “Who are you?” Authorization asks “What are you allowed to do?”

## 24. Debugging revision

- Use Logcat to inspect logs and exceptions.
- Read the exception type and the first relevant application stack-trace line.
- Set breakpoints and inspect variables with the debugger.
- Reproduce the issue consistently.
- Check null values, incorrect IDs, lifecycle timing, and thread usage.
- Fix the underlying cause rather than hiding the error.

## 25. Testing revision

- **Unit test:** Tests a small unit of logic in isolation.
- **Instrumented test:** Runs on an Android device or emulator and can use Android framework APIs.
- Test normal cases, boundary cases, and error cases.
- Keep tests repeatable and independent.
- ViewModel logic and data-access behavior are useful test targets.

## 26. Jetpack revision

| Component | Remember |
|---|---|
| ViewModel | Screen state holder; survives configuration changes within its scope |
| LiveData | Lifecycle-aware observable holder |
| Room | Abstraction over SQLite using Entity, DAO, and Database |
| Navigation | Structured navigation graph and back stack |
| Data Binding | XML expressions bind views to data |
| WorkManager | Reliable, deferrable background work with constraints |

## 27. MVVM revision

- **Model:** Data and domain operations.
- **View:** Activity/Fragment; renders state and forwards user actions.
- **ViewModel:** Prepares and exposes UI state.
- View observes ViewModel data.
- ViewModel should not hold Activity or View references.
- Use persistent storage when data must survive process death or app restarts.

## 28. Room revision

- `@Entity` defines a table.
- `@PrimaryKey` defines a unique key.
- `@Dao` defines database operations.
- `@Insert`, `@Query`, `@Update`, and `@Delete` declare operations.
- `@Database` defines the database and entity list.
- Room uses SQLite underneath.
- Database operations should not block the main thread.
- Schema changes require a migration strategy.

## 29. Reactive programming revision

- Observable emits values or events.
- Observer receives values, errors, and completion.
- Operators transform or filter emissions.
- Scheduler controls where work runs.
- RxAndroid provides Android-specific integration, especially the main-thread scheduler.
- Manage errors and subscription disposal.

## 30. Important interview questions with short answers

### Q1. What is Android Jetpack?
Android Jetpack is a collection of libraries and tools that helps developers build maintainable Android applications while handling common tasks consistently.

### Q2. What is the purpose of a ViewModel?
A ViewModel stores and prepares UI-related state and survives configuration changes while its scope remains alive. It is not permanent storage.

### Q3. What is LiveData?
LiveData is a lifecycle-aware observable data holder that allows active UI observers to react to data changes.

### Q4. Differentiate MVC, MVP, and MVVM.
MVC uses a Controller, MVP uses a Presenter, and MVVM uses a ViewModel whose state is observed by the View. Their main difference is how presentation logic and UI communication are organized.

### Q5. Why should a ViewModel not hold an Activity reference?
A ViewModel may outlive an Activity instance during configuration changes. Holding the old Activity can cause leaks and incorrect UI references.

### Q6. What is the Navigation Component?
It is a Jetpack library that helps define destinations and navigation actions and manage navigation/back-stack behavior.

### Q7. What is Data Binding?
Data Binding allows layout XML to bind views to data variables and expressions. View Binding mainly provides type-safe view references and does not provide the same XML expression system.

### Q8. What is WorkManager used for?
It schedules reliable, deferrable background work that can honor constraints such as network availability.

### Q9. What is Room?
Room is a Jetpack persistence library that provides an abstraction over SQLite and generates database-access implementations.

### Q10. Name the three main components of Room.
Entity, DAO, and RoomDatabase.

### Q11. What is a DAO?
A Data Access Object defines methods for querying and modifying data without putting database operations directly in UI classes.

### Q12. Does Room replace SQLite?
No. Room provides an abstraction layer over SQLite.

### Q13. Why should database operations avoid the main thread?
Long database operations can block UI rendering and cause an unresponsive interface.

### Q14. What is reactive programming?
It is a programming approach where code responds to values or events emitted over time, often using composable streams.

### Q15. What is the difference between Observable and Observer?
An Observable produces a sequence of events; an Observer receives those events and terminal notifications.

### Q16. What is the purpose of `map` and `filter`?
`map` transforms each emitted value; `filter` allows only values matching a condition to pass through.

### Q17. What is a Scheduler in RxJava?
A Scheduler determines the execution context for stream work or notifications.

### Q18. Why use `AndroidSchedulers.mainThread()`?
It schedules downstream notifications on Android's main thread so UI updates can be performed safely.

### Q19. What is the difference between ViewModel and Room?
ViewModel manages screen-related state and presentation logic; Room manages structured local persistence through SQLite.

### Q20. Why is architecture important?
It separates responsibilities, making code easier to understand, test, extend, and maintain.

---

# Practice tasks

1. Draw an MVC, MVP, and MVVM diagram for a Student Registration screen.
2. Explain how a ViewModel and LiveData work together when a counter button is clicked.
3. Write an Entity and DAO for a `Course` table with `id`, `title`, and `credits`.
4. Explain why Room is safer and easier to maintain than manually mapping every SQLite Cursor.
5. Write an RxJava stream that filters numbers greater than 10 and doubles them.
6. Explain when WorkManager is appropriate and why it should not be treated as an exact-time scheduler.
7. Compare Room, SharedPreferences, and direct SQLite for different data-storage needs.
8. Give three examples of bugs that Logcat or breakpoints can help diagnose.
9. Write a short testing plan for a Student CRUD screen.
10. Summarize the Unit-IV concepts in your own words without reading the notes.

## Final takeaway

Modern Android development is easier to maintain when UI, screen state, data access, and asynchronous work have clear responsibilities. Jetpack provides libraries for common tasks; MVVM separates screen state from UI rendering; Room reduces repetitive SQLite handling; and reactive tools help compose asynchronous event streams. Use each tool for the problem it is designed to solve, and keep the architecture simple enough for the application.
