# SPL-Token-Vesting

> TIP: 我们需要使用相同的开发环境，以便后期的审计, 项目自行编译请提供可审计环境.

### 编译

```
cd program

docker run -it -v $PWD:/src/rust/solana hoseadevops/solans-program:0.1.0 /bin/bash

export PATH=/root/.local/share/solana/install/active_release/bin:$PATH

cargo build-bpf -v
```

### 部署

> TIP: 请充分测试后关闭升级

```
# https://docs.solana.com/cli/deploy-a-program 
# 为升级预留空间
solana program deploy --program-id ../program/target/deploy/spl_token_vesting-keypair.json ../program/target/deploy/spl_token_vesting.so --url https://api.devnet.solana.com --keypair ../keys/deploy.json
```

### 构建命令行

```
cd cli

cargo build
```

### 锁仓

> TIP: 操作失误 token 将不能从锁仓中取出 请充分测试

###### 创建锁仓 获取种子
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


### devnet testing 

```
# id_owner = 锁仓账户
# id_dest = 解锁目标账户 
# id_new_dest = 新的解锁目标账户

solana-keygen new --outfile ../keys/id_deploy.json --force
solana-keygen new --outfile ../keys/id_owner.json --force
solana-keygen new --outfile ../keys/id_dest.json --force  
solana-keygen new --outfile ../keys/id_new_dest.json --force
solana-keygen new --outfile ../keys/id_anyone.json --force

# 获取 SOL
solana airdrop 2 --url https://api.devnet.solana.com ../keys/id_deploy.json
solana airdrop 2 --url https://api.devnet.solana.com ../keys/id_deploy.json
solana airdrop 2 --url https://api.devnet.solana.com ../keys/id_deploy.json

solana airdrop 2 --url https://api.devnet.solana.com ../keys/id_owner.json
solana airdrop 2 --url https://api.devnet.solana.com ../keys/id_owner.json
solana airdrop 2 --url https://api.devnet.solana.com ../keys/id_owner.json

solana airdrop 2 --url https://api.devnet.solana.com ../keys/id_anyone.json
solana airdrop 2 --url https://api.devnet.solana.com ../keys/id_anyone.json

solana balance --url https://api.devnet.solana.com ../keys/id_deploy.json
solana balance --url https://api.devnet.solana.com ../keys/id_owner.json
solana balance --url https://api.devnet.solana.com ../keys/id_anyone.json


# 创建测试 Token
spl-token create-token --url https://api.devnet.solana.com
# token mint BLX3JUJoTRdj6YeeP54wAGotXC4FDAaAx59WbQ1imV7r

# 创建 owner ATA
spl-token create-account BLX3JUJoTRdj6YeeP54wAGotXC4FDAaAx59WbQ1imV7r --url https://api.devnet.solana.com --owner ../keys/id_owner.json
## Creating account 29XJSWUwBVw3w7t65LdFb7hk6m9p7xmyGtoKYfU3qj5k

# 铸币
spl-token mint BLX3JUJoTRdj6YeeP54wAGotXC4FDAaAx59WbQ1imV7r 10000 --url https://api.devnet.solana.com 29XJSWUwBVw3w7t65LdFb7hk6m9p7xmyGtoKYfU3qj5k --fee-payer ../keys/id_owner.json

# 获取token余额
spl-token balance BLX3JUJoTRdj6YeeP54wAGotXC4FDAaAx59WbQ1imV7r --url https://api.devnet.solana.com --owner ../keys/id_owner.json

## 修改合约 processor.rs AAmGoPDFLG6bE82BgZWjVi8k95tj9Tf3vUN7WvtUm2BU 为 BLX3JUJoTRdj6YeeP54wAGotXC4FDAaAx59WbQ1imV7r 并重现编译部署
# 部署合约
solana program deploy --program-id ../program/target/deploy/spl_token_vesting-keypair.json ../program/target/deploy/spl_token_vesting.so --url https://api.devnet.solana.com --keypair ../keys/id_deploy.json
# 升级
solana program deploy --program-id S41sCbAddb9EqBzdt3hPbNEGC5qbXwi2rmqdryeSRve ../program/target/deploy/spl_token_vesting.so --url https://api.devnet.solana.com --keypair ../keys/id_deploy.json
Program Id: S41sCbAddb9EqBzdt3hPbNEGC5qbXwi2rmqdryeSRve

# 创建受益人 ATA
spl-token create-account BLX3JUJoTRdj6YeeP54wAGotXC4FDAaAx59WbQ1imV7r --url https://api.devnet.solana.com --owner ../keys/id_dest.json
# 7SZBE8cU3g8E8J7XoWF1xKZa2xMRqvN4YnnG2SLJALTd

# 创建锁仓
echo "RUST_BACKTRACE=1 ./target/debug/spl-token-vesting-cli \
--url https://api.devnet.solana.com \
--program_id S41sCbAddb9EqBzdt3hPbNEGC5qbXwi2rmqdryeSRve  \
create \
--mint_address BLX3JUJoTRdj6YeeP54wAGotXC4FDAaAx59WbQ1imV7r \
--source_owner ../keys/id_owner.json \
--source_token_address 29XJSWUwBVw3w7t65LdFb7hk6m9p7xmyGtoKYfU3qj5k  \
--destination_token_address 7SZBE8cU3g8E8J7XoWF1xKZa2xMRqvN4YnnG2SLJALTd \
--amounts 1639724343,1639724343,1639724343,! \
--release-times 2,28504431,1639635407,! \
--payer ../keys/id_owner.json" \
--verbose | bash

The seed of the contract is: CiDwVBFgWV9E5MvXWoLgnEgn2hK7rJikbvfWavzAR4P

# 查询token 余额
spl-token balance BLX3JUJoTRdj6YeeP54wAGotXC4FDAaAx59WbQ1imV7r --url https://api.devnet.solana.com --owner ../keys/id_owner.json

# 查看锁仓
echo "RUST_BACKTRACE=1 ./target/debug/spl-token-vesting-cli \
--url https://api.devnet.solana.com \
--program_id S41sCbAddb9EqBzdt3hPbNEGC5qbXwi2rmqdryeSRve \
info \
--seed CiDwVBFgWV9E5MvXWoLgnEgn2hK7rJikbvfWavzAR4P " | bash

# 解锁
echo "RUST_BACKTRACE=1 ./target/debug/spl-token-vesting-cli \
--url https://api.devnet.solana.com \
--program_id S41sCbAddb9EqBzdt3hPbNEGC5qbXwi2rmqdryeSRve \
unlock \
--seed CiDwVBFgWV9E5MvXWoLgnEgn2hK7rJikbvfWavzAR4P \
--payer ../keys/id_anyone.json" \
--verbose | bash

# 查询token 余额
spl-token balance BLX3JUJoTRdj6YeeP54wAGotXC4FDAaAx59WbQ1imV7r --url https://api.devnet.solana.com --owner ../keys/id_owner.json
spl-token balance BLX3JUJoTRdj6YeeP54wAGotXC4FDAaAx59WbQ1imV7r --url https://api.devnet.solana.com --owner ../keys/id_dest.json
```