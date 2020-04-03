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

```bash
#!/bin/bash

notify() {
  [ -n "$3" ] && SUBJECT="$3" || SUBJECT="Cron Error"
  echo -e "$1" # | mailx -s "$SUBJECT" "$2" # unix2dos/dos2unix
  exit
}

DBA1="admin1@hostname.localdomain"
# DBA2="admin2@hostname.localdomain"
# DBA3="admin3@hostname.localdomain"
# DBA4="admin4@hostname.localdomain"

MAILS="$DBA1,$DBA2,$DBA3,$DBA4"

if [[ "$#" != 2 ]] ; then
  notify "Cron Error -> '$0' (1)" "$MAILS" "Directory $1 not exist.." && exit
fi

LOG_DIR="`readlink -f $1`"
LOG_FILE="`echo "$2" | sed "s/\///g"`"

if [ ! -d "$LOG_DIR" ] ; then
  notify "Cron Error '$0' (2)" "$MAILS" "Directory '$LOG_DIR' not exist.."
  exit
fi

if [ -d "$LOG_DIR/$LOG_FILE" ] ; then
  notify "Cron Error '$0' (3)" "$MAILS" "Path '$LOG_DIR/$LOG_FILE' is directory.."
  exit
fi

FILENAME="`basename "$0"`"
FILE_PATH="`readlink -f "$0"`"
DIR_PATH="`dirname "$FILE_PATH"`"

FILT="$DIR_PATH/.$FILENAME.filtered"
COMP="$DIR_PATH/.$FILENAME.compared"

RES="Could't create files:\n - $FILT\n -or\n - $COMP"

touch "$FILT" "$COMP" 2>/dev/null

[ "$?" != "0" ] && notify "$RES" "$MAILS"

KW1="err|crit|fail|warn|alert|emerg|denied|deny"
KW2="unread|unreach|miss|problem|block|terminat"
KW3="reject|inject|eject|remove|purge|clean|clear"
KW4="password check failed|authentication failure"

KEYWORDS="$KW1|$KW2|$KW3|$KW4"

find "$LOG_DIR" -mindepth 1 -maxdepth 1 -type f -name "$LOG_FILE" \
        -exec grep -aE "$KEYWORDS" {} 2>/dev/null \; >> "$FILT"

RES="`diff "$FILT" "$COMP"`"

cat "$FILT" > "$COMP"

rm -f "$FILT"

SUBJ="Found Keywords in '$LOG_DIR/$LOG_FILE'"

[ -n "$RES" ] && notify "$RES" "$MAILS" "$SUBJ"
```
