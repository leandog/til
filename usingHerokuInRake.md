# using heroku in rake

While doing some ETL with heroku, I found myself trying to script pulling down the database from heroku.

From the cmd line you'd do something like this:

```bash
heroku pg:backups capture
curl -o db.dump `heroku pg:backup public-url`
```

So hey, let's put in a rake task so we don't forget how to do that:
```ruby
    task :fetch_db do
      `heroku pg:backups capture`
      system "curl -o db.dump `heroku pg:backups public-url`"
    end
```

Uhoh.

```bash
/Users/stevejackson/.rvm/gems/ruby-2.1.3@global/gems/bundler-1.7.3/lib/bundler/definition.rb:385:in `validate_ruby!': Your Ruby version is 1.9.3, but your Gemfile specified 2.1.3 (Bundler::RubyVersionMismatch)
```

It turns out heroku has a ruby version specified in its internals somewhere and that conflicts with the project Gemfile.

Luckily, bundler has a solution:
```ruby
    task :fetch_db do
      Bundler.with_clean_env { `heroku pg:backups capture` }
      Bundler.with_clean_env { system "curl -o db.dump `heroku pg:backups public-url`" }
    end
```

