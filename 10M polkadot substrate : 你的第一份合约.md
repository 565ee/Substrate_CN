• [介绍](#index1)  
• [教程目标](#index2)  
• [更新你的 Rust 环境](#index3)  
• [安装 Substrate 合约节点](#index4)  
• [创建一个新的智能合约项目](#index5)  
• [测试默认合约](#index6)  
• [建立合同](#index7)  
• [启动 Substrate 智能合约节点](#index8)  
• [部署合约](#index9)  
• [在区块链上创建实例](#index10)  
• [调用智能合约](#index11)  
• [Substrate Tutorials , Substrate 教程](#index98)  
• [Contact 联系方式](#index99) 

# <span id='index1'>• 介绍</span>  
正如您在区块链基础知识中了解到的那样，去中心化应用程序最常被编写为智能合约。尽管 Substrate 主要是用于构建自定义区块链的框架和工具包，但它也可以为智能合约提供平台。本教程演示了如何构建一个基本的智能合约以在基于 Substrate 的链上运行。在本教程中，您将探索使用墨水！作为编写基于 Rust 的智能合约的编程语言。

# <span id='index2'>• 教程目标</span>  
通过完成本教程，您将实现以下目标：

了解如何创建智能合约项目。
使用墨水构建和测试智能合约！智能合约语言。
在本地 Substrate 节点上部署智能合约。
通过浏览器与智能合约交互。

# <span id='index3'>• 更新你的 Rust 环境</span>  
对于本教程，您需要将一些 Rust 源代码添加到您的 Substrate 开发环境中。

要更新您的开发环境：

• 在您的计算机上打开终端外壳。
• 切换到编译 Substrate 节点模板的根目录。
• 通过运行以下命令更新您的 Rust 环境：
```
rustup component add rust-src --toolchain nightly
```

• 通过运行以下命令验证您是否安装了 WebAssembly 目标：
```
rustup target add wasm32-unknown-unknown --toolchain nightly
```

# <span id='index4'>• 安装 Substrate 合约节点</span>  
为了简化本教程，您可以下载适用于 Linux 或 macOS 的预编译 Substrate 节点。默认情况下，预编译的二进制文件包括智能合约的 FRAME 托盘。或者，您可以通过在本地计算机上contracts-node运行来手动构建预配置。cargo install contracts-node

如果无法下载预编译的节点，可以使用类似如下的命令在本地编译：
```
cargo install contracts-node --git https://github.com/paritytech/substrate-contracts-node.git --tag <latest-tag> --force --locked
```

# <span id='index5'>• 创建一个新的智能合约项目</span>  
您现在已准备好开始开发新的智能合约项目。

为智能合约项目生成文件：

• 在您的计算机上打开终端外壳。
• flipper通过运行以下命令创建一个名为的新项目文件夹：
```
cargo contract new flipper
```

• 通过运行以下命令切换到新的项目文件夹：
```
cd flipper/
```

# <span id='index6'>• 测试默认合约</span>  
在lib.rs源代码文件的底部，有简单的测试用例来验证合约的功能。您可以使用链下测试环境测试此代码是否按预期运行。

测试合约：

• 如果需要，在您的计算机上打开终端外壳。
• 如果需要，请验证您是否位于flipper项目文件夹中。
• 通过运行以下命令，使用test子命令和nightly工具链执行合约的默认测试：flipper
```
cargo +nightly test
```

该命令应显示类似于以下内容的输出，以指示测试成功完成：
```
running 2 tests
test flipper::tests::it_works ... ok
test flipper::tests::default_works ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

# <span id='index7'>• 建立合同</span>  
测试默认合约后，您就可以将此项目编译为 WebAssembly。

为这个智能合约构建 WebAssembly：

• 如果需要，在您的计算机上打开终端外壳。
• 验证您是否位于flipper项目文件夹中。
• flipper通过运行以下命令编译智能合约：
```
cargo +nightly contract build
```

此命令为项目构建 WebAssembly 二进制flipper文件、包含合约应用程序二进制接口 (ABI) 的元数据文件以及.contract用于部署合约的文件。例如，您应该看到类似于以下的输出：

# <span id='index8'>• 启动 Substrate 智能合约节点</span>  
如果您已成功安装substrate-contracts-node，您可以为您的智能合约启动一个本地区块链节点。

要启动预配置contracts-node：

• 如果需要，在您的计算机上打开终端外壳。
• 通过运行以下命令以本地开发模式启动contracts节点：
```
substrate-contracts-node --dev
```

• 在 Web 浏览器中导航到Contracts UI ，然后单击Yes allow this application access。
• 选择本地节点。

# <span id='index9'>• 部署合约</span>  
然而，在 Substrate 上部署智能合约与在传统智能合约平台上部署有点不同。对于大多数智能合约平台，您必须在每次进行更改时部署全新的智能合约源代码块。例如，标准 ERC20 代币已被部署到以太坊数千次。即使更改很小或仅影响某些初始配置设置，每次更改都需要完全重新部署代码。每个智能合约实例消耗相当于完整合约源代码的区块链资源，即使实际上没有更改任何代码。

在 Substrate 中，合约部署过程分为两个步骤：

• 将合约代码上传到区块链。
• 创建合约实例。

上传合约代码
在本教程中，您将使用 Contracts UI 前端flipper在 Substrate 链上部署合约。

上传智能合约源代码：

• 在 Web 浏览器中打开合同 UI 。
• 确认您已连接到本地节点。
• 单击添加新合同。
• 点击上传新合同代码。
• 选择用于创建合约实例的账户。

您可以选择任何现有帐户，包括预定义帐户，例如alice.

• 输入智能合约的描述性名称，例如 Flipper Contract。
• 浏览并选择或拖放flipper.contract包含捆绑的 Wasm blob 和元数据的文件到上传部分。
![image](https://user-images.githubusercontent.com/28084126/178223817-b779619e-084e-4bcd-9556-d1132afb8f6c.png)

# <span id='index10'>• 在区块链上创建实例</span>  
智能合约作为 Substrate 区块链上账户系统的扩展存在。当您创建此智能合约的实例时，Substrate 会创建一个新AccountId的来存储智能合约管理的任何余额并允许您与合约进行交互。

上传智能合约并单击Next后，Contracts UI 会显示有关智能合约内容的信息。

创建实例：

• 查看并接受智能合约初始版本的默认部署构造函数选项。
• 查看并接受默认的Max Gas Allowed。200000
![image](https://user-images.githubusercontent.com/28084126/178224140-148ec15a-da46-42e8-9e1d-172923214829.png)

• 单击下一步。

事务现在已排队。如果您需要进行更改，您可以单击返回来修改输入。
![image](https://user-images.githubusercontent.com/28084126/178224243-f2d6944e-7222-4373-9522-765a642ca884.png)

• 单击上传并实例化。

根据您使用的帐户，系统可能会提示您输入帐户密码。如果您使用预定义帐户，则无需提供密码。
![image](https://user-images.githubusercontent.com/28084126/178224330-f7e211b8-5240-40b6-b156-6ac6739f237a.png)

# <span id='index11'>• 调用智能合约</span>  
现在您的合约已经部署在区块链上，您可以与它进行交互。默认的 Flipper 智能合约有两个功能-flip()并且get()- 您可以使用 Contracts UI 来试用它们。

获取（）函数
您将合约的初始值设置为实例化合约时flipper的值。您可以使用该函数来验证当前值为。valuefalseget()false

测试get()功能：

• 从帐户列表中选择任何帐户。

• 本合同没有限制谁可以发送get()请求。

• 从Message to Send列表中选择get(): bool 。
• 点击阅读。
• 验证false调用结果中是否返回了该值。
![image](https://user-images.githubusercontent.com/28084126/178225250-a54f0a44-27c0-4a7b-aac6-a189c290fd89.png)

测试 flip（）函数
该flip()函数将值从 更改false为true。

测试flip()功能：

• 从帐户列表中选择任何预定义的帐户。

• 该flip()函数是一个改变链状态的交易，需要一个有资金的账户来执行调用。因此，您应该选择具有预定义帐户余额的帐户，例如alice帐户。

• 从要发送的消息列表中选择翻转（） 。
• 单击呼叫。
• 在调用结果中验证事务是否成功。
• 验证新值是否true在调用结果中。
![image](https://user-images.githubusercontent.com/28084126/178225538-b3e30704-3391-4f87-aa99-a9f0a6d0c7f1.png)

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
![微信公众号565 ee](https://user-images.githubusercontent.com/28084126/178225732-a7858aae-de14-46ed-b60a-06e955a854f5.png)
