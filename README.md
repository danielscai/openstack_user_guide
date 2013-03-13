 openstack 基本功能


1.虚拟机操作
==================


1.1. 从image启动虚拟机
------------------

需要参数 image id 和 flavor id

运行`nova image-list` 得到image id

     # nova image-list
     +--------------------------------------+-------------------------+--------+--------+
     | ID                                   | Name                    | Status | Server |
     +--------------------------------------+-------------------------+--------+--------+
     | 9ff7fcf4-f502-41c5-9a85-9a89ba2493f4 | centos_4M               | ACTIVE |        |
     | 533a5c7b-2177-4db2-b128-b88ac1779b5b | precise                 | ACTIVE |        |
     +--------------------------------------+-------------------------+--------+--------+

运行`nova flavor-list`得到flavor id

     # nova flavor-list
     +----+-----------+-----------+------+-----------+------+-------+-------------+-----------+-------------+
     | ID | Name      | Memory_MB | Disk | Ephemeral | Swap | VCPUs | RXTX_Factor | Is_Public | extra_specs |
     +----+-----------+-----------+------+-----------+------+-------+-------------+-----------+-------------+
     | 1  | m1.tiny   | 512       | 0    | 0         |      | 1     | 1.0         | True      | {}          |
     | 2  | m1.small  | 2048      | 20   | 0         |      | 1     | 1.0         | True      | {}          |
     | 3  | m1.medium | 4096      | 40   | 0         |      | 2     | 1.0         | True      | {}          |
     | 4  | m1.large  | 8192      | 80   | 0         |      | 4     | 1.0         | True      | {}          |
     | 5  | m1.xlarge | 16384     | 160  | 0         |      | 8     | 1.0         | True      | {}          |
     +----+-----------+-----------+------+-----------+------+-------+-------------+-----------+-------------+

运行`nova boot`命令，启动一台虚拟机，使用precise 的image，m1.tiny的flavor

     # nova  boot --image 533a5c7b-2177-4db2-b128-b88ac1779b5b --flavor 1  vm1

1.2. 从volume启动虚拟机
------------------

主要参数和从image 启动一样，另外需要 block_device_mapping参数

使用`nova volume-create` 命令，从一个可启动的image 生成一个可启动的volume，image格式文件必须是raw格式

     # nova volume-create --image-id 533a5c7b-2177-4db2-b128-b88ac1779b5b --display-name precise-vol 10
     +---------------------+--------------------------------------+
     | Property            | Value                                |
     +---------------------+--------------------------------------+
     | attachments         | []                                   |
     | availability_zone   | nova                                 |
     | created_at          | 2013-03-12T07:47:03.224001           |
     | display_description | None                                 |
     | display_name        | precise-vol                          |
     | id                  | f18fc147-c16d-4a5a-a5cb-44cef08a6352 |
     | image_id            | 533a5c7b-2177-4db2-b128-b88ac1779b5b |
     | metadata            | {}                                   |
     | size                | 10                                   |
     | snapshot_id         | None                                 |
     | status              | creating                             |
     | volume_type         | None                                 |
     +---------------------+--------------------------------------+

使用 nova boot 命令，从这个volume启动

*tips:虽然是从volume启动，但是任然需要--image 参数，openstack不会从这个image启动*
*这是folsome版的一个bug，这个bug已经在grizzly版中得到修正，参考 http://docs.openstack.org/trunk/openstack-compute/admin/content/boot-from-volume.html
https://bugs.launchpad.net/nova/+bug/1008622*

     # nova boot --image 533a5c7b-2177-4db2-b128-b88ac1779b5b --flavor 1 --block_device_mapping vda=f18fc147-c16d-4a5a-a5cb-44cef08a6352:::0 vm2-vol

1.3. 删除虚拟机
------------------

运行`nova delete` 删除虚拟机

     # nova delete vm2-vol

运行`nova list`查看是否删除成功

1.4. 迁移虚拟机
------------------

使用`nova-manage service list`查看可用的nova-compute节点，
:-)状态为正常节点，XXX状态为故障，需要修复

     # nova-manage service list 
     Binary           Host                                 Zone             Status     State Updated_At
     nova-cert        controller2                          nova             enabled    :-)   2013-03-12 08:04:46
     nova-consoleauth controller2                          nova             enabled    :-)   2013-03-12 08:04:46
     nova-scheduler   controller2                          nova             enabled    :-)   2013-03-12 08:04:45
     nova-compute     compute1                             nova             enabled    :-)   2013-03-12 08:04:44
     nova-compute     compute2                             nova             enabled    XXX   2013-03-11 07:29:21

