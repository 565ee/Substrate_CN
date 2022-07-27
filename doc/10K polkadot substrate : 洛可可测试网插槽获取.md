• [介绍](#index1)  
• [设置钱包](#index2)  
• [获取 ParaID](#index3)  
• [生成平行链创世和 Wasm 文件](#index4)  
• [启动您的整理器](#index5)  
• [注册为平行线程](#index6)  
• [请求您的平行链插槽](#index7)  
• [Substrate Tutorials , Substrate 教程](#index98)  
• [Contact 联系方式](#index99)  

# <span id='index1'>• 介绍</span>  
Rococo是 Parity 的平行链公共测试网络。这将引导您了解如何将平行链加入其中。

由于 Rococo 是一个共享测试网络，因此本教程需要额外的步骤来准备本地测试不需要的环境。在开始之前，请验证以下内容：

您知道如何生成和修改链规范文件，如添加受信任节点中所述。
您知道如何使用添加受信任节点中描述的方法之一生成和存储密钥。
您已完成平行链教程并在本地计算机上测试了平行链。

# <span id='index2'>• 设置钱包</span>  
要对 Rococo 执行任何操作，您需要 ROC 代币。要存储这些，您需要一个与 Substrate 兼容的加密货币钱包。对于任何公共环境中的此操作，您都不能使用开发密钥和帐户。有许多选项可用于设置钱包帐户。有关其中一些选项的信息，请参阅Polkadot wiki 上的构建钱包。如果您刚刚开始，您可以使用polkadot-js扩展名. 拥有帐户后，请务必：

备份您的秘密种子短语。
记下您accountID正在使用默认的 42 SS58 前缀与 Rococo 一起使用。
获取ROC，加入洛可可龙头矩阵频道，在龙头频道中使用!drip <accountID>指令获取100 ROC到你的钱包。
https://matrix.to/#/#rococo-faucet:matrix.org

# <span id='index3'>• 获取 ParaID</span>  
使用最少 5 ROC 的免费余额，在 Rococo 上注册为平行线程：

转到Rococo的 Polkadot-JS 应用平行线程部分。
保留一个独特的ParaID. 您将被分配到下一个可用的 ID -请注意这一点。
![image](https://user-images.githubusercontent.com/28084126/178095957-fd200a96-83bd-4289-8ce6-67b32fe38012.png)

请注意，在 Rococo 上，ParaID数字0-999是为系统平行链保留的，1000-1999也是为公用事业平行链保留的。只有2000尚未保留的数字及以上才能用于社区平行链。

# <span id='index4'>• 生成平行链创世和 Wasm 文件</span>  
注册平行链所需的文件必须明确指定正确的中继链和权限ParaID。在本教程中，中继链rococo不是rococo-local在连接本地平行链教程中使用。

• 配置您的链规范以使用：

ParaID你的洛可可
您的整理者节点的开发密钥和帐户的独特替代方案。虽然Alice这样的帐户可以使用，但您绝对不应该使用它们！

• 为洛可可生成适当的平行链创世状态。

• 为 Rococo 生成平行链运行时 Wasm blob。

# <span id='index5'>• 启动您的整理器</span>  
对于嵌入式中继链和平行链，您必须拥有可公开访问和发现的整理者对等端口。这样你就可以与洛可可验证节点对等，否则你将无法产生区块！对等端口使用--port <collator node>-- --port <relay node>CLI 标志设置，请务必分别为两个节点执行此操作。您很可能希望至少您的整理者的--ws-port <ws port>端口也打开，以允许您自己（和其他人）通过 Polkadot 应用程序 UI 或 API 调用与其连接。

# <span id='index6'>• 注册为平行线程</span>  
在成为平行链之前，您必须在 Rococo 上注册为平行线程：

• 注册为平行线程。使用您保留的ParaID.
![image](https://user-images.githubusercontent.com/28084126/178096127-11f79040-44ae-4f83-a657-794c4f5118ae.png)

• 检查您是否registrar.Registered在 Polkadot-JS 应用程序中看到一个事件以验证注册成功。
![image](https://user-images.githubusercontent.com/28084126/178096148-e0f16b1d-6be2-4e8b-9c57-387c09bbcba9.png)

• 单击Parachains -> Parathreads并验证您的 parathread 注册是否为Onboarding：
![image](https://user-images.githubusercontent.com/28084126/178096167-dc6f50be-8f36-43e4-abfd-091eb7d707f0.png)
  
# <span id='index7'>• 请求您的平行链插槽</span>  
在平行链作为平行线程激活后，相关项目团队应在 Rococo 上打开永久或临时平行链插槽的请求。

• 永久插槽是分配给当前在 Polkadot 上拥有平行链插槽的团队（在成功的插槽租赁拍卖之后）的平行链插槽，因此需要不断测试他们的代码库以与现实世界中的最新前沿功能兼容（洛可可）。只有有限数量的永久插槽可用（请参阅下面的注释）。

• 临时插槽是平行链插槽，以连续的循环方式动态分配。在每个租期开始时，一定数量的平行线程（最多为中继链配置中定义的最大值）会在一定时间内自动升级为平行链。在租约结束期间处于活动状态的平行链会自动降级为平行线程，以释放插槽供其他人在后续期间使用。临时插槽旨在供尚未在 Polkadot 上拥有平行链插槽的团队使用，并计划在不久的将来使用。

动态分配的目标是帮助团队更频繁地测试他们的运行时，并且以更精简的方式。

Rococo 运行时需要sudo访问分配槽 ( AssignSlotOrigin = EnsureRoot<Self::AccountId>;)。目前，洛可可sudo密钥由 Parity Technologies 控制，因此要进行获取插槽所需的操作，请在完成上述操作并准备连接后前往Subport repo 并打开一个！Rococo Slot Request激活您的插槽后，Parity 团队成员将做出回应。最终，该过程旨在通过洛可可式治理框架由社区驱动。

• 使用具有AssignSlotOrigin源的帐户，请按照以下步骤分配临时槽：
为 Rococo打开Polkadot-JS 应用程序。
单击开发人员>外部组件。
选择您要用于提交交易的帐户。
选择assignedSlots托盘。
选择assignTempParachainSlot功能。
插入您保留的ParaID. 确保使用您之前保留的 ParaID。
选择. Current_ LeasePeriodStart如果当前插槽已满，您将被分配下一个可用插槽。
签署并提交交易。

• 给定 1 天的租期，洛可可分配的平行链插槽的当前设置是（在撰写本文时）：
永久插槽最短持续时间：1 年（365 天）
临时时段最短持续时间：3 天
永久插槽的最大数量：最多 25 个永久插槽
最大临时插槽数：最多 20 个临时插槽
每个租期分配的最大临时时段：每 3 天临时租期最多 5 个临时时段

Parity 团队激活您的插槽后，您可以在 Rococo 测试网络上测试您的平行链。请注意，当您的临时插槽租约结束时，平行链会自动降级为平行线程。注册和批准的插槽以循环方式自动循环，因此您可以期望不时作为平行链重新上线。

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
![微信公众号565 ee](https://user-images.githubusercontent.com/28084126/178096339-d092c4be-cddb-4694-a139-e49ce15bf27b.png)

