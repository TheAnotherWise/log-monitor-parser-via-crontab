# Log parser via crontab



```bash
#!/bin/bash

int_error() {
echo $1
exit
}

SCRIPT_NAME=$0
SCRIPT_PATH=`readlink -f $0`
SCRIPT_DIR=`dirname $SCRIPT_PATH`

[ ! -d "$SCRIPT_DIR" ] && int_error "Script dir '$SCRIPT_DIR' not exists.."

VARLOG_DIR=/var/log

[ ! -d "$VARLOG_DIR" ] && int_error "Log dir '$VARLOG_DIR' not exists.."

FILT=$SCRIPT_DIR/.$SCRIPT_NAME.filtered
COMP=$SCRIPT_DIR/.$SCRIPT_NAME.compared

[ -d "$FILT" ] && int_error "'$FILT' cannot be directory.."
[ -d "$COMP" ] && int_error "'$COMP' cannot be directory.."

touch $FILT

[ "$?" != "0" ] && int_error "Cannot create file '$FILT'"

touch $COMP

[ "$?" != "0" ] && int_error "Cannot create file '$COMP'"

VARLOG_KW_KERN="error|critical|failed|problem"
VARLOG_KW_BOOT="error|critical|failed|problem"
VARLOG_KW_AUTH="password check failed|authentication failure"
VARLOG_KW_DPKG="upgrade|install|purge|remove"

tail -25000 $VARLOG_DIR/kern.log 2>/dev/null | grep -iE "$VARLOG_KW_KERN" >> $FILT
tail -25000 $VARLOG_DIR/boot.log 2>/dev/null | grep -iE "$VARLOG_KW_BOOT" >> $FILT
tail -25000 $VARLOG_DIR/auth.log 2>/dev/null | grep -iE "$VARLOG_KW_AUTH" >> $FILT
tail -25000 $VARLOG_DIR/dpkg.log 2>/dev/null | grep -iE "$VARLOG_KW_DPKG" >> $FILT

RES=`diff $FILT $COMP`

cat $FILT > $COMP

rm -f $FILT

[ -n "$RES" ] && echo -e "$RES"
```
