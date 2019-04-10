
1、安装rpm-build
\# yum -y install rpm-build

2、创建用户(禁用root)
useradd devops

3、切换用户
 su - devops
 
4、创建目录
echo "%_topdir %(echo $HOME)/rpmbuild" >>  ~/.rpmmacros
mkdir -p ~/rpmbuild/{BUILD,RPMS,SOURCES,SPECS,SRPMS}
##### 
    以下为目录所对应存放文件的解释：
    BUILD：源码解压以后放的目录
    RPMS：制作完成后的rpm包存放目录
    SOURCES：存放源文件，配置文件，补丁文件等放置的目录【常用】
    SPECS：存放spec文件，作为制作rpm包的文件，即：nginx.spec……【常用】
    SRPMS：src格式的rpm包目录
    BuiltRoot：虚拟安装目录，即在整个install的过程中临时安装到这个目录，把这个目录当作根来用的，所以在这个目录下的文件，才是真正的目录文件。最终，Spec文件中最后有清理阶段，这个目录中的内容将被删除
5、查看环境变量、命令
rpmbuild --showrc 显示所有的宏，以下划线开头，一个下划线：定义环境的使用情况，二个下划线：通常定义的是命令


【教程】https://fedoraproject.org/wiki/How_to_create_an_RPM_package/zh-cn

【例子】https://github.com/nmilford/rpm-etcd/blob/master/etcd.spec