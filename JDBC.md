# JDBC

## 重點整理

### 何謂JDBC
#### 全名 : Java Database Connectivity 
#### 共分成4種
1. JDBC-ODBC bridge driver (少用)
2. Native-API driver (少用)
3. Network Protocol driver (金流系統會需要用到)
    * 透過middleware存取DB
5. Thin driver (主流)
    * 效率好,制式化規則

---
### transaction   **m10**

#### 4個特性 (ACID)
1. 基元性(**A**tomicity)
    * 全部成功或失敗(100% 或 0%)
3. 一致性(**C**onsistency)
    * DB裡的不同table間的primary key要一致
    * 修改時要保持一致性
3. 獨立性(**I**solation)
    * 每個transaction相互不影響,也就是說執行結果不會受到其他transaction影響而改變
4. 永久性(**D**urability)
    * commit後,資料就永久改變。無法回復之前的資料

#### 備註
* InnoDB(預設)才具有transaction功能
* MyIsAM就沒有transaction功能

#### 範例 **m11,12**
```java=
// transcation
System.out.println("transcation...");
try {
	conn.setAutoCommit(false); // 設定auto commit 為false
	stmt = conn.createStatement();
	stmt.execute("Update city set populatio = 100 where name = 'Kabul'");
	stmt.executeQuery("select * from city where name = 'Kabul'");
	conn.commit(); // 資料無誤才commit
			// 資料有誤會有exception,要用try-catch包起來
} catch (SQLException e) {
	try {          // 避免rollback失敗,再用一個try-catch(通常失敗會是系統問題)                     
		conn.rollback(); // 資料有誤就rollback
	} catch (SQLException e1) {
		e1.printStackTrace();
	}
	e.printStackTrace();
}
```
---

### JDBC呼叫物件順序

#### 1.註冊驅動程式
Class.forName(JDBC_DRIVER); **m03**
#### 2.建立與資料庫連線
==conn===DriverManager.getConnection(DB_URL,USER,PASS);  **m03**
#### 3.建立敘述(取得db裡資料的方法)
* 有三種方式
1. ==stmt===conn.createStatement();    **m04** 
    * 需輸入SQL完整敘述,不加參數
```java=
// STEP4:Execute a query
System.out.println("Creating statement...");
stmt = conn.createStatement();
String sql;
sql = "SELECT id,name,countrycode,district,population FROM city limit 10";
ResultSet rs = stmt.executeQuery(sql);

```
2. ==stmt===conn.prepareStatement();   **m06**
    * 需輸入SQL部分敘述,加參數
##### 方法(給予參數)
* setInt(index,value)
* setString(index,value)
```java=
// STEP4:Execute a query   (針對第4步驟做更改)
System.out.println("Creating statement...");
stmt =conn.prepareStatement("select id,name,countrycode,district,population from city limit ?"); 
stmt.setInt(1, 10);	
ResultSet rs = stmt.executeQuery();			
stmt.clearParameters();  // 清除傳入參數
```
3. CallableStatement ==cstmt== = conn.prepareCall("{call procedurename(?)}"); **m09**
    * 不需要給SQL敘述,透過方法呼叫,可以給參數
    * 要配合stored procedure
    * 若是考慮資安,會常用此方法
```java=
// STEP4: CallableStatement
System.out.println("Creating CallableStatement...");
CallableStatement cstmt = conn.prepareCall("{call getCity(?)}");
cstmt.setInt(1, 20);
ResultSet rs =cstmt.executeQuery();
cstmt.clearParameters();   // 清除傳入參數
```

#### 3-1.取得database的系統資料(驅動程式,使用者,資料庫)
DatabaseMetaData dbmd = ==conn.getMetaData()==; **m08**
* Connection包含DatabaseMetaData的物件
* 要先透過Connection取得DatabaseMetaData的物件,才可以使用DatabaseMetaData的相關方法去取得系統資料(驅動程式,使用者,資料庫)
```java=
// STEP3-1: DatabaseMetaData
DatabaseMetaData dbmd = conn.getMetaData();                  
// 透過Connection呼叫DatabaseMetaData的物件					
// 使用DatabaseMetabase的相關方法取得資料
System.out.println("Driver Name: " + dbmd.getDriverName());     
System.out.println("Driver Version: " + dbmd.getDriverVersion());   
System.out.println("User Name: " + dbmd.getUserName());
System.out.println("Database Product Name: " + dbmd.getDatabaseProductName());       
System.out.println("Database Product Version: " + dbmd.getDatabaseProductVersion()); 
```
##### 方法
* getDriverName()
* getDriverVersion()
* getUserName()
* DatabaseProductName()
* DatabaseProductVersion()
#### 4.查詢後回傳的資料物件(ResultSet)
ResultSet ==rs== =stmt.executeQuery(); **m05**

