---
layout: mypost
title: regularly upload server information
categories: [linux, shell, ubuntu]
---

# regularly upload server information to application system

### application server -> ubuntu20.04.6 LTS

### Real-time monitoring of network traffic

```shell
# vi /etc/rc.d/traffic_monitor.sh

----------------------------------------------

#!/bin/bash

PATH=/bin:/usr/bin:/sbin:/usr/sbin:/usr/local/bin:/usr/local/sbin;

export PATH

function traffic_monitor {
  # System version
  OS_NAME=$(sed -n '1p' /etc/issue)
  # Net name
  eth=$1
  #Judge whether the network card exists or not. Exit if it does not exist.
  if [ ! -d /sys/class/net/$eth ];then
      echo -e "Network-Interface Not Found"
      echo -e "You system have network-interface:\n`ls /sys/class/net`"
      exit 5
  fi
  while [ "1" ]
  do
    # status
    STATUS="fine"
    # Get the traffic received and sent by the network port at the current time
    RXpre=$(cat /proc/net/dev | grep $eth | tr : " " | awk '{print $2}')
    TXpre=$(cat /proc/net/dev | grep $eth | tr : " " | awk '{print $10}')
    # Get the traffic received and sent by the network port after 1 second
    sleep 1
    RXnext=$(cat /proc/net/dev | grep $eth | tr : " " | awk '{print $2}')
    TXnext=$(cat /proc/net/dev | grep $eth | tr : " " | awk '{print $10}')
    clear
    # Calculate the actual inbound and outbound traffic for this 1 second
    RX=$((${RXnext}-${RXpre}))
    TX=$((${TXnext}-${TXpre}))
    # If the received traffic is greater than the MB order of magnitude, 
    # the MB unit will be displayed, otherwise the KB order of magnitude will be displayed.
    if [[ $RX -lt 1024 ]];then
      RX="${RX}B/s"
    elif [[ $RX -gt 1048576 ]];then
      RX=$(echo $RX | awk '{print $1/1048576 "MB/s"}')
      $STATUS="busy"
    else
      RX=$(echo $RX | awk '{print $1/1024 "KB/s"}')
    fi
    # It is judged that if the sending traffic is greater than the MB order of magnitude, 
    # the MB unit is displayed, otherwise the KB order of magnitude is displayed.
    if [[ $TX -lt 1024 ]];then
      TX="${TX}B/s"
      elif [[ $TX -gt 1048576 ]];then
      TX=$(echo $TX | awk '{print $1/1048576 "MB/s"}')
    else
      TX=$(echo $TX | awk '{print $1/1024 "KB/s"}')
    fi
    # Print information
    echo -e "==================================="
    echo -e "Welcome to Traffic_Monitor stage"
    echo -e "version 1.0"
    echo -e "Since 2014.2.26"
    echo -e "Created by showerlee"
    echo -e "BLOG: http://www.showerlee.com"
    echo -e "==================================="
    echo -e "System: $OS_NAME"
    echo -e "Date:   `date +%F`"
    echo -e "Time:   `date +%k:%M:%S`"
    echo -e "Port:   $1"
    echo -e "Status: $STATUS"
    echo -e  " \t     RX \tTX"
    echo "------------------------------"
    # Print real-time traffic
    echo -e "$eth \t $RX   $TX "
    echo "------------------------------"
    # Exit information
    echo -e "Press 'Ctrl+C' to exit"
  done
}
# Judge execution parameters
if [[ -n "$1" ]];then
  # Execution function
  traffic_monitor $1
else
  echo -e "None parameter,please add system netport after run the script! \nExample: 'sh traffic_monitor eth0'"
fi
----------------------------------------------
# sh traffic_monitor.sh eth0
```

### Get IP

```sh
#!/bin/bash
# Get the IP address of the local application server to monitor
IP=`ifconfig -a|grep inet|grep -v 127.0.0.1|grep -v inet6|awk '{print $2}'|tr -d "addr:"`
echo "IP address："$IP
```

### Get CPU occupancy

