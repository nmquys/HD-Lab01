# NDLab01 - Ping va Traceroute bang Packet Tracer

**MSSV:** [22110179]  
**Họ tên:** [Nguyen Minh Quy]  
**Phien ban Packet Tracer:** [9.0.1]

## 1. Bai toan va topology

Bai thuc hanh ket noi hai LAN qua hai router va kiem tra duong di den Eagle Server bang ping va tracert. Topology duoi day dung dung thiet bi va mang trong Requirement, khong dung vi du R0-R3 trong Instruction.

```text
hostPod1A (172.16.1.1) --+
                          +-- S1-Central -- Fa0/0 R2-Central S0/0/0
hostPod1B (172.16.1.2) --+                                  |
                                                    10.10.10.4/30
                                                    DCE at R1-ISP
                                                            |
                                       S0/0/0 R1-ISP Fa0/0 -- Eagle Server
                                         10.10.10.6       192.168.254.254
```

Ket noi cong: host A/B vao cac cong access cua S1; cong uplink S1 noi voi `FastEthernet0/0` tren R2; `Serial0/0/0` tren R2 noi voi `Serial0/0/0` tren R1 bang cap serial DCE, dau DCE nam o R1; `FastEthernet0/0` tren R1 noi voi NIC cua Eagle Server. Trong Packet Tracer, chon model/module router co dung ten cong nay va dat dau cap DCE o R1. Dung loai cap Ethernet phu hop hoac Auto-Connect cho cac ket noi Ethernet.

### Bang dia chi

| Thiet bi | Cong giao dien | IPv4 / subnet mask | Gateway / ghi chu |
|---|---|---|---|
| Thiet bi | Cong giao dien | IPv4 / subnet mask | Gateway / ghi chu |
|---|---|---|---|
| R1-ISP | Serial0/0/0 | 10.10.10.6 /30 | Dau DCE noi R2; clock rate 64000 |
| R1-ISP | FastEthernet0/0 | 192.168.254.253 /24 | LAN cua Eagle Server |
| R2-Central | Serial0/0/0 | 10.10.10.5 /30 | Duong WAN den R1 |
| R2-Central | FastEthernet0/0 | 172.16.255.254 /16 | Gateway LAN Pod |
| Eagle Server | NIC | 192.168.254.254 /24 | Gateway 192.168.254.253 |
| hostPod1A | NIC | 172.16.1.1 /16 | Gateway 172.16.255.254 |
| hostPod1B | NIC | 172.16.1.2 /16 | Gateway 172.16.255.254 |
| S1-Central | VLAN 1 (tuy chon) | 172.16.254.1 /16 | Gateway 172.16.255.254 |

Mang WAN la `10.10.10.4/30` (dia chi dung duoc `.5` va `.6`, broadcast `.7`). LAN Pod la `172.16.0.0/16`; LAN server la `192.168.254.0/24`.

### Du doan

Do dung subnet mask `/16`, hai host va dia chi `172.16.255.254` duoc xem la cung mang truc tiep. Luu luong den `192.168.254.254` duoc gui den R2. R2 chuyen tiep qua ket noi serial den R1; R1 chuyen goi den LAN server noi truc tiep. Goi phan hoi can route chieu ve tren R1 den `172.16.0.0/16`, sau do R2 chuyen den host. Vi vay, ping dau-cuoi chi duoc ky vong thanh cong khi cac cong giao dien, cau hinh host, ca hai static route va gateway cua server deu dung.

## 2. Trien khai

Nhap cac lenh router trong [R1-ISP.txt](R1-ISP.txt) va [R2-Central.txt](R2-Central.txt). Cau hinh IP, subnet mask va gateway cho tung PC trong Packet Tracer tai Desktop > IP Configuration. Dat NIC cua Eagle Server la `192.168.254.254 /24`, gateway `192.168.254.253`. S1 khong can dia chi IP de chuyen tiep frame; dia chi VLAN 1 trong bang chi la cau hinh quan tri tuy chon.

### Giai thich CLI

- `enable` vao che do EXEC dac quyen; `configure terminal` vao che do cau hinh toan cuc.
- `interface Serial0/0/0` chon cong WAN. `ip address ...` gan dia chi va subnet mask cho dau ket noi; `/30` tao subnet point-to-point co hai dia chi host dung duoc.
- `clock rate 64000` cap xung nhip cho dau DCE cua ket noi serial. Xac nhan dau DCE bang `show controllers serial 0/0/0`; khong dat lenh nay o dau DTE.
- `interface FastEthernet0/0` chon cong Ethernet cua LAN. Dia chi tren router la default gateway cua cac host; `/16` phu hop voi LAN Pod duoc giao.
- `no shutdown` bat cong giao dien ve mat quan tri. Neu thieu lenh nay, cong van o trang thai administratively down.
- Lenh `ip route 192.168.254.0 255.255.255.0 10.10.10.6` tren R2 gom mang dich cua server, subnet mask dich va dia chi next-hop cua R1. Route tren R1 den `172.16.0.0/16` qua `10.10.10.5` tao duong ve.
- `show ip interface brief` hien thi trang thai cong va line protocol; `show ip route` hien thi cac route connected/local/static; `show running-config` hien thi cau hinh dang chay. `ipconfig`, `ping` va `tracert` kiem tra dia chi host, kha nang ket noi va cac hop tren duong di.
- `copy running-config startup-config` luu cau hinh dang chay de su dung sau khi khoi dong lai.

