---
title: [文章翻譯] Getting started with Flutter GPU
tags:
  - Flutter
  - GPU
comments: true
categories:
  - Flutter
abbrlink: 8ddf18fb
date: 2024-09-27 19:51:40
---

在 Flutter 中建立自訂渲染器並渲染 3D 場景。

Flutter 3.24 版本引進了新的低階圖形 API，稱為 Flutter GPU。此外，還有一個由 Flutter GPU 支援的 3D 渲染函式庫，稱為 Flutter Scene (套件：flutter_scene)。Flutter GPU 和 Flutter Scene 目前都處於預覽階段，僅在 Flutter 的主要通道上提供 (由於依賴實驗性功能)，需要啟用 Impeller，並且可能會偶爾引入重大變更。

本文包含這兩個套件的兩個「入門」指南：

1. 🔺 **進階：**  Flutter GPU 入門
如果你是經驗豐富的圖形程式設計師，或者你對低階圖形感興趣，並希望在 Flutter 中從頭開始建立渲染器，那麼本指南將幫助你開始使用 Flutter GPU。你將從頭開始繪製你的第一個三角形……在 Flutter 中！

2. 💚 **中級：**  使用 Flutter Scene 進行 3D 渲染
如果你是一位希望將 3D 功能添加到你的應用程式中的 Flutter 開發人員，或者你希望使用 Dart 和 Flutter 建立 3D 遊戲，那麼本指南適合你！你將設定一個匯入並在 Flutter 中渲染 3D 資產的專案。


### Flutter GPU 入門

⚠️ 警告！⚠️  Flutter GPU 本質上是一個低階 API。絕大多數從 Flutter GPU 的存在中受益的 Flutter 開發人員，很可能會通過使用在 pub.dev 上發布的更高階渲染函式庫 (例如 Flutter Scene 渲染套件) 來實現。如果你對 Flutter GPU API 本身不感興趣，並且只對 3D 渲染感興趣，請跳轉到  使用 Flutter Scene 進行 3D 渲染。

<figure>
<img alt="" src="https://cdn-images-1.medium.com/max/900/0*hAqIOVkaI1IWnOHE" />
<figcaption>閃閃發光。這是一個射線行進的帶符號距離場。你可以使用 Flutter GPU 渲染它，但使用自訂片段著色器也可以做到。 </figcaption>
</figure>


### Flutter GPU 入門

Flutter GPU 是 Flutter 內建的低階圖形 API。它允許你通過編寫 Dart 程式碼和 GLSL 著色器在 Flutter 中建立和整合自訂渲染器。不需要原生平台程式碼。

目前，Flutter GPU 處於早期預覽階段，並提供基本的柵格化 API，但隨著 API 接近穩定狀態，將會繼續添加和改進更多功能。

Flutter GPU 也需要啟用 Impeller。這意味著它只能在 Impeller 支援的平台上使用。在撰寫本文時，Impeller 支援：

* iOS (預設啟用)
* macOS (可選預覽)
* Android (可選預覽)

我們使用 Flutter GPU 的目標是最終支援所有 Flutter 的平台目標。最終目標是在 Flutter 中培養跨平台渲染解決方案的生態系統，這些解決方案對套件作者易於維護，對使用者易於安裝。

3D 渲染只是一個可能的用例。Flutter GPU 還可以用於建立專門的 2D 渲染器，或者執行更為非正統的操作，例如渲染 4D 空間的 3D 切片，或者投影非歐幾里得空間。

一個由 Flutter GPU 支援的自訂 2D 渲染器的一個很好的用例範例是依賴骨骼網格變形的 2D 角色動畫格式。Spine 2D 就是一個很好的範例。這種骨骼網格解決方案通常具有動畫片段，這些片段可以操作層次結構中骨骼的平移、旋轉和縮放屬性，並且每個頂點都有幾個相關的「骨骼權重」，這些權重決定哪些骨骼應該影響頂點以及影響程度。

使用 Canvas 解決方案 (例如 drawVertices)，需要在 CPU 上為每個頂點應用骨骼權重變換。使用 Flutter GPU，骨骼變換可以以統一陣列的形式傳遞給頂點著色器，甚至可以傳遞給紋理採樣器，允許根據骨骼的狀態和每個頂點的骨骼權重，在 GPU 上並行計算每個頂點的最終位置。

說到這裡，讓我們通過一個溫和的入門來開始使用 Flutter GPU：繪製你的第一個三角形！

