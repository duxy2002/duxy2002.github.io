# Expires レスポンスヘッダー

## Expiresヘッダとは
ExpiresヘッダはレスポンスヘッダのひとつでApacheやNginxなどのWEBサーバー側で適切な設定をすることによって、
新しいファイルが存在しているかどうかを確認することなく、ブラウザでキャッシュ済みのファイルを強制的に適用するEtagやIMSより優先度が高いヘッダです。
キャッシュ済みのファイルを強制的に読み込む期間を指定するため、
Expiresヘッダで指定した期間は強制的にコンピュータ内に保存されたキャッシュファイルを読み込むことになります。

## Expires はキャッシュの期限切れ日時を指定
Expires ヘッダーはコンテンツキャッシュの期限をコントロールし、 HTTP 1.0 から存在している。

> RFC 2616 “Hypertext Transfer Protocol — HTTP/1.1” では次のように書かれている
> 
> 
> The Expires entity-header field gives the date/time after which the response is considered stale.
> http://tools.ietf.org/html/rfc2616#section-14.21

expiration time は HTTP-Date 形式(rfc1123-date | rfc850-date | asctime-date)でかく。

例) Expires: Thu, 01 Dec 1994 16:00:00 GMT

## 不正な日時
   
   問題は、なんで日時のフィールドに “-1” なんていう数字が設定されているのか?
   
   
   HTTP/1.1 clients and caches MUST treat other invalid date formats, especially including the value “0”, as in the past (i.e., “already expired”).
   http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html
   
   ということで、Expires に HTTP-Date 以外の形式が設定されると、”already expired” とみなされてキャッシュされない。
    「過去」を動的に計算するのはかったるいので、シンプルに「-1」決め打ちにしているのだろう。
   
   なお、未来の日付については “HTTP/1.1 servers SHOULD NOT send Expires dates more than one year in the future.” というように制限されているが、過去の日付については、そのような記述がないからなのか、多様な過去の日付が使われている。

## Expires と Cache-Control の違い
HTTP/1.1 ではキャッシュをより細かく制御するために Cache-Control ヘッダーが追加され、max-age でキャッシュ期間を設定できる。
 例えば Cache-Control: max-age=300 とすると、クライアントに 300秒はキャッシュするように促す。
max-age と expires の両方が設定されていた場合 max-age を優先する。

> If a response includes both an Expires header and a max-age directive, the max-age directive overrides the Expires header, 
> even if the Expires header is more restrictive.
> http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html

IETF Httpbis ワーキンググループの一人で、”Caching Tutorial” などでも有名な Mark Nottingham は
•ライブラリーを使って expires と Cache-Control:max-age の両方を設定する(Apache の mod_expires、 nginx の ngx_http_headers_module など)
•Cache-Control: max-age だけを設定する

のいずれかを推奨している。

## 実際の例

ここからは、実際の Web サーバがどのような Expires/Cache-Control を返しているか確認。

