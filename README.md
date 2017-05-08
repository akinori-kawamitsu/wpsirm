# wpsirm
WordPress Security Incident Response Manual ver. 0.1.1

# この文書について

- WordPressを用いたウェブサイト制作者向けのセキュリティマニュアルとして作成した。
- 本マニュアルは主にサイトへの不正侵入、改ざんを対象としている。
- 個々の作業手順（サイトの削除、自動インストール手順など）や、各種連絡先については利用者が独自に加筆すること。


# 基本姿勢

 1. もっとも大事なのはエンドユーザー（サイト閲覧者）である。
 2. エンドユーザーを守ることなく、顧客企業の信用と利益を守ることはできない。
 3. 日常の備えとアップデートは非常時の迅速な回復に直結する。
 4. このマニュアルは常に見直され、アップデートされなければならない。

# Web制作段階のセキュリティ対策
## 環境整備

パソコンのOSのアップデートと、セキュリティソフトの導入は必須。

これらは、インシデント発生時に、考えられる原因からローカルPCを排除するために常時行っておくべき作業である。
また、紛失・盗難に備えてログインパスワードは十分強いものにし、可能ならば、ドライブの暗号化も行う。

## クライアントとの非常時の対応手順の共有
不正侵入などのインシデントが発生した場合は、速やかにサイトを閉鎖しなければならない。

問題が起きてから閉鎖の許可を取るのは多くの困難が予想される。担当者の不在あるいは閉鎖に対する強い忌避感から許可されない可能性もある。

非常時には先にアクセス遮断を行ってから連絡をする等の条項を、保守契約に入れておく。

## 非常時の連絡先リストを用意する。

 - 社内の担当者と連絡先のリスト
 - クライアントの担当者と連絡先
 - レンタルサーバー会社の非常時の連絡先
 - セキュリティ専門会社の連絡先

## WordPressのインストール

### レンタルサーバー側が提供している「簡単インストール」を用いる。

手動でアップロードした場合、ファイルやフォルダ類のパーミッションが不適切な設定となるリスクを抱えることになる。
推奨されるパーミッションはサーバーごとに異なることがある。サーバーが提供しているインストール機能を用いれば適切に設定できるため、これを用いる。

このインストール方法は、作られるデータベースの接頭辞がwp_になるため忌避されることもあるが、あとに記載するセキュリティプラグインで問題なく変更できる。

### WordPressの自動更新を有効にする

WordPressの自動アップデートは有効にする。

WordPress4.7.2のアップデートで明らかになったことは、脆弱性を修正するのに与えられた猶予は1週間しかないことである。

この期間内ではすべてのサイトでプラグインとの相性チェックを行う余裕はない。自動的にアップデートしたうえで、不具合を個別に解消するというアプローチをとるべきである。

また、プラグインのアップデートも自動化する。

プラグインの自動アップデートはデフォルトでは無効になっているため、functions.phpファイルに以下のコードを加える。

    add_filter( 'auto_update_plugin', '__return_true' );

### 推奨プラグインと設定

#### [BackWPup](https://ja.wordpress.org/plugins/backwpup/)

利用者数が比較的多く、更新も今のところ安定している。日本語での解説が多いのも利点。

設定
- バックアップの**ファイル形式は「Tar.gzip」**を選択する。（zipだとファイル名が日本語だと保存しない）
- バックアップ先の設定に、「folder」は可能な限り避ける。（同一サーバー内ではバックアップごと死んでしまう可能性がある。）
- 可能な限り、バックアップ先はDropbox, AWS(S3), Azure, GoogleDrive などを選択する。
- 「job」については、ファイルとデータベースを含めたバックアップは月1回あるいはサイトに変更を加える際に実施する。
- データベースのバックアップは、毎日実施する。WordPressはアップデート前にデータベースのバックアップが必要だが、自動アップデートではいつアップデートされるかわからないため、データベースのバックアップを毎日実施することで対応する。

