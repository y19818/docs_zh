---
id: evm-simple
title: A simple EVM script and NEAR CLI interaction
sidebar_label: Simple EVM Script
---

### 内容:
1. <a href="#evm-simple">`evm-simple`例子</a>
2. 用<a href="#near-cli">NEAR CLI</a>完成EVM合约交互

---

<h3 id="evm-simple">`evm-simple` 例子</h3>

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/WWR_Xi7cJYY" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe><br/>

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/HE5N-aFe6ek" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe><br/>

下面的例子使用了部署在betanet上EVM的现有智能合约，并进行了简单的读写操作。欢迎关注<a href="https://github.com/near-examples/evm-simple" target="_blank">这个资源库</a>。它使用了两个现有的NEAR betanet账户，所以你不需要创建一个。即将到来的一节将介绍如何从Solidity构建和部署合同，以及如何从您自己的betanet帐户进行部署，但我们现在将保持这个尽可能的容易。对于这个 "evm-simple "的例子，我们将与NEAR宠物店的一个版本进行交互，这是一个基于<a href="https://www.trufflesuite.com/tutorials/pet-shop" target="_blank">Truffle starter项目</a>的dApp。在这个应用程序中，有两个功能:

1. `getAdopters()` — 返回一个由16个Ethereum地址组成的数组，对应16只被领养或需要领养的宠物。例如，当用户领养第一只宠物时，我们可以期望从这个函数返回的值中得到他们的Ethereum地址，这个地址是返回的数组中16个项目中的第一个。如果宠物没有被领养，则使用<a href="https://ethereum.org/en/glossary/#zero-address" target="_blank">零地址</a>。这个函数是只读的。
2. `adopt(uint petId)` — 这将宠物的领养分配给发送交易的Ethereum地址，转变（修改）状态。

这个项目使用了两个npm包: <a href="https://www.npmjs.com/package/web3-eth-contract" target="_blank">web3-eth-contract</a> 和 <a href="https://www.npmjs.com/package/near-web3-provider" target="_blank">near-web3-provider</a>. 前者是Web3中常见的依赖包，它与后者很好地结合在一起[在这里详细描述](/docs/develop/evm/near-web3-provider).

整个文件有42行，对新人很友好。在下面的截图中，为了便于阅读，函数`callEvm`被折叠了。文件的最后一行调用`start`，它将发送两个近似betanet的账户名到`callEvm`。

<a href="https://github.com/near-examples/evm-simple/blob/0bef42aab9f7386269a9a5c17860d37530fc42da/index.js#L4-L9" target="_blank"><img loading="lazy" src="/docs/assets/evm/evm-simple-callEvm-collapsed-1024x482.jpg" alt="EVM Simple example showing that mike.betanet and josh.betanet will be calling the same function back-to-back" width="1024" height="482" srcset="/docs/assets/evm/evm-simple-callEvm-collapsed-1024x482.jpg 1024w, /docs/assets/evm/evm-simple-callEvm-collapsed-300x141.jpg 300w, /docs/assets/evm/evm-simple-callEvm-collapsed-768x362.jpg 768w, /docs/assets/evm/evm-simple-callEvm-collapsed-1536x723.jpg 1536w, /docs/assets/evm/evm-simple-callEvm-collapsed-2048x964.jpg 2048w" sizes="(max-width: 1024px) 100vw, 1024px"></a>

✋ **快速说明**: 本页所有的代码截图都在最新提交时链接到GitHub项目

现在让我们看看扩展后的`callEvm`函数，并讨论高亮的部分:

<a href="https://github.com/near-examples/evm-simple/blob/0bef42aab9f7386269a9a5c17860d37530fc42da/index.js#L11-L41" target="_blank"><img loading="lazy" src="/docs/assets/evm/evm-simple-with-sections-numbers-improved-1024x978.jpg" alt="The evm-simple example highlighting various sections" width="1024" height="978" srcset="/docs/assets/evm/evm-simple-with-sections-numbers-improved-1024x978.jpg 1024w, /docs/assets/evm/evm-simple-with-sections-numbers-improved-300x286.jpg 300w, /docs/assets/evm/evm-simple-with-sections-numbers-improved-768x733.jpg 768w, /docs/assets/evm/evm-simple-with-sections-numbers-improved-1536x1467.jpg 1536w, /docs/assets/evm/evm-simple-with-sections-numbers-improved-2048x1956.jpg 2048w" sizes="(max-width: 1024px) 100vw, 1024px"></a>

分解每个步骤:

