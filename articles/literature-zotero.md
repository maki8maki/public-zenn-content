---
title: "文献管理のすすめ"
emoji: "🕌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["zotero", "latex"]
published: false
---

## はじめに

論文を執筆するときに引用文献の情報を正しく記載することは非常に重要なことです。文献管理ツールはインターネット上の文献情報を登録し、リスト化することでこの作業を手助けしてくれます。

ここでは、私がおすすめしたい「[Zotero](https://www.zotero.org/)」という文献管理ツールと設定などについて紹介します。

:::message
使い方や設定は一から検証したわけではありません。間違い等がありましたら教えて頂けると幸いです。
:::

## 実行環境

- Mac 14.7.4
- Google Chrome
- Zotero 7

WindowsでもZoteroは使用可能ですが、以下で紹介するMacのものとは見た目や配置等が若干異なります。

:::message alert
Zoteroはバージョンによって仕様が大きく変更されることもあるので注意してください。
:::

## Zoteroについて

Zoteroはフリーで使える文献管理ツールです。無料で使える容量は300MBですが、下で紹介するクラウドストレージとの連携によってこの制限を気にしなくてよくなります。
また、Zoteroの最大の特徴はプラグインが充実していることです。様々な人がプラグインを作成しており、これらを活用することでより便利に使用できます。

## 使い方

### 前準備

[Zotero](https://www.zotero.org/)にアクセスし、アカウントの作成・デスクトップアプリのダウンロードします。
また、Chromeを使用する場合は[Zotero Connector](https://chromewebstore.google.com/detail/zotero-connector/ekhagklcjbdpajgpjgmbionohlpdbjgc?pli=1)もChromeに追加しましょう。Zotero Connectorはブラウザで開いた文献情報をZoteroに保存するChrome拡張です。

続いて、クラウドストレージに添付ファイルを保存するフォルダを作成します。私はGoogle Driveでマイドライブ直下に「Zotero」というフォルダを作成しました。

### 初期設定

インストールしたZoteroを起動し、初期設定をしていきます。「Zotero」→「環境設定」（Windowsでは「編集」→「設定」）から設定画面を開きます。

「一般」タブから「ウェブページからアイテムを作成するときに自動的にスナップショットを作成する」のチェックを外すことをおすすめします。文献をZoteroへ保存したときにHTMLも保存してくれる機能ですが、個人的には必要ないと考えています。

![初期設定1](/images/literature-zotero/initial_setting1.png)

また、「キーワードと件名標目を含むアイテムに自動的にタグを付ける」のチェックを外すことをおすすめしている記事もあります。これは文献がキーワードをメタ情報として持っている場合、自動的にタグとして登録してくれる機能です。同じ内容でも別の言葉として登録されうる（robotとRobotなど）ので、OFFにしたほうがいいかもしれません。私は後から気づいたのでONになっています。

![初期設定2](/images/literature-zotero/initial_setting2.png)

続いてファイルのリネームについての設定をします。「Automatically rename locally added files」と「リンクされたファイルの名前を変更」をONにします。文献のPDF等を保存したときに指定のフォーマットにしたがってファイル名を変更してくれるので、フォルダ内の文献を探しやすくなります。

![初期設定3](/images/literature-zotero/initial_setting3.png)

フォーマットのカスタマイズは「Customize Filename Format」をクリックした先で行います。詳細な内容は[公式ドキュメント](https://www.zotero.org/support/file_renaming)を参照してください。私は、以下のように設定しています。

![初期設定4](/images/literature-zotero/initial_setting4.png)

```text
{{ authors max="1" suffix="_" }}{{ year suffix="_" }}{{ title truncate="100" }}
```

「同期」タブで先ほど作成したアカウントを入力し、ログインします。その後「自動的に同期」と「全文を同期する」をONにし、「ファイルの同期」に関する設定をOFFにします。

![初期設定5](/images/literature-zotero/initial_setting5.png)

「詳細」タブの「ファイルとフォルダ」で保存場所の設定をします。「基本ディレクトリ」には最初に作成した添付ファイルを保存するフォルダを指定します。「Data directory」はライブラリ情報等が保存されますが、これはデフォルトのまま変更しません。

![初期設定6](/images/literature-zotero/initial_setting6.png)

最後に、メニューの「表示」の「サブコレクションからアイテムを表示」をONにします。

![初期設定7](/images/literature-zotero/initial_setting7.png)

### Attangerの導入

[Attanger](https://github.com/MuiseDestiny/zotero-attanger)はZoteroの文献データと添付ファイルをクラウドストレージで同期するのに必要（とされている）プラグインです。

:::message
Zoteroがバージョン7にアップグレードされ、従来の[ZotFile](https://zotfile.com/)が使えなくなりました。AttangerはZotFileの代替となるプラグインです。
:::

#### インストール

最初に[リリースページ](https://github.com/MuiseDestiny/zotero-attanger/releases)から最新版のAttanger（zotero-attanger.xpi）をダウンロードします。

続いて、「ツール」→「Plugins」からプラグイン画面を開き、右上の歯車から「Install Plugin From File」を選択して先程の.xpiファイルを開きます。

![Attanger1](/images/literature-zotero/attanger1.png)

以上でAttangerのインストールは完了です。

#### 設定

「Attach Type」を「Link」に変更します。また、「Destination Path」の「Root Directory」にクラウドストレージへのパスを指定します。

以下の設定は必須でないですが、設定すると少し便利になります。

「Destination Path」の「Subfolder」ではファイルの保存時のサブフォルダを指定します。私は `{{collection}}` とし、Zoteroのコレクション（重複登録可能なフォルダのようなもの）にしたがって保存するように設定しています。他の記法は不明なので、情報があれば提供していただけると嬉しいです。

「Source Path」の「Root Directory」は「Attach new file」というコマンドが、新しいファイル（ダウンロードされたばかりなど）を探すフォルダを指定します。ブラウザで指定されているダウンロードフォルダに設定することをおすすめします。

![Attanger2](/images/literature-zotero/attanger2.png)

### 文献の追加

Zoteroに文献を追加する方法は主に以下の方法があります。

#### (a) ブラウザの拡張機能を用いる

Chromeに追加したZotero Connectorを使用します（ChromiumベースのブラウザならChromeでなくても使用できるみたいですが、確認していません）。

追加したい文献のWebページを開き、拡張機能のアイコンをクリックするだけで好きなコレクションに文献情報を登録できます。さらに、本文PDFにアクセス可能な場合は自動で追加してくれます。

#### (b) 本文PDFで追加する

手元に本文PDFがある場合は、ライブラリ表示画面にドラッグ＆ドロップするだけで追加できます。また、ライブラリ画面上部の「添付ファイルを追加する」アイコン→「Add File」からも追加可能です。

PDFに埋め込まれたメタデータから文献情報を読み込み、自動で追加してくれます。しかし、古い文献等では失敗することも多く、全ての情報が記録されているとも限らないため、あまりおすすめする方法ではありません。

![Add1](/images/literature-zotero/add1.png)

#### (c) 文献情報をインポートする

他の文献管理ツールから乗り換えなどで手元にbibtex等の文献情報（の一覧）がある場合は、「ファイル」→「インポート」から一括登録が可能です。

ライブラリにインポートした文献へ本文PDFを追加したい場合は以下の方法があります。

手元にファイルがある場合は、文献を選択→右クリック→「添付ファイルを追加する」（AttangerではなくZotero元々の機能）→「File」→PDFファイルを選択するという手順で追加可能です。

PDFへのアクセス権があれば、文献を選択（複数選択可）→右クリック→「Find Full Text」とすると自動的にPDFを探し、ダウンロードしてくれます。**大学等の機関が個別に契約して閲覧権を得ている場合、短時間に複数のPDFファイルを自動取得すると、「大量ダウンロードによる不正利用」とみなされ、所属機関全体のアクセスが禁止される可能性もあります。ダウンロードは一件ずつ、時間をあけてください。**

## 文献の管理

Zoteroには「コレクション」と「タグ」という管理方法があります。

コレクションはフォルダのようなもので、階層にわけることもできます。さらに、1つの文献を複数のコレクションに振り分けることもできます。
「マイ・ライブラリ」の右クリックから「新規コレクション」を選択することでコレクションを作成できます。作成したコレクションの右クリックから「新規サブコレクション」を選択することでコレクションの下にサブコレクションを作成できます。文献を右クリックし、「コレクションに追加」から各文献をコレクションに追加できます。

タグは名前の通りで、文献にタグ付けをして管理できます。タグから文献を検索もでき、AND検索も可能です。

私は研究テーマごとにコレクション分けをしていました。未・既読によって分けるという考えもあると思います。タグ付けについては主に文献のキーワードでしていましたが、あまり活用はしていません。

## おすすめのプラグイン

ここでは、おすすめのプラグインを紹介します。一応、おすすめ順に並べています。

### Better BibTex for Zotero

[Better BibTex for Zotero](https://retorque.re/zotero-better-bibtex/)（BBT）はbibtex形式での文献リストのエクスポート機能を拡張するプラグインです。LaTeXで論文を書く人には使用を強くおすすめします。

GitHubの[最新リリース](https://github.com/retorquere/zotero-better-bibtex/releases/latest)から.xpiファイルをダウンロードし、Attangerのときと同様にプラグインを追加します。（公式の[インストールページ](https://retorque.re/zotero-better-bibtex/installation/index.html)も参照してください。）

設定→Better BibTexタブで設定を変更できます。
「Citation key formula」では引用キーのフォーマットをカスタマイズできます。私は `auth.lower + year + veryshorttitle` としています。このとき、引用キーは `watkins1992Qlearning` のようになります。その他の書き方については[公式ドキュメント](https://retorque.re/zotero-better-bibtex/citing/)を参照してください。

![BBT1](/images/literature-zotero/bbt1.png)

日本語文献の場合、中国語読みによる引用キーがつけられてしまうことがあります。設定の「Apply kuroshiro romajization in Japanese names/titles. Use a lot of memory」がONになっていることを確認してください。文献情報の「言語」を「日本語」や「Japanese」とすると、ローマ字読みで引用キーを作成してくれます。また、引用キーに日本語をそのまま用いることもできます。[こちら](https://qiita.com/shiro_takeda/items/dfb857e69aa8ed2cc977)を参照してください。

![BBT2](/images/literature-zotero/bbt2.png)

個別に引用キーを変更したい場合は、文献を右クリック→「Better BibTex」→「Change BibTex key...」からできます。

その他の設定も公式の[設定に関するページ](https://retorque.re/zotero-better-bibtex/installation/preferences/index.html)を参照して変更してください。

「Export」→「Fields」→「Fields to omit from export (comma-separated)」は設定することをおすすめします。これは出力しないフィールドを指定するものです。

![BBT3](/images/literature-zotero/bbt3.png)

文献やコレクションを右クリック→「選択されたアイテム（コレクション）をエクスポート...」を選択し、フォーマットで「Better BibTex」を指定してOKとすると、bibtexファイルを出力できます。

![BBT4](/images/literature-zotero/bbt4.png)

また、エントリー名を変えたり、フィールドを変更・追加したりも可能です。例えば、arXivなどのプレプリントはデフォルトではmiscエントリーとして出力されますが、文献情報のその他に `tex.entrytype: preprint` を追加するとエントリー名が `preprint` に変わります。他のフィールドも `tex.hoge: fuga` という書き方で変更できます。

### Translate for Zotero

[Translate for Zotero](https://github.com/windingwind/zotero-pdf-translate)はZotero上でPDF等を翻訳するプラグインです。デフォルトではGoogle翻訳を使用できますが、ここでは性能が高いとされているDeepLを利用する方法を紹介します。

GitHubの[最新リリース](https://github.com/windingwind/zotero-pdf-translate/releases/latest)から.xpiファイルをダウンロードし、Attangerのときと同様にプラグインを追加することでインストールできます。

DeepLで翻訳するために、以下のページからアカウントの作成とAPIの取得をしてください。毎月50万字以内の翻訳が無料です。

https://www.deepl.com/pro-api?cta=header-pro-api

まず、必須の設定を説明します。設定→Translateで設定を変更できます。
Serviceの「Translation Services」で「DeepL (Free Plan)🔑」を選択し、「Secret」に上記で取得したAPIキーを入力してください。続いて、「Translate From」、「To」にそれぞれ翻訳前・後の言語を指定します。これらは翻訳時にも指定できます。

以下は私のおすすめの設定です。
Generalの「Auto-Trans Selection」「Auto-Trans Annotaion」「Replace Raw when Adding to Note」「Auto play prononciation」をOFFにします。そして、それ以外をONにします。以下ではこの設定にしたがった説明となります。

![translation1](/images/literature-zotero/translation1.png)

続いて、使い方についてです。
文献のPDFファイルを開き、翻訳したい箇所を選択するとポップアップが表示されます。「Translate」ボタンをクリックすると、右側のメニューに原文と翻訳後の文章が表示されます。表示されない場合はメニューのTranslateアイコンをクリックしてみてください。

![translation2](/images/literature-zotero/translation2.png)

![translation3](/images/literature-zotero/translation3.png)

### Notero

Zoteroのメモ機能はあまり優れていません。[Notero](https://github.com/dvanoni/notero)を使うことで、[Notion](https://www.notion.com/ja)というツールにZoteroの情報を連携し、Notion上でメモを追加していくことができます。以下で使い方を説明しますが、NoteroのREADMEも参照してください。

最初にNotionのアカウントを作成してください。

Noteroの.xpiファイルを[ダウンロード](https://download.notero.vanoni.dev/)し、Attangerと同様にプラグインを追加します。

次にNotionとの接続設定を行います。設定→Noteroから「Notion Connection」の「Connect to Notion」をクリックするとブラウザに移動します。ワークスペースを選択した後に、連携するデータベースを選択します。Notero側で用意しているテンプレートを利用するか、自分で作成したデータベースを利用するか選べます。Noteroへのアクセスを許可し、Zoteroを開くと終了です。[ここ](https://github.com/dvanoni/notero?tab=readme-ov-file#connect-to-notion)には動作の手順動画があるので参考にしてください。

Noteroの設定ではNotionと連携するコレクションの変更もできます。

Zoteroに文献を追加したり、情報を更新したりすると、Notionにも自動で反映されます。しかし、**Notionの変更はZoteroに反映されない**ので注意してください。

ZoteroからNotionへ移動するには「Notion」というファイルを選択します。開いたページにメモを書き込めます。

![notero1](/images/literature-zotero/notero1.png)

また、Notionのデータベースでタイトルにカーソルを合わせ、「開く」をクリックしても同じページを開けます。

![notero2](/images/literature-zotero/notero2.png)

## 参考

https://www.zotero.org/

https://github.com/MuiseDestiny/zotero-attanger

https://retorque.re/zotero-better-bibtex/index.html

https://github.com/windingwind/zotero-pdf-translate

https://github.com/dvanoni/notero

https://zenn.dev/hrak0x59/articles/af55b02041d055

https://note.com/sdeso/n/n013952313c1b

https://qiita.com/shiro_takeda/items/dfb857e69aa8ed2cc977

https://note.com/sangmin/n/nfd689feb6391

https://qiita.com/shimohara7192/items/5451278a5defc382b3c2
