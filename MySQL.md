# MySQL
###### tags: `MySQL` `MariaDB` 
## 下載

1. [下載XAMPP](https://www.apachefriends.org/zh_tw/index.html)
2. 只安裝MYSQL server
3. 從apache config >httpd.config >設定 Listen:80 > 81
4. 開啟admin,localhost > localhost :81 
* 預設是80,但被系統拿去用了
---
## 安裝MySQL

### 設定密碼(為了教學只是示範,不用真的設定)
1. 打開shell
```shell=
# 預設沒有密碼
mysql -u root 

# 設定密碼
set password for root@localhost=password('yourpassword')

# 取消密碼
set password for root@localhost=password('')

# 離開
\q
```
* root@localhost : 只允許本機連線 (MySQL預設使用者)
* root% 或 ::1   : 允許從其他電腦連線
2. 找使用者
```shell=
select user();
```
**備註:** 設定密碼同時==須更改apache config的設定檔==才能登入
apache config > phpMyAdmin >['password']='輸入新的密碼' 
### 刪除不需要user就能登入的帳號
* 新版已移除,不需要操作。當初為了方便而設立,現為安全考量而移除。

### 建立帳號
root 可以授權
一般user不能授權

```shell=
# 創建帳號
grant all *.* to test identified by 'test';
# 給予所有的db跟table權限到test使用者,密碼為test

# 立即寫入資料庫
flush privileges;
```
**備註** : 第一個\*號代表所有DB,第二個\*號代表所有table

---

## 快速導覽

### 目錄結構
1. bin : 放執行檔(指令)
    * mysqladmin:看admin相關函數
    * myisamchk :檢查表格有無損壞
    * mysqldump :備份
    * mysqlbinlog:查看歷程記錄
    * mysqlshow:看db跟tables
3. script : Perl script (Linux常用)
4. data : 放資料庫

### 常用指令
```shell=
# 連上別台電腦
mysql -h 輸入ip -u 帳號 -p

# 查看資料庫
show databases

# 使用資料庫
use 資料庫名稱;

# 查看表格
show tables;

# 看表格欄位
desc 表格名稱;

# 看表格內所有資料
select * from 表格名稱;

# 看多少個使用者
show processlist;
```


---

## 設計資料庫 ==重要==
### 觀念 
* 避免異常發生(透過相依性互相牽制)
    * 異常輸入
    * 異常刪除
    * 異常更新
* 透過相依性將資料分成好幾個table,彼此互相驗證資料是否正確
* 術語:外鍵參考外部資料的主鍵彼此驗證資料
* 備註: 每一個table都會有自己的主鍵,透過主鍵找到該table的所有資料。

### 正歸化步驟 
1. 去除重複(一個欄位只能放一個資料)
2. 去除相依性(有相依性的資料各自獨立成一個table)

舉例
```bash
# 原始資料
 _______________
|               |
| employeeID    |
| employeename  |  
| employeejob   |
| departmentID  |
| departmenName |
| skill         |
|_______________|

# 檢查有無重複
  A:無重複

# 檢查相依性,依照相依性分成3個table

   employee             department        empolyeeSkills   
 _______________      ________________     ____________
|               |    |                |   |            |  
| employeeID    |    | departmentID   |   | skill      |
| employeename  |    | departmenName  |   | employeeID |
| employeejob   |    |                |   |____________|
| departmentID  |    |                |
|_______________|    |________________|

主鍵:employeeID       主鍵:departmentID     主鍵:employeeID
外鍵:departmentID     無外鍵                 外鍵:employeeID


employee.departmentID(外鍵)會參考department.departmentID(主鍵)
employeeSkills.employeeID(外鍵)會參考employee.employeeID(主鍵)
```

備註:
* 有n個table,通常會有n-1個table互相驗證
* 主鍵也可以是外鍵
* 每個table只能有一個主鍵
* 多個欄位可以成為一個主鍵

---
## 創建database,table

### 命名規則
1. 不分大小寫
2. 不可以使用'/',  '\\'跟'.'
### create
#### create database
```sql=
create database employee;
show databases;
use employee;
```
#### create table
```sql=

```
### drop
### alter

### 參考講義謝謝 :+1: 

---

## 輸入,刪除,更新

### insert
### delete
### update

### 匯出
### 匯入
---

## 基礎查詢
### select columns from tables
### where
### aliase 別名
### distinct 去除重複
### group by 分類
### having
### order by
### limit

---

|      |      |      |
| ---- | ---- | ---- |
|      | select     |  where    |
| Text | group by | having |

---

## 進階查詢
### join





---

## 參考資料
1. 