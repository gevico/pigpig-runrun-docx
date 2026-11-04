# 文档目的
这篇文档用于记录解决 model 算子的指令映射的，稍后会更新到

## 第一，实际处理算子的映射

通过记录 py 向下层 cpp，cuda 的调用点对实际的算子进行记录，并通过这个记录来整理出来最开始的算子实现

py 的 tensor 分成底层的实际内存和上层的数据读取，这种分层在具体 gw100 的具体代码中也体现出来，对于上层的数据读取并非实现关键，通过一般代码就可以完成，只是要去考虑如何进行抽象，给具体层还是在特定序列前转化？目前项目实施是后者。

在 py 中上层数据 impl 的转化的算子不是重点，重点是实际处理底层数据的算子，比如拷贝，乘加等操作，目前项目的底层指令还没有完全实现。

下面针对 qwen 进行算子统计
```py
import torch
import torch.nn as nn

class RMSNorm(nn.Module):
    def __init__(self, dim: int, eps: float = 1e-6):
        super().__init__()
        self.eps = eps
        self.weight = nn.Parameter(torch.ones(dim))

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        variance = x.pow(2).mean(-1, keepdim=True)
        x_normed = x * torch.rsqrt(variance + self.eps)
        return self.weight * x_normed
```


```shell
grep -v "aten.view\|aten.expand\|aten.transpose\|aten._unsafe_view\|aten.t.default\|aten.unsqueeze\|aten.slice" research/outputs/trace_ops_qwen2.5-0.5b_decode_eager.txt | sed 's/^[[:space:]]*[0-9]*[[:space:]]*//' | sort | uniq -c | sort -rn
```

