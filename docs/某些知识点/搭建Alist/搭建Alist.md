# 搭建Alist

### 在linux中搭建宝塔

>自己的
 
外网面板地址: http://101.36.116.202:26353/c2f7b819
内网面板地址: http://10.7.84.68:26353/c2f7b819
username: ews24oln
password: 5080d0a6

### 下载Alist


``curl -fsSL "https://nn.ci/alist.sh" | bash -s install``


 下载成功
Alist 安装成功！

访问地址：http://YOUR_IP:5244/

配置文件路径：/opt/alist/data/config.json
$查看管理员信息，请执行
cd /opt/alist
./alist admin

用户名和密码：

```
sername: admin
password: hsB9yfnF

```

查看状态：systemctl status alist
启动服务：systemctl start alist
重启服务：systemctl restart alist
停止服务：systemctl stop alist

温馨提示：如果端口无法正常访问，请检查 服务器安全组、本机防火墙、Alist状态
