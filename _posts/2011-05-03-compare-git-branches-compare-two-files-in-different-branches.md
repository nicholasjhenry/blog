---
title: Compare Git Branches, Compare Two Files in Different Branches
canonical_url: https://blog.firsthand.ca/2011/05/compare-git-branches-compare-two-files.html
---
Git offers a couple of great commands for comparing branches. To get a comparison of branches in a "status" type format:

``` bash
git diff --name-status branch1..branch2
```

Will give you output something like:

``` bash
M       config/boot.rb
M       config/database.yml
M       config/deploy.rb
M       config/environment.rb
M       config/environments/development.rb
M       config/environments/production.rb
M       config/environments/staging.rb
```

Now that you have a list of files, you may wish to get a diff of a specific file:

``` bash
git difftool branch1:config/environment.rb branch2:config/environment.rb
```

On OSX, this will open the awesome FileMerge.app allowing you to compare both files.

![File merge]({{ '/assets/posts/2011-05-03-compare-git-branches-compare-two-files-in-different-branches/file-merge.png' | relative_url }})
