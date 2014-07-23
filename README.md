SEND_EMAIL worker
======
#Prerequisite
In order to send email, configuration parameters need to be set in the worker's code section on Iron.io HUD. [Reference] (http://dev.iron.io/worker/reference/configuration-variables/#config-via-hud)
>  
```
{
  "SMTP_ADDRESS"  : "address",
  "SMTP_PORT"     : "port",
  "SMTP_USERNAME" : "username",
  "SMTP_PASSWORD" : "password"
}
```

#Types of mailer
*  Confirmed
*  Incomplete
*  Malicious
*  Simple

###Required parameters
For all types of mailer, these parameters are required by the worker :
* `type`: string, required, type of mail to send.
* `to`: string, required, email address of the recipient.
* `from`: string, required, email address of the sender.
* `subject`: string, required, subject of the email. Refer to a I18n key in locales directory.

##Confirmed/Incomplete/Malicious mailer parameters
####Required
* `retailer`: array of key/value pairs, required, general information about the retailer.
* `shopping_cart`: shopping_cart object, required, general information about the shopping cart. [Reference] (https://github.com/dakis/base/blob/dev/lib/shopping_cart.rb)
* `customer`: array of key/value pairs, required, general information about the customer.
* `items`: array item objects, required, item(s) in shopping cart. [Reference] (https://github.com/dakis/base/blob/dev/lib/shopping_cart_item.rb)
* `total`: array of key/value pairs, required, contains information about shopping cart totals (subtotal, shipping, taxes, total).

####Optional
* `lang`: string, language of the email, used by I18n. DEFAULT: 'en'.
* `remote_layout`: string, required, retailer's remote layout of the email body.
* `pickup_address`: string, retailer's store address for picked up orders.
* `transaction`: array of key/value pairs, transaction information from Desjardins payment.
* `fulfillment`: array of key/value pairs, information about order fulfiller(s).
* `promo`: array of promotion items object, promotion item(s) in shopping cart. [Reference] (https://github.com/dakis/base/blob/dev/lib/promotion_cart_item.rb)
* `special_instruction`: array of instruction string, special instruction or comments left by the customer.
* `taxes`: array of key/value pairs, contains information about shopping cart taxes.

##Simple mailer parameters
####Optional
* `content`: string, html content of the email body.
* `layout`: string, the local layout of the email body, included in the worker directory.
* `remote_layout`: string, retailer's remote layout of the email body.
* `layout_params`: array of key/value pairs, specific variables passed to the worker.
* `lang`: string, language of the email, used by I18n. DEFAULT: 'en'.

####Local simple mailer layout types
* simple

####Simple mailer parameters example
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

#How to add a new worker
This is the procedure to add a new mailer to the worker

1. Add new mailer class in the /mailers directory. Be sure to require the base mailer : `require 'lib/mailer'`.
2. In the initialize function of the new mailer, be sure to include the `super` clause to use the base mailer constructor. Also, you need to define `@to`, `@from`, `@subject` variables.
3. Add a `get_body_content` function to tell how to build the `@body` of the email.
4. Add a `when` clause for the new mailer type in the case statement in the `mailer.rb` file at the root of the worker.
5. Send the parameters to the worker. Don't forget to specify the `type` in the sent parameters to use the new mailer.

####New mailer class example
The `@h[]` contains the parameters sent to the worker.

This is a example of the `mailer/newmailer.rb` file.
```
require 'lib/mailer'

class NewMailer < Mailer
  def initialize params
    super

    @to = @h["to"]
    @from = @h["from"]
    @subject = t(@h["subject"])
  end

  def get_body_content
    "Content of the email's body"
  end
end
```

This is an example of the new when statement in the `/mailer.rb` file.
```
unless params[:type].nil?
  mailer = case params[:type]
          when "confirmed"
            ConfirmedMailer.new params
          when "incomplete"
            IncompleteMailer.new params
          when "malicious"
            MaliciousMailer.new params
          when "simple"
            SimpleMailer.new params
		  when "new"
			NewMailer.new params
      end

  mailer.build_body
  mailer.send
end
```
