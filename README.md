Ops
===

[![Code Climate](https://codeclimate.com/github/primedia/ops.png)](https://codeclimate.com/github/primedia/ops)

This gem provides standardized support for obtaining environment, version, and heartbeat information from Sinatra or Rails-based web applications.

It optionally provides support for obtaining configuration data from a service named Confusion.

**You will likely want to block or restrict access to the following routes:**

Route            | Notes
---------------- | -----
`/ops/env`       | Exposes all of your environment variables (e.g. any API keys set as environment variables) to the public
`/ops/confusion` | Exposes all of your `confusion` keys and values to the public (if you're using `confusion`)

Typical usage:

```
/ops/version      - displays version info as HTML
/ops/version.json - displays version info as JSON
/ops/heartbeat    - returns 'OK' if the app is alive
/ops/env          - display the currently set environment variables
/ops/confusion    - returns configuration data from Confusion as JSON (optional)
```

This gem replaces the now-deprecated [ops_routes](https://github.com/primedia/ops_routes).

Installation
------------

### For Rails 3 apps:

1. Add the gem to your project's Gemfile:
    ```ruby
    gem 'ops'
    gem 'configuration_client', '~> 0.7.0' # optional
    ```

2. Add the following to application.rb:

    ```ruby
    Ops.setup do |config|
      config.file_root = Rails.root
      config.environment = Rails.env
      config.use_confusion = true # optional
    end
    ```

3. mount the gem in routes.rb:

    ```ruby
    mount Ops.new, :at => "/ops"
    ```

### For Sinatra apps:

1. Add the gem to your project's Gemfile:

    ```ruby
    gem 'ops'
    gem 'configuration_client', '~> 0.7.0' # optional
    ```

2. Add the following to config.ru:

    ```ruby
    require 'ops'

    #...

    Ops.setup do |config|
      config.file_root = File.dirname __FILE__
      config.environment = ENV['RACK_ENV']
      config.use_confusion = true # optional
    end

    run Rack::URLMap.new \
      "/"    => YourAppClass,
      "/ops" => Ops.new
    ```

    ```ruby
    # Implementation within rack cascade:
    run Rack::Cascade.new([
      NewHomeGuide,
      ListingSearch::App,
      Ops.rack_app('/ops')
    ])
    ```

Adding Custom Heartbeats
------------------------

Additionally, you can specify custom heartbeat monitoring pages as follows:

```ruby
Ops.add_heartbeat :mysql do
  conn = ActiveRecord::Base.connection
  migrations = conn.select_all("SELECT COUNT(1) FROM schema_migrations;")
  conn.disconnect!
end
```

The mysql example shown above would be accessed at ops/heartbeat/mysql. The heartbeat page will return a `200 ‘OK’` as long as the provided block returns true. If an error is raised, the heartbeat does not exist, or the block returns a falsey value, a `500` will be returned instead.