title: Padrino - 'constant Logger::Format not defined'

```
=> Problem loading ./config/boot.rb
=> constant Logger::Format not defined
  /home/kurt/.rvm/gems/ruby-1.9.2-p180@padrino-testing/gems/activerecord-1.6.0/lib/active_record/support/clean_logger.rb:5:in `remove_const'
  /home/kurt/.rvm/gems/ruby-1.9.2-p180@padrino-testing/gems/activerecord-1.6.0/lib/active_record/support/clean_logger.rb:5:in `<class:Logger>'
  /home/kurt/.rvm/gems/ruby-1.9.2-p180@padrino-testing/gems/activerecord-1.6.0/lib/active_record/support/clean_logger.rb:3:in `<top (required)>'
  /home/kurt/.rvm/gems/ruby-1.9.2-p180@padrino-testing/gems/activerecord-1.6.0/lib/active_record.rb:28:in `require'
  /home/kurt/.rvm/gems/ruby-1.9.2-p180@padrino-testing/gems/activerecord-1.6.0/lib/active_record.rb:28:in `<top (required)>'
  /home/kurt/.rvm/gems/ruby-1.9.2-p180@global/gems/bundler-1.0.12/lib/bundler/runtime.rb:68:in `require'
  /home/kurt/.rvm/gems/ruby-1.9.2-p180@global/gems/bundler-1.0.12/lib/bundler/runtime.rb:68:in `block (2 levels) in require'
  /home/kurt/.rvm/gems/ruby-1.9.2-p180@global/gems/bundler-1.0.12/lib/bundler/runtime.rb:66:in `each'
  /home/kurt/.rvm/gems/ruby-1.9.2-p180@global/gems/bundler-1.0.12/lib/bundler/runtime.rb:66:in `block in require'
  /home/kurt/.rvm/gems/ruby-1.9.2-p180@global/gems/bundler-1.0.12/lib/bundler/runtime.rb:55:in `each'
  /home/kurt/.rvm/gems/ruby-1.9.2-p180@global/gems/bundler-1.0.12/lib/bundler/runtime.rb:55:in `require'
  /home/kurt/.rvm/gems/ruby-1.9.2-p180@global/gems/bundler-1.0.12/lib/bundler.rb:120:in `require'
  /home/kurt/projects/padrino-testing/sample_blog/config/boot.rb:8:in `<top (required)>'
  /home/kurt/.rvm/rubies/ruby-1.9.2-p180/lib/ruby/site_ruby/1.9.1/rubygems/custom_require.rb:36:in `require'
  /home/kurt/.rvm/rubies/ruby-1.9.2-p180/lib/ruby/site_ruby/1.9.1/rubygems/custom_require.rb:36:in `require'
  /home/kurt/.rvm/gems/ruby-1.9.2-p180@padrino-testing/gems/padrino-gen-0.9.29/lib/padrino-gen/generators/cli.rb:24:in `load_boot'
  /home/kurt/.rvm/gems/ruby-1.9.2-p180@padrino-testing/gems/thor-0.14.6/lib/thor/task.rb:22:in `run'
  /home/kurt/.rvm/gems/ruby-1.9.2-p180@padrino-testing/gems/thor-0.14.6/lib/thor/invocation.rb:118:in `invoke_task'
  /home/kurt/.rvm/gems/ruby-1.9.2-p180@padrino-testing/gems/thor-0.14.6/lib/thor/invocation.rb:124:in `block in invoke_all'
  /home/kurt/.rvm/gems/ruby-1.9.2-p180@padrino-testing/gems/thor-0.14.6/lib/thor/invocation.rb:124:in `each'
  /home/kurt/.rvm/gems/ruby-1.9.2-p180@padrino-testing/gems/thor-0.14.6/lib/thor/invocation.rb:124:in `map'
  /home/kurt/.rvm/gems/ruby-1.9.2-p180@padrino-testing/gems/thor-0.14.6/lib/thor/invocation.rb:124:in `invoke_all'
  /home/kurt/.rvm/gems/ruby-1.9.2-p180@padrino-testing/gems/thor-0.14.6/lib/thor/group.rb:226:in `dispatch'
  /home/kurt/.rvm/gems/ruby-1.9.2-p180@padrino-testing/gems/thor-0.14.6/lib/thor/base.rb:389:in `start'
  /home/kurt/.rvm/gems/ruby-1.9.2-p180@padrino-testing/gems/padrino-gen-0.9.29/bin/padrino-gen:15:in `<main>'
Please specify generator to use (project, app, mailer, controller, model, migration, plugin)
```

To avoid this error, just specify the activerecord version in your gemfile (3.0.7), and rebuild your bundle:

```ruby
gem 'activerecord', '~>3.0.7', :require => "active_record"
...
gem 'padrino' # remove the version number here
```

```bash
rm Gemfile.lock
gem uninstall padrino activerecord i18n
bundle install
```

source: 

* http://groups.google.com/group/padrino/browse_thread/thread/8326c286a8346b70
* https://github.com/padrino/padrino-framework/issues/572
