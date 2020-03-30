# EC2-CICD構築
CodeCommit-CodeBuild-CodeDeployをCodePipelineで構築
ちなLAMP

## 大まかな流れ  
1. EC2インスタンスの作成
1. DBの作成(今回は要らない)


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

* ページを作成
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
    # bash上でテストしたいならCLI版
    $ sudo apt install php7.2

    # バージョン確認
    $ php -v
    ````
* ページを作成
    ````bash:ターミナル
    sudo vi /var/www/html/index.php
    ````
    ````vi:/var/www/html/index.php
    <html>
        <body>
            Hello World.<br>
            <?php echo 'hoge'; ?>
        </body>
    </html>
    ````

#### 所有者の変更   
    ubuntuのapacheのユーザとグループはwww-data。apacheではない
    ````bash:ターミナル
    # グループ一覧
    cat /etc/group
    
    # www-dataグループ追加
    $ sudo usermod -a -G www-data ubuntu
    ````
    ssh再接続して、所属グループを確認
    ````bash:ターミナル
    groups
    ````

    ````bash:ターミナル
    sudo chown -R ubuntu:www-data /var/www
    sudo chmod 2775 /var/www
    ````

### DBの作成
今回はCICD構築がメインのため割愛

### CodeCommitの設定
https://docs.aws.amazon.com/ja_jp/codecommit/latest/userguide/welcome.html

開発者はHTTPSとSSHで接続できる。ユーザごとに証明書(ユーザ名とパスワード)発行するか、開発者にssh-keygenしてもらって公開鍵をアップロードするか

#### IAMの設定
アイデンティティベースのポリシー (IAMポリシー) でソースのコミットが出来るようにする

* IAMグループを作る  
    個人ならIAMユーザに直接ポリシー付与してもいいと思うが、チーム開発なら作る

    アタッチするポリシーは「AWSCodeCommit~」系列の中から選ぶ
    * AWSCodeCommitFullAccess

* IAMユーザを作る  
    1. 「プログラムによるアクセス」を選択
    1. グループを選択
    1. シークレットアクセスキーは使わないので削除
    1. 「AWS CodeCommit の HTTPS Git 認証情報」から証明書をダウンロード

    ダウンロードしたCSVファイルにユーザ名とパスワードが記載されている。これを使用してHTTPS接続を行う


#### CodeDeploy


#### CodeCommit


#### CodeBuild



