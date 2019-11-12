#Horovod MNIST Example
We provide an example script `horovod_mnist_estimator.py` which is a Tornasole-enabled Horovod training script
that uses the Estimator interface of TensorFlow.

This is an example of how you can log a distributed training job with smdebug.

## Integrating Tornasole
Below we call out the changes for Tornasole in the above script and describe them

**Importing TornasoleTF**
```
import smdebug.tensorflow as smd
```
**Saving gradients**

We need to wrap our optimizer with hook.wrap_optimizer, and use this optimizer to minimize loss.
This will also enable us to access the gradients during analysis without having to identify which tensors out of the saved ones are the gradients.
```
opt = hook.wrap_optimizer(opt)
```


**Setting save interval**

You can set different save intervals for different modes.
This can be done by passing a dictionary as save_config to the hook.
This dictionary should have the mode as key and a SaveConfig object as value.
```
smd.TornasoleHook(...,
    save_config=smd.SaveConfig(save_interval=FLAGS.tornasole_frequency),
```
**Setting the right mode**

Notice the calls to `hook.set_mode` at various places in the code.
```
ts_hook.set_mode(smd.modes.TRAIN)
```

```
ts_hook.set_mode(smd.modes.EVAL)
```
**Passing the hook**

We need to pass this hook to a monitored session and use this session for running the job.
```
ts_hook = smd.TornasoleHook(...)
mnist_classifier.train(..., hooks=[...,ts_hook])
```

```
mnist_classifier.evaluate(..., hooks=[..., ts_hook])
```
## Running the example
### Environment
Ensure you are in a python environment which has tornasole_core installed. If you followed the recommended instructions of using Amazon Deep Learning AMI, then you might want to activate the tensorflow_p36 environment as follows.
```
source activate tensorflow_p36
```
### Tornasole Path
We recommend saving tornasole outputs on S3 by passing the
flag `--tornasole_path` in the format `s3://bucket_name/prefix`.
The commands below will be shown with local path however so you can
run them immediately without having to setup S3 permissions.

### Command

To run on a machine with 4 GPUs:
```
horovodrun -np 4 -H localhost:4 python horovod_mnist_estimator.py\
 --tornasole_path ~/ts_output/ts_horovod/logs\
 --steps 5000\
 --tornasole_frequency 100\
 --reductions False
 --save_all True
```

To run on 4 machines with 4 GPUs each:
```
horovodrun -np 16 -H server1:4,server2:4,server3:4,server4:4 python horovod_mnist_estimator.py\
 --tornasole_path ~/ts_output/ts_horovod/logs\
 --steps 5000\
 --tornasole_frequency 100\
 --reductions False
 --save_all True
```

### Analysis
Refer [this page](../../../rules/README.md) for more details on analysis.

### More
Please refer to [Tornasole Tensorflow page](../../README.md).