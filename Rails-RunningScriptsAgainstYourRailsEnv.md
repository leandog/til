# Running Ruby Scripts Against Your Rails Environment

Ever needed to do something in a Rails environment that was too hairy for a
REPL but it still didn't merit a rake task?

My pair and I had a chore to run in production to kick off some asyncronous
processing for about 4000 records. This was a one-off task that we didn't need
to codify in a rake task for the future, but it was a little too complicated to
quickly develop in the Rails console.

We were thinking through the code for the task in a text file on our machine
and realized it would be great if we could simply execute the file we had
written against the Rails enviroment. Of course, our script could require our
rails environment itself, but we weren't sure exactly which file(s) to require
to accomplish that.

A quick `rails --help` mentioned the existence of the `rails runner` command.
It turns out, this tool does exactly what we needed! It loads your Rails
environment (as specified by the `RAILS_ENV` environment variable or the `-e`
option) and executes some code against it.

It can take a string to `eval`:

```
$ rails runner -e test 'puts Rails.env'
  => test
```

Or a script file to process:

```
$ echo 'puts Rails.env' > my_script_file.rb
$ RAILS_ENV=test rails runner my_script_file.rb
  => test
```

Check out the [Rails Guides][guides] for more details

[guides]: http://guides.rubyonrails.org/command_line.html#rails-runner
