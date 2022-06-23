• [介绍](#index1)  
• [生成自己的密钥](#index2)  
• [生成第二组密钥](#index3)  
• [修改现有链规范](#index4)  
• [将链规范转换为使用原始格式](#index5)  
• [启动第一个节点](#index6)  
• [允许其他参与者加入](#index8)  
• [Substrate Tutorials , Substrate 教程](#index98)  
• [Contact 联系方式](#index99)  

# <span id='index1'>• 介绍</span>  
本教程说明了如何使用一组私有验证器来启动一个小型的独立区块链网络。

所有区块链都要求网络中的节点就消息集及其顺序达成一致，以成功创建区块并从一个区块推进到下一个区块。每个块代表特定时间点的数据状态，节点对该状态的协议称为共识。有几种不同的算法用于达成共识，包括：

• 工作量证明共识取决于验证者节点为将有效块添加到链中所做的计算工作。

• 股权证明共识选择验证者根据他们锁定为网络中的股权的加密货币持有量将有效块添加到链中。

• 权威共识证明依赖于一组经批准的账户身份来充当验证者。与已批准账户关联的节点有权将交易放入区块中。

Substrate 节点模板使用权威证明共识模型，也称为权威轮或Aura共识。Aura 共识协议将区块生产限制在以循环方式创建区块的授权账户（权限）的轮换列表中。

在本教程中，您将看到这种共识模型在实践中是如何工作的，首先，通过使用作为节点模板一部分的预定义帐户，然后向批准集添加新权限。

# <span id='index2'>• 生成自己的密钥</span>  
现在您知道如何使用命令行选项启动和连接正在运行的节点作为对等节点，您可以生成自己的密钥，而不是使用预定义的帐户密钥。重要的是要记住，区块链网络中的每个参与者都负责生成唯一的密钥。

您已经使用了一些命令行选项来使用预定义alice和bob帐户启动本地区块链节点。您还可以使用命令行选项生成随机密钥以与 Substrate 一起使用。

作为最佳实践，当您为生产区块链生成密钥时，您应该使用从未连接到互联网的气隙计算机。至少，在生成任何打算在不受您控制的公共或私有区块链上使用的密钥之前，您应该断开互联网连接。

但是，对于本教程，您可以使用 node templatekey子命令在本地生成随机密钥并保持与 Internet 的连接。

• 在您的计算机上打开终端外壳。

• 切换到编译 Substrate 节点模板的根目录。

• 通过运行以下命令生成随机密钥短语和密钥：
```
./target/release/node-template key generate --scheme Sr25519 --password-interactive
```

• 为生成的密钥键入密码。

该命令生成密钥并显示类似于以下内容的输出：
```
Secret phrase:  pig giraffe ceiling enter weird liar orange decline behind total despair fly
Secret seed:       0x0087016ebbdcf03d1b7b2ad9a958e14a43f2351cd42f2f0a973771b90fb0112f
Public key (hex):  0x1a4cc824f6585859851f818e71ac63cf6fdc81018189809814677b2a4699cf45
Account ID:        0x1a4cc824f6585859851f818e71ac63cf6fdc81018189809814677b2a4699cf45
Public key (SS58): 5CfBuoHDvZ4fd8jkLQicNL8tgjnK8pVG9AiuJrsNrRAx6CNW
SS58 Address:      5CfBuoHDvZ4fd8jkLQicNL8tgjnK8pVG9AiuJrsNrRAx6CNW
```

• 使用您刚刚生成的帐户的密码短语使用 Ed25519 签名方案派生密钥。

例如，运行类似于以下的命令：
```
./target/release/node-template key inspect --password-interactive --scheme Ed25519 "pig giraffe ceiling enter weird liar orange decline behind total despair fly"
```

• 键入用于生成密钥的密码。

该命令显示类似于以下内容的输出：
```
Secret phrase `pig giraffe ceiling enter weird liar orange decline behind total despair fly` is account:
Secret seed:       0x0087016ebbdcf03d1b7b2ad9a958e14a43f2351cd42f2f0a973771b90fb0112f
Public key (hex):  0x2577ba03f47cdbea161851d737e41200e471cd7a31a5c88242a527837efc1e7b
Public key (SS58): 5CuqCGfwqhjGzSqz5mnq36tMe651mU9Ji8xQ4JRuUTvPcjVN
Account ID:        0x2577ba03f47cdbea161851d737e41200e471cd7a31a5c88242a527837efc1e7b
SS58 Address:      5CuqCGfwqhjGzSqz5mnq36tMe651mU9Ji8xQ4JRuUTvPcjVN
```

# <span id='index3'>• 生成第二组密钥</span>  
• 对于本教程，专用网络仅包含两个节点，因此您需要两组密钥。您有几个选项可以继续本教程：

您可以将密钥用于预定义帐户之一。

您可以使用本地计算机上的不同身份重复上一节中的步骤，以生成第二个密钥对。

您可以派生一个子密钥对来模拟本地计算机上的第二个身份。

您可以招募其他参与者来生成加入您的私有网络所需的密钥。

• 出于说明目的，本教程中使用的第二组键是：

Sr25519：5EJPj83tJuJtTVE2v7B9ehfM7jNT44CBFaPWicvBwYyUKBS6 用于aura.
Ed25519：5FeJQsfmbbJLTH1pvehBxrZrT5kHvJFj84ZaY5LK7NU87gZS 为grandpa.

# <span id='index4'>• 修改现有链规范</span>  
生成用于区块链的密钥后，您就可以使用这些密钥对创建自定义链规范，然后与称为验证器的受信任网络参与者共享您的自定义链规范。

为了使其他人能够参与您的区块链网络，您应该确保他们生成自己的密钥。如果其他参与者已经生成了他们的密钥对，您可以创建自定义链规范来替换local您之前使用的链规范。

本教程说明了如何创建一个两节点网络。您可以按照相同的步骤将更多节点添加到您的网络。

以前，您使用命令行选项使用预定义的local链规范将节点添加到区块链。--chain local无需编写全新的链规范，您可以修改之前使用的规范。

要基于现有规范创建新的链规范：

• 在您的计算机上打开终端外壳。

• 切换到编译 Substrate 节点模板的根目录。

• 将本地链规范导出到customSpec.json通过运行以下命令命名的文件：
```
./target/release/node-template build-spec --disable-default-bootnode --chain local > customSpec.json
```

• customSpec.json通过运行以下命令预览文件中的前几个字段：
```
head customSpec.json
```
该命令显示文件中的第一个字段。例如：
```
{
  "name": "Local Testnet",
  "id": "local_testnet",
  "chainType": "Local",
  "bootNodes": [],
  "telemetryEndpoints": null,
  "protocolId": null,
  "properties": null,
  "consensusEngine": null,
  "codeSubstitutes": {},
```

• customSpec.json通过运行以下命令预览文件中的最后一个字段：
```
tail -n 80 customSpec.json
```
此命令显示 Wasm 二进制字段后面的最后部分，包括运行时中使用的几个托盘的详细信息，例如sudo和balances托盘。

• customSpec.json在文本编辑器中打开文件。

• 修改aura字段以通过为每个网络参与者添加 Sr25519 SS58 地址密钥来指定有权创建块的节点。
```
"aura": {
    "authorities": [
      "5CfBuoHDvZ4fd8jkLQicNL8tgjnK8pVG9AiuJrsNrRAx6CNW",
      "5EJPj83tJuJtTVE2v7B9ehfM7jNT44CBFaPWicvBwYyUKBS6"
    ]
  },
```

• 修改该grandpa字段以通过为每个网络参与者添加 Ed25519 SS58 地址密钥来指定有权完成区块的节点。
```
"grandpa": {
    "authorities": [
      [
        "5CuqCGfwqhjGzSqz5mnq36tMe651mU9Ji8xQ4JRuUTvPcjVN",
        1
      ],
      [
        "5FeJQsfmbbJLTH1pvehBxrZrT5kHvJFj84ZaY5LK7NU87gZS",
        1
      ]
    ]
  },
```

# <span id='index5'>• 将链规范转换为使用原始格式</span>  
在您准备好包含您想要使用的信息的链式规范后，您必须先将其转换为原始规范，然后才能使用它。原始链规范包含与未转换规范相同的信息。但是，原始链规范还包含节点用来引用其本地存储中的数据的编码存储密钥。分发原始链规范可确保每个节点使用正确的存储密钥存储数据。

要将链规范转换为使用原始格式：

在您的计算机上打开终端外壳。

切换到编译 Substrate 节点模板的根目录。

通过运行以下命令将customSpec.json链规范转换为带有文件名的原始格式：customSpecRaw.json
```
./target/release/node-template build-spec --chain=customSpec.json --raw --disable-default-bootnode > customSpecRaw.json
```

# <span id='index6'>• 启动第一个节点</span>  
作为私有区块链网络的第一个参与者，您负责启动第一个节点，称为bootnode。

启动第一个节点：

• 在您的计算机上打开终端外壳。

• 切换到编译 Substrate 节点模板的根目录。

• 通过运行类似于以下的命令，使用自定义链规范启动第一个节点：
```
./target/release/node-template \
--base-path /tmp/node01 \
--chain ./customSpecRaw.json \
--port 30333 \
--ws-port 9945 \
--rpc-port 9933 \
--telemetry-url "wss://telemetry.polkadot.io/submit/ 0" \
--validator \
--rpc-methods Unsafe \
--name MyNode01
```

• [将密钥添加到密钥库](#index7)  
启动第一个节点后，尚未生成任何块。下一步是将两种类型的密钥添加到网络中每个节点的密钥库中。

对于每个节点：

添加aura权限密钥以启用块生产。

添加grandpa权限密钥以启用块完成。


要将密钥插入密钥库：

• 在您的计算机上打开终端外壳。

• 切换到编译 Substrate 节点模板的根目录。

• 通过运行类似于以下的命令插入aura从子命令生成的密钥：key
```
./target/release/node-template key insert --base-path /tmp/node01 \
--chain customSpecRaw.json \
--scheme Sr25519 \
--suri <your-secret-seed> \
--password-interactive \
--key-type aura
```

• 通过运行类似于以下的命令插入grandpa从子命令生成的密钥：key
```
./target/release/node-template key insert --base-path /tmp/node01 \
--chain customSpecRaw.json \
--scheme Ed25519 \
--suri <your-secret-key> \
--password-interactive \
--key-type gran
```

• node01通过运行以下命令验证您的密钥是否在密钥库中：
```
ls /tmp/node01/chains/local_testnet/keystore
```
该命令显示类似于以下内容的输出：
```
617572611441ddcb22724420b87ee295c6d47c5adff0ce598c87d3c749b776ba9a647f04
6772616e1441ddcb22724420b87ee295c6d47c5adff0ce598c87d3c749b776ba9a647f04
```

# <span id='index8'>• 允许其他参与者加入</span>  
您现在可以使用--bootnodes和--validator命令行选项允许其他验证者加入网络。

将第二个验证器添加到专用网络：

• 在第二台计算机上打开终端外壳。

• 切换到编译 Substrate 节点模板的根目录。

• 通过运行以下命令启动第二个区块链节点：
```
./target/release/node-template \
--base-path /tmp/node02 \
--chain ./customSpecRaw.json \
--port 30334 \
--ws-port 9946 \
--rpc-port 9934 \
--telemetry-url "wss://telemetry.polkadot.io/submit/ 0" \
--validator \
--rpc-methods Unsafe \
--name MyNode02 \
--bootnodes /ip4/127.0.0.1/tcp/30333/p2p/12D3KooWLmrYDLoNTyTYtRdDyZLWDe1paxzxTw5RgjmHLfzW96SX \
--password-interactive
```

• 通过运行类似于以下的命令添加aura从子命令生成的密钥：key
```
./target/release/node-template key insert --base-path /tmp/node02 \
--chain customSpecRaw.json \
--scheme Sr25519 \
--suri <second-participant-secret-seed> \
--password-interactive \
--key-type aura
```

• 通过运行类似于以下的命令，grandpa将从子命令生成的密钥添加到本地密钥库：key
```
./target/release/node-template key insert --base-path /tmp/node02 \
--chain customSpecRaw.json \
--scheme Ed25519 \
--suri <second-participant-secret-seed> \
--password-interactive \
--key-type gran
```

• node02通过运行以下命令验证您的密钥是否在密钥库中：
```
ls /tmp/node02/chains/local_testnet/keystore
```
该命令显示类似于以下内容的输出：
```
617572611a4cc824f6585859851f818e71ac63cf6fdc81018189809814677b2a4699cf45
6772616e1a4cc824f6585859851f818e71ac63cf6fdc81018189809814677b2a4699cf45
```
Substrate 节点在插入grandpa密钥后需要重新启动，因此您必须在看到区块完成之前关闭并重新启动节点。

• 按 Control-c 关闭节点。

• 通过运行以下命令重新启动第二个区块链节点：
```
./target/release/node-template \
--base-path /tmp/node02 \
--chain ./customSpecRaw.json \
--port 30334 \
--ws-port 9946 \
--rpc-port 9934 \
--telemetry-url "wss://telemetry.polkadot.io/submit/ 0" \
--validator \
--rpc-methods Unsafe \
--name MyNode02 \
--bootnodes /ip4/127.0.0.1/tcp/30333/p2p/12D3KooWLmrYDLoNTyTYtRdDyZLWDe1paxzxTw5RgjmHLfzW96SX \
--password-interactive
```

在两个节点都将其密钥添加到各自的密钥库（位于 和 下）/tmp/node01并/tmp/node02重新启动后，您应该会看到相同的创世块和状态根哈希。

您还应该看到每个节点都有一个对等节点（1 peers），并且它们产生了一个区块提议（best: #2 (0xe111…c084)）。几秒钟后，您应该会看到两个节点上的新块都已完成。

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
![微信公众号565 ee](https://user-images.githubusercontent.com/28084126/175346471-66d6a419-50e7-4ae1-a9fd-cb56d2c6c725.png)
