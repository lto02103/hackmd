# mongoDB
###### tags: `NoSQL` `mongoDB`



## 視讀架構

### 符號解釋
1. {} 條件判斷,資料判斷
    * { key : value }
    * { 方式 : value }
3. [] 超過兩個條件(資料)以上
4. () 代表函數

### 分析結構
* 第一種
    * find()的\$in,\$nin,\$gt,\$lt,$not
    * find()的\$all,\$size,\$slice,\$elemath
##### key > 方式 > value
    
```javascript=
(
{key : {$方式 : value} }

)

```
```javascript=
(
{key : 
    {$方式:
        [value1,value2]     # 注意中括號
    } 
}
)

```
* 第二種 
    * find()的\$and,\$or,\$nor
    * update()的全部:\$set,\$inc,\$each,\$addToSet,\$pop,\$pull
##### 方式 > key > value
```javascript=
(
{$方式:
    {key:value}
}

)

```

##### 方式 > key > 方式> value
```javascript=
(
{$方式:[             # 注意中括號
    { key1:
        { $方式:value, $方式:value} },
    { key2: {$方式:value} }
   ]
}

)
```

---
## 速度
**一次塞1000筆** ==快==
```javascript=
var user1 = {
name : "Mark",
age : 18,
bmi : 10
},
count = 1000,
users = [];
for (var i=0;i<count;i++){
users.push(user1);      
}
db.user.insert(users,{ordered:false})   # for loop 結束才塞
```


**一次塞一筆** ==慢==
```javascript=1
var user1 = {
name : "Mark",
age : 18,
bmi : 10
},
count = 1000,
users = [];
for (var i=0;i<count;i++){
db.user.insert(users,{ordered:false})   # 一次塞一筆
}

```
---


## 搜尋基本操作 (find)
**find的搜尋條件**
==一次只找一個值==

### and
##### 找出符合age大於等於30,小於60==且==fans大於等於200
* 標準做法
```javascript=
db.usr.find(   
{"$and":[                       # 用and處理

{age:{"$gte":30,"$lt":60}},     # 條件1
{fans:{"$gte":200}}             # 條件2

]}

)

```

### or
##### 找出符合fans小於等於100==或==likes小於100
```javascript=
db.user.find(
{"$or":[                 # 用or處理

{fans:{"$lte":100}},     # 條件1
{likes:{"$lt":100}}      # 條件2
 

]}

)
```

### in
##### 找出age為25或60的人
**寫法1** ==快==
```javascript=
db.user.find(
{age:   # 條件

  {"$in":[         # 用in處理
          25,      # 子條件1
          60       # 子條件2
            ]
  }
}

)
```
**寫法2**
```javascript=
db.user.find(
{"$or":[    # 用or處理

  {age:25},  # 條件1
  {age:60}   # 條件2
  
  ]}
)
```

### not
```javascript=
db.user.find(
{likes:

    {"$not":
    {"$gt":100}}

}

)
```


---

**練習**

1. 傳回dht11資料集的總資料數 25筆
```javascript=
db.dht11.count() 
```

2. 查詢並傳回所有溫度值為24的資料  5筆
```javascript=
db.dht11.find(
{溫度:24}

)
```
3. 所有溫度大於25度的資料  0筆
```javascript=
db.dht11.find(
{溫度:"$gt25"}
)
```
4. 所有溫度大於22且小於30度的資料 15筆
```javascript=
db.dht11.find(
{"$and":[
    {溫度:{"$gt":22,"$lt":30}}
    
    ]
})
```

5. 所有溫度為22、23或24的資料  15筆
```javascript=
db.dht11.find(
{溫度:   
    {"$in":[
        22,
        23,
        24
        ]
    }    
    
})
```

6. 溫度大於25度的資料筆數  0筆
```javascript=
db.dht11.count(
{溫度:{"$gt":25}   
})
```
7. 傳回溫度值大於或等於24，而且，濕度值大於或等於60的資料 1筆
```javascript=
db.dht11.find(
{"$and":[
    {溫度:{"$gte":24}},   
    {濕度:{"$gte":60}}
    ]
})
```

---
## 更新 (update)

### update
##### 更改欄位資料

```javascript=
原本資料
db.user.insert({"name":"mark","age":18});
db.user.insert({"name":"steven","age":23});
db.user.insert({"name":"jj","age":23});
db.user.insert({"name":"mark","age":40});
db.user.insert({"name":"mark","age":50});
```
##### 若有多筆資料,只會更改第一筆資料
```javascript=
db.user.update(

{name:"mark"},         # 查詢條件
{name:"mark",age:22}   # 更改條件

)
```
### set
##### 若要更改多筆資料 要配合修改器(set,inc) + multi:true
```javascript=
db.user.update(

{name:"mark"},         # 查詢條件
{$set:{age:18}},       # 更改條件
                       # 更改多筆資料 
{
multi:true
}                      # true才會更改多筆資料
    

)
```
### inc
**練習**
##### 每年多一歲
```javascript=
db.user.update(

{},                    # 所有條件都更改
{$inc:{age:1}},        # 加一歲

{
multi:true             # 更改多筆資料
}

)
```
---
### each
##### 新增多筆資料
```javascript=
db.user.insert(
{
name:"mark", 
fans:["steven","crisis","stanly"]
})

db.user.update(
{name:"mark"},                                 # 查詢條件
{$push: {fans:{ $each:["peter","mary"] } }     # 更改條件
                                               # push + each :新增多筆資料
                                               
})
```