<figure>
<img alt="" src="https://cdn-images-1.medium.com/max/1024/0*JEI3fLDGcRHWKruT" />
</figure>


### 將 Flutter GPU 添加到你的專案中

首先，請注意，Flutter GPU 目前處於早期預覽階段，可能會出現 API 斷裂。目前 API 已經可以實現很多功能，但經驗豐富的圖形工程師可能會注意到一些缺失的通用功能。在接下來的幾個月中，Flutter GPU 將會進行很多規劃。

出於這些原因，強烈建議你現在針對 Flutter GPU 開發套件時，針對  主要通道  的頂端進行操作。如果你遇到任何意外行為、錯誤或功能請求，請使用標準  Flutter 錯誤範本  在 GitHub 上提交錯誤。所有與 Flutter GPU 相關的追蹤錯誤都將標記為  flutter-gpu 標籤。

因此，在嘗試使用 Flutter GPU 之前，請通過執行以下命令將 Flutter 切換到主要通道。

```
flutter channel main
flutter upgrade
```

現在建立一個新的 Flutter 專案。

```
flutter create my_cool_renderer
cd my_cool_renderer
```

接下來，將 flutter_gpu SDK 套件添加到你的 pubspec 中。

```
flutter pub add flutter_gpu --sdk=flutter
```


### 建立和匯入著色器捆綁包。

為了使用 Flutter GPU 渲染任何東西，你需要編寫一些 GLSL 著色器。Flutter GPU 的著色器與 Flutter 的  片段著色器  功能所消耗的著色器具有不同的語義，特別是在統一綁定方面。你還需要定義一個頂點著色器與片段著色器一起使用。

從定義最簡單的著色器開始。你可以將著色器放置在專案中的任何位置，但對於本範例，請建立一個 shaders 目錄，並用兩個著色器填充它：simple.vert 和 simple.frag。

```
// 複製到：shaders/simple.vert

in vec2 position;

void main() {
 gl_Position = vec4(position, 0.0, 1.0);
}
```

在繪製三角形時，你將擁有一份數據列表，該列表定義每個頂點。在本例中，它只列出了 2D 位置。對於這些頂點中的每一個，簡單的頂點著色器將這些 2D 位置分配給剪輯空間輸出內在 gl_Position。

```
// 複製到：shaders/simple.frag

out vec4 frag_color;

void main() {
 frag_color = vec4(0, 1, 0, 1);
}
```

片段著色器更簡單；它輸出一個 RGBA 顏色，範圍為 (0, 0, 0, 0) 到 (1, 1, 1, 1)。因此，所有內容都將被陰影為綠色。

好了，現在你有了著色器，請使用 Flutter 的提前 (AOT) 著色器編譯器編譯它們。為了設定著色器捆綁包的自動化構建，我們建議使用  flutter_gpu_shaders  套件。

使用 pub 將 flutter_gpu_shaders 作為你的專案中的常規依賴項添加。

```
flutter pub add flutter_gpu_shaders
```

Flutter GPU 著色器被捆綁到 .shaderbundle 文件中，這些文件可以作為常規資產添加到你的專案的資產捆綁包中。著色器捆綁包包含針對平台目標的編譯後的著色器源程式碼。

接下來，建立一個著色器捆綁包清單文件，該文件描述著色器捆綁包的內容。將以下內容添加到專案根目錄中的 my_renderer.shaderbundle.json 中。

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

著色器捆綁包中的每個條目都可以具有任意的名稱。在本例中，名稱為「SimpleVertex」和「SimpleFragment」。這些名稱用於在你的應用程式中查找著色器。

接下來，使用 flutter_gpu_shaders 套件構建 shaderbundle。你可以通過啟用實驗性的「原生資產」功能來添加一個自動觸發構建的鉤子。使用以下命令來啟用原生資產並安裝 native_assets_cli 套件。

```
flutter config --enable-native-assets
flutter pub add native_assets_cli
```

啟用原生資產功能後，在 hook 目錄下添加一個 build.dart 腳本，它將自動觸發構建著色器捆綁包。

```
// 複製到：hook/build.dart

import 'package:native_assets_cli/native_assets_cli.dart';
import 'package:flutter_gpu_shaders/build.dart';

void main(List<String> args) async {
 await build(args, (config, output) async {
 await buildShaderBundleJson(
 buildConfig: config,
 buildOutput: output,
 manifestFileName: 'my_renderer.shaderbundle.json'
 );
 });
}
```

