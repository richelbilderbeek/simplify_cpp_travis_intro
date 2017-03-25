# simplify_cpp_travis_intro

Article for 'Simplify C++!', 'Continuous integration with Travis CI'.

The article is in progress and spontaneous feedback is welcomed. Feel encouraged to do, by posting an Issue.

# Continuous integration with Travis CI

Minecraft players can tell you: automation is great! It usually takes some time to set up, but then
you profit indefinitely. But where my minor programming students set up some Minecraft farming
contraption, I like to harvest as much information from my code that I get get. This ranges from
the basic question 'Does it build (today)?' to getting code coverage, profiling information and whatever
I choose.

This article introduces how to add continuous integration and other goodies to your code.

## Advantages of setting up a continuous integration service

There are many advantages of setting up a continuous integration service:

 * Errors are detected directly after a commit
 * The build script shows all steps to build your project, and is tested to work
 * Quality can be enforced in incoming Pull Requests

## Disadvantages of setting up a continuous integration service

There are some potential disadvantages of setting up a continuous integration service:

 * Disruption of the work process due to a false negative (for example, a Heisenbug)
 * It takes some time to set up
 * For commercial development, a continuous integration service comes with a price

## Overview

This article will show the setups:

 * Building a C++98 Hello World program
 * Building a C++17 Hello World program
 * Adding static code analysis by `cppcheck`
 * Measuring code coverage
 * Profiling
 * Adding OCLint

## The tools I use are just an example

This article shows some advantages using a continuous
integration service. To have complete examples, I had
to pick some specific programs/websites/services out of many.

In the table below, you can see the tools I picked for this article, and alternatives
that probably are just as good:

Tools I will be using|Examples of alternatives that are just as good
---|---
GCC|clang
qmake|autoconf, cmake
cppcheck|clang-tidy, OCLint
Git|BitKeeper, Gnu Bazaar, Mercurial
GitHub|Bitbucket, Sourceforge,GNU Savannah
Travis CI|Jenkins, Appveyor, Codeship, Wercker


## Building a C++98 Hello World program

The first example shows how to test that a C++98 Hello World program compiles.

This is the code that will be built:

```
#include <iostream>

int main() 
{
  std::cout << "Hello world\n";
}
```

Travis needs to be instructed what to do when triggered.
These instruction must be put in a file named `.travis.yml`
and looks like this:

```
language: cpp
compiler: gcc

script: 
  - g++ main.cpp -o travis_gcc_cpp98
  - ./travis_gcc_cpp98
```

First, the language and compiler are specified. In the `script` part, the project is built and then run.

When Travis CI is triggered, it will result in the following build log:

![Travis CI build log of the C++98 Hello World program](travis_gcc_cpp98.png)

From this day on, we have code that is checked to compile everytime the code changes.
Also, the same happens if a Pull Request is sent to us: already before merging, GitHub
shows if that merge will break the build.

## [Building a C++17 Hello World program ](https://github.com/richelbilderbeek/travis_gcc_cpp17) 

To celebrate C++17 has been completed [1], I will add a C++17 feature:

```
#include <iostream>

int main() 
{
  static_assert("C++17");
  std::cout << "Hello world\n";
}
```

To build this project successfully, a more elaborate Travis build script is needed:

```
language: cpp
compiler: gcc
dist: trusty

before_install:
  - sudo add-apt-repository -y ppa:ubuntu-toolchain-r/test
  - sudo apt-get update -qq

install: 
  - sudo apt-get install -qq g++-6
  - sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-6 90

script: 
  - g++ main.cpp -std=c++17 -o travis_gcc_cpp17
  - ./travis_gcc_cpp17
```

Travis will setup a newer distro (Trusty Tahr), instead of the one used by default (Precise Pangolin).
A PPA is added for (even) newer packages and `g++-6` is installed. Then `g++` is redirected to use `g++-6`.

This Travis script is one of many ways to achieve the same and not necessarily the best in all aspects. 
Feel encouraged to suggest improvements to any of the GitHubs used in this article. As when you'd
submit me a Pull Request, Travis will show me both if this C++17 code still compiles and how
long it took to run the script.


## Adding static code analysis by `cppcheck`

Spot the bug:

```
int main() 
{
  static_assert("C++17");
  int a[3] = { 0, 1, 2 };
  a[3] = 4;
}
```

For an experienced C++ developer, this is all too easy. 
But for someone stepping in from another language, it may be not,
as not all programming languages have arrays starting at index zero. 
Additionally, access violations are usually more conspicuous. 

