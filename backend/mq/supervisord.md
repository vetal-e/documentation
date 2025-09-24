# Supervisord

As you read before consumers can normally interrupt the message
procession by many reasons. In the all cases above the interrupted
consumer should be re-run. So you must keep running
`oro:message-queue:consume` command and to do this best we advise you
to delegate this responsibility to
<a href="http://supervisord.org/" target="_blank">Supervisord</a>. With next program
configuration supervisord keeps running four simultaneous instances of
`oro:message-queue:consume` command and cares about relaunch if
instance has dead by any reason.

```ini
[program:oro_message_consumer]
command=/path/to/bin/console --env=prod --no-debug oro:message-queue:consume
process_name=%(program_name)s_%(process_num)02d
numprocs=4
autostart=true
autorestart=true
startsecs=0
user=apache
redirect_stderr=true
```

<!-- Frontend -->
