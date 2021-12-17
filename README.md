# 编译部署

> TIP: 我们需要使用相同的开发环境，以便后期的审计

### 编译

```
cd program

docker run -it -v $PWD:/src/rust/solana hoseadevops/solans-program:0.1.0 /bin/bash

export PATH=/root/.local/share/solana/install/active_release/bin:$PATH

cargo build-bpf -v
```
### 部署
```
solana program deploy --program-id ../program/target/deploy/spl_token_vesting-keypair.json ../program/target/deploy/spl_token_vesting.so --url https://api.devnet.solana.com --keypair ../keys/deploy.json

> TIP: 请充分测试后关闭升级

```

### 创建锁仓
> TIP: 操作失误 token 将不能从锁仓中取出

```
cd cli

cargo build
```

###### 创建 vesting 获取 种子
```
echo "RUST_BACKTRACE=1 ./target/debug/spl-token-vesting-cli \
--url https://api.devnet.solana.com \
--program_id 5ZFe3yW75iGvap4eD34DBgsoYeBofEF3aGpEMGDyZhzj  \
create \
--mint_address 8nwzDEdnsU5uVDW29zCmVPSM8Am2JuhPkMkiWfNMgqQs \
--source_owner ../keys/id_owner.json \
--source_token_address AaM2tHu8qUpmvk3HkfP2Pb4UjoBJcxhq25g6UzU6sTeE  \
--destination_token_address J5vKa4x3ccrGwcHxL6BpTBcVo3WtFAgyuQ6Fctkynsyn \
--amounts 1639724343,1639724343,1639724343,! \
--release-times 2,28504431,1639635407,! \
--payer ../keys/id_owner.json" \
--verbose | bash

# SEED: LX3EUdRUBUa3TbsYXLEUdj9J3prXkWXvLYSWyYyc2P8
```

###### 支持ATA账户
To use [Associated Token Account](https://spl.solana.com/associated-token-account) as destination use `--destination_address`(with public key of `id_dest`) instead of `--destination_token_address`.

###### 查询锁仓状态
```
echo "RUST_BACKTRACE=1 ./target/debug/spl-token-vesting-cli \
--url https://api.devnet.solana.com \
--program_id 5ZFe3yW75iGvap4eD34DBgsoYeBofEF3aGpEMGDyZhzj \
info \
--seed LX3EUdRUBUa3TbsYXLEUdj9J3prXkWXvLYSWyYyc2P8 " | bash
```