安装依赖项
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel

# lua_jit

wget http://luajit.org/download/LuaJIT-2.0.5.tar.gz
tar -xvf LuaJIT-2.0.5.tar.gz
cd LuaJIT-2.0.5
make install