```
$ HEAD http://www.amazon.co.jp
200 OK
Cache-Control: no-cache
Connection: Keep-Alive
Date: Sat, 01 Feb 2014 04:32:47 GMT
Pragma: no-cache
Server: Server
Vary: Accept-Encoding,User-Agent
Content-Type: text/html; charset=Shift_JIS
Expires: -1
Client-Date: Sat, 01 Feb 2014 04:38:36 GMT
Client-Peer: 54.240.250.0:80
Client-Response-Num: 1
Keep-Alive: timeout=2, max=6
Set-Cookie: skin=noskin; path=/; domain=.amazon.co.jp
X-Amz-Id-1: 1G198XTD4DPPTNQP21KE
X-Amz-Id-2: cHJjNQk6G7uTB7JbLr83QfJ9XTssZwUAnkzLTWxuuGYuiE3MdXXS1SuHl9dVeC85

Cache-Content: no-cache
Expires: -1

# reddit
$ HEAD  http://www.reddit.com/r/gaming/comments/1wog7g/wtf_fifa/
200 OK
Cache-Control: private, must-revalidate, max-age=0
Connection: close
Date: Sat, 01 Feb 2014 04:32:30 GMT
Server: '; DROP TABLE servertypes; --
Vary: accept-encoding
Content-Type: text/html; charset=UTF-8
Last-Modified: Sat, 01 Feb 2014 04:30:56 GMT
Client-Date: Sat, 01 Feb 2014 04:38:19 GMT
Client-Peer: 125.56.218.96:80
Client-Response-Num: 1
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
X-Xss-Protection: 1; mode=block

Cache-Control: max-age=0
Expires: blank

$ HEAD 'http://www.youtube.com/watch?v=uE7qVAtNwQk'
200 OK
Cache-Control: no-cache
Connection: close
Date: Sat, 01 Feb 2014 04:32:10 GMT
Server: gwiseguy/2.0
Content-Type: text/html; charset=utf-8
Expires: Tue, 27 Apr 1971 19:44:06 EST
Alternate-Protocol: 80:quic
Client-Date: Sat, 01 Feb 2014 04:38:00 GMT
Client-Peer: 74.125.235.160:80
Client-Response-Num: 1
P3P: CP="This is not a P3P policy! See http://support.google.com/accounts/bin/answer.py?answer=151657&hl=ja-JP for more info."
Set-Cookie: YSC=hEFCdovBRI0; path=/; domain=.youtube.com; httponly
Set-Cookie: PREF=f1=50000000; path=/; domain=.youtube.com; expires=Thu, 02-Oct-2014 16:25:10 GMT
Set-Cookie: VISITOR_INFO1_LIVE=DnbZPuvLZ5I; path=/; domain=.youtube.com; expires=Thu, 02-Oct-2014 16:25:10 GMT
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
X-XSS-Protection: 1; mode=block; report=https://www.google.com/appserve/security-bugs/log/youtube

Cache-Control: no-cache
Expires: Tue, 27 Apr 1971 19:44:06 EST

$ HEAD http://stackoverflow.com/questions/18947314/how-to-add-a-linked-source-folder-in-android-studio
200 OK
Cache-Control: public, max-age=60
Connection: close
Date: Sat, 01 Feb 2014 04:33:35 GMT
Vary: *
Content-Length: 63568
Content-Type: text/html; charset=utf-8
Expires: Sat, 01 Feb 2014 04:34:36 GMT
Last-Modified: Sat, 01 Feb 2014 04:33:36 GMT
Client-Date: Sat, 01 Feb 2014 04:39:26 GMT
Client-Peer: 198.252.206.16:80
Client-Response-Num: 1
X-Frame-Options: SAMEORIGIN

Cache-Control: max-age=60
Expires: Sat, 01 Feb 2014 04:34:36 GMT

$ HEAD http://エロサイトコンテンツ
200 OK
Cache-Control: public, s-maxage=1260
Connection: close
Date: Sat, 01 Feb 2014 04:31:50 GMT
Via: 1.1 varnish
Age: 0
Server: nginx
Content-Type: text/html; charset=UTF-8
Client-Date: Sat, 01 Feb 2014 04:37:40 GMT
Client-Peer: 31.192.116.24:80
Client-Response-Num: 1
Set-Cookie: sid=4187441776523501568; path=/; expires=Tue, 31-Dec-2030 12:00:00 GMT; domain=.エロドメイン.com
Set-Cookie: is_pc=1; path=/; expires=Tue, 31-Dec-2030 12:00:00 GMT; domain=.エロドメイン.com
Set-Cookie: country=JP; path=/; domain=.エロドメイン.com
X-Varnish: 3954156791

Cache-Control: s-max-age=1260(varnish 向け)
Expires: blank
Varnish あり

$ HEAD http://en.wikipedia.org/wiki/HTTP_2.0
200 OK
Cache-Control: private, s-maxage=0, max-age=0, must-revalidate
Connection: close
Date: Sat, 01 Feb 2014 04:33:16 GMT
Via: 1.1 varnish, 1.1 varnish
Age: 5700
Server: Apache
Vary: Accept-Encoding,Cookie
Content-Language: en
Content-Type: text/html; charset=UTF-8
Last-Modified: Mon, 06 Jan 2014 09:29:06 GMT
Client-Date: Sat, 01 Feb 2014 04:39:06 GMT
Client-Peer: 208.80.154.224:80
Client-Response-Num: 1
X-Cache: cp1067 hit (5), cp1054 frontend hit (1)
X-Content-Type-Options: nosniff
X-Powered-By: PHP/5.3.10-1ubuntu3.9+wmf1
X-Varnish: 2829631474 2827122309, 1251628819 1244997693
X-Vary-Options: Accept-Encoding;list-contains=gzip,Cookie;string-contains=enwikiToken;string-contains=enwikiLoggedOut;string-contains=forceHTTPS;string-contains=enwikiSession;string-contains=centralauth_Token;string-contains=centralauth_Session;string-contains=centralauth_LoggedOut;string-contains=mf_useformat;string-contains=stopMobileRedirect

Cache-Control: max-age=0, s-maxage=0(Varnish 向け)
Expires: blank
Varnish あり

$ HEAD http://www.1101.com/suimin/samma/index.html
200 OK
Connection: close
Date: Sat, 01 Feb 2014 04:35:02 GMT
Via: 1.1 varnish
Age: 680137
ETag: "11f057d-b69-4480cd52ac500"
Server: Apache
Content-Type: text/html
Last-Modified: Mon, 10 Mar 2008 03:31:00 GMT
Client-Date: Sat, 01 Feb 2014 04:40:51 GMT
Client-Peer: 116.214.86.27:80
Client-Response-Num: 1
X-Varnish: 2679230077 2574494562

Cache-Control: max-age=0
Expires: blank
Varnish あり

$ HEAD http://cookpad.com/pr/contest/index/495
200 OK
Cache-Control: max-age=0, private, must-revalidate
Connection: Close
Date: Sat, 01 Feb 2014 04:33:59 GMT
ETag: "84f438827d94afb908434e1a21064610"
Server: nginx
Vary: User-Agent
Content-Type: text/html; charset=utf-8
Client-Date: Sat, 01 Feb 2014 04:39:49 GMT
Client-Peer: 54.249.90.123:80
Client-Response-Num: 1
Set-Cookie: cpb=00ncva0n099ac3e2370668444ea0a1209980bf9ecc7cb0036f; path=/; expires=Tue, 19-Jan-2038 03:14:07 GMT
Set-Cookie: v=315-4521644-9159558; path=/; expires=Tue, 19-Jan-2038 03:14:07 GMT
Set-Cookie: FVD=%7Bts+%272014-02-01+13%3A33%3A59%27%7D; path=/; expires=Mon, 03-Mar-2014 04:33:59 GMT
Set-Cookie: f_unique_id=15632df6-4323-464f-81a2-b1889828c59c; path=/; expires=Wed, 01-Feb-2034 04:33:59 GMT
Set-Cookie: cids=495; path=/
Set-Cookie: user_type=0; path=/; expires=Mon, 03-Mar-2014 04:33:59 GMT
Set-Cookie: has_kitchen=0; path=/; expires=Mon, 03-Mar-2014 04:33:59 GMT
Set-Cookie: trial_type=0; path=/; expires=Mon, 03-Mar-2014 04:33:59 GMT
Set-Cookie: _myapp_session=BAh7B0kiD3Nlc3Npb25faWQGOgZFVEkiJTQxMzViNmY5NmIzYTVkNGM2MWM5ZTkwMjhhNzk2OTVhBjsAVEkiEF9jc3JmX3Rva2VuBjsARkkiMW9IRFc3THUyUkNnbWRLdXNGb1MrSW9Yd2djSzN6YnU0azcrcjhSbytxOUk9BjsARg%3D%3D--4ca3911027db73825d5fd98b353ff2cb11578e16; path=/; HttpOnly
Set-Cookie: withoutvns=; path=/; expires=Thu, 01-Jan-1970 00:00:00 GMT
Status: 200 OK
X-ASC: 159
X-ASN: 159
X-Chst: 32a6df64098aa9ada256c6e881941985
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
X-Request-Id: d87e903120c048d56784636a9c608a45
X-Runtime: 0.287018
X-XSS-Protection: 1; mode=block

Cache-Control: max-age=0
Expires: blank

$ HEAD http://www.w3.org/Protocols/rfc2616/rfc2616.html
200 OK
Cache-Control: max-age=21600
Connection: close
Date: Sat, 01 Feb 2014 04:34:20 GMT
Accept-Ranges: bytes
ETag: "805b-3e3073913b100"
Server: Apache/2
Content-Length: 32859
Content-Type: text/html; charset=iso-8859-1
Expires: Sat, 01 Feb 2014 10:34:20 GMT
Last-Modified: Wed, 01 Sep 2004 13:24:52 GMT
Client-Date: Sat, 01 Feb 2014 04:40:10 GMT
Client-Peer: 128.30.52.37:80
Client-Response-Num: 1
P3P: policyref="http://www.w3.org/2001/05/P3P/p3p.xml"

Cache-Control: max-age=21600
Expires: Sat, 01 Feb 2014 10:34:20 GMT

$ HEAD  http://tokyo.craigslist.jp/act/4313870292.html
200 OK
Cache-Control: max-age=300, public
Connection: close
Date: Sat, 01 Feb 2014 03:49:13 GMT
Server: Apache
Vary: Accept-Encoding
Content-Type: text/html; charset=iso-8859-1
Expires: Sat, 01 Feb 2014 03:54:13 GMT
Last-Modified: Sat, 01 Feb 2014 03:49:13 GMT
Client-Date: Sat, 01 Feb 2014 04:49:50 GMT
Client-Peer: 208.82.238.226:80
Client-Response-Num: 1
Client-Transfer-Encoding: chunked
X-Frame-Options: SAMEORIGIN
X-MCP-Cache-Control: max-age=2592000, public

Cache-Control: max-age=300
Expires: Sat, 01 Feb 2014 03:54:13 GMT

$ HEAD http://nginx.org/en/docs/http/ngx_http_headers_module.html
200 OK
Connection: close
Date: Sat, 01 Feb 2014 04:34:46 GMT
Accept-Ranges: bytes
ETag: "528dfca7-1f0c"
Server: nginx/1.5.7
Content-Length: 7948
Content-Type: text/html; charset=utf-8
Last-Modified: Thu, 21 Nov 2013 12:29:27 GMT
Client-Date: Sat, 01 Feb 2014 04:40:35 GMT
Client-Peer: 206.251.255.63:80
Client-Response-Num: 1

Cache-Control: blank
Expires: blank
```

