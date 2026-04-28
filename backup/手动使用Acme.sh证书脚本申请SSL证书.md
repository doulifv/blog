
#准备材料
一台VPS
一枚域名
CloudFlare账号（如使用CF API）

#部署步骤
###安装Acme.sh证书申请脚本
1.运行以下命令，安装Acme.sh证书申请脚本
```
curl https://get.acme.sh | sh -s email=my@example.com
```
>由于Acme.sh脚本调用了GitHub的资源，且GitHub不支持纯IPv6的环境，所以请自行设置DNS64或安装WARP解决
建议替换[my@example.com](mailto:my@example.com)为自己的邮箱
2.切换默认证书签发的CA机构为下面三家的任何一家

Let’s Encrypt （签发速度快、稳定）
```
bash ~/.acme.sh/acme.sh --set-default-ca --server letsencrypt
```
ZeroSSL （稳定、但部分情况无法签发）
```
bash ~/.acme.sh/acme.sh --set-default-ca --server zerossl
```
Buypass（没试过）
```
bash ~/.acme.sh/acme.sh --set-default-ca --server buypass
```
3.设置Acme.sh自动更新，始终与官方保持一致
```
bash ~/.acme.sh/acme.sh --upgrade --auto-upgrade
```
###使用Standalone模式申请SSL证书
此方法使用之前请确保80端口畅通，并且域名已经事先解析到VPS的IP
域名解析至IPv4：
```
bash ~/.acme.sh/acme.sh --issue -d "域名" --standalone -k ec-256
```
域名已解析至IPv6：
```
bash ~/.acme.sh/acme.sh --issue -d "域名" --standalone -k ec-256 --listen-v6
```
###使用CloudFlare API Key申请SSL证书
此方法可以使用泛域名、无需DNS解析，但由于受到CF API限制，不可适用于Freenom系列的免费域名
1.设置CloudFlare Global API Key和登录邮箱
```
export CF_Key="你自己的CloudFlare Global API Key"
export CF_Email="你自己的CloudFlare账户登录邮箱"
```
2.运行一下命令
####单域名
IPv4 或原生双栈VPS
```
bash ~/.acme.sh/acme.sh --issue --dns dns_cf -d "域名" -k ec-256
```
IPv6
```
bash ~/.acme.sh/acme.sh --issue --dns dns_cf -d "域名" -k ec-256 --listen-v6
```
####泛域名
IPv4 或原生双栈VPS
```
bash ~/.acme.sh/acme.sh --issue --dns dns_cf -d "*.域名" -d "域名" -k ec-256
```
IPv6
```
bash ~/.acme.sh/acme.sh --issue --dns dns_cf -d "*.域名" -d "域名" -k ec-256 --listen-v6
```
####安装域名证书
由于acme.sh的证书不能直接使用，因此我们需要安装证书
单域名
```
bash ~/.acme.sh/acme.sh --install-cert -d "域名" --key-file /root/private.key --fullchain-file /root/cert.crt --ecc
```
泛域名
```
bash ~/.acme.sh/acme.sh --install-cert -d "*.域名" --key-file /root/private.key --fullchain-file /root/cert.crt --ecc
```
>运行此命令后，证书crt路径为/root/cert.crt，私钥key路径为/root/private.key，可自行修改

####查看目前申请的证书
```
bash ~/.acme.sh/acme.sh --list
```
####撤销目前申请的证书
```
bash ~/.acme.sh/acme.sh --revoke -d "域名" --ecc
bash ~/.acme.sh/acme.sh --remove -d "域名" --ecc
```
####手动续期证书
```
bash ~/.acme.sh/acme.sh --renew -d "域名" --force --ecc
```
####卸载Acme.sh脚本
```
bash ~/.acme.sh/acme.sh --uninstall
```
[[DELETE]手动使用Acme.sh证书脚本申请SSL证书 | MisakaNo の 小破站](https://blog.misaka.rest/2023/04/02/acme-ssl-cert/?highlight=acme)