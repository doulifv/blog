同步系统时间
chronyd 命令可让您检查系统时钟关闭的时间。如果您在未先安装该实用程序的情况下运行 chronyd 命令，您将收到以下消息：
首先，让我们按如下方式安装它：
安装
```
sudo apt install chrony
```
检查
```
sudo chronyd -Q
```
校准时间
```
sudo chronyd -q
```
卸载
```
apt purge chrony
```

---
查看时区列表
```
timedatectl list-timezones
```
设置系统时区为香港时间
```
timedatectl set-timezone Asia/Hong_Kong
```
手动设置时间和日期
```
timedatectl set-time "2017-06-02 23:26:00"
```
手动设置年月日
```
timedatectl set-time YYYY-MM-DD
```
手动设置时间
```
timedatectl set-time 23:26:00
```
[如何在 Debian 10 上保持系统时间与互联网时间服务器同步 (linux-console.net)](https://cn.linux-console.net/?p=30106#:~:text=%E5%8D%95%E5%87%BB%20Debian%20%E6%A1%8C%E9%9D%A2%E5%8F%B3%E4%B8%8A%E8%A7%92%E7%9A%84%E5%90%91%E4%B8%8B%E7%AE%AD%E5%A4%B4%EF%BC%8C%E7%84%B6%E5%90%8E%E5%8D%95%E5%87%BB%E5%B7%A6%E4%B8%8B%E8%A7%92%E7%9A%84%E8%AE%BE%E7%BD%AE%E5%9B%BE%E6%A0%87%EF%BC%9A%20%E6%88%96%E8%80%85,%E5%9C%A8%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F%E5%90%AF%E5%8A%A8%E5%99%A8%E6%90%9C%E7%B4%A2%E4%B8%AD%E9%94%AE%E5%85%A5%E8%AE%BE%E7%BD%AE%EF%BC%8C%E5%A6%82%E4%B8%8B%E6%89%80%E7%A4%BA%EF%BC%9A%20%E5%8D%95%E5%87%BB%E2%80%9C%E8%AF%A6%E7%BB%86%E4%BF%A1%E6%81%AF%E2%80%9D%E9%80%89%E9%A1%B9%E5%8D%A1%EF%BC%8C%E7%84%B6%E5%90%8E%E9%80%89%E6%8B%A9%E2%80%9C%E6%97%A5%E6%9C%9F%E5%92%8C%E6%97%B6%E9%97%B4%E2%80%9D%E9%80%89%E9%A1%B9%E3%80%82%20%E5%8D%95%E5%87%BB%E2%80%9C%E6%97%A5%E6%9C%9F%E5%92%8C%E6%97%B6%E9%97%B4%E2%80%9D%E8%A7%86%E5%9B%BE%E4%B8%8A%E7%9A%84%E2%80%9C%E8%A7%A3%E9%94%81%E2%80%9D%E6%8C%89%E9%92%AE%EF%BC%8C%E5%B9%B6%E5%9C%A8%E2%80%9C%E8%BA%AB%E4%BB%BD%E9%AA%8C%E8%AF%81%E2%80%9D%E5%AF%B9%E8%AF%9D%E6%A1%86%E4%B8%AD%E6%8F%90%E4%BE%9B%20sudo%2F%E6%8E%88%E6%9D%83%E7%94%A8%E6%88%B7%E7%9A%84%E5%AF%86%E7%A0%81%E3%80%82%20%E7%84%B6%E5%90%8E%EF%BC%8C%E7%A1%AE%E4%BF%9D%E2%80%9C%E8%87%AA%E5%8A%A8%E6%97%A5%E6%9C%9F%E5%92%8C%E6%97%B6%E9%97%B4%E2%80%9D%E6%8C%89%E9%92%AE%E5%B7%B2%E6%89%93%E5%BC%80%E3%80%82)
[Red Hat Enterprise Linux 系统管理员操作指南 第 3 章 配置日期和时间](https://docs.redhat.com/zh-cn/documentation/red_hat_enterprise_linux/7/html/system_administrators_guide/chap-configuring_the_date_and_time)