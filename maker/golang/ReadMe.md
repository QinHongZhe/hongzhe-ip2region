# ip2region xdb golang 生成实现


# 程序编译

通过如下方式编译得到 xdb_maker 可执行程序:
```
# 切换到golang maker 根目录
make
```
编译成功后会在当前目录生成一个 xdb_maker 的可执行文件


# `xdb` 数据生成

通过 `xdb_maker gen` 命令生成 ip2region.xdb 二进制文件:
```
➜  golang git:(v2.0_xdb) ✗ ./xdb_maker gen
./xdb_maker gen [command options]
options:
 --src string    source ip text file path
 --dst string    destination binary xdb file path
```

例如，使用默认的 data/ip.merge.txt 作为源数据，生成一个 ip2region.xdb 到当前目录：
```bash
➜  golang git:(v2.0_xdb) ✗ ./xdb_maker gen --src=../../data/ip.merge.txt --dst=./ip2region.xdb
# 会看到一堆输出，最终会看到类似如下输出表示运行结束
...
2022/06/16 16:38:48 maker.go:317: write done, with 13804 data blocks and (683591, 720221) index blocks
2022/06/16 16:38:48 main.go:89: Done, elapsed: 33.615278847s
```


# `xdb` 数据查询

通过 `xdb_maker search` 命令来测试查询输入的 ip：
```
➜  golang git:(v2.0_xdb) ✗ ./xdb_maker search
./xdb_maker search [command options]
options:
 --db string    ip2region binary xdb file path
```

例如，使用自带的 xdb 文件来运行查询测试：
```bash
➜  golang git:(v2.0_xdb) ✗ ./xdb_maker search --db=../../data/ip2region.xdb
ip2region xdb search test program, commands:
loadIndex : load the vector index for search speedup.
clearIndex: clear the vector index.
quit      : exit the test program
ip2region>> 103.192.227.215
{region:中国|0|香港|0|0, iocount:8, took:87.151µs}
ip2region>> loadIndex
vector index cached
ip2region>> 39.114.2.16
{region:韩国|0|0|0|SK宽带, iocount:2, took:50.005µs}
ip2region>> 120.24.130.96
{region:中国|0|广东省|深圳市|阿里云, iocount:2, took:32.805µs}
ip2region>> 
```

# bench 测试

如果你自主生成了 `xdb` 文件，请确保运行如下的 `xdb_maker bench` 命令来确保生成的的 `xdb` 文件的正确性：
```
➜  golang git:(v2.0_xdb) ✗ ./xdb_maker bench
./xdb_maker bench [command options]
options:
 --db string            ip2region binary xdb file path
 --src string           source ip text file path
 --ignore-error bool    keep going if bench failed
```

例如：使用 data/ip.merge.txt 源文件来 bench 测试 data/ip2region.xdb 这个 xdb 文件：
```bash
➜  golang git:(v2.0_xdb) ✗ ./xdb_maker bench --db=../../data/ip2region.xdb --src=../../data/ip.merge.txt
# 会看到一堆输出，看到类似如下的数据表示 bench 测试通过了，否则就会报错
...
try to bench segment: `224.0.0.0|255.255.255.255|0|0|0|内网IP|内网IP`
|-try to bench ip '224.0.0.0' ...  --[Ok]
|-try to bench ip '231.255.255.255' ...  --[Ok]
|-try to bench ip '239.255.255.255' ...  --[Ok]
|-try to bench ip '247.255.255.255' ...  --[Ok]
|-try to bench ip '255.255.255.255' ...  --[Ok]
Bench finished, {count: 3417955, failed: 0, took: 52.200116397s}
```
*请注意 bench 测试使用的 `src` 文件需要是对应的生成 ip2region.xdb 的源文件相同*。
如果运行过程中有错误会立马停止运行，也可以执行 --ignore-error=true 参数来忽略错误，在最后看 failed 的统计结果。