A static code analysis tool should be able to detect this access violation
and give a friendly reminder to the person that wrote the code.

This Travis script tests the project to build and tests the code statically with `cppcheck`:

```
language: cpp
compiler: gcc
dist: trusty

before_install:
  # C++17
  - sudo add-apt-repository -y ppa:ubuntu-toolchain-r/test
  - sudo apt-get update -qq

install: 
  # C++17
  - sudo apt-get install -qq g++-6
  - sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-6 90
  # cppcheck
  - sudo apt-get install -qq cppcheck

script: 
  # Build and run this project
  - g++ main.cpp -std=c++17 -o travis_gcc_cpp17_cppcheck
  - ./travis_gcc_cpp17_cppcheck
  # cppcheck
  - cppcheck --quiet --error-exitcode=1 . 
```

Since the previous build script, little has changed. In the `install` section, `cppcheck` is installed.
In the `script` section, `cppcheck` is called to test the files in its folder.

This is the Travis output:

![cppcheck finds the access violation](travis_gcc_cpp17_cppcheck.png)

Great, Travis and `cppcheck` have detected this problem.
Due to this, the Travis build has failed.
The person that submitted this commit may have been reminded that in C++ indices at zero.

Adding static code analysis tools to a Travis scripts do not only help experts. 
As in this example, it may help teach beginners. It will give them a friendly reminder
about how the language works, instead of a segmentation fault that may-or-may-not occur.


## External links

 * [My Travis CI C++ tutorial](https://github.com/richelbilderbeek/travis_cpp_tutorial)
 * [GitHub of 'Building a C++98 Hello World program'](https://github.com/richelbilderbeek/travis_gcc_cpp98) 
 * [GitHub of 'Building a C++17 Hello World program'](https://github.com/richelbilderbeek/travis_gcc_cpp17) 
 * [GitHub of 'Adding static code analysis by `cppcheck`'](https://github.com/richelbilderbeek/travis_gcc_cpp17_cppcheck)

## References

 * [1] [Trip report: Winter ISO C++ standards meeting (Kona), C++17 is complete](https://herbsutter.com/2017/03/24/trip-report-winter-iso-c-standards-meeting-kona-c17-is-complete)


# Guidelines from 'Simplify C++' (to be removed)

Your post

The topic of your post should fit well into the blog, i.e. it should be about something that enables us to write cleaner (C++) code, better tests etc. It should, however, not be about a specific commercial product. Please understand that this is not the right place for marketing for companies or commercial individuals. The length of my posts usually is roughly between 600 and 1200 words. Yours should not be much shorter. If it becomes much longer, we should discuss if we can split it into two parts or if the scope can be reduced.

Structure

You are free to structure your post as you see fit. The first heading should be an h1 heading, but it does not need to be before the first paragraph of your post. Be aware, that the table of contents for your post will be generated according to your headings. I will add a short paragraph as an introduction where I’d like to include a short bio of you, so please leave me two or three sentences describing what you do and where people can find you online.

Formatting

I use a MarkDown plugin for formatting. See this MarkDown cheatsheet if you are not familiar with MarkDown. You can also use a few HTML tags, like <h1>, <h2> etc. for headers, <a> for links and <strong> and <em> for bold and italic text, respectively. Keep your paragraphs and sentences short for better readability. If you want to draft your post offline, please paste it as MarkDown or plain text, not as preformatted HTML, like you get it from e.g. MS Word. Such HTML contains lots of unnecessary styles that mess with the blog style and needs some tedious editing.

I don’t have a code formatting plugin like Crayon, because it does not cope well with the MarkDown plugin, so you will have to write code snippets in plain MarkDown. Simply indent the code by 4 spaces, or add a line with the usual triple backticks before and after your code.

Review

When you are done, I will proofread and review your post. I might give you some hints where I think a different wording might be better, and I will fix the spelling and formatting when I find errors. It will help both of us if you use a spell checker, though, and I also find tools like Grammarly quite useful for non-native speakers like myself.

In the review phase, I will also add that introductory paragraph. After that, I will ask you to approve my changes one last time.

Schedule

Your post will be part of the regular blog schedule, which is currently Wednesdays, 3 pm UTC. I will usually schedule your post to be published in the next two weeks after the review and your approval. On rare occasions, I might already have scheduled posts on those dates that I can not reschedule. In those cases, your post will be scheduled as quickly after those fixed posts as possible.

And now, welcome aboard and happy blogging!