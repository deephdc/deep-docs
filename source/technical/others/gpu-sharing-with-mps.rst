.. highlight:: console

GPU sharing with MPS
====================

From [1], MPS is a runtime service designed to let multiple MPI
processes using CUDA to run concurrently on a single GPU. A CUDA program
runs in MPS mode if the MPS control daemon is running on the system.

When CUDA is first initialized in a program, the CUDA driver attempts to
connect to the MPS control daemon. If the connection attempt fails, the
program continues to run as it normally would without MPS. If however,
the connection attempt to the control daemon succeeds, the CUDA driver
then requests the daemon to start an MPS server on its behalf.

If there’s an MPS server already running, and the user id of that server
process matches that of the requesting client process, the control
daemon simply notifies the client process of it, which then proceeds to
connect to the server. If there’s no MPS server already running on the
system, the control daemon launches an MPS server with the same user id
(UID) as that of the requesting client process. If there’s an MPS server
already running, but with a different user id than that of the client
process, the control daemon requests the existing server to shutdown as
soon as all its clients are done. Once the existing server has
terminated, the control daemon launches a new server with the user id
same as that of the queued client process.

The MPS server creates the shared GPU context, manages its clients, and
issues work to the GPU on behalf of its clients. An MPS server can
support upto 16 client CUDA contexts at a time. MPS is transparent to
CUDA programs, with all the complexity of communication between the
client process, the server and the control daemon hidden within the
driver binaries.

From [2], the Volta architecture introduced new MPS capabilities.
Compared to MPS on pre-Volta GPUs, Volta MPS provides a few key
improvements: \* Volta MPS clients submit work directly to the GPU
without passing through the MPS server. \* Each Volta MPS client owns
its own GPU address space instead of sharing GPU address space with all
other MPS clients. \* Volta MPS supports limited execution resource
provisioning for Quality of Service (QoS).

How to use MPS service
----------------------

Start MPS service:

::

   # nvidia-cuda-mps-control -d

Stop MPS service:

::

   # echo quit | nvidia-cuda-mps-control

Testing environment
-------------------

-  HW: Virtual machine on IISAS-GPU cloud, flavor gpu1cpu6 (6 cores,
   24GB RAM, 1 GPU Tesla K20m)
-  SW: Ubuntu 16.04, latest nvidia driver and CUDA (nvidia driver
   version 410.48, CUDA 10.0.130)

Test 1. Test with CUDA native sample nbody, without nvidia-cuda-mps service
---------------------------------------------------------------------------

a. Single CUDA process
~~~~~~~~~~~~~~~~~~~~~~

::

   #./nbody -benchmark -numbodies=512000

   number of bodies = 512000
   512000 bodies, total time for 10 iterations: 29438.994 ms
   = 89.047 billion interactions per second
   = 1780.930 single-precision GFLOP/s at 20 flops per interaction

b. Two processes at the same time:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

   # ./nbody -benchmark -numbodies=512000

   512000 bodies, total time for 10 iterations: 52418.652 ms
   = 50.010 billion interactions per second
   = 1000.194 single-precision GFLOP/s at 20 flops per interaction

Performance of each process reduced to about 1/2 due to parallel
execution (overall GPU performance is the same). nvidia-smi shows both
processes using GPU at the same time. No GPU conflicts detected.

Test 2. Test with CUDA native sample nbody, with nvidia-cuda-mps service
------------------------------------------------------------------------

.. _a.-single-cuda-process-1:

a. Single CUDA process:
~~~~~~~~~~~~~~~~~~~~~~~

Same performance as without MPS server.

b. Two processes with different user IDs:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The second process is blocked (waiting without termination), it starts
computation only the first process is terminated. Performance is the
same as without MPS server.

In both case, nvidia-smi indicates nvidia-cuda-mps-server is using GPU,
not the nbody process.

c. Two processes with the same user ID:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Both processes will run in parallel. Performance will be evenly divided
between processes, like without MPS service (Test 1b).

