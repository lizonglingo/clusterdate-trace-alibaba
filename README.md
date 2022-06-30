## clusterdate-trace-alibaba

本项目为数据挖掘课程报告的代码仓库。

对Alibaba的[cluster-trace-gpu-v2020项目](https://github.com/alibaba/clusterdata/tree/master/cluster-trace-gpu-v2020)提供的数据集进行分析，探索一些集群资源调度方面的问题，主要使用到Darts中的时序预测进行建模和预测。

## 目录说明

- darts: darts库的demo用于熟悉该预测工具的使用。
- gpu_cluster: 包含对数据集的处理和尝试。
- microservice_trace: 对Alibaba的另一个数据集的简单处理，但是最终没有选择使用该数据集。
- pytorch-forecasting: 尝试了一些预测模型的效果，最终选择了N-BEATS模型。

其中报告涉及到的主要代码在`/gpu_claster/darts/time_forecasting.ipynb`和`/gpu_claster/darts/N-BEATS.ipynb`中。
