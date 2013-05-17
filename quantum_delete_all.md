清空 quantum 数据

*作者: Daniels Cai
*主页: http://dnscai.com

1.清空floatingip 
---

    for i in `quantum floatingip-list  | grep -v "\---"  | grep -v "^| id" | awk '{print $2}' `; do quantum floatingip-delete $i ; done


2.清空路由gateway 
---

    for i in `quantum router-list  | grep -v "\---"  | grep -v "^| id" | awk '{print $2}' `; do quantum router-gateway-clear $i ; done


查看路由下的 port
    quantum port-list -- --device_id 64ffbf96-5cba-4043-bef2-b3a846d1ab42 --device_owner network:router_gateway

查看路由本身的port
    quantum port-list -- --device_id 64ffbf96-5cba-4043-bef2-b3a846d1ab42 

3. 清空路由所有的interface
---

    for i in `quantum router-list  | grep -v "\---"  | grep -v "^| id" | awk '{print $2}' `; do quantum router-interface-delete $i  `quantum port-list -- --device_id  $i  | grep subnet_id | sed 's/.*subnet_id": "\([^"]*\).*/\1/g'` ; done 

4.删除路由
---

    for i in `quantum router-list  | grep -v "\---"  | grep -v "^| id" | awk '{print $2}' `; do quantum router-delete $i  `quantum port-list -- --device_id  $i  | grep subnet_id | sed 's/.*subnet_id": "\([^"]*\).*/\1/g'` ; done 


5.清空port
---

for i in `quantum port-list  | grep -v "\---"  | grep -v "^| id" | awk '{print $2}' `;do quantum port-delete $i ; done 


6. 删除subnet 
---

    for i in `quantum  subnet-list | grep -v "\---"  | grep -v "^| id" | awk '{print $2}' `;do  quantum subnet-delete $i ; done

7. 删除网络
---

    for i in `quantum  net-list | grep -v "\---"  | grep -v "^| id" | awk '{print $2}' `;do  quantum net-delete $i ; done








