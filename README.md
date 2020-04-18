# WebChat
ブラウザ上で動作可能なwebチャットアプリです。
## 参考リンク
- [WebRTC入門2016](https://html5experts.jp/series/webrtc2016/)

## 説明
上記のリンクを参考に作成しました。
上記からの変更点は、
- サーバー上でWebRTCを動作させるため、HTTPS通信に対応
- Websocket通信によるテキストチャット機能を実装

なおTURNは未実装です。

WSサーバー<=(localhost:3002)=>Webサーバー<=(https)=>クライアント

映像と音声はクライアント同士でP2P通信をしています。

## 使い方
1. webサーバーにアクセス（部屋名を指定する場合はURLの末尾に"?部屋名"をつける）
2. play videoを押す
3. 相手がplay videoをしている段階で、connectボタンを押す
4. 切るときはhang up
5. フォームにメッセージを入力してsendを押すとチャットを投稿できる

## 動作に必要なもの
- Webサーバー
    - apache2 v2.4.38
    - 要HTTPS対応（証明書の取得はLet's Encrypeを使用）
    - 要socket .io通信のリバースプロキシ対応
- WSサーバー
    - node.js v12.16.2
    - socket. io v2.3.0(npmによるインストール)
- Webブラウザ
    - ChromeまたはFire Foxで動作確認済み

## ファイルの説明
### multi.html
- クライアント用ファイル
- webサーバーの公開ディレクトリに配置('/var/www/http/'等)

### server/signaling_room.js
- wsサーバー用ファイル
- node.js のディレクトリに配置

## 環境構築手順
1. ファイルの配置

上記の通りにファイルを配置する。
ファイル内のポート番号、ＵＲＬは環境により適宜書き換える。

2. apacheの設定

httpsで送られてきたWS通信をWSサーバーに流すため、リバースプロキシの設定を行う。以下はdebian系の場合。
```bash
/etc/apache2/sites-available/default-ssl.conf
```
< VirtualHost _default_:443>内に以下を追記。(ポート番号が3002のとき)
```bash
# リライトを有効化
RewriteEngine on
# パスが /socket.io/ で始まり、クエリに transport=websocket が含まれているとき、
RewriteCond "%{REQUEST_URI}" "^/socket.io/" [NC]
RewriteCond "%{QUERY_STRING}" "transport=websocket" [NC]
# パスを保持したまま WebSocket プロトコルでローカルホストにプロキシを通す。
RewriteRule "/(.*)" "ws://127.0.0.1:3002/$1" [P,L]
# もちろんフォワードプロキシは無効化
ProxyRequests off
# それ以外のリクエストのうち /socket.io/ で始まるものは、localhost に通す。
ProxyPass "/socket.io/" "http://127.0.0.1:3002/socket.io/"
ProxyPassReverse "/socket.io/" "http://127.0.0.1:3002/socket.io/"
```
apacheを起動する。

3. node.jsの設定、起動

以下のコマンドを入力
```bash
$ cd server
# init
$ npm init
#install socket.io
$ npm install socket.io
# start node.js
$ node signaling_room.js
```