```sh
#!/bin/bash
#
#Script function description：Obtain and calculate CPU utilization according to "/proc/stat" file
#
#Formula for calculation CPU time: CPU_TIME=user+system+nice+idle+iowait+irq+softirq
#Formula for calculating CPU utilization: cpu_usage=(idle2-idle1)/(cpu2-cpu1)*100
#Default interval
TIME_INTERVAL=10

LAST_CPU_INFO=$(cat /proc/stat | grep -w cpu | awk '{print $2,$3,$4,$5,$6,$7,$8}')
LAST_SYS_IDLE=$(echo $LAST_CPU_INFO | awk '{print $4}')
LAST_TOTAL_CPU_T=$(echo $LAST_CPU_INFO | awk '{print $1+$2+$3+$4+$5+$6+$7}')
sleep ${TIME_INTERVAL}
NEXT_CPU_INFO=$(cat /proc/stat | grep -w cpu | awk '{print $2,$3,$4,$5,$6,$7,$8}')
NEXT_SYS_IDLE=$(echo $NEXT_CPU_INFO | awk '{print $4}')
NEXT_TOTAL_CPU_T=$(echo $NEXT_CPU_INFO | awk '{print $1+$2+$3+$4+$5+$6+$7}')
 
#System idle time
SYSTEM_IDLE=`echo ${NEXT_SYS_IDLE} ${LAST_SYS_IDLE} | awk '{print $1-$2}'`
#CPU total time
TOTAL_TIME=`echo ${NEXT_TOTAL_CPU_T} ${LAST_TOTAL_CPU_T} | awk '{print $1-$2}'`
CPU_USAGE=`echo ${SYSTEM_IDLE} ${TOTAL_TIME} | awk '{printf "%.2f", 100-$1/$2*100}'`
 
echo "CPU Usage:${CPU_USAGE}%"
```

### Get memory occupancy

```sh
#!/bin/bash
mem_total=$(awk '$1~/^(MemTotal)/{print $2}' /proc/meminfo)
mem_available=$(awk '$1~/^(MemFree)/{print $2}' /proc/meminfo)
#t=`echo ${mem_total} ${mem_available} | awk '{print $1-$2}'`
s=`echo ${mem_available} ${mem_total} | awk '{print $1/$2}'`
mem_remaining=`echo ${s} | awk '{print $1*100}'`
echo memory usage：${mem_remaining}%
```

### Get disk occupancy

```sh
#!/bin/bash
file_dir="/tmp/df_file.tmp"
res_file="/tmp/df_file.res"
#The calculation only takes the second and third columns, that is, total and used
/bin/df -k |awk 'NR>1{print $2,$3}' > ${file_dir}
#Calculate the amount of disk total
df_total=`cat ${file_dir}|awk -v t=0 '{t+=$1}END{print t}'`
#Calculate the total disk usage
df_used=`cat ${file_dir}|awk -v u=0 '{u+=$2}END{print u}'`
#Calculate the total utilization rate
used_total=`echo ${df_used} ${df_total} | awk '{printf "%.2f", $1/$2*100}'`

echo ${used_total}
```

### Get the main details script

Background operation：`nohup ./information.sh > myout.file 2>&1 &`

