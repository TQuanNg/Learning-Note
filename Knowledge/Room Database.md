## Database
```
database/
	dao/
	entity/
	converter/
	relation/ (optional)
```

#### Step 1: Create **Database**: This is Room entry point
```
@Database(    
	entities = [StockEntity::class],
    version = 1
)

@TypeConverters(Converters::class) //  to register converter
abstract class AppDatabase : RoomDatabase() {    
    abstract fun stockDao(): StockDao
}
```

#### Step 2: Create **DatabaseModule**:
This **provide**:
	- singleton database
	- DAO instances
**Singleton Pattern**: Ensure that only ONE database instance in the app
Why?
- Database is expensive
- Should not be created multiple times
- Prevent corruption + memory leaks

#### Step 3: Set up **Converters**
Room only support primitive types, so for custom types, it need converter:
```
data class Stock()
```
which require:
```
class Converters {  
  
	@TypeConverter
		fun fromList(value: List<String>): String {  
		return value.joinToString(",")  
	}  
	  
	@TypeConverter 
		fun toList(value: String): List<String> {  
		return value.split(",")  
	}  
}
```

#Kotlin:companion object = static in Java: Shared for entire app, only 1 copy exist

#### DAO (Data Access Object): handles: insert, update, delete, query
```
@Dao  
interface StockDao {  
  
	@Query("SELECT * FROM stocks")  
	fun getStocks(): Flow<List<StockEntity>>
	
	@Insert(onConflict = OnConflictStrategy.REPLACE)  
	suspend fun insertStocks(stocks: List<StockEntity>)
```

❗When DAO should return **Flow**?
When automatic updates whenever database data changes. So it can **observe continuously**
Use Flow when:
- data displayed on screen
- data changes over time
- reactive UI needed

Ex: Stock list, bookmark list, chats
```
@Query("SELECT * FROM stocks")  
fun getStocks(): Flow<List<StockEntity>>
```


❗When DAO should use **suspend fun**?
When you only need **one-time operation/result**

💡**synchronized() {}**: to avoid race condition(when there might be two or more thread running)

💡**fallbackToDestructiveMigration**(): In development, it is fine since this method is to destroy old database when change schema, but in production, this should set to false, and should properly provide adequate migration