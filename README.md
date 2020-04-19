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

if [ "$#" != 3 ] ; then
  RES="/bin/bash $0 log_name dir_path filename\r\n"
  notify "$RES" "$MAILS"
fi

LOG_NAME="$1"
LOG_DIR="`readlink -f $2`"
LOG_FILE="`echo "$3" | sed "s/\///g"`"

if [ "$LOG_NAME" =~ ^[a-zA-Z0-9]{3,9}$ ] ; then
  RES="'$LOG_NAME' allowed characters [a-zA-Z0-9]{3,9}\r\n"
  notify "$RES" "$MAILS"
fi

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
    | xargs -0 grep -aiE "$KEYWORDS" 2>&1 \
    | grep -aiEv "$FILTERS" 2>&1 >> "$FILT" 

RES="`diff "$FILT" "$COMP"`"

cat "$FILT" > "$COMP"

rm -f "$FILT"

SUBJ="Found Keywords! ($LOG_NAME: $LOG_FILE)"

if [[ "`echo $RES | wc -c`" -gt "$MAX_SIZE" ]] ; then
  RES="Result too large ($COMP)\r\n"
fi

[ -n "$RES" ] && notify "$RES" "$MAILS" "$SUBJ"
```

#### `/root/crontab.d/log-monitor.cron`
```bash
LOG_MONITOR_PATH="/root/crontab.d/log-monitor"

# ---

VAR_LOG="/var/log"

*/5 * * * * /bin/bash $LOG_MONITOR_PATH/generic.sh System $VAR_LOG messages >> $LOG_MONITOR_PATH/.generic.log.messages 2>&1
*/5 * * * * /bin/bash $LOG_MONITOR_PATH/generic.sh System $VAR_LOG secure >> $LOG_MONITOR_PATH/.generic.log.secure 2>&1
*/5 * * * * /bin/bash $LOG_MONITOR_PATH/generic.sh System $VAR_LOG dmesg >> $LOG_MONITOR_PATH/.generic.log.dmesg 2>&1
*/5 * * * * /bin/bash $LOG_MONITOR_PATH/generic.sh System $VAR_LOG cron >> $LOG_MONITOR_PATH/.generic.log.cron 2>&1
*/5 * * * * /bin/bash $LOG_MONITOR_PATH/generic.sh System $VAR_LOG syslog >> $LOG_MONITOR_PATH/.generic.log.syslog 2>&1
*/5 * * * * /bin/bash $LOG_MONITOR_PATH/generic.sh System $VAR_LOG maillog >> $LOG_MONITOR_PATH/.generic.log.maillog 2>&1
*/5 * * * * /bin/bash $LOG_MONITOR_PATH/generic.sh System $VAR_LOG grubby >> $LOG_MONITOR_PATH/.generic.log.grubby 2>&1
*/5 * * * * /bin/bash $LOG_MONITOR_PATH/generic.sh System $VAR_LOG firewalld >> $LOG_MONITOR_PATH/.generic.log.firewalld 2>&1
*/5 * * * * /bin/bash $LOG_MONITOR_PATH/generic.sh System $VAR_LOG yum.log >> $LOG_MONITOR_PATH/.generic.log-yum.log 2>&1
*/5 * * * * /bin/bash $LOG_MONITOR_PATH/generic.sh System $VAR_LOG kern.log >> $LOG_MONITOR_PATH/.generic.log-kern.log 2>&1
*/5 * * * * /bin/bash $LOG_MONITOR_PATH/generic.sh System $VAR_LOG daemon.log >> $LOG_MONITOR_PATH/.generic.log-daemon.log 2>&1
*/5 * * * * /bin/bash $LOG_MONITOR_PATH/generic.sh System $VAR_LOG boot.log >> $LOG_MONITOR_PATH/.generic.log-boot.log 2>&1

JBCS_HTTPD_ROOT="/opt/rh/jbcs-httpd24/root/var/log/httpd"

*/5 * * * * /bin/bash $LOG_MONITOR_PATH/httpd.sh Apache $JBCS_HTTPD_ROOT pccp_access.log >> $LOG_MONITOR_PATH/.httpd.log-pccp_access.log 2>&1
```
