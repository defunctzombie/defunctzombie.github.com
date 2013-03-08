---
title: cron SHELL power
layout: post
---
If you don't know what [cron](http://en.wikipedia.org/wiki/Cron) is this post is not for you.

Using the `SHELL` variable in cron is more powerful than you may realize.

## typical crontab

Most people will have this type of setup in their crontab.

```
NODE_ENV=production
OTHER_VAR=foo

*/10 * * * * /path/to/node /path/to/my/script.js
```

If you don't want to repeat `/path/to/node` (or your runtime) over and over, you will add a `PATH` variable to go with the other variables.

But what happens if you want to use something like [nvm](https://github.com/creationix/nvm) or [rvm](https://rvm.io/) or [virtualenv](https://pypi.python.org/pypi/virtualenv), etc? It is not uncommon to have the above change to something like the following

```
*/10 * * * * /path/to/my/launcher.sh
*/10 * * * * /path/to/my/launcher_another.sh
```

Now you have several shell scripts which invoke the required commands to setup the environment and then run whatever program.

## enter SHELL

There is a little known special env variable for cron: `SHELL`. Most people know this variable can be used to change the shell your scripts run run (i.e. `SHELL=/bin/bash`), but it can actually run any file!

So lets say I use nvm and want to setup my environment. Instead of making custom launchers for each command, I can simply do the following:

```
SHELL=/path/to/setup/cron.bash

*/10 * * * * node $HOME/foo.js
```

Now lets look at what `cron.bash` might look like

<script src="https://gist.github.com/5112421.js?file=cron.bash"></script>

For the most part it looks just like any other shell script. The important magical parts are the last 4 lines. These lines put back the SHELL variable to `/bin/bash` and then execute a bash shell to run the cronline command (the stuff for the specific cronjob).

Now our cron files have a consistent environment setup and we can simply run whatever commands we need without further PATH tricks or nonsense.

<script src="https://gist.github.com/5112421.js?file=cron"></script>

Go forth and update your dirty crontabs!

