# iptables
# tạm thời là note các ý chính


##### a. Bảng filter

Trong bảng filter có các chain được xây dựng sẵn:
irewall
-  chain INPUT: đây là chain dùng để lọc các gói tin vào firewall
- Chain OUTPUT: đây là chain dùng để lọc các gói tin ra firewall
- Chain FORWARD : đây là chain dùng để chuyển gói tin qua lại giữa các card mạng với nhau. ( ở đây tôi hiểu là nếu là hệ thống 1-1 thì có lẽ không cần thiết, còn nếu là hệ thống  PC1--firewall--PC2 thì nếu chỉ có INPUT HAY OUTPUT thì giữa 2 máy PC vẫn chưa trao đổi với nhau được mà phải thông qua chain này).

NOte: bạn nên hiểu thế này firewall giống 1 cái hộp đen khi các packet đi vào sẽ được chứa trong nó rồi tùy bạn quyết định sẽ làm gì với các packet trong đó. 

Các target được sử dụng trong các chain này có thể là ACCEPT (đồng ý) , DROP (xóa bỏ)



##### b. Bảng NAT
Trong bảng NAT có các **chain** và có 3 chain được xây dựng sẵn trong table NAT:
- **Chain** PREROUTING : đấy là chain dùng để thay đổi địa chỉ đích của gói tin và trong chain FREROUTING target(tác vụ) được sử dụng là DNAT (destination NAT):
VD:
```
# iptables -t NAT -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 10.0.30.100:80
```
Câu lệnh này có ý nghĩa. Đối với gói tin tcp có port đích là 80 thì sẽ được đổi địa chỉ đích thành 10.0.30.100
ví dụ trên đã định nghĩa rõ cho chúng ta:nói ra chọn bảng NAT áp dụng rule và trong chain PREROUTING sử dụng target DNAT.

- **Chain** POSROUTING : đây là chain dùng để thay đổi địa chỉ nguồn của gói tin và targer được sư dụng là SNAT (source NAT) 
VD:
```
#iptables -t NAT -A POSTROUTING -p tcp -j SNAT --to-source 10.0.30.200
```
Câu lệnh này có ý nghĩa đổi địa chỉ nguồn đối với gói tin tcp thành 10.0.30.200

- **Chain** OUTPUT : ít dùng nên bỏ qua












phần ví dụ về snat ( 1 kiểu ping từ lan ra wan chứ không phải là kỹ thuật giúp ping từ lan ra wan, ở đây có thể hiểu là nó giúp cho local truy nhập được ra ngoài internet, mà internet là môi trường mở nên khi ta ping đến một địa chỉ thuộc mạng wan thì nó giống như kiểu ta truy nhập vào 1 dịch vụ nào đó trên internet ví dụ như web,mail...):

SNAT & MASQUERADE
Để tạo kết nối `transparent` giữa mạng LAN 192.168.0.1 với Internet bạn lập cấu hình cho tường lửa Iptables như sau:
# echo 1 > /proc/sys/net/ipv4/ip_forward cho phép forward các packet qua máy chủ đặt Iptables
# iptables -t nat -A POSTROUTING -o eth0 -j SNAT –to-source 210.40.2.71 đổi IP nguồn cho các packet ra card mạng eth0 là 210.40.2.71. Khi nhận được packet vào từ Internet, Iptables sẽ tự động đổi IP đích 210.40.2.71 thành IP đích tương ứng của máy tính trong mạng LAN 192.168.0/24.
Hoặc bạn có thể dùng MASQUERADE thay cho SNAT như sau:
# iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
(MASQUERADE thường được dùng khi kết nối đến Internet là pp0 và dùng địa chỉ IP động)

phần ví dụ về DNAT ( lúc đầu tôi nghĩ snat là 1 kiểu kỹ thuật giúp ta ping được từ lan ra wan và rôi áp suy nghĩ đó vào dnat nhưng không được thì tôi mới biết là mình đã nhâm, DNAT là kỹ thuật giúp chúng ta public các dịch vụ trong local ra ngoài hoặc cho phép mạng bên ngoài truy nhập đến được nhưng thông qua các port ví dụ như, dịch vụ web chẳng hạn cổng 80 -- 172.16.1.130:80 ...):

DNAT
Giả sử bạn đặt các máy chủ Proxy, Mail và DNS trong mạng DMZ. Để tạo kết nối trong suốt từ Internet vào các máy chủ này bạn là như sau:
# echo 1 > /proc/sys/net/ipv4/ip_forward
# iptables -t nat -A PREROUTING -i eth0 -p tcp –dport 80 -j DNAT –to-destination 192.168.1.2
# iptables -t nat -A PREROUTING -i eth0 -p tcp –dport 25 -j DNAT –to-destination 192.168.1.3
# iptables -t nat -A PREROUTING -i eth0 -p udp –dport 53 -j DNAT –to-destination 192.168.1.4

prerouting: ( DNAT for incoming traffic ) tức là khi từ mạng ngoài eth0 muốn truy nhập vào mạng local thì nó sẽ ánh xạ địa chỉ ( -j DNAT --to-destination ip_local) wan tới địa chỉ local

postrouting: tức là định tuyến xong rôi sẽ nat địa chỉ cho phù hợp với bảng định tuyến của firewall ( ví dụ: địa chỉ ở eth1 sẽ kiểu như routing với eth0 rồi từ đấy nó sẽ ra internet thông qua [nat qua] eth0)


Các địa chỉ IP nội bộ được chuyển sang IP NAT như sau:
NAT Router đảm nhận việc chuyển dãy IP nội bộ 169.168.0.x sang dãy IP mới 203.162.2.x. Khi có gói liệu với IP nguồn là 192.168.0.200 đến router, router sẽ đổi IP nguồn thành 203.162.2.200 sau đó mới gởi ra ngoài. Quá trình này gọi là SNAT (Source-NAT, NAT nguồn). Router lưu dữ liệu trong một bảng gọi là bảng NAT động. Ngược lại, khi có một gói từ liệu từ gởi từ ngoài vào với IP đích là 203.162.2.200, router sẽ căn cứ vào bảng NAT động hiện tại để đổi địa chỉ đích 203.162.2.200 thành địa chỉ đích mới là 192.168.0.200. Quá trình này gọi là DNAT (Destination-NAT, NAT đích). Liên lạc giữa 192.168.0.200 và 203.162.2.200 là hoàn toàn trong suốt (transparent) qua NAT router. NAT router tiến hành chuyển tiếp (forward) gói dữ liệu từ 192.168.0.200 đến 203.162.2.200 và ngược lại.

Cách đóng giả địa chỉ IP (masquerade)
Đây là một kĩ thuật khác trong NAT.
NAT Router chuyển dãy IP nội bộ 192.168.0.x sang một IP duy nhất là 203.162.2.4 bằng cách dùng các số hiệu cổng (port-number) khác nhau. Chẳng hạn khi có gói dữ liệu IP với nguồn 192.168.0.168:1204, đích 211.200.51.15:80 đến router, router sẽ đổi nguồn thành 203.162.2.4:26314 và lưu dữ liệu này vào một bảng gọi là bảng masquerade động. Khi có một gói dữ liệu từ ngoài vào với nguồn là 221.200.51.15:80, đích 203.162.2.4:26314 đến router, router sẽ căn cứ vào bảng masquerade động hiện tại để đổi đích từ 203.162.2.4:26314 thành 192.168.0.164:1204


