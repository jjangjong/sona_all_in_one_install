# References
https://wiki.onosproject.org/display/ONOS/SONA+User+Guide

https://github.com/sonaproject

# Ubuntu 14.04 to 16.04 Upgrade
Ucloud를 통해 지급되는 Ubuntu 14.04 VM의 경우 Ubuntu Repository 이슈로 Newton 이상의 Openstack 설치가 어려우므로 16.04로 Upgrade한다.

Ubuntu Mirror 변경
```
sed -i '47s/mirror2\.g\.ucloudbiz/security\.ubuntu/g' /etc/apt/sources.list
sed -i '48s/mirror2\.g\.ucloudbiz/security\.ubuntu/g' /etc/apt/sources.list
sed -i '49s/mirror2\.g\.ucloudbiz/security\.ubuntu/g' /etc/apt/sources.list
sed -i '50s/mirror2\.g\.ucloudbiz/security\.ubuntu/g' /etc/apt/sources.list
sed -i '51s/mirror2\.g\.ucloudbiz/security\.ubuntu/g' /etc/apt/sources.list
sed -i '52s/mirror2\.g\.ucloudbiz/security\.ubuntu/g' /etc/apt/sources.list

sed -i 's/mirror2\.g\.ucloudbiz/archive\.ubuntu/g' /etc/apt/sources.list
```

잘 변경되었는지 확인
```
~# cat /etc/apt/sources.list


# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to
# newer versions of the distribution.

deb http://archive.ubuntu.com/ubuntu/ xenial main restricted
deb-src http://archive.ubuntu.com/ubuntu/ xenial main restricted

## Major bug fix updates produced after the final release of the
## distribution.
deb http://archive.ubuntu.com/ubuntu/ xenial-updates main restricted
deb-src http://archive.ubuntu.com/ubuntu/ xenial-updates main restricted

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team. Also, please note that software in universe WILL NOT receive any
## review or updates from the Ubuntu security team.
deb http://archive.ubuntu.com/ubuntu/ xenial universe
deb-src http://archive.ubuntu.com/ubuntu/ xenial universe
deb http://archive.ubuntu.com/ubuntu/ xenial-updates universe
deb-src http://archive.ubuntu.com/ubuntu/ xenial-updates universe

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team, and may not be under a free licence. Please satisfy yourself as to
## your rights to use the software. Also, please note that software in
## multiverse WILL NOT receive any review or updates from the Ubuntu
## security team.
deb http://archive.ubuntu.com/ubuntu/ xenial multiverse
deb-src http://archive.ubuntu.com/ubuntu/ xenial multiverse
deb http://archive.ubuntu.com/ubuntu/ xenial-updates multiverse
deb-src http://archive.ubuntu.com/ubuntu/ xenial-updates multiverse

## Uncomment the following two lines to add software from the 'backports'
## repository.
## N.B. software from this repository may not have been tested as
## extensively as that contained in the main release, although it includes
## newer versions of some applications which may provide useful features.
## Also, please note that software in backports WILL NOT receive any review
## or updates from the Ubuntu security team.
# deb http://archive.ubuntu.com/ubuntu/ lucid-backports main restricted universe multiverse
# deb-src http://archive.ubuntu.com/ubuntu/ lucid-backports main restricted universe multiverse

## Uncomment the following two lines to add software from Canonical's
## 'partner' repository.
## This software is not part of Ubuntu, but is offered by Canonical and the
## respective vendors as a service to Ubuntu users.
# deb http://archive.ubuntu.com/ubuntu lucid partner
# deb-src http://archive.ubuntu.com/ubuntu lucid partner

deb http://security.ubuntu.com/ubuntu xenial-security main restricted
deb-src http://security.ubuntu.com/ubuntu xenial-security main restricted
deb http://security.ubuntu.com/ubuntu xenial-security universe
deb-src http://security.ubuntu.com/ubuntu xenial-security universe
deb http://security.ubuntu.com/ubuntu xenial-security multiverse
deb-src http://security.ubuntu.com/ubuntu xenial-security multiverse

# deb http://archive.ubuntu.com/saltstack/ubuntu xenial main # disabled on upgrade to xenial
```


update&upgrade&dist-upgrade
```
~# apt-get update #Update 시 마지막 몇 Line은 Fail이 날 수 있으니 무시한다
~# apt-get upgrade
~# apt-get dist-upgrade
```

Ubuntu 16.04로 Upgrade
```
~# apt-get install update-manager-core
~# do-release-upgrade
```
설치 중간에 중간에 나오는 질문은 모두 'Yes'한다.
특히 'Restart services during package upgrades without asking?' 질문이 있는데 'Yes' 한다.

Reboot 후 Upgrade된 Ubuntu Version 확인
```
root@lb5004:~# cat /etc/issue
Ubuntu 16.04.4 LTS \n \l
```


# Pre-requisite
Ubuntu 16.04가 설치된 VM