运行 `nova show` 查看虚拟机运行在哪一个物理计算节点上

     # nova show vm1
     | OS-EXT-SRV-ATTR:hypervisor_hostname | compute1 

运行 `nova live-migration` ，迁移到compute2节点上

     # nova live-migration vm1 compute2

1.5. 对instance进行快照
------------------

使用 `nova image-create` 对运行的instance进行快照，得到的快照是一个image,可以使用nova image-list 查看

     # nova image-create vm1 vm1-snap

1.6. 对instance的volume进行快照
------------------

如果虚拟机是从volume启动，对虚拟机快照就是直接对虚拟机启动的volume进行快照。

运行`cinder list`查看vm启动的volume id ，attached to 是虚拟机id

     # cinder list 
     +--------------------------------------+-----------+--------------------------------------+------+-------------+--------------------------------------+
     |                  ID                  |   Status  |             Display Name             | Size | Volume Type |             Attached to              |
     | 444da795-8482-463d-aa27-d31b3b0c6b4f |   in-use  | ce87b5bf-8295-4569-821e-82f3e2c923bd |  10  |     None    | f18fc147-c16d-4a5a-a5cb-44cef08a6352 |
     +--------------------------------------+-----------+--------------------------------------+------+-------------+--------------------------------------+

使用`nova volume-snapshot-create` 对虚拟机的volume进行快照

     # nova volume-snapshot-create --force True --display-name vol-snap1 444da795-8482-463d-aa27-d31b3b0c6b4f
     +---------------------+--------------------------------------+
     | Property            | Value                                |
     +---------------------+--------------------------------------+
     | created_at          | 2013-03-12T09:25:16.669465           |
     | display_description | None                                 |
     | display_name        | vol-snap1                            |
     | id                  | 8cd8c57a-5ffa-4c92-98a2-ec10ae899166 |
     | size                | 10                                   |
     | status              | creating                             |
     | volume_id           | 444da795-8482-463d-aa27-d31b3b0c6b4f |
     +---------------------+--------------------------------------+

运行 `cinder snapshot-list` 查看

     # cinder snapshot-list
     +--------------------------------------+--------------------------------------+-----------+------------------------------+------+
     |                  ID                  |              Volume ID               |   Status  |         Display Name         | Size |
     +--------------------------------------+--------------------------------------+-----------+------------------------------+------+
     | 8cd8c57a-5ffa-4c92-98a2-ec10ae899166 | 444da795-8482-463d-aa27-d31b3b0c6b4f | available |          vol-snap1           |  10  |
     +--------------------------------------+--------------------------------------+-----------+------------------------------+------+

1.7. 从volume的snapshot启动
------------------

先将volume的snapshot转换成一个新的volume ，大小为10G

     # nova volume-create --snapshot-id 8cd8c57a-5ffa-4c92-98a2-ec10ae899166 --display-name vol-from-snap 10 
     +---------------------+--------------------------------------+
     | Property            | Value                                |
     +---------------------+--------------------------------------+
     | attachments         | []                                   |
     | availability_zone   | nova                                 |
     | created_at          | 2013-03-12T09:34:53.498823           |
     | display_description | None                                 |
     | display_name        | vol-from-snap                        |
     | id                  | f18b2df7-b3be-44ad-b521-c69f04ab8c49 |
     | metadata            | {}                                   |
     | size                | 10                                   |
     | snapshot_id         | 8cd8c57a-5ffa-4c92-98a2-ec10ae899166 |
     | status              | creating                             |
     | volume_type         | None                                 |
     +---------------------+--------------------------------------+

得到新的volume id 后，从volume启动

     # nova boot --image 533a5c7b-2177-4db2-b128-b88ac1779b5b --flavor 1 --block_device_mapping vda=f18b2df7-b3be-44ad-b521-c69f04ab8c49:::0 vm2

1.8. 使用metadata
------------------

编写`user data`,参考https://help.ubuntu.com/community/CloudInit

**更改主机名**

     # cat cloudconfig.yaml
     #cloud-config
     hostname: mynode
     fqdn: mynode.example.com

启动时，加上 `--user-data` 参数,如果metadata service工作正常，主机名会变成mynode,反之则证明metadata service工作不正常

     # nova boot --user-data cloudconfig.yaml --image 533a5c7b-2177-4db2-b128-b88ac1779b5b --flavor 1  vm3

