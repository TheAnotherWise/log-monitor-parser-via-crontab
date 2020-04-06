t# Log parser via crontab (with notification by mail)

## How works
 * **initialization step** - first execution will grab all keywords, probably huge ammount of text and send notification
 * **progress step** - every next execution will grab just new keywords
 
## Importants
 * `.compared`, `.filtered` is created inside same directory like script as hidden file
 * deletion of `.compared` file will start **initialization** again on next ececution
 * `.filtered` file is temporary file, removed every end of script

## Requirements
 * `mailx` or `sendmail` client
 * configured service `sendmail` or `postfix`
 * `unix2dos`, `dos2unix`

## Tested on
 - Red Hat Enterprise Linux 6 / 7 / 8
 - CentOS 6 / 7 / 8
 - Oracle Linux 6 / 7 / 8
 - Debian 8 / 9 / 10
 - Ubuntu 14 / 16 / 18

# With validation
```bash
#!/bin/bash

notify() {
  [ -n "$3" ] && SUBJECT="$3" || SUBJECT="Cron Error"
  echo -e "$1" # | mailx -s "$SUBJECT" "$2" # unix2dos/dos2unix
  exit
}

MAX_SIZE="10240000"

DBA1="admin1@hostname.localdomain"
# DBA2="admin2@hostname.localdomain"
# DBA3="admin3@hostname.localdomain"
# DBA4="admin4@hostname.localdomain"

MAILS="$DBA1,$DBA2,$DBA3,$DBA4"

if [ "$#" != 2 ] ; then
  RES="/bin/bash $0 abs_dir_path filename_regex\r\n"
  notify "$RES" "$MAILS"
fi

LOG_DIR="`readlink -f $1`"
LOG_FILE="`echo "$2" | sed "s/\///g"`"

if [ ! -d "$LOG_DIR" ] ; then
  RES="'$LOG_DIR' must exist as dir..\r\n"
  notify "$RES" "$MAILS"
fi

if [ -d "$LOG_DIR/$LOG_FILE" ] ; then
  RES="'$LOG_DIR/$LOG_FILE' cannot be dir..\r\n"
  notify "$RES" "$MAILS"
fi

FILENAME="`basename "$0"`"
FILE_PATH="`readlink -f "$0"`"
DIR_PATH="`dirname "$FILE_PATH"`"

FILT="$DIR_PATH/.$FILENAME.$LOG_FILE.filtered"
COMP="$DIR_PATH/.$FILENAME.$LOG_FILE.compared"

if [ -d "$FILT" ] || [ -d "$COMP" ] ; then
  RES="Cannot be dir (one of is):\r\n - '$FILT'\r\n - '$COMP'\r\n"
  notify "$RES" "$MAILS"
fi

touch "$FILT" "$COMP" 2>/dev/null

if [ "$?" != "0" ] ; then
  RES="Could not 'touch' (one of):\r\n - '$FILT'\r\n - '$COMP'\r\n"
  notify "$RES" "$MAILS"
fi

KW1="err|crit|fail|warn|alert|emerg|denied|deny|"
KW2="unread|unreach|miss|problem|block|terminat|exclude"
KW3="reject|inject|eject|remove|purge|clean|clear|close"
KW4="password check failed|authentication failure"

KEYWORDS="$KW1|$KW2|$KW3|$KW4"

find "$LOG_DIR" -mindepth 1 -maxdepth 1 -type f -name "$LOG_FILE" \
        -exec grep -aiE "$KEYWORDS" {} 2>/dev/null \; >> "$FILT"

RES="`diff "$FILT" "$COMP"`"

cat "$FILT" > "$COMP"

rm -f "$FILT"

SUBJ="Found keywords ($LOG_FILE)"

if [[ "`stat -c%s "$COMP"`" -gt "$MAX_SIZE" ]] ; then
  RES="'"$COMP"' too large.."
fi

[ -n "$RES" ] && notify "$RES" "$MAILS" "$SUBJ"
```

# Example install

## /root/crontab.d/log-monitor.cron
```bash
*/5 * * * * /bin/bash /root/crontab.d/log-monitor/generic.sh /var/log yum.log
*/5 * * * * /bin/bash /root/crontab.d/log-monitor/generic.sh /var/log kern.log
*/5 * * * * /bin/bash /root/crontab.d/log-monitor/generic.sh /var/log maillog
*/5 * * * * /bin/bash /root/crontab.d/log-monitor/generic.sh /var/log daemon.log
*/5 * * * * /bin/bash /root/crontab.d/log-monitor/generic.sh /var/log cron*
*/5 * * * * /bin/bash /root/crontab.d/log-monitor/generic.sh /var/log secure*

*/5 * * * * /bin/bash /root/crontab.d/log-monitor/httpd.sh /var/log/httpd app_access.log*
```

### `generic.sh` vs `httpd.sh`
 There is just one difference between this files, diffretent KEYWORDS
 
`generic.sh`
```bash
# ...
 
KW1="err|crit|fail|warn|alert|emerg|denied|deny|"
KW2="unread|unreach|miss|problem|block|terminat|exclude"
KW3="reject|inject|eject|remove|purge|clean|clear|close"
KW4="password check failed|authentication failure"

KEYWORDS="$KW1|$KW2|$KW3|$KW4"
# ...
```
 
`httpd.sh`
```bash
# ...
 
KEYWORDS=" 5[0-9]{2}"

# ... 