```text
5 aten.rsqrt.default                            float32[1, 1, 1] -> float32[1, 1, 1]
   5 aten.pow.Tensor_Scalar                        float32[1, 1, 896] -> float32[1, 1, 896]
   5 aten.mul.Tensor                               float32[1, 1, 896] float32[1, 1, 1] -> float32[1, 1, 896]
   5 aten.mul.Tensor                               bfloat16[896] bfloat16[1, 1, 896] -> bfloat16[1, 1, 896]
   5 aten.mean.dim                                 float32[1, 1, 896] -> float32[1, 1, 1]
   5 aten.add.Tensor                               float32[1, 1, 1] -> float32[1, 1, 1]
   5 aten._to_copy.default                         float32[1, 1, 896] -> bfloat16[1, 1, 896]
   5 aten._to_copy.default                         bfloat16[1, 1, 896] -> float32[1, 1, 896]
   4 aten.mul.Tensor                               bfloat16[1, 2, 1, 64] bfloat16[1, 1, 1, 64] -> bfloat16[1, 2, 1, 64]
   4 aten.mul.Tensor                               bfloat16[1, 14, 1, 64] bfloat16[1, 1, 1, 64] -> bfloat16[1, 14, 1, 64]
   4 aten.mm.default                               bfloat16[1, 896] bfloat16[896, 4864] -> bfloat16[1, 4864]
   4 aten.clone.default                            bfloat16[1, 2, 7, 9, 64] -> bfloat16[1, 2, 7, 9, 64]
   4 aten.cat.default                              (bfloat16[1, 2, 8, 64],bfloat16[1, 2, 1, 64]) -> bfloat16[1, 2, 9, 64]
   4 aten.addmm.default                            bfloat16[128] bfloat16[1, 896] bfloat16[896, 128] -> bfloat16[1, 128]
   4 aten.add.Tensor                               bfloat16[1, 1, 896] bfloat16[1, 1, 896] -> bfloat16[1, 1, 896]
   3 aten.arange.default                            -> int64[1]
   2 aten.silu.default                             bfloat16[1, 1, 4864] -> bfloat16[1, 1, 4864]
   2 aten.neg.default                              bfloat16[1, 2, 1, 32] -> bfloat16[1, 2, 1, 32]
   2 aten.neg.default                              bfloat16[1, 14, 1, 32] -> bfloat16[1, 14, 1, 32]
   2 aten.mul.Tensor                               float32[1, 1, 64] -> float32[1, 1, 64]
   2 aten.mul.Tensor                               bfloat16[1, 14, 1, 9] -> bfloat16[1, 14, 1, 9]
   2 aten.mul.Tensor                               bfloat16[1, 1, 4864] bfloat16[1, 1, 4864] -> bfloat16[1, 1, 4864]
   2 aten.mm.default                               bfloat16[1, 896] bfloat16[896, 896] -> bfloat16[1, 896]
   2 aten.mm.default                               bfloat16[1, 4864] bfloat16[4864, 896] -> bfloat16[1, 896]
   2 aten.cat.default                              (bfloat16[1, 2, 1, 32],bfloat16[1, 2, 1, 32]) -> bfloat16[1, 2, 1, 64]
   2 aten.cat.default                              (bfloat16[1, 14, 1, 32],bfloat16[1, 14, 1, 32]) -> bfloat16[1, 14, 1, 64]
   2 aten.bmm.default                              bfloat16[14, 1, 9] bfloat16[14, 9, 64] -> bfloat16[14, 1, 64]
   2 aten.bmm.default                              bfloat16[14, 1, 64] bfloat16[14, 64, 9] -> bfloat16[14, 1, 9]
   2 aten.addmm.default                            bfloat16[896] bfloat16[1, 896] bfloat16[896, 896] -> bfloat16[1, 896]
   2 aten.add.Tensor                               bfloat16[1, 2, 1, 64] bfloat16[1, 2, 1, 64] -> bfloat16[1, 2, 1, 64]
   2 aten.add.Tensor                               bfloat16[1, 14, 1, 9] bfloat16[1, 1, 1, 9] -> bfloat16[1, 14, 1, 9]
   2 aten.add.Tensor                               bfloat16[1, 14, 1, 64] bfloat16[1, 14, 1, 64] -> bfloat16[1, 14, 1, 64]
   2 aten._to_copy.default                         float32[1, 14, 1, 9] -> bfloat16[1, 14, 1, 9]
   2 aten._to_copy.default                         float32[1, 1, 64] -> bfloat16[1, 1, 64]
   2 aten._to_copy.default                         bfloat16[1, 14, 1, 9] -> float32[1, 14, 1, 9]
   2 aten._softmax.default                         float32[1, 14, 1, 9] -> float32[1, 14, 1, 9]
   1 aten.zeros.default                             -> bfloat16[]
   1 aten.where.self                               bool[1, 1, 1, 9] bfloat16[] bfloat16[] -> bfloat16[1, 1, 1, 9]
   1 aten.sin.default                              float32[1, 1, 64] -> float32[1, 1, 64]
   1 aten.scalar_tensor.default                     -> bfloat16[]
   1 aten.mul.Tensor                               float32[1, 1, 1] float32[32] -> float32[1, 1, 32]
   1 aten.mm.default                               bfloat16[1, 896] bfloat16[896, 151936] -> bfloat16[1, 151936]
   1 aten.le.Tensor                                int64[1, 1, 1, 9] int64[1, 1, 1, 1] -> bool[1, 1, 1, 9]
   1 aten.embedding.default                        bfloat16[151936, 896] int64[1, 1] -> bfloat16[1, 1, 896]
   1 aten.cos.default                              float32[1, 1, 64] -> float32[1, 1, 64]
   1 aten.cat.default                              (float32[1, 1, 32],float32[1, 1, 32]) -> float32[1, 1, 64]
   1 aten.arange.default                            -> int64[9]
   1 aten.add.Tensor                               int64[9] -> int64[9]
   1 aten.add.Tensor                               int64[1] -> int64[1]
   1 aten._to_copy.default                         int64[1, 1, 1] -> float32[1, 1, 1]
```

上面三段内容具体做了一下操作，通过 py 脚本去拦截所有算子，过滤没有真正处理内存的算子，统计内存处理的算子并进行整理，没有去整理整个 model，仅仅只是对 atten + rms 的简单一层进行周总结

抛开具体处理数据格式不谈，真正需要处理的算子如下
```text

  25 aten.mul.Tensor
  17 aten.add.Tensor
  17 aten._to_copy.default
   9 aten.mm.default
   9 aten.cat.default
   6 aten.addmm.default
   5 aten.rsqrt.default
   5 aten.pow.Tensor_Scalar
   5 aten.mean.dim
   4 aten.neg.default
   4 aten.clone.default
   4 aten.bmm.default
   4 aten.arange.default
   2 aten.silu.default
   2 aten._softmax.default
   1 aten.zeros.default
   1 aten.where.self
   1 aten.sin.default
   1 aten.scalar_tensor.default
   1 aten.le.Tensor
   1 aten.embedding.default
   1 aten.cos.default
```

这就是我们需要去实现的算子映射：
1.矩阵计算类（3 种，共 19 次）：mm (9)、addmm (6)、bmm (4)
2.逐元素与归约计算类（12 种，共 69 次）：mul (25)、add (17)、rsqrt (5)、pow (5)、mean (5)、neg (4)、silu (2)、_softmax (2)、sin (1)、cos (1)、le (1)、where (1)
3.物理数据搬运与初始化类（7 种，共 37 次）：_to_copy (17，精度转换拷贝)、cat (9，拼接拷贝)、clone (4，GQA 连续化拷贝)、arange (4)、embedding (1，词表查表)、zeros (1)、scalar_tensor (1)

