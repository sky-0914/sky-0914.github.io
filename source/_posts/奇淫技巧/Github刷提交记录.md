---
title: Github刷提交记录
tags:
  - Github
categories:
  - 奇淫技巧
---
# Github刷提交记录

https://github.com/sky-0914/git-auto-commit

00 12 * * * cd /home/github && git pull && /usr/bin/node add.js && git commit -a -m 'git auto commit' && git push origin master

*/5 * * * * cd /home/github/test-project && git pull && /usr/bin/node add.js && git commit -a -m 'git auto commit' && git push origin master

5 5 0/3 * * cd /home/github/test-project && git pull && /usr/bin/node add.js && git commit -a -m 'git auto commit' && git push origin master

