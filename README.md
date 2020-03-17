# Log parser via crontab (with notification by mail)

## How works
 * **initialization step** - first execution will grab all keywords, probably huge ammount of text and send notification
 * **progress step** - every next execution will grab just new keywords
 
## Importants
 * `.compared`, `.filtered` is created inside same directory like script as hidden file
 * deletion of `.compared` file will start **initialization** again
 * `.filtered` file is temporary file, removed every end of script
 * use `unix2dos` if want pretty print in *windows*
 
## Requirements
 * `mailx` or `sendmail` client
 * configured service `sendmail` or `postfix`
 * used commands: 
   * `diff`
   * `touch`, `cat`, `tail`, `grep`, `rm`, `echo`, `exit`
   * `test`, `basename`, `dirname`, `readlink`
   * `unix2dos`, `dos2unix`

## Tested on
 - Red Hat Enterprise Linux 6 / 7 / 8
 - CentOS 6 / 7 / 8
 - Oracle Linux 6 / 7 / 8
 - Debian 8 / 9 / 10
 - Ubuntu 14 / 16 / 18
  
#

```bash
#!/bin/bash

DBA1="admin1@hostname.localdomain"
# DBA2="admin2@hostname.localdomain"
# DBA3="admin3@hostname.localdomain"
# DBA4="admin4@hostname.localdomain"

EMAILS="$DBA1,$DBA2,$DBA3,$DBA4"

notify() {
  [ -n "$3" ] && SUBJECT="$3" || SUBJECT="Cron Error"
  echo -e "$1" # | unix2dos | mailx -s "$SUBJECT" "$2"
  exit
}

SCRIPT_NAME="`basename "$0"`"
SCRIPT_PATH="`readlink -f "$0"`"
SCRIPT_DIR="`dirname "$SCRIPT_PATH"`"

FILT="$SCRIPT_DIR/.$SCRIPT_NAME.filtered"
COMP="$SCRIPT_DIR/.$SCRIPT_NAME.compared"

RESULT="Could't create files:\n - $FILT\n -or\n - $COMP"

touch "$FILT" "$COMP" 2>/dev/null

[ "$?" != "0" ] && notify "$RESULT" "$EMAILS"

KEYWORDS1="err|crit|fail|warn|alert|emerg|denied|deny"
KEYWORDS2="unread|unreach|miss|problem|block|terminat"
KEYWORDS3="reject|inject|eject|remove|purge|clean|clear"

KEYWORDS="$KEYWORDS1|$KEYWORDS2|$KEYWORDS3"

#### BEGIN ######################
LOG0_DIR="/var/log"

LOG0_FILE0="auth.log"
LOG0_FILE1="dpkg.log"

LOG0_KW0="$KEYWORDS|password check failed|authentication failure"
LOG0_KW1="$KEYWORDS|install|upgrade"

find "$LOG0_DIR" -mindepth 1 -maxdepth 1 -type f -name "$LOG0_FILE0" \
-exec cat {} \; 2>/dev/null | grep -iE "$LOG0_KW0" >> "$FILT"
find "$LOG0_DIR" -mindepth 1 -maxdepth 1 -type f -name "$LOG0_FILE1" \
-exec cat {} \; 2>/dev/null | grep -iE "$LOG0_KW1" >> "$FILT"
#### END ######################

RES="`diff "$FILT" "$COMP"`"

cat "$FILT" > "$COMP"

rm -f "$FILT"

MAIL_SUBJECT="Found Keywords in '$LOG0_DIR/$LOG0_FILE0,$LOG0_FILE1'"

[ -n "$RES" ] && notify "$RESULT" "$EMAILS" "$MAIL_SUBJECT"
```