```sh
#!/bin/bash

PATH=/bin:/usr/bin:/sbin:/usr/sbin:/usr/local/bin:/usr/local/sbin;

export PATH

interval_time=20

function traffic_monitor {

  # System version
  OS_NAME=$(sed -n '1p' /etc/issue)
  # Net name
  eth=$1
  #Judge whether the network card exists or not. Exit if it does not exist.
  if [ ! -d /sys/class/net/$eth ];then
      echo -e "Network-Interface Not Found"
      echo -e "You system have network-interface:\n`ls /sys/class/net`"
      exit 5
  fi
  while [ "1" ]
  do
    # status
    STATUS="fine"
    # Get the traffic received and sent by the network port at the current time
    RXpre=$(cat /proc/net/dev | grep $eth | tr : " " | awk '{print $2}')
    TXpre=$(cat /proc/net/dev | grep $eth | tr : " " | awk '{print $10}')
    # Get the traffic received and sent by the network port after 1 second
    sleep 1
    RXnext=$(cat /proc/net/dev | grep $eth | tr : " " | awk '{print $2}')
    TXnext=$(cat /proc/net/dev | grep $eth | tr : " " | awk '{print $10}')
    clear
    # Get the actual inbound and outbound traffic for this 1 second
    RX=$((${RXnext}-${RXpre}))
    TX=$((${TXnext}-${TXpre}))
	RX=$(echo $RX | awk '{print "%.2f", $1/1024}')
    TX=$(echo $TX | awk '{print "%.2f", $1/1024}')
    # Print real-time traffic
    #echo -e "$RX   $TX "
	
	#-------------memory----------------
	mem_total=$(awk '$1~/^(MemTotal)/{print $2}' /proc/meminfo)
	mem_available=$(awk '$1~/^(MemFree)/{print $2}' /proc/meminfo)
	#t=`echo ${mem_total} ${mem_available} | awk '{print $1-$2}'`
	s=`echo ${mem_available} ${mem_total} | awk '{print $1/$2}'`
	mem_remaining=`echo ${s} | awk '{print "%.2f", $1*100}'`
	#echo memory usage：${mem_remaining}%
	
	#------------CPU-----------------
	TOTAL_TIME=5
	LAST_CPU_INFO=$(cat /proc/stat | grep -w cpu | awk '{print $2,$3,$4,$5,$6,$7,$8}')
	LAST_SYS_IDLE=$(echo $LAST_CPU_INFO | awk '{print $4}')
	LAST_TOTAL_CPU_T=$(echo $LAST_CPU_INFO | awk '{print $1+$2+$3+$4+$5+$6+$7}')
	sleep ${TOTAL_TIME}
	NEXT_CPU_INFO=$(cat /proc/stat | grep -w cpu | awk '{print $2,$3,$4,$5,$6,$7,$8}')
	NEXT_SYS_IDLE=$(echo $NEXT_CPU_INFO | awk '{print $4}')
	NEXT_TOTAL_CPU_T=$(echo $NEXT_CPU_INFO | awk '{print $1+$2+$3+$4+$5+$6+$7}')
	
	#System idle time
	SYSTEM_IDLE=`echo ${NEXT_SYS_IDLE} ${LAST_SYS_IDLE} | awk '{print $1-$2}'`
	#CPU total time
	TOTAL_TIME=`echo ${NEXT_TOTAL_CPU_T} ${LAST_TOTAL_CPU_T} | awk '{print $1-$2}'`
	CPU_USAGE=`echo ${SYSTEM_IDLE} ${TOTAL_TIME} | awk '{printf "%.2f", 100-$1/$2*100}'`
	
	#echo "CPU Usage:${CPU_USAGE}%"
	
	#--------------disk-------------
	file_dir="/tmp/df_file.tmp"
	res_file="/tmp/df_file.res"
	#The calculation only takes the second and third columns, that is, total and used
	/bin/df -k |awk 'NR>1{print $2,$3}' > ${file_dir}
	#Calculate the amount of disk total
	df_total=`cat ${file_dir}|awk -v t=0 '{t+=$1}END{print t}'`
	#Calculate the total disk usage
	df_used=`cat ${file_dir}|awk -v u=0 '{u+=$2}END{print u}'`
	#Calculate the total utilization rate
	used_total=`echo ${df_used} ${df_total} | awk '{printf "%.2f", $1/$2*100}'`
	#echo $used_total
	
	#--------------IP--------------
	IP=`ifconfig -a|grep inet|grep -v 127.0.0.1|grep -v inet6|awk '{print $2}'|tr -d "addr:"`
	#echo "IP address："$IP
	
	HOSTNAME="application_server_0"
	
	#--------------request------------
	curl -d "hostname=${HOSTNAME}&ip=${IP}&cpu=${CPU_USAGE}&memory=${mem_remaining}&disk=${used_total}&netFlowOut=${TX}&netFlowIn=${RX}" http://127.0.0.1:7008/gateway/api/v1.0/sys/cluster
	
	#Request interval
	sleep $interval_time
	
  done
}
# Judge execution parameters

#if [[ -n "$1" ]];then

  # Execution function

  traffic_monitor eth0

#else

  #echo -e "None parameter,please add system netport after run the script! \nExample: 'sh traffic_monitor eth0'"

  # fi
```

