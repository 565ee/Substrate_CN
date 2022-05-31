# 目录  
• [什么是Substrate？](#index1)  
• [安装 Rust 和 Rust 工具链](#index2)  
• [使用节点模板准备一个 Substrate 节点](#index3)  
• [安装前端模板](#index4)  
• [启动本地 Substrate 节点](#index5)  
• [启动前端模板](#index6)  
• [将资金从一个帐户转移到另一个帐户](#index7)  
• [停止本地节点](#index8)  
• [Substrate Tutorials , Substrate 教程](#index98)  
• [Contact 联系方式](#index99)  

# <span id='index1'>• 什么是Substrate？</span>  
Substrate 是一个用于构建区块链的开源、模块化和可扩展框架。

Substrate 的设计从一开始就具有灵活性，并允许创新者设计和构建满足其需求的区块链网络。它提供了构建自定义区块链节点所需的所有核心组件。

为了帮助您入门，Substrate 开发者中心提供了一个开箱即用的基于 Substrate 的节点模板。无需进行任何更改，您就可以使用此节点模板创建具有一些预定义用户帐户和资金的工作区块链网络。

# <span id='index2'>• 安装 Rust 和 Rust 工具链</span>  
手动安装和配置 Rust：

1. rustup通过运行以下命令进行安装：
```
curl https://sh.rustup.rs -sSf | sh
```

2. bin通过运行以下命令，配置当前 shell 以重新加载 PATH 环境变量，使其包含 Cargo目录：
```
source ~/.cargo/env
```

3. stable通过运行以下命令将 Rust 工具链配置为默认为最新版本：
```
rustup default stable
rustup update
```

4. 通过运行以下命令添加nightly版本和nightlyWebAssembly ( ) 目标：wasm
```
rustup update nightly
rustup target add wasm32-unknown-unknown --toolchain nightly
```

5. 通过运行以下命令来验证您的安装：
```
rustc --version
rustup show
```

前面的步骤将引导您完成 Rust 和 Rust 工具链的安装和配置，以便您自己看到完整的过程。

# <span id='index3'>• 使用节点模板准备一个 Substrate 节点</span>  
这基板节点模板 提供了一个工作开发环境，以便您可以立即开始在 Substrate 上构建。

编译 Substrate 节点模板：

1. latest通过运行以下命令，使用版本分支克隆节点模板存储库：
```
git clone https://github.com/substrate-developer-hub/substrate-node-template
```

2. 通过运行以下命令切换到节点模板目录的根目录：
```
cd substrate-node-template
git checkout latest
```

3. 通过运行以下命令编译节点模板：
```
cargo build --release
```
您应该始终使用该--release标志来构建优化的工件。

# <span id='index4'>• 安装前端模板</span>  
1. 通过运行以下命令克隆前端模板存储库：
```
git clone https://github.com/substrate-developer-hub/substrate-front-end-template
```

2. 通过运行以下命令切换到前端模板目录的根目录：
```
cd substrate-front-end-template
git checkout latest
```

3. 通过运行以下命令安装前端模板的依赖项：
```
yarn install
```

# <span id='index5'>• 启动本地 Substrate 节点</span>  
1. 切换到编译 Substrate 节点模板的根目录。
通过运行以下命令以开发模式启动节点：
```
./target/release/node-template --dev
```

2. 通过查看终端中显示的输出来验证您的节点是否已启动并成功运行。

终端应显示类似于此的输出：
![本地节点](https://user-images.githubusercontent.com/28084126/170838892-b26eb0b1-8cdf-438c-9c1a-66d6bdc97cb6.png)

如果之后的数字在finalized增加，则您的区块链正在生成新块并就它们所描述的状态达成共识。

我们将在后面的教程中查看日志输出中报告的详细信息。目前，只需要知道您的节点正在运行并生成块即可。

3. 保持显示节点输出的终端打开以继续。

# <span id='index6'>• 启动前端模板</span>  
Substrate 前端模板由用户界面组件组成，使您能够与 Substrate 节点交互并执行一些常见任务。

要使用前端模板：

1. 在您的计算机上打开一个新的终端 shell，切换到安装前端模板的根目录。

2. 通过运行以下命令启动前端模板：
```
yarn start
```

3. 在浏览器中打开http://localhost:8000查看前端模板。

顶部有一个账户选择列表，用于在您想要执行链上操作时选择要使用的账户。模板的顶部还显示有关您连接到的链的信息。
![账号](https://user-images.githubusercontent.com/28084126/170839024-526f1200-9cc3-4cb1-9b92-1a01c5db60f1.png)

# <span id='index7'>• 将资金从一个帐户转移到另一个帐户</span>  
既然您在本地计算机上运行了一个区块链节点，并且您有一个可用于执行链上操作的前端模板，那么您就可以探索与区块链交互的不同方式了。

默认情况下，前端模板包含多个组件，可让您尝试不同的常见任务。对于本教程，您可以执行简单的转账操作，将资金从一个账户转移到另一个账户。

将资金转入账户：
![Transfer](https://user-images.githubusercontent.com/28084126/170839086-7fe4bd51-5169-41bb-a1a4-14b8a4c2cf83.png)

# <span id='index8'>• 停止本地节点</span> 
传输成功后，您可以继续探索前端模板组件或停止本地 Substrate 节点。因为您--dev在启动节点时指定了该选项，所以停止本地节点会停止区块链并清除所有持久块数据，以便您下次启动节点时可以以干净的状态开始。

停止本地 Substrate 节点：

1. 返回到显示节点输出的终端 shell。

2. 按 Control-c 终止正在运行的进程。

3. 验证您的终端返回到substrate-node-template目录中的终端提示符。

# <span id='index98'>• Substrate Tutorials , Substrate 教程</span>  
CN 中文 Github  [Substrate 教程 : github.com/565ee/Substrate_CN](https://github.com/565ee/Substrate_CN)  
CN 中文 CSDN    [Substrate 教程 : blog.csdn.net/wx468116118](https://blog.csdn.net/wx468116118/category_11846056.html)  
EN 英文 Github  [Substrate Tutorials : github.com/565ee/Substrate_EN](https://github.com/565ee/Substrate_EN)  
EN 英文 dev.to  [Substrate Tutorials : dev.to/565ee](https://dev.to/565ee/substrate-tutorials-5n4) 

# <span id='index99'>• Contact 联系方式</span>  
Homepage : [565.ee](https://565.ee)  
微信公众号 : wx468116118  
微信 QQ   : 468116118  
GitHub   : [github.com/565ee](https://github.com/565ee)   
CSDN     : [blog.csdn.net/wx468116118](https://blog.csdn.net/wx468116118)  
Email    : 468116118@qq.com  
![微信公众号565 ee](https://user-images.githubusercontent.com/28084126/171232270-c26800f2-fc8b-403c-964b-5b6cda86d21d.png)