## 3. Kiem chung va mo phong

Can bo sung ket qua thuc te va tham chieu anh sau khi chay Packet Tracer. Cac muc nay duoc de trang thai chua thuc hien vi workspace khong co phan mem mo phong.

| Ma | Lenh / nguon den dich | Ket qua du kien | Ket qua thuc te / minh chung |
|---|---|---|---|
| T1 | host A: `ipconfig` | IP `172.16.1.1`, subnet mask `255.255.0.0`, gateway `172.16.255.254` | Chua thuc hien - chay lenh va chup anh |
| T2 | host A: `ping 172.16.255.254` | Gateway phan hoi | Chua thuc hien - chay lenh va chup anh |
| T3 | host A: `ping 10.10.10.6` | WAN R1 phan hoi neu route va duong ve dung | Chua thuc hien - chay lenh va chup anh |
| T4 | host A: `ping 192.168.254.254` | Eagle Server phan hoi xuyen mang | Chua thuc hien - chay lenh va chup anh |
| T5 | host A: `tracert 192.168.254.254` | Duong di qua R2, R1 roi den server; co the co hop khong phan hoi | Chua thuc hien - chay lenh va chup anh |
| T6 | host A: `ping 172.16.1.2` | host B phan hoi trong cung LAN `/16` | Chua thuc hien - chay lenh va chup anh |
| T7 | R1/R2: `show ip route` | Co route connected va static route den LAN o xa | Chua thuc hien - chup anh tren ca hai router |

Trong Simulation Mode, xoa hoac kiem tra danh sach su kien, gui goi ICMP tu host A den server va buoc qua tung su kien. Trinh tu du kien: host A dung ARP de tim MAC cua gateway; R2 nhan va dinh tuyen goi IP; lien ket serial chuyen goi den R1; R1 dung ARP de tim server neu can roi chuyen tiep goi; server phan hoi qua R1 va static route chieu ve den R2, sau do R2 chuyen goi den host A. Ghi lai cac su kien ARP/ICMP quan sat duoc va moi diem khac voi du doan.

## 4. Phan tich va xu ly su co

Voi moi loi, chi thay doi cau hinh duoc neu, du doan trieu chung, kiem tra bang cac lenh liet ke, sua loi roi chay lai phep thu lien quan.