### slice
##### 限制大小 
```javascript=
db.user.update(
{name:"mark"},                   # 原本條件
{$push :                         # 更改條件  
    {fans :                               # push+each :新增多筆
        {$each:["jack","lnadry","max"],   # slice :只保留最後
        $slice : -5 }
    }
}       
)
```
**addToSet**
##### 變成集合(資料不重複)

```javascript=
db.user.insert(
{name:"mark",
fans:["steven","crisis","stanly"]
})

db.user.update(
{name:"mark"},
{$addToSet: 
    {fans :
        {$each :["steven","jack"]}
    }
}
)
```

### pop
##### 刪除 (從頭或尾) 
* 1: 從尾刪, -1: 從頭刪
```javascript=
db.user.insert(
{name:"mark",
fans:["steven","crisis","stanly"]
})

db.user.update(
{name:"mark"},
{$pop: {fans:1} }
)
```

### pull
##### 刪除 (指定)
```javascript=
db.user.insert(
{name:"mark",
fans:["steven","crisis","stanly"]
})

db.user.update(
{name:"mark"},              # 查詢條件
{$pull: {fans:"crisis"} }    # 更新條件
)
```
---

## 搜尋陣列欄位 (find)

資料
```javascript=
db.user.insert(
[
{id:"1",name:"mark",
fans:["steven","stanly","max"],
x:[10,20,30]},

{id:"2",name:"steven",
fans:["max","stanly"],
x:[5,6,30]},

{id:"3",name:"stanly",
fans:["steven","max"],
x:[15,6,30,40]},

{id:"4",name:"max",
fans:["steven","stanly"],
x:[15,26,330,41,1]}
]
)

```

### all
練習
##### 同時找有max跟steven 
==一個欄位找多個值==
```javascript=
db.user.find(
{fans:  {$all:["steven","max"]}  }
)
```

##### 只找max或steven
==一次只找一個值==
寫法1
```javascript=
db.user.find(
{$or:[
    
    {fans:"steven"},
    {fans:"max"}
]}

)

```
寫法2
```javascript=
db.user.find(
{fans:
   {$in:[
       "max",
       "steven"
       ]
   }
}

)

```
### size
##### 找有粉絲3位的人
```javascript=
db.user.find({"fans":{"$size" :3}})
```

### slice
##### 找mark的第一個粉絲
```javascript=
db.user.find(
{name:"mark"},
{fans: {$slice:1} }

)
```

### elemMatch
##### 找x >30, <100
```javascript=
db.user.find(
{x:
    {$elemMatch: {$gt:30,$lt:100} }
}
)

```
**練習**
##### 改成用find去找
```javascript=
db.user.find(
{$and:[
    {x: {$gt: 30,$lt:100} }

]}

)
```
**練習**
```javascript=
db.stock12.insert([
    {id:1,name:"peter",
	stock:["htc","asus","acer"],
	quantity:[10,20,30]},
    
    {id:2,name:"tony",
	stock:["acer","asus"],
	quantity:[5,6]},
    
    {id:3,name:"obama",
	stock:["htc","acer"],
	quantity:[30,40]},
    
    {id:4,name:"trump",
	stock:["htc","asus"],
	quantity:[330,41]}
]);

```

1. 尋找peter的第一個stock
```javascript=
db.stock12.find(

{name:"peter"},
{stock:{$slice:1}}

)
```

2. 尋找quantity中至少有一個值為大於20小於50的stock

```javascript=
db.stock12.find(
{$and:[
    {quantity: {$gt:20,$lt:50} }   
]}

)


```


3. 尋找stock中同時有htc、asus
```javascript=
# 1
db.stock12.find(
{$and:[
    {stock:"htc"},
    {stock:"asus"}   
]}

)

# 2
db.stock12.find(
{stock:  {$all:["htc","asus"]}  }

)
```

4.尋找有買兩張stock
```javascript=
db.stock12.find(
{stock: {$size:2} }

)

```
---

## 刪除 (remove)

資料
```javascript=

db.user.insert({name:"mark",age:23});
db.user.insert({name:"steven",age:23});
db.user.insert({name:"jj",age:23});
db.user.insert({name:"mark",age:50});
db.user.insert({name:"mark",age:40});
```

### remove
##### 只要條件符合就會刪除 (刪除3筆)
```javascript=
db.user.remove(

{name:"mark"}    # 預設: 只要符合條件就刪除

)
```
##### 若要只刪除一筆
```javascript=
db.user.remove(

{name:"mark"},

{justOne:true}    # 只刪除一筆

) 
```


##### drop 跟remove差別
* drop :刪除資料,連index也刪除
* remove: 只刪資料,不刪除index