運用
- 画像ファイルは横幅を1920px以下に縮小してアップロードする（サイズの大きい画像ファイルをたくさんアップロードしている場合、BackWPupはファイルのバックアップに失敗する。）
- 3kや4kクラスの写真のアップロードは禁止。
- 記事に添付する画像などは、なるべく日本語を避け、短いファイル名とする。（長いファイル名はバックアップが不完全になりやすい）


#### [All In One WP Security & Firewall](https://ja.wordpress.org/plugins/all-in-one-wp-security-and-firewall/)

設定は以下を推奨する。

##### setting>WP Version Info>WP Generator Info

- Remove WP Generator Meta Info:　バージョン情報の入ったメタタグを除去する。　→チェックを入れる

##### 【重要】User Login>Login Lockdown

- Enable Login Lockdown Feature : →チェックを入れる
- Max Login Attempts:　ログイン試行回数。　→3～6回など（適宜設定）
- Login Retry Time Period (min):　ログイン試行時間。　→1分など（適宜設定）
- Time Length of Lockout (min):　ログイン試行失敗時のロックアウト時間　1～60分
- Display Generic Error Message:　ログインエラーメッセージ　→チェックしない
- Instantly Lockout Invalid Usernames:　特定のユーザー名でログインしようとした場合、すぐにロックアウトする。1行に1ユーザー名。　→admin, root, userなど、機械的な攻撃に使われそうな名前を入れる。
- Notify By Email:　→チェックしない。

##### User Login>Force Logout

- ログインから一定時間以上経過すると強制的にログアウトさせる。　→60～480分にセットする。

##### User Registration>Manual Approval

- 新規ユーザー登録の際、管理者の手動での承認を必要にする。　→チェックしない

##### User Registration>Registration Captcha

- 新規ユーザー登録画面において画像認証を有効にする　→チェックしない

##### User Registration>Registration Honeypot

- ロボットにしか見えないフィールドを設置し、そのフィールドに入力があった場合、ローカルホストアドレスにリダイレクトさせる。　→お好きなように

##### 【重要】Database Security>DB Prefix

- 簡単インストールを行った場合、デフォルトのwp_になっているため、データベースのテーブル接頭辞を変更する。
- Generate New DB Table Prefix: →チェックを入れる
- 「Change DB Prefix」ボタンをクリックする。

※テーブル接頭辞とwp-config.phpの設定が同時に変更される。

##### Database Security>DB Backup

- BackWPupプラグインを使って行うため、設定しない。

##### 【重要】Filesystem Security>PHP File Editing

※管理画面からテーマやプラグインファイルの編集をできないようにする。

- Disable Ability To Edit PHP Files:　→チェックを入れる。

##### Filesystem Security>WP File Access

※WPインストールで提供されているreadme.html、license.txt、wp-config-sample.phpなどのファイルへのアクセスを禁止し、WordPressのバージョン情報などを隠す。

- Prevent Access to WP Default Install Files:　→チェックを入れる。

##### 【重要】Brute Force>Rename Login Page

ログインページを、wp-adminから変更する。

- Enable Rename Login Page Feature:　→チェックを入れる
- Login Page URL:　→新しいログインURLを入力
- 上記設定後、「Save Settings」をクリックする。

※次のログインからは新しいURLのログイン画面となる。（wp-adminは404エラーが返るようになる）

##### Brute Force>Login Captcha

- ログイン画面での画像認証　→必要な場合はチェックを入れる。

##### SPAM Prevention

- スパム対策。　→必要な場合にはチェックを入れる。

##### 【重要】Scanner>File Change Detection

※WordPress管理下のファイルの追加、削除、変更を検出し、通知する。→サイトへの侵入や改ざんを早期に発見することができる。

- Enable Automated File Change Detection Scan:　→チェックを入れる。
- Scan Time Interval:　→1Days　に設定する。
- File Types To Ignore:　変更を無視するファイルの拡張子を設定する。1行に1つ。

    jpeg
    jpg
    png
    gif
    css
    scss
    map
	po
	mo

などを設定する。

