---
title: Software Craftsmanship Goes Beyond Code
canonical_url: https://blog.firsthand.ca/2011/11/software-craftsmanship-goes-beyond-code.html
---
Often the discussion of Software Craftsmanship is focused on code and tests. But don't forget about the other components that support your project:

- Make sure your **source control repository is organized**. Don't leave dead branches lying around, left for another developer to work out if they have been merged or not; **if** they should be merged or not. Add a reference to your branch with the issue or story id from your tracker so a developer can read more about the intention of the branch.
- And [don't write commit messages that suck](http://confreaks.net/videos/744-rockymtnruby2011-lightning-talk-do-your-commit-messages-suck).
- Keep your **tracker up-to-date and lean**. Don't mark stories as started when they haven't been. In reality, there should only be one story marked as started per developer on the project. Don't drag stories into the backlog just because you think they are a good idea. Keep the backlog lean, you only need a couple of weeks worth of stories to keep you going. Keep the rest in the icebox. Priorities are going to change.
- Make sure it's easy for a developer to **get started with your project**. Have a [README that guides a developer](http://blog.firsthand.ca/2010/09/24hrs-firsthand-leave-project-how-you.html) to get the dependencies and database setup, and remind them a test suite exists and how to run it. Obviously the more automated things are the better.

Caring needs to go beyond the code, it needs to include all the supporting components of your project. Of course there are other facets of professionalism we can list, but these are some practical details I have experienced recently. What components of a project, beyond code, that you wish developers would give more care to? Let me know in the comments.
