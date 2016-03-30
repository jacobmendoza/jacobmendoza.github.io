---
layout: post
title:  "rspec-tree-runner"
date:   2016-03-11 17:40:57 +0000
categories: GITHUB-ATOM COFFEESCRIPT RSPEC RUBY RAILS
---

# Beginning
<a href="https://atom.io/packages/rspec-tree-runner" target='blank'>rspec-tree-runner</a> is a learning experiment. I wanted to do something with technologies that I do not use every day and that could be directly used by people.

![Image alt]({{ site.baseurl }}/assets/rspec-tree-runner.png "image title")

I'm used to IDEs and although I feel comfortable with the terminal, I surely appreciate some integration with the editor.

The goal was build a complete test runner for **RSpec** and **Ruby on Rails** that worked integrated with the editor, with a good UI and some extras that
help the programmer to get into the TDD flow, easily jumping between the tests and the tested file taking advantage of the Rails conventions.

Also, I wanted to provide the ability to see the structure of the spec file before running the tests. In the vast majority of the
test runners that I have used, there is a process of discovery and parsing of the files. The structure of the tests can be seen when you open the file.

# Atom
<a href="https://atom.io/" target='blank'>Atom</a> is a text editor built by GitHub. The software is completely built under web technologies working over <a href="https://github.com/atom/electron" target='blank'>Electron</a>,
a framework that helps creating cross-platform applications with Javascript, CSS and HTML.

From the web developer point of view that means a very standard toolset to which you are already used, and a fairly reasonable learning curve.
Add to the equation a good package manager (NPM), a well structured API and decent documentation.

The defacto option for Atom packages is CoffeeScript, traditionally used by GitHub (the software that runs the platform was originally written using Ruby on Rails). However, it does not mean that you cannot decide the technologies for your plugin. Plain Javascript or even ES2015 with Babel?. Your choice.

If you are going to use CoffeeScript, do yourself a favor and try to learn the basics. I always try to find a balance
between learning and delivery in my side projects (trying to maintain some bias for action even at home), but spending
some time studying the basics will worth your time. In the <a href="http://coffeescript.org/" target='blank'>CoffeeScript webpage</a> there
is a section to try the language, very useful to test ideas quickly.

# The RSpec spec file
I like RSpec. It offers a beautiful syntax for BDD that leads to clear specification of the subject under test and is very well documented. Let's see a small example:

<script src="https://gist.github.com/jacobmendoza/ab4eff3fc77f8699f2dc.js"></script>

When changing the file in Atom, the package is intended to show the hierarchical structure in the right pane. That means reading the file and parsing its contents.
In order to do that, the package contains a <a href="https://github.com/jacobmendoza/rspec-tree-runner/blob/master/spec-analyzer/spec_analyzer_script.rb" target='blank'>Ruby script</a> that is used every time that we need to parse a file.
It relies on Ripper, a Ruby script parser. When analysing the file with Ripper, we obtain a symbolic expression tree,
basically a way to express a binary tree that in this case is going to contain specific information relative to the parsing process.

<script src="https://gist.github.com/jacobmendoza/164c2c0a713be3ade49e.js"></script>

Let's just say it's not beautiful. We process it from this form that contains information relative to the parsing process
to a tree meaningful in the context of the package, a format that contains information about the tests. In order to do so, we process nodes as
'describe', 'context', 'it', 'feature', 'scenario' and decorate them with specific information, as the line number that they occupy
in the RSpec file.

# Rendering the tree
As said at the beginning of the post, Atom works under web technologies, and that means that the UI is built with HTML/LESS and rendered by Electron.
Manually operating over the DOM in order to get your UI working could be quite painful depending on the level of complexity that you may be managing. <a href="https://github.com/atom-archive/space-pen" target='blank'>Atom SpacePen</a> is supposed to help you with that.

You just need to subclass View which descends from the jQuery prototype. That's extremely useful for DOM manipulation and provides an already known library
for most developers.

# Running the tests and the TDD flow

One of the main goals of the project was try to make easier do TDD in a Rails project with Atom.
Toggle between the test and production code easily helps to get into the flow. I got the inspiration thanks
to <a href="https://github.com/wangyuhere/atom-rails-rspec">this</a> package by <a href="https://github.com/wangyuhere">Yu Wang</a>. Thanks to the
Rails conventions (production code is under 'app' or 'lib' and test code with RSpec is under the spec folder with the same structure. The spec files
have the same name with the '_spec' suffix) is quite easy to find the corresponding file.

The plugin has a keymap that assigns different keystrokes to commands. Atom has a nice way of configuring <a href="http://flight-manual.atom.io/behind-atom/sections/keymaps-in-depth/">these keymaps</a>.
It is easy to run into conflicts between different packages and the <a href="https://atom.io/packages/keybinding-resolver">keybinding resolver</a> is a very useful piece for that matter.

# Future

Developing the package was a nice learning experience. If you are an experienced Ruby or CoffeeScript programmer you will surely
find a lot of things that can be improved (<a href="https://github.com/jacobmendoza/rspec-tree-runner/issues" target='blank'>let me know!</a>). I will try to clean things a bit.

Using the Ruby conventions was helpful. The problem is that it also acts as a limitation, constraining some
testing scenarios. It is something to review and probably redesign.
