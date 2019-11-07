[CUDA Toolkit 10.0 Archive](https://developer.nvidia.com/cuda-10.0-download-archive)

在上面的链接里面下载 runfile `cuda_10.0.130_410.48_linux.run`，下载好之后
```
sudo sh cuda_10.0.130_410.48_linux.run
```

在尝试 `import tensorflow` 的时候遇到以下报错

```shell
ImportError: libcublas.so.10.0: cannot open shared object file: No such file or directory
```

执行以下指令查看
```shell
ls /usr/lib/x86_64-linux-gnu | grep libcublas
ls /usr/local/cuda/lib64 | grep libcublas
```

发现是因为没有把 CUDA 的目录加到环境里面，因此 Tensorflow 找不到。执行以下指令，
```shell
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda-10.0/lib64
```

再 `import tensorflow` 就可以了。**但是好像每次开启新的回话就需要重新执行，我暂时还不知道解决办法**。