進行此更改後，當 Flutter 工具構建專案時，buildShaderBundleJson 將構建著色器捆綁包，並將結果輸出到套件根目錄下的 build/shaderbundles/my_renderer.shaderbundle。

著色器捆綁包格式本身與你使用的 Flutter 特定版本相關聯，並且可能會隨著時間的推移而更改。如果你正在編寫一個構建著色器捆綁包的套件，請不要將生成的 .shaderbundle 文件檢查到你的源程式碼樹中。相反，請使用構建鉤子自動化構建過程 (如前所述)。

這樣一來，使用你的函式庫的開發人員將始終使用正確格式構建新的著色器捆綁包！

現在你已經自動構建了著色器捆綁包，請像匯入常規資產一樣匯入它。在你的專案的 pubspec.yaml 中添加一個資產條目：

```
flutter:
 assets:
 - build/shaderbundles/
```

將來，原生資產功能將允許構建鉤子將數據資產附加到捆綁包中。一旦發生這種情況，就不再需要添加資產匯入規則以及構建鉤子。

接下來，添加一些程式碼在執行時載入著色器。建立 lib/shaders.dart 並添加以下程式碼。

```
// 複製到：lib/shaders.dart

import 'package:flutter_gpu/gpu.dart' as gpu;

const String _kShaderBundlePath =
 'build/shaderbundles/my_renderer.shaderbundle';
// 注意：如果你正在構建一個函式庫，則路徑必須以套件名稱為前綴。例如：
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

這段程式碼為 Flutter GPU 著色器執行時函式庫建立了一個單例獲取器。第一次訪問 shaderLibrary 時，將使用 gpu.ShaderLibrary.fromAsset(shader_bundle_path) 使用構建的資產捆綁包初始化執行時著色器函式庫。

現在專案已經設定好了，可以使用 Flutter GPU 著色器。是時候渲染那個三角形了！


### 繪製你的第一個三角形

在本指南中，你將建立一個 RGBA Flutter GPU 紋理和一個渲染通道，將紋理附加為顏色輸出。然後，你將使用  Canvas.drawImage  在小部件中渲染紋理。

為了簡潔起見，你將放棄最佳實踐，並僅為每個幀重建所有資源。

只要在分配紋理時將其標記為「著色器可讀」，你就可以將其轉換為 dart:ui.Image。要將渲染結果顯示在小部件樹中，請將其繪製到 dart:ui.Canvas 中！

你可以通過使用自訂繪製器搭建小部件樹來訪問 Canvas。將 lib/main.dart 的內容替換為以下內容：

```
import 'dart:typed_data';

import 'package:flutter/material.dart';
import 'package:flutter_gpu/gpu.dart' as gpu;

// 注意：我們之前在設定著色器捆綁包匯入時創建了這個！
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
 // 嘗試訪問 `gpu.gpuContext`。
 // 如果 Flutter GPU 不受支援，則將拋出異常。
 print('Default color format: ' +
 gpu.gpuContext.defaultColorFormat.toString());
 }

 @override
 bool shouldRepaint(covariant CustomPainter oldDelegate) => true;
}
```

現在，執行應用程式。請提醒一下，Flutter GPU 目前需要啟用 Impeller。因此你必須使用 Impeller 支援的平台。在本指南中，我將針對 macOS。

```
flutter run -d macos --enable-impeller
```

<figure>
<img alt="" src="https://cdn-images-1.medium.com/max/1024/0*lKTtaX2ih6dFpSMQ" />
</figure>

如果 Flutter GPU 正在運作，那麼你應該會看到預設顏色格式被列印到控制台中。

```
flutter: Default color format: PixelFormat.b8g8r8a8UNormInt
```

如果 Impeller 未啟用，則在嘗試訪問 gpu.gpuContext 時將拋出異常。

```
Exception: Flutter GPU requires the Impeller rendering backend to be enabled.

The relevant error-causing widget was:
 CustomPaint
```

為了簡單起見，你只會從這裡開始修改 paint 方法。

首先，建立一個 Flutter GPU 紋理，清除它，然後通過將其繪製到 Canvas 中來顯示它。

建立一個與 Canvas 大小相同的紋理。必須選擇一個儲存模式。在本例中，你將紋理標記為 devicePrivate，因為你只會使用從裝置 (GPU) 訪問紋理記憶體的指令。

```
final texture = gpu.gpuContext.createTexture(gpu.StorageMode.devicePrivate,
 size.width.toInt(), size.height.toInt())!;