Nameserver 설정
```
~# cat /etc/resolv.conf

# Dynamic resolv.conf(5) file for glibc resolver(3) generated by resolvconf(8)
#     DO NOT EDIT THIS FILE BY HAND -- YOUR CHANGES WILL BE OVERWRITTEN
nameserver 8.8.8.8
```
apt repo update 및 git 설치
```
# apt-get update
# apt-get install -y git
```

#All-in-One SONA 설치
Devstack Queens를 Clone 한다.
```
# cd ~/
# git clone -b stable/queens https://git.openstack.org/openstack-dev/devstack
```

stack user 생성
```
# ~/devstack/tools/create-stack-user.sh
```

devstack 소스를 /opt/stack/ 디렉토리에 복사하고 stack ownership 설정
```
# cp -R ~/devstack/ /opt/stack
# chown -R stack:stack /opt/stack
```

아래 작업부터 stack 유저로 로그인 하여 수행한다.
```
# su - stack
$ cd ~/devstack
```

Devstack 설정을 위한 local.conf를 생성한다. 하기 Sample에서 IP 정보만 변경한다. (sudo ifconfig 명령으로 확인)
```
[[local|localrc]]
HOST_IP=ipaddress
SERVICE_HOST=ipaddress
RABBIT_HOST=ipaddress
DATABASE_HOST=ipaddress
Q_HOST=ipaddress
 
ADMIN_PASSWORD=nova
DATABASE_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD
SERVICE_TOKEN=$ADMIN_PASSWORD
 
DATABASE_TYPE=mysql
 
# Log
USE_SCREEN=True
SCREEN_LOGDIR=/opt/stack/logs/screen
LOGFILE=/opt/stack/logs/xstack.sh.log
LOGDAYS=1
# Images
FORCE_CONFIG_DRIVE=True
 
# Networks
Q_ML2_TENANT_NETWORK_TYPE=vxlan
Q_ML2_PLUGIN_MECHANISM_DRIVERS=onos_ml2
Q_ML2_PLUGIN_TYPE_DRIVERS=flat,vlan,vxlan
ML2_L3_PLUGIN=onos_router
NEUTRON_CREATE_INITIAL_NETWORKS=False
enable_plugin networking-onos https://github.com/openstack/networking-onos.git
ONOS_MODE=allinone
 
# Services
ENABLED_SERVICES=n-cpu,placement-client,neutron,key,nova,n-api,n-cond,n-sch,n-novnc,n-cauth,placement-api,g-api,g-reg,q-svc,horizon,rabbit,mysql
 
 
NOVA_VNC_ENABLED=True
VNCSERVER_PROXYCLIENT_ADDRESS=$HOST_IP
VNCSERVER_LISTEN=$HOST_IP
 
LIBVIRT_TYPE=qemu
 
# Branches
GLANCE_BRANCH=stable/queens
HORIZON_BRANCH=stable/queens
KEYSTONE_BRANCH=stable/queens
NEUTRON_BRANCH=stable/queens
NOVA_BRANCH=stable/queens
```

하기와 같이 sed 명령어를 이용하여 바로 수정 가능하다.

```
# sudo ifconfig
$ ifconfig
eth0      Link encap:Ethernet  HWaddr 02:00:34:15:00:0b
          inet addr:172.27.0.248  Bcast:172.27.255.255  Mask:255.255.0.0
          inet6 addr: fe80::34ff:fe15:b/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:38749 errors:0 dropped:0 overruns:0 frame:0
          TX packets:33149 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:215120687 (215.1 MB)  TX bytes:3375545 (3.3 MB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:417 errors:0 dropped:0 overruns:0 frame:0
          TX packets:417 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:29761 (29.7 KB)  TX bytes:29761 (29.7 KB)

$ sed -i sed -i 's/ipaddress/172\.27\.0\.248/g' local.conf
$ cat local.conf
```


Stack.sh 명령 수행을 통해 OpenStack을 설치한다.
```
$ cd ~/devstack
$ ./stack.sh
```

설치 완료 후 동작 확인을 위해 Dashboard에 접속해본다. (ID/Password: admin/nova)

ONOS 연동 확인을 위해 Network/Subnet을 생성해본다.

 ㅇ Network Name: net1, net2
 
 ㅇ Subnet Name: snet1, snet2
 
 ㅇ CIDR: 20.1.1.0/24, 20.2.2.0/24
 
 ㅇ Gateway: 20.1.1.1, 20.2.2.1
 
