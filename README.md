# 使用
1. 修改docker-compose.yml中consul服务hostname为节点名，如consulserver1，consulserver2（不重复即可）
2. 修改consul.json中`bind_addr`为服务器内网ip
3. 修改consul.json中`retry_join`为其他节点ip
4. 启动：`docker-compose up -d`

# 生成agent token
上面对于第一个节点启动的时候会遇到
```
2017/07/08 23:38:24 [WARN] agent: Node info update blocked by ACLs
2017/07/08 23:38:44 [WARN] agent: Coordinate update blocked by ACLs
```
原因在于还未设置`agent token`，生成步骤如下：

```
$ curl \
    --request PUT \
    --header "X-Consul-Token: your_master_token" \
    --data \
'{
  "Name": "Agent Token",
  "Type": "client",
  "Rules": "node \"\" { policy = \"write\" } service \"\" { policy = \"read\" }"
}' http://127.0.0.1:8500/v1/acl/create

{"ID":"fe3b8d40-0ee0-8783-6cc2-ab1aa9bb16c1"}
```
然后加入配置文件，`acl_agent_token`填入上面生成的token
```
{
  "acl_datacenter": "dc1",
  "acl_master_token": "your_master_token",
  "acl_default_policy": "deny",
  "acl_down_policy": "extend-cache",
  "acl_agent_token": "fe3b8d40-0ee0-8783-6cc2-ab1aa9bb16c1"
}
```
重启agent  
对于其他节点，可以不停机加入agent token
```
$ curl \
    --request PUT \
    --header "X-Consul-Token: b1gs33cr3t" \
    --data \
'{
  "Token": "fe3b8d40-0ee0-8783-6cc2-ab1aa9bb16c1"
}' http://127.0.0.1:8500/v1/agent/token/acl_agent_token
```
[详细说明](https://www.consul.io/docs/guides/acl-legacy.html)

