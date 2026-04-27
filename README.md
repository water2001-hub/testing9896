If you have run into window resizing issues with Kali on xfce under QEMU/KVM you are not alone. By creating two files on your system you are fix this in under 3 minutes!

Like, Subscribe, Click the Notification bell, and leave comments. 

Credit to dannyda: 

https://dannyda.com/2020/10/22/how-to...

Commands:

sudo nano /etc/udev/rules.d/50-x-resize.rules

ACTION=="change",KERNEL=="card0", SUBSYSTEM=="drm", RUN+="/usr/local/bin/x-resize"

Ctrl+x, Y, Ctrl+o

--------------------
sudo nano /usr/local/bin/x-resize

#!/bin/bash
Bash required
Should be run as root and saved to /usr/local/bin/x-resize
Requies udev rule: /etc/udev/rules.d/50-x-resize.rules
udev rule content: ACTION=="change",KERNEL=="card0", SUBSYSTEM=="drm", RUN+="/usr/local/bin/x-resize" 
Make sure auto-resize is enabled in virt-viewer/spicy
Credit for Finding Sessions as Root: 
https://unix.stackexchange.com/questi...
Credit for Resizing via udev: 
https://superuser.com/questions/11838...
Ensure Log Directory Exists
LOG_DIR=/var/log/autores;
if [ ! -d $LOG_DIR ]; then
mkdir $LOG_DIR;
fi
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

Ctrl+x, Y, Ctrl+o

Now make the script executable:

sudo chmod +x /usr/local/bin/x-resize

--------
Again, credit to https://dannyda.com/ for this helpful information.
