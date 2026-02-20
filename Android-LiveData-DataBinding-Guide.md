# Android LiveData, Data Binding & Binding Adapters - Complete Guide

## Table of Contents
1. [LiveData Overview](#livedata-overview)
2. [View Binding](#view-binding)
3. [Data Binding](#data-binding)
4. [One-Way vs Two-Way Binding](#one-way-vs-two-way-binding)
5. [Binding Adapters (Custom Data Binding)](#binding-adapters)
6. [Complete Examples](#complete-examples)
7. [Comparison Tables](#comparison-tables)
8. [Best Practices](#best-practices)

---

## LiveData Overview

### What is LiveData?
**LiveData** is a lifecycle-aware observable data holder class that respects the lifecycle of Android components (Activity, Fragment). It only updates observers when they are in an active lifecycle state.

### Key Features
- ✅ **Lifecycle-aware**: Automatically manages subscriptions based on lifecycle state
- ✅ **No memory leaks**: Observers are automatically cleaned up when lifecycle is destroyed
- ✅ **No crashes**: UI updates only when active (STARTED or RESUMED state)
- ✅ **Automatic UI updates**: Observers receive latest data when they become active

### LiveData - Kotlin Implementation

```kotlin
// ViewModel with LiveData
class UserViewModel : ViewModel() {
    // Private MutableLiveData (only ViewModel can modify)
    private val _userName = MutableLiveData<String>()
    
    // Public immutable LiveData (UI observes this)
    val userName: LiveData<String> = _userName
    
    // Private mutable, public immutable pattern
    private val _userAge = MutableLiveData<Int>(0)
    val userAge: LiveData<Int> = _userAge
    
    // Update methods
    fun updateUserName(name: String) {
        _userName.value = name  // Main thread
    }
    
    fun updateUserNameAsync(name: String) {
        _userName.postValue(name)  // Background thread safe
    }
    
    fun incrementAge() {
        _userAge.value = (_userAge.value ?: 0) + 1
    }
}

// Activity observing LiveData
class MainActivity : AppCompatActivity() {
    private lateinit var viewModel: UserViewModel
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        viewModel = ViewModelProvider(this)[UserViewModel::class.java]
        
        // Observe LiveData - automatically lifecycle-aware
        viewModel.userName.observe(this) { name ->
            textViewName.text = name
            Log.d("LiveData", "Name updated: $name")
        }
        
        viewModel.userAge.observe(this, Observer { age ->
            textViewAge.text = "Age: $age"
        })
        
        // Update data
        button.setOnClickListener {
            viewModel.updateUserName("Abhinav Maurya")
            viewModel.incrementAge()
        }
    }
}
```

### LiveData - Java Implementation

```java
// ViewModel with LiveData
public class UserViewModel extends ViewModel {
    // Private MutableLiveData
    private MutableLiveData<String> userName = new MutableLiveData<>();
    private MutableLiveData<Integer> userAge = new MutableLiveData<>(0);
    
    // Public getters returning immutable LiveData
    public LiveData<String> getUserName() {
        return userName;
    }
    
    public LiveData<Integer> getUserAge() {
        return userAge;
    }
    
    // Update methods
    public void updateUserName(String name) {
        userName.setValue(name);  // Main thread
    }
    
    public void updateUserNameAsync(String name) {
        userName.postValue(name);  // Background thread
    }
    
    public void incrementAge() {
        Integer current = userAge.getValue();
        userAge.setValue(current == null ? 1 : current + 1);
    }
}

// Activity observing LiveData
public class MainActivity extends AppCompatActivity {
    private UserViewModel viewModel;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        viewModel = new ViewModelProvider(this).get(UserViewModel.class);
        
        // Observe LiveData
        viewModel.getUserName().observe(this, new Observer<String>() {
            @Override
            public void onChanged(String name) {
                textViewName.setText(name);
                Log.d("LiveData", "Name updated: " + name);
            }
        });
        
        viewModel.getUserAge().observe(this, age -> {
            textViewAge.setText("Age: " + age);
        });
        
        // Update data
        button.setOnClickListener(v -> {
            viewModel.updateUserName("Abhinav Maurya");
            viewModel.incrementAge();
        });
    }
}
```

---

## View Binding

### What is View Binding?
**View Binding** generates a binding class for each XML layout, providing type-safe and null-safe access to views. It's a simpler alternative to `findViewById()`.

### Setup
**build.gradle (Module: app)**
```gradle
android {
    viewBinding {
        enabled = true
    }
}
```

### View Binding - Kotlin Implementation

```kotlin
// Activity with ViewBinding
class MainActivity : AppCompatActivity() {
    // Declare binding variable
    private lateinit var binding: ActivityMainBinding
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Inflate binding
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        
        // Type-safe view access (no findViewById!)
        binding.textViewTitle.text = "Hello ViewBinding"
        binding.buttonSubmit.setOnClickListener {
            val input = binding.editTextName.text.toString()
            binding.textViewResult.text = "Hello, $input"
        }
        
        // Null-safe - if view doesn't exist, compilation fails
        binding.recyclerView.layoutManager = LinearLayoutManager(this)
    }
    
    // Clean up binding (optional but recommended)
    override fun onDestroy() {
        super.onDestroy()
        // binding = null // Not needed for Activity
    }
}

// Fragment with ViewBinding (requires cleanup)
class HomeFragment : Fragment() {
    private var _binding: FragmentHomeBinding? = null
    private val binding get() = _binding!!
    
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        _binding = FragmentHomeBinding.inflate(inflater, container, false)
        return binding.root
    }
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        binding.textView.text = "Fragment with ViewBinding"
        binding.button.setOnClickListener {
            // Handle click
        }
    }
    
    // IMPORTANT: Clean up binding in Fragment
    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null  // Prevent memory leaks
    }
}
```

### View Binding - Java Implementation

```java
// Activity with ViewBinding
public class MainActivity extends AppCompatActivity {
    private ActivityMainBinding binding;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        
        // Inflate binding
        binding = ActivityMainBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());
        
        // Type-safe view access
        binding.textViewTitle.setText("Hello ViewBinding");
        binding.buttonSubmit.setOnClickListener(v -> {
            String input = binding.editTextName.getText().toString();
            binding.textViewResult.setText("Hello, " + input);
        });
        
        binding.recyclerView.setLayoutManager(new LinearLayoutManager(this));
    }
}

// Fragment with ViewBinding
public class HomeFragment extends Fragment {
    private FragmentHomeBinding binding;
    
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        binding = FragmentHomeBinding.inflate(inflater, container, false);
        return binding.getRoot();
    }
    
    @Override
    public void onViewCreated(@NonNull View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        
        binding.textView.setText("Fragment with ViewBinding");
        binding.button.setOnClickListener(v -> {
            // Handle click
        });
    }
    
    @Override
    public void onDestroyView() {
        super.onDestroyView();
        binding = null;  // Prevent memory leaks
    }
}
```

---

## Data Binding

### What is Data Binding?
**Data Binding** extends View Binding with the ability to bind UI components directly to data sources using declarative XML syntax. It supports expressions like `@{viewModel.name}`.

### Setup
**build.gradle (Module: app)**
```gradle
android {
    dataBinding {
        enabled = true
    }
}
```

### Layout Structure
Data Binding requires wrapping your layout in a `<layout>` tag:

```xml
<!-- activity_main.xml -->
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    
    <!-- Data section - declare variables -->
    <data>
        <variable
            name="viewModel"
            type="com.example.app.MainViewModel"/>
        
        <variable
            name="user"
            type="com.example.app.User"/>
    </data>
    
    <!-- Your regular layout -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:padding="16dp">
        
        <!-- Bind data using expressions -->
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{user.name}"/>
        
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{String.valueOf(user.age)}"/>
        
        <!-- Bind click listeners -->
        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Increment Age"
            android:onClick="@{() -> viewModel.incrementAge()}"/>
        
        <!-- Bind visibility -->
        <ProgressBar
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:visibility="@{viewModel.isLoading ? View.VISIBLE : View.GONE}"/>
        
    </LinearLayout>
</layout>
```

### Data Binding - Kotlin Implementation

```kotlin
// Data Model
data class User(
    val name: String,
    val age: Int,
    val email: String
)

// ViewModel with LiveData
class MainViewModel : ViewModel() {
    private val _user = MutableLiveData<User>()
    val user: LiveData<User> = _user
    
    private val _isLoading = MutableLiveData<Boolean>(false)
    val isLoading: LiveData<Boolean> = _isLoading
    
    init {
        _user.value = User("Abhinav Maurya", 25, "abhinav@samsung.com")
    }
    
    fun incrementAge() {
        _user.value = _user.value?.copy(age = (_user.value?.age ?: 0) + 1)
    }
    
    fun updateName(newName: String) {
        _user.value = _user.value?.copy(name = newName)
    }
}

// Activity with Data Binding
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding
    private lateinit var viewModel: MainViewModel
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Inflate with DataBindingUtil
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
        
        // Get ViewModel
        viewModel = ViewModelProvider(this)[MainViewModel::class.java]
        
        // Set variables in layout
        binding.viewModel = viewModel
        
        // CRITICAL: Set lifecycle owner for LiveData to work with Data Binding
        binding.lifecycleOwner = this
        
        // Now layout automatically updates when LiveData changes!
        // No need for observe() calls for bound LiveData
    }
}
```

### Data Binding - Java Implementation

```java
// Data Model
public class User {
    private String name;
    private int age;
    private String email;
    
    public User(String name, int age, String email) {
        this.name = name;
        this.age = age;
        this.email = email;
    }
    
    // Getters (required for Data Binding)
    public String getName() { return name; }
    public int getAge() { return age; }
    public String getEmail() { return email; }
    
    // Setters
    public void setName(String name) { this.name = name; }
    public void setAge(int age) { this.age = age; }
    public void setEmail(String email) { this.email = email; }
}

// ViewModel with LiveData
public class MainViewModel extends ViewModel {
    private MutableLiveData<User> user = new MutableLiveData<>();
    private MutableLiveData<Boolean> isLoading = new MutableLiveData<>(false);
    
    public MainViewModel() {
        user.setValue(new User("Abhinav Maurya", 25, "abhinav@samsung.com"));
    }
    
    public LiveData<User> getUser() { return user; }
    public LiveData<Boolean> getIsLoading() { return isLoading; }
    
    public void incrementAge() {
        User current = user.getValue();
        if (current != null) {
            current.setAge(current.getAge() + 1);
            user.setValue(current);
        }
    }
    
    public void updateName(String newName) {
        User current = user.getValue();
        if (current != null) {
            current.setName(newName);
            user.setValue(current);
        }
    }
}

// Activity with Data Binding
public class MainActivity extends AppCompatActivity {
    private ActivityMainBinding binding;
    private MainViewModel viewModel;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        
        // Inflate with DataBindingUtil
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main);
        
        // Get ViewModel
        viewModel = new ViewModelProvider(this).get(MainViewModel.class);
        
        // Set variables in layout
        binding.setViewModel(viewModel);
        
        // CRITICAL: Set lifecycle owner
        binding.setLifecycleOwner(this);
    }
}
```

---

## One-Way vs Two-Way Binding

### One-Way Binding: `@{}`
**Data flows from ViewModel → UI only**

```xml
<!-- Layout - One-Way -->
<layout>
    <data>
        <variable name="viewModel" type="com.example.MainViewModel"/>
    </data>
    
    <LinearLayout>
        <!-- ViewModel to UI (read-only) -->
        <TextView
            android:text="@{viewModel.userName}"/>
        
        <!-- User cannot change viewModel.userName through this TextView -->
    </LinearLayout>
</layout>
```

**Kotlin**:
```kotlin
class MainViewModel : ViewModel() {
    private val _userName = MutableLiveData<String>("Abhinav")
    val userName: LiveData<String> = _userName
    
    fun updateName(name: String) {
        _userName.value = name  // Only ViewModel can update
    }
}
```

### Two-Way Binding: `@={}`
**Data flows both ways: ViewModel ↔ UI**

```xml
<!-- Layout - Two-Way -->
<layout>
    <data>
        <variable name="viewModel" type="com.example.MainViewModel"/>
    </data>
    
    <LinearLayout>
        <!-- Two-way binding with @={} -->
        <EditText
            android:text="@={viewModel.searchQuery}"/>
        
        <!-- When user types, viewModel.searchQuery automatically updates! -->
        
        <TextView
            android:text="@{viewModel.searchQuery}"/>
        <!-- Shows same text as EditText, stays in sync -->
    </LinearLayout>
</layout>
```

**Kotlin - Two-Way Binding**:
```kotlin
class SearchViewModel : ViewModel() {
    // MUST be MutableLiveData for two-way binding
    val searchQuery = MutableLiveData<String>("")
    
    // Observe changes from UI
    init {
        searchQuery.observeForever { query ->
            Log.d("Search", "User typed: $query")
            // Perform search automatically
            performSearch(query)
        }
    }
    
    private fun performSearch(query: String) {
        // API call or database query
    }
}
```

**Java - Two-Way Binding**:
```java
public class SearchViewModel extends ViewModel {
    // Must be public MutableLiveData
    public MutableLiveData<String> searchQuery = new MutableLiveData<>("");
    
    public SearchViewModel() {
        // Observe changes from UI
        searchQuery.observeForever(query -> {
            Log.d("Search", "User typed: " + query);
            performSearch(query);
        });
    }
    
    private void performSearch(String query) {
        // API call or database query
    }
}
```

### Complete Two-Way Example

**Layout**:
```xml
<layout>
    <data>
        <variable name="viewModel" type="com.example.LoginViewModel"/>
    </data>
    
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:padding="16dp">
        
        <!-- Email input with two-way binding -->
        <EditText
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="Email"
            android:text="@={viewModel.email}"/>
        
        <!-- Password input with two-way binding -->
        <EditText
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="Password"
            android:inputType="textPassword"
            android:text="@={viewModel.password}"/>
        
        <!-- Submit button enabled only when valid -->
        <Button
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Login"
            android:enabled="@{viewModel.isValid}"
            android:onClick="@{() -> viewModel.login()}"/>
        
        <!-- Error message (one-way) -->
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@{viewModel.errorMessage}"
            android:textColor="#FF0000"
            android:visibility="@{viewModel.errorMessage != null ? View.VISIBLE : View.GONE}"/>
        
    </LinearLayout>
</layout>
```

**Kotlin ViewModel**:
```kotlin
class LoginViewModel : ViewModel() {
    // Two-way binding - user can edit
    val email = MutableLiveData<String>("")
    val password = MutableLiveData<String>("")
    
    // One-way binding - ViewModel controls
    private val _errorMessage = MutableLiveData<String?>()
    val errorMessage: LiveData<String?> = _errorMessage
    
    // Computed property using MediatorLiveData
    val isValid: LiveData<Boolean> = MediatorLiveData<Boolean>().apply {
        addSource(email) { value = validate() }
        addSource(password) { value = validate() }
    }
    
    private fun validate(): Boolean {
        val emailValid = email.value?.contains("@") == true
        val passwordValid = (password.value?.length ?: 0) >= 6
        return emailValid && passwordValid
    }
    
    fun login() {
        if (validate()) {
            // Perform login
            viewModelScope.launch {
                try {
                    repository.login(email.value!!, password.value!!)
                    _errorMessage.value = null
                } catch (e: Exception) {
                    _errorMessage.value = e.message
                }
            }
        }
    }
}
```

---

## Binding Adapters (Custom Data Binding)

### What are Binding Adapters?
**Binding Adapters** are methods that let you create custom XML attributes for Data Binding. They execute custom logic when setting values on views.

### Use Cases
- ✅ Loading images from URLs (Glide/Picasso/Coil)
- ✅ Custom view visibility logic
- ✅ Formatting text (currency, dates)
- ✅ RecyclerView adapter submission
- ✅ Any custom view behavior

### Binding Adapters - Kotlin Implementation

```kotlin
// Create BindingAdapters.kt file
import android.view.View
import android.widget.ImageView
import android.widget.TextView
import androidx.databinding.BindingAdapter
import androidx.recyclerview.widget.RecyclerView
import com.bumptech.glide.Glide
import java.text.SimpleDateFormat
import java.util.*

// 1. Load image from URL using Glide
@BindingAdapter("imageUrl")
fun ImageView.loadImage(url: String?) {
    url?.let {
        Glide.with(context)
            .load(it)
            .placeholder(R.drawable.placeholder)
            .error(R.drawable.error)
            .into(this)
    }
}

// 2. Custom visibility based on boolean
@BindingAdapter("visibleIf")
fun View.setVisibleIf(visible: Boolean) {
    visibility = if (visible) View.VISIBLE else View.GONE
}

// 3. Hide if condition true
@BindingAdapter("hideIf")
fun View.setHideIf(hide: Boolean) {
    visibility = if (hide) View.GONE else View.VISIBLE
}

// 4. Format date to string
@BindingAdapter("dateText")
fun TextView.setDateText(date: Date?) {
    date?.let {
        val formatter = SimpleDateFormat("MMM dd, yyyy", Locale.getDefault())
        text = formatter.format(it)
    }
}

// 5. Format currency
@BindingAdapter("currencyText")
fun TextView.setCurrencyText(amount: Double?) {
    text = amount?.let { "₹%.2f".format(it) } ?: "₹0.00"
}

// 6. Submit list to RecyclerView adapter
@BindingAdapter("submitList")
fun <T> RecyclerView.submitList(items: List<T>?) {
    (adapter as? GenericAdapter<T>)?.submitList(items ?: emptyList())
}

// 7. Multiple attributes - Load image with placeholder
@BindingAdapter(value = ["imageUrl", "placeholder"], requireAll = false)
fun ImageView.loadImageWithPlaceholder(url: String?, placeholderId: Int?) {
    Glide.with(context)
        .load(url)
        .placeholder(placeholderId ?: R.drawable.placeholder)
        .into(this)
}

// 8. Set text color based on status
@BindingAdapter("statusColor")
fun TextView.setStatusColor(status: String?) {
    setTextColor(when (status) {
        "active" -> context.getColor(android.R.color.holo_green_dark)
        "pending" -> context.getColor(android.R.color.holo_orange_dark)
        "inactive" -> context.getColor(android.R.color.holo_red_dark)
        else -> context.getColor(android.R.color.black)
    })
}

// 9. Background color binding
@BindingAdapter("backgroundColor")
fun View.setBackgroundColorFromHex(colorHex: String?) {
    colorHex?.let {
        try {
            setBackgroundColor(Color.parseColor(it))
        } catch (e: IllegalArgumentException) {
            // Invalid color format
        }
    }
}

// 10. Enable/disable view based on LiveData
@BindingAdapter("enabledWhen")
fun View.setEnabledWhen(enabled: Boolean) {
    isEnabled = enabled
    alpha = if (enabled) 1.0f else 0.5f
}
```

### Using Binding Adapters in XML

```xml
<layout>
    <data>
        <variable name="user" type="com.example.User"/>
        <variable name="viewModel" type="com.example.ProfileViewModel"/>
    </data>
    
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:padding="16dp">
        
        <!-- Custom imageUrl attribute -->
        <ImageView
            android:layout_width="100dp"
            android:layout_height="100dp"
            app:imageUrl="@{user.profileImageUrl}"/>
        
        <!-- Custom visibility attribute -->
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{user.name}"
            app:visibleIf="@{user.name != null}"/>
        
        <!-- Custom currency formatting -->
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            app:currencyText="@{user.salary}"/>
        
        <!-- Custom date formatting -->
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            app:dateText="@{user.joinDate}"/>
        
        <!-- Custom status color -->
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{user.status}"
            app:statusColor="@{user.status}"/>
        
        <!-- RecyclerView with custom submitList -->
        <androidx.recyclerview.widget.RecyclerView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            app:submitList="@{viewModel.userList}"/>
        
        <!-- Button enabled based on validation -->
        <Button
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Submit"
            app:enabledWhen="@{viewModel.isFormValid}"/>
        
    </LinearLayout>
</layout>
```

### Binding Adapters - Java Implementation

```java
// BindingAdapters.java
import android.view.View;
import android.widget.ImageView;
import android.widget.TextView;
import androidx.databinding.BindingAdapter;
import androidx.recyclerview.widget.RecyclerView;
import com.bumptech.glide.Glide;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;
import java.util.Locale;

public class BindingAdapters {
    
    // 1. Load image from URL
    @BindingAdapter("imageUrl")
    public static void loadImage(ImageView view, String url) {
        if (url != null && !url.isEmpty()) {
            Glide.with(view.getContext())
                .load(url)
                .placeholder(R.drawable.placeholder)
                .error(R.drawable.error)
                .into(view);
        }
    }
    
    // 2. Custom visibility
    @BindingAdapter("visibleIf")
    public static void setVisibleIf(View view, boolean visible) {
        view.setVisibility(visible ? View.VISIBLE : View.GONE);
    }
    
    // 3. Hide if condition
    @BindingAdapter("hideIf")
    public static void setHideIf(View view, boolean hide) {
        view.setVisibility(hide ? View.GONE : View.VISIBLE);
    }
    
    // 4. Format date
    @BindingAdapter("dateText")
    public static void setDateText(TextView view, Date date) {
        if (date != null) {
            SimpleDateFormat formatter = new SimpleDateFormat("MMM dd, yyyy", Locale.getDefault());
            view.setText(formatter.format(date));
        }
    }
    
    // 5. Format currency
    @BindingAdapter("currencyText")
    public static void setCurrencyText(TextView view, Double amount) {
        String formatted = (amount != null) ? 
            String.format("₹%.2f", amount) : "₹0.00";
        view.setText(formatted);
    }
    
    // 6. RecyclerView submit list
    @BindingAdapter("submitList")
    public static <T> void submitList(RecyclerView view, List<T> items) {
        RecyclerView.Adapter adapter = view.getAdapter();
        if (adapter instanceof GenericAdapter) {
            ((GenericAdapter<T>) adapter).submitList(items != null ? items : new ArrayList<>());
        }
    }
    
    // 7. Multiple attributes
    @BindingAdapter(value = {"imageUrl", "placeholder"}, requireAll = false)
    public static void loadImageWithPlaceholder(ImageView view, String url, Integer placeholderId) {
        int placeholder = (placeholderId != null) ? placeholderId : R.drawable.placeholder;
        
        Glide.with(view.getContext())
            .load(url)
            .placeholder(placeholder)
            .into(view);
    }
    
    // 8. Status color
    @BindingAdapter("statusColor")
    public static void setStatusColor(TextView view, String status) {
        int color;
        if ("active".equals(status)) {
            color = view.getContext().getColor(android.R.color.holo_green_dark);
        } else if ("pending".equals(status)) {
            color = view.getContext().getColor(android.R.color.holo_orange_dark);
        } else if ("inactive".equals(status)) {
            color = view.getContext().getColor(android.R.color.holo_red_dark);
        } else {
            color = view.getContext().getColor(android.R.color.black);
        }
        view.setTextColor(color);
    }
    
    // 9. Enable/disable with alpha
    @BindingAdapter("enabledWhen")
    public static void setEnabledWhen(View view, boolean enabled) {
        view.setEnabled(enabled);
        view.setAlpha(enabled ? 1.0f : 0.5f);
    }
}
```

---

## Complete Examples

### Example 1: User Profile Screen with Data Binding

**User.kt**:
```kotlin
data class User(
    val id: Int,
    val name: String,
    val email: String,
    val profileImageUrl: String,
    val status: String,  // "active", "pending", "inactive"
    val salary: Double,
    val joinDate: Date
)
```

**ProfileViewModel.kt**:
```kotlin
class ProfileViewModel(private val userId: Int) : ViewModel() {
    private val _user = MutableLiveData<User>()
    val user: LiveData<User> = _user
    
    private val _isLoading = MutableLiveData<Boolean>(false)
    val isLoading: LiveData<Boolean> = _isLoading
    
    init {
        loadUser()
    }
    
    private fun loadUser() {
        viewModelScope.launch {
            _isLoading.value = true
            try {
                _user.value = repository.getUserById(userId)
            } catch (e: Exception) {
                // Handle error
            } finally {
                _isLoading.value = false
            }
        }
    }
    
    fun updateStatus(newStatus: String) {
        _user.value = _user.value?.copy(status = newStatus)
    }
}
```

**activity_profile.xml**:
```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    
    <data>
        <import type="android.view.View"/>
        <variable
            name="viewModel"
            type="com.example.ProfileViewModel"/>
    </data>
    
    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:padding="16dp">
            
            <!-- Loading indicator -->
            <ProgressBar
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="center"
                app:visibleIf="@{viewModel.isLoading}"/>
            
            <!-- Profile content -->
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                app:hideIf="@{viewModel.isLoading}">
                
                <!-- Profile image with Glide -->
                <ImageView
                    android:layout_width="120dp"
                    android:layout_height="120dp"
                    android:layout_gravity="center"
                    app:imageUrl="@{viewModel.user.profileImageUrl}"/>
                
                <!-- Name -->
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="16dp"
                    android:text="@{viewModel.user.name}"
                    android:textSize="24sp"
                    android:textStyle="bold"
                    android:gravity="center"/>
                
                <!-- Email -->
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="8dp"
                    android:text="@{viewModel.user.email}"
                    android:gravity="center"/>
                
                <!-- Status with custom color -->
                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_gravity="center"
                    android:layout_marginTop="8dp"
                    android:text="@{viewModel.user.status.toUpperCase()}"
                    app:statusColor="@{viewModel.user.status}"/>
                
                <!-- Salary with currency formatting -->
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="16dp"
                    android:text="Salary:"
                    android:textStyle="bold"/>
                
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    app:currencyText="@{viewModel.user.salary}"/>
                
                <!-- Join date with date formatting -->
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="16dp"
                    android:text="Joined:"
                    android:textStyle="bold"/>
                
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    app:dateText="@{viewModel.user.joinDate}"/>
                
            </LinearLayout>
            
        </LinearLayout>
    </ScrollView>
</layout>
```

**ProfileActivity.kt**:
```kotlin
class ProfileActivity : AppCompatActivity() {
    private lateinit var binding: ActivityProfileBinding
    private lateinit var viewModel: ProfileViewModel
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        binding = DataBindingUtil.setContentView(this, R.layout.activity_profile)
        
        val userId = intent.getIntExtra("USER_ID", 1)
        viewModel = ViewModelProvider(
            this,
            ProfileViewModelFactory(userId)
        )[ProfileViewModel::class.java]
        
        binding.viewModel = viewModel
        binding.lifecycleOwner = this
    }
}
```

### Example 2: Search with Two-Way Binding

**SearchViewModel.kt**:
```kotlin
class SearchViewModel : ViewModel() {
    // Two-way binding - user types here
    val searchQuery = MutableLiveData<String>("")
    
    private val _searchResults = MutableLiveData<List<Product>>()
    val searchResults: LiveData<List<Product>> = _searchResults
    
    private val _isSearching = MutableLiveData<Boolean>(false)
    val isSearching: LiveData<Boolean> = _isSearching
    
    init {
        // Observe search query changes
        searchQuery.observeForever { query ->
            if (query.isNotEmpty()) {
                performSearch(query)
            } else {
                _searchResults.value = emptyList()
            }
        }
    }
    
    private fun performSearch(query: String) {
        viewModelScope.launch {
            _isSearching.value = true
            delay(300) // Debounce
            try {
                _searchResults.value = repository.searchProducts(query)
            } catch (e: Exception) {
                _searchResults.value = emptyList()
            } finally {
                _isSearching.value = false
            }
        }
    }
    
    fun clearSearch() {
        searchQuery.value = ""
    }
}
```

**activity_search.xml**:
```xml
<layout>
    <data>
        <variable name="viewModel" type="com.example.SearchViewModel"/>
    </data>
    
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">
        
        <!-- Search box with two-way binding -->
        <EditText
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="Search products..."
            android:text="@={viewModel.searchQuery}"/>
        
        <!-- Clear button -->
        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Clear"
            android:onClick="@{() -> viewModel.clearSearch()}"
            app:visibleIf="@{!viewModel.searchQuery.empty}"/>
        
        <!-- Loading indicator -->
        <ProgressBar
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            app:visibleIf="@{viewModel.isSearching}"/>
        
        <!-- Results RecyclerView -->
        <androidx.recyclerview.widget.RecyclerView
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="1"
            app:submitList="@{viewModel.searchResults}"
            app:hideIf="@{viewModel.isSearching}"/>
        
    </LinearLayout>
</layout>
```

---

## Comparison Tables

### ViewBinding vs DataBinding

| Feature | **View Binding** | **Data Binding** |
|---------|-----------------|------------------|
| **Setup** | `viewBinding { enabled true }` | `dataBinding { enabled true }` |
| **Null Safety** | ✅ Yes | ✅ Yes |
| **Type Safety** | ✅ Yes | ✅ Yes |
| **findViewById** | ❌ Not needed | ❌ Not needed |
| **XML Expressions** | ❌ No | ✅ Yes (`@{}`) |
| **Layout Tag** | Standard XML | `<layout>` wrapper |
| **Two-Way Binding** | ❌ No | ✅ Yes (`@={`}`) |
| **Custom Attributes** | ❌ No | ✅ BindingAdapter |
| **Build Time** | Faster | Slower |
| **Binary Size** | Smaller | Slightly larger |
| **Learning Curve** | Easy | Moderate |
| **Best For** | Simple layouts | Complex UIs, MVVM |

### LiveData vs StateFlow

| Feature | **LiveData** | **StateFlow** |
|---------|-------------|---------------|
| **Platform** | Android-specific | Kotlin Multiplatform |
| **Lifecycle** | ✅ Aware by default | ⚠️ Manual with `collectAsStateWithLifecycle()` |
| **Initial Value** | ❌ Optional | ✅ Required |
| **Threading** | Main thread | Any thread |
| **Operators** | ⚠️ Limited | ✅ Rich (map, filter, combine) |
| **Backpressure** | ❌ No | ✅ Yes |
| **Compose** | ⚠️ Works with observeAsState | ✅ Native support |
| **Testing** | ⚠️ Requires InstantTaskExecutorRule | ✅ Easy with runTest |
| **Cold/Hot** | Hot stream | Hot stream |
| **Best For** | Traditional Android UI | Compose, multiplatform |

### One-Way vs Two-Way Binding

| Aspect | **One-Way `@{}`** | **Two-Way `@={}`** |
|--------|------------------|-------------------|
| **Data Flow** | ViewModel → UI | ViewModel ↔ UI |
| **Use Case** | Display data | User input forms |
| **Performance** | Faster | Slightly slower |
| **Complexity** | Simple | More complex |
| **Example** | TextView, ImageView | EditText, CheckBox, Switch |
| **LiveData Type** | LiveData (immutable) | MutableLiveData |
| **User Interaction** | Read-only | Editable |

---

## Best Practices

### LiveData Best Practices

1. **✅ DO**: Expose immutable LiveData publicly
```kotlin
private val _data = MutableLiveData<String>()
val data: LiveData<String> = _data  // Immutable
```

2. **✅ DO**: Use `observeForever` with caution (manual cleanup needed)
```kotlin
viewModel.data.observeForever { value ->
    // Remember to remove observer manually
}
```

3. **❌ DON'T**: Expose MutableLiveData publicly
```kotlin
val data = MutableLiveData<String>()  // BAD - anyone can modify
```

4. **✅ DO**: Use `postValue()` for background threads
```kotlin
viewModelScope.launch(Dispatchers.IO) {
    _data.postValue("Updated from background")  // Thread-safe
}
```

### Data Binding Best Practices

1. **✅ DO**: Set lifecycleOwner for LiveData in Data Binding
```kotlin
binding.lifecycleOwner = this  // Critical for LiveData updates
```

2. **✅ DO**: Keep expressions simple in XML
```xml
<!-- Good -->
<TextView android:text="@{user.name}"/>

<!-- Avoid complex logic -->
<TextView android:text="@{user.age > 18 ? 'Adult' : 'Minor'}"/>  
<!-- Better: Move to ViewModel -->
```

3. **✅ DO**: Use BindingAdapters for reusable custom logic
```kotlin
@BindingAdapter("loadImage")
fun ImageView.loadImage(url: String?) {
    // Reusable across entire app
}
```

4. **❌ DON'T**: Put business logic in XML
```xml
<!-- BAD -->
<Button android:onClick="@{() -> repository.deleteUser(user.id)}"/>

<!-- GOOD -->
<Button android:onClick="@{() -> viewModel.deleteUser()}"/>
```

### View Binding Best Practices

1. **✅ DO**: Clean up binding in Fragments
```kotlin
override fun onDestroyView() {
    super.onDestroyView()
    _binding = null  // Prevent memory leaks
}
```

2. **✅ DO**: Use lazy initialization for Activity binding
```kotlin
private val binding by lazy {
    ActivityMainBinding.inflate(layoutInflater)
}
```

3. **✅ DO**: Use ViewBinding for simple layouts
```kotlin
// If no expressions needed, prefer ViewBinding over DataBinding
binding.textView.text = "Simple text"
```

### Binding Adapters Best Practices

1. **✅ DO**: Use descriptive names
```kotlin
@BindingAdapter("imageUrl")  // Clear purpose
fun ImageView.loadImage(url: String?) { }
```

2. **✅ DO**: Handle null values gracefully
```kotlin
@BindingAdapter("imageUrl")
fun ImageView.loadImage(url: String?) {
    url?.let {  // Null-safe
        Glide.with(context).load(it).into(this)
    }
}
```

3. **✅ DO**: Use `requireAll = false` for optional multiple attributes
```kotlin
@BindingAdapter(value = ["imageUrl", "placeholder"], requireAll = false)
fun ImageView.loadImage(url: String?, placeholderId: Int?) {
    // Both attributes optional
}
```

4. **❌ DON'T**: Put heavy operations in BindingAdapters
```kotlin
// BAD
@BindingAdapter("processImage")
fun ImageView.processImage(bitmap: Bitmap?) {
    // Heavy image processing - blocks UI
}

// GOOD - Do in ViewModel/Repository
```

---

## Summary

### When to Use What?

**Use LiveData when:**
- ✅ Building traditional Android Views (XML layouts)
- ✅ Need automatic lifecycle management
- ✅ Working with older codebases

**Use StateFlow when:**
- ✅ Building Jetpack Compose UIs
- ✅ Need multiplatform support
- ✅ Want rich Kotlin Coroutines operators

**Use View Binding when:**
- ✅ Simple layouts without expressions
- ✅ Need type-safety without Data Binding overhead
- ✅ Migrating from findViewById

**Use Data Binding when:**
- ✅ Complex UIs with MVVM pattern
- ✅ Need two-way binding for forms
- ✅ Want declarative XML with expressions
- ✅ Require custom attributes (BindingAdapters)

**Use Binding Adapters when:**
- ✅ Loading images from URLs
- ✅ Custom view behavior needed
- ✅ Reusable custom attributes across app
- ✅ Complex formatting (dates, currency, etc.)

---

## Additional Resources

- **Official Docs**: https://developer.android.com/topic/libraries/data-binding
- **LiveData**: https://developer.android.com/topic/libraries/architecture/livedata
- **View Binding**: https://developer.android.com/topic/libraries/view-binding
- **Binding Adapters**: https://developer.android.com/topic/libraries/data-binding/binding-adapters

---

**Created for**: Android Development Reference
**Last Updated**: February 2026
**Author**: Comprehensive guide covering Kotlin and Java implementations