```

如果通過從主機 (CPU) 上載紋理的數據來覆蓋紋理的數據，那麼請改用 StorageMode.hostVisible。

第三個可用的選項是 StorageMode.deviceTransient，它適用於不需要超過單個渲染通道的壽命的附件 (因此它們可以只存在於磁磚記憶體中，並且不需要由 VRAM 分配支援)。很多時候，深度/模板紋理符合這個標準。

接下來，定義一個 RenderTarget。渲染目標包含一個「附件」集合，這些附件描述了每個片段記憶體佈局及其在渲染通道開始和結束時的設置/拆卸行為。

本質上，渲染目標是渲染通道的可重複使用描述符。

現在，定義一個由一個顏色附件組成的非常簡單的渲染目標。

```
final renderTarget = gpu.RenderTarget.singleColor(
 gpu.ColorAttachment(texture: texture, clearValue: Colors.lightBlue));
```

請注意，這段程式碼將 clearValue 設置為淺藍色。每個附件都有一個 LoadAction 和一個 StoreAction，它們分別決定在通道開始和結束時應該對附件的臨時磁磚記憶體執行什麼操作。

預設情況下，顏色附件被設置為 LoadAction.clear (它將磁磚記憶體初始化為給定的顏色)，以及 StoreAction.store (它將結果保存到附加的紋理的 VRAM 分配中)。

現在，建立一個命令緩衝區，使用前面的渲染目標從中生成一個渲染通道，然後立即提交命令緩衝區以清除紋理。

```
final commandBuffer = gpu.gpuContext.createCommandBuffer();
final renderPass = commandBuffer.createRenderPass(renderTarget);
// ... 繪製呼叫將在這裡執行！
commandBuffer.submit();
```

剩下的就是將初始化的紋理繪製到 Canvas 中！

```
final image = texture.asImage();
canvas.drawImage(image, Offset.zero, Paint());
```

<figure>
<img alt="" src="https://cdn-images-1.medium.com/max/1024/0*ebUDtzQOuIGmdlop" />
</figure>

現在你已經將一個渲染通道與顯示到螢幕上的結果連接起來了，你就可以開始繪製三角形了。為此，請設置以下內容：

1.  從我們的著色器建立的渲染管道，以及
2.  一個包含我們幾何體的 GPU 可訪問緩衝區 (三個頂點位置)。

建立渲染管道很容易。你只需要將你的函式庫中的頂點著色器和片段著色器組合起來。

```
final vert = shaderLibrary['SimpleVertex']!;
final frag = shaderLibrary['SimpleFragment']!;
final pipeline = gpu.gpuContext.createRenderPipeline(vert, frag);
```

現在是幾何體。回想一下，「SimpleVertex」著色器只有一個輸入：in vec2 position。因此，要繪製三個頂點，你需要三組兩個浮點數。

```
final vertices = Float32List.fromList([
 -0.5, -0.5, // 第一個頂點
 0.5, -0.5, // 第二個頂點
 0.0, 0.5, // 第三個頂點
]);
final verticesDeviceBuffer = gpu.gpuContext
 .createDeviceBufferWithCopy(ByteData.sublistView(vertices))!;
