# 编译部署

> TIP: 我们需要使用相同的开发环境，以便后期的审计

### 编译

```
cd program

docker run -it -v $PWD:/src/rust/solana hoseadevops/solans-program:0.1.0 /bin/bash

export PATH=/root/.local/share/solana/install/active_release/bin:$PATH

cargo build-bpf -v
```
### 创建锁仓

```
cd cli

cargo build
```