• [介绍](#index1)  
• [学习成果](#index2)  
• [启动模板节点](#index3)  
• [用于sudo派送](#index4)  
• [添加Scheduler托盘](#index5)  
• [构建升级的运行时](#index6)  
• [准备升级的运行时](#index7)  
• [升级运行时](#index8)  
• [Substrate Tutorials , Substrate 教程](#index98)  
• [Contact 联系方式](#index99)  

# <span id='index1'>• 介绍</span>  
Substrate 区块链开发框架的定义特性之一是它支持 无分叉运行时升级。无分叉升级是一种增强区块链运行时的方式，这种方式由区块链本身的功能支持和保护。区块链的 运行时定义了区块链可以保持的状态，还定义了影响该状态更改的逻辑。
![node-diagram](https://user-images.githubusercontent.com/28084126/174053376-f3972b65-3415-4bed-8273-d39aabb8eab8.png)

Substrate 可以在没有硬分叉的情况下部署增强的运行时功能（包括重大更改（！） ） 。因为运行时的定义本身就是 Substrate 链状态中的一个元素，所以网络参与者可以通过外部的方式更新这个值，特别 是函数。由于运行时状态的更新受区块链的共识机制和加密保证的约束，网络参与者可以使用区块链本身以不信任的方式分发更新或扩展的运行时逻辑，而无需分叉链甚至发布新的区块链客户端。set_code

本教程将使用 Substrate Developer Hub 节点模板来探索基于FRAME的运行时的无分叉升级的两种机制。首先， sudo_unchecked_weight 来自Sudo 托盘的函数将用于执行添加调度程序托盘的升级。然后， schedule 调度程序托盘中的功能将用于执行升级，以增加网络帐户的 存在（最低）余额 。

# <span id='index2'>• 学习成果</span>  
• 了解 Substrate 链中的一些基本治理特性
• 获得有关 Substrate 链如何升级其运行时的经验
• 了解如何在运行时使用调度程序托盘

# <span id='index3'>• 启动模板节点</span>  
由于无分叉运行时升级不需要网络参与者重新启动他们的区块链客户端，因此本教程的第一步是按原样启动模板节点。构建并启动未修改的节点模板。
```
cargo run --release -- --dev
```
默认情况下，众所周知的 Alice 帐户development_config在模板节点的 链规范文件的功能中配置为 Sudo 托盘密钥的持有者- 这是使用--dev标志启动节点时使用的配置。这意味着 Alice 的帐户将在本教程中用于执行运行时升级。

# <span id='index4'>• 用于sudo派送</span>  
正如Sudo 托盘的名称所暗示的那样，它提供了与管理单个 sudo（“超级用户”）管理员相关的功能。在FRAME中，Root Origin用于标识运行时管理员；FRAME 的一些功能，包括通过函数更新运行时的能力 ，set_code只有该管理员可以访问。Sudo 托盘维护一个单一的 存储项目：有权访问托盘的可调度功能的帐户的 ID 。Sudo 托盘的sudo功能允许此帐户的持有者调用可调度作为Root 源。

以下伪代码演示了这是如何实现的，请参阅 Sudo 托盘的源代码 以了解更多信息。
```
fn sudo(origin, call) -> Result {
	// Ensure caller is the account identified by the administrator key
	let sender = ensure_signed(origin)?;
	ensure!(sender == Self::key(), Error::RequireSudo);

	// Dispatch the specified call as the Root origin
	let res = call.dispatch(Origin::Root);
	Ok()
}
```

# <span id='index5'>• 添加Scheduler托盘</span>  
由于模板节点没有 在其运行时中包含调度程序托盘，因此本教程中执行的第一次运行时升级将添加该托盘。首先，在模板节点的运行时 Cargo 文件中添加调度器托盘作为依赖项。

runtime/Cargo.toml
```
pallet-scheduler = { default-features = false, git = "https://github.com/paritytech/substrate.git", branch = "polkadot-v0.9.23" }

#--snip--

[features]
default = ['std']
std = [
    #--snip--
    'pallet-scheduler/std',
    #--snip--
]
```

准备升级的 FRAME 运行时的最后一步是增加其 struct的spec_version成员 ：RuntimeVersion
runtime/src/lib.rs
```
pub const VERSION: RuntimeVersion = RuntimeVersion {
	spec_name: create_runtime_str!("node-template"),
	impl_name: create_runtime_str!("node-template"),
	authoring_version: 1,
	spec_version: 101,  // *Increment* this value, the template uses 100 as a base
	impl_version: 1,
	apis: RUNTIME_API_VERSIONS,
	transaction_version: 1,
};
```

# <span id='index6'>• 构建升级的运行时</span>  
```
cargo build --release -p node-template-runtime
```
这里的--release标志将导致更长的编译时间，但也会生成更适合提交到区块链网络的更小的构建工件：存储最小化和优化对于任何区块链都至关重要。

由于我们只构建运行时，Cargo 在运行时 cargo.toml文件中查找需求并只执行这些。请注意， cargo 查找的 runtime/build.rs文件会构建运行时的 Wasm，该文件在runtime/src/lib.rs.

当--release指定标志时，构建工件被输出到 target/release目录；当标志被省略时，它们将被发送到target/debug.

打开 Polkadot JS Apps UI 并自动配置 UI 以连接到本地节点。
使用 Alice 的帐户调用该sudoUncheckedWeight函数并将setCode系统托盘中的函数用作其参数。为了提供由上一个构建步骤生成的构建工件，请将参数“代码”输入字段右侧的“文件上传”开关切换到setCode函数。单击“代码”输入字段，然后选择定义升级运行时的 Wasm 二进制文件： target/release/wbuild/node-template-runtime/node_template_runtime.compact.wasm. 将参数的值保留为_weight默认值0。点击“提交交易”，然后点击“签名并提交”。
![sudo-upgrade](https://user-images.githubusercontent.com/28084126/174057515-0a356a4a-25a2-4cb9-8569-e06a3954ac75.png)

交易被包含在区块中后，Polkadot JS Apps UI 左上角的版本号应该反映运行时版本是 now 101。

如果您仍然看到您的节点在终端运行并在 UI 上报告生成块，那么您已经成功执行了无分叉运行时升级！恭喜！！！

# <span id='index7'>• 准备升级的运行时</span>  
此升级比前一次更简单，只需要更新runtime/src/lib.rs运行时之外的单个值spec_version。

```
pub const VERSION: RuntimeVersion = RuntimeVersion {
	spec_name: create_runtime_str!("node-template"),
	impl_name: create_runtime_str!("node-template"),
	authoring_version: 1,
	spec_version: 102,  // *Increment* this value.
	impl_version: 1,
	apis: RUNTIME_API_VERSIONS,
	transaction_version: 1,
};

/*** snip ***/

parameter_types! {
	pub const ExistentialDeposit: u128 = 1000;  // Update this value.
	pub const MaxLocks: u32 = 50;
}

/*** snip ***/

```

此更改增加了余额托盘的价值 - 从余额托盘的 ExistentialDeposit角度来看，保持帐户处于活动状态所需的最低余额。

```
cargo build --release -p node-template-runtime
```
这将覆盖任何以前的构建工件！因此，如果您想拥有上一个运行时 Wasm 构建文件的副本，请务必将它们复制到其他地方。”

# <span id='index8'>• 升级运行时</span>  
在上一节中，Scheduler 托盘配置了Rootorigin 作为 its ScheduleOrigin，这意味着可以使用sudo函数（not sudo_unchecked_weight）来调用 schedule函数。使用此链接打开 Polkadot JS Apps UI 的 Sudo 选项卡： https ://polkadot.js.org/apps/#/sudo?rpc=ws://127.0.0.1:9944 。

等到所有其他字段都填写完毕后再提供when参数。将参数保留为maybe_periodic空，并将priority参数保留为其默认值0。选择系统托盘的set_code功能作为call参数，并像以前一样提供 Wasm 二进制文件。使“重量覆盖”选项保持禁用状态。一旦所有其他字段都填写完毕，使用未来大约 10 个区块（1 分钟）的区块号来填写when参数并快速提交交易。
![scheduled-upgrade](https://user-images.githubusercontent.com/28084126/174058316-97c8a010-12a5-4547-9d93-73b9c1b4e827.png)

目标区块入链后，Polkadot JS Apps UI 左上角的版本号应该反映运行时版本为 now 102。

然后，您可以通过使用 Polkadot JS Apps UI Chain StateexistentialDeposit应用程序从 Balances 托盘 中查询常量值来观察升级中所做的具体更改。

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
![微信公众号565 ee](https://user-images.githubusercontent.com/28084126/174058658-797942c2-a780-4e28-8b9a-ce5078c05721.png)

