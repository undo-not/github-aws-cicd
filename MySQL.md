# MySQLインストール
RDBを使わずに直接インストールする手順
* インストール
    ````bash:ターミナル
    # インストール
    sudo apt install -y mysql-server-5.7

    # バージョン確認
    mysql  --version
    ````

* パスワード設定  
    Access denied for user ‘root’@’localhost’が出る場合は下記のコマンドも実行する
    ````bash:ターミナル
    sudo service mysqld stop

    # MySQLの権限システムを使用しないで起動
    sudo mysqld_safe --skip-grant-tables

    mysql -u root
    ````
    ````sql:mysqlコマンドプロンプト
    use mysql;

    select User from user;

    truncate table user;

    flush privileges;

    grant all privileges on *.* to root@localhost identified by '<password>' with grant option;    

    quit;
    ````

    ````bash:ターミナル
    mysql -u root -p
    ````