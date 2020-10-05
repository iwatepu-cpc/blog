---
title: "Common LispでCFFIでVulkanのAPIを呼ぶ"
date: 2020-10-05T22:03:18+09:00
author: "cddadr"
draft: false
---

## はじめに

仕様がオープンなグラフィックスAPI「Vulkan」がKhronos Groupから発表されて数年経つ。
公式からはC/C++のヘッダーファイルが提供され、他言語からも扱いやすくなっている。
ここで筆者は有志が作成したCommon Lisp向けのバインディング([3b/cl-vulkan](https://github.com/3b/cl-vulkan))を使おうと考えたが、READMEにはTODOが並び、3年前で更新が止まっている。どうにか自力でバインディングを生やしたい。

Vulkanからはvk.xmlというAPIのインデックスのようなものも提供されており、既存のバインディングはこれをパースして自動生成しているらしい。今回はCFFIの書き方を理解するため、手動でバインディングを生成する。

CFFI(The Common Foreign Function Interface)はCommon Lispの処理系依存を吸収し、統一されたAPIで他言語で作成されたライブラリの関数等を呼び出すためのライブラリである。FFIはCommon Lispの言語仕様の範疇ではないため、処理系毎に独自実装されているのだ。

## 準備

普段使っているLinuxマシンではなく、ろくに環境構築していないWindows PC上でこれを書いているため、新たにCommon Lispの開発環境を準備する。

まずは処理系をインストールする。近年のモダンなプログラミング言語では処理系管理ツールが盛んに利用されているが、有難いことにCommon Lispにも同様なツールが存在する。[Roswell](https://github.com/roswell/roswell)という。名前は、"made with secret alien technology"というCommon Lispの(誰が言い出したのかは知らない)キャッチフレーズと、そこから連想されるロズウェルUFO事件に由来する。

Roswellは[Installation Guide](https://github.com/roswell/roswell/wiki/Installation)を見ながらインストールした。Scoopは使わず、zipファイルをD:\roswellに展開してパスを通した。

これでRoswellによってCommon Lisp処理系のインストールが出来るようになったので、とりあえず最新のSBCL(2.0.0)をインストールする。

```powershell
ros install sbcl-bin
```

以下のように[Initial Recommended Setup](https://github.com/roswell/roswell/wiki/Initial-Recommended-Setup)を済ませる。

```powershell
code C:\Users\tamam\AppData\Local\config\common-lisp\source-registry.conf
```

```lisp
(:source-registry
  (:tree "/Users/tamam/.roswell/lisp/quicklisp/dists/quicklisp/software/")
  :INHERIT-CONFIGURATION)
```

[CFFI](https://github.com/cffi/cffi)を使うので、Roswellからインストールする。

```powershell
ros install cffi
```

CFFIからVulkanを触るには`vulkan-1.dll`が必要だが、これはGPUドライバと一緒にインストールされる。筆者環境では`C:\Windows\System32\vulkan-1.dll`に格納されていた。

## Vulkan Loaderのライブラリを読み込む

まずはCFFIで`vulkan-1.dll`を読み込む。

```lisp
(ql:quickload :cffi)

(defpackage vkffi
  (:use :cl :cffi))

(in-package vkffi)

(define-foreign-library vulkan
			(:windows "vulkan-1.dll"))

(use-foreign-library vulkan)
```

ここまでを実行してエラーが表示されなければ問題ない。

## 実行する関数のシグネチャをチェック

今回は[`vkEnumerateInstanceExtensionProperties`](https://www.khronos.org/registry/vulkan/specs/1.2-extensions/man/html/vkEnumerateInstanceExtensionProperties.html)を実行してみる。これはレイヤー毎に有効化可能な拡張の一覧を取得する関数である。

マニュアルを読むと、戻り値が`VkResult`型で、引数に`const char*`と`uint32_t`と`VkExtensionProperties*`を取ることが示されている。CFFIではビルトイン型は当然扱えるため、そうでないものに注目する。

- [VkResult](https://www.khronos.org/registry/vulkan/specs/1.2-extensions/man/html/VkResult.html)
- [VkExtensionProperties](https://www.khronos.org/registry/vulkan/specs/1.2-extensions/man/html/VkExtensionProperties.html)

それぞれ列挙型と構造体であることが分かる。まずはこれをCFFIで定義する。名前は合わせなくても良い。

```lisp
(defcenum vk-result
  (:VK_SUCCESS 0)
  (:VK_NOT_READY  1)
  ;; 略
  (:VK_ERROR_INVALID_OPAQUE_CAPTURE_ADDRESS_KHR  -1000257000)
  (:VK_ERROR_PIPELINE_COMPILE_REQUIRED_EXT  1000297000))

(defcstruct vk-extension-properties
	    (extension-name :char :count 256) ; VK_MAX_EXTENSION_NAME_SIZE = 256
	    (spec-version :uint32))
```

次に呼び出す外部関数を定義する。正確な関数名が必要なのでtypoに注意すること。引数は順番が重要であるため、名前は何でも良い。

```lisp
(defcfun ("vkEnumerateInstanceExtensionProperties" vk-enumerate-instance-extension-properties) vk-result
	 (p-layer-name (:pointer :char))
	 (p-property-count (:pointer :uint32))
	 (p-properties (:pointer (:struct vk-extension-properties))))
```

## 実行する

`vkEnumerateInstanceExtensionProperties`は第一引数にNULLを指定すると、Vulkanの実装から提供されるレイヤーと暗黙で有効になっているレイヤーの拡張を取得するようになる。また第三引数にNULLを指定すると、存在する拡張の数を第二引数に指定したポインタの参照先に格納される。

まずは拡張の数を取得する。`with-foreign-object`で格納先の`uint32`型変数を用意する。これは自動でポインタとなるため、値の参照には`mem-ref`を使う。

```lisp
(with-foreign-object (%extension-count :uint32)
                     (format t "vkEnumerateInstanceExtensionProperties(nullptr, &extensionCount, nullptr) ... ~a~%"
                             (vk-enumerate-instance-extension-properties (null-pointer) %extension-count (null-pointer)))
                     (format t "~a extensions supported~%" (mem-ref %extension-count :uint32)))
```

筆者環境では以下の出力を得た。

```
vkEnumerateInstanceExtensionProperties(nullptr, &extensionCount, nullptr) ... VK_SUCCESS
13 extensions supported
```

次に拡張のプロパティを取得する。これは第三引数に`VkExtensionProperties`型配列へのポインタを渡せば良い。配列のサイズは先ほど得た拡張の数とする。配列は`mem-aref`によって添字アクセスし、構造体は`getf`によってフィールド名を指定して値を引き出すことが出来る。`char`型の配列は`convert-from-foreign`で変換先の型に`:string`を指定することでLispで扱える文字列に変換する。

```lisp
(with-foreign-object (%extension-count :uint32)
                     (vk-enumerate-instance-extension-properties (null-pointer) %extension-count (null-pointer))
                     (let ((extension-count (mem-ref %extension-count :uint32)))
                       (format t "~a extensions supported~%" extension-count)
                       (with-foreign-object (%extension-properties '(:struct vk-extension-properties) extension-count)
                                            (format t "vkEnumerateInstanceExtensionProperties(nullptr, &extensionCount, &extensionProperties) ... ~a~%"
                                            (vk-enumerate-instance-extension-properties (null-pointer) %extension-count %extension-properties))
                                            (let ((extension-properties
                                                   (loop for i below extension-count
                                                         collect (mem-aref %extension-properties '(:struct vk-extension-properties) i))))
                                              (loop for ep in extension-properties
                                                    do (format t "~a - spec:~a~%"
                                                               (convert-from-foreign (getf ep 'extension-name) :string)
                                                               (getf ep 'spec-version)))))))
```

これで以下のような出力が得られる。

```
13 extensions supported
vkEnumerateInstanceExtensionProperties(nullptr, &extensionCount, &extensionProperties) ... VK_SUCCESS
VK_KHR_device_group_creation - spec:1
VK_KHR_external_fence_capabilities - spec:1
VK_KHR_external_memory_capabilities - spec:1
VK_KHR_external_semaphore_capabilities - spec:1
VK_KHR_get_physical_device_properties2 - spec:2
VK_KHR_get_surface_capabilities2 - spec:1
VK_KHR_surface - spec:25
VK_KHR_surface_protected_capabilities - spec:1
VK_KHR_win32_surface - spec:6
VK_EXT_debug_report - spec:9
VK_EXT_debug_utils - spec:1
VK_EXT_swapchain_colorspace - spec:4
VK_NV_external_memory_capabilities - spec:1
```

## まとめ

WindowsでCFFIを利用してCommon LispからVulkanの関数を呼び出すことが出来た。

CFFIの詳しい使い方については本記事にて記載していないが、[ユーザマニュアル](https://common-lisp.net/project/cffi/manual/cffi-manual.html)があるのでそちらを読むと良い。
