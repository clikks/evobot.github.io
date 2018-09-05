---
title: Memcached操作与应用
author: Evobot
categories: NoSQL
tags:
  - Memcached
abbrlink: 32089b8c
date: 2018-08-15 23:19:30
image:
---

1. memcached命令行
2. memcached数据导出和导入
3. php连接memcached
4. memcached中存储sessions

<!--more-->

---

# Memcached命令行

## Memcached命令行介绍

- memcached命令行操作依赖于Telnet，登录memcached的命令行，使用`telnet 127.0.0.1 11211`登录memcached：

  ```bash
  $ telnet 127.0.0.1 11211
  Trying 127.0.0.1...
  Connected to 127.0.0.1.
  Escape character is '^]'.

  ```

- 在Telnet中，不能随意输入，memcached存储一条数据，使用`set`关键字，如`set key2 0 30 2`，其中最末尾的2表示k-v的value是两位数值或字符，30表示过期时间为30秒，0则是flag，具体flag的作用，可以参考下一节的解释：

  ```bash
  set key2 0 30 2
  12
  STORED

  get key2
  VALUE key2 0 2
  12
  END

  set key1 0 20 3
  abc
  STORED

  get key1
  VALUE key1 0 3
  abc
  END

  ```

## Memcached语法规则

- 语法规则如下：

  ```bash
  <command name> <key> <flags> <exptime> <bytes>\r\n <data block>\r\n
  ```

  > `\r\n`在windows下为Enter键

- 其中`<command name>`可以是`set`、`add`、`replace`和`delete`等：

  - `set`表示按照相应的`<key>`存储该数据，没有的时候增加，有的时候会覆盖；
  - `add`表示按照相应的`<key>`添加该数据，如果该`<key>`已存在则会操作失败；
  - `replace`表示按照相应的`<key>`替换数据，但如果该`<key>`不存在则操作失败；

- `<key>`则是客户端需要保存的数据的键名(key)；

- `<flag>`则是一个16位的无符号的整数（以十进制方式表示），该标志将和需要存储的数据一起存储，并在客户端get数据时返回，客户端可以将此标志作特殊用途，此标志对服务器来说是不透明的；

- `<exptime>`表示过期时间，若为0则存储的数据永不过期（但可以被服务器算法，LRU等替换），如果非0*（unix时间或距离此时的秒数），当过期后，服务器可以保证用户得不到该数据（以服务器时间为准）；

- `<bytes>`表示需要存储的字节数，当用户希望存储空数据时，`<bytes>`可以为0；

- `<data block>`则表示需要存储的内容，输入完成后，需要客户端加上`\r\n`或直接点击Enter键作为结束标志。

## 示例

```bash
set key3 1 100 4
abcd
STORED

replace key3 1 0 3  
123
STORED

get key3
VALUE key3 1 3
123
END

delete key3
DELETED

get key3
END

```

# Memcached数据导入和导出

## 导出

- 如果需要重启memcached的服务时，需要将数据进行导出备份，导出memcached的数据，使用`memcached-tool`命令，命令格式为`memcached-tool [ip:port] dump`：

  ```bash
  $ memcached-tool 127.0.0.1:11211 dump
  Dumping memcache contents
    Number of buckets: 1
    Number of items  : 3
  Dumping bucket 1 - 3 total items
  add name 1 1534346629 6
  evobot
  add age 1 1534346629 2
  24
  add sex 1 1534346629 4
  male

  ```

- 如果需要导出到文件中，只需要将输出重定向到指定文件即可：

  ```bash
  $ memcached-tool 127.0.0.1:11211 dump > memcached_backup
  Dumping memcache contents
    Number of buckets: 1
    Number of items  : 3
  Dumping bucket 1 - 3 total items

  $ cat memcached_backup 
  add name 1 1534346629 6
  evobot
  add age 1 1534346629 2
  24
  add sex 1 1534346629 4
  male

  ```

## 导入

- 将数据导入回memcached，则使用`nc`命令，首先重启memcached服务，因为memcached的数据是存储在内存中的，所以重启后，存储的数据都会清空，然后使用`nc [ip] [port] < backup_file`命令即可导入：

  ```bash
  $ systemctl restart memcached

  $ memcached-tool 127.0.0.1:11211 stats
  #127.0.0.1:11211   Field       Value
           accepting_conns           1
                 auth_cmds           0
               auth_errors           0
                     bytes           0
                bytes_read          33
             bytes_written          54
                cas_badval           0
                  cas_hits           0
                cas_misses           0
                 cmd_flush           0
                   cmd_get           0
                   cmd_set           0
                 cmd_touch           0
               conn_yields           0
     connection_structures          11
          curr_connections          10
                curr_items           0
                 decr_hits           0
               decr_misses           0
               delete_hits           0
             delete_misses           0
         evicted_unfetched           0
                 evictions           0
         expired_unfetched           0
                  get_hits           0
                get_misses           0
                hash_bytes      524288
         hash_is_expanding           0
          hash_power_level          16
                 incr_hits           0
               incr_misses           0
                  libevent 2.0.21-stable
            limit_maxbytes    67108864
       listen_disabled_num           0
                       pid        2328
              pointer_size          64
                 reclaimed           0
              reserved_fds          20
             rusage_system    0.003695
               rusage_user    0.002463
                   threads           4
                      time  1534349889
         total_connections          12
               total_items           0
                touch_hits           0
              touch_misses           0
                    uptime          23
                   version      1.4.15
  $ nc 127.0.0.1 11211 < memcached_backup 
  STORED
  STORED
  STORED
  $ nc 127.0.0.1 11211 
  get name
  END
  get age
  END
  get sex
  END

  ```