1. **Sai gateway tren host (`172.16.255.253` thay vi `.254`).** Luu luong den host cung LAN van co the hoat dong, nhung next-hop da cau hinh khong phai R2 nen luu luong den mang khac se that bai. Kiem tra `ipconfig`, ping gateway va xem `arp -a`; sua gateway thanh `172.16.255.254`. Sau khi sua, ping gateway va cac mang xa du kien thanh cong.
2. **Host B dung `/24` thay vi `/16`.** Gateway `172.16.255.254` bi xem la nam ngoai mang truc tiep nen host khong the tim gateway dung cach. Kiem tra `ipconfig`, ping gateway va xem ARP. Sua subnet mask thanh `255.255.0.0`; sau khi sua, kiem tra gateway, host cung LAN va mang xa.
3. **Fa0/0 tren R2 dang administratively down.** `show ip interface brief` hien `administratively down`; host khong den duoc gateway va tracert khong qua duoc hop dau tien. Bat cong bang `interface Fa0/0`, `no shutdown`; sau do trang thai/protocol du kien up va ping gateway thanh cong.
4. **Mask WAN tren R1 la `/24`, tren R2 la `/30`.** Khong nhat thiet moi phep ping deu that bai: hai dia chi dau cuoi van co the duoc xem la cung mang truc tiep, nhung connected route khong thong nhat va hanh vi dinh tuyen co the gay nham lan. So sanh `show running-config`, `show ip interface brief`, `show ip route` va ping `10.10.10.5`. Dat `/30` o ca hai dau; xac nhan subnet connected la `10.10.10.4/30`, sau do kiem tra lai ket noi dau-cuoi.
5. **Dau DCE chua co clock rate.** Line protocol serial co the van down va goi tin khong qua duoc WAN. Kiem tra DCE/DTE bang `show controllers serial 0/0/0`, xem `show interfaces serial 0/0/0` tren ca hai router. Neu R1 la DCE, dat `clock rate 64000`, dam bao co `no shutdown`, sau do xac nhan line protocol up va ping router dau ben kia.
6. **R1 thieu route chieu ve.** Yeu cau co the den server, nhung goi tra loi den `172.16.0.0/16` khong co route tren R1 nen bi loai bo. Kiem tra `show ip route` tren R1 va R2. Them `ip route 172.16.0.0 255.255.0.0 10.10.10.5` tren R1; sau khi sua, ping va tracert dau-cuoi du kien hoan tat.
7. **Trung dia chi IP host.** Ket noi co the chap chon; ARP co the anh xa dia chi sang cac MAC khac nhau; Packet Tracer cung co the canh bao trung dia chi. Kiem tra `ipconfig`, `arp -a`, bang MAC cua switch va dia chi tren ca hai PC. Gan dia chi rieng biet, xoa/hoc lai ARP neu can, roi kiem tra lai.
8. **Ping thanh cong nhung tracert timeout.** Dieu nay khong chung minh ket noi mat hoan toan: router co the chuyen tiep luu luong nhung khong tra ICMP TTL-exceeded, hoac simulator gioi han phan hoi traceroute. So sanh ping lap lai voi tracert; tren Windows co the dung `tracert -w 1000 -h 4 192.168.254.254` de tang thoi gian cho. Dau `*` chi cho biet khong nhan duoc phan hoi probe, khong khang dinh hop do khong chuyen tiep goi.
9. **Cap/cong giua S1 va R2 cam sai hoac bi rut.** Lien ket LAN co the down hoac noi nham cong. Kiem tra nhan cong/den link, `show ip interface brief` tren R2, `show interfaces status` va `show mac address-table` tren S1, sau do ping gateway. Sua cap va mapping cong; bat cong dang shutdown neu co, roi kiem tra lai.
10. **Gioi han traceroute (`-w 5 -h 4`).** Tren Windows, `-w 5` chi cho moi phan hoi 5 ms nen co the tao dau `*` gia; `-h 4` gioi han toi da bon hop. So sanh voi tracert mac dinh hoac timeout hop ly hon, vi du `tracert -w 1000 -h 4 192.168.254.254`. Tang timeout hoac so hop khi can; gioi han hop thap khong the hien cac hop xa hon.
11. **Gateway cua server la `192.168.254.252`.** Server van co the giao tiep noi mang voi R1, nhung goi phan hoi den host ngoai subnet duoc gui den next-hop sai nen ping dau-cuoi that bai. Kiem tra cau hinh NIC server, ping R1 tu server va ping server tu host A trong khi xem route tren R1. Sua gateway thanh `192.168.254.253`, roi kiem tra lai.
12. **Cay xu ly su co theo tung lop.** (a) Kiem tra IP/mask/gateway tren host bang `ipconfig`; sua neu sai. (b) Ping dia chi cua chinh host va mot host cung LAN; kiem tra cap/link va trung IP. (c) Ping `172.16.255.254`; kiem tra ARP, cong S1 va R2 Fa0/0. (d) Tren R2, kiem tra cong, `show ip route` va ping `10.10.10.6`; xac nhan static route tren R2. (e) Tren R1, kiem tra cong serial/LAN, clocking va route ve `172.16.0.0/16`. (f) Kiem tra IP/mask/gateway server va ping gateway cung LAN. (g) Chay lai ping va tracert dau-cuoi; luu ket qua sau moi lan sua.

## 5. Cau hoi van dap

