---
id: evm-local-setup
title: EVM local setup
sidebar_label: Local Setup
---

---
id: evm-local-setup
title: EVM local setup
sidebar_label: Local Setup

---

本节介绍如何在本地节点上运行NEAR EVM。注意，这比使用测试网需要更长的时间和更多的资源。这是为那些喜欢在本地开发的人准备的。请注意，目前这些说明是原始的。我们的目标是将这个过程Docker化，但它是为勇敢的构建者提供的。

## 目录

- [目录](#table-of-contents)
  - [前提条件](#prerequisites)
  - [获取Rust](#get-rust)
  - [设置NEAR节点](#set-up-near-node)
  - [Postgres](#postgres)
  - [NEAR合约助手](#near-contract-helper)
  - [NEAR CLI](#near-cli)
  - [NEAR Linkdrop](#near-linkdrop)
  - [NEAR钱包](#near-wallet)
  - [NEAR浏览器](#near-explorer)
  - [复制钥匙到宠物店](#copy-key-to-pet-shop)
  - [本地网络迁移](#localnet-migration)
  - [笔记](#notes)
  - [疑难解答](#troubleshooting)

---

### 前提条件

- [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
- [Docker](https://docs.docker.com/get-docker/).
- NodeJS version 12+. 我们建议使用[版本管理器](https://nodejs.org/en/download/package-manager/).
- [Python 3](https://www.python.org/download) 和 [pip3 package](https://pip.pypa.io/en/stable/installing/)
- [Postgres](https://wiki.postgresql.org/wiki/Detailed_installation_guides)
- Rust (请看下面的说明)

### 获取Rust

    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

按照该命令后:

    rustup target add wasm32-unknown-unknown

**注**: 这个自定义目标并不是严格意义上运行NEAR节点所需要的，而是在Rust中构建智能合约所需的。在本指南的后面将需要它。  

另外，[参见文档](/docs/tutorials/contracts/intro-to-rust#3-step-rust-installation)可以获得更多安装Rust的链接和资源。

### 设置NEAR节点

克隆[nearcore库](https://github.com/near/nearcore):

    git clone https://github.com/near/nearcore.git

导航到项目根目录:

    cd nearcore

构建(这将需要一段时间，在这期间，你可以自由移动到之后的步骤。):

    cargo build -p neard --release --features protocol_feature_evm,nightly_protocol_features

当构建完成时，初始化:

    ./target/release/neard --home=$HOME/.near/local init

然后运行:

    ./target/release/neard --home=$HOME/.near/local run

**注**: 按Ctrl + C停止本地节点。如果你想从断开的地方开始，只需再次使用最后的 "run "命令。如果你想从零开始，请删除该文件夹。

    rm -rf ~/.near/local

然后再运行"初始化"和"run"命令。

### Postgres

确保Postgres已经安装在你的机器上。不同的操作系统和设置会有不同的说明。如前所述，这里有一个[详细安装指南](https://wiki.postgresql.org/wiki/Detailed_installation_guides)的链接。

Mac用户可以考虑:

    brew install postgresql

或者获取[Postgres app](https://postgresapp.com/).

**注**:如果数据库出现问题，请仔细阅读[the README](https://github.com/near/near-contract-helper#create-database)的 `near-contract-helper` 

一旦Postgres安装了,输入:

    psql

确保它给你一个提示。如果没有，请按照屏幕上的指示操作。一旦出现预期的提示，你就可以用以下方式退出:

    \q

### NEAR合约助手

克隆[NEAR合约助手库](https://github.com/near/near-contract-helper)

    git clone https://github.com/near/near-contract-helper.git
    cd near-contract-helper

在你的终端/命令提示符下:

    psql

在Postgres提示下:

    create user helper with superuser password 'helper';
    create database accounts_development with owner=helper;

然后:

    yarn migrate
    # or
    npm run migrate

在项目根目录下添加一个名为`.env`的文件，内容如下。我们稍后将对其进行修改:

```dotenv
ACCOUNT_CREATOR_KEY={"account_id":"test.near","public_key":"ed25519:7PGseFbWxvYVgZ89K1uTJKYoKetWs7BJtbyXDzfbAcqX","secret_key":"ed25519:3D4YudUQRE39Lc4JHghuB5WM8kbgDDa34mnrEP5DdTApVH81af7e2dWgNPEaiQfdJnZq1CNPp5im4Rg5b733oiMP"}
MAIL_HOST=smtp.ethereal.email
MAIL_PASSWORD=
MAIL_PORT=587
MAIL_USER=
NEW_ACCOUNT_AMOUNT=10000000000000000000000000
NODE_ENV=development # Node.js environment; either `development` or `production`
NODE_URL=http://127.0.0.1:3030 # from ~/.near/config.json#rpc.addr – for production, use https://rpc.testnet.near.org
PORT=3000 # Used internally by the contract helper; does not have to correspond to the external IP or DNS name and can link to a host machine running the Docker container
TWILIO_ACCOUNT_SID= # account SID from Twilio (used to send security code)
TWILIO_AUTH_TOKEN= # auth token from Twilio (used to send security code)
TWILIO_FROM_PHONE=+15553455 # phone number from which to send SMS with security code (international format, starting with `+`)
WALLET_URL=http://127.0.0.1:4000
INDEXER_DB_CONNECTION=postgres://helper:helper@127.0.0.1/near_indexer_for_wallet_localnet?ssl=require
```

我们将改变环境变量列表中的第一个键(`ACCOUNT_CREATOR_KEY`)，使之成为你的设置中唯一的键。请注意，该键中没有空格，即使是在冒号和逗号之后。这一点很重要，因为我们要添加你的本地节点的私钥。

正如前面在 "nearup "部分提到的，我们有本地网账户**test.near**的私钥。用你喜欢的编辑器，打开文件: 

    ~/.near/local/validator_key.json

去掉所有空格和换行符，使私钥存在于一行。(另外，请看上面代码块中的例子。)现在用你的私钥替换密钥`ACCOUNT_CREATOR_KEY`的值。

最后，启动NEAR合约助手程序，使用:

    npm run start

### NEAR CLI

之后的步骤将使用NEAR命令行接口工具来部署和发送NEAR代币(Ⓝ)到一个账户。CLI的功能远不止于此，正如[这里的文档](/docs/tools/near-cli).

安装方式:

    npm i -g near-cli

### NEAR Linkdrop

有一个名为Linkdrop的智能合约，需要部署到前面步骤中启动节点后自动创建的`test.near`账户中。

克隆NEAR Linkdrop](https://github.com/near/near-linkdrop)库:

    git clone https://github.com/near/near-linkdrop.git
    cd near-linkdrop
    ./build.sh

通过观察这条命令的输出，我们可以看到没有任何合约部署到`test.near`:

    NEAR_ENV=local near state test.near

```
…
Account test.near
{
  …
  code_hash: '11111111111111111111111111111111',
  …
}
```

这里的 `code_hash` 全是“1”说明合约没有成功部署。

部署 linkdrop 合约的时候要注意:

    NEAR_ENV=local near deploy --accountId test.near --wasmFile res/linkdrop.wasm --keyPath ~/.near/local/validator_key.json

现在合约已经部署完毕。检查账户的状态（使用之前的相同命令）将显示`code_hash`为合约代码的hash值。

### NEAR钱包

克隆[NEAR钱包](https://github.com/near/near-wallet).

    git clone https://github.com/near/near-wallet.git
    cd near-wallet

在项目根目录下创建一个名为`.env.local`的文件，并粘贴其内容:

```dotenv
REACT_APP_ACCOUNT_HELPER_URL=http://localhost:3000
REACT_APP_ACCOUNT_ID_SUFFIX=test.near
REACT_APP_IS_MAINNET=false
REACT_APP_NODE_URL=http://localhost:3030
REACT_APP_ACCESS_KEY_FUNDING_AMOUNT="3000000000000000000000000"
```

然后运行这个命令:

    npm run update:static && node --max-http-header-size=16000 ./node_modules/.bin/parcel -p 4000 src/index.html

输入http://127.0.0.1:4000打开钱包

按照界面中的指示创建一个账户。当要求选择恢复选项时，选择**电子邮件恢复**选项。当它要求输入电子邮件时，请输入这个模拟电子邮件地址:

    near@example.com

一封电子邮件尚未发送，但它将出现在我们运行NEAR合约助手的终端选项卡的日志中。导航到该选项卡并搜索该电子邮件地址。在那附近会有一行字，上面写着:

>Confirm your activation code to finish creating your account

下面会出现这样的信息:

```
…
'1. Confirm your activation code to finish creating your account:\n' +
    '154005\n' +
…
```

复制代码（由很多数字组成，在上面的代码段中应该是`154005`），用它来回答钱包中的激活码提示。

下一幕要求用至少3个Ⓝ的隐性账户进行融资。复制地址和用NEAR CLI给账户注资，把下面长地址账户替换成你的账户:

    NEAR_ENV=local near send test.near f81247b9492b80284fa2d90dc498153258a6f08e39db6cfa3356abb43a515432 500 --keyPath ~/.near/local/validator_key.json

期待在界面的右上方看到你的用户 "已登录"。一个界面可能会问你是否要启用双重认证，但我们会让合约保持原样。

---

### NEAR浏览器

前提条件: [Docker](https://docs.docker.com/get-docker/).

克隆[NEAR浏览器](https://github.com/near/near-explorer):

    git clone https://github.com/near/near-explorer.git
    cd near-explorer

我们将使用Docker来运行WAMP路由:

    docker-compose build wamp
    docker-compose up -d wamp

然后进入`后台`目录，创建一个`db`子目录。

    cd back-end
    npm install
    mkdir db

用下列命令启用后端: 
    

    env NEAR_RPC_URL=http://127.0.0.1:3030 WAMP_NEAR_EXPLORER_URL=ws://localhost:8080/ws WAMP_NEAR_EXPLORER_BACKEND_SECRET=back npm run start

你可以期待看到你的屏幕重复显示日志这类的信息:

```
…
Starting regular data stats check from LEGACY_SYNC_BACKEND...
Regular data stats check from LEGACY_SYNC_BACKEND is completed.
Starting regular final timestamp check...
Regular final timestamp check is completed.
Starting regular node status check...
Regular node status check is completed.
…
```

进入`前端`目录:

    cd ..
    cd front-end
    npm install

在3019端口启动前端:

    env WAMP_NEAR_EXPLORER_URL=ws://localhost:8080/ws ./node_modules/.bin/next -p 3019

然后访问前端页面http://127.0.0.1:3019

### 复制钥匙到宠物店

最后，我们将把宠物店迁移（构建和部署）到我们的本地节点，并能够使用我们现在运行的各种服务。导航你的终端或命令提示符回到宠物店目录。

每个本地网都会有自己的唯一密钥，用于创建账户`test.near`。将该文件复制到项目中，遵循签名文件（keystore）的命名惯例:

    cp ~/.near/local/validator_key.json ./neardev/local/test.near.json

### 本地网络迁移

    npx truffle migrate --network near_local

启动网络应用:

    npm run local

打开本地网站：http://localhost:1234，开始交互和领养宠物。

### 笔记

这个存储库尽量保持与原始宠物店的相似性。主要目标是证明宠物店的Solidity合约可以使用Truffle编译和迁移，而不必为NEAR重写成更原生的智能合约语言。

另外值得注意的是，一些JavaScript文件（如`./src/js/app.js`）将与宠物店文件非常相似。这导致了jQuery和其他依赖关系的混合，但作为开发者可以很容易地比较这些代码。 

### 疑难解答

在修改Solidity代码的开发过程中，如果继续出现意想不到的情况，请考虑删除`build`文件夹并重新迁移。

如果您的本地网络遇到问题，您可以考虑删除`~/.near/local`目录，并再次运行nearcore构建命令。

如果你在运行浏览器时遇到问题，信息似乎与数据库有关，试着多运行几次命令，或者直到每次运行后出现相同日志消息的情况。

如果钱包似乎出现了问题，并且控制台日志显示了有关旧账户的消息，你可以尝试清除本地存储。在浏览器的开发者控制台中: 

    localStorage.clear()