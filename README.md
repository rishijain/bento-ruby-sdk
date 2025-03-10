# Bento SDK for Ruby on Rails
[![Build Status](https://travis-ci.org/bentonow/bento-rails-sdk.svg?branch=master)](https://travis-ci.org/bentonow/bento-rails-sdk)

🍱 Simple, powerful email marketing and automation for Ruby on Rails projects!

Track events, update user data, record LTV and more in Ruby. Data is stored in your Bento account so you can easily research and investigate what's going on. Use this gem to integrate Bento into your Ruby on Rails app!

👋 To get personalized support, please tweet @bento or email jesse@bentonow.com!

🐶 Battle-tested on Bento Production (we dog food this gem ourselves)!

🤝 Contributions welcome and rewarded! Add a PR request for a surprise!

## Installation

**Important note:** Faraday is currently a dependency of this gem, and the minimum ruby version required is **2.6** as that's the minimum version required by the Faraday gem.

Add this line to your application's Gemfile:
```ruby
gem 'bento-sdk', github: "bentonow/bento-ruby-sdk", branch: "master"
```

Then, to fetch the gem:

```bash
$ bundle install
```

## Configuration

Configure the SDK in an initializer:

```ruby
# config/initializers/bento.rb

Bento.configure do |config|
  # This is your site's UUID. This scopes all requests.
  # Consider creating a new site for each environment (development and production) in your Bento account. 
  config.site_uuid = '123456789abcdefghijkllmnopqqrstu'

  # This is your (or another user in your team's) API keys.
  # IMPORTANT: Never store these in your source code as they give full access to your Bento account.
  config.publishable_key = ENV['BENTO_PUBLISHABLE_KEY']
  config.secret_key = ENV['BENTO_SECRET_KEY']
end
```

## Optional: ActionMailer

If you would like to use ActionMailer to send your emails, [install our ActionMailer gem](https://github.com/bentonow/bento-actionmailer) separately.

## Typical Usage

```ruby
# User signs up to your app
Bento::Events.track(email: 'test@test.com', type: '$account.signed_up', fields: { first_name: 'Jesse', last_name: 'Hanley' })

# User cancels their account
Bento::Events.track(email: 'test@test.com', type: '$account.canceled')

# User uses a feature
Bento::Events.track(email: 'test@test.com', type: '$feature.used', details: { feature_name: 'KPI Dashboard' })

# Daily job to sync user custom fields and tags
import = Bento::Subscribers.import([
  {email: 'test@bentonow.com', first_name: 'Jesse', last_name: 'Hanley', widget_count: 1000},
  {email: 'test2@bentonow.com', first_name: 'Jesse', last_name: 'Hanley', company_name: 'Tanuki Inc.'},
  {email: 'test3@bentonow.com', first_name: 'Jesse', last_name: 'Hanley', tags: 'lead,new_subscriber', remove_tags: 'customer'}
])

# Easily check for errors
if import.failed?
  raise StandardError, "Oh no! Something went wrong."
end

# Send an email directly via the API (honors subscription status)
Bento::Emails.send(
  to: "test@bentonow.com",
  from: "jesse@bentonow.com", # MUST BE AN AUTHOR IN YOUR ACCOUNT (EMAILS > AUTHORS)
  subject: "Welcome to Bento, {{ visitor.first_name }}!",
  html_body: "<p>Here is a link to your dashboard {{ link }}</p>",
  personalizations: {
      link: "https://example.com/test"
  }
)

# Send a transactional email (always sends, even if user is unsubscribed)
Bento::Emails.send_transactional(
  to: "test@bentonow.com",
  from: "jesse@bentonow.com", # MUST BE AN AUTHOR IN YOUR ACCOUNT (EMAILS > AUTHORS)
  subject: "Reset Password",
  html_body: "<p>Here is a link to reset your password ... {{ link }}</p>",
  personalizations: {
      link: "https://example.com/test"
  }
)

```

## Available Methods

This Ruby SDK does not contain _all_ available API methods. Please refer to the [Bento API docs](https://docs.bentonow.com/) for all available methods. This remains an opinionated SDK based on the top use cases we've found at Bento for Ruby on Rails apps.

### Subscribers

#### Find or Create a Subscriber
Perfect for quickly adding a subscriber to your Bento account or getting their information to use within your application.
```ruby
subscriber = Bento::Subscribers.find_or_create_by(email: 'test@bentonow.com')
subscriber.email
```

#### Import or Update Subscribers in Bulk
Perfect for quickly adding subscribers (or fifty) to your Bento account.
```ruby
Bento::Subscribers.import([
  {email: 'user1@example.com', first_name: 'John'},
  {email: 'user2@example.com', last_name: 'Doe'}
])
```

#### Add a Tag

```ruby
Bento::Subscribers.add_tag(email: 'test@bentonow.com', tag: 'new_tag')
```

#### Add a Tag via Event

```ruby
Bento::Subscribers.add_tag_via_event(email: 'test@bentonow.com', tag: 'event_tag')
```

#### Remove a Tag

```ruby
Bento::Subscribers.remove_tag(email: 'test@bentonow.com', tag: 'old_tag')
```

#### Add a Field

```ruby
Bento::Subscribers.add_field(email: 'test@bentonow.com', key: 'company', value: 'Acme Inc')
```

#### Remove a Field

```ruby
Bento::Subscribers.remove_field(email: 'test@bentonow.com', field: 'company')
```

#### Subscribe a User

```ruby
Bento::Subscribers.subscribe(email: 'test@bentonow.com')
```

#### Unsubscribe a User

```ruby
Bento::Subscribers.unsubscribe(email: 'test@bentonow.com')
```

#### Change a Subscriber's Email

```ruby
Bento::Subscribers.change_email(old_email: 'old@example.com', new_email: 'new@example.com')
```

### Events

#### Track a Basic Event

```ruby
Bento::Events.track(email: 'test@test.com', type: '$completed_onboarding')
```

#### Track an Event with Fields

```ruby
Bento::Events.track(
  email: 'test@test.com',
  type: '$completed_onboarding',
  fields: { first_name: 'Jesse', last_name: 'Pinkman' }, # optional
  details: { some_data: 'some_value' }  # optional
)
```

#### Track a Unique Event (such as a purchase)

```ruby
Bento::Events.track(
  email: 'test@test.com',
  type: '$purchase',
  fields: { first_name: 'Jesse' },
  details: {
    unique: { key: 'test123' },
    value: { currency: 'USD', amount: 8000 }, # in cents
  }
)
```

#### Batch Track Multiple Events

```ruby
Bento::Events.import([
  {email: 'test@bentonow.com', type: 'Login'},
  {email: 'test@bentonow.com', type: 'Purchase', fields: { first_name: 'Jesse', last_name: 'Hanley' }}
])
```

### Emails

#### Send an Email (honors subscription status)

```ruby
Bento::Emails.send(
  to: "test@bentonow.com",
  from: "jesse@bentonow.com", # MUST BE AN AUTHOR IN YOUR ACCOUNT (EMAILS > AUTHORS)
  subject: "Welcome to Bento, {{ visitor.first_name }}!",
  html_body: "<p>Here is a link to your dashboard {{ link }}</p>",
  personalizations: {
      link: "https://example.com/test"
  }
)
```

#### Send a Transactional Email (always sends, even if user is unsubscribed)

```ruby 
Bento::Emails.send_transactional(
  to: "test@bentonow.com",
  from: "jesse@bentonow.com", # MUST BE AN AUTHOR IN YOUR ACCOUNT (EMAILS > AUTHORS)
  subject: "Welcome to Bento, {{ visitor.first_name }}!",
  html_body: "<p>Here is a link to your dashboard {{ link }}</p>",
  personalizations: {
      link: "https://example.com/test"
  }
)
``` 

### Spam API

#### Check if an email is valid

```ruby
Bento::Spam.valid?('test@bentonow.com')
```

#### Check if an email is risky

```ruby
Bento::Spam.risky?('test@bentonow.com')
```

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/bentonow/bento-ruby-sdk. This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [Contributor Covenant](http://contributor-covenant.org) code of conduct.

## DEPRECATED: Bento Analytics

The class Bento::Analytics has now been deprecated. Please only use the above Bento SDK for Ruby on Rails projects.

## License

The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).
