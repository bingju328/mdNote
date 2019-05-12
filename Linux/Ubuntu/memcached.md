安装memcached
先安装lebevent库
apt-get install libevent-dev
下载安装memcached
wget https://memcached.org/latest
mv latest memcached-1.5.12.tar.gz
tar -zxf memcached-1.5.12.tar.gz
cd memcached-1.5.12
./configure --prefix=/usr/local/memcached
make && sudo make install

使用memcached
•启动服务器
/usr/local/memcached/bin/memcached -m 64 -p 11211 -u root -vvv
