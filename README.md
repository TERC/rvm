# Capistrano::RVM

RVM support for Capistrano v3:

## Notes

**If you use this integration with capistrano-rails, please ensure that you have `capistrano-bundler >= 1.1.0`.**

If you want solution with RVM/rubies installer included, give a try to [rvm1-capistrano3](https://github.com/rvm/rvm1-capistrano3).

## Installation

Add this line to your application's Gemfile:

    # Gemfile
    gem 'capistrano', '~> 3.0'
    gem 'capistrano-rvm'

And then execute:

    $ bundle install

## Usage

Require in Capfile to always use `rvm:hook` task:

    # Capfile
    require 'capistrano/rvm'

And you should be good to go!  All tasks will be rvm-aware.

Or, If you prefer to manually manage RVM integration at the task level:

    # Capfile
    require 'capistrano/rvm/no_hook'

With this approach, you must explicitly set `rvm:hook` as a prequisite for tasks
requiring RVM support:

    before TASKNAME, 'rvm:hook'

You may aletnatively use the convenience method `with_rvm`:

    with_rvm TASKNAME

Multiple tasks may be supported in one line:

    with_rvm TASKNAME1, TASKNAME2, TASKNAME3

## Configuration

Everything *should work* for a basic RVM setup *out of the box*.

If you need some special settings, set those in the stage file for your server:

    # deploy.rb or stage file (staging.rb, production.rb or else)
    set :rvm_type, :user                     # Defaults to: :auto
    set :rvm_ruby_version, '2.0.0-p247'      # Defaults to: 'default'
    set :rvm_custom_path, '~/.myveryownrvm'  # only needed if not detected

### RVM path selection: `:rvm_type`

Valid options are:
  * `:auto` (default): just tries to find the correct path.
                       `~/.rvm` wins over `/usr/local/rvm`
  * `:system`: defines the RVM path to `/usr/local/rvm`
  * `:user`: defines the RVM path to `~/.rvm`

### Ruby and gemset selection: `:rvm_ruby_version`

By default the Ruby and gemset is used which is returned by `rvm current` on
the target host.

You can omit the ruby patch level from `:rvm_ruby_version` if you want, and
capistrano will choose the most recent patch level for that version of ruby:

    set :rvm_ruby_version, '2.0.0'

If you are using an rvm gemset, just specify it after your ruby_version:

    set :rvm_ruby_version, '2.0.0-p247@mygemset'

or

    set :rvm_ruby_version, '2.0.0@mygemset'

### Custom RVM path: `:rvm_custom_path`

If you have a custom RVM setup with a different path then expected, you have
to define a custom RVM path to tell capistrano where it is.

### Custom Roles: `:rvm_roles`

If you want to restrict RVM usage to a subset of roles, you may set `:rvm_roles`:

    set :rvm_roles, [:app, :web]

## Restrictions

Capistrano can't use RVM to install rubies or create gemsets, so on the
servers you are deploying to, you will have to manually use RVM to install the
proper ruby and create the gemset.


## How it works

With the default configuration, this gem adds a new tasks `rvm:hook` and
`rvm:check` after each stage-setting task (ex. 'staging', 'production', etc.).

`rvm:hook` sets the `rvm ... do ...` for capistrano when it wants to run `rake`,
`cap`, `gem`, `bundle`, or `ruby`.

`rvm:check` outputs the current version info for rvm plus the active rvm and
gemset.  It only runs when :log_level is set to :debug (including when the
`--trace` argument is passed to `cap`.)

When the `no_hook` configuration is used, `rvm:hook` must be manually set as a
prerequisite for tasks requiring RVM support (with the exception of `rvm:check`.)

## Check your configuration

If you want to check your configuration you can use the `rvm:check` task to
get information about the RVM version and ruby which would be used for
deployment.

    $ cap production rvm:check

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
