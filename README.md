This is currently a fork of https://github.com/solidusio-contrib/solidus_subscriptions.  It supports the alpha version of the gem on rubygems until such a time that an official gem is released.

# SolidusSubscriptions

A Solidus extension for subscriptions. **Important note**: this is
**PRE-RELEASE** software and is currently a work-in-progress. There are **no
guarantees** this will work for your store!

Sponsored by [Goby](https://www.goby.co) - Electrify your routine!

## Installation

Add solidus_subscriptions to your Gemfile:

```ruby
gem 'solidus_subscriptions', github: 'solidusio-contrib/solidus_subscriptions'
```

Bundle your dependencies and run the installation generator:

```shell
bundle
bundle exec rails g solidus_subscriptions:install
```

## Configuration
This gem requires a gateway which supports credit cards in order to process
subscription orders.

Add this to specify the gateway used by the gem:
an initializer.

```ruby
SolidusSubscriptions::Config.default_gateway { my_gateway }
```

## Usage

### Purchasing Subscriptions
By default only Spree::Variants can be subscribed to. To subscribe to a variant, it
must have the `:subscribable` attribute set to true.

To subscribe to a variant include the following parameters when posting to
`/orders/populate` (The add to cart button on the product page):

```js
  {
    // other add to cart params
    subscription_line_item: {
      quantity: 2,              // number of units in each subscription order.
      subscribable_id: 1234,    // Which variant the subscription is for.
      interval_length: 1,       // The time between subscription activations.
      interval_units: "month", // A plural qualifier for length.
                                // Can be one of "day", "week", "month", or "year".
      end_date: '2011/12/13'     // Stop processing after this date
                                // (use null to process the subscription ad nauseam)
    }
  }
```

This will associate a `SolidusSubscriptions::LineItem` to the line item
being added to the cart.

The customer will not be charged for the subscription until it is processed. The
subscription line items should be shown to the user on the cart page by
looping over `Spree::Order#subscription_line_items`.
`SolidusSubscriptions::LineItem#dummy_line_item` may be useful to help you display
the subscription line item with your existing cart infrastructure.

When the order is finalized, a `SolidusSubscriptions::Subscription` will be
created for each group of subscription line items which can be fulfilled by a single
subscription.

#### Example:

An order is finalized and has following associated subscription line items:

1. { subscribable_id: 1, interval_length: 1, interval_units: 'month'}
2. { subscribable_id: 2, interval_length: 1, interval_units: 'month' }
3. { subscribable_id: 1, interval_length: 2, interval_units: 'month' }

This will generate 2 Subscriptions objects. The first related to
subscription_line_items 1 & 2. The second  related to line item 3.

### Processing Subscriptions

To process actionable subscriptions simply run:

`bundle exec rake solidus_subscriptions:process`

To schedule this task we suggest using the [Whenever](https://github.com/javan/whenever) gem.

This task creates ActiveJobs which can be fulfilled by the queue library of your
choice.

### Guest Checkout

Subscriptions require a user to be present to allow them to be managed after
they are purchased.

Because of this you must  disabling guest checkout for orders
which contain `subscription_line_items`.

An example of this would be adding this to the registration page:

```erb
<%# spree/checkout/registration.html.erb %>
<% if Spree::Config[:allow_guest_checkout] && current_order.subscription_line_items.empty? %>
```

This allows guests to add subscriptions to their carts as guests, but forces them
to login or create an account before purchasing them.

## Testing

First bundle your dependencies, then run `rake`. `rake` will default to building the dummy app if it does not exist, then it will run specs, and [Rubocop](https://github.com/bbatsov/rubocop) static code analysis. The dummy app can be regenerated by using `rake test_app`.

```shell
bundle
bundle exec rake
```

When testing your applications integration with this extension you may use it's factories.
Simply add this require statement to your spec_helper:

```ruby
require 'solidus_subscriptions/testing_support/factories'
```

Copyright (c) 2016 Stembolt, released under the New BSD License
