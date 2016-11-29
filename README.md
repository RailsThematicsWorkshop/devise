# Set-up authentication with devise & omniauth

Welcome ! 

The goal of this workshop is to add an authentication mechanism to a sample rails application thanks to the devise gem.
You will be guided through setting up devise and a confirmation module, once it is done, and if you want to go further, you'll have to read the docs and dig a little bit, but don't worry, we'll be here to help.

Download our sample app fotoklub [here](https://github.com/RailsThematicsWorkshop/fotoklub/archive/0.0.1.zip)

Open the docs for quick reference : 

[devise](https://github.com/plataformatec/devise)
[omniauth](https://github.com/intridea/omniauth)

### Installing devise

Add the devise gem in your ``Gemfile``

    gem 'devise'

Bundle the gems and run devise's install generator.

    bundle install
    rails generate devise:install

### Handle flash messages

Flash messages are a way for your application to communicate success or failure, devise uses them to display some messages, add the following somewhere handy in your layout: 

    <div class="row">
      <div class="col-xs-6 col-xs-offset-3">
        <% if flash[:notice] %>
          <div class="alert alert-info" role="alert">
            <%= flash[:notice] %>
          </div>
        <% end %>

        <% if flash[:alert] %>
          <div class="alert alert-danger" role="alert">
            <%= flash[:alert] %>
          </div>
        <% end %>

      </div>
    </div>


### Generate your devise model

     rails generate devise user

Run migrations

    bundle exec rake db:migrate

If you look at `routes.rb` you can see than devise added a line there.

Devise hooked itself into our applications and we can now access new urls.

Run `rake routes` to see the new available routes.

You can copy devise views if you want to modify them to better suit your layout

    rails generate devise:views

### Let's do it

We now have everything ready to use the basics of devise.

Let's say we only want authenticated users to access the photos page.

To restrict a controller or action to logged-in users : 

    before_action :authenticate_user!

To access the current user model : 

    current_user

Let's do several things:

- Hide the photos link if the user is not logged in
- Show the login link if the user is not logged in
- Make sure /photos is not accessible if the user is not logged in
- Add a log out link if we're logged in

### Adding an email confirmation

Email configuration is a pain, depending on your internet provider, etc, it might not work or get caugh into spam.

We're going to use the mailcatcher gem to circumvent this issue in development.

In your Gemfile, add : 

    gem 'mailcatcher'

run ``bundle`` to install it.

To let our rails app know that it has to speak with mail catcher, add the following to ``config/environments/development.rb``

    config.action_mailer.delivery_method = :smtp
    config.action_mailer.smtp_settings = { :address => "localhost", :port => 1025 }

Also add a default host to link to : 

    config.action_mailer.default_url_options = {:host => "localhost:3000"}


And run mailcatcher in another tab : 

    bundle exec mailcatcher

It will create a web interface accessible on  http://localhost:1080/, all the emails that you send through your app are going to end up here, handy !

We're going to use the "confirmable" devise module, it needs a few more fields in the database, let's create a migration to add them : 

    rails g migration add_confirmable_to_devise

Here's the migration file recommended by devise : 

    class AddConfirmableToDevise < ActiveRecord::Migration[5.0]
      def up
          add_column :users, :confirmation_token, :string
          add_column :users, :confirmed_at, :datetime
          add_column :users, :confirmation_sent_at, :datetime
          add_column :users, :unconfirmed_email, :string
          add_index :users, :confirmation_token, unique: true
          execute("UPDATE users SET confirmed_at = date('now')")
        end

        def down
          remove_columns :users, :confirmation_token, :confirmed_at, :confirmation_sent_at, :unconfirmed_email
        end
    end


Run it :

    bundle exec rake db:migrate

Add `:confirmable` to the devise modules in user.rb


Restart your server and you should be all set

### Now what ? 

Congratulations ! You successfully added devise to a rails project and used a devise module !

Thirsty for more ? Here's a suggestion of things to do, don't hesitate to ask for directions or help :

- Use Omniauth to implement login with facebook / twitter / google / etc.
- Add other devise modules, like lockable, to prevent a user from trying to log in too much
- Use devise callbacks to send a welcome email to the user when they confirm their account
