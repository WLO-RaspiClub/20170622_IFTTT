## IFTTTで簡単クラウド連携
### Goal
 * IFTTTをつかえるようにする
 * Raspberry PiからIFTTTのAppletのTrigerを起動する
 * IFTTTのAppletのActionでRaspberry PiのLEDを点灯する
 
### IFTTTについて
 * さまざまなクラウドサービスを連携させて新しい機能をつくるサービス
     * 2010年12月14日サービス開始
     * 米国のサービス
 * IF This Then That の頭文字
     * 「これがきたらあれを動かす」をAppletとして定義する
         * むかしはレシピと言ってたが最近変更になった
     * 「これ」=Triggers、「あれ」=Actions
     * つくったAppletは公開して共有できる
         * 非公開でも利用できる
 * 連携できるクラウドサービス https://ifttt.com/search/services
     * 6/19に数えてみたら928種類あった(重複しているかも？)
     * それぞれのサービスはOAuth2やOpenID等でIFTTTとアカウント連携する
         * IFTTTに権限を委譲するので、IFTTTのアカウント管理は厳重に
 * 利用料は無料
     * 企業向けに、自社サービスにIFTTTを組み込める有償サービスがある
         * https://platform.ifttt.com/
 * 類似サービス
     * Zapier https://zapier.com/
     * Integromat https://www.integromat.com/en/
     * Pipes https://www.pipes.digital/
     * Microsoft Flow https://flow.microsoft.com/
     * MyThings https://mythings.yahoo.co.jp/
     * Node-RED https://nodered.org/
         * FRED https://fred.sensetecnic.com/
         * AT&T Flow Designer https://developer.att.com/campaigns/flow-designer-beta
         * Playground環境 
         
### IFTTTアカウント
 * https://ifttt.com/join
     * Googleアカウント、Facebookアカウント、e-mailによるログイン
     * ![IFTTT signup](https://raw.githubusercontent.com/WLO-RaspiClub/20170622_IFTTT/master/img/signup0.png)
     * サービス連携でも使うのでGoogleアカウントでのサインアップがおすすめ
 * 連携サービスのアカウント登録
     * https://ifttt.com/search/services から連携したいサービスを選択
     * 連携先のサービスにログインした状態でconnectする
 * スマートフォンアプリ
     * iOS https://itunes.apple.com/app/apple-store/id660944635?pt=1752682&ct=/do_button&mt=8
     * Android https://play.google.com/store/apps/details?id=com.ifttt.ifttt&utm_source=/do_button&utm_medium=web
     * スマートフォンの機能（位置、カレンダー、住所、プッシュ通知）と連携
 * Pocket (Webページを後で読むためにブックマークするサービス)との連携例
 * Google Driveとの連携例
     
### IFTTTとRaspberry Piとの連携
 * Maker Webhooks https://ifttt.com/maker_webhooks
 * ConnectするとTriggersとActionsが設定
 * Triggers: Raspberry PiからHTTP GET/POSTすることでTriggersを起動できる
     * IFTTTアカウントで1つだけKeyが割り当てられる。Keyを含むURLにアクセスすることでTriggers起動できる
     * HTTP POSTする場合は3つ引数を渡すことができる
         * 引数はAppletに渡され、条件判断や連携先サービスのActionで活用できる
 * Actions: IFTTTからhttp/httpsでアクセスしてくれる
     * GET/POST/DELETEできる
     * Raspberry PiへグローバルにアクセスできるURLがないと利用できない
     * 通常はFirewall/NATルータ配下にRaspberry Piを接続するので、グローバルにアクセス不可
     * VPN/DynamicDNS等を活用することで実現できる（が難しい）
         * CloudPi http://www.planex.co.jp/products/cloudpi/ のようなサービスもある
 * Glue(のり)になる代替サービスを用いてActionsを
     * たとえばGoogle Drive https://drive.google.com/ 
     * Raspberry PiからGoogle Driveにあるファイルの中身を定期的に取得できる
     * 他のサービスから、Google Drive サービス https://ifttt.com/google_drive をTriggerして、Google Driveにファイルを書き込む
     * 間接的にRaspberry Piに情報を送り込める

### Raspberry PiにスイッチとLEDを接続
 * GPIO (General Purpose Input/Output) 
     * ![GPIO Pin](https://cdn.sparkfun.com/assets/learn_tutorials/4/2/4/header_pinout.jpg)
     * Pin29/30 Pin39/40 Pin33/34が、GNDと隣接しているので使いやすい
 * 簡単につかうために WiringPiを用いる
     * Raspbian JESSIEでは標準で導入
     * ```gpio readall``` で現在のGPIOの状態表示。 ```watch -tn 1 gpio readall``` で１秒ごとに自動更新になるので確認に便利。
     * ![GPIO readall](https://raw.githubusercontent.com/WLO-RaspiClub/20170622_IFTTT/master/img/gpio_readall.png)
     * ```gpio write [PIN] [1|0]``` で 書き込み。[PIN]はreadallの時のwPiの番号をつかう。
         * Pin29/30にLEDをつけてる場合は ```gpio write 21 1``` で点灯、 ```gpio write 21 0```で消灯。
     * ```gpio read [PIN]``` で現在の状態を読める。
         * スイッチをGNDにつなげた場合、デフォルトはpull upなので1、押下時はGNDに落ちるので0となる。
     
  