## Expiresヘッダの使いどころ
Expiresヘッダは指定された期日が過ぎるまで新しいコンテンツを取りに行かないという仕様のため、頻繁に更新されるようなコンテンツには適していません。しかし、逆にIf modified Sinceも送られないため、304レスポンス負荷が発生することもありません。そこで、ＥｔａｇとLast-Modifiedが既に付与されている場合のＥｘｐｉｒｅｓヘッダの期日は状況によって以下の通り調整すべきです。

* 極力ブラウザにキャッシュさせ、コンテンツ更新判定をさせたい

Ｅｘｐｉｒｅｓヘッダが付与されている場合、ＥｔａｇとIf-Modified-Sinceを上書きしてしまいます。
つまり、最新のコンテンツをサーバーから取りに行くことをせずＥｘｐｉｒｅｓヘッダに付与されている期間まで
ブラウザのキャッシュを読みにいってしまいます。

極力ブラウザにキャッシュさせつつも、更新判定を行いたい場合は、Ｅｘｐｉｒｅｓヘッダに過去の日付を指定すれば問題ありません。

```
サーバー側のレスポンスヘッダ
Expires "Mon, 26 Jul 1997 05:00:00 GMT"
```

ブラウザ毎に片方しかサポートしていない場合があるので、Cache-Control "max-age=0"というヘッダも合わせて付けておいた方が良いでしょう。
この場合、ブラウザ側ではＥｘｐｉｒｅｓヘッダをみて既に期限が切れていることを確認し、
つぎに条件つきリクエスト（If-Modified-Since、If-None-Match）をサーバへ送ります。

