# JRuby boot time optimizations

Using JRuby 1.7.0:

    $ ruby -v
    jruby 1.7.0 (1.9.3p203) 2012-10-22 ff1ebbe on Java HotSpot(TM) 64-Bit Server VM 1.7.0_09-b05 [darwin-x86_64]

Compared to MRI:

    ruby 1.9.3p327 (2012-11-10 revision 37606) [x86_64-darwin12.2.0]

    $ time bundle exec rake spec bundle exec rake spec  3,99s user 0,51s system 87% cpu 5,114 total


## Vanilla Rails Application

Generated Rails application with:

    $ rails new jruby-boot --skip-bundle --database jdbcpostgresql --skip-test-unit
    $ cd jruby-boot
    $ bundle install
    $ rails g rspec:install
    $ rake db:create db:migrate

## Running RSpec

Average benchmark results.

### JRuby

Commands:

    $ time bundle exec rake spec
    bundle exec rake spec  85,51s user 4,56s system 201% cpu 44,723 total

    $ time rake spec
    rake spec  69,84s user 3,66s system 198% cpu 37,062 total

    $ time bundle exec rspec spec
    bundle exec rspec spec  51,26s user 2,51s system 190% cpu 28,248 total

    $ time rspec spec
    rspec spec  33,84s user 1,80s system 193% cpu 18,385 total

### JRuby + Spork

Commands:

    $ bundle exec spork

    $ time bundle exec rake spec
    bundle exec rake spec  69,67s user 3,28s system 174% cpu 41,905 total

    $ time rake spec
    rake spec  51,54s user 2,31s system 190% cpu 28,231 total

    $ time bundle exec rspec spec
    bundle exec rspec spec  34,76s user 1,19s system 193% cpu 18,615 total

    $ time rspec spec
    rspec spec  21,46s user 0,72s system 131% cpu 16,836 total

Output when running `rake`:

    .

    Finished in 0.313 seconds
    1 example, 0 failures

    Randomized with seed 43913

      <-- Slave(1) run done!
    rake aborted!
    /Users/filiptepper/.rvm/rubies/jruby-1.7.0/bin/jruby -S rspec ./spec/example_spec.rb failed
    /Users/filiptepper/.rvm/gems/jruby-1.7.0@boot/gems/rspec-core-2.12.0/lib/rspec/core/rake_task.rb:154:in `run_task'
    /Users/filiptepper/.rvm/gems/jruby-1.7.0@boot/gems/rspec-core-2.12.0/lib/rspec/core/rake_task.rb:122:in `initialize'
    org/jruby/RubyBasicObject.java:1673:in `__send__'
    org/jruby/RubyKernel.java:2081:in `send'
    /Users/filiptepper/.rvm/gems/jruby-1.7.0@boot/gems/rspec-core-2.12.0/lib/rspec/core/rake_task.rb:120:in `initialize'
    org/jruby/RubyProc.java:249:in `call'
    org/jruby/RubyArray.java:1612:in `each'
    org/jruby/RubyArray.java:1612:in `each'
    org/jruby/RubyKernel.java:1045:in `load'
    org/jruby/RubyKernel.java:1065:in `eval'
    /Users/filiptepper/.rvm/gems/jruby-1.7.0@boot/bin/ruby_noexec_wrapper:14:in `(root)'
    Tasks: TOP => spec
    (See full trace by running task with --trace)

Prefork is run after every test (it doesn't happen with MRI).

### JRuby + Nailgun

Commands:

    $ ruby --ng-server

    # Nailgun exits after running spec, rspec output goes to terminal running ng-server
    $ time ruby --ng -S bundle exec rake spec
    ruby --ng -S bundle exec rake spec  0,00s user 0,00s system 0% cpu 47,999 total

    $ time ruby --ng -S rake spec
    ruby --ng -S rake spec  0,00s user 0,00s system 0% cpu 49,603 total

    # Nailgun exits after running spec, rspec output goes to terminal running ng-server
    $ time ruby --ng -S bundle exec rspec spec
    ruby --ng -S bundle exec rspec spec  0,00s user 0,00s system 0% cpu 27,415 total

    $ time ruby --ng -S rspec spec
    ruby --ng -S rspec spec  0,00s user 0,00s system 0% cpu 22,526 total

Output when running `time ruby --ng -S rake spec` more than once:

    rake aborted!
    ActiveRecord::JDBCError: ERROR: database "jruby_test" is being accessed by other users
      Detail: There are 1 other session(s) using the database.: DROP DATABASE IF EXISTS "jruby_test"
    arjdbc/jdbc/RubyJdbcConnection.java:191:in `execute'
    /Users/filiptepper/.rvm/gems/jruby-1.7.0@boot/gems/activerecord-jdbc-adapter-1.2.2.1/lib/arjdbc/jdbc/adapter.rb:219:in `_execute'
    /Users/filiptepper/.rvm/gems/jruby-1.7.0@boot/gems/activerecord-jdbc-adapter-1.2.2.1/lib/arjdbc/jdbc/adapter.rb:211:in `execute'
    /Users/filiptepper/.rvm/gems/jruby-1.7.0@boot/gems/activerecord-3.2.9/lib/active_record/connection_adapters/abstract_adapter.rb:280:in `log'
    /Users/filiptepper/.rvm/gems/jruby-1.7.0@boot/gems/activesupport-3.2.9/lib/active_support/notifications/instrumenter.rb:20:in `instrument'
    /Users/filiptepper/.rvm/gems/jruby-1.7.0@boot/gems/activerecord-3.2.9/lib/active_record/connection_adapters/abstract_adapter.rb:275:in `log'
    /Users/filiptepper/.rvm/gems/jruby-1.7.0@boot/gems/activerecord-jdbc-adapter-1.2.2.1/lib/arjdbc/jdbc/adapter.rb:211:in `execute'
    /Users/filiptepper/.rvm/gems/jruby-1.7.0@boot/gems/activerecord-jdbc-adapter-1.2.2.1/lib/arjdbc/postgresql/adapter.rb:479:in `drop_database'
    /Users/filiptepper/.rvm/gems/jruby-1.7.0@boot/gems/activerecord-3.2.9/lib/active_record/railties/databases.rake:608:in `drop_database'
    /Users/filiptepper/.rvm/gems/jruby-1.7.0@boot/gems/activerecord-3.2.9/lib/active_record/railties/databases.rake:517:in `(root)'
    org/jruby/RubyProc.java:249:in `call'
    org/jruby/RubyArray.java:1612:in `each'
    org/jruby/RubyArray.java:1612:in `each'
    /Users/filiptepper/.rvm/gems/jruby-1.7.0@boot/gems/activerecord-3.2.9/lib/active_record/railties/databases.rake:544:in `(root)'
    org/jruby/RubyProc.java:249:in `call'
    org/jruby/RubyArray.java:1612:in `each'
    org/jruby/RubyArray.java:1612:in `each'
    org/jruby/RubyArray.java:1612:in `each'
    org/jruby/RubyKernel.java:1045:in `load'
    Tasks: TOP => db:test:load => db:test:purge
    (See full trace by running task with --trace)

