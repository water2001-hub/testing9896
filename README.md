#!/bin/bash

#sudo vi /etc/udev/rules.d/50-x-resize.rules
#ACTION=="change",KERNEL=="card0", SUBSYSTEM=="drm", RUN+="/usr/local/bin/x-resize"

#sudo vi /usr/local/bin/x-resize
$sudo chmod +x /usr/local/bin/x-resize

LOG_FILE=${LOG_DIR}/autores.log
Function to find User Sessions & Resize their display
function x_resize() {
declare -A disps usrs
usrs=()
disps=()
for i in $(users);do
[[ $i = root ]] && continue # skip root
usrs[$i]=1
done
for u in "${!usrs[@]}"; do
for i in $(sudo ps e -u "$u" | sed -rn 's/.* DISPLAY=(:[0-9]*).*/\1/p');do
disps[$i]=$u
done
done
for d in "${!disps[@]}";do
session_user="${disps[$d]}"
session_display="$d"
session_output=$(sudo -u "$session_user" PATH=/usr/bin DISPLAY="$session_display" xrandr | awk '/ connected/{print $1; exit; }')
echo "Session User: $session_user" | tee -a $LOG_FILE;
echo "Session Display: $session_display" | tee -a $LOG_FILE;
echo "Session Output: $session_output" | tee -a $LOG_FILE;
sudo -u "$session_user" PATH=/usr/bin DISPLAY="$session_display" xrandr --output "$session_output" --auto | tee -a $LOG_FILE;
done
}
echo "Resize Event: $(date)" | tee -a $LOG_FILE
x_resize
