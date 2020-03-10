# Log parser via crontab

```bash
#!/bin/bash

SCRIPT_PATH=`readlink -f $0`
SCRIPT_DIR=`dirname $SCRIPT_PATH`

[ ! -d "$SCRIPT_DIR" ] && exit

VARLOG_DIR=/var/log

[ ! -d "$VARLOG_DIR" ] && exit

FILT=$SCRIPT_DIR/.monitor.filtered
COMP=$SCRIPT_DIR/.monitor.compared

touch $FILT $COMP

KEYWORDS="error|critical|warn"

grep -E "$KEYWORDS" $VARLOG_DIR/kern.log >> $FILT 2>/dev/null
grep -E "$KEYWORDS" $VARLOG_DIR/boot.log >> $FILT 2>/dev/null
grep -E "$KEYWORDS" $VARLOG_DIR/auth.log >> $FILT 2>/dev/null
grep -E "$KEYWORDS" $VARLOG_DIR/dpkg.log >> $FILT 2>/dev/null

RES=`diff $FILT $COMP`
cat $FILT > $COMP

rm -f $FILT

[ -n "$RES" ] && echo -ne "$RES"
```
