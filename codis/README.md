## zabbix监控[codis](https://github.com/CodisLabs/codis)
### 使用方法 
* 将codis.py复制到/etc/zabbix/scripts/下，将userparameter_codis.conf复制到/etc/zabbix/zabbix_agent.d/下<br/>
* [安装requests,kazoo,zabbix_sender](https://github.com/vipwangtian/zabbix/blob/master/doc/install_libs.md)
* zabbix-agent默认是在zabbix用户下执行的，要执行系统命令需要赋权限，执行如下命令
```shell
echo "zabbix ALL=(root) NOPASSWD:/bin/netstat" > /etc/sudoers.d/zabbix
echo 'Defaults:zabbix   !requiretty'  >>  /etc/sudoers.d/zabbix
chmod 600  /etc/sudoers.d/zabbix
```
* 重启zabbix-agent<br/>
```shell
service zabbix-agent restart
```
* 在web端导入Template App Codis Auto Discovery.xml模板，链接到codis-proxy和codis-server部署的主机<br/>
* Template App Codis FE.xml用于监控codis-fe是否存活，可以忽略，只检测9090端口是否被监听。
### 实现原理
模板中包含自动发现，自动发现会调用客户端codis.py --listserver发现codis-server，codis.py --listproxy发现codis-proxy。<br/>
发现方式为从zookeeper中读取集群配置。<br/>
模板中监控项原型proxy-check和Redis-性能数据-check分别每隔30s和60s触发一次zabbix-agent客户端，调用codis.py -p $1 -k \[get_item_proxy/get_item_server\]获取监控数据，数据会通过zabbix_sender上传到zabbix_server。