#### 4-1.取得ResultSet的系統資料(table名稱,欄位資料,型態,名稱)
ResultSetMetaData metadata ===rs.getMetaData()==; **m07**
* ResultSet包含ResultSetMetaData的物件
* 要先透過ResultSet取得ResultSetMetaData的物件,才可以使用ResultSetMetaData的相關方法去取得系統資料(table名稱,欄位資料,型態,名稱)
```java=
// step4-1:get MetaData : table name,Column name,Column type
System.out.println("get meta data...");
ResultSetMetaData metadata = rs.getMetaData();

for (int i = 1; i <= metadata.getColumnCount(); i++) {
	System.out.print(metadata.getTableName(i)+".");
	System.out.print(metadata.getColumnName(i)+"\t|\t");
	System.out.println(metadata.getColumnTypeName(i));
}
```
##### 方法
* getColumnCount()
* getColumnName()
* getColumnTypeName()
* getTableName()

#### 5.將rs的資料透過迴圈將資料取出 **m05**
* 如果是select敘述,透過stmt.executeQuery(),配合while(rs.next())  # select會==回傳rs型態==的資料
```java=
// STEP4:Execute a query
System.out.println("Creating statement...");
stmt = conn.createStatement();
String sql;
sql = "SELECT id,name,countrycode,district,population FROM city limit 10";
ResultSet rs = stmt.executeQuery(sql);
// STEP5:Extract data from result set
while (rs.next()) {
	// Retrieve by column name
	int id = rs.getInt("id");
	String name = rs.getString("name");
	String countryCode = rs.getString("countryCode");
	String district = rs.getString("district");
	int population = rs.getInt("population");
```
* 如果是create,drop,insert,update,delete etc,透過stmt.executeUpdate(),配合if取出值if(a>0)
* 無回傳值,只會回傳異動筆數==整數型態==,故用if判斷是否有異動。

##### 方法
* rs.next
* rs.getInt
* rs.getString
#### 5-1.將資料印出來(原始方法)
```java=
        // Display values
	System.out.print("id: " + id);
	System.out.print(",name: " + name);
	System.out.print(",countryCode: " + countryCode);
	System.out.print(",district: " + district);
	System.out.println(",population: " + population);
}
```
#### 5-1.將rs的資料丟進javabean裡,再將javabean丟進arraylist,再用for-each取得所有資料 **m05** (javabean作法)
* 目的: 為了重複使用資料
* 利用collection(容器物件):arraylist,set,map (能裝物件的物件)
* javabean:只有屬性,getter&setter,constructor的Class。一筆資料就是一個物件
```java=
// Write to java bean (一筆資料>一筆物件)
	City myCity = new City();
	myCity.setId(id);
	myCity.setName(name);
	myCity.setCountrycode(countryCode);
	myCity.setDistrict(district);
	myCity.setPopulation(population);
// STEP5-1: 將java bean丟進去arraylist裡 (ArrayList: Java概念)
// STEP5-2: 要利用泛型
	ArrayList<City> a = new ArrayList<City>();
	a.add(myCity);
// STEP5-3: 用for each 取的所有資料
	for (City city : a) {
		System.out.println(city.getALL()); 
                            // 透過方法取得所有資料
	}
```
---
### 批次處理(Batch Processing)  **m13**
* 增加可讀性,易於維護
* 代表執行很多敘述
* 可配合建立敘述搭配使用
    1. createStatement
    2. PrepareStatement
    3. CallableStatement
##### 方法
* addBatch()
* executeBatch()
```java=
// STEP4:Batch
System.out.println("Batch...");

conn.setAutoCommit(false);
Statement bstmt = conn.createStatement();
bstmt.addBatch("Update city set population = 0 where name= 'Kabul'");
bstmt.addBatch("Update city set population = 99 where name= 'Qandahar'");
bstmt.addBatch("Update city set population = 12 where name='Herat'");
bstmt.executeBatch();
conn.commit();
```

