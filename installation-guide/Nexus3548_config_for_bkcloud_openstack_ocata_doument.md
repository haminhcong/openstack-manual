# Cấu hình cài đặt Switch 10 Gbps Cisco Nexus 3548 để triển khai hệ thống bkcloud OpenStack Ocata

Như đã nêu ở sơ đồ triển khai hệ thống bkcloud OpenStack Ocata, trên Switch 10 Gbps Cisco Nexus 3548 sẽ triển khai các mạng private VLAN có id từ 500-700 và mạng volume data network, do đó chúng ta cần cấu hình Switch 10 Gbps sao cho:

- Cho phép các packet của các mạng VLAN có id từ 500-700 có thể truyền thông giữa các máy tính trong hệ thống cloud thông qua switch.
- Cho phép các packet L2 thông thường (untagged packet) có thể đi qua switch, vì mạng volume data network sử dụng untagged packet.

Để đáp ứng 2 yêu cầu trên, chúng ta tiến hành cấu hình switch 10 Gbps:

- Sử dụng dây kết nối với switch, cắm console port vào switch và cắm USB port vào máy tính.
- Sử dụng phần mềm PuTTY trên Windows để kết nối tới Switch.
- Đăng nhập vào switch với user admin. Nếu không có admin password cần thực hiện việc reset admin password.
- Sử dụng các câu lệnh để cấu hình cho switch.
- Lưu lại cấu hình và chờ 3-5 phút cho switch chấp nhận cấu hình.
- Kiểm tra xem switch đã hoạt động đúng yêu cầu hay chưa.

## Reset admin password trên Switch 10 Gbps Cisco Nexus 3548

Để thực hiện việc reset admin password trên Switch Cisco Nexus 3548, chúng ta làm như sau:

- recycle power

- Khi màn hình BIOS của Switch hiển thị, nhấn Ctrl-] liên tục để vào boot mode của switch - Enter bios mode, press Ctrl-] to enter boot mode.
- To reset admin password:

```conf

switch(boot)# configure terminal
switch(boot-config)# admin-password <new password>
WARNING! Remote Authentication for login through console has been
disabled
switch(boot-config)# exit
switch(boot)#

```

Sau khi thay đổi được password, tiến hành boot vào OS của switch theo các bước sau:

## Cisco Nexus 3548 configuration command for bkcloud OpenStack Ocata

Các câu lệnh được sử dụng để để cấu hình cho Cisco 10 Gbps Switch đáp ứng hai yêu cầu của hệ thống bkcloud OpenStack Ocata đã được nêu ra phía trên như sau:

- Kiểm tra xem các port của các máy tính trong hệ thống đang kết nối tới các port nào thông qua quan sát switch và câu lệnh: 