Test 2 has been repeated with nbody commands placed inside Docker
containers instead of baremetal, the same behavior. Note that Docker set
user ID at root by default.

Test 3. Test with Docker using mariojmdavid/tensorflow-1.5.0-gpu image, without nvidia-cuda-mps service
-------------------------------------------------------------------------------------------------------

Command used in the test:

::

   # sudo docker run --runtime=nvidia --rm -ti mariojmdavid/tensorflow-1.5.0-gpu /home/tf-benchmarks/run-bench.sh all

a. Single container:
~~~~~~~~~~~~~~~~~~~~

all tests passed.

b. Two containers in parallel:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

the second container shows different error messages according to the
running benchmarks. Some error message is rather clear like that

::

   2018-10-03 13:58:33.135064: W tensorflow/core/framework/op_kernel.cc:1198] Resource exhausted: OOM when allocating tensor with shape[1000] and type float on /job:localhost/replica:0/task:0/device:GPU:0 by allocator GPU_0_bfc

Some error messages are rather internal

::

   2018-10-03 13:51:00.160626: W tensorflow/stream_executor/stream.cc:1901] attempting to perform BLAS operation using StreamExecutor without BLAS support
   Traceback (most recent call last):
     File "/home/tf-benchmarks/benchmark_alexnet.py", line 221, in <module>
       tf.app.run()
     File "/usr/local/lib/python2.7/dist-packages/tensorflow/python/platform/app.py", line 124, in run
       _sys.exit(main(argv))
     File "/home/tf-benchmarks/benchmark_alexnet.py", line 217, in main
       run_benchmark()
     File "/home/tf-benchmarks/benchmark_alexnet.py", line 210, in run_benchmark
       timing_entries.append(time_tensorflow_run(sess, grad, "Forward-backward"))
     File "/home/tf-benchmarks/benchmark_alexnet.py", line 136, in time_tensorflow_run
       _ = session.run(target_op)
     File "/usr/local/lib/python2.7/dist-packages/tensorflow/python/client/session.py", line 895, in run
       run_metadata_ptr)
     File "/usr/local/lib/python2.7/dist-packages/tensorflow/python/client/session.py", line 1128, in _run
       feed_dict_tensor, options, run_metadata)
     File "/usr/local/lib/python2.7/dist-packages/tensorflow/python/client/session.py", line 1344, in _do_run
       options, run_metadata)
     File "/usr/local/lib/python2.7/dist-packages/tensorflow/python/client/session.py", line 1363, in _do_call
       raise type(e)(node_def, op, message)
   tensorflow.python.framework.errors_impl.InternalError: Blas GEMM launch failed : a.shape=(128, 9216), b.shape=(9216, 4096), m=128, n=4096, k=9216
            [[Node: affine1/affine1/MatMul = MatMul[T=DT_FLOAT, transpose_a=false, transpose_b=false, _device="/job:localhost/replica:0/task:0/device:GPU:0"](Reshape, affine1/weights/read)]]
   Caused by op u'affine1/affine1/MatMul', defined at:
     File "/home/tf-benchmarks/benchmark_alexnet.py", line 221, in <module>
       tf.app.run()
     File "/usr/local/lib/python2.7/dist-packages/tensorflow/python/platform/app.py", line 124, in run
       _sys.exit(main(argv))
     File "/home/tf-benchmarks/benchmark_alexnet.py", line 217, in main
       run_benchmark()
     File "/home/tf-benchmarks/benchmark_alexnet.py", line 181, in run_benchmark
       last_layer = inference(images)
     File "/home/tf-benchmarks/benchmark_alexnet.py", line 120, in inference
       affn1 = _affine(resh1, 256 * 6 * 6, 4096)
     File "/home/tf-benchmarks/benchmark_alexnet.py", line 76, in _affine
       affine1 = tf.nn.relu_layer(inpOp, kernel, biases, name=name)
     File "/usr/local/lib/python2.7/dist-packages/tensorflow/python/ops/nn_impl.py", line 272, in relu_layer
       xw_plus_b = nn_ops.bias_add(math_ops.matmul(x, weights), biases)
     File "/usr/local/lib/python2.7/dist-packages/tensorflow/python/ops/math_ops.py", line 2022, in matmul
       a, b, transpose_a=transpose_a, transpose_b=transpose_b, name=name)
     File "/usr/local/lib/python2.7/dist-packages/tensorflow/python/ops/gen_math_ops.py", line 2516, in _mat_mul
       name=name)
     File "/usr/local/lib/python2.7/dist-packages/tensorflow/python/framework/op_def_library.py", line 787, in _apply_op_helper
       op_def=op_def)
     File "/usr/local/lib/python2.7/dist-packages/tensorflow/python/framework/ops.py", line 3160, in create_op
       op_def=op_def)
     File "/usr/local/lib/python2.7/dist-packages/tensorflow/python/framework/ops.py", line 1625, in __init__
       self._traceback = self._graph._extract_stack()  # pylint: disable=protected-access
   InternalError (see above for traceback): Blas GEMM launch failed : a.shape=(128, 9216), b.shape=(9216, 4096), m=128, n=4096, k=9216
            [[Node: affine1/affine1/MatMul = MatMul[T=DT_FLOAT, transpose_a=false, transpose_b=false, _device="/job:localhost/replica:0/task:0/device:GPU:0"](Reshape, affine1/weights/read)]]