ONOS에 접속하여 정상 동작을 확인한다.
```
$ ssh -p 8101 karaf@10.1.1.5
Password authentication
Password: karaf
Welcome to Open Network Operating System (ONOS)!
     ____  _  ______  ____     
    / __ \/ |/ / __ \/ __/   
   / /_/ /    / /_/ /\ \     
   \____/_/|_/\____/___/     
                               
Documentation: wiki.onosproject.org      
Tutorials:     tutorials.onosproject.org 
Mailing lists: lists.onosproject.org     

Come help out! Find out how at: contribute.onosproject.org 

Hit '<tab>' for a list of available commands
and '[cmd] --help' for help on a specific command.
Hit '<ctrl-d>' or type 'system:shutdown' or 'logout' to shutdown ONOS.
onos> openstack-networks 
ID                                      Name                Network Mode        VNI                 Subnets 
90b74fc7-c6e7-4254-830c-ab46ece25f83    net1                VXLAN               96                  [20.1.1.0/24]
84001230-81d4-48dd-9fbd-c7c48ab47814    net2                VXLAN               91                  [20.2.2.0/24]
onos> 
```
 
# Exercise 1: East-west routing
Openstack에서 서로 다른 Network에 속한 VM간 통신이 필요한 경우(=east-west routing), 가상의 router를 생성하여 Network를 연동해야 한다.

Horizon을 통해 Router를 생성한다.
```
onos> openstack-routers 
ID                                      Name                External            Internal
7bfb0e90-faa1-4fbc-8d9a-4167abd66a5d    router              []                  []  
```

Horizon을 통해 생성한 router에 net1, net2를 연결한다. (인터페이스 추가 메뉴를 이용)
```
onos> openstack-routers 
ID                                      Name                External            Internal
7bfb0e90-faa1-4fbc-8d9a-4167abd66a5d    router              []                  [20.1.1.1, 20.2.2.1]

onos> openstack-ports 
ID                                      Network             MAC                 Fixed IPs
d064ef9c-1a29-4b1d-91e9-e5f506f4a212    net2                fa:16:3e:70:46:cc   [20.2.2.1]
1981f54f-86f3-4b56-88e4-6ea0621792a2    net1                fa:16:3e:55:0a:43   [20.1.1.1]
```

net1, net2에 각각 vm을 생성한다.
```
onos> openstack-ports 
ID                                      Network             MAC                 Fixed IPs
be4e9afe-96e4-4c2e-a99d-be055ff1bfa8    net2                fa:16:3e:8c:77:f4   [20.2.2.5]
d064ef9c-1a29-4b1d-91e9-e5f506f4a212    net2                fa:16:3e:70:46:cc   [20.2.2.1]
6134aed6-2a4c-47e1-b00d-6dc5f0dccbe1    net1                fa:16:3e:ba:5e:c3   [20.1.1.5]
1981f54f-86f3-4b56-88e4-6ea0621792a2    net1                fa:16:3e:55:0a:43   [20.1.1.1]
```
 
vm간 ping이 되는 것을 확인한다.
```
Mincheolui-MacBook-Pro:~ mincheolpark$ ssh sdn@172.27.0.3
Welcome to Ubuntu 16.04.3 LTS (GNU/Linux 4.4.0-101-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

99 packages can be updated.
0 updates are security updates.


*** System restart required ***
Last login: Thu Jun 21 07:42:24 2018 from 172.16.101.11
sdn@mcpark-all-in-one:~$ virsh list
 Id    Name                           State
----------------------------------------------------
 3     instance-00000003              running
 4     instance-00000004              running

sdn@mcpark-all-in-one:~$ virsh console 4
Connected to domain instance-00000004
Escape character is ^]

$ ifconfig
eth0      Link encap:Ethernet  HWaddr FA:16:3E:8C:77:F4  
          inet addr:20.2.2.5  Bcast:20.2.2.0  Mask:255.255.255.0
          inet6 addr: fe80::f816:3eff:fe8c:77f4/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1450  Metric:1
          RX packets:96 errors:0 dropped:0 overruns:0 frame:0
          TX packets:516 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:7296 (7.1 KiB)  TX bytes:40846 (39.8 KiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

$ route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         20.2.2.1        0.0.0.0         UG    0      0        0 eth0
20.2.2.0        0.0.0.0         255.255.255.0   U     0      0        0 eth0
$ ping 20.1.1.5
PING 20.1.1.5 (20.1.1.5): 56 data bytes
64 bytes from 20.1.1.5: seq=0 ttl=64 time=6.455 ms
64 bytes from 20.1.1.5: seq=1 ttl=64 time=1.475 ms
64 bytes from 20.1.1.5: seq=2 ttl=64 time=1.387 ms

--- 20.1.1.5 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 1.387/3.105/6.455 ms
$ 
```

router에서 interface 하나를 삭제하고 ping이 안되는 것을 확인한다.

확인 후 이후 Exercise를 위해 다시 interface를 연결해둔다.
 
# Exercise 2: SONA CLI
현재 ONOS CLI에서 사용가능한 SONA 관련 CLI는 하기와 같다.
```
onos> openstack-
openstack-delete-peer-router         openstack-direct-ports
openstack-floatingips                openstack-networks
openstack-node-check                 openstack-node-init
openstack-nodes                      openstack-peer-routers
openstack-ports                      openstack-purge-rules
openstack-purge-states               openstack-routers
openstack-security-groups            openstack-sync-rules
openstack-sync-states                openstack-update-peer-router
openstack-update-peer-router-vlan
```

