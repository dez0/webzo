SEND_EMAIL worker
======
#Types of mailer
*  Confirmed
*  Incomplete
*  Malicious
*  Simple

##Required parameters
For all types of mailer, these parameters are required by the worker :
* `type`: string, required, type of mail to send.
* `to`: string, required, email address of the recipient.
* `from`: string, required, email address of the sender.
* `subject`: string, required, subject of the email. Refer to a I18n key in locales directory.
-----------

##Confirmed mailer parameters
####Required
* `remote_layout`: string, required, retailer's remote layout of the email body.

####Optional
* `transaction`: array of key/value pairs, transaction information from Desjardins payment.
-----------

##Simple mailer parameters
####Optional
* `content`: string, html content of the email body.
* `layout`: string, the local layout of the email body.
* `remote_layout`: string, retailer's remote layout of the email body.
* `layout_params`: array of key/value pairs, specific variables passed to the worker.
* `lang`: string, language of the email, used by I18n. DEFAULT: 'en'.
-----------
###Local layout types
* simple

>     
```
<!DOCTYPE html>
<html>
  <head>
    <meta content="text/html; charset=UTF-8" http-equiv="Content-Type" />
  </head>
  <body>
    <%= @body %>
  </body>
</html>
```

-----------
##Parameters example
>     params = { :type => "simple",
               :to => "vincent.desautels@gmail.com",
               :from => "info@dakis.com",
               :subject => "simple_test",
               :content => "<p>This is a simple test...</p>",
               :layout => "simple",
               :remote_layout => "http://bobsphoto.com/mailer.html",
               :layout_params => {
                  :title => "Test",
                  :footer => "Copyright Dakis 2014"
               },
               :lang => "en"
             }
