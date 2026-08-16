---
title: Leave a project how you would like to find it
canonical_url: https://blog.firsthand.ca/2010/09/24hrs-firsthand-leave-project-how-you.html
---
I started a new CMS project yesterday with [Radiant](http://www.radiantcms.org). One aspect I am really focusing on is to ensure that all the "other" details are taking care of during the development, so **when** another developer needs to work on it, they can get started on it as quickly as possible.

The new [Bundler](http://gembundler.com/) and Rails 2.x `config.gem` have certainly helped with installing dependent gems, but I still think it's beneficial to have a step by step guide in the form of a README file on getting the application running and any other aspects of the application the developer should know about.

[GitHub](http://www.github.com) presents the README file of the homepage of the repository.

![README file on Github]({{ '/assets/posts/2010-09-02-leave-a-project-how-you-would-like-to-find-it/firsthand-leave-project-how-you.png' | relative_url }})

**Please replace the default README file in your** Rails or Radiant application (or any other open source application). Here's a basic outline of what you could include:

- Application setup and database bootstrapping
- Testing frameworks used and how to run the tests
- References used on the project (for example, if you used a third-party API, link to those documents)
- Any other interesting notes that you discovered during development, but would have liked to have known before you got started
- Populating the database with "demo" data
- How to easily preview application emails (hint: rake tasks)

Are there any other items that should be in the README? Do you have any links to examples of great README's?
