ImmortalWrt 25.12.1 + Docker + Canon G3000 AirPrint 重装手册
已验证成功的环境： ImmortalWrt 25.12.1 / EFI + ext4 / N100 四网口 / Canon G3000 USB / Docker / AirPrint
一、最终成功方案
Docker 镜像：chuckcharlie/cups-avahi-airprint:latest
LAN 地址：192.168.5.1
CUPS 管理员：admin
Canon USB ID：04a9:1794
最终打印队列：Canon_G3000_series@immortalwrt
最终验证：Windows 已自动发现并连接 Canon G3000 series @ cups-airprint。
二、Docker 网络问题时关闭 IPv6
sysctl -w net.ipv6.conf.all.disable_ipv6=1
sysctl -w net.ipv6.conf.default.disable_ipv6=1
cat > /etc/docker/daemon.json <<'EOF'
{
  "ipv6": false,
  "ip6tables": false
}
EOF
/etc/init.d/dockerd restart
sleep 5
三、拉取镜像
docker pull chuckcharlie/cups-avahi-airprint:latest
四、删除旧容器并清空旧配置
docker rm -f cups 2>/dev/null
rm -rf ~/cups
mkdir -p ~/cups
chmod 777 ~/cups
五、创建 CUPS + AirPrint 容器（核心命令）
docker run -d \
  --name cups \
  --restart unless-stopped \
  --net=host \
  --privileged \
  -v ~/cups:/config \
  -v /dev/bus/usb:/dev/bus/usb \
  -e CUPSADMIN=admin \
  -e CUPSPASSWORD=admin \
  -e AVAHI_HOSTNAME=cups-airprint \
  chuckcharlie/cups-avahi-airprint:latest
六、检查容器和 USB
docker ps
docker logs cups --tail 100
docker exec cups ps -ef | grep -E 'cupsd|avahi|dbus'
docker exec cups lsusb
docker exec cups lpstat -p -d
docker exec cups lpstat -v
Canon G3000 正常情况下应能看到 USB ID：04a9:1794。
七、进入 CUPS 添加打印机
浏览器打开：http://192.168.5.1:631
Administration 登录：用户名 admin；默认密码 admin。
进入 Administration → Add Printer → 选择 Canon G3000 USB 打印机。
添加完成后确认打印机为 Shared Yes / Accepting Yes。
八、确认打印机配置
docker exec cups lpstat -l -p Canon_G3000_series@immortalwrt
docker exec cups grep -i -A 20 'Canon_G3000' /config/printers.conf
成功配置中应看到：Shared Yes、Accepting Yes、State Idle。
九、防火墙（LAN 访问 CUPS + mDNS）
uci add firewall rule
uci set firewall.@rule[-1].name='CUPS'
uci set firewall.@rule[-1].src='lan'
uci set firewall.@rule[-1].target='ACCEPT'
uci set firewall.@rule[-1].proto='tcp'
uci set firewall.@rule[-1].dest_port='631'
uci commit firewall

uci add firewall rule
uci set firewall.@rule[-1].name='mDNS'
uci set firewall.@rule[-1].src='lan'
uci set firewall.@rule[-1].target='ACCEPT'
uci set firewall.@rule[-1].proto='udp'
uci set firewall.@rule[-1].dest_port='5353'
uci commit firewall

/etc/init.d/firewall restart
十、重置 CUPS Administration 密码
只需要修改 admin 密码时：
docker exec cups sh -c 'echo "admin:新密码" | chpasswd'
例如：
docker exec cups sh -c 'echo "admin:12345678" | chpasswd'
如果 admin 用户本身不存在，可重新创建：
docker exec cups sh -c 'adduser -S -G lpadmin --no-create-home admin 2>/dev/null || true; echo "admin:新密码" | chpasswd'
十一、最终验证
Windows：设置 → 蓝牙和设备 → 打印机和扫描仪 → 添加设备。
应自动发现：Canon G3000 series @ cups-airprint，并最终显示“打印机准备就绪”。
iPhone：照片 → 分享 → 打印 → 选择打印机，应能看到 Canon G3000 series。
十二、这次安装中特别要避免的错误
•	不要换回 ydkn/cups。
•	不要把宿主机 ~/cups/config 挂载到容器 /etc/cups。
•	不要挂载 /var/run/dbus。
•	不要挂载 /var/spool/cups。
•	不要挂载 /etc/avahi/services。
•	不要手工启动容器内的 avahi-daemon 或 dbus-daemon；该镜像启动脚本会负责。
•	必须保留 --net=host，否则 AirPrint 的 mDNS 组播可能无法正常工作。
•	不要因为容器里没有 avahi-browse 就判断 AirPrint 失败；该命令在本次镜像中并不存在。
十三、最精简的一键恢复命令
# 如果 Docker 已安装并且需要恢复这套已验证配置

sysctl -w net.ipv6.conf.all.disable_ipv6=1
sysctl -w net.ipv6.conf.default.disable_ipv6=1

cat > /etc/docker/daemon.json <<'EOF'
{
  "ipv6": false,
  "ip6tables": false
}
EOF

/etc/init.d/dockerd restart
sleep 5

docker pull chuckcharlie/cups-avahi-airprint:latest

docker rm -f cups 2>/dev/null
rm -rf ~/cups
mkdir -p ~/cups
chmod 777 ~/cups

docker run -d \
  --name cups \
  --restart unless-stopped \
  --net=host \
  --privileged \
  -v ~/cups:/config \
  -v /dev/bus/usb:/dev/bus/usb \
  -e CUPSADMIN=admin \
  -e CUPSPASSWORD=admin \
  -e AVAHI_HOSTNAME=cups-airprint \
  chuckcharlie/cups-avahi-airprint:latest

docker ps
docker logs cups --tail 100
docker exec cups lsusb

注意：上面的“一键恢复命令”负责恢复容器和基础环境；Canon G3000 打印机队列是在 CUPS 网页 Administration → Add Printer 中添加的。若 ~/cups 被删除，原有打印机配置也会被删除，需要重新添加。
