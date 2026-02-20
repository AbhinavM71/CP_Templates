# Android Room Database - Complete Guide

**Comprehensive Reference: Kotlin & Java Examples**

---

## Table of Contents
1. [Room Architecture Overview](#room-architecture)
2. [Setup & Dependencies](#setup)
3. [Entities](#entities)
4. [DAOs (Data Access Objects)](#daos)
5. [Database Class](#database)
6. [TypeConverters](#typeconverters)
7. [Database Migrations](#migrations)
8. [Repository Pattern](#repository)
9. [MVVM Integration](#mvvm-integration)
10. [Complete CRUD Examples](#complete-examples)
11. [Kotlin vs Java Comparison](#kotlin-java-comparison)
12. [Best Practices](#best-practices)
13. [Testing Room Database](#testing)

---

## Room Architecture Overview

### What is Room Database?

**Room** is an abstraction layer over SQLite that provides:
- ✅ **Compile-time verification** of SQL queries
- ✅ **Type-safe database access**
- ✅ **LiveData/Flow support** for reactive updates
- ✅ **Coroutines support** for async operations
- ✅ **Migration support** for schema changes
- ✅ **TypeConverters** for complex data types
- ✅ **Relationships** (One-to-One, One-to-Many, Many-to-Many)

### Core Components

```
┌─────────────────────────────────────┐
│          Android App                │
├─────────────────────────────────────┤
│      ViewModel (Business Logic)     │
├─────────────────────────────────────┤
│    Repository (Data Abstraction)    │
├─────────────────────────────────────┤
│       Room Database (@Database)     │
├─────────────────────────────────────┤
│    DAO (@Dao) - CRUD Operations     │
├─────────────────────────────────────┤
│    Entity (@Entity) - Table Model   │
├─────────────────────────────────────┤
│         SQLite Database             │
└─────────────────────────────────────┘
```

**Three Main Components:**

1. **Entity** - Represents a database table (annotated with `@Entity`)
2. **DAO** - Data Access Object - defines database operations (annotated with `@Dao`)
3. **Database** - Database holder and main access point (annotated with `@Database`)

---

## Setup & Dependencies

### build.gradle (Module: app)

**Kotlin DSL:**
```kotlin
dependencies {
    def room_version = "2.6.1"
    
    // Room core
    implementation "androidx.room:room-runtime:$room_version"
    kapt "androidx.room:room-compiler:$room_version"  // Kotlin annotation processor
    
    // Room Kotlin Extensions + Coroutines
    implementation "androidx.room:room-ktx:$room_version"
    
    // Room testing
    testImplementation "androidx.room:room-testing:$room_version"
    
    // Coroutines
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.8.1'
    
    // Lifecycle components
    implementation 'androidx.lifecycle:lifecycle-viewmodel-ktx:2.8.4'
    implementation 'androidx.lifecycle:lifecycle-livedata-ktx:2.8.4'
}
```

**Groovy (Java):**
```groovy
dependencies {
    def room_version = "2.6.1"
    
    // Room core
    implementation "androidx.room:room-runtime:$room_version"
    annotationProcessor "androidx.room:room-compiler:$room_version"  // Java annotation processor
    
    // Room testing
    testImplementation "androidx.room:room-testing:$room_version"
}
```

### Enable Kapt (Kotlin only)

**build.gradle (Module: app):**
```kotlin
plugins {
    id 'kotlin-kapt'  // Add this for Kotlin annotation processing
}
```

---

## Entities

### Basic Entity

**Entity** represents a table in the database. Each field is a column.

#### Kotlin Entity

```kotlin
package com.example.roomdemo

import androidx.room.ColumnInfo
import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity(tableName = "users")
data class User(
    @PrimaryKey(autoGenerate = true)
    val id: Long = 0,
    
    @ColumnInfo(name = "full_name")
    val name: String,
    
    val email: String,
    
    @ColumnInfo(name = "age")
    val age: Int,
    
    @ColumnInfo(defaultValue = "true")
    val isActive: Boolean = true,
    
    val createdAt: Long = System.currentTimeMillis()
)
```

#### Java Entity

```java
package com.example.roomdemo;

import androidx.room.ColumnInfo;
import androidx.room.Entity;
import androidx.room.PrimaryKey;

@Entity(tableName = "users")
public class User {
    @PrimaryKey(autoGenerate = true)
    public long id = 0;
    
    @ColumnInfo(name = "full_name")
    public String name;
    
    public String email;
    
    @ColumnInfo(name = "age")
    public int age;
    
    @ColumnInfo(defaultValue = "true")
    public boolean isActive = true;
    
    public long createdAt = System.currentTimeMillis();
    
    // Default constructor required
    public User() {}
    
    // Constructor with parameters
    public User(String name, String email, int age) {
        this.name = name;
        this.email = email;
        this.age = age;
    }
    
    // Getters and setters (required for Room in Java)
    public long getId() { return id; }
    public void setId(long id) { this.id = id; }
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    
    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }
    
    public boolean isActive() { return isActive; }
    public void setActive(boolean active) { isActive = active; }
    
    public long getCreatedAt() { return createdAt; }
    public void setCreatedAt(long createdAt) { this.createdAt = createdAt; }
}
```

### Entity with Relationships

#### One-to-Many Relationship

**Parent Entity (User):**
```kotlin
@Entity(tableName = "users")
data class User(
    @PrimaryKey(autoGenerate = true)
    val id: Long = 0,
    val name: String,
    val email: String
)
```

**Child Entity (Task with Foreign Key):**
```kotlin
@Entity(
    tableName = "tasks",
    foreignKeys = [
        ForeignKey(
            entity = User::class,
            parentColumns = ["id"],
            childColumns = ["userId"],
            onDelete = ForeignKey.CASCADE  // Delete tasks when user deleted
        )
    ],
    indices = [Index(value = ["userId"])]  // Index for performance
)
data class Task(
    @PrimaryKey(autoGenerate = true)
    val id: Long = 0,
    
    val title: String,
    
    val description: String,
    
    val isCompleted: Boolean = false,
    
    val userId: Long  // Foreign key
)
```

**Java Version:**
```java
@Entity(
    tableName = "tasks",
    foreignKeys = @ForeignKey(
        entity = User.class,
        parentColumns = "id",
        childColumns = "userId",
        onDelete = ForeignKey.CASCADE
    ),
    indices = {@Index(value = "userId")}
)
public class Task {
    @PrimaryKey(autoGenerate = true)
    public long id;
    
    public String title;
    public String description;
    public boolean isCompleted;
    public long userId;  // Foreign key
    
    public Task(String title, String description, long userId) {
        this.title = title;
        this.description = description;
        this.userId = userId;
    }
}
```

### Embedded Objects

**Kotlin:**
```kotlin
// Address is NOT a separate table
data class Address(
    val street: String,
    val city: String,
    val zipCode: String
)

@Entity(tableName = "users")
data class User(
    @PrimaryKey(autoGenerate = true)
    val id: Long = 0,
    
    val name: String,
    
    @Embedded  // Embeds all fields of Address into users table
    val address: Address
)

// Creates table: users (id, name, street, city, zipCode)
```

### Ignoring Fields

**Kotlin:**
```kotlin
@Entity
data class User(
    @PrimaryKey val id: Long,
    val name: String,
    
    @Ignore  // This field won't be stored in database
    val tempStatus: String? = null
)
```

---

## DAOs (Data Access Objects)

### DAO Interface

**DAO** provides methods to access database. Room validates queries at compile-time.

#### Kotlin DAO (with Coroutines)

```kotlin
package com.example.roomdemo

import androidx.lifecycle.LiveData
import androidx.room.*
import kotlinx.coroutines.flow.Flow

@Dao
interface UserDao {
    
    // ========== INSERT ==========
    
    // Insert single user (returns row ID)
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertUser(user: User): Long
    
    // Insert multiple users
    @Insert
    suspend fun insertUsers(users: List<User>): List<Long>
    
    // ========== UPDATE ==========
    
    // Update user
    @Update
    suspend fun updateUser(user: User): Int  // Returns number of rows updated
    
    // Update specific fields
    @Query("UPDATE users SET name = :newName WHERE id = :userId")
    suspend fun updateUserName(userId: Long, newName: String)
    
    // ========== DELETE ==========
    
    // Delete user object
    @Delete
    suspend fun deleteUser(user: User): Int  // Returns number of rows deleted
    
    // Delete by ID
    @Query("DELETE FROM users WHERE id = :userId")
    suspend fun deleteUserById(userId: Long)
    
    // Delete all
    @Query("DELETE FROM users")
    suspend fun deleteAllUsers()
    
    // ========== QUERY (READ) ==========
    
    // Get single user
    @Query("SELECT * FROM users WHERE id = :userId")
    suspend fun getUserById(userId: Long): User?
    
    // Get all users - LiveData (observes changes automatically)
    @Query("SELECT * FROM users ORDER BY name ASC")
    fun getAllUsersLiveData(): LiveData<List<User>>
    
    // Get all users - Flow (Compose compatible)
    @Query("SELECT * FROM users ORDER BY name ASC")
    fun getAllUsersFlow(): Flow<List<User>>
    
    // Get all users - List (one-time query)
    @Query("SELECT * FROM users ORDER BY name ASC")
    suspend fun getAllUsers(): List<User>
    
    // Search users
    @Query("SELECT * FROM users WHERE name LIKE '%' || :query || '%' OR email LIKE '%' || :query || '%'")
    fun searchUsers(query: String): Flow<List<User>>
    
    // Get active users only
    @Query("SELECT * FROM users WHERE isActive = 1")
    fun getActiveUsers(): Flow<List<User>>
    
    // Count users
    @Query("SELECT COUNT(*) FROM users")
    suspend fun getUserCount(): Int
    
    // ========== COMPLEX QUERIES ==========
    
    // Get users with task count (JOIN)
    @Query("""
        SELECT users.*, COUNT(tasks.id) as taskCount 
        FROM users 
        LEFT JOIN tasks ON users.id = tasks.userId 
        GROUP BY users.id 
        ORDER BY taskCount DESC
    """)
    fun getUsersWithTaskCount(): Flow<List<UserWithTaskCount>>
    
    // Get user with all tasks (one-to-many)
    @Transaction
    @Query("SELECT * FROM users WHERE id = :userId")
    suspend fun getUserWithTasks(userId: Long): UserWithTasks?
    
    // Get users by age range
    @Query("SELECT * FROM users WHERE age BETWEEN :minAge AND :maxAge")
    fun getUsersByAgeRange(minAge: Int, maxAge: Int): Flow<List<User>>
}
```

**Data classes for complex queries:**
```kotlin
// For JOIN query
data class UserWithTaskCount(
    @Embedded val user: User,
    val taskCount: Int
)

// For one-to-many relationship
data class UserWithTasks(
    @Embedded val user: User,
    @Relation(
        parentColumn = "id",
        entityColumn = "userId"
    )
    val tasks: List<Task>
)
```

#### Java DAO

```java
package com.example.roomdemo;

import androidx.lifecycle.LiveData;
import androidx.room.*;
import java.util.List;

@Dao
public interface UserDao {
    
    // ========== INSERT ==========
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    long insertUser(User user);
    
    @Insert
    List<Long> insertUsers(List<User> users);
    
    // ========== UPDATE ==========
    
    @Update
    int updateUser(User user);
    
    @Query("UPDATE users SET name = :newName WHERE id = :userId")
    void updateUserName(long userId, String newName);
    
    // ========== DELETE ==========
    
    @Delete
    int deleteUser(User user);
    
    @Query("DELETE FROM users WHERE id = :userId")
    void deleteUserById(long userId);
    
    @Query("DELETE FROM users")
    void deleteAllUsers();
    
    // ========== QUERY (READ) ==========
    
    @Query("SELECT * FROM users WHERE id = :userId")
    User getUserById(long userId);
    
    // LiveData - observes changes
    @Query("SELECT * FROM users ORDER BY name ASC")
    LiveData<List<User>> getAllUsersLiveData();
    
    // Simple list
    @Query("SELECT * FROM users ORDER BY name ASC")
    List<User> getAllUsers();
    
    @Query("SELECT * FROM users WHERE name LIKE '%' || :query || '%'")
    LiveData<List<User>> searchUsers(String query);
    
    @Query("SELECT * FROM users WHERE isActive = 1")
    LiveData<List<User>> getActiveUsers();
    
    @Query("SELECT COUNT(*) FROM users")
    int getUserCount();
    
    // Complex query with JOIN
    @Query("SELECT users.*, COUNT(tasks.id) as taskCount " +
           "FROM users LEFT JOIN tasks ON users.id = tasks.userId " +
           "GROUP BY users.id ORDER BY taskCount DESC")
    LiveData<List<UserWithTaskCount>> getUsersWithTaskCount();
}
```

---

## Database Class

### Kotlin Database

```kotlin
package com.example.roomdemo

import android.content.Context
import androidx.room.Database
import androidx.room.Room
import androidx.room.RoomDatabase
import androidx.room.TypeConverters
import androidx.room.migration.Migration
import androidx.sqlite.db.SupportSQLiteDatabase

@Database(
    entities = [User::class, Task::class],  // All entities
    version = 2,  // Increment for migrations
    exportSchema = true  // Export schema for migrations
)
@TypeConverters(Converters::class)  // Custom type converters
abstract class AppDatabase : RoomDatabase() {
    
    // DAO getters
    abstract fun userDao(): UserDao
    abstract fun taskDao(): TaskDao
    
    // Singleton pattern
    companion object {
        @Volatile
        private var INSTANCE: AppDatabase? = null
        
        fun getDatabase(context: Context): AppDatabase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext,
                    AppDatabase::class.java,
                    "app_database"  // Database file name
                )
                    .addMigrations(MIGRATION_1_2)  // Add migrations
                    // .fallbackToDestructiveMigration()  // DEV ONLY! Deletes all data
                    .build()
                INSTANCE = instance
                instance
            }
        }
        
        // Migration from version 1 to 2
        val MIGRATION_1_2 = object : Migration(1, 2) {
            override fun migrate(database: SupportSQLiteDatabase) {
                // Add new column to existing table
                database.execSQL("""
                    ALTER TABLE users ADD COLUMN phone TEXT
                """.trimIndent())
            }
        }
    }
}
```

### Java Database

```java
package com.example.roomdemo;

import android.content.Context;
import androidx.room.Database;
import androidx.room.Room;
import androidx.room.RoomDatabase;
import androidx.room.TypeConverters;
import androidx.room.migration.Migration;
import androidx.sqlite.db.SupportSQLiteDatabase;

@Database(
    entities = {User.class, Task.class},
    version = 2,
    exportSchema = true
)
@TypeConverters(Converters.class)
public abstract class AppDatabase extends RoomDatabase {
    
    public abstract UserDao userDao();
    public abstract TaskDao taskDao();
    
    private static volatile AppDatabase INSTANCE;
    
    public static AppDatabase getDatabase(final Context context) {
        if (INSTANCE == null) {
            synchronized (AppDatabase.class) {
                if (INSTANCE == null) {
                    INSTANCE = Room.databaseBuilder(
                        context.getApplicationContext(),
                        AppDatabase.class,
                        "app_database"
                    )
                        .addMigrations(MIGRATION_1_2)
                        // .fallbackToDestructiveMigration()  // DEV ONLY!
                        .build();
                }
            }
        }
        return INSTANCE;
    }
    
    static final Migration MIGRATION_1_2 = new Migration(1, 2) {
        @Override
        public void migrate(SupportSQLiteDatabase database) {
            database.execSQL("ALTER TABLE users ADD COLUMN phone TEXT");
        }
    };
}
```

---

## TypeConverters

### Why TypeConverters?

SQLite only supports basic types (Integer, String, Real, Blob). **TypeConverters** serialize/deserialize complex types.

**Supported by default:**
- `int`, `Integer`, `long`, `Long`, `float`, `Float`, `double`, `Double`
- `String`, `byte[]`, `boolean`, `Boolean`

**Requires TypeConverter:**
- `Date`, `UUID`, `List<String>`, Custom objects, Enums

### Kotlin TypeConverters

```kotlin
package com.example.roomdemo

import androidx.room.TypeConverter
import com.google.gson.Gson
import com.google.gson.reflect.TypeToken
import java.text.SimpleDateFormat
import java.util.*

class Converters {
    
    private val gson = Gson()
    
    // ========== Date Converter ==========
    
    @TypeConverter
    fun fromTimestamp(value: Long?): Date? {
        return value?.let { Date(it) }
    }
    
    @TypeConverter
    fun dateToTimestamp(date: Date?): Long? {
        return date?.time
    }
    
    // ========== UUID Converter ==========
    
    @TypeConverter
    fun fromUUID(uuid: UUID?): String? {
        return uuid?.toString()
    }
    
    @TypeConverter
    fun toUUID(uuidString: String?): UUID? {
        return uuidString?.let { UUID.fromString(it) }
    }
    
    // ========== List<String> Converter ==========
    
    @TypeConverter
    fun fromStringList(list: List<String>?): String? {
        return list?.joinToString(",")
    }
    
    @TypeConverter
    fun toStringList(data: String?): List<String>? {
        return data?.split(",")?.map { it.trim() }
    }
    
    // ========== Custom Object (using Gson) ==========
    
    @TypeConverter
    fun fromAddress(address: Address?): String? {
        return gson.toJson(address)
    }
    
    @TypeConverter
    fun toAddress(addressString: String?): Address? {
        return try {
            gson.fromJson(addressString, Address::class.java)
        } catch (e: Exception) {
            null
        }
    }
    
    // ========== List<CustomObject> Converter ==========
    
    @TypeConverter
    fun fromTaskList(tasks: List<Task>?): String? {
        return gson.toJson(tasks)
    }
    
    @TypeConverter
    fun toTaskList(tasksString: String?): List<Task>? {
        val listType = object : TypeToken<List<Task>>() {}.type
        return gson.fromJson(tasksString, listType)
    }
    
    // ========== Enum Converter ==========
    
    @TypeConverter
    fun fromUserStatus(status: UserStatus?): String? {
        return status?.name
    }
    
    @TypeConverter
    fun toUserStatus(statusString: String?): UserStatus? {
        return statusString?.let { UserStatus.valueOf(it) }
    }
    
    // ========== Map<String, String> Converter ==========
    
    @TypeConverter
    fun fromStringMap(map: Map<String, String>?): String? {
        return gson.toJson(map)
    }
    
    @TypeConverter
    fun toStringMap(mapString: String?): Map<String, String>? {
        val mapType = object : TypeToken<Map<String, String>>() {}.type
        return gson.fromJson(mapString, mapType)
    }
}

// Example enum
enum class UserStatus {
    ACTIVE, INACTIVE, PENDING, BANNED
}

// Example custom object
data class Address(
    val street: String,
    val city: String,
    val zipCode: String
)
```

**Using TypeConverters in Entity:**
```kotlin
@Entity
data class User(
    @PrimaryKey val id: UUID,  // Requires UUID converter
    val name: String,
    val tags: List<String>,  // Requires List<String> converter
    val address: Address?,  // Requires custom object converter
    val status: UserStatus,  // Requires enum converter
    val createdAt: Date  // Requires Date converter
)
```

### Java TypeConverters

```java
package com.example.roomdemo;

import androidx.room.TypeConverter;
import com.google.gson.Gson;
import com.google.gson.reflect.TypeToken;
import java.lang.reflect.Type;
import java.text.SimpleDateFormat;
import java.util.*;

public class Converters {
    
    private static final Gson gson = new Gson();
    
    // ========== Date Converter ==========
    
    @TypeConverter
    public static Date fromTimestamp(Long value) {
        return value == null ? null : new Date(value);
    }
    
    @TypeConverter
    public static Long dateToTimestamp(Date date) {
        return date == null ? null : date.getTime();
    }
    
    // ========== UUID Converter ==========
    
    @TypeConverter
    public static String fromUUID(UUID uuid) {
        return uuid == null ? null : uuid.toString();
    }
    
    @TypeConverter
    public static UUID toUUID(String uuidString) {
        return uuidString == null ? null : UUID.fromString(uuidString);
    }
    
    // ========== List<String> Converter ==========
    
    @TypeConverter
    public static String fromStringList(List<String> list) {
        return list == null ? null : String.join(",", list);
    }
    
    @TypeConverter
    public static List<String> toStringList(String data) {
        return data == null ? null : Arrays.asList(data.split(","));
    }
    
    // ========== Custom Object Converter ==========
    
    @TypeConverter
    public static String fromAddress(Address address) {
        return gson.toJson(address);
    }
    
    @TypeConverter
    public static Address toAddress(String addressString) {
        try {
            return gson.fromJson(addressString, Address.class);
        } catch (Exception e) {
            return null;
        }
    }
    
    // ========== List<CustomObject> Converter ==========
    
    @TypeConverter
    public static String fromTaskList(List<Task> tasks) {
        return gson.toJson(tasks);
    }
    
    @TypeConverter
    public static List<Task> toTaskList(String tasksString) {
        Type listType = new TypeToken<List<Task>>(){}.getType();
        return gson.fromJson(tasksString, listType);
    }
    
    // ========== Enum Converter ==========
    
    @TypeConverter
    public static String fromUserStatus(UserStatus status) {
        return status == null ? null : status.name();
    }
    
    @TypeConverter
    public static UserStatus toUserStatus(String statusString) {
        return statusString == null ? null : UserStatus.valueOf(statusString);
    }
}
```

**Add Gson dependency:**
```gradle
implementation 'com.google.code.gson:gson:2.10.1'
```

---

## Database Migrations

### Migration Strategies

1. **Destructive Migration** (Dev only) - `fallbackToDestructiveMigration()`
2. **Manual Migration** - Custom SQL migrations
3. **Auto Migration** (Limited) - Room generates (simple changes only)

### Kotlin Migrations

```kotlin
// Migration 1→2: Add column
val MIGRATION_1_2 = object : Migration(1, 2) {
    override fun migrate(database: SupportSQLiteDatabase) {
        database.execSQL("""
            ALTER TABLE users ADD COLUMN phone TEXT
        """.trimIndent())
    }
}

// Migration 2→3: Add new table
val MIGRATION_2_3 = object : Migration(2, 3) {
    override fun migrate(database: SupportSQLiteDatabase) {
        database.execSQL("""
            CREATE TABLE tasks (
                id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
                title TEXT NOT NULL,
                description TEXT NOT NULL,
                isCompleted INTEGER NOT NULL DEFAULT 0,
                userId INTEGER NOT NULL,
                FOREIGN KEY(userId) REFERENCES users(id) ON DELETE CASCADE
            )
        """.trimIndent())
        
        // Create index for foreign key
        database.execSQL("CREATE INDEX index_tasks_userId ON tasks(userId)")
    }
}

// Migration 3→4: Rename column (complex - requires temp table)
val MIGRATION_3_4 = object : Migration(3, 4) {
    override fun migrate(database: SupportSQLiteDatabase) {
        // Step 1: Create new table with correct schema
        database.execSQL("""
            CREATE TABLE users_new (
                id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
                full_name TEXT NOT NULL,
                email TEXT NOT NULL,
                age INTEGER NOT NULL,
                phone TEXT,
                isActive INTEGER NOT NULL DEFAULT 1,
                createdAt INTEGER NOT NULL
            )
        """.trimIndent())
        
        // Step 2: Copy data from old table
        database.execSQL("""
            INSERT INTO users_new (id, full_name, email, age, phone, isActive, createdAt)
            SELECT id, name, email, age, phone, isActive, createdAt FROM users
        """.trimIndent())
        
        // Step 3: Drop old table
        database.execSQL("DROP TABLE users")
        
        // Step 4: Rename new table
        database.execSQL("ALTER TABLE users_new RENAME TO users")
    }
}

// Add migrations to database
Room.databaseBuilder(context, AppDatabase::class.java, "app_database")
    .addMigrations(MIGRATION_1_2, MIGRATION_2_3, MIGRATION_3_4)
    .build()
```

### Java Migrations

```java
static final Migration MIGRATION_1_2 = new Migration(1, 2) {
    @Override
    public void migrate(SupportSQLiteDatabase database) {
        database.execSQL("ALTER TABLE users ADD COLUMN phone TEXT");
    }
};

static final Migration MIGRATION_2_3 = new Migration(2, 3) {
    @Override
    public void migrate(SupportSQLiteDatabase database) {
        database.execSQL("CREATE TABLE tasks (" +
                "id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL, " +
                "title TEXT NOT NULL, " +
                "description TEXT NOT NULL, " +
                "isCompleted INTEGER NOT NULL DEFAULT 0, " +
                "userId INTEGER NOT NULL, " +
                "FOREIGN KEY(userId) REFERENCES users(id) ON DELETE CASCADE)");
        
        database.execSQL("CREATE INDEX index_tasks_userId ON tasks(userId)");
    }
};

static final Migration MIGRATION_3_4 = new Migration(3, 4) {
    @Override
    public void migrate(SupportSQLiteDatabase database) {
        // Create new table
        database.execSQL("CREATE TABLE users_new (" +
                "id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL, " +
                "full_name TEXT NOT NULL, " +
                "email TEXT NOT NULL, " +
                "age INTEGER NOT NULL, " +
                "phone TEXT, " +
                "isActive INTEGER NOT NULL DEFAULT 1, " +
                "createdAt INTEGER NOT NULL)");
        
        // Copy data
        database.execSQL("INSERT INTO users_new (id, full_name, email, age, phone, isActive, createdAt) " +
                "SELECT id, name, email, age, phone, isActive, createdAt FROM users");
        
        // Drop old table
        database.execSQL("DROP TABLE users");
        
        // Rename new table
        database.execSQL("ALTER TABLE users_new RENAME TO users");
    }
};
```

### Testing Migrations

```kotlin
@RunWith(AndroidJUnit4::class)
class MigrationTest {
    private val TEST_DB = "migration_test"
    
    @get:Rule
    val helper: MigrationTestHelper = MigrationTestHelper(
        InstrumentationRegistry.getInstrumentation(),
        AppDatabase::class.java.canonicalName,
        FrameworkSQLiteOpenHelperFactory()
    )
    
    @Test
    fun migrate1To2() {
        // Create database version 1
        helper.createDatabase(TEST_DB, 1).apply {
            execSQL("INSERT INTO users (id, name, email, age) VALUES (1, 'Test', 'test@test.com', 25)")
            close()
        }
        
        // Run migration
        helper.runMigrationsAndValidate(TEST_DB, 2, true, MIGRATION_1_2)
        
        // Validate migration worked
        val db = helper.runMigrationsAndValidate(TEST_DB, 2, true, MIGRATION_1_2)
        db.query("SELECT * FROM users").use { cursor ->
            assertTrue(cursor.moveToFirst())
            // Verify data preserved
        }
    }
}
```

---

## Repository Pattern

### Kotlin Repository

```kotlin
package com.example.roomdemo

import androidx.lifecycle.LiveData
import kotlinx.coroutines.flow.Flow

class UserRepository(private val userDao: UserDao) {
    
    // Expose LiveData/Flow from DAO
    val allUsers: Flow<List<User>> = userDao.getAllUsersFlow()
    
    // Insert
    suspend fun insert(user: User): Long {
        return userDao.insertUser(user)
    }
    
    suspend fun insertMultiple(users: List<User>): List<Long> {
        return userDao.insertUsers(users)
    }
    
    // Update
    suspend fun update(user: User) {
        userDao.updateUser(user)
    }
    
    suspend fun updateUserName(userId: Long, newName: String) {
        userDao.updateUserName(userId, newName)
    }
    
    // Delete
    suspend fun delete(user: User) {
        userDao.deleteUser(user)
    }
    
    suspend fun deleteById(userId: Long) {
        userDao.deleteUserById(userId)
    }
    
    suspend fun deleteAll() {
        userDao.deleteAllUsers()
    }
    
    // Query
    suspend fun getUserById(userId: Long): User? {
        return userDao.getUserById(userId)
    }
    
    fun searchUsers(query: String): Flow<List<User>> {
        return userDao.searchUsers(query)
    }
    
    fun getActiveUsers(): Flow<List<User>> {
        return userDao.getActiveUsers()
    }
    
    suspend fun getUserCount(): Int {
        return userDao.getUserCount()
    }
}
```

### Java Repository

```java
package com.example.roomdemo;

import androidx.lifecycle.LiveData;
import android.os.AsyncTask;
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
    
    // Expose LiveData
    public LiveData<List<User>> getAllUsers() {
        return userDao.getAllUsersLiveData();
    }
    
    // Insert (background thread)
    public void insert(User user) {
        executorService.execute(() -> userDao.insertUser(user));
    }
    
    // Update
    public void update(User user) {
        executorService.execute(() -> userDao.updateUser(user));
    }
    
    // Delete
    public void delete(User user) {
        executorService.execute(() -> userDao.deleteUser(user));
    }
    
    public void deleteById(long userId) {
        executorService.execute(() -> userDao.deleteUserById(userId));
    }
    
    // Search
    public LiveData<List<User>> searchUsers(String query) {
        return userDao.searchUsers(query);
    }
}
```

---

## MVVM Integration

### Kotlin ViewModel

```kotlin
package com.example.roomdemo

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import kotlinx.coroutines.flow.*
import kotlinx.coroutines.launch

class UserViewModel(private val repository: UserRepository) : ViewModel() {
    
    // Expose Flow from repository
    val allUsers: StateFlow<List<User>> = repository.allUsers
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = emptyList()
        )
    
    private val _currentUser = MutableStateFlow<User?>(null)
    val currentUser: StateFlow<User?> = _currentUser.asStateFlow()
    
    private val _isLoading = MutableStateFlow(false)
    val isLoading: StateFlow<Boolean> = _isLoading.asStateFlow()
    
    // Search query
    private val _searchQuery = MutableStateFlow("")
    val searchQuery: StateFlow<String> = _searchQuery.asStateFlow()
    
    val searchResults: StateFlow<List<User>> = searchQuery
        .debounce(300)  // Wait 300ms after user stops typing
        .flatMapLatest { query ->
            if (query.isEmpty()) {
                flowOf(emptyList())
            } else {
                repository.searchUsers(query)
            }
        }
        .stateIn(viewModelScope, SharingStarted.Lazily, emptyList())
    
    // Insert user
    fun insertUser(user: User) = viewModelScope.launch {
        _isLoading.value = true
        try {
            val id = repository.insert(user)
            // Handle success (e.g., show toast)
        } catch (e: Exception) {
            // Handle error
        } finally {
            _isLoading.value = false
        }
    }
    
    // Update user
    fun updateUser(user: User) = viewModelScope.launch {
        repository.update(user)
    }
    
    // Delete user
    fun deleteUser(user: User) = viewModelScope.launch {
        repository.delete(user)
    }
    
    // Load specific user
    fun loadUser(userId: Long) = viewModelScope.launch {
        _currentUser.value = repository.getUserById(userId)
    }
    
    // Update search query
    fun updateSearchQuery(query: String) {
        _searchQuery.value = query
    }
}
```

### Java ViewModel

```java
package com.example.roomdemo;

import androidx.lifecycle.LiveData;
import androidx.lifecycle.ViewModel;

public class UserViewModel extends ViewModel {
    private final UserRepository repository;
    private final LiveData<List<User>> allUsers;
    
    public UserViewModel(UserRepository repository) {
        this.repository = repository;
        this.allUsers = repository.getAllUsers();
    }
    
    public LiveData<List<User>> getAllUsers() {
        return allUsers;
    }
    
    public void insert(User user) {
        repository.insert(user);
    }
    
    public void update(User user) {
        repository.update(user);
    }
    
    public void delete(User user) {
        repository.delete(user);
    }
    
    public LiveData<List<User>> searchUsers(String query) {
        return repository.searchUsers(query);
    }
}
```

### ViewModelFactory

**Kotlin:**
```kotlin
class UserViewModelFactory(
    private val repository: UserRepository
) : ViewModelProvider.Factory {
    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        if (modelClass.isAssignableFrom(UserViewModel::class.java)) {
            @Suppress("UNCHECKED_CAST")
            return UserViewModel(repository) as T
        }
        throw IllegalArgumentException("Unknown ViewModel class")
    }
}
```

**Java:**
```java
public class UserViewModelFactory implements ViewModelProvider.Factory {
    private final UserRepository repository;
    
    public UserViewModelFactory(UserRepository repository) {
        this.repository = repository;
    }
    
    @NonNull
    @Override
    public <T extends ViewModel> T create(@NonNull Class<T> modelClass) {
        if (modelClass.isAssignableFrom(UserViewModel.class)) {
            return (T) new UserViewModel(repository);
        }
        throw new IllegalArgumentException("Unknown ViewModel class");
    }
}
```

---

## Complete CRUD Examples

### Kotlin Complete Example

**1. Application Class:**
```kotlin
class MyApp : Application() {
    val database by lazy { AppDatabase.getDatabase(this) }
    val repository by lazy { UserRepository(database.userDao()) }
}
```

**2. Activity with CRUD:**
```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding
    private lateinit var viewModel: UserViewModel
    private lateinit var adapter: UserAdapter
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        
        // Initialize ViewModel
        val app = application as MyApp
        viewModel = ViewModelProvider(
            this,
            UserViewModelFactory(app.repository)
        )[UserViewModel::class.java]
        
        setupRecyclerView()
        observeData()
        setupClickListeners()
    }
    
    private fun setupRecyclerView() {
        adapter = UserAdapter(
            onEditClick = { user -> showEditDialog(user) },
            onDeleteClick = { user -> viewModel.deleteUser(user) }
        )
        binding.recyclerView.adapter = adapter
        binding.recyclerView.layoutManager = LinearLayoutManager(this)
    }
    
    private fun observeData() {
        // Observe all users
        lifecycleScope.launch {
            viewModel.allUsers.collect { users ->
                adapter.submitList(users)
            }
        }
        
        // Observe loading state
        lifecycleScope.launch {
            viewModel.isLoading.collect { isLoading ->
                binding.progressBar.isVisible = isLoading
            }
        }
    }
    
    private fun setupClickListeners() {
        binding.fabAdd.setOnClickListener {
            showAddDialog()
        }
        
        binding.editTextSearch.addTextChangedListener { text ->
            viewModel.updateSearchQuery(text.toString())
        }
    }
    
    private fun showAddDialog() {
        val dialogView = layoutInflater.inflate(R.layout.dialog_add_user, null)
        val editName = dialogView.findViewById<EditText>(R.id.editName)
        val editEmail = dialogView.findViewById<EditText>(R.id.editEmail)
        val editAge = dialogView.findViewById<EditText>(R.id.editAge)
        
        AlertDialog.Builder(this)
            .setTitle("Add User")
            .setView(dialogView)
            .setPositiveButton("Add") { _, _ ->
                val user = User(
                    name = editName.text.toString(),
                    email = editEmail.text.toString(),
                    age = editAge.text.toString().toIntOrNull() ?: 0
                )
                viewModel.insertUser(user)
            }
            .setNegativeButton("Cancel", null)
            .show()
    }
    
    private fun showEditDialog(user: User) {
        // Similar to showAddDialog but with pre-filled data
        // Call viewModel.updateUser(updatedUser)
    }
}
```

**3. RecyclerView Adapter:**
```kotlin
class UserAdapter(
    private val onEditClick: (User) -> Unit,
    private val onDeleteClick: (User) -> Unit
) : ListAdapter<User, UserAdapter.UserViewHolder>(UserDiffCallback()) {
    
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): UserViewHolder {
        val binding = ItemUserBinding.inflate(
            LayoutInflater.from(parent.context), parent, false
        )
        return UserViewHolder(binding)
    }
    
    override fun onBindViewHolder(holder: UserViewHolder, position: Int) {
        holder.bind(getItem(position))
    }
    
    inner class UserViewHolder(private val binding: ItemUserBinding) :
        RecyclerView.ViewHolder(binding.root) {
        
        fun bind(user: User) {
            binding.textName.text = user.name
            binding.textEmail.text = user.email
            binding.textAge.text = "Age: ${user.age}"
            
            binding.buttonEdit.setOnClickListener { onEditClick(user) }
            binding.buttonDelete.setOnClickListener { onDeleteClick(user) }
        }
    }
    
    class UserDiffCallback : DiffUtil.ItemCallback<User>() {
        override fun areItemsTheSame(oldItem: User, newItem: User): Boolean {
            return oldItem.id == newItem.id
        }
        
        override fun areContentsTheSame(oldItem: User, newItem: User): Boolean {
            return oldItem == newItem
        }
    }
}
```

### Java Complete Example

**1. Application Class:**
```java
public class MyApp extends Application {
    private AppDatabase database;
    private UserRepository repository;
    
    @Override
    public void onCreate() {
        super.onCreate();
        database = AppDatabase.getDatabase(this);
        repository = new UserRepository(database.userDao());
    }
    
    public UserRepository getRepository() {
        return repository;
    }
}
```

**2. Activity with CRUD:**
```java
public class MainActivity extends AppCompatActivity {
    private UserViewModel viewModel;
    private UserAdapter adapter;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        // Initialize ViewModel
        MyApp app = (MyApp) getApplication();
        UserViewModelFactory factory = new UserViewModelFactory(app.getRepository());
        viewModel = new ViewModelProvider(this, factory).get(UserViewModel.class);
        
        setupRecyclerView();
        observeData();
    }
    
    private void setupRecyclerView() {
        RecyclerView recyclerView = findViewById(R.id.recyclerView);
        adapter = new UserAdapter();
        recyclerView.setAdapter(adapter);
        recyclerView.setLayoutManager(new LinearLayoutManager(this));
    }
    
    private void observeData() {
        viewModel.getAllUsers().observe(this, users -> {
            adapter.submitList(users);
        });
    }
    
    public void addUser(View view) {
        // Show dialog and call viewModel.insert()
    }
}
```

---

## Kotlin vs Java Comparison

| Feature | **Kotlin** | **Java** |
|---------|-----------|----------|
| **Coroutines** | ✅ Native `suspend` functions | ❌ Requires AsyncTask/Executors |
| **Data Classes** | ✅ `data class` auto-generates | ❌ Manual getters/setters/equals |
| **Null Safety** | ✅ Built-in (`?`, `!!`) | ❌ Manual null checks |
| **Flow** | ✅ Native support | ❌ Only LiveData |
| **Default Values** | ✅ `val id: Long = 0` | ❌ Must initialize in constructor |
| **String Templates** | ✅ Multiline SQL | ❌ String concatenation |
| **Annotation Processor** | `kapt` | `annotationProcessor` |
| **Boilerplate** | Minimal | Verbose |
| **Primary Keys** | ✅ `val id: Long = 0` | ❌ Handle `long` vs `Long` |

### Code Comparison

**Kotlin (concise):**
```kotlin
@Entity
data class User(
    @PrimaryKey(autoGenerate = true) val id: Long = 0,
    val name: String,
    val email: String
)
```

**Java (verbose):**
```java
@Entity
public class User {
    @PrimaryKey(autoGenerate = true)
    public long id;
    public String name;
    public String email;
    
    public User() {}
    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }
    // + getters/setters (8+ lines)
}
```

---

## Best Practices

### 1. Database Design

✅ **DO:**
- Use meaningful table names (`users`, `tasks`)
- Add indexes for frequently queried columns
- Use foreign keys with appropriate `onDelete` actions
- Export schema for migration testing

❌ **DON'T:**
- Store large BLOBs (images) - use file paths instead
- Use `SELECT *` in production (specify columns)
- Forget to handle migrations

### 2. DAO Design

✅ **DO:**
- Use `suspend` functions for single queries (Kotlin)
- Use `Flow` or `LiveData` for observing changes
- Use `@Transaction` for complex operations
- Keep DAOs focused (one per entity or related group)

❌ **DON'T:**
- Return `List` for large datasets (use `Flow` or `PagingSource`)
- Perform database operations on main thread
- Use raw queries when Room provides type-safe alternatives

### 3. Repository Pattern

✅ **DO:**
- Abstract DAO behind repository
- Handle exceptions in repository
- Use repository for multiple data sources (local + remote)
- Keep repository methods simple

❌ **DON'T:**
- Expose DAO directly to ViewModel
- Put business logic in repository (belongs in ViewModel)

### 4. Migrations

✅ **DO:**
- Increment version for schema changes
- Test migrations with `MigrationTestHelper`
- Export schema files
- Use descriptive migration names

❌ **DON'T:**
- Use `fallbackToDestructiveMigration()` in production
- Skip migration testing
- Forget to handle data transformation

### 5. Performance

✅ **DO:**
- Use indexes on foreign keys and search columns
- Batch inserts with `@Insert` accepting `List`
- Use `@Transaction` for multiple related operations
- Profile database queries with Android Profiler

❌ **DON'T:**
- Query database on main thread
- Fetch all data at once (use pagination)
- Ignore query performance warnings

### 6. TypeConverters

✅ **DO:**
- Handle null values gracefully
- Use efficient serialization (Gson/Moshi)
- Cache serializer instances
- Document complex conversions

❌ **DON'T:**
- Convert large objects (split into entities)
- Ignore conversion errors
- Use slow serialization in hot paths

---

## Testing Room Database

### Unit Testing DAO

**Kotlin:**
```kotlin
@RunWith(AndroidJUnit4::class)
class UserDaoTest {
    private lateinit var database: AppDatabase
    private lateinit var userDao: UserDao
    
    @Before
    fun setup() {
        val context = ApplicationProvider.getApplicationContext<Context>()
        database = Room.inMemoryDatabaseBuilder(context, AppDatabase::class.java)
            .allowMainThreadQueries()  // Test only!
            .build()
        userDao = database.userDao()
    }
    
    @After
    fun teardown() {
        database.close()
    }
    
    @Test
    fun insertAndGetUser() = runBlocking {
        // Insert
        val user = User(name = "Test User", email = "test@test.com", age = 25)
        val id = userDao.insertUser(user)
        
        // Retrieve
        val retrieved = userDao.getUserById(id)
        
        // Assert
        assertNotNull(retrieved)
        assertEquals("Test User", retrieved?.name)
        assertEquals("test@test.com", retrieved?.email)
    }
    
    @Test
    fun deleteUser() = runBlocking {
        val user = User(name = "Test", email = "test@test.com", age = 25)
        val id = userDao.insertUser(user)
        
        userDao.deleteUserById(id)
        
        val retrieved = userDao.getUserById(id)
        assertNull(retrieved)
    }
}
```

---

## Summary

### Room Components Recap

1. **Entity** - Maps to table (`@Entity`)
2. **DAO** - Database operations (`@Dao`)
3. **Database** - Container (`@Database`)
4. **TypeConverters** - Complex type serialization
5. **Migrations** - Schema version management
6. **Repository** - Data abstraction layer
7. **ViewModel** - UI state + business logic

### When to Use Room

✅ **Use Room when:**
- Structured relational data
- Offline-first apps
- Complex queries with relationships
- Need compile-time query verification
- LiveData/Flow reactive updates

❌ **Consider alternatives when:**
- Simple key-value storage (use DataStore)
- Large files/media (use file system)
- NoSQL requirements (use Firebase/Realm)

---

**Perfect for your Samsung daemon/middleware projects** - persistent device state, configuration storage, offline data sync! 🚀

---

**Created for**: Android Development Reference  
**Last Updated**: February 2026  
**Author**: Comprehensive Kotlin & Java Guide  
**Topics Covered**: Room Database, Entities, DAOs, TypeConverters, Migrations, MVVM, CRUD Operations
