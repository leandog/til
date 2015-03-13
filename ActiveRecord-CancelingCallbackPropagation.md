# Canceling ActiveRecord Callback Propagation

My pair and I needed to default the value of a boolean field to false if not specified.

Normally, the best way to do this would be to add that default value to your database schema and make sure it's not null. For example:

```ruby
# migration file...
change_column :promotion, :reusable, default: false, null: false
```

This way, ActiveRecord will automatically set `reusable` to false for all new `Promotion` instances.

However, we were using [STI][1] for our promotions, and only certain types of promotions need to have a value in this field. It would be cleaner to leave other promotion records with a `null` value for that column, rather than polluting these records with data they don't need.

So rather than altering our database schema, we decided to make use of [ActiveRecord callbacks][2]. We added a `before_save` hook that sets `reusable` to false if not specified:

```ruby
# app/models/promotions/first_year_free.rb
class Promotions::FirstYearFree < Promotion
  before_save do |record|
    record.reusable ||= false
  end
end
```

Seems simple enough, right?

After we added this code, nearly all of our unit tests for this class failed with this weird error: `ActiveRecord::RecordNotSaved`. The stack traces pointed back to our lines of code that were supposed to save our model. However, the models didn't have any validation errors to indicate a reason for these failures.

After much trial and error, we discovered the problem: **ActiveRecord callbacks can stop the propagation of the current event by returning `false`.**

`before_*` callbacks that return `false` will prevent the action from continuing, and will result in raising the `ActiveRecord::RecordNotSaved` error. `after_*` callbacks that return `false` will prevent subsequent `after_*` callbacks from being executed, and does not raise an error. Some sparse documentation on this ~~atrocity~~ feature can be found [here][3].

To fix our issue, we simply altered our callback to return something other than `false`:

```ruby
# app/models/promotions/first_year_free.rb
class Promotions::FirstYearFree < Promotion
  before_save do |record|
    record.reusable ||= false
    nil
  end
end
```

and our tests were green again!

[1]: http://api.rubyonrails.org/classes/ActiveRecord/Base.html#class-ActiveRecord::Base-label-Single+table+inheritance
[2]: http://guides.rubyonrails.org/active_record_callbacks.html
[3]: http://api.rubyonrails.org/classes/ActiveRecord/Callbacks.html#module-ActiveRecord::Callbacks-label-Canceling+callbacks
