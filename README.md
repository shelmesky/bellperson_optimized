# Contributors
- [FOO]jackoelv
- [FOO]ROy
- [FOO]Gabe
- [FOO-Hacker]Jerry
- [FOO-TEST]我是鱼饵
- [FOO-C2]devid
- [FOO]石头
- [FOO]jerry
- [FOO]Waitting for you ...
# Add 3080 dev Branch
- AMD 3970x 12min and can be onchain now.
- tune based on 3090.

# C2 in 880s tested.
- this is special for 3080, checkout from 3090, many things to do.
- Team work requests are welcome!

From ZQBC

# HOWTO
- get the code:

```bash
cd ../lotus_code_path && git clone https://github.com/jackoelv/bellperson.git && git checkout origin/2080Ti
```
- patch the filecoin-ffi submodule

```bash
cd ./lotus_code_path && git submodule update --init --recursive
cd ./lotus_code_path/extern/filecoin-ffi/rust/
vi Cargo.toml 
```
> in the end of the file add patch code:

```rust
[patch.crates-io]
bellperson = { path = "../../../../bellperson" }
```
- then update cargo package

```bash
cd ./lotus_code_path/extern/filecoin-ffi/rust/
cargo update
cd ./lotus_code_path
RUSTFLAGS="-C target-cpu=native -g" FFI_BUILD_FROM_SOURCE=1 make clean all
```

> enjoy it!

# DONATION

Jennifer suggested that I should have a donation wallet.

fils are welcome if you like.

my wallet addr:

> f1ki5mgbm4cyz43oamnbvv5bjrqdsvkphuxxs2h4a

# FAQ

> openCL error

 if you see error like this:
```bash
Status error code: CL_MEM_OBJECT_ALLOCATION_FAILURE (-4)
```
 because the MEM needs exceeds maxinum gpu mem. I am looking for new solution to caculate the suitable mem needs.
a temporary solution is change the variable ：
- src/gpu/multiexp.rs
```rust
324: jack_chunk = (jack_chunk as f64 / 10f64).ceil() as usize;
```
 you can increase `10f64` to `11f64` even bigger to reduce mem needs.
 I am trying a new solution which change window_size for lower cost of mem. 
 In prover.rs, b_g2_aux caculation size of <<G as CurveAffine>::Projective> is doubled,  so with same window_size and num_groups, mem needs increase fast.  

> attention: this code removed from 3090 branch.
# bellperson [![Crates.io](https://img.shields.io/crates/v/bellperson.svg)](https://crates.io/crates/bellperson)

> This is a fork of the great [bellman](https://github.com/zkcrypto/bellman) library.

`bellman` is a crate for building zk-SNARK circuits. It provides circuit traits
and primitive structures, as well as basic gadget implementations such as
booleans and number abstractions.

## Backend

There are currently two backends available for the implementation of Bls12 381:
- [`paired`](https://github.com/filecoin-project/paired) - pure Rust implementation
- [`blstrs`](https://github.com/filecoin-project/blstrs) - optimized with hand tuned assembly, using [blst](https://github.com/supranational/blst)

They can be  selected at compile time with the mutually exclusive features `pairing` and `blst`. Specifying one of them is enough for a working library, no additional features need to be set.
The default for now is `pairing`, as the secure and audited choice.

## GPU

This fork contains GPU parallel acceleration to the FFT and Multiexponentation algorithms in the groth16 prover codebase under the compilation feature `gpu`, it can be used in combination with `pairing` or `blst`.

### Requirements
- NVIDIA or AMD GPU Graphics Driver
- OpenCL

( For AMD devices we recommend [ROCm](https://rocm-documentation.readthedocs.io/en/latest/Installation_Guide/Installation-Guide.html) )

### Environment variables

The gpu extension contains some env vars that may be set externally to this library.

- `BELLMAN_NO_GPU`

    Will disable the GPU feature from the library and force usage of the CPU.

    ```rust
    // Example
    env::set_var("BELLMAN_NO_GPU", "1");
    ```

- `BELLMAN_VERIFIER`

    Chooses the device in which the batched verifier is going to run. Can be `cpu`, `gpu` or `auto`.

    ```rust
    Example
    env::set_var("BELLMAN_VERIFIER", "gpu");
    ```

- `BELLMAN_CUSTOM_GPU`

    Will allow for adding a GPU not in the tested list. This requires researching the name of the GPU device and the number of cores in the format `["name:cores"]`.

    ```rust
    // Example
    env::set_var("BELLMAN_CUSTOM_GPU", "GeForce RTX 2080 Ti:4352, GeForce GTX 1060:1280");
    ```

- `BELLMAN_CPU_UTILIZATION`

    Can be set in the interval [0,1] to designate a proportion of the multiexponenation calculation to be moved to cpu in parallel to the GPU to keep all hardware occupied.

    ```rust
    // Example
    env::set_var("BELLMAN_CPU_UTILIZATION", "0.5");
    ```

#### Supported / Tested Cards

Depending on the size of the proof being passed to the gpu for work, certain cards will not be able to allocate enough memory to either the FFT or Multiexp kernel. Below are a list of devices that work for small sets. In the future we will add the cuttoff point at which a given card will not be able to allocate enough memory to utilize the GPU.

| Device Name            | Cores | Comments       |
|------------------------|-------|----------------|
| Quadro RTX 6000        | 4608  |                |
| TITAN RTX              | 4608  |                |
| Tesla V100             | 5120  |                |
| Tesla P100             | 3584  |                |
| Tesla T4               | 2560  |                |
| Quadro M5000           | 2048  |                |
| GeForce RTX 2080 Ti    | 4352  |                |
| GeForce RTX 2080 SUPER | 3072  |                |
| GeForce RTX 2080       | 2944  |                |
| GeForce RTX 2070 SUPER | 2560  |                |
| GeForce GTX 1080 Ti    | 3584  |                |
| GeForce GTX 1080       | 2560  |                |
| GeForce GTX 2060       | 1920  |                |
| GeForce GTX 1660 Ti    | 1536  |                |
| GeForce GTX 1060       | 1280  |                |
| GeForce GTX 1650 SUPER | 1280  |                |
| GeForce GTX 1650       | 896   |                |
|                        |       |                |
| gfx1010                | 2560  | AMD RX 5700 XT |

### Running Tests

To run the multiexp_consistency test you can use:

```bash
RUST_LOG=info cargo test --features gpu -- --exact multiexp::gpu_multiexp_consistency --nocapture
```

### Considerations

Bellperson uses `rust-gpu-tools` as its OpenCL backend, therefore you may see a
directory named `~/.rust-gpu-tools` in your home folder, which contains the
compiled binaries of OpenCL kernels used in this repository.

## License

Licensed under either of

- Apache License, Version 2.0, |[LICENSE-APACHE](LICENSE-APACHE) or
   http://www.apache.org/licenses/LICENSE-2.0)
- MIT license ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

at your option.

### Contribution

Unless you explicitly state otherwise, any contribution intentionally
submitted for inclusion in the work by you, as defined in the Apache-2.0
license, shall be dual licensed as above, without any additional terms or
conditions.
