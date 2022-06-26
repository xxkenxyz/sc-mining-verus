1. Install OS Linux menggunakan SD Card
2. Flash EEMC STB dengan OS Linux
3. Update OS Linux dan install ccminer dg perintah sbb
apt-get update

apt-get upgrade

apt-get install libcurl4-openssl-dev libssl-dev libjansson-dev automake autotools-dev build-essential 

apt-get install git

git clone --single-branch -b ARM https://github.com/monkins1010/ccminer.git

 cd ccminer 
 chmod +x build.sh
 chmod +x configure.sh  
 chmod +x autogen.sh 
./build.sh

./ccminer -a verus -o stratum+tcp://ap.luckpool.net:3956 -u RWagyqtRs8aXvoMnjsUFqx23RFJUKhpUHX.stb5 -p x -t 8

--------------------------------------------------------------------
Agar autorun ccminernya 
1. Buat file dengan nama bebas contoh nano autorun.sh dan isi
./ccminer/ccminer -a verus -o stratum+tcp://ap.luckpool.net:3960 -u RWagyqtRs8aXvoMnjsUFqx23RFJUKhpUHX.stb5 -p x -t 3

2. Buat file monitoring.sh dengan cara ketik nano monitoring.sh dan isi sbb
#!/bin/bash
HOME_DIR="/root"
MSG_FILE="$HOME_DIR/monitoring.msg"
SYSLOG_FILE="$HOME_DIR/sysinfo.log"
echo -e "Monitoring $(hostname)-Bogor1\n" > $MSG_FILE
function getboardtemp() {
        if [ -f /etc/armbianmonitor/datasources/soctemp ]; then
                read raw_temp </etc/armbianmonitor/datasources/soctemp 2>/dev/null
                if [ ! -z $(echo "$raw_temp" | grep -o "^[1-9][0-9]*\.\?[0-9]*$") ] && (( $(echo "${raw_temp} < 200" |bc -l) )); then
                        # Allwinner legacy kernels output degree C
                        board_temp=${raw_temp}
                else
                        board_temp=$(awk '{printf("%d",$1/1000)}' <<<${raw_temp})
                fi
        elif [ -f /etc/armbianmonitor/datasources/pmictemp ]; then
                # fallback to PMIC temperature
                board_temp=$(awk '{printf("%d",$1/1000)}' </etc/armbianmonitor/datasources/pmictemp)
        fi
        # Some boards, such as the Orange Pi Zero LTS, report shifted CPU temperatures
        board_temp=$((board_temp + CPU_TEMP_OFFSET))
} # getboardtemp

function get_ip_addresses() {
        local ips=()
        for f in /sys/class/net/*; do
                local intf=$(basename $f)
                # match only interface names starting with e (Ethernet), br (bridge), w (wireless), r (some Ralink drivers use ra<number> format)
                if [[ $intf =~ $SHOW_IP_PATTERN ]]; then
                        local tmp=$(ip -4 addr show dev $intf | awk '/inet/ {print $2}' | cut -d'/' -f1)
                        # add both name and IP - can be informative but becomes ugly with long persistent/predictable device names
                        #[[ -n $tmp ]] && ips+=("$intf: $tmp")
                        # add IP only
			[[ -n $tmp ]] && ips+=("$tmp")
                fi
        done
        echo "${ips[@]}"
} # get_ip_addresses

# get uptime, logged in users and load in one take
UPTIME=$(uptime | sed s/^.*up// | awk -F, '{ if ( $3 ~ /user/ ) { print $1 $2 } else { print $1 }}' | sed -e 's/:/\ hours\ /' -e 's/ min//' -e 's$

getboardtemp
ip_address=$(get_ip_addresses &)

echo -e "Uptime : "$UPTIME >> $MSG_FILE
echo -e "CPU Temp : "$board_temp" C" >> $MSG_FILE
echo -e "IP : "$ip_address >> $MSG_FILE
echo -e "Pool address : "$(cat $HOME_DIR/doge-zergpool.log | grep "Starting on" | awk '{print $5}')"" >> $MSG_FILE
echo -e "Hashrate : "$(tail -1 $HOME_DIR/doge-zergpool.log | grep "accept" | awk '{print $7}')" kH/s" >> $MSG_FILE
echo -e "Last share : "$(tail -1 $HOME_DIR/doge-zergpool.log | grep "accept" | awk '{print $4}') >> $MSG_FILE
# Send Notif Telegram
TOKEN=1821035906:AAEhQ0o3VxhEkvAUyWIRa8HlKuO9TKiAnJ4
CHAT_ID=(260720640) #ID_TELEGRAM diisi dengan id yang akan menerima notifnya.
MESSAGE=`cat $MSG_FILE`
URL="https://api.telegram.org/bot$TOKEN/sendMessage"

for ID in "${CHAT_ID[@]}"
do

        curl -s -X POST $URL -d chat_id=$ID -d text="$MESSAGE";

done

3. buat crontab -e dengan isi sbb
@reboot bash /root/autorun.sh > /root/autorun.log 2>&1
*/5 * * * * bash /root/monitoring.sh > /root/monitoring.log 2>&1
0 */4 * * * /sbin/shutdown -r now

4. reboot stb nya