- 在导入后会发现memcached中无法get到导入的值，这是因为导出的文件中，记录了数据的时间戳，当服务器时间大于这个时间戳的时候，数据就会过期，所以导入的数据已经过期，无法get；

- 为了防止这种情况，我们在导入前对备份文件中的时间戳进行修改为比当前服务器时间大即可：

  ```bash
  $ cat memcached_backup
  add name 1 1534346629 6
  evobot
  add age 1 1534346629 2
  24
  add sex 1 1534346629 4
  male 

  # 获取一个小时后的时间戳
  $ date -d "+1 hours" +%s
  1534353769

  # 将备份文件中的时间戳修改
  $ vi memcached_backup
  $ cat memcached_backup 
  add name 1 1534353769 6
  evobot
  add age 1 1534346629 2
  24
  add sex 1 100 4
  male

  # 重新导入数据
  $ systemctl restart memcached
  $ nc 127.0.0.1 11211 < memcached_backup 
  STORED
  STORED
  STORED

  $ nc 127.0.0.1 11211
  get name
  VALUE name 1 6
  evobot
  END
  get age
  END
  get sex
  VALUE sex 1 4
  male
  END

  ```

- 可以看到，修改了时间戳的数据，在导入后可以查询到，而没有修改时间戳的数据，如age，则已经过期。

# php连接memcached

- 对于站点来说，例如PHP的站点，要使用memcached，则需要一个对应的模块让站点客户端能够连接到Memcached，在php-fpm中，使用`php-fpm -m`可以查看到php支持的所以模块，而php能够支持memcached，也需要安装对应的模块；

## php7以下版本

