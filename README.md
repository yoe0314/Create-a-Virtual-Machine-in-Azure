# Create a Virtual Machine in Azure

###### tags:`Azure VM` `2022`
[![hackmd-github-sync-badge](https://hackmd.io/P8trmX9zSvyvtTsmoL0ryQ/badge)](https://hackmd.io/P8trmX9zSvyvtTsmoL0ryQ)

- [Create a Virtual Machine in Azure](#create-a-virtual-machine-in-azure)
  * [1/ 建立VM](#1-建立-vm)
  * [2/ Cloud Shell 登入VM](#2-cloud-shell-登入-vm)
  * [3/ 本機PowerShell來停止或啟動VM](#3-本機-powershell-來停止或啟動-vm)
  * [4/ 調整VM規格](#4-調整-vm-規格)
  * [5/ 延伸VM功能](#5-延伸-vm-功能)
  * [Reference](#reference)

本篇內容將透過Azure Portal來建立一台Linux VM，Cloud Shell 登入VM、使用本機PowerShell來停止或啟動VM、調整VM規格。

## 1/ 建立VM
* Azure Portal 上方Search Bar搜尋`虛擬機器` or `Virtual Machine`
![](https://i.imgur.com/wXlqdzc.png)

* 基本
    * `資源群組`：可以新建一Resouce Group存放本次要建立的資源。
    * `虛擬機器名稱`：LinuxVM1(自行更改)。
    * `可用性選項`：備援機制，因為是測試用可以選擇不需要備援。
    * `影像`：Ubuntu Server 18.04 LTS - Gen2。
    * `大小`：選擇最小規格即可。
    * `Administrator帳戶`：可以選擇SSH公開金鑰或是密碼。
    * `連接埠規則`：一般建議不要有公用連接埠，測試用可以勾選允許。
    ![](https://i.imgur.com/VBz1gTs.png)

* 網路
    * `虛擬網路`：可以新建一個新的VNet。
    * `子網路`：在建立VNet時可以一併建立。
    > 詳細建立VNet的步驟可以去看我[先前的筆記](https://hackmd.io/-Y0NjGc3RgK2_UobhexsUg?both#2-%E5%BB%BA%E7%AB%8B%E8%99%9B%E6%93%AC%E7%B6%B2%E8%B7%AF)。
    * `公用IP`：給他一個公用IP供我們後續連線使用。
    * 其他選項預設即可。
    ![](https://i.imgur.com/BgrA1ah.png)

* `管理`、`進階`、`標籤`使用預設值即可，填完後建立。

## 2/ Cloud Shell 登入VM

* 建立好VM後，到VM的概觀記下Public IP。
![](https://i.imgur.com/baUxnM6.png)

* 接著我們使用Azure Cloud Shell來操作SSH，以便連接VM。
    * 開啟Cloud Shell
![](https://i.imgur.com/Bhz7p8r.png)
    * 指定你的管理員帳戶和VM的Public IP。
    * 剛才建立的Linux VM Public是20.48.21.9，管理帳號是local_admin。
    * 所以輸入 `ssh local_admin@20.48.21.9`。
    * 輸入yes接受VM的公開金鑰，並且輸入你的密碼。
![](https://i.imgur.com/mHZE4Dr.png)
    * 現在已經可以直接操作Linux VM。

## 3/ 本機PowerShell來停止或啟動VM
* 微軟會針對VM運行的每一分鐘收費，只要他處於已分配狀態(allocated)就還是會收費。
* 必須將VM關閉並且解除配置(deallocate)，才會停止收費，解除VM配置意即釋出任何指派給他的動態資源，Public IP / Private IP 等資源。

![](https://i.imgur.com/gBh1Eg0.png)
* 除了VM概觀上面有啟動、重新啟動、停止的按鈕外，我們也可以透過PowerShell來停止和啟動VM。
* 使用Cloud Shell時Azure會自動與Azure AD認證你的User account，但若是使用本機的Azure PowerShell的話就必須先==手動認證==才能夠繼續使用。
    > 本機安裝Azure PowerShell模組
    > [安裝 Azure Az PowerShell 模組 | Microsoft Docs
](https://docs.microsoft.com/zh-tw/powershell/azure/install-az-ps?view=azps-8.2.0)

* 開啟本機的PowerShell
```powershell
輸入 Connect-AzAccount，此時會出現登入畫面。
```

![](https://i.imgur.com/JJc3JVA.png)
* 登入成功後，會顯示你的訂用帳戶。
![](https://i.imgur.com/TUqa1vS.png)
* 透過指令來列出資源群組內的VM。
```powershell
Get-AzVM -ResourceGroupName 'VM_Lab’
```
![](https://i.imgur.com/jzZVU3Q.png)

* 使用指令來停止VM。
```powershell
Stop-AzVM -Name 'LinuxVM1' -ResourceGroupName 'VM_Lab' –Force

要加上-Force參數，確保VM不但會停止、也會解除配置
```
![](https://i.imgur.com/oT9DYH9.png)

* 可以到Azure Portal上面看VM是否有成功被關機。
*![](https://i.imgur.com/4TB98Ox.png)

* 我們再透過指令，再次啟動VM。
```powershell
Get-AzVM -Name 'LinuxVM1' -ResourceGroupName 'VM_Lab' | Start-AzVM
```
![](https://i.imgur.com/5Kdsidl.png)
* VM已成功開機。
![](https://i.imgur.com/oV217mL.png)

## 4/ 調整VM規格
:::warning
:warning: **注意** ! Azure調整VM規格時，必須重啟VM
:::
* 在VM頁面中，點選`大小`，選擇不同規格，點選`調整大小`。
![](https://i.imgur.com/sYJnTtV.png)
* 調整規格時需要一點時間。
![](https://i.imgur.com/7XCzwuo.png)

* 重新整理後應該就可以看到VM的規格已經改變。
![](https://i.imgur.com/4PhZfuj.png)

## 5/ 延伸VM功能
* 可以`啟用診斷紀錄`來收集VM的log。
* 啟用來賓層級監視
![](https://i.imgur.com/IjJ16rR.png)

:::info
:memo: **VM的診斷設定，需要存放在Storage Account，所以要事先建立。**
:::
## Reference
* [快速入門：在 Azure 入口網站中建立 Linux 虛擬機器](https://docs.microsoft.com/zh-tw/azure/virtual-machines/linux/quick-create-portal)
* [安裝 Azure Az PowerShell 模組 | Microsoft Docs
](https://docs.microsoft.com/zh-tw/powershell/azure/install-az-ps?view=azps-8.2.0)
* Microsoft Azure for Dummies, Warner, Timothy L.
