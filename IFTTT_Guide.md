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
     * FRED https://fred.sensetecnic.com/
     * Microsoft Flow https://flow.microsoft.com/ja-jp/
     * MyThings https://mythings.yahoo.co.jp/
     * Playground環境 
  
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
     * 他のサービスのTriggersでGoogle Driveにファイルを書き込めば、間接的にRaspberry Piに情報を送り込める
   
  
