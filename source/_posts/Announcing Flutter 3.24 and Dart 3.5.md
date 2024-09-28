---
title: 【文章翻譯】Announcing Flutter 3.24 and Dart 3.5
tags:
  - Flutter
abbrlink: 6243ded4
date: 2024-08-07 15:23:41
---
【文章內容使用 Gemini 1.5 Pro 自動產生】

## 宣布 Flutter 3.24 和 Dart 3.5

今天，我們將揭曉 [Flutter 3.24](https://medium.com/p/6c040f87d1e4/edit) 和 [Dart 3.5](https://medium.com/dartlang/dart-3.5-6ca36259fa2f)，以及 [I/O 2024 Connect 系列](https://ioconnectchina.googlecnapps.cn/) 的最後一站，該系列將在幾個小時內在中國舉行——中國是世界上最活躍的 Flutter 社群之一，這使得這個時刻變得格外特別。

<figure>
<img alt="Dash flies in and lands to the left of an outline of 3.24. She presses a button that fills the 3.24 in with Flutter blue." src="https://cdn-images-1.medium.com/max/1024/1*jzRGig761LnPlvokq2FaVA.gif" />
</figure>

我們在 5 月份的 [Google I/O](https://io.google/2024/) 上啟動了一系列 [令人興奮的更新](https://medium.com/flutter/io24-5e211f708a37)，包括將 WebAssembly 編譯支援升級到 stable channel，改進 Impeller，以及預覽 Dart macros 的未來。

Flutter 3.24 和 Dart 3.5 發佈版本在我們幫助您製作出色的高效能應用程式（這些應用程式可以透過單個共用程式碼庫到達行動、網頁和桌面上的使用者）的使命之上。它們包括對新的 Flutter GPU API 的早期預覽，對網頁上元素嵌入的改進，以及幾個針對那些想要為 iOS 生態系統構建的開發人員的令人興奮的更新，包括對 Swift Package Manager 的早期支援，以及對 Cupertino Widgets 的功能更新。

讓我們開始吧！

### Impeller：為跨平台圖形效能樹立新標竿

從歷史上看，跨平台框架需要在視覺效果上做出妥協，因為它們依賴於底層平台提供的更高級別的抽象。Flutter 採用了不同的方法，它擁有自己的渲染層，可以在每台設備上提供硬體加速的圖形和流暢的效能。我們在 [Impeller](https://docs.flutter.dev/perf/impeller) 和 [著色器](https://docs.flutter.dev/ui/design/graphics/fragment-shaders) 方面取得了重大進展，為圖形（如 3D）解鎖了令人興奮的新可能性。

我們很興奮地分享全新的 [Flutter GPU API](https://github.com/flutter/engine/blob/main/docs/impeller/Flutter-GPU.md) 的早期預覽，這是一個功能強大、低階的圖形 API，直接整合到 Flutter SDK 中。API 允許您定義自訂柵格管道，並將繪製調用直接提交到 GPU，從而可以建立專用的渲染器，例如 2D Canvas 的替代方案、3D 場景圖，甚至用於創造視覺上令人驚豔、高效能和沉浸式體驗的粒子系統，而無需使用通常引擎級別需要的容量。

<figure>
<img alt="" src="https://cdn-images-1.medium.com/max/796/0*QC1D0LdTgLynDOnV" />
<figcaption>3D animation of a sci-fi space helmet rendered in flutter_scene.</figcaption>
</figure>

考慮到 API 的低階性，我們預計對於那些沒有大量圖形開發經驗的開發人員來說，會有一段學習曲線。這就是為什麼我們正在投資渲染套件，例如新的 `flutter_scene` 套件，它利用 Flutter GPU API 允許匯入動畫 glTF 模型並構造 3D 場景，使您能夠輕鬆地在 Flutter 和 Dart 中建立 3D 應用程式和遊戲，如下所示。

<iframe src="https://cdn.embedly.com/widgets/media.html?src=https%3A%2F%2Fwww.youtube.com%2Fembed%2FY-DFVKPikVM%3Ffeature%3Doembed&amp;display_name=YouTube&amp;url=https%3A%2F%2Fwww.youtube.com%2Fwatch%3Fv%3DY-DFVKPikVM&amp;image=https%3A%2F%2Fi.ytimg.com%2Fvi%2FY-DFVKPikVM%2Fhqdefault.jpg&amp;key=a19fcc184b9711e1b4764040d3dc5c07&amp;type=text%2Fhtml&amp;schema=youtube" width="640" height="480" frameborder="0" scrolling="no"><a href="https://medium.com/media/21e51abf698b844782de55d81e3cd7b4/href">https://medium.com/media/21e51abf698b844782de55d81e3cd7b4/href</a></iframe>

儘管 Flutter GPU API 提供了令人興奮的可能性，但它仍然處於早期預覽階段，我們可能會對 API 進行重大變更。我們建議在使用 Flutter GPU 時針對 Flutter 的主頻道進行開發。在部落格文章 [Introducing Flutter GPU & Flutter Scene](https://medium.com/flutter/getting-started-with-flutter-gpu-f33d497b7c11) 中了解更多資訊。

### 為 iOS 和 macOS 打造 Flutter：讓為 Apple 生態系統交付美麗、快速的應用程式變得更加容易

我們的目標是讓您能夠構建出色的應用程式，這些應用程式感覺起來像原生應用程式，並能完美地執行。這項工作的一部分是優化效能，以及最大限度地提高 Flutter 與底層平台的相容性，包括存取 Apple 生態系統的全部功能。

在此版本中，我們引入了對 Swift Package Manager 的早期支援，解鎖了對蓬勃發展的 Swift 套件生態系統的存取，並使 Flutter 外掛能夠利用大量預先構建的功能來加速開發。一旦 Swift Package Manager (SPM) 被 Plugin 開發人員廣泛採用，它應該會簡化 Flutter 安裝過程本身，並降低新手的入門門檻，特別是那些不熟悉 iOS 生態系統的人。我們鼓勵 Plugin 作者 [嘗試將 SPM 支援添加到您的外掛中](https://docs.flutter.dev/packages-and-plugins/swift-package-manager/for-plugin-authors#how-to-add-swift-package-manager-support-to-an-existing-flutter-plugin)，並提供您體驗的 [回饋](https://github.com/flutter/flutter/issues)。

接下來，我們希望讓您始終能夠滿足設計師的要求，並在 iOS 上提供高保真度的體驗。為了實現這一點，我們開始著手現代化和擴展 Cupertino Widget，解決了 [Cupertino 中的 15 個問題](https://github.com/flutter/flutter/issues?q=is%3Aissue+is%3Aclosed+label%3A%22f%3A+cupertino%22+sort%3Aupdated-desc+closed%3A2024-04-01..2024-07-01+)，並在 [Widget 目錄](https://docs.flutter.dev/ui/widgets/cupertino) 中加入了 37 個缺少的 Cupertino Widget。

最後，我們為 Flutter macOS 應用程式添加了 [`platform_view`](https://docs.flutter.dev/platform-integration/macos/platform-views) 和 [webview](https://docs.flutter.dev/platform-integration/web/web-content-in-flutter) 支援，允許將原生 macOS UI 組件無縫整合到您的 Flutter 應用程式中，以提供更完整、更完善的使用者體驗。

展望未來，我們很興奮能更多地投資於其他 Cupertino Widget 的保真度，與我們的生態系統一起推出 Swift Package Manager，並提供其他調查，讓整合和與 Apple 平台的互操作變得更加容易。

### 強調充滿活力的 Flutter 社群的全球影響力

我們還想感謝社群的貢獻，包括您的貢獻！這組版本包含來自 167 多位獨特貢獻者的近 1,500 次提交，其中包括 49 位 *全新* 貢獻者。我們深受 Flutter 社群持續的高水準活動、承諾和增長所鼓舞，包括積極構建框架的那些人。謝謝您！

我們共同努力的影響力正在世界各地展現出來，創造出數百萬人每天使用的令人難以置信的應用程式和體驗。例如，以下是 [案例研究](http://flutter.dev/showcase/xiaomi) 的搶先看，展示了中國科技公司小米的團隊如何以及為什麼使用 Flutter 為該公司備受歡迎的新型電動汽車 [小米 SU7](https://www.mi.com/global/discover/article?id=3263&amp;ref=renatomitra.com) 開發一個配套應用程式。

<iframe src="https://cdn.embedly.com/widgets/media.html?src=https%3A%2F%2Fwww.youtube.com%2Fembed%2FwfD7ZQhwACU%3Ffeature%3Doembed&amp;display_name=YouTube&amp;url=https%3A%2F%2Fwww.youtube.com%2Fwatch%3Fv%3DwfD7ZQhwACU&amp;image=https%3A%2F%2Fi.ytimg.com%2Fvi%2FwfD7ZQhwACU%2Fhqdefault.jpg&amp;key=a19fcc184b9711e1b4764040d3dc5c07&amp;type=text%2Fhtml&amp;schema=youtube" width="854" height="480" frameborder="0" scrolling="no"><a href="https://medium.com/media/1e141755d6ab1cd3b7962281efd5e6d3/href">https://medium.com/media/1e141755d6ab1cd3b7962281efd5e6d3/href</a></iframe>

在世界各地出現的許多其他令人興奮的 Flutter 應用程式範例：

* [SNCF Connect](http://flutter.dev/showcase/sncf-connect)，法國鐵路和歐洲最大的 Flutter 應用程式（擁有超過 150 個螢幕）的擁有者，與奧運會合作為 Flutter 應用程式交付了許多更新，使數百萬遊客能夠在奧運會期間穿梭於法國各地。
* [Wolt](http://flutter.dev/showcase/wolt)，DoorDash 國際的一部分，使用 Flutter 擴展到商家零售市場。
* [惠而浦](http://flutter.dev/showcase/whirlpool)，一家擁有全球影響力的《財富》500 強公司，正在使用 Flutter 在巴西探索新的銷售管道。
* [Monta](http://flutter.dev/showcase/monta)，一家丹麥的電動汽車充電生態系統初創公司，在短短 3 個月內使用 Flutter 將其第一個行動應用程式推向市場，後來又成功地將其 Web 應用程式移植到 Flutter。

### 總結

以上只是這些版本中 Flutter 和 Dart 的許多新功能和更新中的一小部分，您可以在 [Flutter 3.24 技術部落格](https://medium.com/p/6c040f87d1e4/edit) 文章和 [Dart 3.5 部落格文章](https://medium.com/dartlang/dart-3.5-6ca36259fa2f) 中了解更多資訊。

展望未來，我們對 Flutter 的未來充滿期待。我們仍然致力於我們的使命，並且感謝您——無論是貢獻者、社群成員還是 Flutter 開發人員——成為這段非凡旅程的一部分。我們迫不及待想看看您接下來會建立什麼！

<img src="https://medium.com/_/stat?event=post.clientViewed&referrerSource=full_rss&postId=204b7d20c45d" width="1" height="1" alt=""><hr><p><a href="https://medium.com/flutter/flutter-3-24-dart-3-5-204b7d20c45d">宣布 Flutter 3.24 和 Dart 3.5</a> 最初發佈在 <a href="https://medium.com/flutter">Flutter</a> 上的 Medium，人們在那裡透過突出顯示和回應這個故事來繼續討論。</p> 