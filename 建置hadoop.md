# 在Linux建置hadoop 
###### tags: `hadoop` `Server` `Linux` `ubuntu`
步驟
1. 創建hadoop帳號
```shell=1
sudo -i
adduser hadoop
password:hadoop (自行設定)
# 接著duble check
# 接著6次 enter
grep '^hadoop' /etc/passwd /etc/shadow #檢查帳號是否建立
```
2. 安裝hadoop
 ```shell=1
# 下載
wget http://ftp.tc.edu.tw/pub/Apache/hadoop/common/hadoop-3.2.1/hadoop-3.2.1.tar.gz 
 # 請右鍵製貼上
 # hadoop目前為3.2.1版本
 # 若版本更新自行去官網搜尋 https://www.apache.org/dyn/closer.cgi/hadoop/common
 
 # 確認
 ls -lh
 # 343MB
 
 # 查看
 tar -tvf hadoop.... # tab自動完成
 
 # 解開
 tar -xvf hadoop.... -C /usr/local  # tab自動完成
 # -C (change到 後面的目錄)
 # /usr/local 裝第三方軟體的目錄
 
 # 確認
 ls -l /usr/local
 
 # 查看檔案目錄真正大小
 du -sh /usr/local/ha.... # tab
 # 898MB
 
 # 查看使用狀況
 df -h /usr/local
 # 2%
 
 # 到別人家
 cd
 cd /usr/local
 
 # 建立捷徑
 ln -s hadoop-3.2.1 hadoop
 
 # 確認
 ls -l
 ```
3. 安裝openJDK8
 * 因為oracle java8 限制下載,故Debian無法透過透過apt install安裝oracle java8,因此使用openJDK8
```bash=1
apt search '^openJDK' |grep '8'
apt install openjdk-8-jdk
# 查詢版本
java -version
javac -version
```
4. 設定JAVA_HOME的PATH
```shell=1
# 找javac路徑
which javac #不是java喔
ls -l /usr/bin/javac
ls -l /usr/alternatives/javac
ls -l /usr/lib/...../java-8.../javac #tab
# 找到真正路徑(就是JAVA的HOME:/usr/lib/jvm/java-8-openjdk-amd64)

# 開始設定
sudo -i
cd /etc/profile.d # JAVA_HOME的PATH設定檔放這裡
ls *.sh  # 確定沒有變色(不用設定特別權限)
nano openjdk.sh

# in nano
export JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64"
# 宣告環境變數
# ^+s > ^+x

# 立即載入
. openjdk.sh
echo $JAVA_HOME

# 確認JAVA_HOME的內容是否正確
$JAVA_HOME/bin/javac -version
# 1.8.0_242 成功
```
5. 執行hadoop
   * 切換成hadoop帳號
```shell=1
# 切換帳號
su - hadoop

# 進到hadoop的bin裡
cd /usr/local/hadoop/bin

# 確認有無hadoop的執行檔
ls-l

# 執行
hadoop version  #command not found
# 目前路徑不會去搜尋目前目錄

# 執行目前目錄,會依$PATH去搜尋
./hadoop version  # hadoop 3.2.1 (成功)

```
6. 停用IPv6
* Hadoop只支援IPv4,把IPv6停掉,速度會快一倍


---

# 小提醒
* 更改設定 (用hadoop帳號登入)
    * 字體:16~18
    * 時間:改成US
    * 更改更新通知:從不通知,兩個禮拜通知一次

:::info
安裝完要再一次備份VM :+1: 
:::


---
**連結**
1. [在Hadoop上的注意事項](/CkvZCoF7QVu_oii4tve1kA)