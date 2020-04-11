```bash
#!/bin/bash

echo -e "\r\n-----"
echo -e "`date +"%Y-%m-%d %H:%M:%S"` $0 $@"

notify() {
  TMPFILE=`mktemp /tmp/XXXXX.txt`
  echo -e "$1" > $TMPFILE
  [ -n "$3" ] && SUBJECT="$3" || SUBJECT="Cron Error"
  echo "" | mailx -s "$SUBJECT" -a $TMPFILE "$2"
  rm -f "$TMPFILE"
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
  RES="'$LOG_DIR' must exist as dir\r\n"
  notify "$RES" "$MAILS"
fi

if [ -d "$LOG_DIR/$LOG_FILE" ] ; then
  RES="'$LOG_DIR/$LOG_FILE' cannot be dir\r\n"
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

KW1="err|crit|fail|warn|alert|emerg|denied|deny"
KW2="unread|unreach|miss|problem|block|terminat|exclude"
KW3="reject|inject|eject|remove|purge|clean|clear|close"
KW4="password check failed|authentication failure"

KEYWORDS="$KW1|$KW2|$KW3|$KW4"

FL1="dupablada"

FILTERS="$FL1"

find "$LOG_DIR" -mindepth 1 -maxdepth 1 -type f -name "$LOG_FILE" -print0 2>&1 \
        | xargs -0 grep -aiE "$KEYWORDS" {} 2>&1 \
        | grep -iEav "$FILTERS" >> "$FILT" 2>&1

RES="`diff "$FILT" "$COMP"`"

cat "$FILT" > "$COMP"

rm -f "$FILT"

SUBJ="Found Keywords! ($LOG_FILE)"

if [[ "`echo $RES | wc -c`" -gt "$MAX_SIZE" ]] ; then
  RES="Result too large ($COMP)\r\n"
fi

[ -n "$RES" ] && notify "$RES" "$MAILS" "$SUBJ"
```

# Example install

## /root/crontab.d/log-monitor.cron
```bash
LOG_MONITOR_PATH="/root/crontab.d/log-monitor"

*/5 * * * * /bin/bash $LOG_MONITOR_PATH/generic.sh /var/log messages >> $LOG_MONITOR_PATH/generic.log 2>&1
*/5 * * * * /bin/bash $LOG_MONITOR_PATH/generic.sh /var/log secure >> $LOG_MONITOR_PATH/generic.log 2>&1
*/5 * * * * /bin/bash $LOG_MONITOR_PATH/generic.sh /var/log dmesg >> $LOG_MONITOR_PATH/generic.log 2>&1
*/5 * * * * /bin/bash $LOG_MONITOR_PATH/generic.sh /var/log cron >> $LOG_MONITOR_PATH/generic.log 2>&1
*/5 * * * * /bin/bash $LOG_MONITOR_PATH/generic.sh /var/log syslog >> $LOG_MONITOR_PATH/generic.log 2>&1
*/5 * * * * /bin/bash $LOG_MONITOR_PATH/generic.sh /var/log maillog >> $LOG_MONITOR_PATH/generic.log 2>&1
*/5 * * * * /bin/bash $LOG_MONITOR_PATH/generic.sh /var/log grubby >> $LOG_MONITOR_PATH/generic.log 2>&1
*/5 * * * * /bin/bash $LOG_MONITOR_PATH/generic.sh /var/log firewalld >> $LOG_MONITOR_PATH/generic.log 2>&1
 
*/5 * * * * /bin/bash $LOG_MONITOR_PATH/generic.sh /var/log yum.log >> $LOG_MONITOR_PATH/generic.log 2>&1
*/5 * * * * /bin/bash $LOG_MONITOR_PATH/generic.sh /var/log kern.log >> $LOG_MONITOR_PATH/generic.log 2>&1
*/5 * * * * /bin/bash $LOG_MONITOR_PATH/generic.sh /var/log daemon.log >> $LOG_MONITOR_PATH/generic.log 2>&1
*/5 * * * * /bin/bash $LOG_MONITOR_PATH/generic.sh /var/log boot.log >> $LOG_MONITOR_PATH/generic.log 2>&1

*/5 * * * * /bin/bash $LOG_MONITOR_PATH/httpd.sh /var/log/httpd app_access.log* >> $LOG_MONITOR_PATH/generic.log 2>&1
```
