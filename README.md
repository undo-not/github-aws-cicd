# EC2-CodeDeploy-codepipeline-GitHub構築
ちなLAMP構成

## 大まかな流れ  
1. EC2インスタンスの作成

### EC2インスタンスの作成
AWSコンソールのGUIから操作してインスタンスを作成。Ubuntuが良い。AmazonLinuxの場合、yumではなくAmazon-Linux-Extrasというレポジトリからインストールする必要があるので注意。

インスタンス作成時に80番(HTTP)と22番(SSH)ポートを開放し、キーペアを設定する。これはRootユーザ用のもので、開発者や自動デプロイ用のユーザはLinuxから別途払い出すこと。

HTTPSの場合や、接続元を絞る場合のやり方は後で書く

#### サーバーへ接続
とりあえずaptを更新
````bash:ターミナル
sudo apt update
````

とりあえず時間を確認
````bash:ターミナル
date
````

日本に変更
````bash:ターミナル
sudo timedatectl set-timezone Asia/Tokyo
````

タイムゾーンを確認する
````bash:ターミナル
timedatectl
````

#### Apache2.4
* インストール
    ````bash:ターミナル
    sudo apt install apache2
    ````

    ````bash:ターミナル
    # apacheを起動
    $ sudo systemctl restart apache2

    # 確認（active:runningになればOK）
    $ sudo systemctl status apache2

    # システム起動時に毎回起動するように設定
    $ sudo systemctl enable apache2

    # 確認
    $ sudo systemctl is-enabled apache2
    ````

    この時点でパブリックIPをブラウザで直打すればApache2 Ubuntu Default Pageが表示されるはず

* httpd.confの設定  
    UbuntuのApache2ではhttpd.confではなく、複数のconfファイルで管理している
    ````bash:ターミナル
    # 移動
    $ cd /etc/apache2
    ````
    ServerAdminは/etc/apache2/sites-enabledの000-default.conf  
    ServerNameは/etc/apache2/conf-available/fqdn.conf  

    追加の設定は、なるべく追記せず、別のファイルに分けた方が管理しやすい。

    参考
    * https://help.ubuntu.com/lts/serverguide/httpd.html
    * http://www.yamamo10.jp/yamamoto/comp/home_server/WEB_server3/apache/index.php

    ````vi:000-default.conf
    # ServerAdminの変更
    ServerAdmin <<YOUR E-MAIL ADDRESS>>
    ````

    ````vi:fqdn.conf
    # ServerName
    ServerName <<YOUR SERVER NAME>>:80
    ````

    fqdn.confを有効にする
    ````bash:ターミナル
    sudo a2enconf fqdn
    ````

    ページを作成する
    ````bash:ターミナル
    # ドキュメントルートに移動
    cd /var/www/html

    # ファイルを編集
    sudo cp index.html index_org.html
    sudo vi index.html

    <h1>Hello World!</h1>
    ````
    パブリックIPをブラウザで直打ちしてHello World!が表示されていればOK

#### PHP7.2
* インストール
    ````bash:ターミナル
    # インストール可能なものを確認
    $ php -v

    # インストール
    $ sudo apt install php7.2-cli

    # バージョン確認
    $ php -v
    ````
    ページを作成する
    ````bash:ターミナル
    sudo vi /var/www/html/info.php
    ````
    ````vim:/var/www/html/index.php
    <html>
        <body>
            Hello World.<br>
            <?php echo 'hoge'; ?>
        </body>
    </html>
    ````
