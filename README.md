# Windows Server 2003 AD 升級到 2022演示

Windows 網域樹系目前最高提供到2016版本，2003 AD要升級為2022中間有些許問題，包括介面、網路設定、FRS轉DFSR......等等。

## 一、環境建置

#### （一）、虛擬機配置

本次Lab建立三台虛擬機進行作業，如下說明：
1. Windows Server 2003, IP：10.100.100.170 （第一台AD）
2. Windows Server 20012R2, IP：10.100.100.171、DNS：指原本的AD，也就是第一台 （中繼AD）
3. Windows Server 2022, IP：10.100.100.172 、DNS：指上一台，也就是成為中繼站的AD（升級後最後一台AD）


#### （二）、第一台AD環境建置

以上說明3台虛擬機的配置後，接下來直接建立AD網域、樹系，域名取名為：`anontokyo.local`。
需要將其他的`DC`加入到`anontokyo.local`，因為2003預設是2000等級，所以這裡先將樹系等級升級，並且確認樹系等級提升至2003。
![image](https://github.com/Janalexei9/WINSRV_2003_AD_update_to_2022/assets/155059505/6e710d79-bd8f-4409-b265-e7b890085ddb)

![image](https://github.com/Janalexei9/WINSRV_2003_AD_update_to_2022/assets/155059505/70e39122-48ab-4035-9865-ec21df844088)

![image](https://github.com/Janalexei9/WINSRV_2003_AD_update_to_2022/assets/155059505/dc16b931-50f6-493b-a22e-004c9027a46f)


## 二、五大角色移轉

#### （一）、第二台機器進入網域並轉移五大角色

將第二台加入網域並升級為`DC`，中間過程較簡單，不再贅述。
![image](https://github.com/Janalexei9/WINSRV_2003_AD_update_to_2022/assets/155059505/fd1cf01b-b6ae-4753-80f9-ab5525e71cf4)

接下來需要將五大角色轉移到2012R2上，以下演示一次：

到2012的`操作主機`點選`RID` `PDC` `基礎結構`，並點選`變更`；到`Active Directory 網域及信任` 一樣進行`變更`。

![image](https://github.com/Janalexei9/WINSRV_2003_AD_update_to_2022/assets/155059505/90d87bb2-9031-40ae-8fc1-0dd0fc8b7ab6)

![image](https://github.com/Janalexei9/WINSRV_2003_AD_update_to_2022/assets/155059505/3ea0d990-1c77-41ca-a1a0-e4095746111b)

![image](https://github.com/Janalexei9/WINSRV_2003_AD_update_to_2022/assets/155059505/3bafaf14-3d29-4c20-825b-644809c4444f)

最後一個比較麻煩，打開`CMD`，輸入`regsvr32 schmmgmt.dll`；電腦搜尋`mmc`，點選`新增/移除嵌入式管理單元`，將`Active Directory 架構`加進去，再來`變更目錄伺服器`，選取2012主機並確定。

![image](https://github.com/Janalexei9/WINSRV_2003_AD_update_to_2022/assets/155059505/0f3b7637-fb99-4524-ba54-d5febbbeb1db)

![image](https://github.com/Janalexei9/WINSRV_2003_AD_update_to_2022/assets/155059505/636dfcf2-7bfc-41ce-9f60-eb95794139fd)

![image](https://github.com/Janalexei9/WINSRV_2003_AD_update_to_2022/assets/155059505/3530d1f1-6b2d-4473-9136-301e03bb760a)

![image](https://github.com/Janalexei9/WINSRV_2003_AD_update_to_2022/assets/155059505/42127a5f-f56f-431f-9166-b7748e1b8a8b)

![image](https://github.com/Janalexei9/WINSRV_2003_AD_update_to_2022/assets/155059505/1633506e-9088-4153-88f6-c11cfb3207ec)

![image](https://github.com/Janalexei9/WINSRV_2003_AD_update_to_2022/assets/155059505/0bb7d6b2-420c-4695-8bff-3cded116813b)

![image](https://github.com/Janalexei9/WINSRV_2003_AD_update_to_2022/assets/155059505/3259bda6-4ffe-4d45-9288-85d0f7c9ab7f)

至此，五大角色轉移完成，在`CMD`輸入`netdom query fsmo` 確認是否轉移成功。接續將舊`DC`移除，`CMD`輸入`DCPROMO`

![image](https://github.com/Janalexei9/WINSRV_2003_AD_update_to_2022/assets/155059505/c1271a19-90bf-4941-b001-b7b17c8dea19)

![image](https://github.com/Janalexei9/WINSRV_2003_AD_update_to_2022/assets/155059505/ae62af99-ab3e-47c8-b0a6-8bd5f9ea5055)


#### （二）、2012移轉至2022

這步驟會出現錯誤，主要跟FRS與DFSR有關。

在這之前，我們先把樹系提升至2012R2，

![image](https://github.com/Janalexei9/WINSRV_2003_AD_update_to_2022/assets/155059505/7982859b-bdc0-47ee-a70f-f944e4ad5d08)

![image](https://github.com/Janalexei9/WINSRV_2003_AD_update_to_2022/assets/155059505/cb6f467f-0ca6-40e7-acbd-811ca76af294)

開啟`CMD`依照以下步驟進行指令的輸入：

1. `dfsrmig /SetGlobalState 1`，等待全域進入`準備完成`。
![image](https://github.com/Janalexei9/WINSRV_2003_AD_update_to_2022/assets/155059505/c92b0e77-29dd-4483-b49c-4a363f1a553b)

2. `dfsrmig /getglobalstate`，顯示`DFSR全域狀態`進入`準備完成`
![image](https://github.com/Janalexei9/WINSRV_2003_AD_update_to_2022/assets/155059505/5a9c8c30-363e-4c47-8e89-be7c3c82b2ae)

3. `dfsrmig /setglobalstate 2`，重新導向複寫。
![image](https://github.com/Janalexei9/WINSRV_2003_AD_update_to_2022/assets/155059505/75964214-edaf-4623-9bb6-0ceab732062d)

4. `dfsrmig /setglobalstate 3`，狀態顯示`已排除`
![image](https://github.com/Janalexei9/WINSRV_2003_AD_update_to_2022/assets/155059505/ad814ec9-1ed1-4f57-9610-66dde4812b54)

5. `dfsrmig /getglobalstate` `dfsrmig /getmigrationstate`，檢查狀態。
![image](https://github.com/Janalexei9/WINSRV_2003_AD_update_to_2022/assets/155059505/002f61ac-407f-429c-a87c-3fbacc10be8c)

接下來將2022升級成DC後加入網域，並轉移五大角色，最後2012 DC退出網域，升級樹系。

![image](https://github.com/Janalexei9/WINSRV_2003_AD_update_to_2022/assets/155059505/1bf06321-bbac-4c42-86f4-4c4eaa1ecb77)

![image](https://github.com/Janalexei9/WINSRV_2003_AD_update_to_2022/assets/155059505/ebb244f8-b305-47b3-8ec4-6c14977a5110)

![image](https://github.com/Janalexei9/WINSRV_2003_AD_update_to_2022/assets/155059505/283a9bf6-f22d-44a4-afe7-c1e085da1d60)


到這裡，2003到2022 AD的升級動作就結束了，樹系最高等級就是2016。
