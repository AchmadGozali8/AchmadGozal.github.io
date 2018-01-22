---
title:  "Mysqld doesn't listen any port in slackware"
date: 2018-01-22 22:03
categories: [mysql, mysqld, slackware]
tags: [mysql, mysqld, slackware]
---
Why mysqld not listen any port whyyy??????

Networking is disabled by default to improve security. If you want to allow network connections, comment out this line in /etc/rc.d/rc.mysqld:

```
#SKIP="--skip-networking"
```

so you will be get mysqld run in mysql daemon default port (3306)
