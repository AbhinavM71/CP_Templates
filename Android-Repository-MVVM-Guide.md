# Android Repository Pattern in MVVM - Complete Guide

**Comprehensive Reference: Kotlin & Java Examples**

---

## Table of Contents
1. [Repository Pattern Overview](#overview)
2. [Why Repository Pattern?](#why)
3. [MVVM Architecture with Repository](#architecture)
4. [Single Data Source Repository](#single-source)
5. [Multiple Data Sources Repository](#multiple-sources)
6. [Repository with Room Database](#room)
7. [Repository with Retrofit (Network)](#retrofit)
8. [Repository with Room + Retrofit](#room-retrofit)
9. [Caching Strategies](#caching)
10. [Error Handling Patterns](#error-handling)
11. [Repository with Paging 3](#paging)
12. [Repository with WorkManager](#workmanager)
13. [Dependency Injection (Hilt)](#hilt)
14. [Testing Repository](#testing)
15. [Kotlin vs Java Comparison](#comparison)
16. [Best Practices](#best-practices)
17. [Complete Real-World Examples](#examples)

---

## Repository Pattern Overview

### What is Repository Pattern?

**Repository** is a design pattern that provides a **single source of truth** for data by abstracting data sources from the rest of the application.

**Definition:**
> Repository mediates between the domain and data mapping layers, acting like an in-memory collection of domain objects.

**Key Concept:**
```
ViewModel should NOT know where data comes from
↓
Repository handles: Room, Retrofit, SharedPreferences, Files
↓
Single interface to ViewModel
```

### Core Responsibilities

1. ✅ **Data Abstraction** - Hide Room/Retrofit/Cache implementation
2. ✅ **Single Source of Truth** - One place for all data operations
3. ✅ **Caching Logic** - Network first? Cache first? Offline first?
4. ✅ **Data Synchronization** - Local ↔ Remote sync
5. ✅ **Error Handling** - Centralized error management
6. ✅ **Testing** - Easy to mock for unit tests

### Repository vs DAO vs ViewModel

| Component | **Responsibility** | **Knows About** | **Returns** |
|-----------|-------------------|-----------------|-------------|
| **Entity** | Data model | Nothing | Plain data |
| **DAO** | Database operations ONLY | Room/SQLite | Data + LiveData/Flow |
| **Repository** | Data source management | DAO + API + Cache | Data + LiveData/Flow |
| **ViewModel** | Business logic + UI state | Repository ONLY | LiveData/StateFlow |
| **UI** | Display data | ViewModel ONLY | Nothing |

---

## Why Repository Pattern?

### Problems Without Repository

**❌ BAD: ViewModel directly accessing data sources**

```kotlin
// ViewModel WITHOUT Repository (BAD!)
class UserViewModel(
    private val userDao: UserDao,        // Too many dependencies!
    private val userApi: UserApi,
    private val sharedPrefs: SharedPreferences,
    private val fileManager: FileManager
) : ViewModel() {
    
    // ViewModel knows about Room
    fun getUsers() = userDao.getAllUsers()
    
    // ViewModel knows about Retrofit
    suspend fun refreshUsers() {
        val networkUsers = userApi.getUsers()
        userDao.insertUsers(networkUsers)  // ViewModel doing data logic!
    }
}
```

**Problems:**
1. ❌ **Too many dependencies** - ViewModel tightly coupled
2. ❌ **No abstraction** - Can't switch data sources
3. ❌ **Hard to test** - Must mock 4+ dependencies
4. ❌ **Business logic in ViewModel** - Repository logic mixed with UI logic
5. ❌ **No caching strategy** - Where to implement?

### Solution: Repository Pattern

**✅ GOOD: ViewModel with Repository**

```kotlin
// ViewModel WITH Repository (GOOD!)
class UserViewModel(
    private val repository: UserRepository  // Single dependency!
) : ViewModel() {
    
    // ViewModel doesn't know WHERE data comes from
    val users = repository.getAllUsers()
    
    fun refreshUsers() = viewModelScope.launch {
        repository.refreshUsers()  // Repository handles caching!
    }
}
```

**Benefits:**
1. ✅ **Single dependency** - Only Repository injected
2. ✅ **Abstraction** - ViewModel doesn't know Room/Retrofit exist
3. ✅ **Easy testing** - Mock only Repository
4. ✅ **Separation of concerns** - Data logic in Repository
5. ✅ **Centralized caching** - Repository decides strategy

---

## MVVM Architecture with Repository

### Complete Architecture Diagram

```
┌─────────────────────────────────────────────────────┐
│                    UI LAYER                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
│  │  Activity   │  │  Fragment   │  │  Composable │ │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘ │
│         │                 │                 │        │
│         └─────────────────┴─────────────────┘        │
│                           │                          │
└───────────────────────────┼──────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────┐
│                 VIEWMODEL LAYER                     │
│  ┌──────────────────────────────────────────────┐   │
│  │          ViewModel (Business Logic)          │   │
│  │  • LiveData/StateFlow                        │   │
│  │  • UI State Management                       │   │
│  │  • Error Handling                            │   │
│  └──────────────────┬───────────────────────────┘   │
└─────────────────────┼───────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────┐
│              REPOSITORY LAYER                       │
│  ┌──────────────────────────────────────────────┐   │
│  │              Repository                      │   │
│  │  • Single Source of Truth                   │   │
│  │  • Caching Strategy                         │   │
│  │  • Data Synchronization                     │   │
│  └──┬────────────┬────────────┬────────────┬───┘   │
└─────┼────────────┼────────────┼────────────┼───────┘
      │            │            │            │
      ▼            ▼            ▼            ▼
┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
│   Room   │ │ Retrofit │ │  Cache   │ │   File   │
│ Database │ │ Network  │ │ (Memory) │ │ Storage  │
└──────────┘ └──────────┘ └──────────┘ └──────────┘
```

### Data Flow

**Read Operation:**
```
UI → ViewModel.getUsers()
    ↓
Repository.getUsers()
    ↓
1. Check Cache (in-memory)
2. Check Room Database (local)
3. Fetch from Network (remote)
4. Save to Cache + Room
    ↓
Return Flow<List<User>> to ViewModel
    ↓
ViewModel exposes StateFlow to UI
    ↓
UI observes and displays
```

**Write Operation:**
```
UI → ViewModel.saveUser(user)
    ↓
Repository.saveUser(user)
    ↓
1. Save to Room (local)
2. Sync to Network (remote)
3. Update Cache
    ↓
Emit updated data to observers
```

---

## Single Data Source Repository

### Repository with Room Only

**When to use:**
- Offline-only apps
- No network requirements
- Local database CRUD

#### Kotlin Implementation

```kotlin
package com.example.repository

import androidx.lifecycle.LiveData
import kotlinx.coroutines.flow.Flow
import com.example.data.local.UserDao
import com.example.data.model.User

class UserRepository(
    private val userDao: UserDao
) {
    
    // ========== OBSERVABLE DATA ==========
    
    // Flow - Reactive, Compose-friendly
    val allUsers: Flow<List<User>> = userDao.getAllUsersFlow()
    
    // LiveData - Traditional Android
    val allUsersLiveData: LiveData<List<User>> = userDao.getAllUsersLiveData()
    
    // Active users only
    fun getActiveUsers(): Flow<List<User>> {
        return userDao.getActiveUsers()
    }
    
    // Search with query
    fun searchUsers(query: String): Flow<List<User>> {
        return userDao.searchUsers("%$query%")
    }
    
    // ========== CREATE ==========
    
    suspend fun insertUser(user: User): Long {
        return userDao.insertUser(user)
    }
    
    suspend fun insertUsers(users: List<User>): List<Long> {
        return userDao.insertUsers(users)
    }
    
    // ========== READ ==========
    
    suspend fun getUserById(userId: Long): User? {
        return userDao.getUserById(userId)
    }
    
    suspend fun getAllUsersList(): List<User> {
        return userDao.getAllUsers()
    }
    
    fun getUsersByAgeRange(minAge: Int, maxAge: Int): Flow<List<User>> {
        return userDao.getUsersByAgeRange(minAge, maxAge)
    }
    
    suspend fun getUserCount(): Int {
        return userDao.getUserCount()
    }
    
    // ========== UPDATE ==========
    
    suspend fun updateUser(user: User): Int {
        return userDao.updateUser(user)
    }
    
    suspend fun updateUserName(userId: Long, newName: String) {
        userDao.updateUserName(userId, newName)
    }
    
    // ========== DELETE ==========
    
    suspend fun deleteUser(user: User): Int {
        return userDao.deleteUser(user)
    }
    
    suspend fun deleteUserById(userId: Long) {
        userDao.deleteUserById(userId)
    }
    
    suspend fun deleteAllUsers() {
        userDao.deleteAllUsers()
    }
}
```

#### Java Implementation

```java
package com.example.repository;

import androidx.lifecycle.LiveData;
import com.example.data.local.UserDao;
import com.example.data.model.User;
import java.util.List;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class UserRepository {
    
    private final UserDao userDao;
    private final ExecutorService executorService;
    
    public UserRepository(UserDao userDao) {
        this.userDao = userDao;
        this.executorService = Executors.newSingleThreadExecutor();
    }
    
    // ========== OBSERVABLE DATA ==========
    
    public LiveData<List<User>> getAllUsers() {
        return userDao.getAllUsersLiveData();
    }
    
    public LiveData<List<User>> getActiveUsers() {
        return userDao.getActiveUsers();
    }
    
    public LiveData<List<User>> searchUsers(String query) {
        return userDao.searchUsers("%" + query + "%");
    }
    
    // ========== CREATE ==========
    
    public void insertUser(User user) {
        executorService.execute(() -> userDao.insertUser(user));
    }
    
    public void insertUsers(List<User> users) {
        executorService.execute(() -> userDao.insertUsers(users));
    }
    
    // ========== UPDATE ==========
    
    public void updateUser(User user) {
        executorService.execute(() -> userDao.updateUser(user));
    }
    
    public void updateUserName(long userId, String newName) {
        executorService.execute(() -> userDao.updateUserName(userId, newName));
    }
    
    // ========== DELETE ==========
    
    public void deleteUser(User user) {
        executorService.execute(() -> userDao.deleteUser(user));
    }
    
    public void deleteUserById(long userId) {
        executorService.execute(() -> userDao.deleteUserById(userId));
    }
    
    public void deleteAllUsers() {
        executorService.execute(() -> userDao.deleteAllUsers());
    }
}
```

---

## Multiple Data Sources Repository

### Architecture with Multiple Sources

```
Repository
    ├── Local Source (Room Database)
    ├── Remote Source (Retrofit API)
    ├── Cache Source (In-Memory Cache)
    └── Preference Source (DataStore/SharedPrefs)
```

### Data Source Interfaces

**Kotlin Data Sources:**

```kotlin
// 1. Network API
interface UserApi {
    @GET("users")
    suspend fun getUsers(): List<UserNetwork>
    
    @GET("users/{id}")
    suspend fun getUser(@Path("id") id: Long): UserNetwork
    
    @POST("users")
    suspend fun createUser(@Body user: UserNetwork): UserNetwork
    
    @PUT("users/{id}")
    suspend fun updateUser(@Path("id") id: Long, @Body user: UserNetwork): UserNetwork
    
    @DELETE("users/{id}")
    suspend fun deleteUser(@Path("id") id: Long): Response<Unit>
}

// 2. Local Database DAO
@Dao
interface UserDao {
    @Query("SELECT * FROM users")
    fun getAllUsersFlow(): Flow<List<User>>
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertUsers(users: List<User>)
    
    @Query("SELECT COUNT(*) FROM users")
    suspend fun getUserCount(): Int
    
    @Query("DELETE FROM users")
    suspend fun deleteAll()
}

// 3. In-Memory Cache
class UserCache {
    private val cache = mutableMapOf<Long, User>()
    private var lastUpdateTime: Long = 0
    
    fun put(user: User) {
        cache[user.id] = user
    }
    
    fun putAll(users: List<User>) {
        users.forEach { cache[it.id] = it }
        lastUpdateTime = System.currentTimeMillis()
    }
    
    fun get(id: Long): User? = cache[id]
    
    fun getAll(): List<User> = cache.values.toList()
    
    fun isCacheValid(maxAgeMillis: Long = 5 * 60 * 1000): Boolean {
        return System.currentTimeMillis() - lastUpdateTime < maxAgeMillis
    }
    
    fun clear() {
        cache.clear()
        lastUpdateTime = 0
    }
}

// 4. Network Models
data class UserNetwork(
    val id: Long,
    val name: String,
    val email: String,
    val age: Int
) {
    fun toUser() = User(
        id = id,
        name = name,
        email = email,
        age = age
    )
}

fun User.toNetwork() = UserNetwork(
    id = id,
    name = name,
    email = email,
    age = age
)
```

### Complete Multi-Source Repository

#### Kotlin Implementation

```kotlin
package com.example.repository

import kotlinx.coroutines.flow.*
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.withContext
import retrofit2.HttpException
import java.io.IOException

class UserRepository(
    private val userDao: UserDao,
    private val userApi: UserApi,
    private val userCache: UserCache
) {
    
    // ========== CACHING STRATEGIES ==========
    
    /**
     * STRATEGY 1: NETWORK FIRST
     * - Fetch fresh data from network
     * - Save to database and cache
     * - Fallback to cache on error
     */
    suspend fun getUsersNetworkFirst(forceRefresh: Boolean = false): Resource<List<User>> {
        return withContext(Dispatchers.IO) {
            try {
                if (forceRefresh || !userCache.isCacheValid()) {
                    // Fetch from network
                    val networkUsers = userApi.getUsers()
                    val users = networkUsers.map { it.toUser() }
                    
                    // Save to database
                    userDao.insertUsers(users)
                    
                    // Update cache
                    userCache.putAll(users)
                    
                    Resource.Success(users)
                } else {
                    // Return cache
                    Resource.Success(userCache.getAll())
                }
            } catch (e: HttpException) {
                // Network error - return cached data
                val cached = userDao.getAllUsers()
                if (cached.isNotEmpty()) {
                    Resource.Error("Network error", cached)
                } else {
                    Resource.Error("No cached data", null)
                }
            } catch (e: IOException) {
                // Network unavailable - return cached
                val cached = userDao.getAllUsers()
                if (cached.isNotEmpty()) {
                    Resource.Success(cached)
                } else {
                    Resource.Error("No internet connection", null)
                }
            }
        }
    }
    
    /**
     * STRATEGY 2: CACHE FIRST (Fast UI)
     * - Show cached data immediately
     * - Fetch network in background
     * - Update cache silently
     */
    fun getUsersCacheFirst(): Flow<Resource<List<User>>> = flow {
        // Emit loading
        emit(Resource.Loading())
        
        // Emit cached data immediately
        val cached = userDao.getAllUsers()
        if (cached.isNotEmpty()) {
            emit(Resource.Success(cached))
        }
        
        // Fetch network in background
        try {
            val networkUsers = userApi.getUsers()
            val users = networkUsers.map { it.toUser() }
            
            // Update database
            userDao.insertUsers(users)
            
            // Update cache
            userCache.putAll(users)
            
            // Emit fresh data
            emit(Resource.Success(users))
        } catch (e: Exception) {
            // Network fetch failed - keep showing cached data
            if (cached.isEmpty()) {
                emit(Resource.Error(e.message ?: "Unknown error", null))
            }
        }
    }.flowOn(Dispatchers.IO)
    
    /**
     * STRATEGY 3: OFFLINE FIRST
     * - Always return database
     * - Sync in background
     */
    fun getUsersOfflineFirst(): Flow<List<User>> {
        // Trigger background sync
        syncUsers()
        
        // Return database flow (updates automatically)
        return userDao.getAllUsersFlow()
    }
    
    /**
     * STRATEGY 4: HYBRID (Recommended)
     * - Check cache validity
     * - Return cache if valid
     * - Fetch network if expired
     */
    fun getUsersHybrid(): Flow<Resource<List<User>>> = flow {
        emit(Resource.Loading())
        
        // Check cache validity
        if (userCache.isCacheValid()) {
            emit(Resource.Success(userCache.getAll()))
        } else {
            // Cache expired - fetch network
            try {
                val networkUsers = userApi.getUsers()
                val users = networkUsers.map { it.toUser() }
                
                userDao.insertUsers(users)
                userCache.putAll(users)
                
                emit(Resource.Success(users))
            } catch (e: Exception) {
                // Fallback to database
                val dbUsers = userDao.getAllUsers()
                if (dbUsers.isNotEmpty()) {
                    emit(Resource.Success(dbUsers))
                } else {
                    emit(Resource.Error(e.message ?: "Error", null))
                }
            }
        }
    }.flowOn(Dispatchers.IO)
    
    // ========== INDIVIDUAL USER ==========
    
    suspend fun getUser(userId: Long, forceRefresh: Boolean = false): Resource<User> {
        return withContext(Dispatchers.IO) {
            try {
                if (forceRefresh) {
                    // Fetch from network
                    val networkUser = userApi.getUser(userId)
                    val user = networkUser.toUser()
                    
                    // Save to database
                    userDao.insertUsers(listOf(user))
                    
                    // Update cache
                    userCache.put(user)
                    
                    Resource.Success(user)
                } else {
                    // Try cache first
                    userCache.get(userId)?.let {
                        return@withContext Resource.Success(it)
                    }
                    
                    // Try database
                    userDao.getUserById(userId)?.let {
                        return@withContext Resource.Success(it)
                    }
                    
                    // Finally network
                    val networkUser = userApi.getUser(userId)
                    val user = networkUser.toUser()
                    
                    userDao.insertUsers(listOf(user))
                    userCache.put(user)
                    
                    Resource.Success(user)
                }
            } catch (e: Exception) {
                Resource.Error(e.message ?: "User not found", null)
            }
        }
    }
    
    // ========== WRITE OPERATIONS ==========
    
    suspend fun createUser(user: User): Resource<User> = withContext(Dispatchers.IO) {
        try {
            // Save locally first (optimistic update)
            val localId = userDao.insertUser(user)
            
            // Sync to network
            val networkUser = userApi.createUser(user.toNetwork())
            val syncedUser = networkUser.toUser()
            
            // Update with server ID
            userDao.updateUser(syncedUser)
            userCache.put(syncedUser)
            
            Resource.Success(syncedUser)
        } catch (e: Exception) {
            Resource.Error("Failed to create user", null)
        }
    }
    
    suspend fun updateUser(user: User): Resource<User> = withContext(Dispatchers.IO) {
        try {
            // Update locally first
            userDao.updateUser(user)
            userCache.put(user)
            
            // Sync to network
            val networkUser = userApi.updateUser(user.id, user.toNetwork())
            val syncedUser = networkUser.toUser()
            
            // Update with server response
            userDao.updateUser(syncedUser)
            userCache.put(syncedUser)
            
            Resource.Success(syncedUser)
        } catch (e: Exception) {
            // Local update succeeded, network sync failed
            Resource.Error("Sync failed", user)
        }
    }
    
    suspend fun deleteUser(userId: Long): Resource<Unit> = withContext(Dispatchers.IO) {
        try {
            // Delete locally first
            userDao.deleteUserById(userId)
            userCache.clear()
            
            // Sync to network
            userApi.deleteUser(userId)
            
            Resource.Success(Unit)
        } catch (e: Exception) {
            Resource.Error("Delete failed", null)
        }
    }
    
    // ========== BACKGROUND SYNC ==========
    
    private fun syncUsers() {
        // Trigger WorkManager or coroutine for background sync
    }
    
    suspend fun refreshUsers() {
        getUsersNetworkFirst(forceRefresh = true)
    }
    
    suspend fun clearCache() {
        userCache.clear()
        userDao.deleteAll()
    }
}

// ========== RESULT WRAPPER ==========

sealed class Resource<T> {
    class Loading<T> : Resource<T>()
    data class Success<T>(val data: T) : Resource<T>()
    data class Error<T>(val message: String, val data: T?) : Resource<T>()
    
    fun isLoading() = this is Loading
    fun isSuccess() = this is Success
    fun isError() = this is Error
    
    fun getOrNull(): T? = when (this) {
        is Success -> data
        is Error -> data
        else -> null
    }
}
```

#### Java Implementation

```java
package com.example.repository;

import androidx.lifecycle.LiveData;
import androidx.lifecycle.MutableLiveData;
import retrofit2.Call;
import retrofit2.Callback;
import retrofit2.Response;
import java.util.List;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class UserRepository {
    
    private final UserDao userDao;
    private final UserApi userApi;
    private final UserCache userCache;
    private final ExecutorService executorService;
    
    public UserRepository(UserDao userDao, UserApi userApi, UserCache userCache) {
        this.userDao = userDao;
        this.userApi = userApi;
        this.userCache = userCache;
        this.executorService = Executors.newSingleThreadExecutor();
    }
    
    // ========== NETWORK FIRST ==========
    
    public LiveData<Resource<List<User>>> getUsersNetworkFirst(boolean forceRefresh) {
        MutableLiveData<Resource<List<User>>> result = new MutableLiveData<>();
        result.setValue(Resource.loading(null));
        
        executorService.execute(() -> {
            try {
                if (forceRefresh || !userCache.isCacheValid()) {
                    // Network call (blocking)
                    Response<List<UserNetwork>> response = userApi.getUsersSync().execute();
                    
                    if (response.isSuccessful() && response.body() != null) {
                        List<User> users = convertToUsers(response.body());
                        
                        // Save to database
                        userDao.insertUsers(users);
                        
                        // Update cache
                        userCache.putAll(users);
                        
                        result.postValue(Resource.success(users));
                    } else {
                        // Network error - fallback to cache
                        List<User> cached = userDao.getAllUsers();
                        if (!cached.isEmpty()) {
                            result.postValue(Resource.error("Network error", cached));
                        } else {
                            result.postValue(Resource.error("No cached data", null));
                        }
                    }
                } else {
                    // Return cache
                    result.postValue(Resource.success(userCache.getAll()));
                }
            } catch (Exception e) {
                List<User> cached = userDao.getAllUsers();
                if (!cached.isEmpty()) {
                    result.postValue(Resource.success(cached));
                } else {
                    result.postValue(Resource.error(e.getMessage(), null));
                }
            }
        });
        
        return result;
    }
    
    // ========== CACHE FIRST ==========
    
    public LiveData<Resource<List<User>>> getUsersCacheFirst() {
        MutableLiveData<Resource<List<User>>> result = new MutableLiveData<>();
        
        // Show cached data immediately
        executorService.execute(() -> {
            List<User> cached = userDao.getAllUsers();
            if (!cached.isEmpty()) {
                result.postValue(Resource.success(cached));
            }
            
            // Fetch network in background
            userApi.getUsersAsync().enqueue(new Callback<List<UserNetwork>>() {
                @Override
                public void onResponse(Call<List<UserNetwork>> call, 
                                     Response<List<UserNetwork>> response) {
                    if (response.isSuccessful() && response.body() != null) {
                        executorService.execute(() -> {
                            List<User> users = convertToUsers(response.body());
                            userDao.insertUsers(users);
                            userCache.putAll(users);
                            result.postValue(Resource.success(users));
                        });
                    }
                }
                
                @Override
                public void onFailure(Call<List<UserNetwork>> call, Throwable t) {
                    if (cached.isEmpty()) {
                        result.postValue(Resource.error(t.getMessage(), null));
                    }
                }
            });
        });
        
        return result;
    }
    
    // ========== CREATE/UPDATE/DELETE ==========
    
    public LiveData<Resource<User>> createUser(User user) {
        MutableLiveData<Resource<User>> result = new MutableLiveData<>();
        
        executorService.execute(() -> {
            // Save locally first
            long localId = userDao.insertUser(user);
            
            // Sync to network
            userApi.createUserAsync(user.toNetwork()).enqueue(new Callback<UserNetwork>() {
                @Override
                public void onResponse(Call<UserNetwork> call, Response<UserNetwork> response) {
                    if (response.isSuccessful() && response.body() != null) {
                        User syncedUser = response.body().toUser();
                        executorService.execute(() -> {
                            userDao.updateUser(syncedUser);
                            userCache.put(syncedUser);
                            result.postValue(Resource.success(syncedUser));
                        });
                    } else {
                        result.postValue(Resource.error("Sync failed", user));
                    }
                }
                
                @Override
                public void onFailure(Call<UserNetwork> call, Throwable t) {
                    result.postValue(Resource.error(t.getMessage(), user));
                }
            });
        });
        
        return result;
    }
    
    // Helper method
    private List<User> convertToUsers(List<UserNetwork> networkUsers) {
        return networkUsers.stream()
            .map(UserNetwork::toUser)
            .collect(Collectors.toList());
    }
}

// ========== RESOURCE WRAPPER ==========

class Resource<T> {
    public enum Status { SUCCESS, ERROR, LOADING }
    
    private Status status;
    private T data;
    private String message;
    
    private Resource(Status status, T data, String message) {
        this.status = status;
        this.data = data;
        this.message = message;
    }
    
    public static <T> Resource<T> success(T data) {
        return new Resource<>(Status.SUCCESS, data, null);
    }
    
    public static <T> Resource<T> error(String message, T data) {
        return new Resource<>(Status.ERROR, data, message);
    }
    
    public static <T> Resource<T> loading(T data) {
        return new Resource<>(Status.LOADING, data, null);
    }
    
    public Status getStatus() { return status; }
    public T getData() { return data; }
    public String getMessage() { return message; }
}
```

---

## Repository with Room Database

**See previous Room Database guide for complete DAO implementation.**

**Simple Room Repository:**

```kotlin
class UserRepository(private val userDao: UserDao) {
    
    // Observable data
    val allUsers: Flow<List<User>> = userDao.getAllUsersFlow()
    
    // CRUD operations
    suspend fun insert(user: User) = userDao.insertUser(user)
    suspend fun update(user: User) = userDao.updateUser(user)
    suspend fun delete(user: User) = userDao.deleteUser(user)
    suspend fun getUserById(id: Long) = userDao.getUserById(id)
    
    // Custom queries
    fun searchUsers(query: String) = userDao.searchUsers(query)
    fun getActiveUsers() = userDao.getActiveUsers()
}
```

---

## Repository with Retrofit (Network Only)

**API Service:**

```kotlin
interface UserApi {
    @GET("users")
    suspend fun getUsers(): List<UserDto>
    
    @GET("users/{id}")
    suspend fun getUser(@Path("id") id: Long): UserDto
    
    @POST("users")
    suspend fun createUser(@Body user: UserDto): UserDto
    
    @PUT("users/{id}")
    suspend fun updateUser(@Path("id") id: Long, @Body user: UserDto): UserDto
    
    @DELETE("users/{id}")
    suspend fun deleteUser(@Path("id") id: Long): Response<Unit>
}
```

**Network-Only Repository:**

```kotlin
class UserNetworkRepository(private val api: UserApi) {
    
    suspend fun getUsers(): Result<List<User>> = try {
        val response = api.getUsers()
        Result.success(response.map { it.toUser() })
    } catch (e: Exception) {
        Result.failure(e)
    }
    
    suspend fun createUser(user: User): Result<User> = try {
        val response = api.createUser(user.toDto())
        Result.success(response.toUser())
    } catch (e: Exception) {
        Result.failure(e)
    }
}
```

---

## Repository with Room + Retrofit (Recommended)

**This is the production-ready pattern combining local + remote data.**

### Complete Implementation

```kotlin
class UserRepository(
    private val userDao: UserDao,
    private val userApi: UserApi
) {
    
    /**
     * Get users with Room + Retrofit sync
     * Returns Flow that emits database updates
     */
    fun getUsers(): Flow<Resource<List<User>>> = flow {
        emit(Resource.Loading())
        
        // Emit cached data first (fast UI)
        emitAll(
            userDao.getAllUsersFlow().map { Resource.Success(it) }
        )
        
        // Fetch network in background
        try {
            val networkUsers = userApi.getUsers()
            val users = networkUsers.map { it.toUser() }
            
            // Update database (will trigger new Flow emission)
            userDao.deleteAll()
            userDao.insertUsers(users)
        } catch (e: HttpException) {
            emit(Resource.Error("Network error: ${e.code()}", null))
        } catch (e: IOException) {
            emit(Resource.Error("No internet connection", null))
        }
    }.flowOn(Dispatchers.IO)
    
    /**
     * Refresh users (force network fetch)
     */
    suspend fun refreshUsers() = withContext(Dispatchers.IO) {
        try {
            val networkUsers = userApi.getUsers()
            userDao.deleteAll()
            userDao.insertUsers(networkUsers.map { it.toUser() })
        } catch (e: Exception) {
            throw e
        }
    }
    
    /**
     * Create user (local first, then sync)
     */
    suspend fun createUser(user: User): Resource<User> = withContext(Dispatchers.IO) {
        try {
            // Save locally (optimistic update)
            val localId = userDao.insertUser(user)
            val localUser = user.copy(id = localId)
            
            // Sync to server
            val networkUser = userApi.createUser(localUser.toDto())
            val serverUser = networkUser.toUser()
            
            // Update with server ID/data
            userDao.updateUser(serverUser)
            
            Resource.Success(serverUser)
        } catch (e: Exception) {
            Resource.Error("Failed to create user", null)
        }
    }
}
```

---

## Caching Strategies

### 1. Network First (Fresh Data Priority)

```kotlin
suspend fun networkFirst(): Resource<List<User>> {
    return try {
        // Fetch from network
        val data = api.getUsers()
        
        // Save to cache
        dao.insertUsers(data)
        
        Resource.Success(data)
    } catch (e: Exception) {
        // Fallback to cache
        val cached = dao.getAllUsers()
        if (cached.isNotEmpty()) {
            Resource.Success(cached)
        } else {
            Resource.Error("No data", null)
        }
    }
}
```

**Use when:** Real-time data critical (stock prices, news)

### 2. Cache First (Fast UI Priority)

```kotlin
fun cacheFirst(): Flow<Resource<List<User>>> = flow {
    // Show cache immediately
    val cached = dao.getAllUsers()
    if (cached.isNotEmpty()) {
        emit(Resource.Success(cached))
    }
    
    // Update from network
    try {
        val data = api.getUsers()
        dao.insertUsers(data)
        emit(Resource.Success(data))
    } catch (e: Exception) {
        if (cached.isEmpty()) {
            emit(Resource.Error("No data", null))
        }
    }
}
```

**Use when:** User experience priority (social media feed)

### 3. Offline First (Offline Support)

```kotlin
fun offlineFirst(): Flow<List<User>> {
    // Return database only
    return dao.getAllUsersFlow()
        .onStart {
            // Trigger background sync
            syncInBackground()
        }
}
```

**Use when:** Offline-first apps (note-taking, todos)

### 4. Stale-While-Revalidate

```kotlin
fun staleWhileRevalidate(): Flow<Resource<List<User>>> = flow {
    // 1. Emit cached data immediately
    val cached = dao.getAllUsers()
    emit(Resource.Success(cached))
    
    // 2. Fetch fresh data in background
    try {
        val fresh = api.getUsers()
        dao.insertUsers(fresh)
        // Database Flow will emit updated data automatically
    } catch (e: Exception) {
        // Keep showing cached data
    }
}
```

**Use when:** Best balance for most apps

---

## Error Handling Patterns

### Sealed Class (Recommended)

```kotlin
sealed class Resource<T> {
    data class Success<T>(val data: T) : Resource<T>()
    data class Error<T>(val message: String, val data: T? = null) : Resource<T>()
    data class Loading<T>(val data: T? = null) : Resource<T>()
    
    fun getOrNull(): T? = when (this) {
        is Success -> data
        is Error -> data
        is Loading -> data
    }
    
    fun getOrThrow(): T = when (this) {
        is Success -> data
        is Error -> throw Exception(message)
        is Loading -> throw IllegalStateException("Data is loading")
    }
}
```

**Usage in Repository:**
```kotlin
suspend fun getUser(id: Long): Resource<User> {
    return try {
        val user = api.getUser(id)
        Resource.Success(user)
    } catch (e: HttpException) {
        Resource.Error("Network error: ${e.code()}")
    } catch (e: IOException) {
        Resource.Error("No internet")
    } catch (e: Exception) {
        Resource.Error(e.message ?: "Unknown error")
    }
}
```

**Usage in ViewModel:**
```kotlin
fun loadUser(id: Long) = viewModelScope.launch {
    when (val result = repository.getUser(id)) {
        is Resource.Success -> _user.value = result.data
        is Resource.Error -> _error.value = result.message
        is Resource.Loading -> _loading.value = true
    }
}
```

### Result Type (Kotlin Standard)

```kotlin
suspend fun getUser(id: Long): Result<User> {
    return try {
        val user = api.getUser(id)
        Result.success(user)
    } catch (e: Exception) {
        Result.failure(e)
    }
}

// Usage
val result = repository.getUser(1)
result.onSuccess { user ->
    // Handle success
}.onFailure { exception ->
    // Handle error
}
```

---

## Repository with Paging 3

### Paging Repository

```kotlin
class UserRepository(private val userDao: UserDao) {
    
    fun getUsersPaged(): Flow<PagingData<User>> {
        return Pager(
            config = PagingConfig(
                pageSize = 20,
                enablePlaceholders = false
            ),
            pagingSourceFactory = { userDao.getUsersPaged() }
        ).flow
    }
}

// DAO
@Dao
interface UserDao {
    @Query("SELECT * FROM users ORDER BY name ASC")
    fun getUsersPaged(): PagingSource<Int, User>
}

// ViewModel
class UserViewModel(repository: UserRepository) : ViewModel() {
    val users = repository.getUsersPaged()
        .cachedIn(viewModelScope)
}
```

---

## Repository with WorkManager

### Background Sync Repository

```kotlin
class UserRepository(
    private val userDao: UserDao,
    private val userApi: UserApi,
    private val workManager: WorkManager
) {
    
    /**
     * Schedule background sync with WorkManager
     */
    fun scheduleSyncUsers() {
        val syncRequest = PeriodicWorkRequestBuilder<SyncUsersWorker>(
            15, TimeUnit.MINUTES
        )
            .setConstraints(
                Constraints.Builder()
                    .setRequiredNetworkType(NetworkType.CONNECTED)
                    .build()
            )
            .build()
        
        workManager.enqueueUniquePeriodicWork(
            "sync_users",
            ExistingPeriodicWorkPolicy.KEEP,
            syncRequest
        )
    }
    
    /**
     * Sync users (called by Worker)
     */
    suspend fun syncUsers() {
        try {
            val networkUsers = userApi.getUsers()
            userDao.deleteAll()
            userDao.insertUsers(networkUsers.map { it.toUser() })
        } catch (e: Exception) {
            // Log sync failure
        }
    }
}

// Worker
class SyncUsersWorker(
    context: Context,
    params: WorkerParameters
) : CoroutineWorker(context, params) {
    
    override suspend fun doWork(): Result {
        return try {
            val repository = (applicationContext as MyApp).repository
            repository.syncUsers()
            Result.success()
        } catch (e: Exception) {
            Result.retry()
        }
    }
}
```

---

## Dependency Injection (Hilt)

### Repository with Hilt

**Module:**
```kotlin
@Module
@InstallIn(SingletonComponent::class)
object RepositoryModule {
    
    @Provides
    @Singleton
    fun provideUserRepository(
        userDao: UserDao,
        userApi: UserApi,
        userCache: UserCache
    ): UserRepository {
        return UserRepository(userDao, userApi, userCache)
    }
}
```

**ViewModel:**
```kotlin
@HiltViewModel
class UserViewModel @Inject constructor(
    private val repository: UserRepository
) : ViewModel() {
    
    val users = repository.getUsersHybrid()
        .stateIn(viewModelScope, SharingStarted.Lazily, Resource.Loading())
    
    fun refreshUsers() = viewModelScope.launch {
        repository.refreshUsers()
    }
}
```

**Activity:**
```kotlin
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    
    private val viewModel: UserViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        lifecycleScope.launch {
            viewModel.users.collect { resource ->
                when (resource) {
                    is Resource.Success -> showUsers(resource.data)
                    is Resource.Error -> showError(resource.message)
                    is Resource.Loading -> showLoading()
                }
            }
        }
    }
}
```

---

## Testing Repository

### Unit Testing with Mocks

**Kotlin Test:**

```kotlin
@RunWith(MockitoJUnitRunner::class)
class UserRepositoryTest {
    
    @Mock
    private lateinit var userDao: UserDao
    
    @Mock
    private lateinit var userApi: UserApi
    
    @Mock
    private lateinit var userCache: UserCache
    
    private lateinit var repository: UserRepository
    
    @Before
    fun setup() {
        repository = UserRepository(userDao, userApi, userCache)
    }
    
    @Test
    fun `getUsersNetworkFirst success saves to database`() = runTest {
        // Arrange
        val mockUsers = listOf(mockUser())
        val networkUsers = mockUsers.map { it.toNetwork() }
        whenever(userApi.getUsers()).thenReturn(networkUsers)
        
        // Act
        val result = repository.getUsersNetworkFirst()
        
        // Assert
        assertTrue(result is Resource.Success)
        verify(userDao).insertUsers(mockUsers)
        verify(userCache).putAll(mockUsers)
    }
    
    @Test
    fun `getUsersNetworkFirst network error returns cache`() = runTest {
        // Arrange
        val cachedUsers = listOf(mockUser())
        whenever(userApi.getUsers()).thenThrow(HttpException(Response.error(500, mockBody())))
        whenever(userDao.getAllUsers()).thenReturn(cachedUsers)
        
        // Act
        val result = repository.getUsersNetworkFirst()
        
        // Assert
        assertTrue(result is Resource.Success)
        assertEquals(cachedUsers, result.getOrNull())
    }
    
    @Test
    fun `createUser saves locally then syncs`() = runTest {
        // Arrange
        val user = mockUser()
        whenever(userDao.insertUser(user)).thenReturn(1L)
        whenever(userApi.createUser(any())).thenReturn(user.toNetwork())
        
        // Act
        val result = repository.createUser(user)
        
        // Assert
        assertTrue(result is Resource.Success)
        verify(userDao).insertUser(user)
        verify(userApi).createUser(any())
    }
    
    private fun mockUser() = User(1L, "Test User", "test@test.com", 25)
}
```

### Fake Repository for Integration Tests

```kotlin
class FakeUserRepository : UserRepository {
    
    private val users = mutableListOf<User>()
    
    override suspend fun getUsers(): Resource<List<User>> {
        return Resource.Success(users.toList())
    }
    
    override suspend fun createUser(user: User): Resource<User> {
        users.add(user)
        return Resource.Success(user)
    }
    
    override suspend fun deleteUser(userId: Long): Resource<Unit> {
        users.removeIf { it.id == userId }
        return Resource.Success(Unit)
    }
}
```

---

## Kotlin vs Java Comparison

| Feature | **Kotlin** | **Java** |
|---------|-----------|----------|
| **Coroutines** | ✅ Native `suspend` functions | ❌ Callbacks/ExecutorService |
| **Flow** | ✅ `Flow<T>` for reactive streams | ❌ LiveData only |
| **Null Safety** | ✅ Built-in (`?`, `!!`) | ❌ Manual null checks |
| **Sealed Classes** | ✅ `sealed class Resource<T>` | ❌ Enum + if/else |
| **Extension Functions** | ✅ `User.toNetwork()` | ❌ Utility classes |
| **Data Classes** | ✅ Auto-generated equals/hashCode | ❌ Manual implementation |
| **Default Parameters** | ✅ `fun get(force: Boolean = false)` | ❌ Method overloading |
| **Code Length** | 40% less code | Verbose |

### Code Comparison

**Kotlin (60 lines):**
```kotlin
class UserRepository(
    private val dao: UserDao,
    private val api: UserApi
) {
    fun getUsers() = flow {
        emit(Resource.Loading())
        val cached = dao.getAllUsers()
        emit(Resource.Success(cached))
        
        try {
            val network = api.getUsers()
            dao.insertUsers(network.map { it.toUser() })
        } catch (e: Exception) {
            if (cached.isEmpty()) {
                emit(Resource.Error(e.message))
            }
        }
    }
}
```

**Java (150+ lines):**
```java
public class UserRepository {
    private UserDao dao;
    private UserApi api;
    
    public LiveData<Resource<List<User>>> getUsers() {
        MutableLiveData<Resource<List<User>>> result = new MutableLiveData<>();
        
        new AsyncTask<Void, Void, Void>() {
            @Override
            protected Void doInBackground(Void... voids) {
                List<User> cached = dao.getAllUsers();
                result.postValue(Resource.success(cached));
                
                api.getUsers().enqueue(new Callback<List<UserNetwork>>() {
                    @Override
                    public void onResponse(Call<List<UserNetwork>> call, Response<List<UserNetwork>> response) {
                        if (response.isSuccessful()) {
                            new AsyncTask<Void, Void, Void>() {
                                @Override
                                protected Void doInBackground(Void... voids) {
                                    dao.insertUsers(convertToUsers(response.body()));
                                    return null;
                                }
                            }.execute();
                        }
                    }
                    
                    @Override
                    public void onFailure(Call<List<UserNetwork>> call, Throwable t) {
                        if (cached.isEmpty()) {
                            result.postValue(Resource.error(t.getMessage(), null));
                        }
                    }
                });
                return null;
            }
        }.execute();
        
        return result;
    }
}
```

---

## Best Practices

### ✅ DO:

1. **Single Responsibility**
```kotlin
// GOOD - One repository per domain entity
class UserRepository(dao: UserDao, api: UserApi)
class ProductRepository(dao: ProductDao, api: ProductApi)

// BAD - God repository
class AppRepository(userDao, productDao, orderDao, apiService)
```

2. **Consistent Return Types**
```kotlin
// GOOD - Always return Flow or LiveData
fun getUsers(): Flow<List<User>>
fun getUser(id: Long): Flow<User>

// BAD - Mixed return types
fun getUsers(): List<User>  // Blocking!
fun getUser(id: Long): Flow<User>
```

3. **Error Handling**
```kotlin
// GOOD - Use Resource wrapper
suspend fun getUser(): Resource<User>

// BAD - Let exceptions propagate
suspend fun getUser(): User  // Throws!
```

4. **Dependency Injection**
```kotlin
// GOOD - Constructor injection
class UserRepository @Inject constructor(
    private val dao: UserDao,
    private val api: UserApi
)

// BAD - Manual instantiation
class UserRepository {
    val dao = AppDatabase.getInstance().userDao()
    val api = RetrofitClient.getInstance()
}
```

5. **Caching Strategy**
```kotlin
// GOOD - Clear strategy defined
fun getUsers(forceRefresh: Boolean = false): Flow<Resource<List<User>>>

// BAD - No caching control
fun getUsers(): List<User>
```

### ❌ DON'T:

1. **Business Logic in Repository**
```kotlin
// BAD - Validation in repository
fun createUser(user: User) {
    if (user.age < 18) throw Exception("Too young")  // Belongs in ViewModel!
    dao.insertUser(user)
}

// GOOD - Pure data operations
fun createUser(user: User) = dao.insertUser(user)
```

2. **Expose Data Sources**
```kotlin
// BAD - Exposes DAO
class UserRepository(val userDao: UserDao)

// GOOD - Encapsulates DAO
class UserRepository(private val userDao: UserDao)
```

3. **Main Thread Operations**
```kotlin
// BAD - Blocking main thread
fun getUsers(): List<User> = dao.getAllUsers()  // Blocks!

// GOOD - Async operations
suspend fun getUsers(): List<User> = dao.getAllUsers()
```

4. **Multiple Sources in ViewModel**
```kotlin
// BAD - ViewModel knows about DAO + API
class UserViewModel(val dao: UserDao, val api: UserApi)

// GOOD - ViewModel only knows Repository
class UserViewModel(val repository: UserRepository)
```

### Repository Naming

```kotlin
// ✅ GOOD
UserRepository
ProductRepository
OrderRepository

// ❌ BAD
UserDataSource  // Too generic
DatabaseHelper  // Implementation detail
ApiManager  // Wrong abstraction level
DataRepository  // Too vague
```

---

## Complete Real-World Examples

### Example 1: E-Commerce App Repository

```kotlin
// Product Entity
@Entity
data class Product(
    @PrimaryKey val id: Long,
    val name: String,
    val price: Double,
    val imageUrl: String,
    val category: String,
    val inStock: Boolean,
    val lastUpdated: Long = System.currentTimeMillis()
)

// Product DAO
@Dao
interface ProductDao {
    @Query("SELECT * FROM products WHERE category = :category")
    fun getProductsByCategory(category: String): Flow<List<Product>>
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertProducts(products: List<Product>)
    
    @Query("DELETE FROM products WHERE lastUpdated < :timestamp")
    suspend fun deleteOldProducts(timestamp: Long)
}

// Product API
interface ProductApi {
    @GET("products")
    suspend fun getProducts(@Query("category") category: String): List<ProductDto>
    
    @GET("products/{id}")
    suspend fun getProduct(@Path("id") id: Long): ProductDto
}

// Product Repository
class ProductRepository @Inject constructor(
    private val productDao: ProductDao,
    private val productApi: ProductApi
) {
    
    /**
     * Get products with smart caching
     * - Cache valid for 5 minutes
     * - Shows cached data immediately
     * - Refreshes in background
     */
    fun getProductsByCategory(category: String): Flow<Resource<List<Product>>> = flow {
        emit(Resource.Loading())
        
        // Emit cached products immediately
        emitAll(
            productDao.getProductsByCategory(category)
                .map { Resource.Success(it) }
        )
        
        // Check if refresh needed (5 minute cache)
        try {
            val networkProducts = productApi.getProducts(category)
            val products = networkProducts.map { it.toProduct() }
            
            // Update database
            productDao.insertProducts(products)
            
            // Clean old products (older than 1 hour)
            val oneHourAgo = System.currentTimeMillis() - (60 * 60 * 1000)
            productDao.deleteOldProducts(oneHourAgo)
            
        } catch (e: HttpException) {
            // Network error - keep showing cached data
            if (e.code() == 404) {
                emit(Resource.Error("Products not found", null))
            }
        } catch (e: IOException) {
            // No internet - cached data already emitted
        }
    }.flowOn(Dispatchers.IO)
    
    /**
     * Search products across all categories
     */
    fun searchProducts(query: String): Flow<List<Product>> {
        return productDao.searchProducts("%$query%")
    }
    
    /**
     * Add product to cart (local only)
     */
    suspend fun addToCart(productId: Long, quantity: Int) {
        // Local cart logic
    }
}

// Product ViewModel
@HiltViewModel
class ProductViewModel @Inject constructor(
    private val repository: ProductRepository
) : ViewModel() {
    
    private val _selectedCategory = MutableStateFlow("electronics")
    
    val products = _selectedCategory
        .flatMapLatest { category ->
            repository.getProductsByCategory(category)
        }
        .stateIn(viewModelScope, SharingStarted.Lazily, Resource.Loading())
    
    fun selectCategory(category: String) {
        _selectedCategory.value = category
    }
}
```

### Example 2: Social Media App Repository

```kotlin
// Post Repository with pagination
class PostRepository @Inject constructor(
    private val postDao: PostDao,
    private val postApi: PostApi
) {
    
    /**
     * Infinite scrolling with Paging 3
     */
    fun getPostsPaged(): Flow<PagingData<Post>> {
        return Pager(
            config = PagingConfig(
                pageSize = 20,
                prefetchDistance = 5,
                enablePlaceholders = false
            ),
            remoteMediator = PostRemoteMediator(postDao, postApi),
            pagingSourceFactory = { postDao.getPostsPaged() }
        ).flow
    }
    
    /**
     * Create post with optimistic update
     */
    suspend fun createPost(content: String, imageUri: Uri?): Resource<Post> {
        return withContext(Dispatchers.IO) {
            try {
                // 1. Save locally first (optimistic)
                val tempPost = Post(
                    id = 0,  // Temp ID
                    content = content,
                    imageUrl = imageUri?.toString(),
                    timestamp = System.currentTimeMillis(),
                    isSynced = false
                )
                val localId = postDao.insertPost(tempPost)
                
                // 2. Upload to server
                val serverPost = postApi.createPost(
                    content = content,
                    image = imageUri?.let { uploadImage(it) }
                )
                
                // 3. Update with server ID
                val syncedPost = serverPost.toPost().copy(isSynced = true)
                postDao.updatePost(syncedPost)
                
                Resource.Success(syncedPost)
            } catch (e: Exception) {
                // Local post saved, sync failed
                Resource.Error("Post saved locally, will sync later", null)
            }
        }
    }
    
    /**
     * Like post with instant feedback
     */
    suspend fun likePost(postId: Long) {
        // Update UI immediately
        postDao.incrementLikes(postId)
        
        // Sync in background
        try {
            postApi.likePost(postId)
        } catch (e: Exception) {
            // Rollback on failure
            postDao.decrementLikes(postId)
        }
    }
}
```

---

## Summary

### Key Takeaways

1. **Repository = Single Source of Truth**
   - ViewModel doesn't know about Room/Retrofit/SharedPreferences
   - All data operations go through Repository

2. **Caching Strategies**
   - Network First: Fresh data priority
   - Cache First: Fast UI priority
   - Offline First: Offline support priority
   - Hybrid: Balance of all

3. **Error Handling**
   - Use `Resource<T>` or `Result<T>` wrapper
   - Always handle network errors gracefully
   - Provide cached data when possible

4. **Testing**
   - Easy to mock Repository
   - Test caching logic independently
   - Fake repositories for integration tests

5. **Modern Stack (2026)**
```
Hilt + Room + Retrofit + Paging 3 + Flow + Coroutines
    ↓
Repository Pattern
    ↓
Single Source of Truth
```

### When to Use Repository

✅ **Use Repository when:**
- Multiple data sources (local + remote)
- Need caching strategy
- Offline support required
- Complex data operations
- Want testable architecture

❌ **Skip Repository when:**
- Single data source (just Room)
- Simple CRUD operations
- No caching needed
- Prototype/POC

---

**Perfect for your Samsung middleware projects!** 🚀

Sync daemon state:
- Local: Room database (device state)
- Remote: REST API (server sync)
- Cache: In-memory (fast access)
- Repository: Handles sync logic

---

**Created for**: Android Development Reference  
**Last Updated**: February 2026  
**Author**: Comprehensive Kotlin & Java Guide  
**Topics**: Repository Pattern, MVVM, Room, Retrofit, Caching, Error Handling, Testing