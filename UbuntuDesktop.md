# Ubuntu desktop
###### tags: `desktop` `Linux` `docker` `ubuntu`
步驟
1. 灌VMware
    * 預設 spilt,500G 
        * 若是機房可選one disk
        * spilt 是考慮到拷貝及回家使用的需求
2. 新增VMs
    * 放在\VMs\Ubuntu1804Desktop
    * 2個 process,VM中的VM
        * 之後要在linux裡建立docker
    * RAM :12288MB (12*1024)
    * 選NAT路徑
    * DVD/CD:放入ubuntu_desktop_64.iso
        *  安裝完記得退片
3. 安裝Ubuntu desktop
    * 語系英文,最不容易出問題
    * 要走LVM(才可隨時調整disk大小)
    * SWAP
    ```
    
                swap in
          <------------------↰
         |                   |
     ____|____           ____|____
    |         |         |        |
    |   RAM   |         |  SWAP  |   swap(硬碟割出來)
    |_________|         |________|
         |                   |
         |                   |
         ↳------------------>
                swap out
                
    ```
   **超過ram,可以丟到swap區,等有空間再丟回來**
4. 更新
```shell=1
# 系統升級
sudo -i
apt update
apt upgrade
reboot
# 安裝必要套件
sudo -i
apt install dkms
apt install build-essential
# 檢查核心版本
uname -r    # 原本:5.0.0-23
# 安裝核心
apt install linux-headers-generic
apt install linux-headers-5.3.0-28-generic # 要升級的核心版本
# 安裝open VMware tools
apt install open-vm-tools-desktop
reboot
```


|         |            RedHat             |            Ubuntu             |
|:-------:|:-----------------------------:|:-----------------------------:|
| update  |           自動更新            |         每天下指令前          |
| upgrade | 稱為update,不能升級。==會離職== | 視情況(機器學習&深度學習版本) |

5. 更改設定
    * 字體:16~18
    * 時間:改成US
    * 更改更新通知:從不通知,兩個禮拜通知一次
6. 更改倉庫設定檔
    ```shell=1
    # 確認倉庫的設定檔 
    ls -l /etc/apt/source.list 
    # 出現 /etc/apt/source.list  : 主設定檔
    # 出現 /etc/apt/source.list.d/*.list :其他第三方的倉庫設定
    # e.g. MariaDB
    
    # 進nano編輯
    nano /etc/apt/sources.list
    # ^+\ 
    tw.archive.ubuntu.com 改成 free.nchc.org.tw
    # ^+s > ^+x
    
    # 檢查
    apt update
    # 順利執行就是成功
    ```
8. 安裝Fliezilla (自行考量)
    * 傳輸檔案方便 
    I. 安裝fliezilla主程式
    II. 將desktop接上socket
    ```shell=1
    sudo -i
    apt update
    apt install openssh-server
    lsof -nPi | grep ':22' # 檢查socket有無接上22/tcp(ssh的port number)
    ip addr show | grep 'inet ' #顯示IPv4的ip  
    ```
   * 連線
    III. 開啟fliezilla,打上ip,帳密,22，即可連線
:::info
**建立完可以備份一個clean的VM** :+1: 
:::

---
# 題外話
###### tags: 表格
Desktop 與 Sever 建置上的差異


|            | server  |   desktop    |
| ---------- |:-------:|:------------:|
| Networking | bridged |     NAT      |
| VM中的VM   |    x    | v 要玩docker |
| RAM        |   24G   |     12G      |
| DISK       |  500G   |     500G     |
| Process    |    4    |      2       |

