# clusterdate-trace-gpu2020
 
# 数据表和数据列说明

- **pai_job_table**: 作业加载信息.
- **pai_task_table**: 任务加载信息.
- **pai_instance_table**: 实例加载信息.
- **pai_sensor_table**: 实例资源监控器信息.
- **pai_group_tag_table**: instance semantic information（实例语义信息）.
- **pai_machine_spec**: machine specification（机器规格）.
- **pai_machine_metric**: machine resource metrics with respect to the instance（与实例相关的机器资源指标）.



## pai_job_table
**job launch information.**

| Columns    | Example Entry                                                |
|:-----------|:-------------------------------------------------------------|
| job_name   | 4b3f04b66a525d2df903eb16                                     |
| inst_id    | 8cb3bec23d14dbde320b6613452e768cbbf35b8bd64ee28fcceb77d3c47d |
| user       | 58540f191766                                                 |
| status     | Terminated                                                   |
| start_time | 4550879.0                                                    |
| end_time   | 4551416.0                                                    |

- `job_name`: 用户提交的作业名. 已经被脱敏处理 (similar to `user_name`, `worker_name`, `inst_name`, etc. below).
- `inst_id`: instance id assigned, to be joined with `inst_id` in [pai_sensor_table](#pai_sensor_table) and [pai_group_tag_table](#pai_group_tag_table).
- `user`: user name.
- `status`: job status, including 'Running', 'Terminated', 'Failed', 'Waiting'; only 'Terminated' tasks are successful.
- `start_time`: timestamp of job submission time.
- `end_time`: timestamp of job completion time.

**Note of `time`**: Both `start_time` and `end_time` are in *seconds* and have been deducted by a constant number for desensitization. Still, if translated to Unix time in UTC+8 timezone ("Asia/Shanghai"), they will have the same time of the day (e.g., "08:59:59 UTC+8") and the same day of the week (e.g., "Sunday") as the original traces, while having fake dates, months, and years.



## pai_task_table

**task launch information.**

| Columns    | Example Entry            |
|:-----------|:-------------------------|
| job_name   | 4f057009a5d481acec67b088 |
| task_name  | tensorflow               |
| inst_num   | 1.0                      |
| status     | Terminated               |
| start_time | 1739162.0                |
| end_time   | 1739200.0                |
| plan_cpu   | 600.0                    |
| plan_mem   | 29.296875                |
| plan_gpu   | 50.0                     |
| gpu_type   | MISC                     |

- `job_name`: job name; same as the entry in [pai_job_table](#pai_job_table).
- `task_name`: job和task是1:1或1:N的。**most jobs have only one task, but some may launch multiple tasks of different names** (roles), e.g., `ps`, `worker`, `evaluator`.
- `inst_num`: number of instances launched by the task.
- `status`: task status.
- `start_time`: task的启动时间在job的start_time之后，这之间存在任务调度的延迟。timestamp of task launch time. The gap between `job.start_time` and the earliest `task.start_time` in the job implies its wait time before launching (scheduling latency).
- `end_time`: timestamp of task completion time.
- `plan_cpu`: number of CPU cores requested in percentage (i.e., 600.0 is 6 vCPU cores) .
- `plan_mem`: GB of main memory requested.
- `plan_gpu`: number of GPUs requested in percentage (i.e., 50.0 is 50% GPU).
- `gpu_type`: type of GPUs assigned to this task. `MISC` is short for "miscellaneous", indicating GPUs of older generations, e.g., NVIDIA Tesla K40m, K80, M60.



## pai_instance_table

**instance launch information.**

| Columns     | Example Entry                                                |
|:------------|:-------------------------------------------------------------|
| job_name    | af724763f4f5d0beef445849                                     |
| task_name   | worker                                                       |
| inst_name   | 0d39aa867a79c16eff67daa8f6248f09af8346b177c9e3e23645c48354a8 |
| worker_name | 54dbcd2db287841c03d0639b2a93e783a090ea085348f8cdb8e603d8b96f |
| inst_id     | e387fbc18d80cc3c9ca4f1f13ff1d46778c9a25eaaeca2a95314fdf20d8e |
| status      | Terminated                                                   |
| start_time  | 2081336.0                                                    |
| end_time    | 2083889.0                                                    |
| machine     | 471dda9ed84965451e042145                                     |

- `job_name`: job name; same as the entry in [pai_job_table](#pai_job_table).
- `task_name`: task name; same as the entry in [pai_task_table](#pai_task_table).
- `inst_name`: name of instance in each task.
- `worker_name`: 用来区分实例的信息，因为一个task可能有多个instance，它比inst_name更细节。information to distinguish instances; it is more detailed than `inst_name` and to be joined with `worker_name` in [pai_sensor_table](#pai_sensor_table) and [pai_machine_metric](#pai_machine_metric).
- `status`: instance status.
- `start_time`: timestamp of instance launch time.
- `end_time`: timestamp of instance completion time.
- `machine`: 实例所在的机器名称。the name of machine that the instance resides on, to be joined with `machine` in [pai_machine_spec](##pai_machine_spec) and [pai_machine_metric](#pai_machine_metric).



## pai_sensor_table

**instance resource sensor information.**

| Columns         | Example Entry                                                |
|:----------------|:-------------------------------------------------------------|
| job_name        | a9449d475665e3bf0512520b                                     |
| task_name       | worker                                                       |
| worker_name     | bcecec52225d6b4ae6bc724ce0269a02026195a364f54cf4850c2cca0054 |
| inst_id         | 589de47b56f88129837f506134b874e0356dc0931732a687bcf907fb8325 |
| machine         | 6884752e3565b15cafe14218                                     |
| gpu_name        | /dev/nvidia0                                                 |
| cpu_usage       | 140.1451612903226                                            |
| gpu_wrk_util    | 16.0625                                                      |
| avg_mem         | 1.4627511160714286                                           |
| max_mem         | 2.3935546875                                                 |
| avg_gpu_wrk_mem | 1.2446746826171875                                           |
| max_gpu_wrk_mem | 2.3994140625                                                 |
| read            | 21271328.384615384                                           |
| write           | 16376189.815384615                                           |
| read_count      | 2922.4461538461537                                           |
| write_count     | 3419.7846153846153                                           |

- `job_name`: job name; same as the entry in [pai_job_table](#pai_job_table).
- `task_name`: task name; same as the entry in [pai_task_table](#pai_task_table).
- `worker_name`: worker name; same as the entry in [pai_instance_table](#pai_instance_table).
- `inst_id`: instance id; same as the entry in [pai_job_table](#pai_job_table).
- `machine`: machine name; same as the entry in [pai_instance_table](#pai_instance_table).
- `gpu_name`: name of the GPU on that machine (not `gpu_type`).
- `cpu_usage`: number of CPU cores used in percentage (i.e., 600.0 is 6 vCPU cores) (c.f. `plan_cpu` in [pai_task_table](#pai_task_table)).
- `gpu_wrk_util`: number of GPUs used in percentage (i.e., 50.0 is 50% GPU) (c.f., `plan_gpu` in [pai_task_table](#pai_task_table)).
- `avg_mem`: GB of main memory used (in average) (c.f., `plan_mem` in [pai_task_table](#pai_task_table)).
- `max_mem`: GB of main memory used (maximum) (c.f., `plan_mem` in [pai_task_table](#pai_task_table)).
- `avg_gpu_wrk_mem`: GB of GPU memory used (in average).
- `max_gpu_wrk_mem`: GB of GPU memory used (maximum).
- `read`: Bytes of network input.
- `write`: Bytes of network output.
- `read_count`: Number of times of network read input.
- `write_count`: Number of times of network write output.

**Note of `sensor`**: all the sensor metrics (CPU, GPU, Memory, I/O) in this table are collected for each *instance* (indexed by  `worker_name`) but *not task*, taking the average of all data in the instance's lifetime (except for `max_mem` and `max_gpu_wrk_mem` being the maximum).



## pai_group_tag_table

**instance semantic information.**

| Columns       | Example Entry                                                |
|:--------------|:-------------------------------------------------------------|
| inst_id       | f7f6218cb5cb82e00b85476691d15d5055c143a351396d8f81737421dbd6 |
| user          | d2d3b77d342e                                                 |
| gpu_type_spec | V100M32                                                      |
| group         | fbeb14d671c629b6e82bee889fe4bb4c                             |
| workload      | nmt                                                          |

- `inst_id`: instance id; same as the entry in [pai_job_table](#pai_job_table).
- `user`: user name; same as the entry in [pai_job_table](#pai_job_table).
- `gpu_type_spec`: being empty if the instance does not specify GPU type requirements, else being one of the `gpu_type` in [pai_task_table](#pai_task_table).
- `group`: 表示某些实例具有相似的自定义输入的语义标签，例如入口脚本、命令行参数、数据源和接收器；因此，具有相同组标签的实例被视为重复实例。a semantic tag that indicates some instances have similar customized inputs, e.g., entry scripts, command-line parameters, data sources and sinks; consequently, instances with the same group tag are considered as repeated instances. Please refer to the trace analysis paper for detailed discussion.
- `workload`: we study some Deep Learning tasks by investigating their customized inputs (mentioned above) and record their workload in this field; around 9% instances have this tag, including `graphlearn`, `ctr`, `bert`, etc.



## pai_machine_spec

**machine specification.**

| Columns  | Example Entry            |
|:---------|:-------------------------|
| machine  | 82e5af3cd5c4af3f56c62c54 |
| gpu_type | T4                       |
| cap_cpu  | 96                       |
| cap_mem  | 512                      |
| cap_gpu  | 2                        |

- `machine`: machine name; same as the entry in [pai_instance_table](#pai_instance_table).
- `gpu_type`: GPU type; same as the entry in [pai_task_table](#pai_task_table).
- `cap_cpu`: CPU capacity; number of CPU cores in the machine.
- `cap_mem`: memory capacity; GB of main memory capacity in the machine.
- `cap_gpu`: GPU capacity; number of GPU in the machine.



## pai_machine_metric

**machine resource metrics with respect to the instance.**

| Columns             | Example Entry                                                |
|:--------------------|:-------------------------------------------------------------|
| worker_name         | b739d0d058e0db100aaf47e48a4d61320c95c2f5a334a8262d5e830d849c |
| machine             | 74e1c8457b01c76b314b22bb                                     |
| start_time          | 6150435                                                      |
| end_time            | 6150689                                                      |
| machine_cpu_iowait  | 0.0028667003281999497                                        |
| machine_cpu_kernel  | 3.583656890012642                                            |
| machine_cpu_usr     | 14.928745108999438                                           |
| machine_gpu         | 87.82875849911859                                            |
| machine_load_1      | 18.298592909228066                                           |
| machine_net_receive | 111649584.57135652                                           |
| machine_num_worker  | 5.053068410462776                                            |
| machine_cpu         | 18.515268699340282                                           |

- `worker_name`: worker name; same as the entry in [pai_instance_table](#pai_instance_table).
- `machine`: machine name; same as the entry in [pai_instance_table](#pai_instance_table).
- `start_time`: timestamp of instance launch time; same as the entry in [pai_instance_table](#pai_instance_table).
- `end_time`: timestamp of instance completion; same as the entry in [pai_instance_table](#pai_instance_table).
- `machine_cpu_iowait` : machine-level metrics of CPU I/O wait.
- `machine_cpu_kernel` : machine-level metrics of CPU kernel usage.
- `machine_cpu_usr` : machine-level metrics of CPU user usage.
- `machine_gpu` : machine-level metrics of GPU utilization.
- `machine_load_1` : machine-level metrics of 1-min load average.
- `machine_net_receive` : machine-level metrics of network received bytes.
- `machine_num_worker` : machine-level metrics of number of co-located instances (workers).
- `machine_cpu` : machine-level metrics of CPU overall usage.

**Note of `machine_` metrics**: these metrics are machine-level metrics, taking average of the sensor data during the instance's (indexed by `worker_name`) lifetime.