**注入脚本**

    # cat myscript.sh
    #!/bin/sh
    echo "Hello World.  The time is now $(date -R)!" | tee /root/output.txt

这段脚本会记录被注入脚本运行的时间，记录在 `/root/output.txt` 路径下。

     # nova boot --user-data myscript.sh --image 533a5c7b-2177-4db2-b128-b88ac1779b5b --flavor 1 --block_device_mapping vda=f18fc147-c16d-4a5a-a5cb-44cef08a6352:::0 vm4

登录到虚拟机查看是否生效

     root@vm4:~# cat output.txt 
     Hello World.  The time is now Wed, 13 Mar 2013 02:58:12 +0000!

**插入文件**
（测试没有成功）

     # cat write_files 
     #cloud-config
     write_files:
     -   encoding: b64
         content: CiMgVGhpcyBmaWxlIGNvbnRyb2xzIHRoZSBzdGF0ZSBvZiBTRUxpbnV4...
         owner: root:root
         path: /root/write_files
         permissions: '0644'
     -   encoding: gzip
         content: !!binary |
             H4sIAIDb/U8C/1NW1E/KzNMvzuBKTc7IV8hIzcnJVyjPL8pJ4QIA6N+MVxsAAAA=
         path: /usr/bin/hello
         permissions: '0755'

创建新的虚拟机测试

     # nova boot --user-data write_files --image 533a5c7b-2177-4db2-b128-b88ac1779b5b --flavor 1 --block_device_mapping vda=92952076-d80b-4fc0-ae4b-894638d1972b:::0 vm5

更多实例，请参考http://bazaar.launchpad.net/~cloud-init-dev/cloud-init/trunk/files/head:/doc/examples/

1.9. 虚拟机扩容
------------------



2.存储操作
==================

2.1. 镜像操作
------------------

**创建镜像

自己制作镜像，或者去 [cloud ubuntu]: http://cloud-images.ubuntu.com/ 下载ubuntu官方镜像，ubuntu官方镜像已经集成cloudinit 可以直接使用metadata service   
下载 precise的img文件 ，并上传到glance中

     # wget http://cloud-images.ubuntu.com/precise/current/precise-server-cloudimg-amd64-disk1.img
     # glance image-create --name precise --is-public true --container-format bare --disk-format qcow2 < precise-server-cloudimg-amd64-disk1.img
     +------------------+--------------------------------------+
     | Property         | Value                                |
     +------------------+--------------------------------------+
     | checksum         | 2cd973b67600cef111a2c951ef1084d2     |
     | container_format | bare                                 |
     | created_at       | 2013-03-13T05:10:06                  |
     | deleted          | False                                |
     | deleted_at       | None                                 |
     | disk_format      | qcow2                                |
     | id               | 2fd11347-c951-4f81-ac54-205afd783791 |
     | is_public        | True                                 |
     | min_disk         | 0                                    |
     | min_ram          | 0                                    |
     | name             | precise                              |
     | owner            | None                                 |
     | protected        | False                                |
     | size             | 233832448                            |
     | status           | active                               |
     | updated_at       | 2013-03-13T05:10:31                  |
     +------------------+--------------------------------------+

运行 `glance image-list` 查看镜像状态，active为可以使用。

     # glance image-list
     +--------------------------------------+-------------------------+-------------+------------------+-------------+--------+
     | ID                                   | Name                    | Disk Format | Container Format | Size        | Status |
     +--------------------------------------+-------------------------+-------------+------------------+-------------+--------+
     | 2fd11347-c951-4f81-ac54-205afd783791 | precise                 | qcow2       | bare             | 233832448   | active |
     +--------------------------------------+-------------------------+-------------+------------------+-------------+--------+

**删除镜像

运行 `glance image-delete` 删除对应的image，参数为image id

     # glance image-delete 2fd11347-c951-4f81-ac54-205afd783791


2.2. 卷操作 
------------------

2.2.1 创建，删除卷
------------------

**创建空卷

`cinder create` 需要一个参数size，这里指定10，默认单位为G;名称为empty-vol 

     # cinder create --display-name empty-vol 10
     +---------------------+--------------------------------------+
     |       Property      |                Value                 |
     +---------------------+--------------------------------------+
     |     attachments     |                  []                  |
     |  availability_zone  |                 nova                 |
     |      created_at     |      2013-03-13T05:19:05.765784      |
     | display_description |                 None                 |
     |     display_name    |              empty-vol               |
     |          id         | eed6ef03-14f4-4cac-8182-3e19999f896d |
     |       metadata      |                  {}                  |
     |         size        |                  10                  |
     |     snapshot_id     |                 None                 |
     |        status       |               creating               |
     |     volume_type     |                 None                 |
     +---------------------+--------------------------------------+

