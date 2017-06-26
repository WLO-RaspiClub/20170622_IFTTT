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
     * https://github.com/WLO-RaspiClub/20170622_IFTTT/blob/master/pocket.md
 * Google Driveとの連携例(後述)
 * LINE Notify https://notify-bot.line.me/ja/ なども便利
     
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
     * ```LED、スイッチをつけているGPIOピンのモードをoutに指定する必要あり。
             gpio [-g] mode ピン番号 モード
          Pin29/30にLEDをつけてる場合は、 gpio write 21 out　してから下記をためす
     * ```gpio write [PIN] [1|0]``` で 書き込み。[PIN]はreadallの時のwPiの番号をつかう。
         * Pin29/30にLEDをつけてる場合は ```gpio write 21 1``` で点灯、 ```gpio write 21 0```で消灯。
     * ```gpio read [PIN]``` で現在の状態を読める。
         * スイッチをGNDにつなげた場合、デフォルトはpull upなので1、押下時はGNDに落ちるので0となる。
 
### スイッチでIFTTTのTriggers起動
 * smart-button.pl https://github.com/obgm/smart-button
     * インストール (スクリプト１つだけなので)
         * ```wget https://raw.githubusercontent.com/obgm/smart-button/master/smart-button.py```
         * ``` chmod +x smart-button.py```
         * サービス化するとでRaspberry Pi起動時に自動起動させることもできる。詳細は https://github.com/obgm/smart-button#using-as-a-system-service 
     * ためしてみる
         * ``` ./smart-button.py -P 5 -t 2 -l 'echo button' ```
         * sudoしてないので警告が表示されるが、piユーザなら動くはず
         * ```-P 5``` はGPIOポートの指定。```gpio readall```のBCMの番号で指定
             * Pin29/30にスイッチを接続した場合は```-P 5```
         * ```-t 2``` は長押し時間指定(指定秒以上の押下で-lのコマンドが動く)
             * -c にコマンド指定すると、長押し時間以下で動作させられる 
     * IFTTTの認証鍵確認
         * https://ifttt.com/maker_webhooks の右上の「Documentation」ボタンの先のページに記載あり
         * ![Trigger](https://raw.githubusercontent.com/WLO-RaspiClub/20170622_IFTTT/master/img/ifttt_maker_key.png)
     * ボタン押下でトリガ起動
         * ``` ./smart-button.py -P 5 -t 2 -l 'echo button;curl -X POST https://maker.ifttt.com/trigger/button/with/key/__Your__IFTTT__TRIGGER__KEY__ ;echo' ```
             * ```button```にはIFTTTのMaker WebhooksでEvent Nameに指定する文字列を入れる（後述）
             * ```__Your__IFTTT__TRIGGER__KEY__```のところにIFTTTの認証鍵を記載する
             * ``` Congratulations! You've fired the button event ``` と表示されたらトリガ完了。
             * ```sudoでないと動作しない？            

### IFTTTのTriggersでGoogle Driveのファイルを更新する
 * Google Driveにログインした状態でIFTTTのアカウント連携
     * IFTTTのsearch https://ifttt.com/search でGoogle Driveを指定
     * Connect ボタン押下
     * (Googleアカウント複数ある場合は)アカウント選択ダイアログで選択
     * 連携内容表示されるので「許可」
     * https://ifttt.com/google_drive の右上にSettingsが表示されたら完了
 * Appletを作成
     * My Applets https://ifttt.com/my_applets 右上の New Appletを押下
     * if + this then thatの 「+ this」を押下
     * Choose a service でMaker Webhooksを検索
     * 「Recieve a web request」選択
     * Event Nameにbuttonと入れてCreate Trigger
         * smart-button.py の-lのcurlのURL内の「button」に相当
     *  if + this then thatの 「+ that」を押下
     * Choose action service で google driveを検索
     * Append to a documentを選択
     * Action内容設定
         * 指定内容にはingredient(成分)が指定できる
             * 中括弧で囲んで指定。 Add ingredientボタンで、指定したTriggerで使えるingredientのヒントが表示
         * Document nameにはファイル名 (LED1.txtとか)
         * Contentには追加する文字列(```When: {{OccurredAt}}```でTriggerのイベント日時)
         * Drive folder pathにGoogle Driveのpath
         * Create Actionボタン押下
     * Review and finish で内容確認して、Finishボタン押下で完了

### Google Driveのコンテンツ更新でLEDを点灯する
#### 方針
 * Raspberry Piから定期的にGoogle Driveのファイルを取得する（ポーリング）
     * IFTTTのGoogle Drive ServiceのActions「Append to a document」は、Google Docsのファイルを作成する。
     * ファイルの「共有可能なリンクを取得」で共有して、Raspberry Piからcurl(コマンドラインhttpクライアント)でテキストとして取得
     * Actions毎に行が増えるので、行数が偶数か奇数かでLEDの点灯/消灯を決める
     
#### 手順
 * Google DriveのファイルIDを取得する
     * ![GoogleDrive01](https://raw.githubusercontent.com/WLO-RaspiClub/20170622_IFTTT/master/img/gdrive01.png)
     * ![GoogleDrive02](https://raw.githubusercontent.com/WLO-RaspiClub/20170622_IFTTT/master/img/gdrive02.png)
     * URLをコピーして、id=以降の文字列を確認、テキストエディタ等に保存しておく。
 * 以下のシェルスクリプトを作成(vi polling.sh)　
     * GDRIVEID=の行に取得しておいたGoogle DriveのファイルIDを記載する
 ```polling.sh
 #!/bin/sh
GDRIVEID="___googleDriveFileIDStrings___"
GDRIVEURL="https://docs.google.com/document/u/1/export?format=txt&id=${GDRIVEID}"
while true
do
  LINES=$(curl -s -L -o - ${GDRIVEURL} | wc -l)
  ONOFF=$(( ${LINES} % 2))
  echo -n ${ONOFF}
  echo -n " "
  gpio write 29 ${ONOFF}
  sleep 5
done
```
 * ターミナルで実行すると、12秒周期(取得7秒+sleep5秒)程度でGoogle Driveにアクセスして、点灯/消灯する
     * ```sh polling.sh```
     
     
 
 
             
  