サーバー側は、送られてきたIf-Modified-Since、If-None-Match でコンテンツの変更ありなしを判断して 304レスポンスを返します。
サーバーから304レスポンスを受け取った場合、ブラウザはローカルのキャッシュを利用するという動作になります。

ただし条件つきリクエストを送ってくるのは、前回にサーバから Last-Modified、Etag を送ってある場合のみです。
またサーバ側は、もし If-Modified-Since、If-None-Match がクライアントから送られてきたら、
変更ありなしを判断して 304レスポンスを返せば通信量が減らせます
（Apacheなど普通のWEBミドルウェアなら静的コンテンツの場合は勝手に処理してくれるでしょうけど）。
サーバから304レスポンスを受け取った場合、ブラウザはローカルのキャッシュを利用します。

* Expires: -1について

過去の日付を指定すれば期限切れと判断されます。しかし、毎回過去の日付をしらべてられないですよね。
そこで、共通のある値を指定することでどのようなコンテンツであれ、期限切れと定義させる方法があります。

RFCでは以下のように定義されています。


> HTTP/1.1 clients and caches MUST treat other invalid date formats, especially including the value “0”, as in the past (i.e., “already expired”).
> http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html

このように-1というHTTP-DATE以外の値を指定してあげて不正な日時と判断させ、結果期限切れとしてしまう荒技もあります。

 