### JRuby + Nailgun + Spork

Commands:

    $ ruby --ng-server
    $ spork

    # Nailgun exits after running spec, rspec output goes to terminal running ng-server
    $ time ruby --ng -S bundle exec rake spec
    ruby --ng -S bundle exec rake spec  0,00s user 0,00s system 0% cpu 30,581 total

    $ time ruby --ng -S rake spec
    ruby --ng -S rake spec  0,00s user 0,00s system 0% cpu 33,481 total

    # Nailgun exits after running spec, rspec output goes to terminal running ng-server
    $ time ruby --ng -S bundle exec rspec spec
    ruby --ng -S bundle exec rspec spec  0,00s user 0,00s system 0% cpu 9,924 total

    $ time ruby --ng -S rspec spec
    ruby --ng -S rspec spec  0,00s user 0,00s system 0% cpu 3,246 total

Output in `ng-server` after running `time ruby --ng -S bundle exec rake spec`:

    .

    Finished in 0.229 seconds
    1 example, 0 failures

    Randomized with seed 55397

      <-- Slave(2) run done!
    rake aborted!
    /Users/filiptepper/.rvm/rubies/jruby-1.7.0/bin/jruby -S rspec ./spec/example_spec.rb failed
    /Users/filiptepper/.rvm/gems/jruby-1.7.0@boot/gems/rspec-core-2.12.0/lib/rspec/core/rake_task.rb:154:in `run_task'
    /Users/filiptepper/.rvm/gems/jruby-1.7.0@boot/gems/rspec-core-2.12.0/lib/rspec/core/rake_task.rb:122:in `initialize'
    org/jruby/RubyBasicObject.java:1673:in `__send__'
    org/jruby/RubyKernel.java:2081:in `send'
    /Users/filiptepper/.rvm/gems/jruby-1.7.0@boot/gems/rspec-core-2.12.0/lib/rspec/core/rake_task.rb:120:in `initialize'
    org/jruby/RubyProc.java:249:in `call'
    org/jruby/RubyArray.java:1612:in `each'
    org/jruby/RubyArray.java:1612:in `each'
    org/jruby/RubyKernel.java:1045:in `load'
    org/jruby/RubyKernel.java:1065:in `eval'
    /Users/filiptepper/.rvm/gems/jruby-1.7.0@boot/bin/ruby_noexec_wrapper:14:in `(root)'
    Tasks: TOP => spec
    (See full trace by running task with --trace)

Output after running `time ruby --ng -S rake spec`:

    .

    Finished in 0.207 seconds
    1 example, 0 failures

    Randomized with seed 24351

      <-- Slave(2) run done!
    rake aborted!
    /Users/filiptepper/.rvm/rubies/jruby-1.7.0/bin/jruby -S rspec ./spec/example_spec.rb failed
    /Users/filiptepper/.rvm/gems/jruby-1.7.0@boot/gems/rspec-core-2.12.0/lib/rspec/core/rake_task.rb:154:in `run_task'
    /Users/filiptepper/.rvm/gems/jruby-1.7.0@boot/gems/rspec-core-2.12.0/lib/rspec/core/rake_task.rb:122:in `initialize'
    org/jruby/RubyBasicObject.java:1673:in `__send__'
    org/jruby/RubyKernel.java:2081:in `send'
    /Users/filiptepper/.rvm/gems/jruby-1.7.0@boot/gems/rspec-core-2.12.0/lib/rspec/core/rake_task.rb:120:in `initialize'
    org/jruby/RubyProc.java:249:in `call'
    org/jruby/RubyArray.java:1612:in `each'
    org/jruby/RubyArray.java:1612:in `each'
    org/jruby/RubyKernel.java:1045:in `load'
    Tasks: TOP => spec
    (See full trace by running task with --trace)

Output after running `time ruby --ng -S rspec spec`:

    org.jruby.exceptions.RaiseException: (SystemExit) exit