openstack-nodes: Compute/Gateway/Controller Node에 대한 정보 조회
```
onos> openstack-nodes
Hostname            Type           Integration Bridge      Management IP           Data IP             VLAN Intf           Uplink Port    State
compute-01          COMPUTE        of:00000000000000a1     10.1.1.5                10.1.1.5                                               COMPLETE
controller          CONTROLLER     null                    10.1.1.5                                                                       COMPLETE
Total 2 nodes
```

openstack-node-check: Node별 상태 조회
```
onos> openstack-node-check compute-01
[Integration Bridge Status]
OK br-int=of:00000000000000a1 available=true {managementAddress=10.1.1.5, protocol=OF_14, channelId=10.1.1.5:59440}
OK vxlan portNum=1 enabled=true {adminState=enabled, portName=vxlan, portMac=42:50:00:b0:c4:a5}
```

openstack-node-init: Node 초기화 시 사용
```
onos> openstack-node-init -a
Initializing compute-01
Initializing controller
Done.
```

openstack-networks, openstack-ports, openstack-routers, openstack-security-groups: OpenStack에서 생성한 network/port/router/security-group 정보 조회
```
onos> openstack-networks
ID                                      Name                Network Mode        VNI                 Subnets
90b74fc7-c6e7-4254-830c-ab46ece25f83    net1                VXLAN               96                  [20.1.1.0/24]
84001230-81d4-48dd-9fbd-c7c48ab47814    net2                VXLAN               91                  [20.2.2.0/24]

onos> openstack-ports
ID                                      Network             MAC                 Fixed IPs
be4e9afe-96e4-4c2e-a99d-be055ff1bfa8    net2                fa:16:3e:8c:77:f4   [20.2.2.5]
d064ef9c-1a29-4b1d-91e9-e5f506f4a212    net2                fa:16:3e:70:46:cc   [20.2.2.1]
6134aed6-2a4c-47e1-b00d-6dc5f0dccbe1    net1                fa:16:3e:ba:5e:c3   [20.1.1.5]
fc528e8a-6b4b-4cea-9d20-ebb57327e4a0    net1                fa:16:3e:23:c0:78   [20.1.1.1]

onos> openstack-security-groups
Hint: use --json option to see security group rules as well

ID                                      Name
9216577e-51f9-4aaa-862e-cd0d545ea5c1    default
940b7b99-11a2-4bea-af39-eb7b6520f52a    default

onos> openstack-routers
ID                                      Name                External            Internal
7bfb0e90-faa1-4fbc-8d9a-4167abd66a5d    router              []                  [20.1.1.1, 20.2.2.1]
```

openstack-sync-states: Openstack의 Neutron에서 보유한 정보 기준으로 ONOS의 분산 Store를 초기화
```
onos> openstack-sync-states
Synchronizing OpenStack security groups
ID                                      Name
9216577e-51f9-4aaa-862e-cd0d545ea5c1    default
940b7b99-11a2-4bea-af39-eb7b6520f52a    default

Synchronizing OpenStack networks
ID                                      Name                VNI                 Subnets
84001230-81d4-48dd-9fbd-c7c48ab47814    net2                91                  [dc17bc95-6863-47f2-a5b3-17b2096beee3]
90b74fc7-c6e7-4254-830c-ab46ece25f83    net1                96                  [bdaec5da-0dce-46d6-9326-2d3afa706a9a]

Synchronizing OpenStack subnets
ID                                      Network             CIDR
bdaec5da-0dce-46d6-9326-2d3afa706a9a    net1                20.1.1.0/24
dc17bc95-6863-47f2-a5b3-17b2096beee3    net2                20.2.2.0/24

Synchronizing OpenStack ports
ID                                      Network             MAC                 Fixed IPs
6134aed6-2a4c-47e1-b00d-6dc5f0dccbe1    net1                fa:16:3e:ba:5e:c3   [20.1.1.5]
be4e9afe-96e4-4c2e-a99d-be055ff1bfa8    net2                fa:16:3e:8c:77:f4   [20.2.2.5]
d064ef9c-1a29-4b1d-91e9-e5f506f4a212    net2                fa:16:3e:70:46:cc   [20.2.2.1]
fc528e8a-6b4b-4cea-9d20-ebb57327e4a0    net1                fa:16:3e:23:c0:78   [20.1.1.1]

Synchronizing OpenStack routers
ID                                      Name                External            Internal
7bfb0e90-faa1-4fbc-8d9a-4167abd66a5d    router                                  [20.1.1.1, 20.2.2.1]

Synchronizing OpenStack floating IPs
ID                                      Floating IP         Fixed IP
```

