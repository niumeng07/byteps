# BytePS Environment Variables

## Required for workers

For each worker machine, you must specify the following variables:

```
export DMLC_ROLE=worker
export DMLC_PS_ROOT_URI=a.b.c.d
export DMLC_PS_ROOT_PORT=p
export DMLC_WORKER_ID=x
export DMLC_NUM_WORKER=y
```

If you have ```NVIDIA_VISIBLE_DEVICES``` set, you can run ```launcher/launcher.py YOUR_COMMAND``` to start your job. 

Alternatively, you can start the job on each GPU after specifying:

```
export BYTEPS_LOCAL_RANK=r
export BYTEPS_LOCAL_SIZE=s
```

If you have RDMA network available, you should set:

```
DMLC_ENABLE_RDMA=1
```

Otherwise, set it to 0.

## Required for servers and scheduler

BytePS uses the same environment variables as MXNet for server and scheduler:
https://mxnet.incubator.apache.org/versions/master/faq/distributed_training.html#manually-launching-jobs

In short, you should configure the same DMLC_* variables as the worker, except that DMLC_ROLE should be either server or scheduler.

Also, set DMLC_ENABLE_RDMA accordingly as workers.

## BytePS debug

If you are using launcher.py, you can enable gdb and get the backtrace (if the program terminates abnormally) by setting:

```
export BYTEPS_ENABLE_GDB=1
```

You can let BytePS print verbose logs by setting:

```
export BYTEPS_LOG_LEVEL=INFO
```

You can also let BytePS print values of a given tensor (specified by a key in integer) during different stages and iterations:

```
export BYTEPS_DEBUG_SAMPLE_TENSOR=xxxx
```

By default, if there is only one worker machine, BytePS won't connect to servers or schedulers because it is not needed. However, for debug purposes, you can force the worker to push and pull:

```
export BYTEPS_FORCE_DISTRIBUTED=1
```

The logging in the ps-lite middleware and on the server side is controlled by PS_VERBOSE. You can set the following to enable verbose output:

```
export PS_VERBOSE=2
```

## Performance tuning

There are several knobs that may impact the performance of BytePS. If you are not sure what they mean, you can leave them unmodified, i.e., by not setting them.

The most important one is the number of GPUs per PCIe switches. You should configure it according to your hardware. However, if you really do not know the hardware setup, you can leave it unmodified. BytePS should work as well as Horovod in that case, although a correct configuration may give you better performance than Horovod.

```
export BYTEPS_PCIE_SWITCH_SIZE=x
```

You can also configure the tensor partition size. A smaller size improves BytePS pipelining, but may have higher other overhead like NCCL coordination, ZMQ message headers, etc. The default and recommended value is 4096000 (in bytes).

```
export BYTEPS_PARTITION_BYTES=y
```

The rest do not impact the performance much. However, you can still experiment them if you have time. 

You can increase the number of concurrent NCCL streams used in local merging. However, this may lead to occasional hanging problem due to NCCL implementation.

```
export BYTEPS_NCCL_NUM_RINGS=z
```

BytePS uses group NCCL calls to reduce NCCL invoking overhead. You can try to increase the group sizes:

```
export BYTEPS_NCCL_GROUP_SIZE=w
```

Servers can also be the performance bottleneck, e.g., when there are only one server but multiple workers. 
You can try to increase the number of push threads on the servers (default is 1):
 
```
export SERVER_PUSH_NTHREADS=v
```

Increasing the number of engine CPU threads may also improves server performance:

```
export MXNET_CPU_WORKER_NTHREADS=p
```