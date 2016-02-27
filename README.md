# whm-new-user-emails-to-admin
This is my noobish hard to read bash script which works for me.

SUMARY

It is made for Linux server with WHM/cPanel control panel installed and its aim is to email WHM server admin new emails that was received/sent by new cpanel users so the server admin can suspend user if he send out SPAM/abuse.

IN DETAIL

- Find latest (newest) n cPanel accounts, filter out suspended accounts out of them

- Find a few recently created mail files in its mail directory (/home/username/mail)

- Send that email files contents to the WHM/cPanel server admin via email so he can check if email is not an SPAM/abuse of the service.

Script also prevent sending email files that already been sent and also files that contains phrasses set by admin in file "whitephrasses".
