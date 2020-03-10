# Log parser via crontab

```bash
#!/bin/bash

SCRIPT_PATH=`readlink -f $0`
SCRIPT_DIR=`dirname $SCRIPT_PATH`

[ ! -d "$SCRIPT_DIR" ] && exit

VARLOG_DIR=/var/log

[ ! -d "$VARLOG_DIR" ] && exit

FILT=$SCRIPT_DIR/.example.filtered
COMP=$SCRIPT_DIR/.example.compared

touch $FILT $COMP

VARLOG_KW_KERN="error|critical|failed|problem"
VARLOG_KW_BOOT="error|critical|failed|problem"
VARLOG_KW_AUTH="password check failed|authentication failure"
VARLOG_KW_DPKG="upgrade|install|purge|remove"

tail -25000 $VARLOG_DIR/kern.log | grep -E "$VARLOG_KW_KERN" >> $FILT 2>/dev/null
tail -25000 $VARLOG_DIR/boot.log | grep -E "$VARLOG_KW_BOOT" >> $FILT 2>/dev/null
tail -25000 $VARLOG_DIR/auth.log | grep -E "$VARLOG_KW_AUTH" >> $FILT 2>/dev/null
tail -25000 $VARLOG_DIR/dpkg.log | grep -E "$VARLOG_KW_DPKG" >> $FILT 2>/dev/null

RES=`diff $FILT $COMP`
cat $FILT > $COMP

rm -f $FILT

[ -n "$RES" ] && echo -e "$RES"
```