openstack-purge-rules: ONOS에서 설치한 SONA 관련 flow rule을 wipe-out
```
onos> openstack-purge-rules
Successfully purged flow rules installed by OpenStack networking application.
onos> flows
deviceId=of:00000000000000a1, flowRuleCount=0
deviceId=ovsdb:10.1.1.5, flowRuleCount=0
```

openstack-sync-rules: SONA 관련 flow rule 재설치

```
onos> openstack-sync-rules
Successfully requested re-installing flow rules.
onos> flows
deviceId=of:00000000000000a1, flowRuleCount=13
    id=b0000bb4b9281, state=ADDED, bytes=0, packets=0, duration=1, liveType=UNKNOWN, priority=42000, tableId=0, appId=org.onosproject.openstacknetworking, payLoad=null, selector=[ETH_TYPE:ipv4, IP_PROTO:17, UDP_SRC:68, UDP_DST:67], treatment=DefaultTrafficTreatment{immediate=[OUTPUT:CONTROLLER], deferred=[], transition=None, meter=[], cleared=false, StatTrigger=null, metadata=null}
    id=b00000cf08aec, state=ADDED, bytes=0, packets=0, duration=0, liveType=UNKNOWN, priority=40000, tableId=0, appId=org.onosproject.openstacknetworking, payLoad=null, selector=[ETH_TYPE:arp, ARP_OP:2, ARP_TPA:20.2.2.5, ARP_THA:FA:16:3E:8C:77:F4], treatment=DefaultTrafficTreatment{immediate=[OUTPUT:5], deferred=[], transition=None, meter=[], cleared=false, StatTrigger=null, metadata=null}
    id=b00001bbf7df7, state=ADDED, bytes=0, packets=0, duration=0, liveType=UNKNOWN, priority=40000, tableId=0, appId=org.onosproject.openstacknetworking, payLoad=null, selector=[ETH_TYPE:arp, ARP_OP:2, ARP_TPA:20.1.1.5, ARP_THA:FA:16:3E:BA:5E:C3], treatment=DefaultTrafficTreatment{immediate=[OUTPUT:4], deferred=[], transition=None, meter=[], cleared=false, StatTrigger=null, metadata=null}
    id=b0000fbcbc22b, state=ADDED, bytes=0, packets=0, duration=1, liveType=UNKNOWN, priority=40000, tableId=0, appId=org.onosproject.openstacknetworking, payLoad=null, selector=[ETH_TYPE:arp, ARP_OP:1], treatment=DefaultTrafficTreatment{immediate=[OUTPUT:FLOOD], deferred=[], transition=None, meter=[], cleared=false, StatTrigger=null, metadata=null}
    id=b00001461d721, state=ADDED, bytes=0, packets=0, duration=1, liveType=UNKNOWN, priority=0, tableId=0, appId=org.onosproject.openstacknetworking, payLoad=null, selector=[], treatment=DefaultTrafficTreatment{immediate=[], deferred=[], transition=TABLE:10, meter=[], cleared=false, StatTrigger=null, metadata=null}
    id=b00004ac6c2f2, state=ADDED, bytes=0, packets=0, duration=0, liveType=UNKNOWN, priority=30000, tableId=10, appId=org.onosproject.openstacknetworking, payLoad=null, selector=[IN_PORT:4, ETH_TYPE:ipv4], treatment=DefaultTrafficTreatment{immediate=[TUNNEL_ID:60], deferred=[], transition=TABLE:20, meter=[], cleared=false, StatTrigger=null, metadata=null}
    id=b000072e2e83e, state=ADDED, bytes=0, packets=0, duration=0, liveType=UNKNOWN, priority=30000, tableId=10, appId=org.onosproject.openstacknetworking, payLoad=null, selector=[IN_PORT:5, ETH_TYPE:ipv4], treatment=DefaultTrafficTreatment{immediate=[TUNNEL_ID:5b], deferred=[], transition=TABLE:20, meter=[], cleared=false, StatTrigger=null, metadata=null}
    id=b0000e8f1b0d8, state=ADDED, bytes=0, packets=0, duration=1, liveType=UNKNOWN, priority=0, tableId=10, appId=org.onosproject.openstacknetworking, payLoad=null, selector=[], treatment=DefaultTrafficTreatment{immediate=[], deferred=[], transition=TABLE:20, meter=[], cleared=false, StatTrigger=null, metadata=null}
    id=b00007314ee8c, state=ADDED, bytes=0, packets=0, duration=1, liveType=UNKNOWN, priority=0, tableId=20, appId=org.onosproject.openstacknetworking, payLoad=null, selector=[], treatment=DefaultTrafficTreatment{immediate=[], deferred=[], transition=TABLE:30, meter=[], cleared=false, StatTrigger=null, metadata=null}
    id=b00000d2ffa7a, state=ADDED, bytes=0, packets=0, duration=1, liveType=UNKNOWN, priority=30000, tableId=30, appId=org.onosproject.openstacknetworking, payLoad=null, selector=[ETH_DST:FE:00:00:00:00:02], treatment=DefaultTrafficTreatment{immediate=[], deferred=[], transition=TABLE:40, meter=[], cleared=false, StatTrigger=null, metadata=null}
    id=b00007a117617, state=ADDED, bytes=0, packets=0, duration=1, liveType=UNKNOWN, priority=0, tableId=30, appId=org.onosproject.openstacknetworking, payLoad=null, selector=[], treatment=DefaultTrafficTreatment{immediate=[], deferred=[], transition=TABLE:50, meter=[], cleared=false, StatTrigger=null, metadata=null}
    id=b0000204719e5, state=ADDED, bytes=0, packets=0, duration=0, liveType=UNKNOWN, priority=30000, tableId=50, appId=org.onosproject.openstacknetworking, payLoad=null, selector=[ETH_TYPE:ipv4, IPV4_DST:20.1.1.5/32, TUNNEL_ID:60], treatment=DefaultTrafficTreatment{immediate=[ETH_DST:FA:16:3E:BA:5E:C3, OUTPUT:4], deferred=[], transition=None, meter=[], cleared=false, StatTrigger=null, metadata=null}
    id=b0000fcf5d547, state=ADDED, bytes=0, packets=0, duration=0, liveType=UNKNOWN, priority=30000, tableId=50, appId=org.onosproject.openstacknetworking, payLoad=null, selector=[ETH_TYPE:ipv4, IPV4_DST:20.2.2.5/32, TUNNEL_ID:5b], treatment=DefaultTrafficTreatment{immediate=[ETH_DST:FA:16:3E:8C:77:F4, OUTPUT:5], deferred=[], transition=None, meter=[], cleared=false, StatTrigger=null, metadata=null}
deviceId=ovsdb:10.1.1.5, flowRuleCount=0
```

