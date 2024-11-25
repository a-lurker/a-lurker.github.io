# openLuup and email

openLuup has a SMTP (Simple Mail Transfer Protocol) email server built in. You can use it to send and receive emails directly by openLuup.

It only handles messages within the LAN. However you could set up say Telegram to forward emails to the outside world. Most email clients can be set up to RX messsages.

## Mail boxes
There are a number of predefined mailbox addresses but you can add your own:

- events@openLuup.local - only the subject line of any mail sent here will be retained. The message will be saved in a file in `../cmh-ludl/events/`.
- images@openLuup.local – used for camera trigger emails, or other messages with image attachments. Images are written to the `image/` folder in the openLuup home directory.
- mail@openLuup.local - a normal mailbox from which full messages, with attachments, may be retrieved. The message will be saved in a file in `../cmh-ludl/mail/`.
- openLuup@openLuup.local – general destination for openLuup messages. This mailbox performs no processing on incoming messages. An email black hole.
- postmaster@openLuup.local – required for an SMTP compliant server.
- test@openLuup.local – all message data lines, including headers and body, will be written to the log to facilitate debugging. Beware of trying this on long messages!

Add your own mailbox:

```lua
luup_register_handler ("MyEmailHandler", "me@mymail.local")
```

You can view the server mailbox information at the openLuup console:

http://openLuup_IP_address:3480/console?page=smtp

## Email server port
The server port defaults to 2525 but it can be changed in the start up code. Other commonly used ports are 25 and 587.

```lua
luup.attr_set ("openLuup.SMTP.Port", 587) -- use port 587 instead of the 2525 default
```

## Email management
Your email client can be used to manage the deletion of files from the mailboxes, or, alternatively, a timed scene using the openLuup SendToTrash and EmptyTrash actions will do the job automatically.
See http://openLuup_IP_address:3480/console?page=images.

## Handling camera images
Many cameras faciltate the emailing of images to wherever. In this case we look at emailing these images to openLuup.

The I_openLuupCamera1.xml implementation file is virtual and ready for use. You can validate its existence using the following URl: http://openLuupIP:3480/I_openLuupCamera1.xml.

There are two openLuup plugin actions:
1. serviceId = openLuup, action = SendToTrash
2. serviceId = openLuup, action = EmptyTrash

### SendToTrash
This action moves selected files to openLuup’s trash/ folder, and has four parameters:

- Folder - the path of a folder below the openLuup home directory. For the application described here, this should be `images`.
- MaxDays - the maximum age in days of files which should be retained.
- MaxFiles - the maximum number of files that should be retained.
- FileTypes - a string listing the types the types of files to which this applies, for example `jpg gif bmp`. It may also be set to `*` in which case, any file is applicable. Note that this does not act on subfolders or their contents.

```lua
local openLuupID = 2 -- always the case
luup.call_action ("openLuup", "SendToTrash", Folder, MaxDays, MaxFiles, FileTypes)
```

One or both of MaxDays and MaxFiles may be specified. If either of the conditions is true, then the file is moved to trash.

Moving files to the trash folder is reversible (manually!) but to delete them permanently, you may use the following action.

### EmptyTrash
This action has one parameter:
- AreYouSure - needs to have the value `yes` (case-insensitive) to delete all the files in openLuup’s `trash/` folder.

```lua
local openLuupID = 2 -- always the case
luup.call_action ("openLuup", "EmptyTrash", AreYouSure)
```

The purpose of the AreYouSure parameter is to avoid accidental presses of the EmptyTrash button on the device actions page having any effect.

The above actions are easily included into scheduled scenes which might, for example, run the SendToTrash action early in the morning every day, and the EmptyTrash every weekend.

## Camera devices
Thanks to user "ramwal". A couple of camera device examples:

Two devices created for two cameras. A D-Link DCS-932L and a Foscam FI8919w, both older cameras with the settings below. They both giving an image on the camera device.

```lua
-- Foscam FI8919w
luup.attr_set ("ip", "192.168.0.120", 93)
luup.attr_set ("ip", "192.168.0.120", 94)

luup.variable_set ("urn:micasaverde-com:serviceId:Camera1", "DirectStreamingURL", "/videostream.cgi?user=ramwal&pwd=xxx", 93)

– D-Link DCS-932L
luup.attr_set ("ip", "192.168.0.15", 95)
luup.attr_set ("ip", "192.168.0.15", 96)

luup.variable_set ("urn:micasaverde-com:serviceId:Camera1", "DirectStreamingURL", "/mjpeg.cgi?usr=admin&pwd=xxx", 95)
```

## Plugin developers
Developers may want to send an email to openLuup:

```lua
local serverIP    = "openLuupIpAddress"
local sentFrom    = "myPlugin@openLuup"
local sentTo      = "events@openLuup.local"
local subjectLine = "Message dated " .. os.date "%c"
local message     = "Plugin says hello\r\n"

local smtp = require "socket.smtp"

local mesgt =
{
    headers = {
        from    = sentFrom,
        to      = sentTo,
        subject = subjectLine},
    body = message
}

local result, x = smtp.send
{
    from     = sentFrom,
    rcpt     = {sentTo},
    source   =  smtp.message(mesgt),
    server   = serverIP,
    port     = "2525",

-- optional usage of a password
-- user     = sentFrom,
-- password = "openLuupMailPassword",
}
```

An example file `1732490422.msg` saved to `../cmh-ludl/mail` when an email is sent to mail@openLuup.local. Both the subject line and the body is retained:

```lua
Received: from (localhost) [ip_address]
 by (openLuup.smtp v18.4.12) [ip_address];
 Mon, 25 Nov 2024 10:20:22 +1000
MIME-Version: 1.0
Subject: Message dated Mon Nov 25 10:20:22 2024
Date: Sun, 24 Nov 2024 23:20:22 -0000
From: myPlugin@openLuup
To: mail@openLuup.local
Content-Type: text/plain; charset="iso-8859-1"
X-Mailer: LuaSocket 3.0.0

Plugin says hello

```

An example file `1732425254.msg` saved to `../cmh-ludl/events` when an email is sent to events@openLuup.local. Note only the subject line is retained; the body is discarded:

```lua
X-Mailer: LuaSocket 3.0.0
To: events@openLuup.local
From: myPlugin@openLuup
Date: Sun, 24 Nov 2024 05:14:14 -0000
MIME-Version: 1.0
Subject: Message dated Sun Nov 24 16:14:14 2024
Content-Type: text/plain
Received: from (localhost) [ip_address] by (openLuup.smtp v18.4.12) [ip_address]; Sun, 24 Nov 2024 16:14:14 +1000
```
## Debugging
Add the following to your Lua Startup, then reload. Don't forget to remove this when finished debugging.

```lua
do -- SMTP debug
    local smtp = require "openLuup.smtp"
    smtp.ABOUT.DEBUG = true
end
```

## Testing the server from some other computer using telnet
At the command prompt type: telnet openluup_ip_address 2525

```bash
C:\Users\XYZ>telnet openluup_ip_address 2525
220 (openLuup.smtp v18.4.12) [openluup_ip_address] Service ready
```

Connection was successful now type: quit

```bash
quit
221 (openLuup.smtp v18.4.12) [openluup_ip_address] Service closing transmission channel


Connection to host lost.

C:\Users\XYZ>
```
