# Log parser via crontab



```bash
#!/bin/bash

error_handler() {
echo -e "$1"
exit
}

SCRIPT_NAME=$0
SCRIPT_PATH=`readlink -f $0`
SCRIPT_DIR=`dirname $SCRIPT_PATH`

[ ! -d "$SCRIPT_DIR" ] && error_handler "dir '$SCRIPT_DIR' not exists.."

FILT=$SCRIPT_DIR/.$SCRIPT_NAME.filtered
COMP=$SCRIPT_DIR/.$SCRIPT_NAME.compared

[ -d "$FILT" ] && error_handler "'$FILT' cannot be dir.."
[ -d "$COMP" ] && error_handler "'$COMP' cannot be dir.."

touch $FILT $COMP 2>/dev/null

[ "$?" != "0" ] && error_handler "permission denied?\n - $FILT\n - $COMP"

KEYWORDS_1="err|crit|fail|warn|alert|emerg|denied|deny"
KEYWORDS_2="unread|unreachable|reject|missing|problem"

DEFAULT_KEYWORDS="$KEYWORDS_1|$KEYWORDS_2"

# BEGIN ######################

VARLOG_DIR=/var/log

[ ! -d "$VARLOG_DIR" ] && error_handler "dir '$VARLOG_DIR' not exists.."

VARLOG_KW_KERN="$DEFAULT_KEYWORDS"
VARLOG_KW_BOOT="$DEFAULT_KEYWORDS"
VARLOG_KW_AUTH="password check failed|authentication failure|$DEFAULT_KEYWORDS"
VARLOG_KW_DPKG="upgrade|install|purge|remove|$DEFAULT_KEYWORDS"

tail -25000 $VARLOG_DIR/kern.log 2>/dev/null | grep -iE "$VARLOG_KW_KERN" >> $FILT
tail -25000 $VARLOG_DIR/boot.log 2>/dev/null | grep -iE "$VARLOG_KW_BOOT" >> $FILT
tail -25000 $VARLOG_DIR/auth.log 2>/dev/null | grep -iE "$VARLOG_KW_AUTH" >> $FILT
tail -25000 $VARLOG_DIR/dpkg.log 2>/dev/null | grep -iE "$VARLOG_KW_DPKG" >> $FILT

# END ######################

RES=`diff $FILT $COMP`

cat $FILT > $COMP

rm -f $FILT

[ -n "$RES" ] && echo -e "$RES"
```