* 搭配PrepareStatement **m14**
```java=
// STEP4:Batch + PrepareStatement
System.out.println("Batch...");

conn.setAutoCommit(false);
PreparedStatement pstmt = conn.prepareStatement("Update city set population = ? where name=?");
pstmt.setInt(1, 100);
pstmt.setString(2, "Kabul");
pstmt.addBatch();
			
pstmt.setInt(1, 999999);
pstmt.setString(2, "Qandahar");
pstmt.addBatch();
			
pstmt.setInt(1, 12345);
pstmt.setString(2, "Herat");
pstmt.addBatch();
			
pstmt.executeBatch();
conn.commit();
```
* 執行批次新增處理 (讀CSV檔,批次新增) **m15** 
```java=
// STEP4:Batch + PrepareStatement + CSV
System.out.println("Insert Batch...");

String sql_insert = "insert into parts(pid,pname,pcolor) values (?,?,?);";

// transaction 開始
conn.setAutoCommit(false); 

PreparedStatement bstmt = conn.prepareStatement(sql_insert); // 開始statement
List<List<String>> records = new ArrayList<>();   // new一個arraylist,
try (BufferedReader br = new BufferedReader(new FileReader("data/goods.csv"))) {
    String line;   							   // CSV檔是用,號做分隔
    while ((line = br.readLine()) != null) {  // 非null每次讀一列資料,
		String[] values = line.split(",");    // 以,號做切割,將資料放進字串型態的陣列裡
		records.add(Arrays.asList(values));   // 將字串型態的陣列(values)用add加進陣列(records)裡
        						  // List<List<String>>
	}
} catch (IOException e) {
}
for (int i = 0; i < records.size(); i++) {
	for (int j = 0; j < records.get(i).size(); j++) {
		bstmt.setString(j+1, records.get(i).get(j));
	}
	bstmt.addBatch();
}
System.out.println("bstate:" + bstmt);

bstmt.executeBatch();
conn.commit(); // transaction結束
```
---
### 連接池(Connection Pool) **m18**

#### 目的 : 透過連結池,能夠減少輸入輸出,使效能增加

#### 需匯入3個jar檔
1. common-dbcp-1.4.jar
2. mysql-connector-java-8.0.16.jar
3. org.apache.commons.pool.jar

#### 1.建立連結池物件
```java=
// 先建立Connection Pool的物件
	private static GenericObjectPool gPool =null;
	
	public DataSource setUpPool() throws Exception{
		Class.forName(JDBC_DRIVER);
		
		/* Create an Instance of GenericObjectPool 
		That Holds Our Pool of connections Objects! */
		gPool = new GenericObjectPool();
		gPool.setMaxActive(5);   // 最大連接數
		
		/* Creates a ConnectionFactory Object Which Will Be Use by the Pool 
		to Create the Connection Object! */
		ConnectionFactory cf = new DriverManagerConnectionFactory(DB_URL,USER ,PASS);
		
		/* Creates a PoolableConnectionFactory That Will Wraps the Connection Object 
		Created by the ConnectionFactory to Add Object Pooling Functionality */
		PoolableConnectionFactory pcf = new PoolableConnectionFactory(cf, gPool, null, null, false, true);
		return new PoolingDataSource(gPool);
	}
	public GenericObjectPool getConnectionPool() {
		return gPool;
	}
	
	// This Method is Used to Print The Connection Pool Status
	private void printDbSatus() {
		System.out.println(
				"Max.:"+getConnectionPool().getMaxActive()
				+";Active:"+getConnectionPool().getNumActive()
				+";Idle:"+getConnectionPool().getNumIdle()
				);
	}
	
```

#### 2.建立連接池
```java=
	public static void main(String[] args) {
    
		MyConnectionPool connPool = new MyConnectionPool(); //new一個connPool
		Connection conn = null;
		PreparedStatement pstmt = null;
		try {
			DataSource dataSource =connPool.setUpPool();   // 建立連線池->datasource
			connPool.printDbSatus();
			
			// Performing Database operation
			System.out.println("\n==A New Connection Object For Db Transaction==\n");
			conn= dataSource.getConnection();    // 建立連線->conn
			connPool.printDbSatus();
			
			pstmt=conn.prepareStatement("Select * from City limit 10");       // 建立敘述->pstmt
			ResultSet rs = pstmt.executeQuery();
```

#### 3.建立連線 ->conn
#### 4.建立敘述 ->pstmt
#### 5.回傳ResultSet物件 ->rs
---
### 參考資料
1. [JDBC語法](https://developer.android.com/reference/java/sql/package-summary)

