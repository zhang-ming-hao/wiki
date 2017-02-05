1. 编译Python  
    ./configure --enable-shared --prefix=/usr/local/python27 -enable-unicode=ucs4  
    make  
    make install  
    cp /usr/local/python27/lib/libpython2.7.so.1.0 /usr/local/lib  
    cd /usr/local/lib  
    ln -s libpython2.7.so.1.0 libpython2.7.so  
    
1. 安装Oracle客户端  
    rpm -ivh oracle-instantclient11.2-basic-11.2.0.4.0-1.x86_64.rpm  
    rpm -ivh oracle-instantclient11.2-devel-11.2.0.4.0-1.x86_64.rpm  
    rpm -ivh oracle-instantclient11.2-sqlplus-11.2.0.4.0-1.x86_64.rpm  
    
1. 修改环境变量  
    vi /etc/bashrc  
    export ORACLE_HOME=/usr/lib/oracle/11.2/client64  
    export PATH=$ORACLE_HOME/bin: $PATH  
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH: $ORACLE_HOME/lib:/usr/local/lib  
    export TNS_ADMIN=/usr/lib/oracle  
    source /etc/bashrc  
    
1. 编译glibc  
    mkdir /opt/glibc-2.14-build  
    mkdir /opt/glibc-2.14-install  
    cd /opt/glibc-2.14-install  
    /root/oracle/glibc-2.14/configure -prefix=/opt/glibc-2.14-build/  
    make -j4  
    make install  
    
    vi /etc/bashrc  
    export LD_LIBRARY_PATH=/opt/glibc-2.14-build/lib: $LD_LIBRARY_PATH   
    
1. 安装cx_Oracle  
    rpm -ivh cx_Oracle-5.2.1-11g-py27-1.x86_64.rpm  
    cp -rf /usr/lib64/python2.7/site-packages/cx*  
    /usr/local/lib/python2.7/site-packages/  
    