### bulk
==效能比單純remove快==
```javascript=
# 先新增二筆資料
var bulk = db.user.initializeUnorderedBulkOp();
bulk.insert( { name: "mark"} );
bulk.insert( { name: "hoho"} );
bulk.execute();

# 然後再一口氣刪除掉   
var bulk = db.user.initializeUnorderedBulkOp();
bulk.find({}).remove();   # 找到所有條件,並移除
bulk.execute();    # 執行

```

---
## 搜尋的cursor運用
### limit(指定上限數)
##### 只顯示10筆
```javascript=
# 先新增1000筆
for (var i=0;i<1000;i++){
db.user.insert({x:i})
}

db.user.find().limit(10)
```

### skip (跳過幾筆資料)
##### 顯示11~20筆
```javascript=
db.user.find().skip(10).limit(10)
```
### sort (排序)
##### 1: 由小到大 , -1 :由大到小
```javascript=
db.user.find().skip(10).limit(10).sort({x:1})
```

---

## 搜尋原理
### index (索引)
==資料量大,index才有用處==
##### 比較有index跟沒有index的差別
```javascript=
# 產生10000筆資料
for (var i=0;i<10000;i++){
db.test.insert({
"x" : i
})
}

# 無索引,有sort
db.test.find({ x : {"$gt" : 5000}}).sort({x : -1}).explain("executionStats")

秒數:20s
掃描:10000筆
佔內存


# 有index,有sort
db.test.ensureIndex({ "x" : 1 })
db.test.find({ "x" : {"$gt" : 5000}}).sort({"x" : -1}).explain("executionStats")

秒數:2s
掃描:4999筆
不佔內存
```

##### 不要使用索引的時機點
* 搜尋結果 佔 原始資料 ==60%以上==(隨便找都找的到) 

```javascript=
# 新增資料
for (var i=0;i<10000;i++){
var value = (i<6000)?"1":"2" ;
db.test2.insert({
"x" : value
})
}

# 無index
db.test2.find({"x" : "1"}).explain("executionStats")

秒數:2s
掃描:10000筆

# 有index
db.test2.ensureIndex({ "x" : 1 })
db.test2.find({"x" : "1"}).explain("executionStats")

秒數:3s
掃描:6000筆
```

---

## 複合索引

### 先後順序會影響速度
```javascript=
# 資料
db.user.insert(
[
{ "name" : "mark00" , age:20 },
{ "name" : "mark01" , age:25 },
{ "name" : "mark02" , age:10 },
{ "name" : "mark03" , age:18 },
{ "name" : "mark04" , age:26 },
{ "name" : "mark05" , age:40 },
{ "name" : "mark06" , age:51 },
{ "name" : "mark07" , age:20 },
{ "name" : "mark08" , age:51 },
{ "name" : "mark00" , age:30 },
{ "name" : "mark00" , age:100 }
]
)

```
##### 狀況一
##### 1. 無index
```javascript=
db.user.find({}).sort({"age":1}).explain("executionStats") 
秒數:0s
掃描:11筆
佔內存

```
##### 2. {"name": 1 ,"age" : 1}的索引
```javascript=
db.user.ensureIndex({ "name" : 1 , "age" : 1 })
db.user.find({}).sort({"age" : 1}).explain("executionStats")

秒數:0s
掃描:11筆
佔內存
```


##### 3. {"age": 1 ,"name" : 1}的索引 ==好==
```javascript=
db.user.ensureIndex({ "age": 1 , "name" : 1 })
db.user.find({}).sort({"age" : 1}).explain("executionStats")

秒數:0s
掃描:11筆
不佔內存
```

##### 狀況二
##### 1. 無index
```javascript=
db.user.find({"name" : "mark00"}).sort({"age" : 1}).explain("executionStats")
秒數:0s
掃描:11筆
佔內存
```
##### 2. {"name": 1 ,"age" : 1}的索引  ==好==
```javascript=
db.user.ensureIndex({ "name" : 1 , "age" : 1 })
db.user.find({"name" : "mark00"}).sort({"age" : 1}).explain("executionStats")
秒數:0s
掃描:3筆
不佔內存
```


##### 3. {"age": 1 ,"name" : 1}的索引
```javascript=
db.user.ensureIndex({ "age": 1 , "name" : 1 })
db.user.find({"name" : "mark00"}).sort({"age" : 1}).explain("executionStats")
秒數:0s
掃描:11筆
不佔內存
```

* 建index時,要考慮效率

---
## 備份
### 倒進去,再倒出來
##### 不建議,轉json沒有詳細紀錄資料類型,會有資料面問題

```javascript=
# 匯出
mongoexport --db test --collection Currency --out Currency.json

指令             db         collection           路徑
# 匯入
mongoimport --db test --collection Currency --file Currency.json
```

##### 建議,轉bson會記錄所有資料型態
```javascript=
# 匯出
mongodump --collection Currency --db test --out "c:\share"
指令            collection           db          路徑        

# 匯入
mongorestore --db test --collection Currency c:\share\test\Currency.bson

```