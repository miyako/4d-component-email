# 4d-component-email
Construct MIME (useful for SMTP_QuickSend from 4D Internet Commands)

## About

With a little bit of MIME, it is possible to send alternative body (plain text+HTML) messages with attachments using ``SMTP_QuickSend``.

## Examples

```
  //$1:charset (TEXT, optional, default=utf-8)
  //$0:MIME
$message:=MIME_New ("utf-8")

  //headers

  //$1:MIME (OBJECT)
  //$2:headerName (TEXT)
  //$3:headerValue (TEXT)
  //$4:charset (TEXT, optional)

MIME_ADD_HEADER ($message;"Subject";"test message")

  //attachment

  //$1:MIME (OBJECT)
  //$2:attachment (->TEXT, ->PICTURE, ->BLOB)
  //$3:fileName (TEXT)
  //$4:MIME type (TEXT)
  //$5:charset (TEXT, optional)
  //$6:content-id (TEXT, optional)
  //$7:content-disposition (TEXT, optional, default=inline)

READ PICTURE FILE(Get 4D folder(Current resources folder)+"sample.png";$image)
MIME_ADD_PART ($message;->$image;"logo.png";"image/png";"";"img1";"inline")

  //alternative body (plain+HTML)

  //$1:MIME (OBJECT)
  //$2:plain text body (TEXT)
  //$3:HTML body (TEXT)
  //$4:charset(TEXT, optional)
$plain:="sample"
$html:="<html><body><strong>sample</strong><img src=\"cid:img1\" /></body></html>"
MIME_ADD_BODY ($message;$plain;$html)

$MIME:=MIME_Export_to_variable ($message)

$hostName:="exchange.4d.com"
$from:="keisuke.miyako@4d.com"
$to:="keisuke.miyako@4d.com"
$user:="keisuke.miyako"
$password:="..."
$param:=8  //MIME+STARTTLS
$port:=587

  //no need for SMTP_SetPrefs, SMTP_Charset (it is implied in the MIME)
$error:=SMTP_QuickSend ($hostName;$from;$to;$subject;$MIME;$param;$port;$user;$password)
ALERT(Choose($error=0;"OK";IT_ErrorText ($error)))
```
