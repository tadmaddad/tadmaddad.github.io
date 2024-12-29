---
layout: post
title: Chrome拡張機能の改ざんに関する備忘録
subtitle: Chrome extension Investigation
share-img: /assets/img/path.jpg
date: 2024-12-29 14:00:00 +0900
tags: [Security]
author: Tad
---

#### はじめに

先日、Cyberhaven社のChrome拡張機能の一部が改ざんされ、ユーザー情報が不正に収集されているという報告を目にしました。Cyberhaven社は、データの流れを可視化し、不正な移動をリアルタイムで検出・防止する「Data Detection and Response（DDR）」を提供するセキュリティ企業とのこと、同社CEOの声明では、一連のインシデントに関し以下の内容が触れられていました。

**Cyberhaven社CEOの声明（要約）**

- **攻撃手法**: 正規の従業員アカウントがフィッシング攻撃により侵害され、不正な拡張機能がリリースされた。
- **影響範囲**: 拡張機能をインストールしたユーザーが標的となり、不正にデータが収集された可能性がある。
- **対応策**: 拡張機能の削除および影響を受けたユーザーへの通知。
- **今後の対策**: 内部セキュリティの強化および二段階認証の徹底。

参考: [Cyberhaven’s Chrome extension security incident and what we’re doing about it](https://www.cyberhaven.com/blog/cyberhavens-chrome-extension-security-incident-and-what-were-doing-about-it)

また、同社は当該インシデントに関しての初期分析結果についても公開しています。

**Cyberhavenの初期分析結果（要約）**

- **Initial attack vector：**
攻撃者はChrome Web Store Developer Supportを騙ったフィッシングメールをCyberhaven社の従業員（Chrome Extension 開発者）に送付。従業員はメールのリンクをクリックした後にGoogle認証ページに遷移し、悪意あるサードパーティアプリケーション（"Privacy Policy Extension"）に対して意図せず意図せず認証した模様。なお、この従業員のGoogleアカウントについてはGoogle Advanced Protectionが有効であり、またMFAも設定されていたが、今回MFAのプロンプトは表示されなかったとのこと。よって、Cyberhaven社曰く、従業員の認証情報が侵害された訳ではないとのこと。

- **Uploading the malicious extension：**
攻撃者は、悪意あるサードパーティアプリケーションを通じて必要な権限を入手し、Chrome Web Storeに不正なChrome拡張機能をアップロード。その後、Chrome Web Storeのセキュリティ審査プロセスを経て、この不正な拡張機能が公開承認されたとのこと。不正なChrome拡張機能（バージョン：24.10.4）は、正規のCyberhaven社公式のChrome拡張機能をベースにしており、この正規の拡張機能に不正なコードを追加したとのこと。

- 不正なChrome拡張機能のハッシュ値：DDF8C9C72B1B1061221A597168f9BB2C2BA09D38D7B3405E1DACE37AF1587944

- **Analysis of the malicious payload：**
不正なChrome拡張機能には、2つのファイルで構成されており、一つは正規のworker.jsファイルを改変し、C&Cサーバに接続して構成情報をローカルストレージにダウンロード。もう一つは、新たに追加されたcontent.jsファイルで、特定のウェブサイトのユーザデータを収集した後に、worker.jsを通じて入手した構成情報に含まれるサイトに対して送信する模様。なお、Cyberhaven社の調査では、ターゲットとなっていた特定のウェブサイトは、"*.facebook.com"関連のドメインとのことで、他のウェブサイトが標的になったケースは確認されておらず、今回の攻撃が特定のターゲットを狙ったものではなく、facebook.comの広告ユーザーを対象とした一般的な攻撃であると考えられるとのことであった。

- worker.js のハッシュ値：0B871BDEE9D8302A48D6D6511228CAF67A08EC60
- content.js のハッシュ値：AC5CC8BCC05AC27A8F189134C2E3300863B317FB
- C&C：cyberhavenext[.]pro

参考: [Cyberhaven’s preliminary analysis of the recent malicious Chrome extension](https://www.cyberhaven.com/engineering-blog/cyberhavens-preliminary-analysis-of-the-recent-malicious-chrome-extension)

**X上での関連ツイート**

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Cyberhaven breach reported. Employee phished and pushed malicious chrome extension.<br><br>Command and Control:<br>149.28.124.84<br>cyberhavenext[.]pro<br><br>File Hashes:<br>content.js AC5CC8BCC05AC27A8F189134C2E3300863B317FB<br><br>worker.js 0B871BDEE9D8302A48D6D6511228CAF67A08EC60</p>&mdash; Christopher Stanley (@cstanley) <a href="https://twitter.com/cstanley/status/1872365853318225931?ref_src=twsrc%5Etfw">December 26, 2024</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

X上では、Christopher氏（SpaceXやXのセキュリティの人っぽい）が本インシデントについて触れており、一連のポストの中で、"Interesting cert history:" として以下のドメインが挙げられていた。
```
cyberhavenext[.]pro
parrottalks[.]info
ext.linewizeconnect[.]com
readermodeext[.]info
bookmarkfc[.]info
censortracker[.]pro
yujaverity[.]info
wayinai[.]live
vpncity[.]live
moonsift[.]store
primusext[.]pro
internxtvpn[.]pro
uvoice[.]live
```

**その他の改ざんされた拡張機能の有無調査**

上記のポストを踏まえ、他にも改ざんされた拡張機能が存在すると考え調査を実施。まずは、VTで各ドメインを調査してみたところ、ほとんどのドメインがここ一ヶ月以内で取得されたものであることが判明した。
![vt_search_cyberhavenext](/assets/img/2024-12-29-vt_search_cyberhavenext.png)


| ドメイン名                     | ドメイン取得日    |
| ------------------------- | ---------- |
| cyberhavenext[.]pro       | 2024-12-25 |
| parrottalks[.]info        | 2024-12-24 |
| ext.linewizeconnect[.]com | 2024-10-14 |
| readermodeext[.]info      | 2024-12-05 |
| bookmarkfc[.]info         | 2024-12-26 |
| censortracker[.]pro       | 2024-12-23 |
| yujaverity[.]info         | 2024-12-24 |
| wayinai[.]live            | 2024-12-11 |
| vpncity[.]live            | 2024-12-12 |
| moonsift[.]store          | 2024-12-06 |
| primusext[.]pro           | 2024-12-25 |
| internxtvpn[.]pro         | 2024-12-24 |
| uvoice[.]live             | 2024-12-25 |

なお、これらドメインのほとんどがIPアドレス`149.28.124[.]84`に紐づいており、2024/12/29時点（UTCだと12/28）においても、新たなドメインが紐づけられており、キャンペーンは継続中のようにも思われます。
![vt_search_ip](/assets/img/2024-12-29-vt_search_ip.png)

また、上記ドメインはいずれもCyberhaven社のケースと同様に改ざんされた拡張機能に関連したものと思われたことから、ドメイン文字列を使い、chromeウェブストア上で拡張機能を検索してみたところ、幾つかの拡張機能が見つかりました。しかし、この時点ではこれら拡張機能が改ざんされているかは不明だったことから、拡張機能の更新日が新しくなく、且つドメイン作成日と近しい拡張機能に絞って、該当拡張機能をダウンロードすることにしました。

拡張機能のダウンロードは、同じくChrome拡張機能である「Chrome extension source viewer(CRX)」を使って行った。この拡張機能は、chromeウェブストアにて任意の拡張機能のページにて、「CRX」ボタンをクリックすると、拡張機能のダウンロードかソース閲覧ができます。
![crx_download](/assets/img/2024-12-29-crx_download.png)

まずは、拡張機能のダウンロード（zipファイル）を行ったのちに拡張機能を構成するファイルのうち、.jsファイルに着目して内容を確認してみたところ、Cyberhaven社のケース同様に改ざんらしき不正なコードを発見しました。（スクショはCRXのソース閲覧機能で該当箇所を表示）
![crx_code](/assets/img/2024-12-29-crx_malicious_code.png)

**IoC情報**
- C&C
  - 149.28.124[.]84
  - 149.248.2[.]160