**从image 创建卷
    
使用`glance image-list`查看可用image id，运行`cinder create`命令创建volume

     # cinder create --image-id 533a5c7b-2177-4db2-b128-b88ac1779b5b --display-name vol-from-image 10
     +---------------------+--------------------------------------+
     |       Property      |                Value                 |
     +---------------------+--------------------------------------+
     |     attachments     |                  []                  |
     |  availability_zone  |                 nova                 |
     |      created_at     |      2013-03-13T05:27:47.160512      |
     | display_description |                 None                 |
     |     display_name    |            vol-from-image            |
     |          id         | 4f111593-12e9-471b-939b-d72fee230cbd |
     |       image_id      | 533a5c7b-2177-4db2-b128-b88ac1779b5b |
     |       metadata      |                  {}                  |
     |         size        |                  10                  |
     |     snapshot_id     |                 None                 |
     |        status       |               creating               |
     |     volume_type     |                 None                 |
     +---------------------+--------------------------------------+

**对卷进行snapshot

对卷进行snapshot，参数为volume id

     # cinder snapshot-create --display-name vol-snapshot1 4f111593-12e9-471b-939b-d72fee230cbd

运行`cinder snapshot-list`查看snapshot

     # cinder snapshot-list
     +--------------------------------------+--------------------------------------+-----------+------------------------------+------+
     |                  ID                  |              Volume ID               |   Status  |         Display Name         | Size |
     +--------------------------------------+--------------------------------------+-----------+------------------------------+------+
     | 647e1bb6-6087-48e4-8310-1b3047893ae5 | 4f111593-12e9-471b-939b-d72fee230cbd |  creating |        vol-snapshot1         |  10  |
     +--------------------------------------+--------------------------------------+-----------+------------------------------+------+

**从volume snapshot创建卷

在虚拟机从volume snapshot启动相关章节有详细描述

     # nova volume-create --snapshot-id 8cd8c57a-5ffa-4c92-98a2-ec10ae899166 --display-name vol-from-snap 10 
     +---------------------+--------------------------------------+
     | Property            | Value                                |
     +---------------------+--------------------------------------+
     | attachments         | []                                   |
     | availability_zone   | nova                                 |
     | created_at          | 2013-03-12T09:34:53.498823           |
     | display_description | None                                 |
     | display_name        | vol-from-snap                        |
     | id                  | f18b2df7-b3be-44ad-b521-c69f04ab8c49 |
     | metadata            | {}                                   |
     | size                | 10                                   |
     | snapshot_id         | 8cd8c57a-5ffa-4c92-98a2-ec10ae899166 |
     | status              | creating                             |
     | volume_type         | None                                 |
     +---------------------+--------------------------------------+

2.2.2 在虚拟机中挂载卸载卷
-----------------------

**向虚拟机中添加卷

使用`nova volume-attach <server> <volume> <device>`将卷添加到虚拟机中

     # nova volume-attach b30c0914-b87b-41d4-a381-7168d6be4cf3 eed6ef03-14f4-4cac-8182-3e19999f896d /dev/vdb
     +----------+--------------------------------------+
     | Property | Value                                |
     +----------+--------------------------------------+
     | device   | /dev/vdb                             |
     | id       | eed6ef03-14f4-4cac-8182-3e19999f896d |
     | serverId | b30c0914-b87b-41d4-a381-7168d6be4cf3 |
     | volumeId | eed6ef03-14f4-4cac-8182-3e19999f896d |
     +----------+--------------------------------------+

到虚拟机中查看vdb是都添加成功
     # fdisk -l  | grep vdb
     Disk /dev/vdb doesn't contain a valid partition table
     Disk /dev/vdb: 10.7 GB, 10737418240 bytes

**将卷从虚拟机中卸载

使用`nova volume-detach <server> <volume>`将卷从虚拟机中卸载，卸载前请保证磁盘已经umount，否则数据无法保证数据一致性。

     # nova volume-detach b30c0914-b87b-41d4-a381-7168d6be4cf3 eed6ef03-14f4-4cac-8182-3e19999f896d

到虚拟机中查看vdb是都卸载成功
     # fdisk -l  | grep vdb
     # 

2.2.3 卷的在线扩容
------------------
未完成

2.2.4 卷的线下扩容
------------------
未完成








