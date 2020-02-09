# 在Hadoop上的注意事項
###### tags: `hadoop` `ubuntu` `Linux` 
1. 要檢查沒有其他使用者才可以使用poweroff或是shutdown
 ```shell=1
 sudo -i  # 切成#
 who
 lsof -nPi | grep -i 'esta' 
 ```
 
    Who
    1. 0 :圖形介面(desktop)的terminal
    2. pts/1 :遠端terminal(putty) (配合最後一欄的IP:VMnet8的IP)
    3. tty/3 :3號的console (ctrl+alt+3)
:::warning
若不檢查,直接poweroff。hadoop上的這些資料==會損毀==
  * namenode
  * nodemanager
  * spark master 

:::
 
 

|      | poweroff |            shutdown            |
| ---- | -------- |:------------------------------:|
| 功用 | 直接關閉 | 預設等待1分鐘,會通知其他使用者 |

2. 檢查時區
```shell=1
timedatectl
```
:::warning
若==時間不對==,叢集會跑不起來。
:::


---

3. 正規步驟(玩Linux時)
###### tags: `小心離職` `資安`
* server走putty
* desktop直接走使用者
```bash=1



win10(Host)        Linux(使用者)         管理系統(root)
 _________   帳密①   _________ sudo -i②   _________
|         |------->|         |--------->|         |
|  putty  |        |         |          |   GOD   | 安裝軟體才會進來
|_________|<-------|_________|<---------|_________|
             exit⑥    ↑   |      exit③      |    ↑
                      |   |                 |    |
                      |   | su - hadoop④    |    |
               exit⑤  |   |                 |    |
                      |   |                 |    |
                    __|___↓__   su - hadoop |    |
                   |         |<----x--------     |   
                   |   man   |   不准(會離職)      |
                   |         |___________________|
                   |_________|    sudo -i (失敗)
                   管理應用程式     


```
* 帳密:管理員使用的login帳號(e.g :ubuntu)
* Debian,ubuntu的root帳號是被停用的,只能由使用者login,再切root
* 在root裡,只能exit,不能su
* 要正常exit,否則檔案==會損毀==
* 一個使用者只開一個terminal
:::danger
切一般使用者不用密碼就是錯誤的 ==會離職== 
```shell=1
ps -f #檢查(process fullist)
```
:::
4. 正規步驟(正式上班時)
    * 只能走putty
```bash=1



win10(Host)  
 _________   
|         |
|  putty  |
|_________|
   |    ↑    
   |    |    
   |    |    
   |    |    
   |    |    
   |    |   exit②   _________ 
   |    |__________|         |
   |               |   man   |
    -------------->|         |
     帳密(hadoop)①  |_________|
                    管理應用程式     


```


---




