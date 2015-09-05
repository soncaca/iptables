# iptables
# tạm thời là note các ý chính

phần ví dụ về snat ( ping từ lan ra wan) :

SNAT & MASQUERADE
Để tạo kết nối `transparent` giữa mạng LAN 192.168.0.1 với Internet bạn lập cấu hình cho tường lửa Iptables như sau:
# echo 1 > /proc/sys/net/ipv4/ip_forward cho phép forward các packet qua máy chủ đặt Iptables
# iptables -t nat -A POSTROUTING -o eth0 -j SNAT –to-source 210.40.2.71 đổi IP nguồn cho các packet ra card mạng eth0 là 210.40.2.71. Khi nhận được packet vào từ Internet, Iptables sẽ tự động đổi IP đích 210.40.2.71 thành IP đích tương ứng của máy tính trong mạng LAN 192.168.0/24.
Hoặc bạn có thể dùng MASQUERADE thay cho SNAT như sau:
# iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
(MASQUERADE thường được dùng khi kết nối đến Internet là pp0 và dùng địa chỉ IP động)

phần ví dụ về DNAT ( ping từ wan vào lan ):

DNAT
Giả sử bạn đặt các máy chủ Proxy, Mail và DNS trong mạng DMZ. Để tạo kết nối trong suốt từ Internet vào các máy chủ này bạn là như sau:
# echo 1 > /proc/sys/net/ipv4/ip_forward
# iptables -t nat -A PREROUTING -i eth0 -p tcp –dport 80 -j DNAT –to-destination 192.168.1.2
# iptables -t nat -A PREROUTING -i eth0 -p tcp –dport 25 -j DNAT –to-destination 192.168.1.3
# iptables -t nat -A PREROUTING -i eth0 -p udp –dport 53 -j DNAT –to-destination 192.168.1.4