- 首先下载memcached的php模块包，[memcache模块下载](http://pecl.php.net/get/memcache-2.2.7.tgz)，解压后进入目录；

- 执行`/usr/local/php-fpm/bin/phpize`，然后再进行编译：

  ```bash
  $ /usr/local/php-fpm/bin/phpize 
  Configuring for:
  PHP Api Version:         20160303
  Zend Module Api No:      20160303
  Zend Extension Api No:   320160303

  $ ./configure --with-php-config=/usr/local/php-fpm/bin/php-config 

  $ make && make install
  ```

- 完成后会在php的扩展模块的目录下生成`memcache.so`的文件，接着编辑`php.ini`文件，添加`extension=memcache.so`保存退出即可；

- 使用`php-fpm -m`查看模块是否生效。

## php7版本

- 首先安装依赖包`autoconf`、`gcc-c++`;

- 下载[libmemcache](https://launchpad.net/libmemcached/1.0/1.0.18/+download/libmemcached-1.0.18.tar.gz)软件包，解压并进行编译安装：

  ```bash
  $ ./configure -prefix=/usr/local/libmemcached --with-memcached --enable-memaslap --enable-sasl

  $ make && make install
  ```

- 然后到github下载支持php7的[memcache](https://github.com/php-memcached-dev/php-memcached)软件包，解压编译：

  ```bash
  [php-memcached-master]# /usr/local/php-fpm/bin/phpize 
  Configuring for:
  PHP Api Version:         20160303
  Zend Module Api No:      20160303
  Zend Extension Api No:   320160303

  $ ./configure --with-php-config=/usr/local/php-fpm/bin/php-config --with-libmemcached-dir=/usr/local/libmemcached/

  $ make && make install
  Installing shared extensions:     /usr/local/php-fpm/lib/php/extensions/no-debug-non-zts-20160303/

  $ ls /usr/local/php-fpm/lib/php/extensions/no-debug-non-zts-20160303/
  memcached.so  opcache.a  opcache.so

  ```

- 最后编辑php.ini，添加模块即可。

## 测试php连接memcache

- 首先下载测试文件：

  ```bash
  curl www.apelearn.com/study_v2/.memcache.txt > 1.php 2>/dev/null
  ```

- 然后使用`/usr/local/php-fpm/bin/php 1.php`命令测试文件，输出如下,则表示模块正常：

  ```bash
  $ php 1.php
  Get key1 value: This is first value<br>Get key1 value: This is replace value<br>Get key2 value: Array
  (
      [0] => aaa
      [1] => bbb
      [2] => ccc
      [3] => ddd
  )
  <br>Get key1 value: <br>Get key2 value: <br>

  ```

- 对于php7，由于使用的memcached的模块不是memcache，是memcached，所以测试文件应该写成如下内容：

  ```php
  <?php
  //连接Memcache Memcache
  $mem = new Memcached;
  $mem->addServer("localhost", 11211);
  //保存数据
  $mem->set('key1', 'This is first value',time()+60);
  $val = $mem->get('key1');
  echo "Get key1 value: " . $val ."<br>";
  //替换数据
  $mem->replace('key1', 'This is replace value', time()+60);
  $val = $mem->get('key1');
  echo "Get key1 value: " . $val . "<br>";
  //保存数组数据
  $arr = array('aaa', 'bbb', 'ccc', 'ddd');
  $mem->set('key2', $arr, time()+60);
  $val2 = $mem->get('key2');
  echo "Get key2 value: ";
  print_r($val2);
  echo "<br>";
  //删除数据
  $mem->delete('key1');
  $val = $mem->get('key1');
  echo "Get key1 value: " . $val . "<br>";
  //清除所有数据
  $mem->flush();
  $val2 = $mem->get('key2');
  echo "Get key2 value: ";
  print_r($val2);
  echo "<br>";
  //关闭连接
  $mem->quit();
  ?>
  ```



# Memcached中存储session

## PHP默认session

- Memcached中存储session是用在LAMP/LNMP环境中的，是用来解决多个web服务器集群，用户访问时，由于session分散，导致用户出现频繁需要登录的情况；

- 在php.ini配置文件中，有`session.save_handler = files`配置项，该配置项表示session存储在文件中，默认存储位置为`/tmp`目录；

- 创建session测试php脚本文件`mem_se.php`，放到nginx默认虚拟主机目录下，文件内容如下：

  ```php
  <?php 
  session_start(); 
  if (!isset($_SESSION['TEST'])) { 
  $_SESSION['TEST'] = time(); 
  } 
  $_SESSION['TEST3'] = time(); 
  print $_SESSION['TEST']; 
  print "<br><br>"; 
  print $_SESSION['TEST3']; 
  print "<br><br>"; 
  print session_id(); 
  ?> 
  ```

- 然后使用`curl localhost/mem_se.php`访问文件，查看tmp目录下是否生成session：

  ```bash
  $ curl localhost/mem_se.php
  1534436654<br><br>1534436654<br><br>89drjq88pipbu57omrufrhd8vl 

  $ ls /tmp/
  php_error.log
  php-fcgi-dedecms.sock
  php-fcgi-discuz.sock
  sess_89drjq88pipbu57omrufrhd8vl

  $ cat /tmp/sess_89drjq88pipbu57omrufrhd8vl 
  TEST|i:1534436654;TEST3|i:1534436654;
  ```

- session文件名是一串随机字符串。

## memcached存储session

- 修改`php.ini`文件，注释`session.save_handler = files`配置项；

- 然后增加以下两行配置，该配置对应的是php7以下版本：

  ```bash
  session.save_handler = memcache
  session.save_path = "tcp://192.168.67.128:11211"
  ```

- 如果为php7以上版本，使用的模块为memcached，则配置如下：

  ```bash
  session.save_handler = memcached
  session.save_path = "192.168.67.128:11211"
  ```

- 如果上面的配置无法生效，可以采用下面两种方式进行配置：

  ```bash
  // 在httpd.conf中对应的虚拟主机中添加
  php_value session.save_handler "memcache"
  php_value session.save_path "tcp://192.168.67.128:11211"
  或者
  // 在php-fpm.conf对应的pool中添加
  php_value[session.save_handler] = memcache
  php_value[session.save_path] = "tcp://192.168.67.128:11211"
  ```

- 配置完成后重启php-fpm服务，然后删除之前/tmp目录下生成的session，再次访问mem_se.php文件：

  ```、bash
  $ curl localhost/mem_se.php
  1534437560<br><br>1534437560<br><br>2rnnmgkjre11aim0iicg1aas26 

  # 查看tmp目录已经没有生成session文件
  $ ls /tmp/
  hsperfdata_root
  pear
  php_error.log
  php-fcgi-dedecms.sock
  php-fcgi-discuz.sock

  ```

- 将memcached的数据dump出来，查看session是否保存成功：

  ```bash
  $ memcached-tool 127.0.0.1:11211 dump > session.txt
  Dumping memcache contents
    Number of buckets: 1
    Number of items  : 1
  Dumping bucket 3 - 1 total items

  $ cat session.txt 
  add memc.sess.key.2rnnmgkjre11aim0iicg1aas26 0 1534439000 37
  TEST|i:1534437560;TEST3|i:1534437560;

  ```

- 或者使用telnet登录memcached的命令行进行get：

  ```bash
  $ curl localhost/mem_se.php
  1534437881<br><br>1534437881<br><br>h7lajo2p1ftepnhq701rpg1d6d 

  $ telnet 127.0.0.1 11211
  Trying 127.0.0.1...
  Connected to 127.0.0.1.
  Escape character is '^]'.
  get memc.sess.key.h7lajo2p1ftepnhq701rpg1d6d
  VALUE memc.sess.key.h7lajo2p1ftepnhq701rpg1d6d 0 37
  TEST|i:1534437881;TEST3|i:1534437881;
  END
  ```

---