openstack-direct-ports: OpenStack에서 PCI-Passthrough or SR-IOV 용으로 생성한 port 정보 조회

openstack-peer-routers: Gateway Node와 연동 중인 물리 라우터 정보 조회
```
onos> openstack-peer-routers
Router IP           Mac Address         VLAN ID
172.27.0.1          44:4C:A8:A7:EC:C7   None
```

openstack-update-peer-router: 물리 라우터의 Mac or Vlan ID 정보 수동 변경


openstack-delete-peer-router: 물리 라우터 정보 삭제

# Exercise 3: SONA Gateway 구성을 통한 North-South Routing
OpenStack에서 VxLAN Network를 사용하는 VM이 외부 인터넷 망과 통신을 하기 위해서는(=Norsh-South Routing) OpenStack Network Node 구성이 필요하다. Network Node에서 수행하는 Routing은 2가지 모드가 있는데, 1) Source NAT, 2) Floating IP 기반 Routing이 있다.

SONA에서는 Network Node의 역할을 Gateway Node가 수행한다. Gateway Node는 Network Node가 제공하는 기능 외 하기의 Feature를 제공한다.
  1) Scability 보장, 2) Agentless, 3) 순수 OVS 기반 구현으로 Smart NIC, 물리 스위치 등으로 기능 Offload 가능

Gateway Node 생성을 위한 VM 생성 및 OVS 설치
```
sdn@mcpark-all-in-one-gw:~$ sudo ovs-vsctl --version
sudo: unable to resolve host mcpark-all-in-one-gw
ovs-vsctl (Open vSwitch) 2.8.1
DB Schema 7.15.0
sdn@mcpark-all-in-one-gw:~$ ifconfig
ens3      Link encap:Ethernet  HWaddr fa:16:3e:ab:ad:a7  
          inet addr:10.1.1.15  Bcast:10.1.1.255  Mask:255.255.255.0
          inet6 addr: fe80::f816:3eff:feab:ada7/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1450  Metric:1
          RX packets:16974 errors:0 dropped:0 overruns:0 frame:0
          TX packets:14828 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:32646745 (32.6 MB)  TX bytes:1239476 (1.2 MB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:736 errors:0 dropped:0 overruns:0 frame:0
          TX packets:736 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1 
          RX bytes:55296 (55.2 KB)  TX bytes:55296 (55.2 KB)
```