* 強制的にキャッシュさせたい場合

先ほどの例は少し矛盾しています、なぜならそもそも３０４レスポンスを返してほしければ、Expiresヘッダを付与すべきではないからです。
Expiresヘッダを付与する場合はむしろ、「強制的にキャッシュさせたい」といった場合に利用されることのほうが多いでしょう。
ブラウザにキャッシュさせる命令をExpiresヘッダで行うのは簡単です。
```
サーバー側のレスポンスヘッダ
Expires "Mon, 26 Jul 2016 05:00:00 GMT"
```

かなり未来の日付を設定していますが、このようにすれば指定期日までサーバー自身にリクエストを送信しないため、サーバー側の負荷が大幅に下がります。
 気をつけなければいけない点は、１年以上先の日付は推奨されていないことです。１０年とか１００年先とかはやめましょう。

* ファイルの最終更新日以降指定期間だけ有効

つぎは、ファイルの最終更新日が過ぎてから特定の期間だけサーバーにアクセスさせないようなトリッキーな設定も可能です。

Apacheの場合以下のように設定します。
``` 
ExpiresDefault "modification plus 5 minutes"
 ```
 
このようにするとファイルの最終更新日から５分間はサーバーに問い合わせないという動作が可能です。

* キャッシュさせたくない場合

そもそも、ブラウザにキャッシュさせたくない場合は、Expiresヘッダは付与すべきではありません。
またETAGやLast-Modifiedも削除すべきです。しかし、手軽に実現出来る一番簡単な方法は以下のキャッシュコントロールヘッダを付与することです。
```html
Cache-Control "no-cache"
```
no-cacheを付与することによって、毎回最新のコンテンツを取りに行くことになります。
こちらはアクセス毎に内容が変更されたりサーバーにアクセスしてもらわないと困るようなコンコンテンツの場合、
特にスクリプト言語等で生成する動的コンテンツなどに適しています。
アクセス毎に内容が変わったり、サーバにアクセスしてもらわないと困るようなコンコンテンツの場合です。
スクリプト言語等で生成する動的コンテンツは、このようにした方が安全です。

## Expiresヘッダの注意点

最後にExpiresヘッダの注意点をいくつかまとめます。

* Expiresの日付は1週間以上に設定すること。
* RFCのガイドラインに違反するので、1年以上先には設定しないこと。
* EtagやLast-Modifiedより強いキャッシュなので更新の多いページや動的更新されるページでは利用しない。
* サーバーに問い合わせないためアクセスログも収集出来なくなる。

特に最後のアクセスログが収集出来ないというのは状況によって致命的になります。しっかり検討してExpiresヘッダを付与しましょう。

HTTP/1.1 ではキャッシュをより細かく制御するために Cache-Control ヘッダーが追加されより柔軟なキャッシュコントロールが可能となりました。次回はそのあたりを詳しく説明していきます。


## 補足

iPhoneやAndoid等のネィティブアプリ内のWebViewはキャッシュが強いので、サーバからのキャッシュコントロールに関わらず、ローカルのキャッシュが利用されることがあります（恐らくWebViewの設定でキャッシュポリシーを変更できるかとは思います）。