1. 实例化`NearProvider`:
    - `networkId`与你想交互的NEAR网络相关联。(请参见NEAR Web3 Provider's中的网络列表和合理的默认值。 <a href="https://github.com/near/near-web3-provider/blob/88a62702aea31bffa372ffc450cfb78ffb0b0082/src/network-config.js" target="_blank">`network-config.js` file</a>.)
    - 对于已签署的交易，`masterAccountId`，即该网络上将签署交易的NEAR账户的名称。如果提供者只需要读取状态，则应该省略`masterAccountId`，加入`isReadOnly: true`代替。(我们在[宠物店文档]中介绍了一个例子。(/docs/develop/evm/near-pet-shop).)  <a href="/docs/develop/evm/near-web3-provider#instantiating-read-only" target="_blank">更多内容在这文档中</a>.
    - `keyStore`作为存放私钥的位置。根据上面那行代码的注释，NEAR的库<a href="https://github.com/near/near-api-js" target="_blank">near-api-js</a>（由NEAR Web3提供者使用）有不同类型的密钥存储。在这个例子中，有位于项目的`private-keys`目录中的密钥。有关这方面的更多信息，请参见<a href="https://github.com/near-examples/evm-simple#private-keys" target="_blank">相应的README</a>。(注意：在[NEAR宠物店文档](/docs/develop/evm/near-pet-shop)中，前端将为浏览器使用不同类型的密钥存储。)
2. 实例化智能合约并设置提供者。
3. 得出与NEAR账户相关联的Ethereum地址。这使用了一个小的帮助函数`nearAccountToEvmAddress`，它只需要20个字节的哈希NEAR账户名，<a href="https://github.com/near/near-web3-provider/blob/88a62702aea31bffa372ffc450cfb78ffb0b0082/src/utils.js#L298" target="_blank">这里</a>所示。
4. 像平常一样使用Web3发送一个交易。这里我们从派生的Ethereum地址中调用函数`adopt`，表示要收养第一只宠物，因此数组中该索引的参数为`0`。
5. 调用合约的只读函数`getAdopters`。步骤1中实例化的提供者可以签署交易，也可以调用只读函数。

最后，<a href="https://github.com/near-examples/evm-simple" target="_blank">按照README</a>中的说明运行这个项目。你会看到这个简单的NodeJS应用将第一只宠物的领养者设置为`mike.betanet`，显示被领养的宠物列表，然后让`josh.betanet`领养第一只宠物并再次显示结果。

<h3 id="near-cli">NEAR CLI 命令</h3>

让我们来演示一下如何使用终端或命令提示符与之前的同一合约进行交互。

NEAR CLI是一个用JavaScript编写的命令行界面工具。它使用 <a href="https://github.com/near/near-api-js" target="_blank">near-api-js</a>进行 RPC 调用，提供实用功能，并普遍简化了部署和在与合约和 NEAR 账户交互的体验。在[该工具的文档](/docs/tools/near-cli)中有更多的信息，但假设你<a href="https://nodejs.org/en/download/package-manager/" target="_blank">拥有Node 12+</a>，你可以在全局范围内用以下方法安装它:

    npm install -g near-cli

在前面的`evm-simple`例子中，有一个收养合约的工件。这是在编译合约时创建的，位于`build/contracts`目录中。它包含了调用函数和处理其返回值所需的应用程序二进制接口（ABI）信息、合约数据以及其他细节，包括这个合约的部署位置。让我们看一下工件文件的底部，<a href="https://github.com/near-examples/evm-simple/blob/0bef42aab9f7386269a9a5c17860d37530fc42da/build/contracts/Adoption.json" target="_blank">Adoption.json</a>。:

<a href="https://github.com/near-examples/evm-simple/blob/0bef42aab9f7386269a9a5c17860d37530fc42da/build/contracts/Adoption.json#L1193-L1200" target="_blank"><img loading="lazy" src="/docs/assets/evm/adoption-artifact-improved-1024x257.jpg" alt="Code snippet of the networks key showing where the Adoption smart contract has been deployed to: what chain id and what Ethereum address" width="1024" height="257" srcset="/docs/assets/evm/adoption-artifact-improved-1024x257.jpg 1024w, /docs/assets/evm/adoption-artifact-improved-300x75.jpg 300w, /docs/assets/evm/adoption-artifact-improved-768x193.jpg 768w, /docs/assets/evm/adoption-artifact-improved-1536x385.jpg 1536w, /docs/assets/evm/adoption-artifact-improved-2048x514.jpg 2048w" sizes="(max-width: 1024px) 100vw, 1024px"></a>

这里有两点需要注意:

1. 链ID为`1313161554`。这已在<a href="https://chainid.network/" target="_blank">https://chainid.network</a>注册为NEAR（目前需要做一些小的改动，以区分testnet和主网链ID）。
2. `address`键包含所部署的领养合约的Ethereum地址: `0xAdf11a39283CEB00DEB90a5cE9220F89c6C27E67`

让我们用NEAR CLI来调用只读函数`getAdopters`:

    NEAR_ENV=betanet near evm-view evm 0xAdf11a39283CEB00DEB90a5cE9220F89c6C27E67 getAdopters '[]' --abi /path/to/build/contracts/Adoption.json --accountId mike.betanet

如果这个命令被翻译成中文，它将会说,

“嘿，NEAR CLI，在部署到账户`evm`的EVM合约上使用命令`evm-view`，调用位于地址`0xAdf...E67`的Ethereum智能合约，给函数`getAdopters`空参数。哦对了，这里就是你可以找到包含ABI的地方。”

运行该命令后，你会看到类似这样的内容:

<img loading="lazy" src="/docs/assets/evm/Screen-Shot-2021-01-01-at-11.07.17-AM-1024x1016.png" alt="The result returned from calling the Adoption smart contract's getAdopters() function in Terminal" width="504" height="500" srcset="/docs/assets/evm/Screen-Shot-2021-01-01-at-11.07.17-AM-1024x1016.png 1024w, /docs/assets/evm/Screen-Shot-2021-01-01-at-11.07.17-AM-300x298.png 300w, /docs/assets/evm/Screen-Shot-2021-01-01-at-11.07.17-AM-150x150.png 150w, /docs/assets/evm/Screen-Shot-2021-01-01-at-11.07.17-AM-768x762.png 768w, /docs/assets/evm/Screen-Shot-2021-01-01-at-11.07.17-AM.png 1522w" sizes="(max-width: 504px) 100vw, 504px">

数组中的项目地址`0x5d60a489b2f457cb351b0faabf5f9746d6bd4a8c`。这是从NEAR账户`josh.betanet`中推导出来的Ethereum地址，通过一个快速的NodeJS <a href="https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop" target="_blank">REPL</A>技巧来证明。:

<img loading="lazy" src="/docs/assets/evm/josh.betanet-eth-address-1024x442.jpg" alt="Use NodeJS's REPL, we show that josh.betanet is the Ethereum address of 0x5d60a489b2f457cb351b0faabf5f9746d6bd4a8c" width="813" height="351" srcset="/docs/assets/evm/josh.betanet-eth-address-1024x442.jpg 1024w, /docs/assets/evm/josh.betanet-eth-address-300x130.jpg 300w, /docs/assets/evm/josh.betanet-eth-address-768x332.jpg 768w, /docs/assets/evm/josh.betanet-eth-address-1536x663.jpg 1536w, /docs/assets/evm/josh.betanet-eth-address.jpg 1556w" sizes="(max-width: 813px) 100vw, 813px">

(提醒一下，Ethereum地址是不区分大小写的，所以不要被这里显示的以`...4A8C`结尾的两个地址与`...4a8c`混淆。大写只用<a href="https://eips.ethereum.org/EIPS/eip-55" target="_blank">作为一个精巧的校验</a>，帮助用户减少错误。)

数组中的第二项都是0，意味着这个 "宠物 "还没有被领养，所以我们使用NEAR CLI从NEAR账户`mike.betanet`中发送一个签名交易，在没有Web界面的情况下手动领养这个宠物。我们之所以能做到这一点，是因为`mike.betanet`的私钥是一个特殊的、函数调用的访问密钥，这意味着NEAR代币（Ⓝ）不能被转移，也不能删除账户等。

    NEAR_ENV=betanet near evm-call evm 0xAdf11a39283CEB00DEB90a5cE9220F89c6C27E67 adopt '["1"]' --abi /path/to/build/contracts/Adoption.json --accountId mike.betanet

运行该命令后，我们再重新运行之前的 "evm-view "命令的 "getAdopters"，可以看到数组中的第二项已经更新为Ethereum地址的 "mike.betanet"。:

<img loading="lazy" src="/docs/assets/evm/Screen-Shot-2021-01-01-at-2.57.49-PM-1024x318.png" alt="Screenshot showing output highlighting the second pet ID is now adopted by the Ethereum address for mike.betanet" width="1024" height="318" srcset="/docs/assets/evm/Screen-Shot-2021-01-01-at-2.57.49-PM-1024x318.png 1024w, /docs/assets/evm/Screen-Shot-2021-01-01-at-2.57.49-PM-300x93.png 300w, /docs/assets/evm/Screen-Shot-2021-01-01-at-2.57.49-PM-768x238.png 768w, /docs/assets/evm/Screen-Shot-2021-01-01-at-2.57.49-PM.png 1528w" sizes="(max-width: 1024px) 100vw, 1024px">

我们已经演示了如何使用NEAR CLI来访问和转变状态。新增的三个与EVM相关的命令是:

1. [near evm-view](/docs/tools/near-cli#near-evm-view)
2. [near evm-call](/docs/tools/near-cli#near-evm-call)
3. [near evm-init-dev](/docs/tools/near-cli#near-evm-dev-init) — 这将在 [EVM 测试部分](/docs/develop/evm/evm-testing)中讨论这个问题。
