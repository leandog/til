# Including Only a Subset of Associated Records

Recently, my pair and I were implementing a search feature for administrators to find customer records. In our app, customers can have many "affiliations" with external partners, and it's common to search for a customer by their ID in one of these external systems. So if an administrator searches with one of these external IDs, we want to display the matching ID in the search results so they can know they found the right record.

For example, imagine our database contains two customers, Bob and Alice. Bob has two affiliations, one with ID 1111 and another with ID 2222. When the administrator searches for customers with the ID "2222", we want to display the following:

```
Results: (1 of 2)
  Bob (affiliate #2222)
```

If we're not careful, however, we might easily write our code such that the page simply displays Bob's first affiliation number, rather than the one the admin was searching for:

```html.erb
<% search_results.each do |customer| %>
  <li><%= customer.name %> (affiliate #<%= customer.affiliations.first.id %>)</li>
<% end %>
```

```
Results: (1 of 2)
  Bob (affiliate #1111)
```

This is confusing! The admin searched for customer with ID 2222, but he got the customer with ID 1111. Of course, we could display *all* of the customers' affiliations, but that could quickly get unwieldy.  We want to preferentially display the ID the administrator searched for so they know they got the right customer.

We could do this in Ruby, a la:

```html.erb
<% search_results.each do |customer| %>
  <li><%= customer.name %> (affiliate #<%= customer.affiliations.find { |a| a.id =~ /#{params[:affiliate_id]}/ } %>)</li>
<% end %>
```

But that sucks. We can do better!

Anyone who knows SQL will tell you that what we want to accomplish is easy with JOINs:

```sql
SELECT *
FROM customers c
  INNER JOIN affiliations a on a.customer_id = c.id
WHERE a.affiliation_id = '2222';
```

and ActiveRecord lets us do JOINs:

```ruby
customer =
  Customer
    .joins(:affiliations)
    .where(affiliations: { affiliation_id: "2222" })
```

But look at what happens when we read the customer's affiliations:

```ruby
customer.affiliations
# => [<Affiliation id: 1, affiliation_id: "1111", ...>,
#     <Affiliation id: 2, affiliation_id: "2222", ...>]
```

Despite our `where` clause to filter out the unwanted affiliations, we still get all of them when we call `customer.affiliations`.

The problem is that we're missing a piece in our ActiveRecord query. The `joins` call allows us to reference affiliations in our `where` call, but ActiveRecord will only instantiate Customer records from the query results. So when we call `affiliations` on the Customer instance, ActiveRecord will go query the database again without our `WHERE` clause from before.

To get only the associated records we want, we can leverage another method in ActiveRecord: `includes`

As you may know, `includes` is a method that allows us to ["eager load"][1] associated records. This method is usually employed to avoid the ["N+1 queries" problem][2], but we can use it here to basically trick ActiveRecord into retrieving only a subset of the customer's affiliations:

```ruby
customer =
  Customer
    .joins(:affiliations)
    .where(affiliations: { affiliation_id: "2222" })
    .includes(:affiliations)

customer.affiliations
# => [<Affiliation id: 2, affiliation_id: "2222", ...>]
```

It works! We get all the customers with affiliations matching the given id, and we didn't get any of the other affiliations of the customer.

The reason this works is that `includes` tells ActiveRecord to instantiate the customer's affiliations association using the data included via the `JOIN` clause in the query. ActiveRecord will do and then cache the associated records so that when you call `affiliations` on a Customer it does not query the database again. This caching behavior is what allows `includes` to address the N+1 problem and also what allows us to retrieve only a subset of a record's association.

[1]: http://guides.rubyonrails.org/active_record_querying.html#eager-loading-associations
[2]: https://secure.phabricator.com/book/phabcontrib/article/n_plus_one/
