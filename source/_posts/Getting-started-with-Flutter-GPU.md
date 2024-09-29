---
title: 【文章翻譯】Getting started with Flutter GPU
tags:
  - Flutter
  - GPU
comments: true
categories:
  - Flutter
abbrlink: 8ddf18fb
date: 2024-08-07 02:02:39
---
【文章內容使用 Gemini 1.5 Pro 自動產生】

## 在 Flutter 中建立自訂渲染器和渲染 3D 場景

Flutter 3.24 版本推出了一個新的低階圖形 API，稱為 [Flutter GPU](https://github.com/flutter/engine/blob/main/docs/impeller/Flutter-GPU.md)。還有一個由 Flutter GPU 提供支援的 3D 渲染函式庫，稱為 [Flutter Scene](https://pub.dev/packages/flutter_scene)（套件：`flutter_scene`）。Flutter GPU 和 Flutter Scene 目前都處於預覽階段，僅在 Flutter 的 [main channel](https://docs.flutter.dev/release/upgrade#other-channels) 上提供（因為依賴於實驗性功能），需要 [啟用 Impeller](https://docs.flutter.dev/perf/impeller#availability)，並且可能會偶爾推出重大變更。

本文包含這兩個套件的兩個「入門」指南：

1. 🔺 **進階：** [開始使用 Flutter GPU](#d558)<br>如果您是一位經驗豐富的圖形程式設計師，或者您對低階圖形感興趣，並想要從頭開始在 Flutter 中建立渲染器，那麼本指南將幫助您開始使用 Flutter GPU。您將從頭開始繪製第一個三角形……在 Flutter 中！
2. 💚 **中級：** [使用 Flutter Scene 進行 3D 渲染](#6b35)<strong><br></strong>如果您是一位想要將 3D 功能加入到應用程式中的 Flutter 開發人員，或者您想要使用 Dart 和 Flutter 建立 3D 遊戲，那麼本指南適合您！您將設定一個專案，在 Flutter 中匯入並渲染 3D 資產。

### 開始使用 Flutter GPU

⚠️ 警告！⚠️ Flutter GPU 最終是一個低階 API。絕大多數將從 Flutter GPU 的存在中受益的 Flutter 開發人員，很有可能透過使用在 pub.dev 上發佈的更高級別渲染函式庫來實現，例如 Flutter Scene 渲染套件。如果您對 Flutter GPU API 本身不感興趣，只對 3D 渲染感興趣，請跳轉到 [使用 Flutter Scene 進行 3D 渲染](#6b35)。

<figure>
<img alt="" src="https://cdn-images-1.medium.com/max/900/0*hAqIOVkaI1IWnOHE" />
<figcaption>閃閃發光。這是一個射線行進符號距離場。您可以使用 Flutter GPU 渲染它，但也可以使用 [自訂片段著色器](https://docs.flutter.dev/ui/design/graphics/fragment-shaders) 來實現。</figcaption>
</figure>

### 開始使用 Flutter GPU

Flutter GPU 是 Flutter 內建的低階圖形 API。它允許您透過撰寫 Dart 程式碼和 GLSL 著色器在 Flutter 中建立和整合自訂渲染器。無需原生平台程式碼。

目前，Flutter GPU 處於早期預覽階段，並提供基本的柵格化 API，但是隨著 API 接近穩定狀態，將繼續加入和改進更多功能。

Flutter GPU 還需要 [啟用 Impeller](https://docs.flutter.dev/perf/impeller#availability)。這意味著它只能在 Impeller 支援的平台上使用。在撰寫本文時，Impeller 支援：

* iOS（預設啟用）
* macOS（預覽版本選擇性開啟）
* Android（預覽版本選擇性開啟）

我們在 Flutter GPU 中的目標是最終支援所有 Flutter 的平台目標。最終目標是在 Flutter 中促進一個跨平台渲染解決方案的生態系統，這些方案對於套件作者來說易於維護，對於使用者來說易於安裝。

3D 渲染只是一個可能的使用案例。Flutter GPU 還可以被用來建立專用的 2D 渲染器，或者做一些更非正統的事情，例如渲染 4D 空間的 3D 切片，或投影非歐幾里得空間。

由 Flutter GPU 支援的自訂 2D 渲染器的一個很好的使用案例範例是依賴骨骼網格變形的 2D 角色動畫格式。Spine 2D 就是一個很好的範例。此類骨骼網格解決方案通常具有動畫片段，這些片段會操縱層級結構中骨骼的平移、旋轉和縮放屬性，並且每個頂點都有一些相關的「骨骼權重」，這些權重決定哪些骨骼應該影響頂點以及影響程度。

使用像 `drawVertices` 這樣的 Canvas 解決方案，需要在 CPU 上為每個頂點應用骨骼權重轉換。使用 Flutter GPU，骨骼轉換可以以統一陣列或甚至紋理採樣器的形式傳遞到頂點著色器，允許根據骨骼狀態和每個頂點骨骼權重在 GPU 上並行計算每個頂點的最終位置。

言歸正傳，讓我們透過一個溫和的介紹開始使用 Flutter GPU：繪製第一個三角形！

<figure>
<img alt="" src="https://cdn-images-1.medium.com/max/1024/0*JEI3fLDGcRHWKruT" />
</figure>

### 將 Flutter GPU 加入您的專案

首先，請注意 Flutter GPU 目前處於早期預覽狀態，可能會容易出現 API 故障。目前的 API 已經可以實現很多功能，但經驗豐富的圖形工程師可能會注意到一些缺少的常用功能。Flutter GPU 在接下來的幾個月中將會增加許多功能。

基於這些原因，強烈建議您在針對 Flutter GPU 開發套件時，目前使用 [main channel](https://docs.flutter.dev/release/upgrade#other-channels) 的最新版本進行操作。如果您遇到任何意外行為、錯誤或有功能請求，請使用標準的 [Flutter 議題範本](https://github.com/flutter/flutter/issues/new/choose) 在 GitHub 上提交議題。與 Flutter GPU 相關的所有追蹤議題都被貼上了 [flutter-gpu 標籤](https://github.com/flutter/flutter/labels/flutter-gpu)。

因此，在嘗試使用 Flutter GPU 之前，請透過執行以下命令將 Flutter 切換到 main channel。

```
flutter channel main
flutter upgrade
```

現在建立一個新的 Flutter 專案。

```
flutter create my_cool_renderer
cd my_cool_renderer
```

接下來，將 flutter_gpu SDK 套件加入到您的 pubspec 中。

```
flutter pub add flutter_gpu --sdk=flutter
```

### 建立和匯入著色器組合包。

為了使用 Flutter GPU 渲染任何內容，您需要撰寫一些 GLSL 著色器。Flutter GPU 的著色器與 Flutter 的 [片段著色器](https://docs.flutter.dev/ui/design/graphics/fragment-shaders) 功能使用的著色器具有不同的語義，特別是在統一綁定方面。您還將定義一個頂點著色器來與片段著色器一起使用。

從定義最簡單的著色器開始。您可以將著色器放置在專案中的任何位置，但在此範例中，建立一個 `shaders` 目錄，並用兩個著色器填充它：`simple.vert` 和 `simple.frag`。

```
// 複製到：shaders/simple.vert

in vec2 position;

void main() {
 gl_Position = vec4(position, 0.0, 1.0);
}
```

在繪製三角形時，您將擁有一個定義每個頂點的資料列表。在本例中，它僅列出 2D 位置。對於這些頂點中的每一個，一個簡單的頂點著色器將這些 2D 位置分配給剪輯空間輸出內在 `gl_Position`。

```
// 複製到：shaders/simple.frag

out vec4 frag_color;

void main() {
 frag_color = vec4(0, 1, 0, 1);
}
```

片段著色器更簡單；它輸出一個 RGBA 顏色，範圍為 (0, 0, 0, 0) 到 (1, 1, 1, 1)。因此，所有內容都將被渲染為綠色。

好的，現在您已經有了著色器，請使用 Flutter 的預先編譯 (AOT) 著色器編譯器編譯它們。若要設定著色器組合包的自動建立，我們建議您使用 [flutter_gpu_shaders](https://pub.dev/packages/flutter_gpu_shaders) 套件。

使用 pub 將 flutter_gpu_shaders 作為專案中的普通相依加入。

```
flutter pub add flutter_gpu_shaders
```

Flutter GPU 著色器被組合到 `.shaderbundle` 檔案中，這些檔案可以作為普通資產加入到專案的資產組合包中。著色器組合包包含針對平台目標編譯的著色器原始碼。

接下來，建立一個著色器組合包清單檔案，以描述著色器組合包的內容。將以下內容加入到專案根目錄中的 `my_renderer.shaderbundle.json`。

```
{
 "SimpleVertex": {
 "type": "vertex",
 "file": "shaders/simple.vert"
 },
 "SimpleFragment": {
 "type": "fragment",
 "file": "shaders/simple.frag"
 }
}
```

著色器組合包中的每個條目都可以具有任意的名稱。就這個例子來說，名稱為「SimpleVertex」和「SimpleFragment」。這些名稱用於在您的應用程式中查找著色器。

接下來，使用 `flutter_gpu_shaders` 套件建立 shaderbundle。您可以透過啟用實驗性的「原生資產」功能來加入一個自動觸發建立的鉤子。使用以下命令來啟用原生資產並安裝 `native_assets_cli` 套件。

```
flutter config --enable-native-assets
flutter pub add native_assets_cli
```

在啟用原生資產功能後，在 `hook` 目錄下加入一個 `build.dart` 腳本，該腳本將自動觸發建立著色器組合包。

```
// 複製到：hook/build.dart

import 'package:native_assets_cli/native_assets_cli.dart';
import 'package:flutter_gpu_shaders/build.dart';

void main(List<String> args) async {
 await build(args, (config, output) async {
 await buildShaderBundleJson(
 buildConfig: config,
 buildOutput: output,
 manifestFileName: 'my_renderer.shaderbundle.json');
 });
}
```

進行此更改後，當 Flutter 工具建立專案時，`buildShaderBundleJson` 將建立著色器組合包，並將結果輸出到套件根目錄下的 `build/shaderbundles/my_renderer.shaderbundle`。

著色器組合包格式本身與您使用的 Flutter 特定版本相關聯，並且可能會隨著時間推移而發生變化。如果您正在編寫建立著色器組合包的套件，請不要將產生的 `.shaderbundle` 檔案檢查到您的原始碼樹中。相反，請使用建立鉤子來自動化建立過程（如前所述）。

透過這種方式，使用您函式庫的開發人員將始終使用正確格式建立新的著色器組合包！

現在您已經自動建立了著色器組合包，請像普通資產一樣匯入它。將一個資產條目加入到專案的 `pubspec.yaml`：

```
flutter:
 assets:
 - build/shaderbundles/
```

未來，原生資產功能將允許建立鉤子將資料資產附加到組合包中。一旦發生這種情況，將不再需要在建立鉤子旁邊加入資產匯入規則。

接下來，加入一些程式碼以在執行時期時載入著色器。建立 `lib/shaders.dart` 並加入以下程式碼。

```
// 複製到：lib/shaders.dart

import 'package:flutter_gpu/gpu.dart' as gpu;

const String _kShaderBundlePath =
 'build/shaderbundles/my_renderer.shaderbundle';
// NOTE: 如果您正在建立一個函式庫，路徑必須以套件名稱為前綴
// 例如：
// 'packages/my_cool_renderer/build/shaderbundles/my_renderer.shaderbundle'

gpu.ShaderLibrary? _shaderLibrary;
gpu.ShaderLibrary get shaderLibrary {
 if (_shaderLibrary != null) {
 return _shaderLibrary!;
 }
 _shaderLibrary = gpu.ShaderLibrary.fromAsset(_kShaderBundlePath);
 if (_shaderLibrary != null) {
 return _shaderLibrary!;
 }

 throw Exception("Failed to load shader bundle! ($_kShaderBundlePath)");
}
```

此程式碼為 Flutter GPU 著色器執行時期時函式庫建立了一個單例 getter。第一次存取 `shaderLibrary` 時，將使用 `gpu.ShaderLibrary.fromAsset(shader_bundle_path)` 使用已建立的資產組合包初始化執行時期著色器函式庫。

現在專案已經設定好可以使用 Flutter GPU 著色器了。是時候渲染那個三角形了！

### 繪製第一個三角形

在本指南中，您將建立一個 RGBA Flutter GPU `Texture` 和一個將紋理作為顏色輸出附加的 `RenderPass`。然後，您將使用 [`Canvas.drawImage`](https://api.flutter.dev/flutter/dart-ui/Canvas/drawImage.html) 在一個 Widget 中渲染紋理。

為了簡潔起見，您將放棄最佳實務，只是在每個畫面中重新建立所有資源。

只要在配置時將您的 `Texture` 標記為「著色器可讀」，您就可以將其轉換為 `dart:ui.Image`。若要在 Widget 樹中顯示渲染結果，請將其繪製到 `dart:ui.Canvas` 上！

您可以透過使用自訂繪製器來為 Widget 樹搭建腳手架來存取 `Canvas`。將 `lib/main.dart` 的替換為以下內容：

```
import 'dart:typed_data';

import 'package:flutter/material.dart';
import 'package:flutter_gpu/gpu.dart' as gpu;

// 注意: 我們之前在設定著色器組合包匯入時已經建立了這個！
import 'shaders.dart';

void main() {
 runApp(const MyApp());
}

class MyApp extends StatelessWidget {
 const MyApp({super.key});

 @override
 Widget build(BuildContext context) {
 return MaterialApp(
 title: 'Flutter GPU Triangle Example',
 home: CustomPaint(
 painter: TrianglePainter(),
 ),
 );
 }
}

class TrianglePainter extends CustomPainter {
 @override
 void paint(Canvas canvas, Size size) {
 // 嘗試存取 `gpu.gpuContext`。
 // 如果 Flutter GPU 不受支援，將會拋出例外。
 print('Default color format: ' +
 gpu.gpuContext.defaultColorFormat.toString());
 }

 @override
 bool shouldRepaint(covariant CustomPainter oldDelegate) => true;
}
```

現在執行應用程式。提醒一下，Flutter GPU 目前需要 [啟用 Impeller](https://docs.flutter.dev/perf/impeller#availability)。因此，您必須使用 Impeller 支援的平台。對於本指南，我將以 macOS 為目標。

```
flutter run -d macos --enable-impeller
```

<figure>
<img alt="" src="https://cdn-images-1.medium.com/max/1024/0*lKTtaX2ih6dFpSMQ" />
</figure>

如果 Flutter GPU 正在工作，那麼您應該看到預設顏色格式列印到主控台中。

```
flutter: Default color format: PixelFormat.b8g8r8a8UNormInt
```

如果 Impeller 未啟用，則在嘗試存取 `gpu.gpuContext` 時會拋出例外。

```
Exception: Flutter GPU requires the Impeller rendering backend to be enabled.

The relevant error-causing widget was:
 CustomPaint
```

為了簡潔起見，您只會從這裡開始修改 `paint` 方法。

首先，建立一個 Flutter GPU `Texture`，清除它，然後透過將其繪製到 `Canvas` 上來顯示它。

建立一個與 `Canvas` 大小相同的 `Texture`。必須選擇一個 `StorageMode`。在本例中，您將 `Texture` 標記為 `devicePrivate`，因為您只會使用從裝置	（GPU）存取紋理記憶體的指令。

```
final texture = gpu.gpuContext.createTexture(gpu.StorageMode.devicePrivate,
 size.width.toInt(), size.height.toInt())!;
```

如果透過從主機（CPU）上傳來覆寫紋理的資料，則使用 `StorageMode.hostVisible`。

第三個可用的選項是 `StorageMode.deviceTransient`，它對於不需要超過單個 `RenderPass` 生命週期的附件很有用（因此它們可以只存在於磁磚記憶體中，而不需要由 VRAM 分配支援）。通常，深度/模板紋理符合此條件。

接下來，定義一個 `RenderTarget`。渲染目標包含一個「附件」集合，這些附件描述每個片段的記憶體佈局及其在 `RenderPass` 開始和結束時的設定/拆卸行為。

本質上，`RenderTarget` 是 `RenderPass` 的一個可重複使用的描述符。

現在，定義一個只包含一個顏色附件的非常簡單的 `RenderTarget`。

```
final renderTarget = gpu.RenderTarget.singleColor(
 gpu.ColorAttachment(texture: texture, clearValue: Colors.lightBlue));
```

請注意，此程式碼將 `clearValue` 設定為淡藍色。每個附件都有一個 `LoadAction` 和一個 `StoreAction`，它們分別確定在傳遞開始和結束時應該對附件的瞬時磁磚記憶體執行什麼操作。

預設情況下，顏色附件被設定為 `LoadAction.clear`（它會將磁磚記憶體初始化為給定的顏色），以及 `StoreAction.store`（它會將結果保存到附加紋理的 VRAM 分配中）。

現在，建立一個 `CommandBuffer`，從中使用之前建立的 `RenderTarget` 產生一個 `RenderPass`，然後立即提交 `CommandBuffer` 以清除紋理。

```
final commandBuffer = gpu.gpuContext.createCommandBuffer();
final renderPass = commandBuffer.createRenderPass(renderTarget);
// ... 繪製呼叫將放在這裡！
commandBuffer.submit();
```

剩下的就是將已初始化的紋理繪製到 Canvas 上！

```
final image = texture.asImage();
canvas.drawImage(image, Offset.zero, Paint());
```

<figure>
<img alt="" src="https://cdn-images-1.medium.com/max/1024/0*ebUDtzQOuIGmdlop" />
</figure>

現在您已經建立了一個連接到螢幕上顯示結果的 `RenderPass`，您就可以開始繪製三角形了。若要執行此操作，請設定以下內容：

1. 從我們的著色器建立的 `RenderPipeline`，以及
2. 包含我們幾何形狀的 GPU 可存取緩衝區（三個頂點位置）。

建立 `RenderPipeline` 很容易。您只需要將函式庫中的頂點和片段著色器組合在一起。

```
final vert = shaderLibrary['SimpleVertex']!;
final frag = shaderLibrary['SimpleFragment']!;
final pipeline = gpu.gpuContext.createRenderPipeline(vert, frag);
```

現在是幾何形狀。回想一下，「SimpleVertex」著色器只有一個輸入：`in vec2 position`。因此，若要繪製三個頂點，您需要三組兩個浮點數。

```
final vertices = Float32List.fromList([
 -0.5, -0.5, // 第一個頂點
 0.5, -0.5, // 第二個頂點
 0.0, 0.5, // 第三個頂點
]);
final verticesDeviceBuffer = gpu.gpuContext
 .createDeviceBufferWithCopy(ByteData.sublistView(vertices))!;
```

剩下的就是綁定新的資源並呼叫 `renderPass.draw()` 來完成記錄繪製呼叫。

```
renderPass.bindPipeline(pipeline);

final verticesView = gpu.BufferView(
 verticesDeviceBuffer,
 offsetInBytes: 0,
 lengthInBytes: verticesDeviceBuffer.sizeInBytes,
);
renderPass.bindVertexBuffer(verticesView, 3);

renderPass.draw();
```

如果您啟動應用程式，您現在應該看到一個綠色的三角形！

<figure>
<img alt="" src="https://cdn-images-1.medium.com/max/1024/0*LWnGU5WPT_Eom0wJ" />
</figure>

太棒了，您使用 Flutter、Dart 和一些 GLSL 從頭開始建立了一個渲染器！

無論這是您第一次渲染三角形，還是您是一位經驗豐富的圖形專家，我希望您能繼續使用 Flutter GPU，並查看我們正在開發的套件，例如 Flutter Scene。

未來，我們希望發佈友好的初學者程式碼實驗室，深入探討 Flutter GPU 的預設行為和最佳實務。我們還沒有討論頂點屬性佈局、紋理綁定、統一和對齊需求、管道混合、深度和模板附件、透視校正等等！

在此之前，我建議您探索 [Flutter Scene](https://github.com/bdero/flutter_scene)，作為如何使用 Flutter GPU 的更全面的範例。

### 使用 Flutter Scene 進行 3D 渲染

Flutter Scene（套件 flutter_scene）是一個新的由 Flutter GPU 提供支援的 3D 場景圖套件，它使 Flutter 開發人員能夠匯入動畫 glTF 模型並渲染即時 3D 場景。

其目的是提供一個套件，讓使用 Flutter 建立互動式 3D 應用程式和遊戲變得容易。

<figure>
<img alt="" src="https://cdn-images-1.medium.com/max/1024/0*tC68CbPLef2rJp1e" />
</figure>

該套件最初是一個為用 C++ 編寫的 3D 渲染器編寫的 `dart:ui` 擴展，並直接建立在 Flutter 的原生執行時中，但它已針對 Flutter GPU 重新編寫，具有更靈活的介面。

與 Flutter GPU API 本身一樣，Flutter Scene 目前處於早期預覽狀態，需要 [啟用 Impeller](https://docs.flutter.dev/perf/impeller#availability)。Flutter Scene 通常會跟上 Flutter GPU API 的重大變更，因此強烈建議您在嘗試使用 Flutter Scene 時使用 [main channel](https://docs.flutter.dev/release/upgrade#other-channels)。

接下來，使用 Flutter Scene 建立一個應用程式！

### 設定 Flutter Scene 專案

由於強烈建議您在 [main channel](https://docs.flutter.dev/release/upgrade#other-channels) 上使用 Flutter Scene，因此請從切換到它開始。

```
flutter channel main
flutter upgrade
```

接下來，建立一個新的 Flutter 專案。

```
flutter create my_3d_app
cd my_3d_app
```

Flutter Scene 依賴於實驗性的「原生資產」功能，以自動建立著色器。您將在稍後使用原生資產來設定 Flutter Scene 的 3D 模型自動匯入。

使用以下命令啟用原生資產。

```
flutter config --enable-native-assets
```

最後，將 Flutter Scene 作為專案相依性加入。

您還需要在與 Flutter Scene 的 API 互動時使用幾個 `vector_math` 構造，因此也要加入 `vector_math` 套件。

```
flutter pub add flutter_scene vector_math
```

接下來，匯入一個 3D 模型！

### 匯入 3D 模型

首先，您需要一個要渲染的 3D 模型。在本指南中，您將使用一個通用的 [glTF](https://en.wikipedia.org/wiki/GlTF) 範例資產：[DamagedHelmet.glb](https://github.com/KhronosGroup/glTF-Sample-Assets/tree/main/Models/DamagedHelmet)。它看起來像這樣。

<figure>
<img alt="" src="https://cdn-images-1.medium.com/max/912/0*vVWRLxJ348tCxv7T" />
<figcaption>最初的 Damaged Helmet 模型是由 theblueturtle_ 於 2016 年建立的（許可證：[CC BY-NC 4.0 國際](https://creativecommons.org/licenses/by-nc/4.0/legalcode)）。轉換的 glTF 版本是由 ctxwing 於 2018 年建立的（許可證：[CC BY 4.0 國際](https://creativecommons.org/licenses/by/4.0/legalcode)）。</figcaption>
</figure>

您可以從託管在 GitHub 上的 [glTF 範例資產儲存庫](https://raw.githubusercontent.com/KhronosGroup/glTF-Sample-Assets/main/Models/DamagedHelmet/glTF-Binary/DamagedHelmet.glb) 中獲得它。將 `DamagedHelmet.glb` 放置在您的專案根目錄中。

```
curl -O https://raw.githubusercontent.com/KhronosGroup/glTF-Sample-Models/main/2.0/DamagedHelmet/glTF-Binary/DamagedHelmet.glb
```

與大多數即時 3D 渲染器一樣，Flutter Scene 在內部使用一種專用的 3D 模型格式。您可以使用 Flutter Scene 的離線匯入器工具將標準 glTF 二進位檔案（`.glb` 檔案）轉換為這種格式。

將 flutter_scene_importer 套件作為普通相依性加入到專案中。

```
flutter pub add flutter_scene_importer
```

加入此套件使您可以透過 `dart run` 手動呼叫匯入器。

```
dart --enable-experiment=native-assets \
 run flutter_scene_importer:import \
 --input "path/to/my/source_model.glb" \
 --output "path/to/my/imported_model.model"
```

您可以透過使用原生資產建立鉤子來自動執行匯入器。若要執行此操作，首先將 `native_assets_cli` 作為普通專案相依性安裝。

```
flutter pub add native_assets_cli
```

現在您可以撰寫建立鉤子了。使用以下內容建立 `hook/build.dart`。

```
import 'package:native_assets_cli/native_assets_cli.dart';
import 'package:flutter_scene_importer/build_hooks.dart';

void main(List<String> args) {
 build(args, (config, output) async {
 buildModels(buildConfig: config, inputFilePaths: [
 'DamagedHelmet.glb',
 ]);
 });
}
```

使用來自 `flutter_scene_importer` 的 `buildModels` 公用程式，提供要建立的模型列表。路徑相對於專案的建立根目錄。

當 Flutter 工具建立專案時，`buildModels` 現在將建立著色器組合包，並將結果輸出到套件根目錄下的 `build/models/DamagedModel.model`。

匯入的模型格式本身與您使用的 Flutter Scene 特定版本相關聯，並且會隨著時間推移而發生變化。在編寫使用 Flutter Scene 的應用程式或函式庫時，請勿將產生的 `.model` 檔案檢查到您的原始碼樹中。相反，請使用建立鉤子從您的原始模型產生它們（如前所述）。

透過這種方式，隨著時間推移，您升級 Flutter Scene 時，將始終使用正確格式建立新的 `.model` 檔案！

接下來，像普通資產一樣匯入模型。將一個資產條目加入到專案的 `pubspec.yaml`。

```
flutter:
 assets:
 - build/models/
```

未來，原生資產功能將允許建立鉤子將資料資產附加到組合包中。一旦發生這種情況，將不再需要在建立鉤子旁邊加入資產匯入規則。

### 渲染 3D 場景

現在是應用程式的程式碼了。

首先，建立一個有狀態的 Widget，以在畫面之間持久保存場景。

您將根據時間進行動畫，因此請將 `SingleTickerProviderStateMixin` 加入到狀態中，以及一個 `elapsedSeconds` 成員。

```
import 'dart:math';

import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';
import 'package:flutter_scene/camera.dart';
import 'package:flutter_scene/node.dart';
import 'package:flutter_scene/scene.dart';
import 'package:vector_math/vector_math.dart';

void main() {
 runApp(const MyApp());
}

class MyApp extends StatefulWidget{
 const MyApp({super.key});

 @override
 MyAppState createState() => MyAppState();
}

class MyAppState extends State<MyApp> with SingleTickerProviderStateMixin {
 double elapsedSeconds = 0;
 Scene scene = Scene();

 @override
 Widget build(BuildContext context) {
 return MaterialApp(
 title: 'My 3D app',
 home: Placeholder(),
 );
 }
}
```

執行應用程式作為 smoke test，以確保沒有錯誤。請記住 [啟用 Impeller](https://docs.flutter.dev/perf/impeller#availability)！

```
flutter run -d macos --enable-impeller
```

<figure>
<img alt="" src="https://cdn-images-1.medium.com/max/912/0*74qs6ytcTjyVHwML" />
</figure>

在繼續之前，請為動畫設定計時器。覆寫 `MyAppState` 中的 `initState` 以呼叫 `createTicker`。

```
 @override
 void initState() {
 createTicker((elapsed) {
 setState(() {
 elapsedSeconds = elapsed.inMilliseconds.toDouble() / 1000;
 });
 }).start();

 super.initState();
 }
```

只要 Widget 可見，就會在每個畫面中呼叫計時器回呼。呼叫 `setState` 會導致此 Widget 在每個畫面中重新建立。

接下來，載入您之前放置在專案中的 3D 模型，並將其加入到場景中。

使用 `Node.fromAsset` 從資產組合包中載入模型。將以下程式碼放置在 `initState` 中。

```
 Node.fromAsset('build/models/DamagedHelmet.model').then((model) {
 model.name = 'Helmet';
 scene.add(model);
 });
```

`Node.fromAsset` 會非同步地從資產組合包中反序列化模型，並在它準備好被加入到場景中時解析回傳的 `Future<Node>`。

現在 `MyAppState.initState` 應如下所示：

```
 @override
 void initState() {
 createTicker((elapsed) {
 setState(() {
 elapsedSeconds = elapsed.inMilliseconds.toDouble() / 1000;
 });
 }).start();

 Node.fromAsset('build/models/DamagedHelmet.model').then((model) {
 model.name = 'Helmet';
 scene.add(model);
 });

 super.initState();
 }
```

但是，您仍然沒有真正渲染 3D 場景！若要執行此操作，請使用 `Scene.render`，它會接收一個 UI `Canvas`、一個 Flutter Scene `Camera` 和一個大小。

存取 Canvas 的一種方法是建立一個 `CustomPainter`：

```
class ScenePainter extends CustomPainter {
 ScenePainter({required this.scene, required this.camera});
 Scene scene;
 Camera camera;

 @override
 void paint(Canvas canvas, Size size) {
 scene.render(camera, canvas, viewport: Offset.zero & size);
 }

 @override
 bool shouldRepaint(covariant CustomPainter oldDelegate) => true;
}
```

不要忘記將 `shouldRepaint` 覆寫設定為回傳 `true`，這樣自訂繪製器就會在每次重建時重新繪製。

最後，將自訂繪製器加入到原始樹中。

```
 @override
 Widget build(BuildContext context) {
 final painter = ScenePainter(
 scene: scene,
 camera: PerspectiveCamera(
 position: Vector3(sin(elapsedSeconds) * 3, 2, cos(elapsedSeconds) * 3),
 target: Vector3(0, 0, 0),
 ),
 );

 return MaterialApp(
 title: 'My 3D app',
 home: CustomPaint(painter: painter),
 );
 }
```

此程式碼指示相機以連續的圓圈移動，但始終面向原點。

最後，啟動應用程式！

```
flutter run -d macos --enable-impeller
```

<figure>
<img alt="" src="https://cdn-images-1.medium.com/max/796/0*_-OFc0vhBHAhrPrO" />
</figure>

這是我們匯集的完整原始碼。

```
import 'dart:math';

import 'package:flutter/material.dart';
import 'package:flutter_scene/camera.dart';
import 'package:flutter_scene/node.dart';
import 'package:flutter_scene/scene.dart';
import 'package:vector_math/vector_math.dart';

void main() {
 runApp(const MyApp());
}

class MyApp extends StatefulWidget {
 const MyApp({super.key});

 @override
 MyAppState createState() => MyAppState();
}

class MyAppState extends State<MyApp> with SingleTickerProviderStateMixin {
 double elapsedSeconds = 0;
 Scene scene = Scene();

 @override
 void initState() {
 createTicker((elapsed) {
 setState(() {
 elapsedSeconds = elapsed.inMilliseconds.toDouble() / 1000;
 });
 }).start();

 Node.fromAsset('build/models/DamagedHelmet.model').then((model) {
 model.name = 'Helmet';
 scene.add(model);
 });

 super.initState();
 }

 @override
 Widget build(BuildContext context) {
 final painter = ScenePainter(
 scene: scene,
 camera: PerspectiveCamera(
 position: Vector3(sin(elapsedSeconds) * 3, 2, cos(elapsedSeconds) * 3),
 target: Vector3(0, 0, 0),
 ),
 );

 return MaterialApp(
 title: 'My 3D app',
 home: CustomPaint(painter: painter),
 );
 }
}

class ScenePainter extends CustomPainter {
 ScenePainter({required this.scene, required this.camera});
 Scene scene;
 Camera camera;

 @override
 void paint(Canvas canvas, Size size) {
 scene.render(camera, canvas, viewport: Offset.zero & size);
 }

 @override
 bool shouldRepaint(covariant CustomPainter oldDelegate) => true;
}
```

### Flutter 的輝煌未來

如果您能夠成功遵循這些指南之一並讓某樣東西執行起來：太棒了，恭喜您！

Flutter GPU 和 Flutter Scene 都非常年輕，平台支援有限。但我認為總有一天我們會懷念這些不起眼的開端。

透過 Impeller 的努力，Flutter 團隊完全掌控了渲染堆疊，因為我們需要將渲染器專門用於 Flutter 的使用案例。而現在，我們正在開啟 Flutter 歷史的新篇章。一個由您共同掌控渲染的篇章！

Flutter Scene 最初是 Impeller 中的一個 C++ 元件，與 2D Canvas 渲染器以及一個精簡的 `dart:ui` 擴展一起。在我建立它時，我已經意識到 Flutter Engine 不會是它的最終目的地。

3D 渲染器體系結構決策的海洋浩瀚無垠，沒有哪一種通用的 3D 渲染器能夠很好地解決所有使用案例。「通用」和「高效能」通常是相互矛盾的目標。

充其量，樣樣精通只不過是樣樣不精。

在渲染效能的世界中，情況更糟。專門用於一個使用案例通常意味著降低另一個使用案例的效能。

簡而言之，不可能發佈一個可以解決所有使用案例的通用 3D 渲染器。但通過呈現構建您自己解決方案所需的低階 API（Flutter GPU）並在它之上構建一個有用的通用 3D 渲染器（Flutter Scene），使其易於 Flutter 社群檢查和修改，我們正在為 Flutter 開發人員創造一個空間，讓他們可以享受低淘汰風險和高回報。

<figure>
<img alt="" src="https://cdn-images-1.medium.com/max/506/1*jfeUgpEP9AgAz94yVxVW1g.gif" />
</figure>

我迫不及待地想看看您將使用這些新功能創作出什麼。敬請期待 Flutter Scene 的未來版本。有很多新功能即將推出。

在此同時，我要回到工作崗位上。

很快見。:)

<img src="https://medium.com/_/stat?event=post.clientViewed&referrerSource=full_rss&postId=f33d497b7c11" width="1" height="1" alt=""><hr><p><a href="https://medium.com/flutter/getting-started-with-flutter-gpu-f33d497b7c11">Flutter GPU 入門指南</a> 最初發佈在 <a href="https://medium.com/flutter">Flutter</a> 上的 Medium，人們在那裡透過突出顯示和回應這個故事來繼續討論。</p> 