## 两个model 的对比

### config的区别

```text
Key                              | Qwen2.5-0.5B           | Llama-3.2-1B          
----------------------------------------------------------------------------------
architectures                    | ['Qwen2ForCausalLM']   | ['LlamaForCausalLM']  
attention_bias                   | <missing>              | False                 
bos_token_id                     | 151643                 | 128000                
eos_token_id                     | 151643                 | 128001                
head_dim                         | <missing>              | 64                    
hidden_size                      | 896                    | 2048                  
intermediate_size                | 4864                   | 8192                  
max_position_embeddings          | 32768                  | 131072                
max_window_layers                | 24                     | <missing>             
mlp_bias                         | <missing>              | False                 
model_type                       | qwen2                  | llama                 
num_attention_heads              | 14                     | 32                    
num_hidden_layers                | 24                     | 16                    
num_key_value_heads              | 2                      | 8                     
pretraining_tp                   | <missing>              | 1                     
rms_norm_eps                     | 1e-06                  | 1e-05                 
rope_scaling                     | <missing>              | {'factor': 32.0, 'high_freq_factor': 4.0, 'low_freq_factor': 1.0, 'original_max_position_embeddings': 8192, 'rope_type': 'llama3'}
rope_theta                       | 1000000.0              | 500000.0              
sliding_window                   | 32768                  | <missing>             
transformers_version             | 4.40.1                 | 4.45.0.dev0           
use_mrope                        | False                  | <missing>             
use_sliding_window               | False                  | <missing>             
vocab_size                       | 151936                 | 128256                
((.venv) ) wangyuxin@wangyuxindeMacBook-Air gw100-gem5 % 
```

绝大多数的参数只是数值上的不同，需要注意的有两点一个是 attension bias 是否开始，这会影响到qkv 投影的计算，这一点倒是暂时不关我们目前的主题以及rope—scaling 的有无，这一点和之前那个点很类似

### 算子对比

llama 的算子实现如下
```text
  25 aten.mul.Tensor
  17 aten.add.Tensor
  17 aten._to_copy.default
  15 aten.mm.default
   9 aten.cat.default
   5 aten.rsqrt.default
   5 aten.pow.Tensor_Scalar
   5 aten.mean.dim
   4 aten.neg.default
   4 aten.clone.default
   4 aten.bmm.default
   4 aten.arange.default
   2 aten.silu.default
   2 aten._softmax.default
   1 aten.zeros.default
   1 aten.where.self
   1 aten.sin.default
   1 aten.scalar_tensor.default
   1 aten.le.Tensor
   1 aten.embedding.default
   1 aten.cos.default
```


对比

```text
Operator                     |  Qwen2.5 | Llama-3.2 |   Diff
------------------------------------------------------------
aten.view.default            |       37 |        31 |     -6
aten.mul.Tensor              |       25 |        25 |      0
aten.unsqueeze.default       |       21 |        21 |      0
aten._to_copy.default        |       17 |        17 |      0
aten._unsafe_view.default    |       17 |        23 |     +6
aten.add.Tensor              |       17 |        17 |      0
aten.t.default               |       15 |        15 |      0
aten.expand.default          |       13 |        13 |      0
aten.transpose.int           |       10 |        10 |      0
aten.cat.default             |        9 |         9 |      0
aten.mm.default              |        9 |        15 |     +6
aten.slice.Tensor            |        9 |         9 |      0
aten.addmm.default           |        6 |         0 |     -6
aten.mean.dim                |        5 |         5 |      0
aten.pow.Tensor_Scalar       |        5 |         5 |      0
aten.rsqrt.default           |        5 |         5 |      0
aten.arange.default          |        4 |         4 |      0
aten.bmm.default             |        4 |         4 |      0
aten.clone.default           |        4 |         4 |      0
aten.neg.default             |        4 |         4 |      0
aten._softmax.default        |        2 |         2 |      0
aten.silu.default            |        2 |         2 |      0
aten.cos.default             |        1 |         1 |      0
aten.embedding.default       |        1 |         1 |      0
aten.le.Tensor               |        1 |         1 |      0
aten.scalar_tensor.default   |        1 |         1 |      0
aten.sin.default             |        1 |         1 |      0
aten.where.self              |        1 |         1 |      0
aten.zeros.default           |        1 |         1 |      0
```

也就是说，在我工作的最开始阶段，完全不需要管两个 model 上的算子差别，只需要优先实现对应的映射。

## 其他项目实现思路分析

