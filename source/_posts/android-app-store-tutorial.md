---
title: Android app 上架流程
tags:
  - Flutter
  - Android
comments: true
categories:
  - Flutter
abbrlink: 4254a2e9
date: 2020-10-25 19:51:40
---

本篇文章主要介紹在2020年 Goolge 改版後的 Play Store Console 如何上架 Android App

首先到 [Play Store Console](https://play.google.com/console/u/0/developers/) 右邊點擊新增應用程式

![](../images/android-app-deploy/img1.png)

填寫以下應用程式詳細資訊
 
 - 應用程式名稱：會顯示在商店的App名稱
 - 預設語言：預設App使用的語言，作為商店一開始提供的預設語言
 - 應用程式類型：分類只有遊戲與應用程式，目的是要把遊戲區分出來，日後可更改
 - 是否收費：設定App是否下載時需要費用，勾選收費後，需要至 `付費應用程式` 修改設定

最後同意 `開發人員計畫政策` 及 `美國出口法律` 後，即可完成初始設定

![](../images/android-app-deploy/img2.png)

![](../images/android-app-deploy/img3.png)

接著進入到 `資訊主頁`

![](../images/android-app-deploy/img4.png)

第一次設定時會出現 `初始設定` 提示，可依序點擊設定，該步驟都是必須完成的步驟，否則無法完成審查上架

![](../images/android-app-deploy/img5.png)

# 應用程式存取權

設定 App 是否開放給全部使用者

![](../images/android-app-deploy/img6.png)

或是部分功能有使用限制，需設定：
 - 名稱
 - 使用者名稱/電話號碼
 - 密碼
 - 任何操作說明
 
透過此可限制使用者下載應用程式

![](../images/android-app-deploy/img7.png)

![](../images/android-app-deploy/img8.png)

# 廣告

設定App中是否有廣告，若有勾選廣告則會在Play商店上顯示 `含廣告內容` 的標籤

![](../images/android-app-deploy/img10.png)

# 內容分級

根據國際年齡分級聯盟（英語：International Age Rating Coalition，縮寫IARC）設計的簡化各國分級的內容分級問卷，降低產品評比的過程，[參考資料](https://zh.wikipedia.org/wiki/%E5%9C%8B%E9%9A%9B%E5%B9%B4%E9%BD%A1%E5%88%86%E7%B4%9A%E8%81%AF%E7%9B%9F)

填寫內容分級問卷，讓使用者了解App的分類，及是否會有不宜兒童的內容

![](../images/android-app-deploy/img11.png)

首先填寫電子郵件，問卷完成會寄送一份結果至該信箱

![](../images/android-app-deploy/img13.png)

選擇 App 的類別，以 [文藻校務通](https://play.google.com/store/apps/details?id=com.wtuc.ap) 為例，選擇 `參考資訊、新聞或教育內容`

![](../images/android-app-deploy/img12.png)

根據選擇的類型，需填寫相關內容是否有未成年暴力色情訊息

![](../images/android-app-deploy/img14.png)

完成後會顯示基本的報表，點擊右下角提交完成問卷

![](../images/android-app-deploy/img15.png)

完成後，若日後想在修改問卷內容，需至右上角 點擊 `Start new questionaire` 重新提交問卷

![](../images/android-app-deploy/img16.png)

# 目標對象

填寫詳細的App發布目標對象

首先點選 `目標年齡層`，主要是確認目標對象是不是兒童，若對象未滿13歲則需要新增隱私權政策

![](../images/android-app-deploy/img18.png)

勾選 `是否會引起兒童興趣`，若上者點擊13歲以下，可勾選 `是` 宣稱適合兒童

![](../images/android-app-deploy/img19.png)

最後點擊 `儲存` 完成目標對象和內容

![](../images/android-app-deploy/img20.png)

接著就完成一半了～～

![](../images/android-app-deploy/img21.png)

# 應用程式類別及詳細資料

應用程式分類，區分應用程式或遊戲的類別，在Play 商店中，也會有類別排名

![](../images/android-app-deploy/img22.png)

接著設定商店的聯絡詳細資訊，分別為

 - 電子郵件地址 (必填)
 - 電話號碼
 - 網站

**以上訊息皆會在Play商店上顯示**

![](../images/android-app-deploy/img23.png)

並勾選是否要在Play商店外行銷，讓外部網站可搜尋到你的App

![](../images/android-app-deploy/img24.png)

完成後就只剩下最後一個步驟

![](../images/android-app-deploy/img25.png)

# 商店資訊

首先會根據一開始設定的`主要語言`，設定 App 在 Play 商店的資訊，可根據不同語言，設定不同的商店資訊，可點擊 `管理其他語言版本的翻譯內容` 管理其他語言的內容

## 應用程式詳細資料

首先可設定
 - 應用程式名稱：App名稱，作為可供搜尋的關鍵字，上限50字
 - 簡短說明：可在App頁面首要看到簡短說明，上限80字
 - 完整說明：在點擊`關於這個應用程式`後顯示的完整說明

![](../images/android-app-deploy/img26.png)

可參照 Play 商店對應位置

![](../images/android-app-deploy/img26-1.jpg)

![](../images/android-app-deploy/img26-2.jpg)

接著設定 `應用程式圖示` 會顯示在 Play 商店的圖示，**限定尺寸為 `512*512` 的解析度**，上傳後都會以橢圓裁剪顯示

![](../images/android-app-deploy/img27.png)

主要圖片顯示於商店資訊的最頂端，可用於宣傳應用程式，**大小限制 `1024*500` 解析度的圖片**

![](../images/android-app-deploy/img29.png)

螢幕截圖主要分為

 - 手機
 - 七吋平板電腦
 - 十吋平板電腦

基本上，對應類型的裝置截圖都適用，也可自行製作符合規定的尺寸的圖片，**皆為使用 JPEG 或 24 位元 PNG 圖片，長寬比建議16:9**

![](../images/android-app-deploy/img30.png)

![](../images/android-app-deploy/img31.png)

加入影片也會顯示於商店上

![](../images/android-app-deploy/img32.png)

商店資訊設定完成後，若未來要修改都可直接修改，但修改後都需要等商店部署時間，通常都會為半天左右時間

# 上傳App至商店

到 `發佈` 的目錄下，選擇App目前要發布的方式，有分成

- 正式版：會發布給商店中所有設置的地區
- 公開測試：任何使用者可至 Play 商店點擊測試計畫，即可使用此版本
- 封閉測試：由開發人員建立電子郵件清單，或是可透過連結加入測試計畫 https://play.google.com/apps/testing/{app id}
- 內部測試：由開發人員建立電子郵件清單，或是可透過內部邀請測試連結：https://play.google.com/apps/internaltest/{test group id} 加入
- 搶先註冊：若還沒發佈正式版時，可利用此功能，在Play商店中顯示搶先體驗的字樣，並提供測試人員特殊獎勵

不管利用哪種測試方式，接下來上傳App的方式都會相同，例如選擇 `正式版` 發布，並點擊右上方的 `建立新版本`

![](../images/android-app-deploy/img33.png)

接著第一次上架時需點擊 同意使用 `Google Play 應用程式簽署`，Google 會管理你簽署所使用的金鑰，並且該金鑰只能提供給該 App 使用

若今天金鑰遺失，可請帳戶擁有者聯絡[支援小組](https://support.google.com/googleplay/android-developer/contact/otherbugs)重新上傳金鑰

將利用 `Android Studio` 等等的 Android 編譯工具，將原生Android 的 `Apk` 或是 `App Bundle` 上傳至此頁面

**每次新上傳的 `版本號碼(version code)` 皆需大於先前上傳的**

關於金鑰使用詳細 [可參考](https://support.google.com/googleplay/android-developer/answer/7384423)

![](../images/android-app-deploy/img35.png)

## 版本詳細資訊

版本名稱會根據上傳的 `Apk` 或 `App Bundle` 命名

版本資訊會根據商店可提供的語言，以 `XML` 格式撰寫，將這次更新內容寫至 `語言碼(language code)` 中

![](../images/android-app-deploy/img36.png)

完成後點擊儲存，並點擊檢查版本

![](../images/android-app-deploy/img37.png)

接著會發現沒有設定提供地區

![](../images/android-app-deploy/img38.png)

返回至上一頁的最上方，選擇 `國家與地區` 編輯針對正式版的發布國家/地區

![](../images/android-app-deploy/img39.png)

若沒勾選，Play商店就不會發佈至此國家/地區

![](../images/android-app-deploy/img40.png)

接著回到剛剛編輯的版本資訊，點擊 `開始發布(正式版)`

![](../images/android-app-deploy/img41.png)

最後會跳回正式版的頁面，並顯示審查中

![](../images/android-app-deploy/img42.png)

自 2019 年開始，Play 商店在第一次審查時，最久大約會至七天，爾後提交大約都是一下子就完成審查，並都是半天會完全部署至商店(所有使用者都可以看到更新)