```

剩下的就是綁定新的資源並呼叫 renderPass.draw() 來完成記錄繪製呼叫。

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

如果你啟動應用程式，你現在應該會看到一個綠色的三角形！

<figure>
<img alt="" src="https://cdn-images-1.medium.com/max/1024/0*LWnGU5WPT_Eom0wJ" />
</figure>

太棒了，你使用 Flutter、Dart 和一些 GLSL 從頭開始建立了一個渲染器！

無論這是否是你第一次渲染三角形，或者你是一位經驗豐富的圖形專家，我都希望你繼續使用 Flutter GPU 並查看我們正在開發的套件，例如 Flutter Scene。

將來，我們希望發布友好的入門程式碼實驗室，深入探討 Flutter GPU 的預設行為和最佳實踐。我們還沒有討論頂點屬性佈局、紋理綁定、統一和對齊要求、管道混合、深度和模板附件、透視校正等等！

在那之前，我建議你探索  Flutter Scene  ，作為如何使用 Flutter GPU 的一個更全面的範例。


### 使用 Flutter Scene 進行 3D 渲染

Flutter Scene (套件 flutter_scene) 是一個新的由 Flutter GPU 支援的 3D 場景圖套件，它使 Flutter 開發人員能夠匯入動畫 glTF 模型並渲染實時 3D 場景。

我們的目標是提供一個套件，讓使用 Flutter 建立互動式 3D 應用程式和遊戲變得容易。

<figure>
<img alt="" src="https://cdn-images-1.medium.com/max/1024/0*tC68CbPLef2rJp1e" />
</figure>

這個套件最初是一個使用 C++ 編寫的 3D 渲染器的 dart:ui 擴展，並直接構建到 Flutter 的原生執行時中，但它已針對 Flutter GPU 重新編寫，並具有更靈活的介面。

與 Flutter GPU API 本身一樣，Flutter Scene 目前處於早期預覽階段，需要啟用 Impeller。Flutter Scene 通常與 Flutter GPU API 的重大變更保持同步，因此強烈建議你在嘗試使用 Flutter Scene 時使用  主要通道。

接下來，使用 Flutter Scene 建立一個應用程式！


### 設定一個 Flutter Scene 專案

由於強烈建議你針對  主要通道  使用 Flutter Scene，因此請從切換到主要通道開始。

```
flutter channel main
flutter upgrade
```

接下來，建立一個新的 Flutter 專案。

```
flutter create my_3d_app
cd my_3d_app
```

Flutter Scene 依賴於實驗性的「原生資產」功能，以自動構建著色器。你將在稍後使用原生資產來設置 Flutter Scene 的 3D 模型的自動匯入。

使用以下命令啟用原生資產。

```
flutter config --enable-native-assets
```

最後，將 Flutter Scene 作為專案依賴項添加。

你還需要在與 Flutter Scene 的 API 互動時使用幾個 vector_math 構造，因此也添加 vector_math 套件。

```
flutter pub add flutter_scene vector_math
```

接下來，匯入一個 3D 模型！


### 匯入一個 3D 模型

首先，你需要一個要渲染的 3D 模型。在本指南中，你將使用一個通用的  glTF  樣本資產：  DamagedHelmet.glb。以下是它的樣子。

<figure>
<img alt="" src="https://cdn-images-1.medium.com/max/912/0*vVWRLxJ348tCxv7T" />
<figcaption>原始的 Damaged Helmet 模型是由 theblueturtle_ 在 2016 年創建的 (許可證：  CC BY-NC 4.0 國際許可證)。轉換後的 glTF 版本是由 ctxwing 在 2018 年創建的 (許可證：  CC BY 4.0 國際許可證)</figcaption>
</figure>

你可以從  GitHub 上託管的 glTF 樣本資產庫  中獲取它。將 DamagedHelmet.glb 放置在你的專案根目錄中。

```
curl -O https://raw.githubusercontent.com/KhronosGroup/glTF-Sample-Models/main/2.0/DamagedHelmet/glTF-Binary/DamagedHelmet.glb
```

與大多數實時 3D 渲染器一樣，Flutter Scene 在內部使用專用的 3D 模型格式。你可以使用 Flutter Scene 的離線匯入工具將標準 glTF 二進制文件 (.glb 文件) 轉換為這種格式。

將 flutter_scene_importer 套件作為常規依賴項添加到專案中。

```
flutter pub add flutter_scene_importer
```

添加這個套件後，可以使用 dart run 手動呼叫匯入器。

```
dart --enable-experiment=native-assets \
 run flutter_scene_importer:import \
 --input "path/to/my/source_model.glb" \
 --output "path/to/my/imported_model.model"
```

你可以通過使用原生資產構建鉤子來自動執行匯入器。為此，首先將 native_assets_cli 作為常規專案依賴項安裝。

```
flutter pub add native_assets_cli
```

現在你可以編寫構建鉤子了。使用以下內容建立 hook/build.dart。

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

使用 flutter_scene_importer 中的 buildModels 工具，提供要構建的模型列表。這些路徑相對於專案的構建根目錄。

當 Flutter 工具構建專案時，buildModels 現在將構建著色器捆綁包，並將結果輸出到套件根目錄下的 build/models/DamagedModel.model。

匯入的模型格式本身與你使用的 Flutter Scene 特定版本相關聯，並且會隨著時間的推移而更改。在編寫使用 Flutter Scene 的應用程式或函式庫時，請不要將生成的 .model 文件檢查到你的源程式碼樹中。相反，請使用構建鉤子從你的源模型生成它們 (如前所述)。

這樣一來，隨著你升級 Flutter Scene，你將始終使用正確格式構建新的 .model 文件！

接下來，像匯入常規資產一樣匯入模型。在你的專案的 pubspec.yaml 中添加一個資產條目。

```
flutter:
 assets:
 - build/models/
