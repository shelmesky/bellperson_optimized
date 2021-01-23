# TODO

## 优化思路

- 并发与调度
  - 充分利用 CPU 核心数
  - 多显卡并发计算
  - 减少重复的无用计算
  - 并发无依赖的 CPU 计算
  - 提前准备好 GPU 计算所需数据
  - CPU 和 GPU 并发计算
- 调参
  - 充分利用显卡的显存，减少数据存取次数
  - 根据显卡不同的位宽和核心数配置合理的参数
  - 跟换 CUDA 库提升性能
- 算法
  - 语言层面算法
  - 数学层面算法

### 并发与调度的最优解就是：

- CPU 一开始全部跑满
- GPU 一但开跑就全部跑满，持续到结束

```sh
  start                         end
    |                            |
CPU |-------------------->|      |
GPU |      |-------------------->|
```

## 优化步骤

- fork 代码：https://github.com/jackoelv/bellperson
- 熟悉代码，看明白代码大概逻辑
- 搞清楚各段代码耗费的时间多少，是耗在 CPU 还是 GPU
- 搞清楚各个耗时任务的 CPU 和 GPU 是否跑满
- 按照上面的优化思路进行代码优化，并测试
- 记录每次测试的日志，分析效果，便于分享

## 目前任务清单

1. 自助认领任务，添加昵称到任务清单下面
2. `pull requests`

### [1] `build provers` 并发优化

```rs
// build provers
info!("ZQ: build provers start");
let now = Instant::now();
let mut provers = circuits
    .into_par_iter()
    .map(|circuit| -> Result<_, SynthesisError> {
        let mut prover = ProvingAssignment::new();
        prover.alloc_input(|| "", || Ok(E::Fr::one()))?;
        // 这里是一个耗时操作，考虑函数里面再加并发，
        // 充分利用CPU核心数量，目前单个迭代只用到一个核心
        circuit.synthesize(&mut prover)?;
        for i in 0..prover.input_assignment.len() {
            prover.enforce(|| "", |lc| lc + Variable(Index::Input(i)), |lc| lc, |lc| lc);
        }
        Ok(prover)
    })
    .collect::<Result<Vec<_>, _>>()?;
info!("ZQ: build provers  end: {:?}", now.elapsed());
```

开发人员名单：

- （微信昵称，自助添加认领）
- 、

### [2] `a_s` 优化 1

```rs
info!("ZQ: a_s start");

// 。。。

// 这3个fft考虑充分利用显存，同时计算
let mut a =
    EvaluationDomain::from_coeffs(std::mem::replace(&mut prover.a, Vec::new()))?;
let mut b =
    EvaluationDomain::from_coeffs(std::mem::replace(&mut prover.b, Vec::new()))?;
let mut c =
    EvaluationDomain::from_coeffs(std::mem::replace(&mut prover.c, Vec::new()))?;

a.ifft(&worker, &mut fft_kern)?;
a.coset_fft(&worker, &mut fft_kern)?;
b.ifft(&worker, &mut fft_kern)?;
b.coset_fft(&worker, &mut fft_kern)?;
c.ifft(&worker, &mut fft_kern)?;
c.coset_fft(&worker, &mut fft_kern)?;

// 。。。

info!("ZQ: a_s end: {:?}", now.elapsed());
drop(fft_kern);
```

开发人员名单：

- （微信昵称，自助添加认领）
- 、

### [3] `a_s` 优化 2

多显卡并发计算（已完成）

### [4] `inputs` 优化

```rs
info!("ZQ: inputs start");
let now = Instant::now();
let inputs = provers
// 。。。

// 有三个类似的操作
// 可以提取到map外面提前并发计算好结果
// 去掉GPU等待时间
let (
    a_aux_bss,
    a_aux_exps,
    a_aux_skip,
    a_aux_n
) = density_filter(
    a_aux_source.clone(),
    Arc::new(prover.a_aux_density),
    aux_assignment.clone()
);
// 。。。
let (
    b_g1_aux_bss,
    b_g1_aux_exps,
    b_g1_aux_skip,
    b_g1_aux_n
) = density_filter(
    b_g1_aux_source.clone(),
    b_aux_density.clone(),
    aux_assignment.clone()
);
// 。。。
let (
    b_g2_aux_bss,
    b_g2_aux_exps,
    b_g2_aux_skip,
    b_g2_aux_n
) = density_filter(
    b_g2_aux_source.clone(),
    b_aux_density.clone(),
    aux_assignment.clone()
);
//。。。

info!("ZQ: inputs end: {:?}", now.elapsed());
```

开发人员名单：

- （微信昵称，自助添加认领）
- 、

### [5] GPU 参数调整

根据不同的 GPU 参数，调整参数

- 52:`calc_num_groups`
- 76:`calc_best_chunk_size`
- 87:`calc_chunk_size`

开发人员名单：

- （微信昵称，自助添加认领）
- 、

### [6] opencl 更换 CUDA 库

开发人员名单：

- （微信昵称，自助添加认领）
- 、

### [7] 算法优化

开发人员名单：

- （微信昵称，自助添加认领）
- 、

### [8] 不同显卡测试

开发人员名单：

- （微信昵称，自助添加认领）
- 、