Gateway Node는 Data Plane IP 외 외부망과의 연동을 위한 별도의 Uplink가 필요하다(Network Node도 동일). 본 Exercise에서는 Gateway Node가 VM 기반으로 구성되어 있으므로 물리 라우터를 Docker 기반으로 Emulation하여 제공한다.
```
sdn@mcpark-all-in-one-gw:~$ git clone -b 1.13 https://github.com/sonaproject/sona-setup.git
Cloning into 'sona-setup'...
remote: Counting objects: 245, done.
remote: Total 245 (delta 0), reused 0 (delta 0), pack-reused 245
Receiving objects: 100% (245/245), 40.22 KiB | 0 bytes/s, done.
Resolving deltas: 100% (133/133), done.
Checking connectivity... done.
sdn@mcpark-all-in-one-gw:~$ cd sona-setup/
sdn@mcpark-all-in-one-gw:~/sona-setup$ cat externalRouterConfig.ini 
floatingCidr = "172.27.0.1/24"
externalPeerMac = "fa:00:00:00:00:01"
sdn@mcpark-all-in-one-gw:~/sona-setup$ ./
createExternalRouter.sh  docker-cleanup.sh        .git/                    pipework                 
sdn@mcpark-all-in-one-gw:~/sona-setup$ ./createExternalRouter.sh 
running done
sudo: unable to resolve host mcpark-all-in-one-gw
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS              PORTS               NAMES
dce2661fa7cb        opensona/router-docker   "/runssh.sh /usr/sbi…"   1 second ago        Up 1 second         22/tcp              router

sdn@mcpark-all-in-one-gw:~/sona-setup$ ifconfig
br-int    Link encap:Ethernet  HWaddr 46:c5:7d:83:59:48  
          inet6 addr: fe80::e048:39ff:feaa:2a30/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:9 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:738 (738.0 B)

docker0   Link encap:Ethernet  HWaddr 02:42:db:ba:6d:fe  
          inet addr:172.17.0.1  Bcast:172.17.255.255  Mask:255.255.0.0
          inet6 addr: fe80::42:dbff:feba:6dfe/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:8 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:0 (0.0 B)  TX bytes:648 (648.0 B)

ens3      Link encap:Ethernet  HWaddr fa:16:3e:ab:ad:a7  
          inet addr:10.1.1.15  Bcast:10.1.1.255  Mask:255.255.255.0
          inet6 addr: fe80::f816:3eff:feab:ada7/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1450  Metric:1
          RX packets:32643 errors:0 dropped:0 overruns:0 frame:0
          TX packets:29393 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:73882201 (73.8 MB)  TX bytes:2259324 (2.2 MB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:736 errors:0 dropped:0 overruns:0 frame:0
          TX packets:736 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1 
          RX bytes:55296 (55.2 KB)  TX bytes:55296 (55.2 KB)

router    Link encap:Ethernet  HWaddr 46:c5:7d:83:59:48  
          inet6 addr: fe80::44c5:7dff:fe83:5948/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:16 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:1296 (1.2 KB)

vethf364bfc Link encap:Ethernet  HWaddr a6:74:19:8a:60:c8  
          inet6 addr: fe80::a474:19ff:fe8a:60c8/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:16 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:0 (0.0 B)  TX bytes:1296 (1.2 KB)
```
172.27.0.1/24 대역을 Serving하며 MAC 주소가 "fa:00:00:00:00:01"인 Router(port name: router)가 생성되었음을 알 수 있다. 

Gateway 정보를 SONA에 REST API를 통해 전송한다.
```
{
		"nodes" : [ 
		{
			"hostname" : "controller",
                        "type" : "CONTROLLER",
                        "managementIp" : "10.1.1.5",
			"endPoint" : "10.1.1.5",
                        "authentication" : {
                                "version" : "v3",
                                "port" : 80,
                                "protocol" : "HTTP",
                                "project" : "admin",
                                "username" : "admin",
                                "password" : "nova",
                                "perspective" : "PUBLIC"
                        }
		},
                {
                    "hostname" : "gateway-01",
                    "type" : "GATEWAY",
                    "managementIp" : "10.1.1.15",
                    "dataIp" : "10.1.1.15",
                    "integrationBridge" : "of:00000000000000b1",
                    "uplinkPort" : "router"
                }
       		]
}

$ curl --user onos:rocks -X POST -H "Content-Type: application/json" http://172.27.0.3:8181/onos/openstacknode/configure -d @network-cfg.json.all
```

ONOS CLI에서 Gateway Node 정보 확인
```
onos> openstack-nodes
Hostname            Type           Integration Bridge      Management IP           Data IP             VLAN Intf           Uplink Port    State
compute-01          COMPUTE        of:00000000000000a1     10.1.1.5                10.1.1.5                                               COMPLETE
controller          CONTROLLER     null                    10.1.1.5                                                                       COMPLETE
gateway-01          GATEWAY        of:00000000000000b1     10.1.1.15               10.1.1.15                               router         INIT
Total 3 nodes
onos>
```

