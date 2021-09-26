# 背景

笔者在使用 [pypi.org](https://pypi.org/project/py3Fdfs/#modal-close)提供的 FastDFS Python客户端：py3Fdfs-2.2.0.tar.gz时，发现：

  下载文件时偶尔报错 **socket timeout**，随即文件下载失败

因苦于 2.2.0已经是官网最新版本，无奈只能自己动手修复问题



# 说明

通过调试发现下载文件失败的原因在下面这段代码：

```python
def tcp_recv_file(conn, local_filename, file_size, buffer_size=1024):
    '''
    Receive file from server, fragmented it while receiving and write to disk.
    arguments:
    @conn: connection
    @local_filename: string
    @file_size: int, remote file size
    @buffer_size: int, receive buffer size
    @Return int: file size if success else raise ConnectionError.
    '''
    total_file_size = 0
    flush_size = 0
    remain_bytes = file_size
    with open(local_filename, 'wb+') as f:
        while remain_bytes > 0:
            try:
                if remain_bytes >= buffer_size:
                    file_buffer, recv_size = tcp_recv_response(conn, buffer_size, buffer_size)
                else:
                    file_buffer, recv_size = tcp_recv_response(conn, remain_bytes, buffer_size)
                f.write(file_buffer)
                remain_bytes -= buffer_size
                total_file_size += recv_size
                flush_size += recv_size
                if flush_size >= 4096:
                    f.flush()
                    flush_size = 0
            except ConnectionError as e:
                raise ConnectionError('[-] Error: while downloading file(%s).' % e.args)
            except IOError as e:
                raise DataError('[-] Error: while writting local file(%s).' % e.args)
    return total_file_size
```

上面这段代码的问题在于，计算 **剩余的文件大小错误**，如下：

```python
remain_bytes -= buffer_size
```

修改好的代码见 GitHub: https://github.com/Daphnis-z/py3fdfs-pypi.org



# 使用

本次仅仅修改了包内的 fdfs_client/storage_client.py文件，因此有两种使用方式：

1. 已经安装了官网提供的 py3Fdfs-2.2.0.tar.gz的，可以直接替换 storage_client.py，具体路径是

   [Python安装路径]/site-packages/fdfs_client

2. 可以直接从 GitHub下载源码进行安装