1. **Router, switch va thiet bi dau cuoi:** router dinh tuyen giua cac subnet IP; S1 chuyen frame Ethernet trong LAN Pod; PC va Eagle Server phat/nhan luu luong ung dung va ICMP.
2. **Gateway cua host:** `172.16.255.254` la cong R2 nam trong subnet `172.16.0.0/16` cua host, nen day la next-hop cho dich nam ngoai subnet.
3. **Subnet WAN:** `10.10.10.5/30` va `.6/30` cung thuoc `10.10.10.4/30`; dia chi mang `.4`, broadcast `.7`, dia chi host dung duoc `.5` va `.6`.
4. **DCE/DTE:** DCE cap xung nhip serial; DTE nhan xung nhip. Chi dat clock rate o dau DCE va xac nhan bang `show controllers serial`.
5. **Trang thai cong:** `administratively down` nghia la cong bi shutdown; `up/down` la lien ket vat ly up nhung line protocol down; `up/up` la ca hai lop deu up. Kiem tra bang `show ip interface brief` va `show interfaces`.
6. **IP dung nhung ping mang xa that bai:** co the do sai gateway/mask, cong hoac cap down, thieu route chieu di/ve, thieu clock serial, sai gateway server, trung IP hoac ICMP bi chan/khong co phan hoi.
7. **Static route va OSPF:** static route duoc cau hinh thu cong, phu hop voi lab nho va co dinh gom hai router; OSPF tu dong hoc route va mo rong tot hon. Bai nay chon static route de the hien ro ca hai chieu va de kiem chung.
8. **`show ip route`:** hien thi bang dinh tuyen. `C` la connected, `L` la dia chi local cua cong router, `S` la static, `O` la route hoc duoc qua OSPF.
9. **Cac loai ping:** ping dia chi cua chinh may kiem tra TCP/IP stack; ping gateway kiem tra ket noi LAN va cong router; ping dia chi mang xa con phu thuoc vao dinh tuyen, duong ve va phan hoi cua host dich.
10. **ICMP Echo:** Echo Request la goi tin tham do; dich den ket noi va phan hoi se tra Echo Reply.
11. **TTL trong tracert:** moi router giam TTL di mot; khi TTL ve 0, router co the tra ICMP Time Exceeded de may gui nhan dien hop.
12. **Time Exceeded va Request timed out:** Time Exceeded la phan hoi cu the tu router trung gian khi TTL het. Request timed out la khong nhan duoc phan hoi mong doi truoc thoi han, nen chua the ket luan nguyen nhan.
13. **Dau `*` trong tracert:** probe do khong nhan phan hoi kip thoi. Rieng dau `*` khong chung minh router khong chuyen tiep goi va cung khong xac dinh duoc goi bi mat o dau.
14. **Tham so tracert tren Windows:** `-w 5` dat thoi gian cho moi probe la 5 ms; `-h 4` dat so hop toi da la 4.
15. **Ping gateway duoc nhung server that bai:** kiem tra route tren R2 den `192.168.254.0/24`, trang thai/clocking WAN, LAN server connected va route chieu ve tren R1, sau do kiem tra IP/gateway server. Dung `show ip route`, trang thai cong, ping tung next-hop va tracert.
16. **Sua CLI:** trong `interface Serial0/0/0`, dat `ip address <correct-address> <correct-mask>` va `no shutdown`; neu day la dau DCE thi cau hinh `clock rate 64000`. Kiem tra trang thai cong va running-config. `shutdown` tat cong nen can thay bang `no shutdown` cho ket noi nay.
17. **Fa0/0 up/down:** carrier vat ly co nhung line protocol down. Kiem tra `show interfaces Fa0/0`, cap/cong phia ben kia va trang thai cong doi dien, encapsulation/keepalive neu ap dung; sau do sua loi lop lien ket va thu lai.
18. **Trinh tu trong Simulation Mode:** host ARP de tim MAC cua R2, gui ICMP Echo den MAC gateway; R2 dinh tuyen goi va giam TTL; R1 nhan goi va chuyen den server (thuc hien ARP truoc neu can); Eagle Server tra loi. Goi tra loi theo static route tren R1 den R2 roi ve host. Tracert gui probe voi TTL tang dan; phan hoi TTL-expired tu cac router cho biet cac hop co tra loi.

### Dien giai CLI mau

`R2(config)# ip route 192.168.254.0 255.255.255.0 10.10.10.6` tao static route den LAN server qua R1. `R1(config)# ip route 172.16.0.0 255.255.0.0 10.10.10.5` tao route chieu ve LAN Pod qua R2. Kiem tra ca hai bang `show ip route`; kiem tra next-hop bang `ping 10.10.10.6` tu R2 va `ping 10.10.10.5` tu R1; sau do ping server tu host Pod theo ca hai chieu neu thiet bi ho tro.

## 6. Minh chung va checklist nop bai

- [ ] Dung va luu tep `.pkt` voi topology, nhan, cong va cap dung yeu cau.
- [ ] Chup ket qua `show ip interface brief`, `show ip route` va running-config lien quan tren router.
- [ ] Chup `ipconfig`, ket qua T1-T7 va cac su kien ARP/ICMP trong Simulation Mode.
- [ ] Thay moi muc ket qua/minh chung chua thuc hien bang ket qua quan sat, Dat/Khong dat, giai thich va tham chieu anh.
- [ ] Dien MSSV, ho ten, lop va phien ban Packet Tracer; xuat report nay thanh PDF voi ten tep theo yeu cau.
- [ ] Kiem tra tep `.pkt` mo va chay duoc, sau do nop report va project theo huong dan. Han nop trong de la 20/10/2026.

## 7. Repository va lich su Git

Repository: [https://github.com/nmquys/HD-Lab01](https://github.com/nmquys/HD-Lab01)

Snapshot lich su commit truoc commit bo sung muc repository va Git log duoc luu trong [GIT_LOG.md](GIT_LOG.md).