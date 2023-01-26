---
title: "Laravel + CKEditorが便利すぎた件"
emoji: "🚀"
type: "tech"
topics:
  - "docker"
  - "javascript"
  - "laravel"
  - "ckeditor"
published: true
published_at: "2021-12-05 01:33"
---


# 概要
仕事でブログ記事を書く時のエディタを導入することになったので色々調べて実装してみました！

https://qiita.com/kenzooooo/items/ecdce2b12a72cb750a18

# 解説

:::message
ckeditorとJqueryのCDNをscriptタグに定義してください
:::

```php:sample.blade.php
   <textarea id="ckeditor" name="pageBody"></textarea>

    {{-- CKEditor --}}
    <script src="//cdn.ckeditor.com/4.15.0/full/ckeditor.js"></script>
    <script src="//cdnjs.cloudflare.com/ajax/libs/jquery/1.11.2/jquery.min.js"></script>
    <script src="js/ckeditor.js"></script>
```

:::message
CKEDITOR.replace("ckeditor",{})　内に設定を行うことで、ブログの記事を書くエディタが表示されます
:::

https://ckeditor.com/docs/index.html

![スクリーンショット 2021-11-19 3.58.52.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/555632/156d5fdf-6b45-5f87-3cc6-1f5c732cc752.png)

```javascript:js/ckeditor.js

$(function () {

    // エディタへの設定を適用します
    CKEDITOR.replace("ckeditor", {
        uiColor: "#EEEEEE", // UIの色を指定します
        height: 400, // エディタの高さを指定します

        // スペルチェック機能OFF
        scayt_autoStartup: false,

        // Enterを押した際に改行タグを挿入
        enterMode: CKEDITOR.ENTER_BR,

        // Shift+Enterを押した際に段落タグを挿入
        shiftEnterMode: CKEDITOR.ENTER_P,

        // idやclassを指定可能にする
        allowedContent: true,

        // ファイルマネージャー関連
        filebrowserImageBrowseUrl: "/laravel-filemanager?type=Images",
        filebrowserImageUploadUrl:
            "/laravel-filemanager/upload?type=Images&_token=",
        filebrowserBrowseUrl: "/laravel-filemanager?type=Files",
        filebrowserUploadUrl: "/laravel-filemanager/upload?type=Files&_token=",

        // preコード挿入時
        format_pre: {
            element: "pre",
            attributes: {
                class: "code",
            },
        },
        // タグのパンくずリストを削除
        removePlugins: "elementspath",

        // webからコピペした際でもプレーンテキストを貼り付けるようにする
        forcePasteAsPlainText: true,

        // 自動で空白を挿入しないようにする
        fillEmptyBlocks: false,

        // タブの入力を無効にする
        tabSpaces: 0,
    });
};
```

## 画像もアップロードできるようにしてみた

https://www.webopixel.net/php/1264.html




:::message alert
筆者の環境ではエラーが頻発したので対処方法を紹介します
:::

```bash:bash
composer require unisharp/laravel-filemanager

Your requirements could not be resolved to an installable set of packages.

  Problem 1
    - Installation request for unisharp/laravel-filemanager ^2.2 -> satisfiable by unisharp/laravel-filemanager[v2.2.0].
    - unisharp/laravel-filemanager v2.2.0 requires ext-exif * -> the requested PHP extension exif is missing from your system.
```

### 解決方法

:::message
PHPで画像からEXIF情報を取得するためにEXIFモジュールが必要らしい
:::

https://shinsuke789.hatenablog.jp/entry/2020/06/15/100000

:::message
DockerFileに以下を追加し再度ビルドする
:::

```docker:DockerFile
RUN docker-php-ext-install exif
RUN docker-php-ext-configure exif \
            --enable-exif
```

:::message
ビルド成功後、composer require unisharp/laravel-filemanagerを実行するとエラーが解消しました！
:::

```bash:bash
composer require unisharp/laravel-filemanager
```

# 最後に

読んでいただきありがとうございます。
今回の記事はいかがでしたか？
・こういう記事が読みたい
・こういうところが良かった
・こうした方が良いのではないか
などなど、率直なご意見を募集しております。
