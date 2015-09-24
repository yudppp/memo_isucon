ISUCON
==================================

ISUCONのめも

## MySQL

[MySQL :: MySQL 5.1 リファレンスマニュアル :: 4.10.6 ログ ファイルの保守](http://dev.mysql.com/doc/refman/5.1/ja/log-file-maintenance.html)

    grant all privileges on wordpress.* to 'wp_user'@'localhost' identified by 'wp_pass' with grant option;

### mysqldump

    mysqldump -uroot データベース名 > dump.sql
    mysql -uroot データベース名 < dump.sql

### Slow Query

#### 有効にする

```
SET GLOBAL slow_query_log = 1;
show variables like '%slow%';
SET GLOBAL slow_query_log_file = '/tmp/mysql-slow.log';
SET GLOBAL long_query_time = 0.0;
show variables like 'long%';
FLUSH LOGS;
```

#### 無効にする

```
SET GLOBAL slow_query_log = 0;
```

#### pt-query-digest

[Download the Latest Percona Toolkit for Debian and RPM Packages](http://www.percona.com/downloads/percona-toolkit/LATEST/)

    yum localinstall -y http://percona.com/get/percona-toolkit.rpm

（依存も入るけど`sudo yum install -y perl-DBI perl-DBD-MySQL perl-Time-HiRes`で自前で入れることもできる）

Ubuntuなら`aptitude install percona-toolkit`

#### innodb buffer poolを温める

  * [お手軽InnoDBウォームアップを実現するMySQL::Warmerの話をGotanda.pm #2でしてきました | おそらくはそれさえも平凡な日々](http://www.songmu.jp/riji/entry/2014-09-22-gotandapm-mysql-warmer.html)
  * [Kazuho@Cybozu Labs: MySQL のウォームアップ (InnoDB編)](http://labs.cybozu.co.jp/blog/kazuho/archives/2007/10/innodb_warmup.php)
  * [日々の覚書: InnoDB buffer pool dumpで遊ぶ](http://yoku0825.blogspot.jp/2012/08/innodb-buffer-pool-dump.html)
  * [InnoDBのウォームアップに別サーバでdumpしたib_buffer_poolを使ってみる - mikedaの日記](http://mikeda.hatenablog.com/entry/2015/09/21/142746)

```
cpanm MySQL::Warmer
cpanm DBD::mysql
mysql-warmup mydatabase -h db.example.com -u dbuser -p --dry-run
```

`--dry-run`で実行すべきクエリを取得できる。`--dry-run`を付けなければ実行してくれるが、自分の環境では実行できないクエリを実行しようとした。

また時間制限もあるのでどれを実行するかは人間が決めるべき。


## tmpfs

`/etc/fstab`

```
tmpfs  /mnt/tmpfs  tmpfs  defaults,size=8G  0  0
```

## sysctl.conf

`sysctl -p` で適用

  * cannot assign requested はローカルポート
  * ip_conntrack: table full, dropping packet (`dmesg`)

## nginx

[Ruby - ltsv access log summary tool - Qiita](http://qiita.com/edvakf@github/items/3bdd46b53d65cf407fa2)

`parse.rb`を使う

```
cat access.log | ruby parse.rb --since='2015-10-05T02:23' | gist -p
```

60行目の `path = line[:path]` を `gsub` で適当に縮める
（例：`line[:path].gsub(/memo\/(\d+)/, 'memo/:id').gsub(/recent\/(\d+)/, 'recent/:id')`）

`nginx -V` で configure オプション確認

キャッシュがHITしているか確認したい場合はログに `"\tcache_status:$upstream_cache_status` を追加

`/home/isucon` の権限を 755 にすること

### OpenResty

```
sudo aptitude install libreadline-dev libncurses5-dev libpcre3-dev libssl-dev perl make build-essential
./configure --with-pcre-jit --with-luajit --with-http_gzip_static_module

# you need to have ldconfig in your PATH env when enabling luajit. と言われたら
PATH=$PATH:/sbin ./configure --with-pcre-jit --with-luajit --with-http_gzip_static_module
```

`nginx -V`

```
# nginx-full(Ubuntu15.04)
--with-cc-opt='-g -O2 -fPIE -fstack-protector-strong -Wformat -Werror=format-security -D_FORTIFY_SOURCE=2'
--with-ld-opt='-Wl,-Bsymbolic-functions -fPIE -pie -Wl,-z,relro -Wl,-z,now'
--prefix=/usr/share/nginx
--conf-path=/etc/nginx/nginx.conf
--http-log-path=/var/log/nginx/access.log
--error-log-path=/var/log/nginx/error.log
--lock-path=/var/lock/nginx.lock
--pid-path=/run/nginx.pid
--http-client-body-temp-path=/var/lib/nginx/body
--http-fastcgi-temp-path=/var/lib/nginx/fastcgi
--http-proxy-temp-path=/var/lib/nginx/proxy
--http-scgi-temp-path=/var/lib/nginx/scgi
--http-uwsgi-temp-path=/var/lib/nginx/uwsgi
--with-debug
--with-pcre-jit
--with-ipv6
--with-http_ssl_module
--with-http_stub_status_module
--with-http_realip_module
--with-http_auth_request_module
--with-http_addition_module
--with-http_dav_module
--with-http_geoip_module
--with-http_gzip_static_module
--with-http_image_filter_module
--with-http_spdy_module
--with-http_sub_module
--with-http_xslt_module
--with-mail
--with-mail_ssl_module
--add-module=/build/nginx-gKBGMk/nginx-1.6.2/debian/modules/nginx-auth-pam
--add-module=/build/nginx-gKBGMk/nginx-1.6.2/debian/modules/nginx-dav-ext-module
--add-module=/build/nginx-gKBGMk/nginx-1.6.2/debian/modules/nginx-echo
--add-module=/build/nginx-gKBGMk/nginx-1.6.2/debian/modules/nginx-upstream-fair
--add-module=/build/nginx-gKBGMk/nginx-1.6.2/debian/modules/ngx_http_substitutions_filter_module

# OpenResty 1.9.3.1
# ./configure --with-pcre-jit --with-luajit --with-http_gzip_static_module
--prefix=/usr/local/openresty/nginx
--with-cc-opt=-O2
--add-module=../ngx_devel_kit-0.2.19
--add-module=../echo-nginx-module-0.58
--add-module=../xss-nginx-module-0.05
--add-module=../ngx_coolkit-0.2rc3
--add-module=../set-misc-nginx-module-0.29
--add-module=../form-input-nginx-module-0.11
--add-module=../encrypted-session-nginx-module-0.04
--add-module=../srcache-nginx-module-0.30
--add-module=../ngx_lua-0.9.16
--add-module=../ngx_lua_upstream-0.03
--add-module=../headers-more-nginx-module-0.26
--add-module=../array-var-nginx-module-0.04
--add-module=../memc-nginx-module-0.16
--add-module=../redis2-nginx-module-0.12
--add-module=../redis-nginx-module-0.3.7
--add-module=../rds-json-nginx-module-0.14
--add-module=../rds-csv-nginx-module-0.06
--with-ld-opt=-Wl,-rpath,/usr/local/openresty/luajit/lib
--with-pcre-jit
--with-http_gzip_static_module
--with-http_ssl_module
```

## ulimit

`too many open files` はファイルディスクリプタ

[ulimitが効かない不安を無くす設定 | 外道父の匠](http://blog.father.gedow.net/2012/08/08/ulimit-configuration/)

`ulimit -n 65536` が一番良さそう

```/etc/security/limits.conf
isucon hard nofile 65535
isucon soft nofile 65535
```

## gzip

    gzip -r js css
    gzip -k index.html

## supervisord

    sudo supervisorctl status
    sudo supervisorctl reload

環境変数を渡したいとき

```
environment=MARTINI_ENV="production",PORT="8080"
```

## netstat

```
sudo netstat -tlnp
sudo netstat -tnp | grep ESTABLISHED
```

## lsof

```
sudo lsof -nP -i4TCP -sTCP:LISTEN
sudo lsof -nP -i4TCP -sTCP:ESTABLISHED
```

## go

### Martini でログを吐かない

`MARTINI_ENV=production` ではログは消えない

[DSAS開発者の部屋:ISUCON4 予選で workload=5 で 88000点出す方法 (lily white 参戦記)](http://dsas.blog.klab.org/archives/52171878.html)

テンプレートのパース回数が減るらしいので有効にはすべき

```go:app.go
m := martini.Classic()
devnull, err := os.Open(os.DevNull)
if err != nil {
	log.Fatal(err)
}
m.Map(log.New(devnull, "", 0))
```

本当に消したいなら martini のソースコードをいじるしかない


### UNIX domain Socket

```go:app.go
// グローバル変数にしておく
var port = flag.Uint("port", 0, "port to listen")

func init() {
	flag.Parse()
}

// 以下は main() で
sigchan := make(chan os.Signal)
signal.Notify(sigchan, syscall.SIGTERM)
signal.Notify(sigchan, syscall.SIGINT)

var l net.Listener
var err error
sock := "/dev/shm/server.sock"
if *port == 0 {
	ferr := os.Remove(sock)
	if ferr != nil {
		if !os.IsNotExist(ferr) {
			panic(ferr.Error())
		}
	}
	l, err = net.Listen("unix", sock)
	cerr := os.Chmod(sock, 0666)
	if cerr != nil {
		panic(cerr.Error())
	}
} else {
	l, err = net.ListenTCP("tcp", &net.TCPAddr{Port: int(*port)})
}
if err != nil {
	panic(err.Error())
}
go func() {
	// func Serve(l net.Listener, handler Handler) error
	log.Println(http.Serve(l, nil))
}()

<-sigchan
```

### Goアプリケーションの状況を見たい

  * [golang-stats-api-handler/handler.go at master · fukata/golang-stats-api-handler](https://github.com/fukata/golang-stats-api-handler/blob/master/handler.go)

## Gitでpatchファイルを生成する

    git diff --no-prefix HEAD > ~/thisis.patch
    patch --dry-run -p0 < thisis.patch
    patch -p0 < thisis.patch

## おまじない集

### dstat

    dstat -tlamp

これに cpu の状況を確認したいなら `--top-cpu-adv`，IO を確認したいなら `--top-io-adv` でブロッキング IO を確認したいなら `--top-bio-adv` を付ける

### rsync

    rsync -vau -e 'ssh -c arcfour256' /hoge/fuga/ catatsuy.org:/hoge/fuga/

ディレクトリの最後には必ず `/` を付ける

### netstat

    netstat -tlnp

tcp の通信だけ見れる

### 参考 URL

  * [にひりずむ::しんぷる - ngrep 便利！](http://blog.livedoor.jp/xaicron/archives/54419469.html)
  * [dstatの便利なオプションまとめ - Qiita](http://qiita.com/harukasan/items/b18e484662943d834901)
  * [Linux - rsync したいときの秘伝のタレ - Qiita](http://qiita.com/catatsuy/items/66aa402cbb4c9cffe66b)
