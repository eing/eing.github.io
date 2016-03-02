---
layout: post
title:  "Git pre-commit hooks Part 1"
date:   2016-01-28 07:43:59
author: Eing Ong
categories: technology
tags: git maven java jmockit
---
Git pre-commit hook is simply a script that is placed in every git project under {project_root}/.git/hooks/pre-commit. By default, .git/hooks directory is generated for every git project, containing all sample hooks. To activate it, you just have to remove the extension ".sample". Here's picture of my local .git/hooks directory,

![Sample git hooks](/assets/gitHooks/sampleHooks.png)

Now you notice, a file with a symlink "pre-commit@", it's a file linking to "../../pre-commit.sh". Since ".git/hooks" is local to every developer, you'll need to maintain a file that is source code controlled and pre-commit.sh at the project root is what you commit and maintain.  

Next section is what my pre-commit.sh looks like and how I created a script that every developer executes to set up this pre-commit hook on their local development machine.

<h2>Creating a simple git pre-commit hook</h2>
Here's my pre-commit.sh that can also be placed in .git/hooks/pre-commit to be a pre-commit hook. Instead of "mvn clean install test", you can substitude it with any checks you want to run and if you use gradle, place "gradle test" instead.

~~~
git stash -q --keep-index
# Using "mvn test" to run all unit tests and run plugins to assert
#   * code coverage threshold >= 85% (using surefire, enforcer plugins)
#   * FindBugs at low threshold errors (using findbugs-maven-plugin)
#   * Checkstyle has 0 errors (using maven-checkstyle-plugin)
mvn clean install test
RESULTS=$?
# Perform checks
git stash pop -q
if [ $RESULTS -ne 0 ]; then
  echo Error: Commit criteria not met with one or more of the following issues,
  echo 1. Failure(s) in unit tests
  echo 2. Failure to meet 85% code coverage
  echo 3. Failure to meet low FindBugs threshold
  echo 4. Failure to meet 0 Checkstyle errors
  exit 1
fi
# You shall commit
exit 0
~~~
<h2>Creating an installation script</h2>
To create an executeable script that developers can run to create a symlink to pre-commit.sh, and also to create the pre-commit.sh, here's an install_precommit.sh script.

~~~
{ echo "
git stash -q --keep-index
# Using "mvn test" to run all unit tests and run plugins to assert
#   * code coverage threshold >= 85% (using surefire, enforcer plugins)
#   * FindBugs at low threshold errors (using findbugs-maven-plugin)
#   * Checkstyle has 0 errors (using maven-checkstyle-plugin)
mvn clean install test
RESULTS=\$?
# Perform checks
git stash pop -q
if [ \$RESULTS -ne 0 ]; then
  echo Error: Commit criteria not met with one or more of the following issues,
  echo 1. Failure\(s\) in unit tests
  echo 2. Failure to meet 85% code coverage
  echo 3. Failure to meet low FindBugs threshold
  echo 4. Failure to meet 0 Checkstyle errors
  exit 1
fi
# You shall commit
exit 0"
} > pre-commit.sh
pushd .git/hooks
ln -s ../../pre-commit.sh pre-commit
chmod u+x pre-commit
popd
~~~
To execute this install_precommit.sh script,

~~~
$ cd {project root}
$ chmod +x install_precommit.sh
$ ./install_precommit.sh
~~~

Now you have completed adding a git pre-commit client hook.

In my next blog, I'll show the use maven plugins for checkstyle, findbugs & surefire plugins to enforce your pre-commit checks.
