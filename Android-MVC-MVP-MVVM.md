# Android Architecture Patterns: MVC, MVP, MVVM - Complete Guide

**Comprehensive Reference for Android Development: Kotlin & Java**

---

## Table of Contents

1. [Architecture Patterns Overview](#1-overview)
2. [Why Architecture Patterns Matter](#2-why)
3. [MVC Pattern (Model-View-Controller)](#3-mvc)
4. [MVP Pattern (Model-View-Presenter)](#4-mvp)
5. [MVVM Pattern (Model-View-ViewModel)](#5-mvvm)
6. [Detailed Comparison](#6-comparison)
7. [When to Use Each Pattern](#7-when-to-use)
8. [Real-World Scenarios](#8-scenarios)
9. [Testing Each Pattern](#9-testing)
10. [Modern Android Stack (2026)](#10-modern-stack)
11. [Best Practices](#11-best-practices)
12. [Migration Strategies](#12-migration)
13. [Complete Code Examples](#13-examples)

---

## 1. Architecture Patterns Overview

### What are Architecture Patterns?

**Architecture patterns** are proven solutions for organizing code to separate **concerns** (UI, logic, data), making Android apps:

- ✅ **Maintainable** - Easy to modify and extend
- ✅ **Testable** - Unit test business logic independently
- ✅ **Scalable** - Add features without breaking existing code
- ✅ **Team-friendly** - Multiple developers work independently
- ✅ **Debuggable** - Isolate and fix issues quickly

### Core Problem: "God Activity"

**❌ WITHOUT Architecture Pattern:**

```kotlin
// BAD: Everything in one Activity
class BadActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        // UI Logic
        button.setOnClickListener {
            // Business Logic
            val name = editText.text.toString()
            if (name.isEmpty()) {
                // Validation Logic
                Toast.makeText(this, "Enter name", LENGTH_SHORT).show()
                return@setOnClickListener
            }
            
            // Network Logic
            Thread {
                val url = URL("https://api.example.com/users")
                val connection = url.openConnection() as HttpURLConnection
                // ... HTTP code
                
                // Database Logic
                val db = Room.databaseBuilder(this, AppDatabase::class.java, "db").build()
                db.userDao().insert(user)
                
                // UI Update Logic
                runOnUiThread {
                    textView.text = "User saved!"
                }
            }.start()
        }
    }
}
```

**Problems:**
1. ❌ **Untestable** - Can't unit test without Android framework
2. ❌ **Unmaintainable** - 500+ line Activities
3. ❌ **Memory leaks** - Thread references Activity
4. ❌ **Configuration changes** - Loses state on rotation
5. ❌ **No separation** - Everything mixed together

### Solution: Architecture Patterns

**✅ WITH Architecture Pattern:**

```
UI Layer (Activity/Fragment)
    ↓ Only displays data
Controller/Presenter/ViewModel Layer
    ↓ Handles business logic
Model Layer (Repository)
    ↓ Manages data sources
Data Layer (Room, Retrofit, Cache)
```

### Evolution Timeline

```
2010-2014: MVC (Default Android, Activities do everything)
2015-2017: MVP (Google I/O recommendations, Presenters introduced)
2017-2019: MVVM (Jetpack Architecture Components released)
2019-2024: MVVM + LiveData/StateFlow dominant
2024-2026: MVVM + Compose + Hilt (Current industry standard)
```

### The Three Patterns

| Pattern | **Controller** | **Data Flow** | **Testability** | **Complexity** |
|---------|----------------|---------------|-----------------|----------------|
| **MVC** | Activity | Two-way (View ↔ Controller) | Hard | Low |
| **MVP** | Presenter | One-way (View → Presenter → Model) | Medium | Medium |
| **MVVM** | ViewModel | One-way reactive (Model → ViewModel → View) | Easy | High |

---

## 2. Why Architecture Patterns Matter

### Real-World Impact

**Without patterns:**
- 🐌 **Slow development** - Adding features breaks existing code
- 🐛 **More bugs** - Tight coupling causes cascading failures
- 😤 **Team friction** - Merge conflicts in giant Activity files
- 🔥 **Tech debt** - Code becomes "unmaintainable legacy"

**With patterns:**
- ⚡ **Fast development** - New features in isolated modules
- 🛡️ **Fewer bugs** - Separation prevents side effects
- 👥 **Team harmony** - Developers work on different layers
- 🚀 **Scalability** - 10K+ line codebases stay organized

### Samsung R&D Context

**Perfect for your Samsung middleware projects:**

```
Daemon Service (System Software)
    ↓
MVVM Architecture
    ↓
ViewModel: Daemon state management
Repository: IPC + Binder communication
Model: System properties + Device state
    ↓
Local: SharedPreferences (persistent state)
Remote: System services (Binder IPC)
```

**Example: Storage Daemon**
- **Model**: Storage device status (mounted, capacity, health)
- **ViewModel**: State management (mounting, unmounting, notifications)
- **View**: Settings UI, Notification UI
- **Repository**: HAL communication, kernel events (udev)

---

## 3. MVC Pattern (Model-View-Controller)

### Architecture Overview

**MVC** is the **classic pattern** where the **Controller** (Activity/Fragment) handles ALL interactions between **View** (UI) and **Model** (Data).

```
┌─────────────────────────────────────────┐
│             MVC PATTERN                 │
├─────────────────────────────────────────┤
│                                         │
│   USER INPUT                            │
│       ↓                                 │
│   ┌────────┐         ┌──────────┐      │
│   │  VIEW  │ ←────── │  MODEL   │      │
│   │(Layout)│         │  (Data)  │      │
│   └────┬───┘         └─────▲────┘      │
│        │                   │            │
│        │    ┌──────────────┘            │
│        └───→│ CONTROLLER │              │
│             │ (Activity) │              │
│             └────────────┘              │
│                                         │
│   Data Flow: Two-way                    │
│   Controller updates both View & Model │
└─────────────────────────────────────────┘
```

### Components

**1. Model** - Data + Business Logic
```kotlin
// Plain data class
data class User(
    val id: Long,
    val name: String,
    val email: String
)

// Model class handles data operations
class UserModel {
    private val users = mutableListOf<User>()
    
    fun addUser(user: User) {
        users.add(user)
    }
    
    fun getUsers(): List<User> = users.toList()
    
    fun getUserById(id: Long): User? {
        return users.find { it.id == id }
    }
    
    fun deleteUser(id: Long) {
        users.removeIf { it.id == id }
    }
}
```

**2. View** - UI (XML + Activity displays it)
```xml
<!-- activity_main.xml -->
<LinearLayout>
    <EditText
        android:id="@+id/nameEditText"
        android:hint="Enter name"/>
    
    <EditText
        android:id="@+id/emailEditText"
        android:hint="Enter email"/>
    
    <Button
        android:id="@+id/addButton"
        android:text="Add User"/>
    
    <RecyclerView
        android:id="@+id/usersRecyclerView"/>
</LinearLayout>
```

**3. Controller** - Activity/Fragment (handles EVERYTHING)
```kotlin
// Activity acts as Controller
class MainActivity : AppCompatActivity() {
    
    private lateinit var model: UserModel
    private lateinit var adapter: UserAdapter
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        // Initialize Model
        model = UserModel()
        
        // Setup View
        adapter = UserAdapter()
        usersRecyclerView.adapter = adapter
        
        // Handle user input
        addButton.setOnClickListener {
            val name = nameEditText.text.toString()
            val email = emailEditText.text.toString()
            
            // Controller validates
            if (name.isEmpty() || email.isEmpty()) {
                Toast.makeText(this, "Fill all fields", LENGTH_SHORT).show()
                return@setOnClickListener
            }
            
            // Controller updates Model
            val user = User(
                id = System.currentTimeMillis(),
                name = name,
                email = email
            )
            model.addUser(user)
            
            // Controller updates View
            updateUI()
            
            // Controller clears input
            nameEditText.text.clear()
            emailEditText.text.clear()
        }
        
        updateUI()
    }
    
    private fun updateUI() {
        // Controller fetches from Model
        val users = model.getUsers()
        
        // Controller updates View
        adapter.submitList(users)
    }
}
```

### Complete Kotlin MVC Example (Todo App)

```kotlin
// ============ MODEL ============

data class Todo(
    val id: Long,
    val title: String,
    val isCompleted: Boolean = false,
    val createdAt: Long = System.currentTimeMillis()
)

class TodoModel {
    private val todos = mutableListOf<Todo>()
    
    fun addTodo(title: String): Long {
        val id = System.currentTimeMillis()
        todos.add(Todo(id, title, false))
        return id
    }
    
    fun toggleTodo(id: Long) {
        val index = todos.indexOfFirst { it.id == id }
        if (index != -1) {
            val todo = todos[index]
            todos[index] = todo.copy(isCompleted = !todo.isCompleted)
        }
    }
    
    fun deleteTodo(id: Long) {
        todos.removeIf { it.id == id }
    }
    
    fun getTodos(): List<Todo> = todos.toList()
    
    fun getCompletedCount(): Int = todos.count { it.isCompleted }
    
    fun getPendingCount(): Int = todos.count { !it.isCompleted }
}

// ============ VIEW (Layout) ============
// activity_todo.xml
/*
<LinearLayout>
    <EditText android:id="@+id/todoEditText"/>
    <Button android:id="@+id/addButton"/>
    <TextView android:id="@+id/statsTextView"/>
    <RecyclerView android:id="@+id/todosRecyclerView"/>
</LinearLayout>
*/

// ============ CONTROLLER (Activity) ============

class TodoActivity : AppCompatActivity() {
    
    // Model reference
    private lateinit var model: TodoModel
    private lateinit var adapter: TodoAdapter
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_todo)
        
        // Initialize Model
        model = TodoModel()
        
        // Setup RecyclerView
        adapter = TodoAdapter(
            onItemClick = { todo ->
                // Controller handles click
                model.toggleTodo(todo.id)
                updateUI()
            },
            onDeleteClick = { todo ->
                // Controller handles delete
                model.deleteTodo(todo.id)
                updateUI()
            }
        )
        todosRecyclerView.adapter = adapter
        
        // Controller handles add button
        addButton.setOnClickListener {
            val title = todoEditText.text.toString().trim()
            
            // Controller validates
            if (title.isEmpty()) {
                Toast.makeText(this, "Enter todo title", LENGTH_SHORT).show()
                return@setOnClickListener
            }
            
            // Controller updates Model
            model.addTodo(title)
            
            // Controller updates View
            updateUI()
            todoEditText.text.clear()
            
            // Controller shows feedback
            Toast.makeText(this, "Todo added", LENGTH_SHORT).show()
        }
        
        updateUI()
    }
    
    // Controller fetches from Model and updates View
    private fun updateUI() {
        val todos = model.getTodos()
        adapter.submitList(todos)
        
        val stats = "Completed: ${model.getCompletedCount()} | " +
                   "Pending: ${model.getPendingCount()}"
        statsTextView.text = stats
    }
}

// ============ ADAPTER ============

class TodoAdapter(
    private val onItemClick: (Todo) -> Unit,
    private val onDeleteClick: (Todo) -> Unit
) : RecyclerView.Adapter<TodoAdapter.ViewHolder>() {
    
    private var todos = emptyList<Todo>()
    
    fun submitList(newTodos: List<Todo>) {
        todos = newTodos
        notifyDataSetChanged()
    }
    
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.item_todo, parent, false)
        return ViewHolder(view)
    }
    
    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        holder.bind(todos[position])
    }
    
    override fun getItemCount() = todos.size
    
    inner class ViewHolder(view: View) : RecyclerView.ViewHolder(view) {
        private val titleText: TextView = view.findViewById(R.id.titleText)
        private val checkbox: CheckBox = view.findViewById(R.id.checkbox)
        private val deleteButton: ImageButton = view.findViewById(R.id.deleteButton)
        
        fun bind(todo: Todo) {
            titleText.text = todo.title
            checkbox.isChecked = todo.isCompleted
            
            titleText.paintFlags = if (todo.isCompleted) {
                titleText.paintFlags or Paint.STRIKE_THRU_TEXT_FLAG
            } else {
                titleText.paintFlags and Paint.STRIKE_THRU_TEXT_FLAG.inv()
            }
            
            checkbox.setOnClickListener { onItemClick(todo) }
            deleteButton.setOnClickListener { onDeleteClick(todo) }
        }
    }
}
```

### Complete Java MVC Example

```java
// ============ MODEL ============

public class Todo {
    private long id;
    private String title;
    private boolean isCompleted;
    private long createdAt;
    
    public Todo(long id, String title, boolean isCompleted) {
        this.id = id;
        this.title = title;
        this.isCompleted = isCompleted;
        this.createdAt = System.currentTimeMillis();
    }
    
    // Getters and setters
    public long getId() { return id; }
    public String getTitle() { return title; }
    public boolean isCompleted() { return isCompleted; }
    public void setCompleted(boolean completed) { isCompleted = completed; }
}

public class TodoModel {
    private List<Todo> todos = new ArrayList<>();
    
    public long addTodo(String title) {
        long id = System.currentTimeMillis();
        todos.add(new Todo(id, title, false));
        return id;
    }
    
    public void toggleTodo(long id) {
        for (Todo todo : todos) {
            if (todo.getId() == id) {
                todo.setCompleted(!todo.isCompleted());
                break;
            }
        }
    }
    
    public void deleteTodo(long id) {
        todos.removeIf(todo -> todo.getId() == id);
    }
    
    public List<Todo> getTodos() {
        return new ArrayList<>(todos);
    }
    
    public int getCompletedCount() {
        return (int) todos.stream().filter(Todo::isCompleted).count();
    }
    
    public int getPendingCount() {
        return (int) todos.stream().filter(t -> !t.isCompleted()).count();
    }
}

// ============ CONTROLLER (Activity) ============

public class TodoActivity extends AppCompatActivity {
    
    private TodoModel model;
    private TodoAdapter adapter;
    private EditText todoEditText;
    private Button addButton;
    private TextView statsTextView;
    private RecyclerView todosRecyclerView;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_todo);
        
        // Initialize Model
        model = new TodoModel();
        
        // Find views
        todoEditText = findViewById(R.id.todoEditText);
        addButton = findViewById(R.id.addButton);
        statsTextView = findViewById(R.id.statsTextView);
        todosRecyclerView = findViewById(R.id.todosRecyclerView);
        
        // Setup RecyclerView
        adapter = new TodoAdapter(
            todo -> {
                // Controller handles click
                model.toggleTodo(todo.getId());
                updateUI();
            },
            todo -> {
                // Controller handles delete
                model.deleteTodo(todo.getId());
                updateUI();
            }
        );
        todosRecyclerView.setAdapter(adapter);
        
        // Controller handles add button
        addButton.setOnClickListener(v -> {
            String title = todoEditText.getText().toString().trim();
            
            // Controller validates
            if (title.isEmpty()) {
                Toast.makeText(this, "Enter todo title", Toast.LENGTH_SHORT).show();
                return;
            }
            
            // Controller updates Model
            model.addTodo(title);
            
            // Controller updates View
            updateUI();
            todoEditText.getText().clear();
            
            Toast.makeText(this, "Todo added", Toast.LENGTH_SHORT).show();
        });
        
        updateUI();
    }
    
    private void updateUI() {
        List<Todo> todos = model.getTodos();
        adapter.submitList(todos);
        
        String stats = "Completed: " + model.getCompletedCount() + 
                      " | Pending: " + model.getPendingCount();
        statsTextView.setText(stats);
    }
}
```

### MVC Pros & Cons

**✅ Advantages:**
1. **Simple to understand** - Beginner-friendly
2. **Quick to implement** - Minimal boilerplate
3. **Good for small apps** - <5 screens
4. **Direct control** - Activity has access to everything

**❌ Disadvantages:**
1. **Untestable** - Can't test Controller without Android framework
2. **God Activity** - Activities become massive (1000+ lines)
3. **Tight coupling** - View and Controller are the same (Activity)
4. **No separation** - Business logic in Activity
5. **Memory leaks** - Easy to leak Activity context
6. **Configuration changes** - State lost on rotation

### MVC Real-World Scenarios

**✅ Use MVC for:**
- Simple utilities (calculator, unit converter)
- Prototypes/POCs
- Learning Android basics
- Single-screen apps

**❌ Avoid MVC for:**
- Production apps with >5 screens
- Apps requiring unit testing
- Team projects (merge conflicts)
- Apps with complex business logic

---

## 4. MVP Pattern (Model-View-Presenter)

### Architecture Overview

**MVP** separates **View** (Activity/Fragment) from **business logic** using a **Presenter** as a **middleman**. View and Presenter communicate through **interfaces**.

```
┌──────────────────────────────────────────────┐
│              MVP PATTERN                     │
├──────────────────────────────────────────────┤
│                                              │
│   USER INPUT                                 │
│       ↓                                      │
│   ┌────────┐           ┌──────────┐         │
│   │  VIEW  │ ←─────────│ PRESENTER│         │
│   │(Activ.)│───────────→│ (Logic) │         │
│   └────────┘  Interface └─────┬────┘         │
│                              │               │
│                              ↓               │
│                         ┌──────────┐         │
│                         │  MODEL   │         │
│                         │  (Data)  │         │
│                         └──────────┘         │
│                                              │
│   Data Flow: One-way (through interfaces)   │
│   View ↔ Presenter ↔ Model                   │
└──────────────────────────────────────────────┘
```

### Components

**1. Model** - Data + Repository (same as MVC)
```kotlin
data class User(
    val id: Long,
    val name: String,
    val email: String
)

interface UserRepository {
    fun getUsers(): List<User>
    fun addUser(user: User)
    fun deleteUser(id: Long)
}

class UserRepositoryImpl : UserRepository {
    private val users = mutableListOf<User>()
    
    override fun getUsers() = users.toList()
    
    override fun addUser(user: User) {
        users.add(user)
    }
    
    override fun deleteUser(id: Long) {
        users.removeIf { it.id == id }
    }
}
```

**2. View Interface** - Contract between View and Presenter
```kotlin
interface UserView {
    fun showUsers(users: List<User>)
    fun showError(message: String)
    fun showLoading()
    fun hideLoading()
    fun clearInputFields()
}
```

**3. Presenter** - Business logic (no Android dependencies)
```kotlin
class UserPresenter(
    private val view: UserView,
    private val repository: UserRepository
) {
    
    fun loadUsers() {
        view.showLoading()
        
        // Simulate async operation
        Thread {
            try {
                val users = repository.getUsers()
                
                // Update View on main thread
                Handler(Looper.getMainLooper()).post {
                    view.hideLoading()
                    view.showUsers(users)
                }
            } catch (e: Exception) {
                Handler(Looper.getMainLooper()).post {
                    view.hideLoading()
                    view.showError(e.message ?: "Error loading users")
                }
            }
        }.start()
    }
    
    fun addUser(name: String, email: String) {
        // Validation
        if (name.isEmpty() || email.isEmpty()) {
            view.showError("Fill all fields")
            return
        }
        
        // Business logic
        val user = User(
            id = System.currentTimeMillis(),
            name = name,
            email = email
        )
        
        repository.addUser(user)
        view.clearInputFields()
        
        // Reload users
        loadUsers()
    }
    
    fun deleteUser(userId: Long) {
        repository.deleteUser(userId)
        loadUsers()
    }
}
```

**4. View Implementation** - Activity implements interface
```kotlin
class MainActivity : AppCompatActivity(), UserView {
    
    private lateinit var presenter: UserPresenter
    private lateinit var adapter: UserAdapter
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        // Initialize Presenter
        val repository = UserRepositoryImpl()
        presenter = UserPresenter(this, repository)
        
        // Setup UI
        adapter = UserAdapter { userId ->
            presenter.deleteUser(userId)
        }
        usersRecyclerView.adapter = adapter
        
        addButton.setOnClickListener {
            val name = nameEditText.text.toString()
            val email = emailEditText.text.toString()
            presenter.addUser(name, email)
        }
        
        // Load initial data
        presenter.loadUsers()
    }
    
    // ========== VIEW INTERFACE IMPLEMENTATION ==========
    
    override fun showUsers(users: List<User>) {
        adapter.submitList(users)
    }
    
    override fun showError(message: String) {
        Toast.makeText(this, message, LENGTH_SHORT).show()
    }
    
    override fun showLoading() {
        progressBar.visibility = View.VISIBLE
    }
    
    override fun hideLoading() {
        progressBar.visibility = View.GONE
    }
    
    override fun clearInputFields() {
        nameEditText.text.clear()
        emailEditText.text.clear()
    }
}
```

### Complete Kotlin MVP Example (Login Screen)

```kotlin
// ============ MODEL ============

data class LoginCredentials(
    val username: String,
    val password: String
)

data class LoginResult(
    val success: Boolean,
    val message: String,
    val userId: Long? = null
)

interface LoginRepository {
    fun login(username: String, password: String): LoginResult
}

class LoginRepositoryImpl : LoginRepository {
    override fun login(username: String, password: String): LoginResult {
        // Simulate API call
        Thread.sleep(2000)
        
        return if (username == "admin" && password == "password") {
            LoginResult(true, "Login successful", userId = 123)
        } else {
            LoginResult(false, "Invalid credentials")
        }
    }
}

// ============ CONTRACT (VIEW INTERFACE) ============

interface LoginContract {
    
    interface View {
        fun showLoading()
        fun hideLoading()
        fun showLoginSuccess(userId: Long)
        fun showLoginError(message: String)
        fun showValidationError(message: String)
        fun navigateToHome()
    }
    
    interface Presenter {
        fun onLoginClicked(username: String, password: String)
        fun onDestroy()
    }
}

// ============ PRESENTER ============

class LoginPresenter(
    private val view: LoginContract.View,
    private val repository: LoginRepository
) : LoginContract.Presenter {
    
    private var job: Job? = null
    private val scope = CoroutineScope(Dispatchers.Main + SupervisorJob())
    
    override fun onLoginClicked(username: String, password: String) {
        // Validation
        if (username.isEmpty()) {
            view.showValidationError("Username cannot be empty")
            return
        }
        
        if (password.isEmpty()) {
            view.showValidationError("Password cannot be empty")
            return
        }
        
        if (password.length < 6) {
            view.showValidationError("Password must be at least 6 characters")
            return
        }
        
        // Perform login
        view.showLoading()
        
        job = scope.launch {
            val result = withContext(Dispatchers.IO) {
                repository.login(username, password)
            }
            
            view.hideLoading()
            
            if (result.success) {
                result.userId?.let { view.showLoginSuccess(it) }
                view.navigateToHome()
            } else {
                view.showLoginError(result.message)
            }
        }
    }
    
    override fun onDestroy() {
        job?.cancel()
        scope.cancel()
    }
}

// ============ VIEW (Activity) ============

class LoginActivity : AppCompatActivity(), LoginContract.View {
    
    private lateinit var presenter: LoginContract.Presenter
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_login)
        
        // Initialize Presenter
        val repository = LoginRepositoryImpl()
        presenter = LoginPresenter(this, repository)
        
        // Setup UI listeners
        loginButton.setOnClickListener {
            val username = usernameEditText.text.toString()
            val password = passwordEditText.text.toString()
            presenter.onLoginClicked(username, password)
        }
    }
    
    override fun onDestroy() {
        presenter.onDestroy()
        super.onDestroy()
    }
    
    // ========== VIEW INTERFACE IMPLEMENTATION ==========
    
    override fun showLoading() {
        progressBar.visibility = View.VISIBLE
        loginButton.isEnabled = false
    }
    
    override fun hideLoading() {
        progressBar.visibility = View.GONE
        loginButton.isEnabled = true
    }
    
    override fun showLoginSuccess(userId: Long) {
        Toast.makeText(this, "Welcome user $userId!", LENGTH_SHORT).show()
    }
    
    override fun showLoginError(message: String) {
        errorTextView.text = message
        errorTextView.visibility = View.VISIBLE
    }
    
    override fun showValidationError(message: String) {
        Toast.makeText(this, message, LENGTH_SHORT).show()
    }
    
    override fun navigateToHome() {
        startActivity(Intent(this, HomeActivity::class.java))
        finish()
    }
}
```

### Complete Java MVP Example

```java
// ============ CONTRACT (VIEW INTERFACE) ============

public interface LoginContract {
    
    interface View {
        void showLoading();
        void hideLoading();
        void showLoginSuccess(long userId);
        void showLoginError(String message);
        void showValidationError(String message);
        void navigateToHome();
    }
    
    interface Presenter {
        void onLoginClicked(String username, String password);
        void onDestroy();
    }
}

// ============ MODEL ============

public class LoginResult {
    private boolean success;
    private String message;
    private Long userId;
    
    public LoginResult(boolean success, String message, Long userId) {
        this.success = success;
        this.message = message;
        this.userId = userId;
    }
    
    public boolean isSuccess() { return success; }
    public String getMessage() { return message; }
    public Long getUserId() { return userId; }
}

public interface LoginRepository {
    LoginResult login(String username, String password);
}

public class LoginRepositoryImpl implements LoginRepository {
    @Override
    public LoginResult login(String username, String password) {
        // Simulate API call
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        
        if (username.equals("admin") && password.equals("password")) {
            return new LoginResult(true, "Login successful", 123L);
        } else {
            return new LoginResult(false, "Invalid credentials", null);
        }
    }
}

// ============ PRESENTER ============

public class LoginPresenter implements LoginContract.Presenter {
    
    private LoginContract.View view;
    private LoginRepository repository;
    private ExecutorService executorService;
    private Handler mainHandler;
    
    public LoginPresenter(LoginContract.View view, LoginRepository repository) {
        this.view = view;
        this.repository = repository;
        this.executorService = Executors.newSingleThreadExecutor();
        this.mainHandler = new Handler(Looper.getMainLooper());
    }
    
    @Override
    public void onLoginClicked(String username, String password) {
        // Validation
        if (username.isEmpty()) {
            view.showValidationError("Username cannot be empty");
            return;
        }
        
        if (password.isEmpty()) {
            view.showValidationError("Password cannot be empty");
            return;
        }
        
        if (password.length() < 6) {
            view.showValidationError("Password must be at least 6 characters");
            return;
        }
        
        // Perform login
        view.showLoading();
        
        executorService.execute(() -> {
            LoginResult result = repository.login(username, password);
            
            mainHandler.post(() -> {
                view.hideLoading();
                
                if (result.isSuccess()) {
                    view.showLoginSuccess(result.getUserId());
                    view.navigateToHome();
                } else {
                    view.showLoginError(result.getMessage());
                }
            });
        });
    }
    
    @Override
    public void onDestroy() {
        executorService.shutdown();
    }
}

// ============ VIEW (Activity) ============

public class LoginActivity extends AppCompatActivity implements LoginContract.View {
    
    private LoginContract.Presenter presenter;
    private EditText usernameEditText;
    private EditText passwordEditText;
    private Button loginButton;
    private ProgressBar progressBar;
    private TextView errorTextView;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_login);
        
        // Find views
        usernameEditText = findViewById(R.id.usernameEditText);
        passwordEditText = findViewById(R.id.passwordEditText);
        loginButton = findViewById(R.id.loginButton);
        progressBar = findViewById(R.id.progressBar);
        errorTextView = findViewById(R.id.errorTextView);
        
        // Initialize Presenter
        LoginRepository repository = new LoginRepositoryImpl();
        presenter = new LoginPresenter(this, repository);
        
        // Setup listeners
        loginButton.setOnClickListener(v -> {
            String username = usernameEditText.getText().toString();
            String password = passwordEditText.getText().toString();
            presenter.onLoginClicked(username, password);
        });
    }
    
    @Override
    protected void onDestroy() {
        presenter.onDestroy();
        super.onDestroy();
    }
    
    // ========== VIEW INTERFACE IMPLEMENTATION ==========
    
    @Override
    public void showLoading() {
        progressBar.setVisibility(View.VISIBLE);
        loginButton.setEnabled(false);
    }
    
    @Override
    public void hideLoading() {
        progressBar.setVisibility(View.GONE);
        loginButton.setEnabled(true);
    }
    
    @Override
    public void showLoginSuccess(long userId) {
        Toast.makeText(this, "Welcome user " + userId + "!", Toast.LENGTH_SHORT).show();
    }
    
    @Override
    public void showLoginError(String message) {
        errorTextView.setText(message);
        errorTextView.setVisibility(View.VISIBLE);
    }
    
    @Override
    public void showValidationError(String message) {
        Toast.makeText(this, message, Toast.LENGTH_SHORT).show();
    }
    
    @Override
    public void navigateToHome() {
        Intent intent = new Intent(this, HomeActivity.class);
        startActivity(intent);
        finish();
    }
}
```

### MVP Pros & Cons

**✅ Advantages:**
1. **Testable** - Presenter is pure Kotlin/Java (no Android)
2. **Separation of concerns** - View only displays, Presenter handles logic
3. **Interface-based** - Loose coupling through contracts
4. **Easy to mock** - Can test Presenter with fake View
5. **Reusable Presenters** - Same Presenter for Activity/Fragment

**❌ Disadvantages:**
1. **Boilerplate** - Many interfaces to maintain
2. **Memory leaks** - Must clean up Presenter references
3. **One Presenter per View** - Can't reuse across screens
4. **No lifecycle awareness** - Manual cleanup required
5. **Callback hell** - Many interface methods

### MVP Real-World Scenarios

**✅ Use MVP for:**
- Medium-sized apps (10-30 screens)
- Apps requiring testable business logic
- Legacy codebases (pre-Jetpack)
- When ViewModel is not suitable

**❌ Avoid MVP for:**
- New projects (use MVVM instead)
- Simple apps (MVP is overkill)
- Apps using Jetpack Compose (incompatible)

---

## 5. MVVM Pattern (Model-View-ViewModel)

### Architecture Overview

**MVVM** is the **modern Android pattern** where **ViewModel** manages UI state and exposes it to the **View** via **observable data** (LiveData/StateFlow). ViewModel survives configuration changes.

```
┌──────────────────────────────────────────────┐
│              MVVM PATTERN                    │
├──────────────────────────────────────────────┤
│                                              │
│   USER INPUT                                 │
│       ↓                                      │
│   ┌────────┐                                 │
│   │  VIEW  │                                 │
│   │(Activ.)│ ←────── Observable              │
│   └───┬────┘         (LiveData/Flow)         │
│       │                    ↑                 │
│       │ User Events        │                 │
│       ↓                    │                 │
│   ┌──────────────┐         │                 │
│   │  VIEWMODEL   │─────────┘                 │
│   │ (UI Logic)   │                           │
│   └──────┬───────┘                           │
│          │                                   │
│          ↓                                   │
│   ┌──────────────┐                           │
│   │ REPOSITORY   │                           │
│   │ (Data Logic) │                           │
│   └──────┬───────┘                           │
│          │                                   │
│          ↓                                   │
│   ┌──────────────┐                           │
│   │    MODEL     │                           │
│   │(Room/Retrofit│                           │
│   └──────────────┘                           │
│                                              │
│   Data Flow: Unidirectional reactive         │
│   View observes ViewModel                    │
└──────────────────────────────────────────────┘
```

### Components

**1. Model** - Repository + Data Sources
```kotlin
data class User(
    val id: Long,
    val name: String,
    val email: String
)

class UserRepository {
    private val users = mutableListOf<User>()
    
    suspend fun getUsers(): List<User> {
        // Simulate network delay
        delay(1000)
        return users.toList()
    }
    
    suspend fun addUser(user: User) {
        delay(500)
        users.add(user)
    }
    
    suspend fun deleteUser(id: Long) {
        users.removeIf { it.id == id }
    }
}
```

**2. ViewModel** - UI Logic + State Management
```kotlin
class UserViewModel(
    private val repository: UserRepository
) : ViewModel() {
    
    // UI State
    private val _users = MutableLiveData<List<User>>()
    val users: LiveData<List<User>> = _users
    
    private val _isLoading = MutableLiveData<Boolean>()
    val isLoading: LiveData<Boolean> = _isLoading
    
    private val _error = MutableLiveData<String?>()
    val error: LiveData<String?> = _error
    
    init {
        loadUsers()
    }
    
    fun loadUsers() {
        viewModelScope.launch {
            _isLoading.value = true
            _error.value = null
            
            try {
                val userList = repository.getUsers()
                _users.value = userList
            } catch (e: Exception) {
                _error.value = e.message
            } finally {
                _isLoading.value = false
            }
        }
    }
    
    fun addUser(name: String, email: String) {
        if (name.isEmpty() || email.isEmpty()) {
            _error.value = "Fill all fields"
            return
        }
        
        viewModelScope.launch {
            _isLoading.value = true
            
            try {
                val user = User(
                    id = System.currentTimeMillis(),
                    name = name,
                    email = email
                )
                repository.addUser(user)
                loadUsers()  // Refresh list
            } catch (e: Exception) {
                _error.value = e.message
            } finally {
                _isLoading.value = false
            }
        }
    }
    
    fun deleteUser(userId: Long) {
        viewModelScope.launch {
            try {
                repository.deleteUser(userId)
                loadUsers()
            } catch (e: Exception) {
                _error.value = e.message
            }
        }
    }
}
```

**3. View** - Activity/Fragment (observes ViewModel)
```kotlin
class MainActivity : AppCompatActivity() {
    
    private val viewModel: UserViewModel by viewModels {
        object : ViewModelProvider.Factory {
            override fun <T : ViewModel> create(modelClass: Class<T>): T {
                return UserViewModel(UserRepository()) as T
            }
        }
    }
    
    private lateinit var adapter: UserAdapter
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        setupRecyclerView()
        setupObservers()
        setupListeners()
    }
    
    private fun setupRecyclerView() {
        adapter = UserAdapter { userId ->
            viewModel.deleteUser(userId)
        }
        usersRecyclerView.adapter = adapter
    }
    
    private fun setupObservers() {
        // Observe users list
        viewModel.users.observe(this) { users ->
            adapter.submitList(users)
        }
        
        // Observe loading state
        viewModel.isLoading.observe(this) { isLoading ->
            progressBar.visibility = if (isLoading) View.VISIBLE else View.GONE
        }
        
        // Observe errors
        viewModel.error.observe(this) { error ->
            error?.let {
                Toast.makeText(this, it, LENGTH_SHORT).show()
            }
        }
    }
    
    private fun setupListeners() {
        addButton.setOnClickListener {
            val name = nameEditText.text.toString()
            val email = emailEditText.text.toString()
            
            viewModel.addUser(name, email)
            
            nameEditText.text.clear()
            emailEditText.text.clear()
        }
    }
}
```

### Complete Kotlin MVVM Example (Weather App)

```kotlin
// ============ MODEL (Data Layer) ============

data class Weather(
    val city: String,
    val temperature: Double,
    val condition: String,
    val humidity: Int,
    val timestamp: Long = System.currentTimeMillis()
)

sealed class WeatherResult {
    data class Success(val weather: Weather) : WeatherResult()
    data class Error(val message: String) : WeatherResult()
    object Loading : WeatherResult()
}

interface WeatherRepository {
    suspend fun getWeather(city: String): Weather
    suspend fun refreshWeather(city: String): Weather
}

class WeatherRepositoryImpl : WeatherRepository {
    
    override suspend fun getWeather(city: String): Weather {
        // Simulate API call
        delay(2000)
        
        return Weather(
            city = city,
            temperature = (15..35).random().toDouble(),
            condition = listOf("Sunny", "Cloudy", "Rainy").random(),
            humidity = (40..90).random()
        )
    }
    
    override suspend fun refreshWeather(city: String): Weather {
        return getWeather(city)
    }
}

// ============ VIEWMODEL ============

class WeatherViewModel(
    private val repository: WeatherRepository
) : ViewModel() {
    
    // UI State using StateFlow (modern approach)
    private val _weatherState = MutableStateFlow<WeatherResult>(WeatherResult.Loading)
    val weatherState: StateFlow<WeatherResult> = _weatherState.asStateFlow()
    
    // Alternative: Using LiveData (traditional approach)
    private val _weatherLiveData = MutableLiveData<WeatherResult>()
    val weatherLiveData: LiveData<WeatherResult> = _weatherLiveData
    
    private var currentCity = "Delhi"
    
    init {
        loadWeather(currentCity)
    }
    
    fun loadWeather(city: String) {
        if (city.isEmpty()) {
            _weatherState.value = WeatherResult.Error("City name cannot be empty")
            return
        }
        
        currentCity = city
        
        viewModelScope.launch {
            _weatherState.value = WeatherResult.Loading
            
            try {
                val weather = repository.getWeather(city)
                _weatherState.value = WeatherResult.Success(weather)
            } catch (e: Exception) {
                _weatherState.value = WeatherResult.Error(
                    e.message ?: "Failed to fetch weather"
                )
            }
        }
    }
    
    fun refreshWeather() {
        viewModelScope.launch {
            _weatherState.value = WeatherResult.Loading
            
            try {
                val weather = repository.refreshWeather(currentCity)
                _weatherState.value = WeatherResult.Success(weather)
            } catch (e: Exception) {
                _weatherState.value = WeatherResult.Error("Refresh failed")
            }
        }
    }
}

// ============ VIEW (Activity) ============

class WeatherActivity : AppCompatActivity() {
    
    private val viewModel: WeatherViewModel by viewModels {
        object : ViewModelProvider.Factory {
            override fun <T : ViewModel> create(modelClass: Class<T>): T {
                return WeatherViewModel(WeatherRepositoryImpl()) as T
            }
        }
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_weather)
        
        setupUI()
        observeWeather()
    }
    
    private fun setupUI() {
        searchButton.setOnClickListener {
            val city = cityEditText.text.toString()
            viewModel.loadWeather(city)
        }
        
        refreshButton.setOnClickListener {
            viewModel.refreshWeather()
        }
    }
    
    private fun observeWeather() {
        // Using StateFlow with coroutines (Modern)
        lifecycleScope.launch {
            viewModel.weatherState.collect { result ->
                when (result) {
                    is WeatherResult.Loading -> {
                        showLoading()
                    }
                    is WeatherResult.Success -> {
                        hideLoading()
                        displayWeather(result.weather)
                    }
                    is WeatherResult.Error -> {
                        hideLoading()
                        showError(result.message)
                    }
                }
            }
        }
        
        // Alternative: Using LiveData (Traditional)
        /*
        viewModel.weatherLiveData.observe(this) { result ->
            when (result) {
                is WeatherResult.Loading -> showLoading()
                is WeatherResult.Success -> displayWeather(result.weather)
                is WeatherResult.Error -> showError(result.message)
            }
        }
        */
    }
    
    private fun showLoading() {
        progressBar.visibility = View.VISIBLE
        weatherCard.visibility = View.GONE
    }
    
    private fun hideLoading() {
        progressBar.visibility = View.GONE
        weatherCard.visibility = View.VISIBLE
    }
    
    private fun displayWeather(weather: Weather) {
        cityText.text = weather.city
        temperatureText.text = "${weather.temperature}°C"
        conditionText.text = weather.condition
        humidityText.text = "Humidity: ${weather.humidity}%"
    }
    
    private fun showError(message: String) {
        Toast.makeText(this, message, LENGTH_SHORT).show()
    }
}

// ============ VIEW (Jetpack Compose - Modern) ============

@Composable
fun WeatherScreen(viewModel: WeatherViewModel = viewModel()) {
    val weatherState by viewModel.weatherState.collectAsState()
    
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp)
    ) {
        // Search bar
        var cityInput by remember { mutableStateOf("") }
        
        OutlinedTextField(
            value = cityInput,
            onValueChange = { cityInput = it },
            label = { Text("Enter city name") },
            modifier = Modifier.fillMaxWidth()
        )
        
        Spacer(modifier = Modifier.height(8.dp))
        
        Button(
            onClick = { viewModel.loadWeather(cityInput) },
            modifier = Modifier.fillMaxWidth()
        ) {
            Text("Search")
        }
        
        Spacer(modifier = Modifier.height(16.dp))
        
        // Weather display
        when (val state = weatherState) {
            is WeatherResult.Loading -> {
                CircularProgressIndicator(
                    modifier = Modifier.align(Alignment.CenterHorizontally)
                )
            }
            
            is WeatherResult.Success -> {
                WeatherCard(weather = state.weather)
                
                Button(
                    onClick = { viewModel.refreshWeather() },
                    modifier = Modifier
                        .fillMaxWidth()
                        .padding(top = 16.dp)
                ) {
                    Text("Refresh")
                }
            }
            
            is WeatherResult.Error -> {
                Text(
                    text = state.message,
                    color = Color.Red,
                    modifier = Modifier.align(Alignment.CenterHorizontally)
                )
            }
        }
    }
}

@Composable
fun WeatherCard(weather: Weather) {
    Card(
        modifier = Modifier.fillMaxWidth(),
        elevation = CardDefaults.cardElevation(defaultElevation = 4.dp)
    ) {
        Column(
            modifier = Modifier.padding(16.dp),
            horizontalAlignment = Alignment.CenterHorizontally
        ) {
            Text(
                text = weather.city,
                style = MaterialTheme.typography.headlineMedium
            )
            
            Spacer(modifier = Modifier.height(8.dp))
            
            Text(
                text = "${weather.temperature}°C",
                style = MaterialTheme.typography.displayLarge
            )
            
            Spacer(modifier = Modifier.height(8.dp))
            
            Text(
                text = weather.condition,
                style = MaterialTheme.typography.bodyLarge
            )
            
            Text(
                text = "Humidity: ${weather.humidity}%",
                style = MaterialTheme.typography.bodyMedium
            )
        }
    }
}
```

### Complete Java MVVM Example

```java
// ============ MODEL ============

public class Weather {
    private String city;
    private double temperature;
    private String condition;
    private int humidity;
    
    public Weather(String city, double temperature, String condition, int humidity) {
        this.city = city;
        this.temperature = temperature;
        this.condition = condition;
        this.humidity = humidity;
    }
    
    // Getters
    public String getCity() { return city; }
    public double getTemperature() { return temperature; }
    public String getCondition() { return condition; }
    public int getHumidity() { return humidity; }
}

public class WeatherRepository {
    
    public LiveData<Weather> getWeather(String city) {
        MutableLiveData<Weather> result = new MutableLiveData<>();
        
        // Simulate async API call
        new Thread(() -> {
            try {
                Thread.sleep(2000);
                
                Random random = new Random();
                Weather weather = new Weather(
                    city,
                    15 + random.nextInt(20),
                    new String[]{"Sunny", "Cloudy", "Rainy"}[random.nextInt(3)],
                    40 + random.nextInt(50)
                );
                
                result.postValue(weather);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
        
        return result;
    }
}

// ============ VIEWMODEL ============

public class WeatherViewModel extends ViewModel {
    
    private final WeatherRepository repository;
    private final MutableLiveData<Weather> weather = new MutableLiveData<>();
    private final MutableLiveData<Boolean> isLoading = new MutableLiveData<>();
    private final MutableLiveData<String> error = new MutableLiveData<>();
    
    public WeatherViewModel(WeatherRepository repository) {
        this.repository = repository;
    }
    
    public LiveData<Weather> getWeather() {
        return weather;
    }
    
    public LiveData<Boolean> getIsLoading() {
        return isLoading;
    }
    
    public LiveData<String> getError() {
        return error;
    }
    
    public void loadWeather(String city) {
        if (city.isEmpty()) {
            error.setValue("City name cannot be empty");
            return;
        }
        
        isLoading.setValue(true);
        
        repository.getWeather(city).observeForever(result -> {
            isLoading.setValue(false);
            
            if (result != null) {
                weather.setValue(result);
            } else {
                error.setValue("Failed to fetch weather");
            }
        });
    }
}

// ============ VIEW (Activity) ============

public class WeatherActivity extends AppCompatActivity {
    
    private WeatherViewModel viewModel;
    private EditText cityEditText;
    private Button searchButton;
    private ProgressBar progressBar;
    private TextView cityText;
    private TextView temperatureText;
    private TextView conditionText;
    private TextView humidityText;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_weather);
        
        // Find views
        cityEditText = findViewById(R.id.cityEditText);
        searchButton = findViewById(R.id.searchButton);
        progressBar = findViewById(R.id.progressBar);
        cityText = findViewById(R.id.cityText);
        temperatureText = findViewById(R.id.temperatureText);
        conditionText = findViewById(R.id.conditionText);
        humidityText = findViewById(R.id.humidityText);
        
        // Initialize ViewModel
        viewModel = new ViewModelProvider(this, new ViewModelProvider.Factory() {
            @Override
            public <T extends ViewModel> T create(Class<T> modelClass) {
                return (T) new WeatherViewModel(new WeatherRepository());
            }
        }).get(WeatherViewModel.class);
        
        setupObservers();
        setupListeners();
    }
    
    private void setupObservers() {
        // Observe weather data
        viewModel.getWeather().observe(this, weather -> {
            if (weather != null) {
                displayWeather(weather);
            }
        });
        
        // Observe loading state
        viewModel.getIsLoading().observe(this, isLoading -> {
            progressBar.setVisibility(isLoading ? View.VISIBLE : View.GONE);
        });
        
        // Observe errors
        viewModel.getError().observe(this, error -> {
            if (error != null && !error.isEmpty()) {
                Toast.makeText(this, error, Toast.LENGTH_SHORT).show();
            }
        });
    }
    
    private void setupListeners() {
        searchButton.setOnClickListener(v -> {
            String city = cityEditText.getText().toString();
            viewModel.loadWeather(city);
        });
    }
    
    private void displayWeather(Weather weather) {
        cityText.setText(weather.getCity());
        temperatureText.setText(weather.getTemperature() + "°C");
        conditionText.setText(weather.getCondition());
        humidityText.setText("Humidity: " + weather.getHumidity() + "%");
    }
}
```

### MVVM with Jetpack Components

**Modern MVVM Stack (2026):**

```kotlin
// ViewModel with Hilt DI
@HiltViewModel
class UserViewModel @Inject constructor(
    private val repository: UserRepository
) : ViewModel() {
    
    // StateFlow (modern reactive state)
    private val _uiState = MutableStateFlow<UiState>(UiState.Loading)
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()
    
    // Alternative: LiveData (traditional)
    private val _users = MutableLiveData<List<User>>()
    val users: LiveData<List<User>> = _users
    
    // Single Event
    private val _events = Channel<Event>()
    val events = _events.receiveAsFlow()
    
    fun loadUsers() = viewModelScope.launch {
        _uiState.value = UiState.Loading
        
        try {
            val users = repository.getUsers()
            _uiState.value = UiState.Success(users)
        } catch (e: Exception) {
            _uiState.value = UiState.Error(e.message)
        }
    }
    
    fun addUser(name: String, email: String) = viewModelScope.launch {
        try {
            repository.addUser(User(0, name, email))
            _events.send(Event.UserAdded)
            loadUsers()
        } catch (e: Exception) {
            _events.send(Event.ShowError(e.message))
        }
    }
}

// UI State
sealed class UiState {
    object Loading : UiState()
    data class Success(val users: List<User>) : UiState()
    data class Error(val message: String?) : UiState()
}

// Single Events
sealed class Event {
    object UserAdded : Event()
    data class ShowError(val message: String?) : Event()
}

// Activity with Hilt
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    
    private val viewModel: UserViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        observeState()
        observeEvents()
    }
    
    private fun observeState() {
        lifecycleScope.launch {
            viewModel.uiState.collect { state ->
                when (state) {
                    is UiState.Loading -> showLoading()
                    is UiState.Success -> showUsers(state.users)
                    is UiState.Error -> showError(state.message)
                }
            }
        }
    }
    
    private fun observeEvents() {
        lifecycleScope.launch {
            viewModel.events.collect { event ->
                when (event) {
                    is Event.UserAdded -> {
                        Toast.makeText(this@MainActivity, "User added!", LENGTH_SHORT).show()
                    }
                    is Event.ShowError -> {
                        Toast.makeText(this@MainActivity, event.message, LENGTH_SHORT).show()
                    }
                }
            }
        }
    }
}
```

### MVVM Pros & Cons

**✅ Advantages:**
1. **Highly testable** - ViewModel is pure Kotlin, no Android dependencies
2. **Lifecycle-aware** - Survives configuration changes automatically
3. **Reactive** - LiveData/StateFlow observes data changes
4. **Google recommended** - Full Jetpack support
5. **Less boilerplate** - No interface contracts needed
6. **Compose-friendly** - Perfect for Jetpack Compose
7. **No memory leaks** - ViewModel cleans up automatically

**❌ Disadvantages:**
1. **Learning curve** - Requires understanding of LiveData/Flow/Coroutines
2. **Overkill for simple apps** - Too much for calculators/converters
3. **Initial setup** - More code upfront
4. **Debugging** - Reactive streams harder to debug

### MVVM Real-World Scenarios

**✅ Use MVVM for:**
- **ALL new Android projects** (2026 standard)
- Production apps with any complexity
- Apps using Jetpack Compose
- Apps requiring robust testing
- Team projects
- Long-term maintained apps

**❌ Avoid MVVM for:**
- Tiny utilities (<3 screens)
- Quick prototypes where testing isn't needed
- Legacy projects already using MVP (don't migrate unless necessary)

---

## 6. Detailed Comparison

### Side-by-Side Comparison

| Aspect | **MVC** | **MVP** | **MVVM** |
|--------|---------|---------|----------|
| **Testability** | ❌ Hard (Activity = Controller) | ✅ Medium (Presenter testable) | ✅ Easy (ViewModel fully testable) |
| **Complexity** | 🟢 Low | 🟡 Medium | 🔴 High |
| **Boilerplate** | 🟢 Low | 🔴 High (many interfaces) | 🟡 Medium |
| **Lifecycle** | ❌ Manual handling | ❌ Manual cleanup | ✅ Auto-handled |
| **Configuration Changes** | ❌ Loses state | ❌ Manual save/restore | ✅ Survives automatically |
| **Memory Leaks** | ⚠️ High risk | ⚠️ Medium risk | ✅ Low risk |
| **Google Support** | ❌ None | ⚠️ Legacy | ✅ Full Jetpack |
| **Compose Support** | ❌ No | ❌ No | ✅ Yes |
| **Learning Curve** | 🟢 Easy | 🟡 Medium | 🔴 Hard |
| **Team Scale** | ❌ Small teams only | ✅ Medium teams | ✅ Large teams |
| **Code Separation** | ❌ Poor | ✅ Good | ✅ Excellent |
| **Industry Adoption** | 📉 Declining | 📉 Legacy | 📈 Growing |

### Data Flow Comparison

**MVC Data Flow:**
```
User clicks button
    ↓
Activity (Controller) receives click
    ↓
Activity updates Model directly
    ↓
Activity fetches from Model
    ↓
Activity updates View (TextView.setText)
```
**Problems:** Activity does everything, no testability

**MVP Data Flow:**
```
User clicks button
    ↓
Activity (View) calls presenter.onButtonClick()
    ↓
Presenter validates and processes
    ↓
Presenter updates Model
    ↓
Presenter calls view.showResult()
    ↓
Activity updates UI
```
**Benefits:** Separated, testable, but requires interfaces

**MVVM Data Flow:**
```
User clicks button
    ↓
Activity calls viewModel.loadData()
    ↓
ViewModel launches coroutine
    ↓
ViewModel updates LiveData/StateFlow
    ↓
Activity observes and reacts automatically
    ↓
UI updates automatically
```
**Benefits:** Reactive, lifecycle-aware, minimal coupling

### Code Comparison (Same Feature)

**Feature: Load and display user list**

**MVC (60 lines):**
```kotlin
class MainActivity : AppCompatActivity() {
    private val model = UserModel()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        loadButton.setOnClickListener {
            // Controller does everything
            progressBar.visibility = View.VISIBLE
            
            Thread {
                val users = model.getUsers()
                
                runOnUiThread {
                    progressBar.visibility = View.GONE
                    adapter.submitList(users)
                }
            }.start()
        }
    }
}
```

**MVP (120 lines with interfaces):**
```kotlin
// Contract
interface UserContract {
    interface View {
        fun showUsers(users: List<User>)
        fun showLoading()
        fun hideLoading()
    }
    interface Presenter {
        fun loadUsers()
    }
}

// Presenter
class UserPresenter(private val view: View) : Presenter {
    override fun loadUsers() {
        view.showLoading()
        Thread {
            val users = repository.getUsers()
            view.hideLoading()
            view.showUsers(users)
        }.start()
    }
}

// Activity
class MainActivity : AppCompatActivity(), UserContract.View {
    private val presenter = UserPresenter(this)
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        loadButton.setOnClickListener {
            presenter.loadUsers()
        }
    }
    
    override fun showUsers(users: List<User>) {
        adapter.submitList(users)
    }
}
```

**MVVM (80 lines, reactive):**
```kotlin
// ViewModel
class UserViewModel(private val repository: UserRepository) : ViewModel() {
    private val _users = MutableLiveData<List<User>>()
    val users: LiveData<List<User>> = _users
    
    private val _isLoading = MutableLiveData<Boolean>()
    val isLoading: LiveData<Boolean> = _isLoading
    
    fun loadUsers() = viewModelScope.launch {
        _isLoading.value = true
        _users.value = repository.getUsers()
        _isLoading.value = false
    }
}

// Activity
class MainActivity : AppCompatActivity() {
    private val viewModel: UserViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Observe data
        viewModel.users.observe(this) { users ->
            adapter.submitList(users)
        }
        
        viewModel.isLoading.observe(this) { isLoading ->
            progressBar.visibility = if (isLoading) VISIBLE else GONE
        }
        
        loadButton.setOnClickListener {
            viewModel.loadUsers()
        }
    }
}
```

**Winner:** MVVM (reactive, clean, testable, lifecycle-aware)

---

## 7. When to Use Each Pattern

### Decision Tree

```
Starting new Android project?
    ├─ YES → Use MVVM (2026 standard)
    │   └─ Using Jetpack Compose? → Definitely MVVM
    │
    └─ NO (existing project)
        ├─ Currently using MVC?
        │   ├─ App is simple (<5 screens) → Keep MVC
        │   └─ App is growing → Migrate to MVVM
        │
        └─ Currently using MVP?
            ├─ Working well? → Keep MVP
            ├─ Adding Compose? → Migrate to MVVM
            └─ Starting new modules? → Use MVVM for new code
```

### Use Case Matrix

| Project Type | **Recommended** | **Why** |
|-------------|----------------|---------|
| **New project (2026)** | MVVM | Industry standard, Jetpack support |
| **Jetpack Compose app** | MVVM | Only compatible architecture |
| **Simple utility** | MVC | Overkill to use complex patterns |
| **Learning Android** | MVC → MVP → MVVM | Progressive learning |
| **Production app** | MVVM | Testable, maintainable, scalable |
| **Legacy app (pre-2017)** | Keep existing or MVP | Don't migrate unless needed |
| **Team project** | MVVM | Best separation of concerns |
| **Prototype/POC** | MVC | Fast, minimal setup |
| **Enterprise app** | MVVM + Clean Architecture | Professional standard |

### Pattern Selection Criteria

**Choose MVC if:**
- ✅ App has <3 screens
- ✅ No testing requirements
- ✅ Solo developer, short-term project
- ✅ Learning Android basics

**Choose MVP if:**
- ✅ Existing MVP codebase
- ✅ Team familiar with MVP
- ✅ Can't use Jetpack (old Android versions)
- ✅ Medium-sized app (10-30 screens)

**Choose MVVM if:**
- ✅ **ANY new project in 2026** ⭐
- ✅ Using Jetpack Compose
- ✅ Requires unit testing
- ✅ Long-term maintenance
- ✅ Team collaboration
- ✅ Configuration change handling critical
- ✅ Want industry-standard architecture

---

## 8. Real-World Scenarios

### Scenario 1: E-Commerce App (MVVM)

**Requirements:**
- Product listing from API
- Shopping cart
- User authentication
- Order history
- Offline support

**Architecture:**

```
┌─────────────────────────────────────────┐
│           E-COMMERCE MVVM               │
├─────────────────────────────────────────┤
│                                         │
│  UI Layer (Compose/XML)                 │
│  ├─ ProductListScreen                   │
│  ├─ ProductDetailScreen                 │
│  ├─ CartScreen                          │
│  └─ OrderHistoryScreen                  │
│                                         │
│  ViewModel Layer                        │
│  ├─ ProductListViewModel                │
│  ├─ ProductDetailViewModel              │
│  ├─ CartViewModel                       │
│  └─ OrderViewModel                      │
│                                         │
│  Repository Layer                       │
│  ├─ ProductRepository                   │
│  │   ├─ Remote: Retrofit API            │
│  │   └─ Local: Room Database            │
│  ├─ CartRepository                      │
│  │   └─ Local: Room + SharedPrefs       │
│  └─ OrderRepository                     │
│      ├─ Remote: Retrofit API            │
│      └─ Local: Room Database            │
└─────────────────────────────────────────┘
```

**Sample Code:**

```kotlin
// Product ViewModel
@HiltViewModel
class ProductListViewModel @Inject constructor(
    private val productRepository: ProductRepository
) : ViewModel() {
    
    val products = productRepository.getProducts()
        .stateIn(viewModelScope, SharingStarted.Lazily, emptyList())
    
    fun refreshProducts() = viewModelScope.launch {
        productRepository.refreshProducts()
    }
    
    fun addToCart(productId: Long) = viewModelScope.launch {
        cartRepository.addItem(productId)
    }
}

// Compose UI
@Composable
fun ProductListScreen(viewModel: ProductListViewModel = hiltViewModel()) {
    val products by viewModel.products.collectAsState()
    
    LazyColumn {
        items(products) { product ->
            ProductItem(
                product = product,
                onAddToCart = { viewModel.addToCart(product.id) }
            )
        }
    }
}
```

**Why MVVM:**
- ✅ Offline-first with Room + Retrofit
- ✅ Reactive UI with StateFlow
- ✅ Survives configuration changes
- ✅ Easy to test (mock repository)
- ✅ Jetpack Compose compatible

### Scenario 2: Banking App (MVVM + Clean Architecture)

**Requirements:**
- Account balance
- Transaction history
- Fund transfers
- Bill payments
- High security

**Architecture:**

```
Presentation Layer (MVVM)
    ├─ AccountViewModel
    ├─ TransactionViewModel
    └─ TransferViewModel
        ↓
Domain Layer (Use Cases)
    ├─ GetAccountBalanceUseCase
    ├─ GetTransactionsUseCase
    └─ TransferFundsUseCase
        ↓
Data Layer (Repositories)
    ├─ AccountRepository
    │   ├─ Remote: Secure API (SSL Pinning)
    │   └─ Local: Encrypted SharedPrefs
    └─ TransactionRepository
        ├─ Remote: Secure API
        └─ Local: Encrypted Room DB
```

**Why MVVM + Use Cases:**
- ✅ Separation of business logic (Use Cases)
- ✅ Testable domain logic
- ✅ Secure data handling
- ✅ Easy to add biometric auth
- ✅ Reactive balance updates

### Scenario 3: Samsung Daemon Manager (MVVM)

**Your Samsung R&D Project:**

**Requirements:**
- Monitor system daemons
- Start/stop services
- View daemon logs
- Configure daemon parameters
- IPC with system services

**Architecture:**

```
┌───────────────────────────────────────────┐
│       DAEMON MANAGER (MVVM)               │
├───────────────────────────────────────────┤
│                                           │
│  UI Layer                                 │
│  ├─ DaemonListActivity                    │
│  ├─ DaemonDetailActivity                  │
│  └─ LogViewerActivity                     │
│                                           │
│  ViewModel Layer                          │
│  ├─ DaemonListViewModel                   │
│  │   └─ StateFlow<List<DaemonStatus>>     │
│  └─ DaemonDetailViewModel                 │
│      └─ StateFlow<DaemonInfo>             │
│                                           │
│  Repository Layer                         │
│  └─ DaemonRepository                      │
│      ├─ SystemServiceManager (Binder IPC) │
│      ├─ UdevMonitor (Linux udev)          │
│      └─ LogcatReader (logcat -b system)   │
└───────────────────────────────────────────┘
```

**Sample Code:**

```kotlin
// Daemon Status Model
data class DaemonStatus(
    val name: String,
    val pid: Int?,
    val isRunning: Boolean,
    val uptime: Long,
    val cpuUsage: Float
)

// Daemon Repository
class DaemonRepository @Inject constructor(
    private val systemServiceManager: SystemServiceManager,
    private val udevMonitor: UdevMonitor
) {
    
    fun getDaemonStatuses(): Flow<List<DaemonStatus>> = flow {
        while (true) {
            val statuses = systemServiceManager.getAllDaemonStatuses()
            emit(statuses)
            delay(1000)  // Poll every second
        }
    }.flowOn(Dispatchers.IO)
    
    suspend fun startDaemon(name: String): Result<Unit> = withContext(Dispatchers.IO) {
        try {
            systemServiceManager.startService(name)
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    suspend fun stopDaemon(name: String): Result<Unit> = withContext(Dispatchers.IO) {
        try {
            systemServiceManager.stopService(name)
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}

// ViewModel
@HiltViewModel
class DaemonListViewModel @Inject constructor(
    private val repository: DaemonRepository
) : ViewModel() {
    
    val daemons = repository.getDaemonStatuses()
        .stateIn(viewModelScope, SharingStarted.Lazily, emptyList())
    
    private val _events = Channel<Event>()
    val events = _events.receiveAsFlow()
    
    fun startDaemon(name: String) = viewModelScope.launch {
        val result = repository.startDaemon(name)
        if (result.isSuccess) {
            _events.send(Event.ShowSuccess("Daemon $name started"))
        } else {
            _events.send(Event.ShowError("Failed to start $name"))
        }
    }
    
    fun stopDaemon(name: String) = viewModelScope.launch {
        val result = repository.stopDaemon(name)
        if (result.isSuccess) {
            _events.send(Event.ShowSuccess("Daemon $name stopped"))
        } else {
            _events.send(Event.ShowError("Failed to stop $name"))
        }
    }
    
    sealed class Event {
        data class ShowSuccess(val message: String) : Event()
        data class ShowError(val message: String) : Event()
    }
}

// Activity
@AndroidEntryPoint
class DaemonListActivity : AppCompatActivity() {
    
    private val viewModel: DaemonListViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_daemon_list)
        
        observeDaemons()
        observeEvents()
    }
    
    private fun observeDaemons() {
        lifecycleScope.launch {
            viewModel.daemons.collect { daemons ->
                adapter.submitList(daemons)
            }
        }
    }
    
    private fun observeEvents() {
        lifecycleScope.launch {
            viewModel.events.collect { event ->
                when (event) {
                    is DaemonListViewModel.Event.ShowSuccess -> {
                        Snackbar.make(rootView, event.message, LENGTH_SHORT).show()
                    }
                    is DaemonListViewModel.Event.ShowError -> {
                        Snackbar.make(rootView, event.message, LENGTH_LONG)
                            .setAction("Retry") { /* retry */ }
                            .show()
                    }
                }
            }
        }
    }
}
```

**Why MVVM for System Software:**
- ✅ Real-time state updates (StateFlow)
- ✅ Background daemon monitoring
- ✅ Survives configuration changes
- ✅ Easy to test IPC logic
- ✅ Clean separation: UI ← ViewModel ← Repository ← Binder/UDev

---

## 9. Testing Each Pattern

### MVC Testing (Hard)

**Problem:** Controller = Activity (Android framework required)

```kotlin
// ❌ CAN'T test without Robolectric/Espresso
class MainActivityTest {
    @Test
    fun testAddUser() {
        // Need Android framework to test Activity
        // Requires Robolectric or Instrumented tests
    }
}
```

**Workaround:** Extract logic to separate class
```kotlin
class UserLogic {
    fun validateUser(name: String, email: String): Boolean {
        return name.isNotEmpty() && email.contains("@")
    }
}

// ✅ Can test
class UserLogicTest {
    @Test
    fun `validateUser returns true for valid input`() {
        val logic = UserLogic()
        assertTrue(logic.validateUser("John", "john@test.com"))
    }
}
```

### MVP Testing (Medium)

**Advantage:** Presenter has no Android dependencies

```kotlin
// Presenter (Pure Kotlin)
class LoginPresenter(
    private val view: LoginContract.View,
    private val repository: LoginRepository
) {
    fun onLoginClicked(username: String, password: String) {
        if (username.isEmpty()) {
            view.showError("Username required")
            return
        }
        
        val result = repository.login(username, password)
        if (result.success) {
            view.navigateToHome()
        } else {
            view.showError(result.message)
        }
    }
}

// ✅ Easy to test with mocks
class LoginPresenterTest {
    
    @Mock
    lateinit var view: LoginContract.View
    
    @Mock
    lateinit var repository: LoginRepository
    
    private lateinit var presenter: LoginPresenter
    
    @Before
    fun setup() {
        MockitoAnnotations.openMocks(this)
        presenter = LoginPresenter(view, repository)
    }
    
    @Test
    fun `onLoginClicked with empty username shows error`() {
        presenter.onLoginClicked("", "password")
        
        verify(view).showError("Username required")
        verify(repository, never()).login(any(), any())
    }
    
    @Test
    fun `onLoginClicked with valid credentials navigates to home`() {
        whenever(repository.login(any(), any()))
            .thenReturn(LoginResult(true, "Success"))
        
        presenter.onLoginClicked("admin", "password")
        
        verify(view).navigateToHome()
    }
}
```

### MVVM Testing (Easy)

**Advantage:** ViewModel has no Android dependencies + LiveData/Flow testable

```kotlin
// ViewModel (Pure Kotlin + Coroutines)
class LoginViewModel(
    private val repository: LoginRepository
) : ViewModel() {
    
    private val _loginState = MutableStateFlow<LoginState>(LoginState.Idle)
    val loginState: StateFlow<LoginState> = _loginState.asStateFlow()
    
    fun login(username: String, password: String) = viewModelScope.launch {
        if (username.isEmpty()) {
            _loginState.value = LoginState.Error("Username required")
            return@launch
        }
        
        _loginState.value = LoginState.Loading
        
        val result = repository.login(username, password)
        _loginState.value = if (result.success) {
            LoginState.Success
        } else {
            LoginState.Error(result.message)
        }
    }
}

sealed class LoginState {
    object Idle : LoginState()
    object Loading : LoginState()
    object Success : LoginState()
    data class Error(val message: String) : LoginState()
}

// ✅ Very easy to test
class LoginViewModelTest {
    
    @Mock
    lateinit var repository: LoginRepository
    
    private lateinit var viewModel: LoginViewModel
    
    // Use test coroutine dispatcher
    private val testDispatcher = StandardTestDispatcher()
    
    @Before
    fun setup() {
        Dispatchers.setMain(testDispatcher)
        MockitoAnnotations.openMocks(this)
        viewModel = LoginViewModel(repository)
    }
    
    @After
    fun tearDown() {
        Dispatchers.resetMain()
    }
    
    @Test
    fun `login with empty username emits error state`() = runTest {
        viewModel.login("", "password")
        
        val state = viewModel.loginState.value
        assertTrue(state is LoginState.Error)
        assertEquals("Username required", (state as LoginState.Error).message)
    }
    
    @Test
    fun `login with valid credentials emits success state`() = runTest {
        whenever(repository.login(any(), any()))
            .thenReturn(LoginResult(true, "Success"))
        
        viewModel.login("admin", "password")
        testDispatcher.scheduler.advanceUntilIdle()
        
        val state = viewModel.loginState.value
        assertTrue(state is LoginState.Success)
    }
    
    @Test
    fun `login emits loading state before network call`() = runTest {
        whenever(repository.login(any(), any())).then {
            delay(100)
            LoginResult(true, "Success")
        }
        
        viewModel.login("admin", "password")
        
        // Check loading state immediately
        val loadingState = viewModel.loginState.value
        assertTrue(loadingState is LoginState.Loading)
        
        // Advance time
        testDispatcher.scheduler.advanceUntilIdle()
        
        // Check success state
        val successState = viewModel.loginState.value
        assertTrue(successState is LoginState.Success)
    }
}
```

### Testing Summary

| Pattern | **Testability** | **Mocking Required** | **Test Types** |
|---------|----------------|---------------------|----------------|
| **MVC** | ❌ Hard | Robolectric/Espresso | Instrumented only |
| **MVP** | ✅ Medium | View + Repository | Unit + Instrumented |
| **MVVM** | ✅ Easy | Repository only | Unit (fast) + Instrumented |

**Winner:** MVVM (pure Kotlin ViewModels, easy to test)

---

## 10. Modern Android Stack (2026)

### Industry Standard Architecture

```
┌───────────────────────────────────────────────────┐
│         MODERN ANDROID STACK (2026)               │
├───────────────────────────────────────────────────┤
│                                                   │
│  UI Layer                                         │
│  ├─ Jetpack Compose (UI Toolkit)                 │
│  ├─ Material 3 (Design System)                   │
│  └─ Navigation Component                          │
│                                                   │
│  Presentation Layer                               │
│  ├─ ViewModel (Lifecycle-aware)                  │
│  ├─ StateFlow / LiveData (Reactive state)        │
│  └─ Hilt (Dependency Injection)                  │
│                                                   │
│  Domain Layer (Optional - Clean Architecture)    │
│  └─ Use Cases / Interactors                      │
│                                                   │
│  Data Layer                                       │
│  ├─ Repository Pattern                           │
│  ├─ Room (Local Database)                        │
│  ├─ Retrofit + OkHttp (Network)                  │
│  ├─ DataStore (Preferences)                      │
│  └─ WorkManager (Background tasks)               │
│                                                   │
│  Cross-cutting Concerns                          │
│  ├─ Kotlin Coroutines (Async)                   │
│  ├─ Kotlin Flow (Reactive streams)               │
│  └─ Paging 3 (Pagination)                        │
└───────────────────────────────────────────────────┘
```

### Complete Modern Project Structure

```
app/
├─ di/                          # Dependency Injection
│  ├─ AppModule.kt
│  ├─ DatabaseModule.kt
│  └─ NetworkModule.kt
│
├─ data/                        # Data Layer
│  ├─ local/
│  │  ├─ dao/
│  │  │  └─ UserDao.kt
│  │  ├─ entity/
│  │  │  └─ UserEntity.kt
│  │  └─ AppDatabase.kt
│  │
│  ├─ remote/
│  │  ├─ api/
│  │  │  └─ UserApi.kt
│  │  └─ dto/
│  │     └─ UserDto.kt
│  │
│  └─ repository/
│     └─ UserRepositoryImpl.kt
│
├─ domain/                      # Domain Layer (Optional)
│  ├─ model/
│  │  └─ User.kt
│  ├─ repository/
│  │  └─ UserRepository.kt       # Interface
│  └─ usecase/
│     ├─ GetUsersUseCase.kt
│     └─ AddUserUseCase.kt
│
├─ presentation/                # Presentation Layer
│  ├─ ui/
│  │  ├─ user/
│  │  │  ├─ UserListScreen.kt    # Compose UI
│  │  │  ├─ UserViewModel.kt
│  │  │  └─ UserUiState.kt
│  │  │
│  │  └─ theme/
│  │     ├─ Color.kt
│  │     ├─ Theme.kt
│  │     └─ Type.kt
│  │
│  └─ navigation/
│     └─ NavGraph.kt
│
└─ util/                        # Utilities
   ├─ Resource.kt               # Result wrapper
   ├─ Constants.kt
   └─ Extensions.kt
```

### Key Technologies (2026)

**1. Jetpack Compose (UI)**
```kotlin
@Composable
fun UserListScreen(viewModel: UserViewModel = hiltViewModel()) {
    val uiState by viewModel.uiState.collectAsState()
    
    when (uiState) {
        is UiState.Loading -> LoadingIndicator()
        is UiState.Success -> UserList(users = uiState.data)
        is UiState.Error -> ErrorMessage(message = uiState.message)
    }
}
```

**2. Hilt (Dependency Injection)**
```kotlin
@HiltViewModel
class UserViewModel @Inject constructor(
    private val getUsersUseCase: GetUsersUseCase
) : ViewModel() { ... }

@AndroidEntryPoint
class MainActivity : ComponentActivity() { ... }
```

**3. Kotlin Coroutines + Flow**
```kotlin
fun getUsers(): Flow<List<User>> = flow {
    emit(Resource.Loading())
    
    try {
        val users = api.getUsers()
        emit(Resource.Success(users))
    } catch (e: Exception) {
        emit(Resource.Error(e.message))
    }
}.flowOn(Dispatchers.IO)
```

**4. Room Database**
```kotlin
@Entity
data class UserEntity(
    @PrimaryKey val id: Long,
    val name: String,
    val email: String
)

@Dao
interface UserDao {
    @Query("SELECT * FROM users")
    fun getAllUsers(): Flow<List<UserEntity>>
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertUsers(users: List<UserEntity>)
}
```

**5. Retrofit + OkHttp**
```kotlin
interface UserApi {
    @GET("users")
    suspend fun getUsers(): List<UserDto>
    
    @POST("users")
    suspend fun createUser(@Body user: UserDto): UserDto
}
```

### Build.gradle (2026 Dependencies)

```kotlin
// build.gradle.kts (Module: app)

dependencies {
    // Core
    implementation("androidx.core:core-ktx:1.12.0")
    implementation("androidx.lifecycle:lifecycle-runtime-ktx:2.7.0")
    
    // Compose
    implementation(platform("androidx.compose:compose-bom:2024.01.00"))
    implementation("androidx.compose.ui:ui")
    implementation("androidx.compose.material3:material3")
    implementation("androidx.compose.ui:ui-tooling-preview")
    implementation("androidx.activity:activity-compose:1.8.2")
    implementation("androidx.navigation:navigation-compose:2.7.6")
    
    // ViewModel
    implementation("androidx.lifecycle:lifecycle-viewmodel-ktx:2.7.0")
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.7.0")
    
    // Hilt
    implementation("com.google.dagger:hilt-android:2.50")
    kapt("com.google.dagger:hilt-compiler:2.50")
    implementation("androidx.hilt:hilt-navigation-compose:1.1.0")
    
    // Room
    implementation("androidx.room:room-runtime:2.6.1")
    implementation("androidx.room:room-ktx:2.6.1")
    kapt("androidx.room:room-compiler:2.6.1")
    
    // Retrofit
    implementation("com.squareup.retrofit2:retrofit:2.9.0")
    implementation("com.squareup.retrofit2:converter-gson:2.9.0")
    implementation("com.squareup.okhttp3:okhttp:4.12.0")
    implementation("com.squareup.okhttp3:logging-interceptor:4.12.0")
    
    // Coroutines
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.7.3")
    
    // DataStore
    implementation("androidx.datastore:datastore-preferences:1.0.0")
    
    // WorkManager
    implementation("androidx.work:work-runtime-ktx:2.9.0")
    
    // Paging
    implementation("androidx.paging:paging-runtime-ktx:3.2.1")
    implementation("androidx.paging:paging-compose:3.2.1")
    
    // Testing
    testImplementation("junit:junit:4.13.2")
    testImplementation("org.mockito.kotlin:mockito-kotlin:5.2.1")
    testImplementation("org.jetbrains.kotlinx:kotlinx-coroutines-test:1.7.3")
    testImplementation("app.cash.turbine:turbine:1.0.0")
    
    androidTestImplementation("androidx.test.ext:junit:1.1.5")
    androidTestImplementation("androidx.test.espresso:espresso-core:3.5.1")
    androidTestImplementation("androidx.compose.ui:ui-test-junit4")
}
```

---

## 11. Best Practices

### Universal Best Practices

**1. Single Responsibility Principle**
```kotlin
// ❌ BAD: Activity does everything
class MainActivity : AppCompatActivity() {
    fun loadUsers() { /* network */ }
    fun saveToDatabase() { /* database */ }
    fun validateInput() { /* validation */ }
}

// ✅ GOOD: Separated concerns
class UserRepository { fun loadUsers() }
class UserValidator { fun validate() }
class UserViewModel(repository, validator)
class MainActivity { /* only UI */ }
```

**2. Dependency Injection**
```kotlin
// ❌ BAD: Hard-coded dependencies
class UserViewModel {
    private val repository = UserRepositoryImpl()  // Hard to test!
}

// ✅ GOOD: Injected dependencies
class UserViewModel @Inject constructor(
    private val repository: UserRepository  // Easy to mock
)
```

**3. Lifecycle Awareness**
```kotlin
// ❌ BAD: No lifecycle handling
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        Thread {
            val data = loadData()
            textView.text = data  // CRASH if Activity destroyed!
        }.start()
    }
}

// ✅ GOOD: Lifecycle-aware
class MainActivity : AppCompatActivity() {
    private val viewModel: UserViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        lifecycleScope.launch {
            viewModel.data.collect { data ->
                textView.text = data  // Safe, lifecycle-aware
            }
        }
    }
}
```

**4. Error Handling**
```kotlin
// ❌ BAD: Exceptions propagate
suspend fun getUsers(): List<User> {
    return api.getUsers()  // Throws on error!
}

// ✅ GOOD: Wrapped result
suspend fun getUsers(): Resource<List<User>> {
    return try {
        val users = api.getUsers()
        Resource.Success(users)
    } catch (e: Exception) {
        Resource.Error(e.message)
    }
}
```

**5. Reactive Data**
```kotlin
// ❌ BAD: Manual UI updates
class ViewModel {
    fun loadData() {
        data = repository.getData()
        // How to notify UI?
    }
}

// ✅ GOOD: Observable data
class ViewModel : ViewModel() {
    private val _data = MutableStateFlow<List<Item>>(emptyList())
    val data: StateFlow<List<Item>> = _data.asStateFlow()
    
    fun loadData() = viewModelScope.launch {
        _data.value = repository.getData()
        // UI automatically updates!
    }
}
```

### MVVM-Specific Best Practices

**1. State Management**
```kotlin
// ✅ Single source of truth
data class UiState(
    val users: List<User> = emptyList(),
    val isLoading: Boolean = false,
    val error: String? = null
)

class UserViewModel : ViewModel() {
    private val _uiState = MutableStateFlow(UiState())
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()
    
    fun loadUsers() = viewModelScope.launch {
        _uiState.update { it.copy(isLoading = true) }
        
        try {
            val users = repository.getUsers()
            _uiState.update { it.copy(users = users, isLoading = false) }
        } catch (e: Exception) {
            _uiState.update { it.copy(error = e.message, isLoading = false) }
        }
    }
}
```

**2. Single Events**
```kotlin
// Handle one-time events (toast, navigation, etc.)
sealed class Event {
    object ShowSuccessToast : Event()
    data class NavigateToDetail(val userId: Long) : Event()
}

class UserViewModel : ViewModel() {
    private val _events = Channel<Event>()
    val events = _events.receiveAsFlow()
    
    fun onUserClicked(userId: Long) = viewModelScope.launch {
        _events.send(Event.NavigateToDetail(userId))
    }
}

// In Activity
lifecycleScope.launch {
    viewModel.events.collect { event ->
        when (event) {
            is Event.ShowSuccessToast -> showToast("Success")
            is Event.NavigateToDetail -> navigate(event.userId)
        }
    }
}
```

**3. Repository Pattern**
```kotlin
// ✅ Repository as single source of truth
class UserRepository(
    private val localDataSource: UserDao,
    private val remoteDataSource: UserApi
) {
    fun getUsers(): Flow<Resource<List<User>>> = flow {
        emit(Resource.Loading())
        
        // Emit cached data first
        val cachedUsers = localDataSource.getAllUsers().first()
        if (cachedUsers.isNotEmpty()) {
            emit(Resource.Success(cachedUsers))
        }
        
        // Fetch fresh data
        try {
            val remoteUsers = remoteDataSource.getUsers()
            localDataSource.insertAll(remoteUsers)
            emit(Resource.Success(remoteUsers))
        } catch (e: Exception) {
            if (cachedUsers.isEmpty()) {
                emit(Resource.Error(e.message))
            }
        }
    }
}
```

---

## 12. Migration Strategies

### MVC → MVVM Migration

**Step 1: Extract Model**
```kotlin
// Before (MVC)
class MainActivity : AppCompatActivity() {
    private val users = mutableListOf<User>()
    
    fun addUser(user: User) {
        users.add(user)
        updateUI()
    }
}

// After: Extract to Repository
class UserRepository {
    private val users = mutableListOf<User>()
    
    suspend fun addUser(user: User) {
        users.add(user)
    }
    
    fun getUsers(): List<User> = users.toList()
}
```

**Step 2: Create ViewModel**
```kotlin
class UserViewModel(
    private val repository: UserRepository
) : ViewModel() {
    
    private val _users = MutableStateFlow<List<User>>(emptyList())
    val users: StateFlow<List<User>> = _users.asStateFlow()
    
    init {
        loadUsers()
    }
    
    private fun loadUsers() = viewModelScope.launch {
        _users.value = repository.getUsers()
    }
    
    fun addUser(user: User) = viewModelScope.launch {
        repository.addUser(user)
        loadUsers()
    }
}
```

**Step 3: Refactor Activity**
```kotlin
// After: Activity only handles UI
class MainActivity : AppCompatActivity() {
    private val viewModel: UserViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        lifecycleScope.launch {
            viewModel.users.collect { users ->
                adapter.submitList(users)
            }
        }
        
        addButton.setOnClickListener {
            viewModel.addUser(createUser())
        }
    }
}
```

### MVP → MVVM Migration

**MVP (Before):**
```kotlin
interface UserContract {
    interface View {
        fun showUsers(users: List<User>)
    }
    interface Presenter {
        fun loadUsers()
    }
}

class UserPresenter(
    private val view: View,
    private val repository: UserRepository
) : Presenter {
    override fun loadUsers() {
        val users = repository.getUsers()
        view.showUsers(users)
    }
}
```

**MVVM (After):**
```kotlin
class UserViewModel(
    private val repository: UserRepository
) : ViewModel() {
    
    val users = repository.getUsersFlow()
        .stateIn(viewModelScope, SharingStarted.Lazily, emptyList())
}

class MainActivity : AppCompatActivity() {
    private val viewModel: UserViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        lifecycleScope.launch {
            viewModel.users.collect { users ->
                adapter.submitList(users)
            }
        }
    }
}
```

**Benefits:**
- ✅ Less boilerplate (no interfaces)
- ✅ Automatic lifecycle handling
- ✅ Reactive data flow

---

## 13. Complete Code Examples

### Example 1: Todo App (All Three Patterns)

**See sections 3, 4, 5 for complete implementations.**

### Example 2: Login Flow (All Three Patterns)

**See sections 3, 4, 5 for complete implementations.**

### Example 3: Real Production App (MVVM)

**Complete E-Commerce Product Listing:**

```kotlin
// ============ DATA LAYER ============

// Entity
@Entity(tableName = "products")
data class ProductEntity(
    @PrimaryKey val id: Long,
    val name: String,
    val price: Double,
    val imageUrl: String,
    val category: String,
    val inStock: Boolean,
    val rating: Float,
    val description: String
)

// DAO
@Dao
interface ProductDao {
    @Query("SELECT * FROM products WHERE category = :category ORDER BY rating DESC")
    fun getProductsByCategory(category: String): Flow<List<ProductEntity>>
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertProducts(products: List<ProductEntity>)
    
    @Query("DELETE FROM products WHERE category = :category")
    suspend fun deleteProductsByCategory(category: String)
}

// API
interface ProductApi {
    @GET("products")
    suspend fun getProducts(@Query("category") category: String): List<ProductDto>
}

// DTO
data class ProductDto(
    val id: Long,
    val name: String,
    val price: Double,
    val imageUrl: String,
    val category: String,
    val inStock: Boolean,
    val rating: Float,
    val description: String
) {
    fun toEntity() = ProductEntity(
        id, name, price, imageUrl, category, inStock, rating, description
    )
}

// Repository
class ProductRepository @Inject constructor(
    private val productDao: ProductDao,
    private val productApi: ProductApi
) {
    
    fun getProducts(category: String): Flow<Resource<List<ProductEntity>>> = flow {
        emit(Resource.Loading())
        
        // Emit cached data immediately
        val cachedProducts = productDao.getProductsByCategory(category).first()
        if (cachedProducts.isNotEmpty()) {
            emit(Resource.Success(cachedProducts))
        }
        
        // Fetch from network
        try {
            val remoteProducts = productApi.getProducts(category)
            val entities = remoteProducts.map { it.toEntity() }
            
            productDao.deleteProductsByCategory(category)
            productDao.insertProducts(entities)
            
            emit(Resource.Success(entities))
        } catch (e: HttpException) {
            if (cachedProducts.isEmpty()) {
                emit(Resource.Error("Network error: ${e.code()}"))
            }
        } catch (e: IOException) {
            if (cachedProducts.isEmpty()) {
                emit(Resource.Error("No internet connection"))
            }
        }
    }.flowOn(Dispatchers.IO)
}

// ============ DOMAIN LAYER (Optional) ============

data class Product(
    val id: Long,
    val name: String,
    val price: Double,
    val imageUrl: String,
    val category: String,
    val inStock: Boolean,
    val rating: Float,
    val description: String
)

fun ProductEntity.toDomain() = Product(
    id, name, price, imageUrl, category, inStock, rating, description
)

// ============ PRESENTATION LAYER ============

// UI State
sealed class ProductListUiState {
    object Loading : ProductListUiState()
    data class Success(val products: List<Product>) : ProductListUiState()
    data class Error(val message: String) : ProductListUiState()
}

// ViewModel
@HiltViewModel
class ProductListViewModel @Inject constructor(
    private val repository: ProductRepository,
    private val savedStateHandle: SavedStateHandle
) : ViewModel() {
    
    private val category = savedStateHandle.get<String>("category") ?: "electronics"
    
    private val _uiState = MutableStateFlow<ProductListUiState>(ProductListUiState.Loading)
    val uiState: StateFlow<ProductListUiState> = _uiState.asStateFlow()
    
    private val _events = Channel<ProductListEvent>()
    val events = _events.receiveAsFlow()
    
    init {
        loadProducts()
    }
    
    fun loadProducts() {
        viewModelScope.launch {
            repository.getProducts(category).collect { resource ->
                _uiState.value = when (resource) {
                    is Resource.Loading -> ProductListUiState.Loading
                    is Resource.Success -> ProductListUiState.Success(
                        resource.data.map { it.toDomain() }
                    )
                    is Resource.Error -> ProductListUiState.Error(resource.message ?: "Unknown error")
                }
            }
        }
    }
    
    fun onProductClicked(productId: Long) = viewModelScope.launch {
        _events.send(ProductListEvent.NavigateToDetail(productId))
    }
    
    fun onRefresh() {
        loadProducts()
    }
}

// Events
sealed class ProductListEvent {
    data class NavigateToDetail(val productId: Long) : ProductListEvent()
}

// ============ UI LAYER (Compose) ============

@Composable
fun ProductListScreen(
    viewModel: ProductListViewModel = hiltViewModel(),
    onNavigateToDetail: (Long) -> Unit
) {
    val uiState by viewModel.uiState.collectAsState()
    
    // Handle events
    LaunchedEffect(Unit) {
        viewModel.events.collect { event ->
            when (event) {
                is ProductListEvent.NavigateToDetail -> onNavigateToDetail(event.productId)
            }
        }
    }
    
    Column(modifier = Modifier.fillMaxSize()) {
        when (val state = uiState) {
            is ProductListUiState.Loading -> {
                CircularProgressIndicator(
                    modifier = Modifier.align(Alignment.CenterHorizontally)
                )
            }
            
            is ProductListUiState.Success -> {
                LazyColumn {
                    items(state.products) { product ->
                        ProductItem(
                            product = product,
                            onClick = { viewModel.onProductClicked(product.id) }
                        )
                    }
                }
            }
            
            is ProductListUiState.Error -> {
                ErrorMessage(
                    message = state.message,
                    onRetry = { viewModel.onRefresh() }
                )
            }
        }
    }
}

@Composable
fun ProductItem(product: Product, onClick: () -> Unit) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(8.dp)
            .clickable(onClick = onClick),
        elevation = CardDefaults.cardElevation(defaultElevation = 4.dp)
    ) {
        Row(modifier = Modifier.padding(16.dp)) {
            AsyncImage(
                model = product.imageUrl,
                contentDescription = product.name,
                modifier = Modifier.size(80.dp)
            )
            
            Spacer(modifier = Modifier.width(16.dp))
            
            Column {
                Text(
                    text = product.name,
                    style = MaterialTheme.typography.titleMedium
                )
                
                Text(
                    text = "$${product.price}",
                    style = MaterialTheme.typography.bodyLarge,
                    color = MaterialTheme.colorScheme.primary
                )
                
                Row(verticalAlignment = Alignment.CenterVertically) {
                    Icon(
                        imageVector = Icons.Default.Star,
                        contentDescription = null,
                        tint = Color.Yellow,
                        modifier = Modifier.size(16.dp)
                    )
                    Text(
                        text = product.rating.toString(),
                        style = MaterialTheme.typography.bodySmall
                    )
                }
                
                if (!product.inStock) {
                    Text(
                        text = "Out of Stock",
                        color = Color.Red,
                        style = MaterialTheme.typography.bodySmall
                    )
                }
            }
        }
    }
}

// ============ DI (Hilt) ============

@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    
    @Provides
    @Singleton
    fun provideProductApi(): ProductApi {
        return Retrofit.Builder()
            .baseUrl("https://api.example.com/")
            .addConverterFactory(GsonConverterFactory.create())
            .build()
            .create(ProductApi::class.java)
    }
    
    @Provides
    @Singleton
    fun provideProductDao(db: AppDatabase): ProductDao {
        return db.productDao()
    }
    
    @Provides
    @Singleton
    fun provideProductRepository(
        dao: ProductDao,
        api: ProductApi
    ): ProductRepository {
        return ProductRepository(dao, api)
    }
}
```

---

## Summary

### Key Takeaways

**1. MVC (Model-View-Controller)**
- ✅ Simple, beginner-friendly
- ❌ Untestable, God Activity problem
- 📉 Avoid for new projects

**2. MVP (Model-View-Presenter)**
- ✅ Testable Presenter, interface-based
- ❌ Boilerplate, manual lifecycle
- 📉 Legacy, use for existing projects only

**3. MVVM (Model-View-ViewModel)**
- ✅ Highly testable, lifecycle-aware, reactive
- ✅ Google recommended, Jetpack support
- 📈 **Use for ALL new projects in 2026**

### Decision Matrix

| Your Situation | **Choose** |
|---------------|-----------|
| New project 2026 | MVVM |
| Using Compose | MVVM |
| Need testing | MVVM |
| Learning Android | MVC → MVP → MVVM |
| Simple utility | MVC |
| Legacy project | Keep existing |
| Samsung daemon project | MVVM |

### Final Recommendation

**For Abhinav (Samsung R&D):**

Use **MVVM** for your middleware/daemon projects:

```
DaemonManagerApp
├─ UI: Jetpack Compose
├─ ViewModel: Daemon state management
├─ Repository: IPC + Binder + UDev
├─ Local: SharedPreferences (persistent state)
└─ Remote: System services (Binder IPC)
```

**Benefits for system software:**
- ✅ Real-time daemon monitoring (StateFlow)
- ✅ Survives configuration changes
- ✅ Easy to test IPC logic (mock repository)
- ✅ Clean separation: UI ← ViewModel ← Repository ← HAL/Binder

---

**Perfect for competitive programming interview prep + Samsung projects!** 🚀

---

**Created for**: Android Development Learning  
**Last Updated**: February 2026  
**Author**: Comprehensive Kotlin & Java Architecture Guide  
**Topics**: MVC, MVP, MVVM, Architecture Patterns, Testing, Best Practices  
**Target Audience**: Android Developers (Beginner to Advanced)
