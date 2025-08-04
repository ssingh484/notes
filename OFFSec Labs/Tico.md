# IP

target: 
```
192.168.54.143
```
attacker: 
```
192.168.49.54
```

# NMAP

```
PORT      STATE SERVICE
21/tcp    open  ftp
22/tcp    open  ssh
80/tcp    open  http
8080/tcp  open  http-proxy
11211/tcp open  memcache
27017/tcp open  mongod
```

```
69 seconds; Ubuntu)
27017/tcp open  mongodb    MongoDB 4.0.22
| mongodb-databases: 
|   codeName = Unauthorized
|   ok = 0.0
|   code = 13
|_  errmsg = command listDatabases requires authentication
| mongodb-info: 
|   MongoDB Build info
|     sysInfo = deprecated
|     storageEngines
|       1 = ephemeralForTest
|       0 = devnull
|       3 = wiredTiger
|       2 = mmapv1
|     javascriptEngine = mozjs
|     ok = 1.0
|     debug = false
|     gitVersion = 1741806fb46c161a1d42870f6e98f5100d196315
|     modules
|     version = 4.0.22
|     openssl
|       compiled = OpenSSL 1.1.1  11 Sep 2018
|       running = OpenSSL 1.1.1  11 Sep 2018
|     maxBsonObjectSize = 16777216
|     buildEnvironment
|       cxxflags = -Woverloaded-virtual -Wno-maybe-uninitialized -std=c++14
|       distmod = ubuntu1804
|       target_arch = x86_64
|       ccflags = -fno-omit-frame-pointer -fno-strict-aliasing -ggdb -pthread -Wall -Wsign-compare -Wno-unknown-pragmas -Winvalid-pch -Werror -O2 -Wno-unused-local-typedefs -Wno-unused-function -Wno-deprecated-declarations -Wno-unused-but-set-variable -Wno-missing-braces -fstack-protector-strong -fno-builtin-memcmp
|       linkflags = -pthread -Wl,-z,now -rdynamic -Wl,--fatal-warnings -fstack-protector-strong -fuse-ld=gold -Wl,--build-id -Wl,--hash-style=gnu -Wl,-z,noexecstack -Wl,--warn-execstack -Wl,-z,relro
|       target_os = linux
|       cxx = /opt/mongodbtoolchain/v2/bin/g++: g++ (GCC) 5.4.0
|       distarch = x86_64
|       cc = /opt/mongodbtoolchain/v2/bin/gcc: gcc (GCC) 5.4.0
|     bits = 64
|     versionArray
|       1 = 0
|       0 = 4
|       3 = 0
|       2 = 22
|     allocator = tcmalloc
|   Server status
|     codeName = Unauthorized
|     ok = 0.0
|     code = 13
|_    errmsg = command serverStatus requires authentication
| fingerprint-strings: 
|   FourOhFourRequest, GetRequest: 
|     HTTP/1.0 200 OK
|     Connection: close
|     Content-Type: text/plain
|     Content-Length: 85
|     looks like you are trying to access MongoDB over HTTP on the native driver port.
|   mongodb: 
|     errmsg
|     command serverStatus requires authentication
|     code
|     codeName
|_    Unauthorized
```

# PORT 8080

NodeBB
Found a vuln stating potentially broken access control allowing access to admin tabs
Using burpsuite to validate

Found account takeover vuln which just modifies admin password
Used it via burpsuite

Admin can access plugins, including emoji plugin which had arbitrary file upload vuln

Used it to upload newly created ssh key for the root user

ssh as root

==DONE==