```

將來，原生資產功能將允許構建鉤子將數據資產附加到捆綁包中。一旦發生這種情況，就不再需要添加資產匯入規則以及構建鉤子。


### 渲染一個 3D 場景

現在是應用程式的程式碼。

首先，建立一個有狀態的小部件來跨幀持久化場景。

你將根據時間進行動畫處理，因此將 SingleTickerProviderStateMixin 添加到狀態以及 elapsedSeconds 成員。

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

執行應用程式作為煙霧測試，以確保沒有錯誤。請記住啟用 Impeller！

```
flutter run -d macos --enable-impeller
```

<figure>
<img alt="" src="https://cdn-images-1.medium.com/max/912/0*74qs6ytcTjyVHwML" />
</figure>

在繼續之前，請為動畫設置計時器。覆蓋 MyAppState 中的 initState 以呼叫 createTicker。

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

只要小部件可見，計時器回呼就會為每一幀呼叫。呼叫 setState 將觸發這個小部件在每一幀重新構建。

接下來，載入你之前放置在專案中的 3D 模型，並將其添加到場景中。

使用 Node.fromAsset 從資產捆綁包中載入模型。將以下程式碼放在 initState 中。

```
 Node.fromAsset('build/models/DamagedHelmet.model').then((model) {
 model.name = 'Helmet';
 scene.add(model);
 });
```

Node.fromAsset 異步地從資產捆綁包中反序列化模型，並在模型準備好添加到場景中時解析返回的 Future<Node>。

現在 MyAppState.initState 應該如下所示：

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

但是，你仍然沒有實際渲染 3D 場景！為此，請使用 Scene.render，它接受一個 UI Canvas、一個 Flutter Scene 相機和一個大小。

訪問 Canvas 的一種方法是建立一個自訂繪製器：

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

不要忘記將 shouldRepaint 覆蓋設置為返回 true，這樣自訂繪製器就會在每次重建時重新繪製。

最後，將自訂繪製器添加到源程式碼樹中。

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

這段程式碼指示相機以連續的圓圈移動，但始終面向原點。

最後，啟動應用程式！

```
flutter run -d macos --enable-impeller
```

<figure>
<img alt="" src="https://cdn-images-1.medium.com/max/796/0*_-OFc0vhBHAhrPrO" />
</figure>

以下是我們整理的完整源程式碼。

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


### Flutter 光明的未來

如果你能夠成功地遵循其中一個指南並讓它運行起來：哇，恭喜！

Flutter GPU 和 Flutter Scene 都非常年輕，平台支援有限。但我認為有一天我們會懷念這些不起眼的開端。

憑藉 Impeller 的努力，Flutter 團隊完全擁有了渲染堆棧，因為我們需要將渲染器專門用於 Flutter 的用例。現在，我們正在開啟 Flutter 歷史的新篇章。一個由你共同控制渲染的篇章！

Flutter Scene 最初是 Impeller 中的一個 C++ 組件，與 2D Canvas 渲染器以及一個精簡的 dart:ui 擴展一起。在我構建它時，我已經意識到 Flutter 引擎不會是它的最終目的地。

3D 渲染器的架構決策的海洋是浩瀚無垠的，沒有哪一種通用的 3D 渲染器能夠很好地解決所有用例。「通用」和「高性能」通常是相互矛盾的目標。

充其量，在所有方面都達到足夠的水準，幾乎等於在所有方面都無法達到卓越。

在渲染性能的世界裡，情況更加糟糕。專門針對某一個用例，往往意味著降低另一個用例的性能。

簡而言之，不可能發布一個通用的 3D 渲染器，能夠為所有人解決所有用例。但是，通過公開構建自己的解決方案所需的低階 API (Flutter GPU)，以及在它的基礎上構建一個有用的通用 3D 渲染器，這個渲染器易於 Flutter 社區檢查和修改 (Flutter Scene)，我們正在為 Flutter 開發人員創造一個空間，讓他們可以享受低風險的過時風險和高回報。

<figure>
<img alt="" src="https://cdn-images-1.medium.com/max/506/1*jfeUgpEP9AgAz94yVxVW1g.gif" />
</figure>

我迫不及待地想看看你會用這些新功能做些什麼。敬請期待 Flutter Scene 的未來版本。有很多東西正在路上。

同時，我要回到工作崗位了。

很快就會見到你。:)