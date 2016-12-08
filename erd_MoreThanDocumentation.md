# The `erd` Gem -- It's Not Just for Documenting Your Codebase

An [ERD](https://en.wikipedia.org/wiki/Entity%E2%80%93relationship_model) -- Entity Relationship Diagram (or Model) -- can be useful in both understanding and explaining a system. It shows the *things* and the *relationships*, usually in terms of data, that a system needs in order perform it's business functions. (It doesn't usually display the *functions* of the system.)

The [`erd` Ruby Gem](https://github.com/amatsuda/erd) can generate these diagrams from an existing project's code and database. It makes fairly nice-looking PDFs containing the diagram. This is the retro-active use case of this gem.

While thinking and brainstorming about an upcoming project, I stumbled into a nice functionality of that gem. I had a skeleton project up and running but got a Rails' routing error, the kind that renders the entire list of routes on the screen. Scrolling through the list, this really jumped out at me:

    erd_path      /erd      Erd::Engine

"What's that?" \[type...click\] "Wow!"

The app rendered an ER diagram similar to the PDF versions I was used to seeing, with one major difference -- a button labeled "Create Model."

"What does that do?" \[click...type...click\] "Wow!"

The `Erd::Engine` lets you make changes to your schema. You can add or alter columns on existing tables. You can add new tables. It generates migration and class files that implement your changes right in your project.

Pretty. Darn. Cool.

(You'll still need to run `rake db:migrate` on the command line, but when you refresh the screen you'll see your new entities.)

Using `erd` like this to pro-actively *could* be the way of the future. It's certainly a handy tool to have to visualize and think about how you're designing a system.