```bash

cisco3548p(config-vlan)# show interface brief

--------------------------------------------------------------------------------
Ethernet      VLAN   Type Mode   Status  Reason                   Speed     Port
Interface                                                                   Ch #
--------------------------------------------------------------------------------
Eth1/1        80      eth  trunk  up      none                        10G(D) --
Eth1/2        80      eth  trunk  up      none                        10G(D) --
Eth1/3        1       eth  access down    SFP not inserted            10G(D) --
Eth1/4        80      eth  trunk  up      none                        10G(D) --
Eth1/5        80      eth  trunk  up      none                        10G(D) --
Eth1/6        80      eth  trunk  up      none                        10G(D) --
Eth1/7        80      eth  trunk  up      none                        10G(D) --
Eth1/8        1       eth  access down    SFP not inserted            10G(D) --
Eth1/9        1       eth  access down    SFP not inserted            10G(D) --
Eth1/10       1       eth  access down    SFP not inserted            10G(D) --
Eth1/11       1       eth  access down    SFP not inserted            10G(D) --
Eth1/12       1       eth  access down    SFP not inserted            10G(D) --
Eth1/13       1       eth  access down    SFP not inserted            10G(D) --
Eth1/14       1       eth  access down    SFP not inserted            10G(D) --
Eth1/15       1       eth  access down    SFP not inserted            10G(D) --
Eth1/16       1       eth  access down    SFP not inserted            10G(D) --
Eth1/17       1       eth  access down    SFP not inserted            10G(D) --
Eth1/18       1       eth  access down    SFP not inserted            10G(D) --
Eth1/19       1       eth  access down    SFP not inserted            10G(D) --
Eth1/20       1       eth  access down    SFP not inserted            10G(D) --
Eth1/21       1       eth  access down    SFP not inserted            10G(D) --
Eth1/22       1       eth  access down    SFP not inserted            10G(D) --
Eth1/23       1       eth  access down    SFP not inserted            10G(D) --
Eth1/24       1       eth  access down    SFP not inserted            10G(D) --
Eth1/25       1       eth  access down    SFP not inserted            10G(D) --
Eth1/26       1       eth  access down    SFP not inserted            10G(D) --
Eth1/27       1       eth  access down    SFP not inserted            10G(D) --
Eth1/28       1       eth  access down    SFP not inserted            10G(D) --
Eth1/29       1       eth  access down    SFP not inserted            10G(D) --
Eth1/30       1       eth  access down    SFP not inserted            10G(D) --
Eth1/31       1       eth  access down    SFP not inserted            10G(D) --
Eth1/32       1       eth  access down    SFP not inserted            10G(D) --
Eth1/33       1       eth  access down    SFP not inserted            10G(D) --
Eth1/34       1       eth  access down    SFP not inserted            10G(D) --
Eth1/35       1       eth  access down    SFP not inserted            10G(D) --
Eth1/36       1       eth  access down    SFP not inserted            10G(D) --
Eth1/37       1       eth  access down    SFP not inserted            10G(D) --
Eth1/38       1       eth  access down    SFP not inserted            10G(D) --
Eth1/39       1       eth  access down    SFP not inserted            10G(D) --
Eth1/40       1       eth  access down    SFP not inserted            10G(D) --
Eth1/41       1       eth  access down    SFP not inserted            10G(D) --
Eth1/42       1       eth  access down    SFP not inserted            10G(D) --
Eth1/43       1       eth  access down    SFP not inserted            10G(D) --
Eth1/44       1       eth  access down    SFP not inserted            10G(D) --
Eth1/45       1       eth  access down    SFP not inserted            10G(D) --
Eth1/46       1       eth  access down    SFP not inserted            10G(D) --
Eth1/47       1       eth  access down    SFP not inserted            10G(D) --
Eth1/48       1       eth  access down    SFP not inserted            10G(D) --

--------------------------------------------------------------------------------
Port   VRF          Status IP Address                              Speed    MTU
--------------------------------------------------------------------------------
mgmt0  --           up     10.1.3.102                              1000     1500

-------------------------------------------------------------------------------
Interface Secondary VLAN(Type)                    Status Reason
-------------------------------------------------------------------------------
Vlan1     --                                      up     --

```

- Tạo trên switch mạng volume data network có id=80 để làm mạng native VLAN cho phép các untagged packet của máy ảo và ổ đĩa đi qua các card mạng.

```conf

configure terminal
vlan 80
name volume-data-network
state active
no shutdown

```

- Tạo trến switch các mạng VLAN có id 500-700 để switch có thể nhận diện và định tuyến- chuyển tiếp cho các gói tin của các mạng VLAN private network của hệ thống OpenStack (Trên ml2 của neutron khi chúng ta thiết lập cấu hình cho vlan cũng cần thiết lập vào đúng dải này):

```conf

configure terminal
vlan 500-700
state active
no shutdown

```

- Thiết lập interface các máy trong hệ thống ở chế độ **trunk mode**, cho phép các VLAN có id từ 1-1000 đi qua các trunk port này, và thiết lập native VLAN của các interface vận chuyển các untagging packet của mạng volume-data-network là 80.

```conf

configure terminal
interface ethernet 1/1
switchport mode trunk
switchport trunk native vlan 80
switchport trunk allow vlan 1-1000

exit

configure terminal
interface ethernet 1/2
switchport mode trunk
switchport trunk native vlan 80
switchport trunk allow vlan 1-1000

exit

configure terminal
interface ethernet 1/4
switchport mode trunk
switchport trunk native vlan 80
switchport trunk allow vlan 1-1000

exit

configure terminal
interface ethernet 1/5
switchport mode trunk
switchport trunk native vlan 80
switchport trunk allow vlan 1-1000

exit

configure terminal
interface ethernet 1/6
switchport mode trunk
switchport trunk native vlan 80
switchport trunk allow vlan 1-1000

exit

configure terminal
interface ethernet 1/7
switchport mode trunk
switchport trunk native vlan 80
switchport trunk allow vlan 1-1000

exit

```

