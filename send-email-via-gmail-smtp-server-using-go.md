# How to send an E-mail via GMail SMTP Server using GO

**1. INTRODUCTION**

Using GMail SMTP Server you can send E-mails to any domain using your Gmail Credentials. Following are some Emails sending limit criterias.
+ Google limits the number of recipients in a single Email and number of Emails can be sent per day.
+ Current limit is 500 Emails in a day or 500 recipients in a single Email.
+ On reaching threshold limits, You can not send messages for 1 to 24 hours.
+ After Suspension Period counters will get reset automatically and the user can resume sending Emails.
+ For more information about Email sending limits refer following links:
  - Link 1: [Email sending limits](https://support.google.com/a/answer/166852)
  - Link 2: [Error messages once limit is crossed](https://support.google.com/mail/answer/22839)

SMTP/NET Package implements the Simple Mail Transfer Protocol as defined in RFC 5321.
```
func SendMail(addr string, a Auth, from string, to []string, msg []byte) error
```
<br>

**2. PARAMETERS**
+ **addr**  is a Host Server Address along with Port Number separated by Column ':'
+ **a** is a Authentication response from Gmail
+ **from** is an Email Address using which we are authenticating and sending Email
+ **to** can a single Email Address or an array of Email Address to which we want to send an Email
+ **msg** 
  - This parameter should be an RFC 822-style with headers first, a blank line, and then the message body.
  - The content should be CRLF terminated.
  - The headers part includes fields such as "From", "To", "Subject", and "Cc".
  - Sending "Bcc" messages is accomplished by including an email address in the to parameter but not including it in the msg headers. 
  - This function and the net/smtp package are low-level mechanisms and do not provide support for DKIM signing, MIME attachments and  other features.

<br>

**3. SETTINGS**

**3.1:** Before sending Emails using Gmail SMTP Server, Change the required setting using Google Account Security Settings or [Click Here](https://myaccount.google.com/security)

![Google Account Security Settings](https://i.imgur.com/6Hxmb2G.png))

**3.2:** Make sure that 2-Step-Verification is Disabled

![2-Step Virification Disabled](https://i.imgur.com/6Hxmb2G.png)

**3.3:** Turn ON the Less Secure App Access or [Click Here](https://myaccount.google.com/u/0/lesssecureapps)

![Less Secure App Access](https://i.imgur.com/hymkYJ6.png)

**3.4:** If 2-Step-Verification is Enabled, then you will have to create APP Password for your application or device.

![2-Step Virification Enabled](https://i.imgur.com/vcQYoGo.png)

![Generate App Password](https://i.imgur.com/LHfCxdH.png)

**3.5:** For security precaution, Google may require you to complete this additional step while signing-in. [Click Here](https://accounts.google.com/DisplayUnlockCaptcha) to Allow access to your Google account using new Device/App.

![New Device-App](https://i.imgur.com/mEGa22F.png)

==**Note**: It may take an hour or more to reflect any security changes==

<br>

**4. CODE - EMAIL AS HTML**

[Click here](https://github.com/gaurangmacharya/pepithon/blob/master/send-email-via-gmail-smtp-server-using-go.go) To download complete code for sending Email as HTML

**4.1:** Import required packages
- [log](https://golang.org/pkg/log/) :: log.Print() to print important stages and errors
- [fmt](https://golang.org/pkg/fmt/) :: fmt.Sprintf() To print formatted text
- [net/smpt](https://golang.org/pkg/net/smtp/) :: smtp.PlainAuth() is to authenticate account and smtp.SendMail() is to send Email using SMTP Protocol
- [mime/quotedprintable](https://golang.org/pkg/mime/quotedprintable/) :: quotedprintable.NewWriter() Is convert Email body in "Quoted Printable" Format. [Click here](https://en.wikipedia.org/wiki/Quoted-printable) To know more about the format.
``` Go
package main
import (
    "log"
    "fmt"
    "bytes"
    "net/smtp"
    "mime/quotedprintable"
)
```
**4.2:** Set required parameters to authenticating access to SMTP
``` go
from_email:= "from-email@domain"
password  := "gmail-app-password"
host      := "smtp.gmail.com:587"
auth      := smtp.PlainAuth("", from_email, password, "smtp.gmail.com")
```
**4.3:** Set required Email header parameters like From, To and Subject
``` go
header := make(map[string]string)
to_email        := "recipient-email@domain"
header["From"]   = from_email
header["To"]     = to_email
header["Subject"]= "Write Your Subject Here"
```
**4.4:** Set header parameters to define type of Email content.
``` go
header["MIME-Version"]              = "1.0"
header["Content-Type"]              = fmt.Sprintf("%s; charset=\"utf-8\"", "text/html")
header["Content-Disposition"]       = "inline"
header["Content-Transfer-Encoding"] = "quoted-printable"
```
**4.5:** Prepare Formatted header string by looping all Header parameters.
``` go
header_message := ""
for key, value := range header {
    header_message += fmt.Sprintf("%s: %s\r\n", key, value)
}
```
**4.6:** Prepare Quoted-Printable Email body. 
``` go
body := "<h1>This is your HTML Body</h1>"
var body_message bytes.Buffer
temp := quotedprintable.NewWriter(&body_message)
temp.Write([]byte(body))
temp.Close()
```
**4.7:** Prepare final Email message by concatenating header and body.
``` go
final_message := header_message + "\r\n" + body_message.String()
```
**4.8:** Send Email and print log accordingly
``` go
status  := smtp.SendMail(host, auth, from_email, []string{to_email}, []byte(final_message))
if status != nil {
    log.Printf("Error from SMTP Server: %s", status)
}
log.Print("Email Sent Successfully")
```
<br>

**5. CODE EMAIL AS TEXT**
``` go
package main
import (
    "log"
    "net/smtp"
)
func main() {
    to_email     := "recipient-email@domain"
    from_email   := "from-email@domain"
    password     := "gmail-app-password"
    subject_body := "Subject: Write Your Subject\n\n" + "This is your Email Body"
    status       := smtp.SendMail("smtp.gmail.com:587", smtp.PlainAuth("", from_email, password, "smtp.gmail.com"), from_email, []string{to_email}, []byte(subject_body))
    if status != nil {
        log.Printf("Error from SMTP Server: %s", status)
    }
    log.Print("Email Sent Successfully")
}
```
<br>

**6. ERRORS**

Following are some of errors which you may encounter while testing Gmail SMTP Module

**6.1:**. If you have entered wrong credentials
```
2019/09/18 12:21:51 Error from SMTP Server: 535 5.7.8 Username and Password not accepted. Learn more at
5.7.8  https://support.google.com/mail/?p=BadCredentials c8sm5954072pfi.117 - gsmtp
```
**6.2:**. If you have not enabled App Password
```
2019/09/18 11:46:49 Error from SMTP Server: 534 5.7.9 Application-specific password required. Learn more at
5.7.9  https://support.google.com/mail/?p=InvalidSecondFactor s141sm5130851pfs.13 - gsmtp
```
**6.3:**. If you have entered wrong Email Address
```
2019/09/18 13:16:06 Error from SMTP Server: 553 5.1.2 The recipient address <recipient-email> is not a valid RFC-5321
5.1.2 address. w6sm8782758pfj.17 - gsmtp
```

#### You can also try package named [Gomail](https://github.com/go-gomail/gomail) for sending mail via Gmail.