- Files/Directories To Ignore:　→BackWPupのバックアップファイルを入れるフォルダなどを設定する。
- Send Email When Change Detected:　変更を検知したらメールを送信する。　→チェックを入れ、宛先のアドレスを確認する。

#### その他
[Wordfence Security](https://ja.wordpress.org/plugins/wordfence/)
※All In One WP Security & Firewallとの比較あるいは併用を検討中。


##WordPressテーマの選択

選択肢は次の3種類である。

 1. 公式ディレクトリ登録テーマ
 2. 自作テーマ
 3. 有料テーマ

### 1 公式ディレクトリ登録テーマ
最も安全な選択肢である。

WordPressのコーディング基準に準拠しており、開発者とレビュアーによりチェックがなされていること、コードの可読性が高いこと、正常に子テーマを作ることができ、テーマのアップデートに追従しやすいことである。

数が膨大であることや、説明書が英語であること、デザイン性の良さが分かりにくいことから、目的に合ったテーマを見つけにくい難点はある。

### 2 自作テーマ
コードにブラックボックスがないことが利点として挙げられ、改ざんを目視で発見しやすい利点がある。

一方で、技術力が足りないと脆弱なコードが入り込むことがある。入出力のエスケープ処理などは十分に行うこと。

また、テーマ製作者が組織を去った後もメンテナンスできるように分かりやすいコードと説明書は必須である。

### 3 有料で販売されているテーマ

有料で販売されている中でも、機能限定版が公式ディレクトリにも登録されているようなもの（BizVectorなど）は安全性が高い。

公式ディレクトリに登録されていない有料テーマはやむを得ない場合以外には使うべきではない。

WordPressのコーディング基準に反しており、高機能なものほどコードの可読性が悪い。またテーマのアップデートは提供されていても子テーマを正常に作れないため実際にはアップデートできないような問題を抱えたものもある。

### 【論外】公式ディレクトリに登録されていない無料テーマ

「野良テーマ」と呼ばれるテーマであり、使用は禁止。

ほとんどの場合は不正なコードが仕組まれている。同様に、「野良プラグイン」も決して使わないこと。

## WordPressの運用

### メンテナンス用リダイレクトファイルを用意する

メンテナンス表示用のmaintenance.htmlと.htaccessファイルを準備しておく。

メンテナンス表示用の.htaccessのコード

```
ErrorDocument 503 /maintenance.html

&lt;IfModule mod_rewrite.c&gt;
  RewriteEngine On
  RewriteCond %{REQUEST_URI} !=/maintenance.html
  RewriteCond %{REMOTE_ADDR} !=管理者のIPアドレス
  RewriteRule ^.*$ - [R=503,L]
&lt;/IfModule&gt;
```

[Webサイトのメンテナンス中画面を出す正しい作法と.htaccessの書き方](http://web-tan.forum.impressrd.jp/e/2009/06/16/5880)より。

不正侵入や改ざんが発生した場合、もとの.htaccessファイルはバックアップを取ったうえで、メンテナンス表示用の.htaccessをアップロードする。


なお、WordPressの管理画面に入れる場合は、All In One WP Security & Firewall　プラグインを使ってメンテナンス表示させることもできる。（メニューのmaintenanceから設定する）

### 設定と動作状況の確認

制作時に設定したセキュリティプラグインが正常に動いているか、運用開始前と開始後定期的な確認が必要である。

複数名での運用の場合、ユーザーによって権限設定を適切に設定する。

### パスワード

ユーザーには、十分な強度のパスワードを設定してもらい、使いまわしを禁止する。

パスワードが流出した場合、パスワードを使いまわしていると、一か所のインシデントに対し、使いまわしているパスワード全ての変更が必要になる。

全てのサイトで完全に違うパスワードは通常は覚えられないため、ユーザーには共通キーとなるキーワードの前後、内部にサイト別の文字列を混ぜるなどの方法を提示し、使いまわしの禁止と覚えやすさの両立を図る。

この方法で運用する場合も、人間が見たら簡単に運用ルールが分かってしまうような混ぜ方（例えば、[キーワード]+[サイトのドメインの一部]のような単純な組み合わせ）は避けるべきである。パスワードは暗号化されているとはいえ、解読可能なので人間が簡単に法則を発見できるようでは使いまわしと実質的に同じだからである。

[キーワードA]+[サイトのドメインの母音]+[キーワードB]+[サイト別のドメインの一部]

などのように、一定のややこしさを保つようにする。

### プラグインの老朽化の監視

表計算ソフトを使って使用しているプラグインと、そのプラグインを使用しているサイトをリスト化し、最終更新日からの経過月数をチェックする。

最後の更新が12ヶ月を超えている場合、メンテナンスされていないと判断し、代替のプラグインを探す。

### リカバリー演習

ダミーのサイトを作り、半年～1年に1度くらいの頻度でサイトとデータベースのリカバリーの訓練を行う。

訓練内容は

 1. 現サイトの完全削除
 2. 簡単インストールによる再インストール
 3. uploadフォルダコンテンツのアップロード
 4. データベースの入れ替え
 5. wp-config.phpの編集
 6. 確認

である。

##不正侵入対応手順

###1　パソコンから手を放して深呼吸をする

不正侵入などの通報に直面すると、大きなストレスとプレッシャーに襲われる。それは焦る、慌てる、緊張する、パニックになるなどの形で表れ、正常な判断が難しくなる。

厄介なのは自分がその状態であることを自覚しにくいことである。そのため、まずは不用意な操作を避けるためパソコンから手を放して深呼吸を行い、**多くの場合は誤認や誤検知であることを思い出すこと**。

### 2　状況を把握する

#### 2-1 外部からの電話やメールでインシデントの発生が伝えられた場合

まずはその電話やメール自体が、標的型攻撃の一環である可能性を疑う。電話の場合は、状況を確認するなどの理由で一旦電話を切る。メールの場合はまず文面をよく読み、外部へのリンクに誘導されていないか、添付ファイルを開かせようとしていないか、日本語としておかしいところはないか、などを確認する。

文面が信用できそうな場合でも、独自に改ざんなどがないか確認する。

メール内のリンクはクリックせず、URL直打ちや検索などで探す。また[gred](http://check.gred.jp/)や[aguse.jp](https://www.aguse.jp/)などのサービスを使ってリンク先が安全であることを確認する。

また安全が確認された後も、リンク先をたどった末に「ウイルス除去ツール（と称するマルウェア）」をインストールさせるように促す攻撃手法があるので、その場合は決して従わないこと。

#### 2-2 セキュリティプラグイン等からの通知の場合

管理画面へログインしたり、FTPで確認するなど、不正アクセスや不正なファイルの設置があることを確認する。

それらが権限を持つ関係者による正常な作業を誤認したものでないかを確認する。

不正なファイルと思われるファイルをダウンロードし、ソースコードを確認する。

#### 2-3 Google Search Consoleによる通知の場合

Google Search Consoleでなりすましでないことを確認する。あるいは当該サイトを検索したとき、「このサイトはハッキングされている可能性があります」という表示によって知ることもできる。

### 3　サイトを閉鎖する

実際に不正侵入や改ざんなどが確認できたら、プラグインの機能を使ってサイトを閉鎖する。静的なページとの混合サイトの場合、メンテナンス用リダイレクトファイルを使って一般のアクセスを遮断する。

サイトの一時閉鎖についてクライアントに一報を入れる。

ここから先の作業はバックアップなどの日頃の備えを行っていれば、4時間以内には終了する（訓練次第ではさらに短縮できる）。

### 4　メモを用意する。

何時何分に何を確認し、何を行ったかの記録を残す。

### 5　リカバリーファイルの準備

1. All In One WP Security & Firewallが記録したファイルの変更記録をローカルに保管する。
2. ファイルの不正な変更がいつ行われたのかを確認する。
3. 不正な変更前のバックアップファイルとデータベースのバックアップをローカルPC上に用意する。

### 6　証拠の保全

ローカルに証拠保全用のフォルダを作る。このフォルダはローカルのテスト環境（XAMPPなど）の外に作る。（ローカル環境で不正なプログラムが稼働するのを避けるため）

BackWPupを使って現在のファイルとデータベースの圧縮ファイルを作成し、証拠保全用のフォルダに保存する。（圧縮なしでのダウンロードは時間がかかりすぎるため避ける）

WordPressと静的なファイル類が混合されたサイトの場合は、静的なファイル類をFTPで証拠保全用のフォルダに保存する。

### 7　現在のサイトの削除とデータベースパスワードの変更

現在のサイトのサーバーにあるファイル、ディレクトリをすべて削除する。削除方法はレンタルサーバー会社ごとに違うため注意する。
（例えば、Xserverの場合はサーバーの管理パネルから削除する）

現在のサイトのデータベースを削除するとともに、データベースパスワードも変更する。

### 8　サイトの再構築

#### 8-1 WordPressの再インストール、静的ページのアップロード

WordPressはレンタルサーバーの簡単インストール機能を使って再インストールする。

静的ページがある場合は、不正侵入発生前のバックアップをアップロードする。

#### 8-2 バックアップのテーマファイル、uploads内のファイルのアップロード

不正侵入発生前のテーマファイルと、記事の画像などが入っているuploadsファイル類をアップロードする。

#### 8-3 プラグインを再インストールする

サイトの管理画面にログインし、リストに基づいてプラグインを再インストールする。（この時点では有効化はしなくてもOK）

#### 8-4 データベースの入れ替えとwp-config.phpの編集

サーバーのphpMyAdminにアクセスして、新しいサイトのデータベースをすべて削除する。

バックアップのデータベースファイルをインポートする。

wp-config.phpをローカルにダウンロードし、テーブル接頭辞を新しいデータベースの物に変更してアップロードする。

### 9　サイト復旧の確認とパスワード変更

この時点でプラグインの設定、ユーザーアカウントやパスワードなどは、全てバックアップ時の状態に戻っている。

作業者は速やかにパスワードを変更し、他の管理者ユーザーにも即時のパスワードの変更を要請する。すぐに対応できないユーザーは権限を下げて対応する。

サイトに不具合が無いか確認する。

クライアントにサイト復旧の連絡を入れる。

非管理者権限のユーザーにもパスワード変更を要請する。

### 10　手に負えない場合

サーバー管理会社に連絡を入れる。必要に応じてセキュリティ専門会社に依頼する。

### 11　後処理

保全した証拠ファイルを分析して攻撃内容を把握し、被害内容について検討する。
※よく分からない場合も多い。

状況のレポートを作成する。

時系列メモを見返して反省会を行う。

復旧後も異常が無いか継続的に監視する。

## 参考文献

IPA ISEC 情報処理振興事業協会セキュリティセンター　「[情報セキュリティセミナーインシデントマネジメントv1.0](http://www.ipa.go.jp/security/awareness/administrator/incident.pdf)」

「[コンピュータセキュリティインシデント対応ガイド 米国立標準技術研究所による勧告](https://www.ipa.go.jp/files/000025341.pdf)」

Xserver サポートページ　「[よくある質問　トラブル](https://www.xserver.ne.jp/support/faq/faq_service_hp_trouble.php)」

さくらインターネット　さくらのサポート情報　「[不正なアクセスについて-不正アクセス またはウイルスに感染したら-](https://help.sakura.ad.jp/hc/ja/articles/206206051-%E4%B8%8D%E6%AD%A3%E3%81%AA%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6-%E4%B8%8D%E6%AD%A3%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9-%E3%81%BE%E3%81%9F%E3%81%AF%E3%82%A6%E3%82%A4%E3%83%AB%E3%82%B9%E3%81%AB%E6%84%9F%E6%9F%93%E3%81%97%E3%81%9F%E3%82%89-)」

さくらのナレッジ　「[【第3回】防御と不正アクセスからの復旧 ～ 基礎からしっかり！WordPressのセキュリティ対策ことはじめ (3/3) ](http://knowledge.sakura.ad.jp/knowledge/4253/)」