Test 4. Test with Docker using mariojmdavid/tensorflow-1.5.0-gpu image, with nvidia-cuda-mps service
----------------------------------------------------------------------------------------------------

Option “–ipc=host” required for connecting MPS service (full command
“sudo docker run –runtime=nvidia –rm –ipc=host -ti
mariojmdavid/tensorflow-1.5.0-gpu /home/tf-benchmarks/run-bench.sh
all”), see https://github.com/NVIDIA/nvidia-docker/issues/419

Some tests passed but not all

::

   /home/tf-benchmarks/run-bench.sh: line 78:   137 Aborted                 (core dumped) python ${TFTest[$i]}

According to [3], only Tensorflow with version 1.6 and higher can
support MPS.

Test 5. Test with Docker using vykozlov/tf-benchmarks:181004-tf180-gpu image, without and with nvidia-cuda-mps service
----------------------------------------------------------------------------------------------------------------------

Tensorflow 1.8.0, GPU version, python 2, command:

::

   sudo docker run --ipc=host --runtime=nvidia --rm -ti  vykozlov/tf-benchmarks:181004-tf180-gpu  ./tf-benchmarks.sh all

a. without MPS service
~~~~~~~~~~~~~~~~~~~~~~

all tests passed.

b. with MPS service
~~~~~~~~~~~~~~~~~~~

Only first part (forward) of each test passed, then the execution
terminated (core dumped).

Identified reasons why Tensoflow does not work correctly with MPS
-----------------------------------------------------------------

The reasons have been discussed in [3]:

-  stream callbacks are not supported on pre-Volta MPS clients. Calling
   any stream callback APIs will return an error. (from MPS official
   document [4])
-  But CUDA streams are used everywhere in Tensorflow

So Tensorflow will not work with MPS on old (pre-Volta) GPU.

Final remarks:
--------------

-  Without MPS service, native CUDA samples can be executed in parallel
   and the GPU performance is divided among processes
-  With MPS service, CUDA executions with different user IDs are
   serialized, one needs to wait until other finishes.
-  CUDA processes with the same user ID can be executed in parallel.
-  Tensorflow will not work with MPS on old (pre-Volta) GPU.
-  Need to test on newer GPU cards (Volta)

References
----------

1. http://manpages.ubuntu.com/manpages/xenial/man1/alt-nvidia-340-cuda-mps-control.1.html
2. https://docs.nvidia.com/deploy/mps/index.html
3. https://github.com/tensorflow/tensorflow/issues/9080
4. https://docs.nvidia.com/deploy/mps/index.html
