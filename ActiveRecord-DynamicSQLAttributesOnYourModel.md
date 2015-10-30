# Building a Leaderboard Query with Dynamic SQL Model Attributes

Recently, I was asked how to structure a query for a game leaderboard. In SQL,
you could write a query like this:

```sql
SELECT u.*, max(SELECT final_score FROM games WHERE user_id = u.id) as high_score
FROM users u
ORDER BY high_score DESC;
```

But how would you translate that query to ActiveRecord? There solution involves
leveraging a little-known feature of ActiveRecord: dynamic model attributes.

Let's assume a User model like this:

```ruby
class User < ActiveRecord::Base
  has_many :games

  def high_score
    games.order(final_score: :desc).first
  end
end
```

### The n+1 query problem

You definitely *don't* want to do this:

```ruby
@users = Users.all.sort_by(&:high_score)
```

Since `high_score` will issue a query to fetch the user's game records. You'll
end up issuing an extra query for each user returned by your initial query. So
how can we avoid this pitfall?

### Using `includes` doesn't cut it

Whenever you have an n+1 query problem, `ActiveRecord::Base#includes` should be
your first line of defense. Using this method will cause ActiveRecord to
query for records related to the base query all at once, rather than one at a
time.

However, in our case, since `high_score` involves ordering the records, the
records cache created by `includes` will miss, and we'll still issue n+1
queries:

```ruby
# NOTE: Still issues n+1 queries
@users = Users.includes(:games).sort_by(&:high_score)
```

At times like these, I'm sad that ActiveRecord doesn't support syntax like the
following (due to the fact that `includes` uses two separate queries to retrieve
the record sets, instead of getting them all at once with a SQL JOIN):

```ruby
# NOTE: this doesn't work
@users = Users.includes(:games).order(games: { high_score: :desc })
```

So what can we do if `includes` fails us?

### Dynamic SQL attributes to the rescue!

Ever wonder why you don't need to declare attributes in your model classes? No,
it's not because of the db/schema.rb file. It's because ActiveRecord actually
dynamically defines methods on your model instances based on what columns the
database returns from your query.

We can use this fact to great effect for our leaderboard:

```ruby
@users =
  User.select("*, max(SELECT max(score) FROM games WHERE user_id = users.id) AS max_score")
      .order("max_score")
```

This structure allows us to issue the exact SQL query we began with, so we
avoid the n+1 query problem while still retrieving the high score for each
user! And yes, each user instance in `@users` has a method `max_score` that
will return that user's high score.

Two gotchas to note:

1. If we aliased the subquery result as `high_score`, our method definition
in the User model would override this subquery result
2. The argument to `order` must be a string in this case. A symbol will be
prefixed with the `users` table name in the resulting query, and will fail.

There are many other uses for these dynamic model attributes from SQL beyond
subquery results. You can write any valid SQL you want inside most of
ActiveRecord's query methods. This in combination with the dynamic nature of
the library provides a powerful and flexible tool in your Ruby toolchain.
