---
title: Firebugを使ったページロード解析(Net panelの読み方)
author: azu
layout: post
permalink: /2011/0612/res2855/
SBM_count:
  - '00044<>1355427654<>39<>0<>5<>0<>0'
dsq_thread_id:
  - 329429420
categories:
  - Firefox
  - javascript
tags:
  - Firebug
  - Firefox
---
[Software is hard | Page load analysis using Firebug][1] の記事を元に書いています。   
January 18th, 2010に書かれた時点での例を使用しているため、現在のブラウザでは若干挙動が違うかもしれません。   
また、著者の[Jan Odvarko][2]さんに翻訳許可は頂いていますが、翻訳というより意訳や自分の理解のために書いていたので、いろいろ書き加えたりしています。   
Net panel(ネットパネル)について理解したいと思って書いたので、Net Panel自体の解説記事[Software is hard | Introduction to Firebug: Net Panel][3]も一緒に読むといいかもしれません。

翻訳許可のメールをわずか15分で返してくれた著者の*Jan Odvarko*さんに感謝を。   
Thnak you, *Jan Odvarko.*

<!--more-->

  
* * *

原文: [Software is hard | Page load analysis using Firebug][1]

ページ読み込みパフォーマンスはWeb開発者にとって重要なトピックです。   
各ページロードの例において、FirebugのNet panelから得たデータをどのように解析するかを見ていきます。

このテストにおける例は[Firefox 3.6][4] + [Firebug 1.5][5] + [NetExport 0.7][6]においてのNet Panelの結果をNetExportにより[HAR][7]形式で出力したものをオンラインで表示しています。   
([HAR][7]形式はChromeのWeb Inspectorからも出力できます。)

*   [HTTP Archive Viewer 2.0.10][8](オンラインビューアー) 

#### Example 1: Simple Page Load



ここでは2つのリクエストが表示されています。   
1つめのリクエストではページは16msで取得できていて、load event (赤いライン)は58msで発生している事がわかります。ログ上にマウスオーバーさせると詳細な表示がされます。

