# Log parser via crontab (with notification by mail)

## How works
 * **initialization step** - first execution will grab all errors, probably huge ammount of text and send notification
 * **progress step** - every next execution will grab just new errors
 * when u delete `.compared` file, initialization will start again
 * `.filtered` file is temporary file, removed every end of script

## Requirements
 * `mailx` or `sendmail` client
 * configured service `sendmail` or `postfix`
 * used commands: 
   * `diff`
   * `touch`, `cat`, `tail`, `grep`, `rm`, `echo`, `exit`
   * `test`, `basename`, `dirname`, `readlink`

## Tested on
 - Red Hat Enterprise Linux 6 / 7 / 8
 - CentOS 6 / 7 / 8
 - Oracle Linux 6 / 7 / 8
 - Debian 8 / 9 / 10
 - Ubuntu 14 / 16 / 18
  
## Script

```bash
#!/bin/bash

DBA1="admin1@hostname.localdomain"
DBA2="admin2@hostname.localdomain"
# DBA3="admin3@hostname.localdomain"
# DBA4="admin4@hostname.localdomain"

EMAILS="$DBA1,$DBA2,$DBA3,$DBA4"

notify() {
  [ -n "$3" ] && SUBJECT="$3" || SUBJECT="Crontab Script Error"
  echo -e "$1" # | mailx -s "$SUBJECT" "$2" # mailx or sendmail
  exit
}

SCRIPT_NAME="`basename "$0"`"
SCRIPT_PATH="`readlink -f "$0"`"
SCRIPT_DIR="`dirname "$SCRIPT_PATH"`"

FILT="$SCRIPT_DIR/.$SCRIPT_NAME.filtered"
COMP="$SCRIPT_DIR/.$SCRIPT_NAME.compared"

touch "$FILT" "$COMP" 2>/dev/null

KEYWORDS1="err|crit|fail|warn|alert|emerg|denied|deny"
KEYWORDS2="unread|unreachable|missing|problem|block" # reject

KEYWORDS="$KEYWORDS1|$KEYWORDS2"

#### BEGIN ######################
VARLOG_DIR="/var/log"

VARLOG_KW_AUTH="password check failed|authentication failure|$KEYWORDS"
VARLOG_KW_DPKG="upgrade|install|purge|remove|clean|$KEYWORDS"

tail -25000 "$VARLOG_DIR/kern.log" 2>/dev/null | grep -iE "$KEYWORDS" >> "$FILT"
tail -25000 "$VARLOG_DIR/boot.log" 2>/dev/null | grep -iE "$KEYWORDS" >> "$FILT"
#### END ######################

RES="`diff "$FILT" "$COMP"`"

cat "$FILT" > "$COMP"

rm -f "$FILT"

[ -n "$RES" ] && notify "$RES" "$EMAILS" "Errors Inside '/var/log/kern.log,boot.log'"
```
