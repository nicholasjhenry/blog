---
title: Have you tried ctags with Vim?
canonical_url: https://blog.firsthand.ca/2012/11/have-you-tried-ctags-with-vim.html
---
After reading Mislav Marohnić's excellent post on [Vim](http://mislav.uniqpath.com/2011/12/vim-revisited/), I decided to add ctags to my development environment. What does it buy you? It allows you to jump to method or class definitions with these commands:

```text
<C-]> / :tag foo # jump to tag under cursor/named
<C-]> / :tjump foo # choose from a list of matching tags
```

Setup is pretty easy:

```bash
brew install ctags
```

You can generate tags with:

```bash
ctags -R --languages=ruby --exclude=.git
```

But you probably won't want to do all that manually. As Mislav points out in his post, Tim Pope shows us how to [automate tag generation with git hooks](http://tbaggery.com/2011/08/08/effortless-ctags-with-git.html).

I've been using ctags this afternoon and already it has turned Vim into a completely different editor for me.