Gateway Node 초기화
```
onos> openstack-node-init gateway-01
Initializing gateway-01
Done.
onos> openstack-nodes
Hostname            Type           Integration Bridge      Management IP           Data IP             VLAN Intf           Uplink Port    State
compute-01          COMPUTE        of:00000000000000a1     10.1.1.5                10.1.1.5                                               COMPLETE
controller          CONTROLLER     null                    10.1.1.5                                                                       COMPLETE
gateway-01          GATEWAY        of:00000000000000b1     10.1.1.15               10.1.1.15                               router         COMPLETE
Total 3 nodes

sdn@mcpark_all_gw:~/sona-setup$ sudo ovs-vsctl show
47e03982-37ff-4888-8567-b57c68714858
    Bridge br-int
        Controller "tcp:10.1.1.5:6653"
            is_connected: true
        fail_mode: secure
        Port vxlan
            Interface vxlan
                type: vxlan
                options: {key=flow, remote_ip=flow}
        Port br-int
            Interface br-int
                type: internal
        Port router
            Interface router
    ovs_version: "2.8.1"
```

Horizon을 이용, Docker를 통해 생성한 Network(172.27.0.0/24)를 등록하고 Router에 추가한다.
이 때 Network의 경우 외부 Network=예, Network Type=Flat으로 설정한다. 외부 Network의 경우 '관리자' Tab에서만 생성가능하니 참고한다.
```
onos> openstack-networks 
ID                                      Name                Network Mode        VNI                 Subnets 
0e1fc728-9212-4184-b5e6-5241111a4ab4    exnet               FLAT                null                [172.27.0.0/24]
90b74fc7-c6e7-4254-830c-ab46ece25f83    net1                VXLAN               96                  [20.1.1.0/24]
84001230-81d4-48dd-9fbd-c7c48ab47814    net2                VXLAN               91                  [20.2.2.0/24]

onos> openstack-routers 
ID                                      Name                External            Internal
7bfb0e90-faa1-4fbc-8d9a-4167abd66a5d    router              [172.27.0.2]        [20.1.1.1, 20.2.2.1]
```

vm console에 접속하여 North-South Routing 처리가 가능함을 확인한다.
```
$ virsh list
 Id    Name                           State
----------------------------------------------------
 3     instance-00000003              running
 4     instance-00000004              running

$ virsh console 4
Connected to domain instance-00000004
Escape character is ^]

$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: seq=0 ttl=50 time=62.714 ms
64 bytes from 8.8.8.8: seq=1 ttl=50 time=50.856 ms
64 bytes from 8.8.8.8: seq=2 ttl=50 time=51.502 ms

--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 50.856/55.024/62.714 ms
$ sudo ifconfig eth0 mtu 1300 up
$ wget http://www.google.com
Connecting to www.google.com (172.217.25.196:80)
index.html           100% |*******************************| 11738   0:00:00 ETA
```

VM의 Private IP에 1:1로 Mapping되는 Floating IP를 할당함으로써 외부에서 VM에 접속 가능하다.

Horizon을 통해 VM에 Floating IP를 할당한다.
```
onos> openstack-floatingips
ID                                      Floating IP         Fixed IP
ef28fd5a-7b50-4d9c-88e0-ef5b381ecea4    172.27.0.6          20.2.2.5
```

router 컨테이너에 접속하여 Floating IP를 통해 VM을 접속한다.
```
sdn@mcpark_all_gw:~$ sudo docker ps
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS              PORTS               NAMES
45a90e6d80b1        opensona/router-docker   "/runssh.sh /usr/sbi…"   2 hours ago         Up 2 hours          22/tcp              router
sdn@mcpark_all_gw:~$ sudo docker exec -it 45a90e6d80b1 /bin/su
/ # ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:02
          inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:41 errors:0 dropped:0 overruns:0 frame:0
          TX packets:27 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:15548 (15.1 KiB)  TX bytes:2115 (2.0 KiB)

eth1      Link encap:Ethernet  HWaddr FA:00:00:00:00:01
          inet addr:172.27.0.1  Bcast:0.0.0.0  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:40 errors:0 dropped:0 overruns:0 frame:0
          TX packets:36 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:2841 (2.7 KiB)  TX bytes:14602 (14.2 KiB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

/ # ssh cirros@172.27.0.6
The authenticity of host '172.27.0.6 (172.27.0.6)' can't be established.
RSA key fingerprint is SHA256:RF+1ilWSbrnrJY9C5Na3rGZ6zm4RpfxUCOFo3BtAS+A.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.27.0.6' (RSA) to the list of known hosts.
cirros@172.27.0.6's password:
$
$
$ ifconfig
eth0      Link encap:Ethernet  HWaddr FA:16:3E:8C:77:F4
          inet addr:20.2.2.5  Bcast:20.2.2.0  Mask:255.255.255.0
          inet6 addr: fe80::f816:3eff:fe8c:77f4/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1300  Metric:1
          RX packets:187 errors:0 dropped:0 overruns:0 frame:0
          TX packets:704 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:28643 (27.9 KiB)  TX bytes:52873 (51.6 KiB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:77 errors:0 dropped:0 overruns:0 frame:0
          TX packets:77 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:8504 (8.3 KiB)  TX bytes:8504 (8.3 KiB)

$
```

















 
 



