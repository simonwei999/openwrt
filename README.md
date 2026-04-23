qichiyu introduced openclash customized rule:

https://www.youtube.com/watch?v=D841V_xgykg&t=2885s


ssh -L 8631:127.0.0.1:631 root@1190vm66237cm.vicp.fun -p 55099

上海电脑打开 PowerShell，运行隧道命令（保持窗口开着）
ssh -L 8631:127.0.0.1:631 root@1190vm66237cm.vicp.fun -p 55099
添加打印机 → 不在列表中
选择 TCP/IP 设备
主机名 / IP 固定填：127.0.0.1
下一步，设备类型选择：
IPP 打印机 / Internet Printing Protocol
就能搜到苏州的 CUPS 打印机
你可以简单验证一下
隧道连上之后，在上海电脑浏览器打开：
http://127.0.0.1:8631
如果能打开 CUPS 管理页面，就说明隧道通了，打印肯定能用。
打不开再告诉我，我帮你看是哪里卡住。







如何确定上海电脑是否成功连接到苏州N100？
如何在上海电脑上添加打印机？
如何在上海电脑上配置IPP打印机/Internet打印机？