- Cấu hình để cho phép untagged traffic đi qua các trunk port của switch:

```conf

configure terminal
no vlan dot1q tag native

```

Thông tin thêm: 

- <https://www.cisco.com/c/m/en_us/techdoc/dc/reference/cli/nxos/commands/l2/vlan-dot1q-tag-native.html>
- <https://supportforums.cisco.com/t5/lan-switching-and-routing/native-vlan-tagging/td-p/2267039>
- Lưu lại các cấu hình đã thay đổi trên switch vào bộ nhớ để switch tự cấu hình vào lần khởi động lại sau này:

```conf

copy running-config startup-config

```

- Thoát ra khỏi chế độ cấu hình và kiểm tra lại trạng thái cấu hình của switch và của các cổng đã đúng với yêu cầu hay chưa:

```conf

cisco3548p(config-vlan)# show interface brief

--------------------------------------------------------------------------------
Ethernet      VLAN   Type Mode   Status  Reason                   Speed     Port
Interface                                                                   Ch #
--------------------------------------------------------------------------------
Eth1/1        80      eth  trunk  up      none                        10G(D) --
Eth1/2        80      eth  trunk  up      none                        10G(D) --
Eth1/3        1       eth  access down    SFP not inserted            10G(D) --
Eth1/4        80      eth  trunk  up      none                        10G(D) --
Eth1/5        80      eth  trunk  up      none                        10G(D) --
Eth1/6        80      eth  trunk  up      none                        10G(D) --
Eth1/7        80      eth  trunk  up      none                        10G(D) --
Eth1/8        1       eth  access down    SFP not inserted            10G(D) --
Eth1/9        1       eth  access down    SFP not inserted            10G(D) --
Eth1/10       1       eth  access down    SFP not inserted            10G(D) --
Eth1/11       1       eth  access down    SFP not inserted            10G(D) --
Eth1/12       1       eth  access down    SFP not inserted            10G(D) --
Eth1/13       1       eth  access down    SFP not inserted            10G(D) --
Eth1/14       1       eth  access down    SFP not inserted            10G(D) --
Eth1/15       1       eth  access down    SFP not inserted            10G(D) --
Eth1/16       1       eth  access down    SFP not inserted            10G(D) --
Eth1/17       1       eth  access down    SFP not inserted            10G(D) --
Eth1/18       1       eth  access down    SFP not inserted            10G(D) --
Eth1/19       1       eth  access down    SFP not inserted            10G(D) --
Eth1/20       1       eth  access down    SFP not inserted            10G(D) --
Eth1/21       1       eth  access down    SFP not inserted            10G(D) --
Eth1/22       1       eth  access down    SFP not inserted            10G(D) --
Eth1/23       1       eth  access down    SFP not inserted            10G(D) --
Eth1/24       1       eth  access down    SFP not inserted            10G(D) --
Eth1/25       1       eth  access down    SFP not inserted            10G(D) --
Eth1/26       1       eth  access down    SFP not inserted            10G(D) --
Eth1/27       1       eth  access down    SFP not inserted            10G(D) --
Eth1/28       1       eth  access down    SFP not inserted            10G(D) --
Eth1/29       1       eth  access down    SFP not inserted            10G(D) --
Eth1/30       1       eth  access down    SFP not inserted            10G(D) --
Eth1/31       1       eth  access down    SFP not inserted            10G(D) --
Eth1/32       1       eth  access down    SFP not inserted            10G(D) --
Eth1/33       1       eth  access down    SFP not inserted            10G(D) --
Eth1/34       1       eth  access down    SFP not inserted            10G(D) --
Eth1/35       1       eth  access down    SFP not inserted            10G(D) --
Eth1/36       1       eth  access down    SFP not inserted            10G(D) --
Eth1/37       1       eth  access down    SFP not inserted            10G(D) --
Eth1/38       1       eth  access down    SFP not inserted            10G(D) --
Eth1/39       1       eth  access down    SFP not inserted            10G(D) --
Eth1/40       1       eth  access down    SFP not inserted            10G(D) --
Eth1/41       1       eth  access down    SFP not inserted            10G(D) --
Eth1/42       1       eth  access down    SFP not inserted            10G(D) --
Eth1/43       1       eth  access down    SFP not inserted            10G(D) --
Eth1/44       1       eth  access down    SFP not inserted            10G(D) --
Eth1/45       1       eth  access down    SFP not inserted            10G(D) --
Eth1/46       1       eth  access down    SFP not inserted            10G(D) --
Eth1/47       1       eth  access down    SFP not inserted            10G(D) --
Eth1/48       1       eth  access down    SFP not inserted            10G(D) --

--------------------------------------------------------------------------------
Port   VRF          Status IP Address                              Speed    MTU
--------------------------------------------------------------------------------
mgmt0  --           up     10.1.3.102                              1000     1500

-------------------------------------------------------------------------------
Interface Secondary VLAN(Type)                    Status Reason
-------------------------------------------------------------------------------
Vlan1     --                                      up     --


cisco3548p(config-vlan)# show interface switchport
Name: Ethernet1/1
  Switchport: Enabled
  Switchport Monitor: Not enabled
  Operational Mode: trunk
  Access Mode VLAN: 1 (default)
  Trunking Native Mode VLAN: 80 (VLAN0080)
  Trunking VLANs Allowed: 1-1000
  Administrative private-vlan primary host-association: none
  Administrative private-vlan secondary host-association: none
  Administrative private-vlan primary mapping: none
  Administrative private-vlan secondary mapping: none
  Administrative private-vlan trunk native VLAN: none
  Administrative private-vlan trunk encapsulation: dot1q
  Administrative private-vlan trunk normal VLANs: none
  Administrative private-vlan trunk private VLANs: none(0 none)
  Operational private-vlan: none
  Unknown unicast blocked: disabled
  Unknown multicast blocked: disabled

Name: Ethernet1/2
  Switchport: Enabled
  Switchport Monitor: Not enabled
  Operational Mode: trunk
  Access Mode VLAN: 1 (default)
  Trunking Native Mode VLAN: 80 (VLAN0080)
  Trunking VLANs Allowed: 1-1000
  Administrative private-vlan primary host-association: none
  Administrative private-vlan secondary host-association: none
  Administrative private-vlan primary mapping: none
  Administrative private-vlan secondary mapping: none
  Administrative private-vlan trunk native VLAN: none
  Administrative private-vlan trunk encapsulation: dot1q
  Administrative private-vlan trunk normal VLANs: none
  Administrative private-vlan trunk private VLANs: none(0 none)
  Operational private-vlan: none
  Unknown unicast blocked: disabled
  Unknown multicast blocked: disabled

Name: Ethernet1/3
  Switchport: Enabled
  Switchport Monitor: Not enabled
  Operational Mode: access
  Access Mode VLAN: 1 (default)
  Trunking Native Mode VLAN: 1 (default)
  Trunking VLANs Allowed: 1-4094
  Administrative private-vlan primary host-association: none
  Administrative private-vlan secondary host-association: none
  Administrative private-vlan primary mapping: none
  Administrative private-vlan secondary mapping: none
  Administrative private-vlan trunk native VLAN: none
  Administrative private-vlan trunk encapsulation: dot1q
  Administrative private-vlan trunk normal VLANs: none
  Administrative private-vlan trunk private VLANs: none(0 none)
  Operational private-vlan: none
  Unknown unicast blocked: disabled
  Unknown multicast blocked: disabled

Name: Ethernet1/4
  Switchport: Enabled
  Switchport Monitor: Not enabled
  Operational Mode: trunk
  Access Mode VLAN: 1 (default)
  Trunking Native Mode VLAN: 80 (VLAN0080)
  Trunking VLANs Allowed: 1-1000
  Administrative private-vlan primary host-association: none
  Administrative private-vlan secondary host-association: none
  Administrative private-vlan primary mapping: none
  Administrative private-vlan secondary mapping: none
  Administrative private-vlan trunk native VLAN: none
  Administrative private-vlan trunk encapsulation: dot1q
  Administrative private-vlan trunk normal VLANs: none
  Administrative private-vlan trunk private VLANs: none(0 none)
  Operational private-vlan: none
  Unknown unicast blocked: disabled
  Unknown multicast blocked: disabled

Name: Ethernet1/5
  Switchport: Enabled
  Switchport Monitor: Not enabled
  Operational Mode: trunk
  Access Mode VLAN: 1 (default)
  Trunking Native Mode VLAN: 80 (VLAN0080)
  Trunking VLANs Allowed: 1-1000
  Administrative private-vlan primary host-association: none
  Administrative private-vlan secondary host-association: none
  Administrative private-vlan primary mapping: none
  Administrative private-vlan secondary mapping: none
  Administrative private-vlan trunk native VLAN: none
  Administrative private-vlan trunk encapsulation: dot1q
  Administrative private-vlan trunk normal VLANs: none
  Administrative private-vlan trunk private VLANs: none(0 none)
  Operational private-vlan: none
  Unknown unicast blocked: disabled
  Unknown multicast blocked: disabled

Name: Ethernet1/6
  Switchport: Enabled
  Switchport Monitor: Not enabled
  Operational Mode: trunk
  Access Mode VLAN: 1 (default)
  Trunking Native Mode VLAN: 80 (VLAN0080)
  Trunking VLANs Allowed: 1-1000
  Administrative private-vlan primary host-association: none
  Administrative private-vlan secondary host-association: none
  Administrative private-vlan primary mapping: none
  Administrative private-vlan secondary mapping: none
  Administrative private-vlan trunk native VLAN: none
  Administrative private-vlan trunk encapsulation: dot1q
  Administrative private-vlan trunk normal VLANs: none
  Administrative private-vlan trunk private VLANs: none(0 none)
  Operational private-vlan: none
  Unknown unicast blocked: disabled
  Unknown multicast blocked: disabled

Name: Ethernet1/7
  Switchport: Enabled
  Switchport Monitor: Not enabled
  Operational Mode: trunk
  Access Mode VLAN: 1 (default)
  Trunking Native Mode VLAN: 80 (VLAN0080)
  Trunking VLANs Allowed: 1-1000
  Administrative private-vlan primary host-association: none
  Administrative private-vlan secondary host-association: none
  Administrative private-vlan primary mapping: none
  Administrative private-vlan secondary mapping: none
  Administrative private-vlan trunk native VLAN: none
  Administrative private-vlan trunk encapsulation: dot1q
  Administrative private-vlan trunk normal VLANs: none
  Administrative private-vlan trunk private VLANs: none(0 none)
  Operational private-vlan: none
  Unknown unicast blocked: disabled
  Unknown multicast blocked: disabled

  
cisco3548p(config-vlan)# show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Eth1/1, Eth1/2, Eth1/3, Eth1/4
                                                Eth1/5, Eth1/6, Eth1/7, Eth1/8
                                                Eth1/9, Eth1/10, Eth1/11
                                                Eth1/12, Eth1/13, Eth1/14
                                                Eth1/15, Eth1/16, Eth1/17
                                                Eth1/18, Eth1/19, Eth1/20
                                                Eth1/21, Eth1/22, Eth1/23
                                                Eth1/24, Eth1/25, Eth1/26
                                                Eth1/27, Eth1/28, Eth1/29
                                                Eth1/30, Eth1/31, Eth1/32
                                                Eth1/33, Eth1/34, Eth1/35
                                                Eth1/36, Eth1/37, Eth1/38
                                                Eth1/39, Eth1/40, Eth1/41
                                                Eth1/42, Eth1/43, Eth1/44
                                                Eth1/45, Eth1/46, Eth1/47
                                                Eth1/48
80   VLAN0080                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
500  VLAN0500                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
501  VLAN0501                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
502  VLAN0502                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
503  VLAN0503                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
504  VLAN0504                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
505  VLAN0505                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
506  VLAN0506                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
507  VLAN0507                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
508  VLAN0508                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
509  VLAN0509                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
510  VLAN0510                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
511  VLAN0511                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
512  VLAN0512                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
513  VLAN0513                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
514  VLAN0514                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
515  VLAN0515                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
516  VLAN0516                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
517  VLAN0517                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
518  VLAN0518                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
519  VLAN0519                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
520  VLAN0520                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
521  VLAN0521                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
522  VLAN0522                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
523  VLAN0523                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
524  VLAN0524                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
525  VLAN0525                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
526  VLAN0526                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
527  VLAN0527                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
528  VLAN0528                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
529  VLAN0529                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
530  VLAN0530                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
531  VLAN0531                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
532  VLAN0532                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
533  VLAN0533                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
534  VLAN0534                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
535  VLAN0535                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
536  VLAN0536                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
537  VLAN0537                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
538  VLAN0538                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
539  VLAN0539                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
540  VLAN0540                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
541  VLAN0541                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
542  VLAN0542                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
543  VLAN0543                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
544  VLAN0544                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
545  VLAN0545                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
546  VLAN0546                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
547  VLAN0547                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
548  VLAN0548                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
549  VLAN0549                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
550  VLAN0550                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
551  VLAN0551                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
552  VLAN0552                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
553  VLAN0553                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
554  VLAN0554                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
555  VLAN0555                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
556  VLAN0556                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
557  VLAN0557                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
558  VLAN0558                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
559  VLAN0559                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
560  VLAN0560                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
561  VLAN0561                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
562  VLAN0562                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
563  VLAN0563                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
564  VLAN0564                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
565  VLAN0565                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
566  VLAN0566                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
567  VLAN0567                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
568  VLAN0568                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
569  VLAN0569                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
570  VLAN0570                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
571  VLAN0571                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
572  VLAN0572                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
573  VLAN0573                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
574  VLAN0574                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
575  VLAN0575                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
576  VLAN0576                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
577  VLAN0577                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
578  VLAN0578                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
579  VLAN0579                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
580  VLAN0580                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
581  VLAN0581                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
582  VLAN0582                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
583  VLAN0583                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
584  VLAN0584                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
585  VLAN0585                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
586  VLAN0586                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
587  VLAN0587                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
588  VLAN0588                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
589  VLAN0589                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
590  VLAN0590                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
591  VLAN0591                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
592  VLAN0592                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
593  VLAN0593                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
594  VLAN0594                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
595  VLAN0595                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
596  VLAN0596                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
597  VLAN0597                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
598  VLAN0598                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
599  VLAN0599                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
600  VLAN0600                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
601  VLAN0601                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
602  VLAN0602                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
603  VLAN0603                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
604  VLAN0604                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
605  VLAN0605                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
606  VLAN0606                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
607  VLAN0607                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
608  VLAN0608                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
609  VLAN0609                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
610  VLAN0610                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
611  VLAN0611                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
612  VLAN0612                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
613  VLAN0613                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
614  VLAN0614                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
615  VLAN0615                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
616  VLAN0616                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
617  VLAN0617                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
618  VLAN0618                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
619  VLAN0619                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
620  VLAN0620                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
621  VLAN0621                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
622  VLAN0622                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
623  VLAN0623                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
624  VLAN0624                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
625  VLAN0625                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
626  VLAN0626                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
627  VLAN0627                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
628  VLAN0628                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
629  VLAN0629                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
630  VLAN0630                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
631  VLAN0631                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
632  VLAN0632                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
633  VLAN0633                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
634  VLAN0634                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
635  VLAN0635                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
636  VLAN0636                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
637  VLAN0637                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
638  VLAN0638                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
639  VLAN0639                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
640  VLAN0640                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
641  VLAN0641                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
642  VLAN0642                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
643  VLAN0643                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
644  VLAN0644                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
645  VLAN0645                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
646  VLAN0646                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
647  VLAN0647                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
648  VLAN0648                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
649  VLAN0649                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
650  VLAN0650                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
651  VLAN0651                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
652  VLAN0652                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
653  VLAN0653                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
654  VLAN0654                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
655  VLAN0655                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
656  VLAN0656                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
657  VLAN0657                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
658  VLAN0658                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
659  VLAN0659                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
660  VLAN0660                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
661  VLAN0661                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
662  VLAN0662                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
663  VLAN0663                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
664  VLAN0664                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
665  VLAN0665                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
666  VLAN0666                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
667  VLAN0667                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
668  VLAN0668                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
669  VLAN0669                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
670  VLAN0670                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
671  VLAN0671                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
672  VLAN0672                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
673  VLAN0673                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
674  VLAN0674                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
675  VLAN0675                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
676  VLAN0676                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
677  VLAN0677                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
678  VLAN0678                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
679  VLAN0679                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
680  VLAN0680                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
681  VLAN0681                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
682  VLAN0682                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
683  VLAN0683                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
684  VLAN0684                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
685  VLAN0685                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
686  VLAN0686                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
687  VLAN0687                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
688  VLAN0688                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
689  VLAN0689                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
690  VLAN0690                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
691  VLAN0691                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
692  VLAN0692                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
693  VLAN0693                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
694  VLAN0694                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
695  VLAN0695                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
696  VLAN0696                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
697  VLAN0697                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
698  VLAN0698                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
699  VLAN0699                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
700  VLAN0700                         active    Eth1/1, Eth1/2, Eth1/4, Eth1/5
                                                Eth1/6, Eth1/7
                                                
```