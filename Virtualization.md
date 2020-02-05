# Virtualization
###### tags: `ubuntu` `networking` `VM`
**VMware 跟 VirtualBox 差異**

**NetWorking**

| Column 1  | VMware | VirtualBox |
|:---------:| ------ | ----------:|
|    NAT    | VMnet8 |       預設 |
|  Bridged  | VMnet0 |     不使用 |
| host-only | VNnet1 |            |

**VMware**
* NAT
    * VMs間互通(透過VMnet8)
    * Host跟VMs(Guest)共享一張網卡
        * putty可以進VMs
* Bridged
    * VMs間互通(透過VMnet0)
        * 專業版才看的到VMnet0
    * 連接實體網卡
        * intel開頭
* Host-only
    * 只跟主機(Host)本身互通,VMs間不互通
    * 雲端管理員才用的到

備註:Mac要付費
**VirtualBox**
* NAT
    * 預設
    * IP一樣不會相衝
    * VMs間不互通
        * 只出不進
        * 不能用putty進VMs
* Bridged
    * 不討論,數據太大VirtualBox會掛掉,不能裝sever
* Host-Only
    * VMs間互通
    
---

###### tags: `docker` `VM`

|     | 名稱       | 公司                                  | 相容性      |
| --- |:---------- |:------------------------------------- |:----------- |
| 1   | VMware     | VMware                                | 可與2並存   |
| 2   | VirtualBox | Oracle的,從KVM來的(相似90%)           | 可與1並存   |
|     | KVM        | Linux Kernel Virtualization Mechanism |             |
| 3   | Hyper-V    | Microsoft                             | 與1,2不相容 |


:::danger
如果啟動Hyper-V,不能啟動VMware & VirtualBox。
==VMs會掛掉==,要重灌
:::

**為何要安裝Hyper-V呢?**
* docker是主流,以下版本須配合Hyper-V才能使用
    * ce版(community)
    * ee版(enterprise)
:::info
**為了避免這樣的狀況,==建議在Linux裡裝docker==,不要在windows裡裝docker**
:::



---
**正規建構環境架構**

```bash=1
             _____________
            |             |
            |  container  |
           _|_____________|_
          |                |
          |     Docker     |
         _|________________|_
        |                    |
        | Ubuntu1804(VMware) |
       _|____________________|_
      |                        |
      |       Host(win10)      |
      |________________________|
      
```
:::info
機器學習,深度學習有80%在windows灌不起來,還是乖乖地透過Linux :laughing: 
:::
**非正規建構環境架構**

```bash=1
           
           
                  _________          
                 |         |
         -------→| LinuxVM |
         |      _|_________|_
         |     |             |
         |     |   Hyper-V   |
         ↓     |      or     |
       Docker  |  VirtualBox |
       ________|_____________|_
      |                        |
      |       Host(win10)      |
      |________________________|
      
```
:::info
在win10裡面裝docker,一定要用VirtualBox。
如果使用Hyper-V會使VirtualBox跟VMware掛掉。==要重灌== 
:disappointed: 
:::