[<img style="background-image: none; border-right-width: 0px; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="https://efcl.info/wp-content/uploads/2011/06/image_thumb11.png" width="300" height="249" />][9]

*   青の縦線であるDOM Loaded はDOMContentLoaded event 
*   赤の縦線であるPage Loaded はLoad event 
*   灰色のReceiving がページの取得にかかった時間 

[<img style="background-image: none; border-right-width: 0px; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="https://efcl.info/wp-content/uploads/2011/06/image_thumb12.png" width="158" height="77" />][10]

同じように2つ目のリクエストを見てみると、5msでXHRが実行されています。   
そして合計経過時間は408msであることもわかります。

[<img style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="https://efcl.info/wp-content/uploads/2011/06/image_thumb13.png" width="307" height="250" />][11]

2つ目のリクエスト(XHR)上でもマウスオーバーで詳細な情報が表示されています。   
こちらでもload event を表していますが、こちらはXHRのリクエストからの相対値が表示されています。XHRが発生(Started)したのは403msなので 403 &#8211; 357 = 58msでload event が発生したことがわかります。(load eventはページの取得を見た方がわかりやすいからこう見る必要はない気がするけど)

#### Example 2: Connection Limit

各ブラウザは一つのページをダウンロードするのに同時接続制限あります。もし、同時接続の制限に達したとき、他のリソースは接続してるリソースが解放されるまでブラウザ内部のキューで待機させられます。   
次の2つのページログを見ると、同時接続制限数の違いで8つの画像をロードするのにどれぐらいの違いがでるかがわかります。

<div class="harPreviewBox">
</div>

**#1 Cuzillion (同時接続制限 == 6)**

*   Firefox 3.6の同時接続制限のデフォルト値は6で、最初の6つの画像をダウンロード開始した時点でドキュメントは利用可能(DOMContentLoaded event が発生する) 
*   他の2つの画像はコネクションが空くまで待っている(light-brown 色の部分がblocking部分) 
*   他の2つの画像がダウンロードされたときは緑色のconnectionフェーズ無いのは、最初の6つの画像をダウンロードに使用したconnectionを再利用しているためです。 

**#2 Cuzillion (同時接続制限 == 2)**

このテストだと、各画像ごとに2～3秒の時間がかかり、合計経過時間やload eventが発生時間に#1の2倍弱近くかかっていることがわかります。   
同時接続制限が2なのはIE7以下のものやモバイル向けのブラウザなどがあります。   
各ブラウザの同時接続数は[Home &#8211; Browserscope][12]を見るといいでしょう。   
Firefoxではabout:configのnetwork.http.max-persistent-connections-per-serverの設定から変更可能です。

[<img style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="https://efcl.info/wp-content/uploads/2011/06/image_thumb14.png" width="514" height="240" />][13]

同じように#2のポップアップされる詳細のログを見てみると、Blocking(light-brown 色)はページを取得(DOM Loaded)した時から4.63s待たされていて、リソースの取得は2.22sかかり、6.85sで完了している事がわかります。

#### Example 3: Pipelining

#2と同じページを使い、ブラウザの[パイプライン処理][14]がONとなっている場合を見てみる。

<div class="harPreviewBox">
</div>

Firefox(3.6)でパイプライン処理が有効(network.http.pipelining == true)になっている場合を比べてみると、実際にはload eventが発生するまでにかかる時間が増えていることがわかります。   
1から6の画像はそれぞれ異なる接続を利用していますが、7から8の画像は共有接続を利用してダウンロードしています。実際にどれと接続を共有しているかは明らかではありませんが(Firebugからそれを知るAPIはないため)、最後の2つの画像は接続を共有して合計時間を増加させているのは明らかです。

#### Example 4: Persistent Connections

このテストはHTTP *Connection*ヘッダーの効果を現しています。このヘッダーは既存の接続が閉じられているか将来のリクエストのために保持されているのかをサーバーに教えるために使われます。   
それぞれ*Keep-Alive* (default in HTTP/1.1) or *Close*でどのような違いが出るかを見てみましょう。

<div class="harPreviewBox">
</div>

それぞれのログで、最初のページと3つのリクエストとで合計4つが表示されています。   
*Connection*ヘッダーは3つ(file1.php, file2.php, file3.php)のリクエスト時に要求されます。

**#1 Test page (Connection: Keep-Alive)**

*   一つ目のページのログでは、ページダウンロードのための一つの接続しか作られていないことがわかります。緑色のConnectionフェーズが一つだけになっていて接続が再利用されている事がわかります。 

**#2 Test page (Connection: close)**

*   二つ目のログでは、ページダウンロードと*file1.php*のリクエストで接続が再利用されている事がわかります。しかし、他の接続ではConnection: closeなため、また接続を作り直す必要があります。 

#### Example 5: Inline Scripts Block

インラインスクリプトがページのダウンロードをブロックすることはよく知られていますが、ネットワークの観点から見るとどのように見えるかを調べてしましょう。

<div class="harPreviewBox">
</div>

**#1 Cuzillion**

3番目と4番目のリクエストの間に隙間ができています。これは5秒間のインラインスクリプトの実行によりブロッキング発生しています。 (\*ここで言うインラインスクリプトとは<script>/\*実行スクリプト*/</script>という感じで<script>タグ内で実行しているものを言う)   
[Inline Scripts Block][15]のページにて試すことができます。

[<img style="background-image: none; border-right-width: 0px; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="https://efcl.info/wp-content/uploads/2011/06/image_thumb15.png" width="320" height="141" />][16]   
Firefox5に試すとちょっと挙動が異なるけど、インラインスクリプトで隙間ができるのは同じ。   
どちらかというとChromeがログに近い動きをしていた。   
Firefox4からインラインスクリプトの実行タイミングが変わったのも関係するかもしれない。   
詳細は[Software is hard | Script Execution Analysis in Firefox 4][17]に。

**#2 Cuzillion**

こちらはインラインスクリプトをページの底(</body>前かな?)に置いた場合のログです。   
リクエストの合計時間は減っていますが、load eventは#1と同じくインラインスクリプトにより遅れていることがわかります。   
こちらは[Cuzillion][18]のようなイメージだと思います。

特にこれをどうとらえるべきかについては元記事に記述はありませんでしたが、この例はリソースのダウンロード時間よりもload eventなどへブロックが発生している事が重要な気がします。   
負荷が大きいインラインスクリプトやdocument.writeなどのブロッキングを誘発するものがある場合loadやDOMContentLoadedイベントの発生を遅くしてしまいます。そのためdocument.writeそのものを避けることや、load eventなどイベントリスナーにして実行させるなどした方がページの表示は速くなるかと思います。

#### Example 6: Redirects

これはネットパネルにて、リダイレクトがどのように見えるかを表示した例。

<div class="harPreviewBox">
  <div class="harPreviewResizer">
  </div></p>
</div>

**#1 Final Page (one redirect)**

サーバーによって301リダイレクトが行われて、URLの末尾に/がついたことがわかります。

**#2 Final Page (infinite redirection)**

無限ループしているリダイレクトの例   
自分自身にリダイレクトしているのでいつまでもループしてしまいますが、Firefoxではデフォルトで20回で停止しています(*network.http.redirection-limit* の設定値)

 [1]: http://www.softwareishard.com/blog/firebug/page-load-analysis-using-firebug/
 [2]: http://www.softwareishard.com/blog/about/
 [3]: http://www.softwareishard.com/blog/firebug/introduction-to-firebug-net-panel/
 [4]: http://www.mozilla.com/en-US/firefox/all-beta.html
 [5]: http://getfirebug.com/releases/firebug/1.5X/
 [6]: http://www.softwareishard.com/blog/netexport/
 [7]: http://groups.google.com/group/firebug-working-group/web/http-tracing---export-format
 [8]: http://www.janodvarko.cz/har/viewer/
 [9]: https://efcl.info/wp-content/uploads/2011/06/image11.png
 [10]: https://efcl.info/wp-content/uploads/2011/06/image12.png
 [11]: https://efcl.info/wp-content/uploads/2011/06/image13.png
 [12]: http://www.browserscope.org/?category=network
 [13]: https://efcl.info/wp-content/uploads/2011/06/image14.png
 [14]: https://developer.mozilla.org/ja/HTTP_Pipelining_FAQ
 [15]: http://stevesouders.com/cuzillion/?ex=10100&title=Inline+Scripts+Block
 [16]: https://efcl.info/wp-content/uploads/2011/06/image15.png
 [17]: http://www.softwareishard.com/blog/firebug/script-execution-analysis-in-firefox-4/
 [18]: http://stevesouders.com/cuzillion/?c0=bi1hfff1_0_f&c1=bi1hfff1_0_f&c2=bb0hfff0_5_f&t=1307854749986
