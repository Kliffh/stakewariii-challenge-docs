## 概要
- 在shardnet上创建钱包&安装NEAR CLI
- 部署⼀个节点。下载快照并同步数据，然后把节点激活为验证者
- 为验证节点部署⼀个新的质押池（staking pool）
- 配置⼀些⼯具来监控节点状态

## 创建钱包&安装NEAR CLI
账号信息  (sender-stakewar.shardnet.near)
质押池  sender-stakewar.factory.shardnet.near
打开 https ://wallet.shardnet.near.org

根据页面提示创建钱包：
![](file:///C:\Users\ADMINI~1\AppData\Local\Temp\ksohtml\wpsA1A.tmp.jpg)

设置near-cli，根据步骤安装。

    sudo apt update && sudo apt upgrade -y
    
    curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
    sudo apt install build-essential nodejs
    PATH="$PATH"
    node -v
    npm -v
    
    sudo npm install -g near-cli
    export NEAR_ENV=shardnet
    echo 'export NEAR_ENV=shardnet' >> ~/.bashrc
    echo 'export NEAR_ENV=shardnet' >> ~/.bash_profile
    source $HOME/.bash_profile

## 部署节点
### 安装**nodejs npm**

    sudo apt update && sudo apt upgrade -y
    curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
    sudo apt install build-essential nodejs
    PATH="$PATH"
    sudo npm install -g near-cli
    export NEAR_ENV=shardnet
    echo 'export NEAR_ENV=shardnet' >> ~/.bashrc

版本：

    root@iZ0xidz2gpmaq4u0rgpzvkZ:~# node -v
    v18.6.0

    root@iZ0xidz2gpmaq4u0rgpzvkZ:~# npm -v
    8.13.2

### 部署**nearcore**，运行节点

    lscpu | grep -P '(?=.*avx )(?=.*sse4.2 )(?=.*cx16 )(?=.*popcnt )' > /dev/null \
    && echo "Supported" \
    || echo "Not supported"

![](file:///C:\Users\ADMINI~1\AppData\Local\Temp\ksohtml\wps9615.tmp.jpg)

    sudo apt install -y git binutils-dev libcurl4-openssl-dev zlib1g-dev libdw-dev libiberty-dev cmake gcc g++ docker.io protobuf-compiler libssl-dev pkg-config
    clang llvm cargo

    apt install python3-pip
    USER_BASE_BIN=$(python3 -m site --user-base)/bin
    export PATH="$USER_BASE_BIN:$PATH"
    export PATH="/root/.local/bin:$PATH"
    sudo apt install clang build-essential make
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

![](file:///C:\Users\ADMINI~1\AppData\Local\Temp\ksohtml\wps9616.tmp.jpg)

按 1 并按 Enter

    source $HOME/.cargo/env
    git clone https://github.com/near/nearcore
    cd nearcore
    git fetch
    git checkout <cm>
    cargo build -p neard --release --features shardnet

工作初始化目录：

    mkdir -p ~/.near
    ./target/release/neard --home /data/.near init --chain-id shardnet --downloadgenesis
    rm /data/.near/config.json

更换config.json：

    wget -O ~/.near/config.json https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/config.json
    sudo apt-get install awscli -y
    cd ~/.near
    rm -rvf genesis.json

更换genesis.json：

    wget https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcoredeploy/
    shardnet/genesis.json

运行节点：

    cd ~/
    mkdir  near
    cd near
    mkdir bin
    cp ./target/release/neard ~/near/bin/

编写shell启动脚本：

![](file:///C:\Users\ADMINI~1\AppData\Local\Temp\ksohtml\wps9617.tmp.jpg)

脚本和可执行文件拷贝至~/near/bin/,运行节点：

    ./start.sh

查看日志：

![](file:///C:\Users\ADMINI~1\AppData\Local\Temp\ksohtml\wps9618.tmp.jpg)

### 激活节点

    export NEAR_ENV=shardnet
    near login

![](file:///C:\Users\ADMINI~1\AppData\Local\Temp\ksohtml\wps9877.tmp.jpg)

    near generate-key sender-stakewar.factory.shardnet.near

![](file:///C:\Users\ADMINI~1\AppData\Local\Temp\ksohtml\wps9878.tmp.jpg)

    cp ~/.near-credentials/shardnet/sender-stakewar.shardnet.near.json ~/.near/validator_key.json
    cd ~/.near/

validator_key.json中：
编辑“account_id” => sender-stakewar.factory.shardnet.near，且更改private_key为secret_key

    {
	    "account_id": "sender-stakewar.factory.shardnet.near",
    	"public_key": "ed25519:J1BPXgmUXdCyB68nZhzZriLczz5Vf67en28uHwxMTWp9",
    	"secret_key": "ed25519:****"
    }

## 挂载质押池
主账号及质押池分别为： sender-stakewar.shardnet.near， sender-stakewar.factory.shardnet.near

可查看（https://explorer.shardnet.near.org/nodes/validators）

### 部署质押池

    export NEAR_ENV=shardnet
    near call factory.shardnet.near create_staking_pool '{"staking_pool_id":"sender-stakewar", "owner_id": "sender-stakewar.shardnet.near", "stake_public_key":"  ed25519:J1BPXgmUXdCyB68nZhzZriLczz5Vf67en28uHwxMTWp9", "reward_fee_fraction":{"numerator": 5, "denominator": 100},"code_hash":"DD428g9eqLL8fWUxv8QSpVFzyHi1Qd16P8ephYCTmMSZ"}' --accountId="sender-stakewar.shardnet.near" --amount=30 --gas=300000000000000

![](file:///C:\Users\ADMINI~1\AppData\Local\Temp\ksohtml\wps112A.tmp.jpg)

### NEAR 充值和质押

    near state sender-stakewar.shardnet.near
    
    near call sender-stakewar.factory.shardnet.near deposit_and_stake --amount 6000 --accountId sender-stakewar.shardnet.near --gas=300000000000000

### 取消质押

    near call sender-stakewar.factory.shardnet.near unstake '{"amount": "10"}' --accountId sender-stakewar.shardnet.near --gas=300000000000000

### 通过ping 发起新提案，并更新委托人的质押余额

    near call sender-stakewar.factory.shardnet.near ping '{}' --accountId sender-stakewar.shardnet.near --gas=300000000000000

![](file:///C:\Users\ADMINI~1\AppData\Local\Temp\ksohtml\wps112B.tmp.jpg)

## 检查节点
### 查看⽇志

    journalctl -n 100 -f -u neard | ccze -A

### 检查状态

    sudo apt install curl jq
    curl -s http://127.0.0.1:3030/status | jq .version

![](file:///C:\Users\ADMINI~1\AppData\Local\Temp\ksohtml\wpsF9B1.tmp.jpg)

### 检查委托⼈和质押命令

    near view sender-stakewar.factory.shardnet.near get_accounts '{"from_index": 0,"limit": 10}' --accountId sender-stakewar.shardnet.near

![](file:///C:\Users\ADMINI~1\AppData\Local\Temp\ksohtml\wpsF9B2.tmp.jpg)

### ⽹络监控

[http://34.145.111.248:3030/status](http://34.145.111.248:3030/status)

![](file:///C:\Users\ADMINI~1\AppData\Local\Temp\ksohtml\wpsF9C3.tmp.jpg)
