# Mandrill Mailer gem
MandrilMailer class for sending transactional emails through mandril.
Only template based emails are supported at this time.

## Usage
Add `gem 'mandrill_mailer'` to your Gemfile

Add this to your `mail.rb` in initializers:

```
ActionMailer::Base.smtp_settings = {
    :address   => "smtp.mandrillapp.com",
    :port      => 587,
    :user_name => ENV['MANDRILL_USERNAME'],
    :password  => ENV['MANDRILL_PASSWORD'],
    :domain    => 'heroku.com'
  }
ActionMailer::Base.delivery_method = :smtp
MandrillMailer.api_key = ENV['MANDRILL_API_KEY']
```

Don't forget to setup your ENV variables on your server

## Creating a new mailer
Creating a new Mandrill Mailer is similar to a normal Rails mailer:

```
 class InvitationMailer < MandrillMailer
   default from: 'support@example.com'

   def invite(invitation)
     mandrill_mail template: 'Group Invite',
       subject: I18n.t('invitation_mailer.invite.subject'),
       to: {email: invitation.email, name: 'Honored Guest'},
       vars: {
         'OWNER_NAME' => invitation.owner_name,
         'INVITATION_URL' => new_invitation_url(email: invitation.email, secret: invitation.secret)
       }
   end
 end
 ```

 * `#default:`
  * `:from` - set the default from email address for the mailer

* `.mandrill_mail`
   * `:template`(required) - Template name from within Mandrill

   * `:subject`(required) - Subject of the email

   * `:to`(required) - Accepts an email String, or hash with :name and :email keys
     ex. `{email: 'someone@email.com', name: 'Bob Bertly'}`

   * `:vars` - A Hash of merge tags made available to the email. Use them in the
     email by wrapping them in `*||*` vars: {'OWNER_NAME' => 'Suzy'} is used
     by doing: `*|OWNER_NAME|*` in the email template within Mandrill

   * `:template_content` - A Hash of values and content for Mandrill editable content blocks.
     In MailChimp templates there are editable regions with 'mc:edit' attributes that look
     a little like: `<div mc:edit="header">My email content</div>` You can insert content directly into
     these fields by passing a Hash `{'header' => 'my email content'}`

   * `:headers` - Extra headers to add to the message (currently only Reply-To and X-* headers are allowed) {"...": "..."}

   * `:bcc` - Add an email to bcc to

   * `:tags` - Array of Strings to tag the message with. Stats are
   accumulated using tags, though we only store the first 100 we see,
   so this should not be unique or change frequently. Tags should be
   50 characters or less. Any tags starting with an underscore are
   reserved for internal use and will cause errors.

   * `:google_analytics_domains` - Array of Strings indicating for which any
   matching URLs will automatically have Google Analytics parameters appended
   to their query string automatically.

   * `:google_analytics_campaign` - String indicating the value to set for
   the utm_campaign tracking parameter. If this isn't provided the email's
   from address will be used instead.
   
## Sending an email

You can send the email by using the familiar syntax:

`InvitationMailer.invite(invitation).deliver`
   
## Creating a test method
When switching over to Mandrill for transactional emails we found that it was hard to setup a mailer in the console to send test emails easily (those darn designers), but really, you don't want to have to setup test objects everytime you want to send a test email. You can set up a testing 'stub' once and then call the `.test` method to send the test email.

You can test the above email by typing: `InvitationMailer.test(:invite, email:<your email>)` into the Rails Console.

The test for this particular Mailer is setup like so:

```
test_setup_for :invite do |mailer, options|
    invitation = OpenStruct.new({
      email: options[:email],
      owner_name: 'foobar',
      secret: rand(9000000..1000000).to_s
    })
    mailer.invite(invitation).deliver
end
```

The mailer and options passed to the `.test` method are yielded to the block.

The `:email` option is the only required option, make sure to add at least this to your test object.

## Helper Methods
* `.test_friendly_url`(object, route helper as a symbol) - This helper is used for using route helpers, but still staying compatible with using OpenStructs for stubing out the mailing test objects (there is probably a much better way of doing this)