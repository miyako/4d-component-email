# 4d-component-email
Construct MIME with extended ISO-2022-JP support

MIMEを作成するためのユーティリティです。v15のInternet Commands (QuickSend)には，MIMEをそのまま渡すことができるので，MIME構築をプラグインに頼る必要がありません。MIMEを標準の文字列コマンドだけで構築することにより，Internet Commandsのバグや制限を回避することができます。特に下記の点でInternet Commandsよりも優れています。

* 添付ファイルの二重表示防止

Internet Commandsは``multipart/mixed``で固定ですが，これを``multipart/related``とすることにより，Apple MailでHTMLがContent-IDで参照している画像添付ファイルの二重表示を防止することができます。

* 標準テキストとHTMLの混合メッセージ

Internet Commandsは``text/plain``と``text/html``のどちらかですが，``multipart/alternative``で両方とも含めることができます。

* ISO-2022-JPの処理

Internet CommandsはISO-2022-JPの実装に問題がありますが，これを仕様どおり正確にエンコードすることができます。具体的には，半角カタカナ，「はしご高」や①のような文字も送信することができます。

**注記**: 受信メールクライアントによっては機種依存文字が化けるかもしれません。また，一部のメールサーバー（Microsoft Exchangeなど）は件名を解析してから再エンコードするため，長い件名は（折り返し位置が変わることにより）機種依存文字が化けることがあります。

##Example

MIME作成

```
$sample:="ｱｲｳｴｵ髙橋あいうえお①②③"

$message:=MIME_New ("iso-2022-jp")

  //デフォルトはWindows機種依存文字をそのままISO-2022-JPで送信します。
  //化けるようであれば下記を設定してください。
  //はしご高・丸文字は消えます。半角はそのまま送信できます。
  //MIME_SET_OPTION ($message;"useStandardJISX0208";"1")

MIME_ADD_HEADER ($message;"From";"MIYAKO <keisuke.miyako@4d.com>")
MIME_ADD_HEADER ($message;"To";"MIYAKO <keisuke.miyako@4d.com>")
MIME_ADD_HEADER ($message;"Subject";$sample*100)
MIME_ADD_BODY ($message;$sample*100;"")

$MIME:=MIME_Export_to_variable ($message)

$path:=Temporary folder+Generate UUID+".eml"
TEXT TO DOCUMENT($path;$MIME;"us-ascii")

OPEN URL($path)
```

標準テキスト（ISO-2022-JP）

```
  //標準テキストのメールを送信する

  //半角, はしご高，丸文字など，通常のISO-2022-JPでは送信できな文字，かつ長文（特にヘッダーで化けやすい）
$sample_text:="ｱｲｳｴｵ髙橋あいうえお①②③"*100

  //MIMEオブジェクト
  //$1:文字セット (TEXT, optional, default=utf-8)
  //拡張（Windows独自）ISO-2022-JPマップを内部的に持っているので，扱える文字種が多い
$message:=MIME_New ("iso-2022-jp")

  //ヘッダー
  //$1:MIMEオブジェクト (OBJECT)
  //$2:ヘッダー名 (TEXT)
  //$3:ヘッダー値 (TEXT)
  //$4:文字セット (TEXT, optional)
MIME_ADD_HEADER ($message;"From";"MIYAKO <keisuke.miyako@4d.com>")
MIME_ADD_HEADER ($message;"To";"MIYAKO <keisuke.miyako@4d.com>")
MIME_ADD_HEADER ($message;"Subject";$sample_text)

  //本文
  //$1:MIMEオブジェクト (OBJECT)
  //$2:標準テキスト部 (TEXT)
  //$3:HTMLテキスト部 (TEXT)
  //$4:文字セット (TEXT, optional)
MIME_ADD_BODY ($message;$sample_text;"")

$MIME:=MIME_Export_to_variable ($message)

$hostName:="exchange.4d.com"
$from:="keisuke.miyako@4d.com"
$to:="keisuke.miyako@4d.com"
$user:="keisuke.miyako"
$password:=Account password for 4D
$param:=8  //MIME+STARTTLS
$port:=587

  //成形済みMIMEなのでSMTP_SetPrefs, SMTP_Charsetは不要
$error:=SMTP_QuickSend ($hostName;$from;$to;$subject;$MIME;$param;$port;$user;$password)
ALERT(Choose($error=0;"OK";IT_ErrorText ($error)))
```

標準テキストおよびHTML（UTF-8）

```
  //標準テキストとHTMLの混合メールを送信する

  //MIMEオブジェクト
  //$1:文字セット (TEXT, optional, default=utf-8)
$message:=MIME_New ("utf-8")

  //ヘッダー
  //$1:MIMEオブジェクト (OBJECT)
  //$2:ヘッダー名 (TEXT)
  //$3:ヘッダー値 (TEXT)
  //$4:文字セット (TEXT, optional)
MIME_ADD_HEADER ($message;"From";"MIYAKO <keisuke.miyako@4d.com>")
MIME_ADD_HEADER ($message;"To";"MIYAKO <keisuke.miyako@4d.com>")
MIME_ADD_HEADER ($message;"Subject";"test_"+Generate UUID)

  //添付ファイル
  //$1:MIMEオブジェクト (OBJECT)
  //$2:データ (->TEXT, ->PICTURE, ->BLOB)
  //$3:ファイル名 (TEXT)
  //$4:MIMEタイプ (TEXT)
  //$5:文字セット (TEXT, optional)
  //$6:content-id (TEXT, optional)
  //$7:content-disposition (TEXT, optional, default=inline)

READ PICTURE FILE(Get 4D folder(Current resources folder)+"icon.png";$image)
MIME_ADD_PART ($message;->$image;"logo.png";"image/png";$charset;"img1";"inline")

  //本文
  //$1:MIMEオブジェクト (OBJECT)
  //$2:標準テキスト部 (TEXT)
  //$3:HTMLテキスト部 (TEXT)
  //$4:文字セット (TEXT, optional)
$plain:="sample"
$html:="<html><body><strong>sample</strong><img src=\"cid:img1\" /></body></html>"
MIME_ADD_BODY ($message;$plain;$html)

$MIME:=MIME_Export_to_variable ($message)

$hostName:="exchange.4d.com"
$from:="keisuke.miyako@4d.com"
$to:="keisuke.miyako@4d.com"
$user:="keisuke.miyako"
$password:=Account password for 4D
$param:=8  //MIME+STARTTLS
$port:=587

  //成形済みMIMEなのでSMTP_SetPrefs, SMTP_Charsetは不要
$error:=SMTP_QuickSend ($hostName;$from;$to;$subject;$MIME;$param;$port;$user;$password)
ALERT(Choose($error=0;"OK";IT_ErrorText ($error)))
```
