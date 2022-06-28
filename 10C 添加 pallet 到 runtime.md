• [介绍](#index1)  
• [教程目标](#index2)  
• [添加 Nicks 托盘依赖项](#index3)  
• [查看托盘的配置特征](#index4)  
• [为托盘实现 Config 特征](#index5)  
• [启动区块链节点](#index6)  
• [启动前端模板](#index7)  
• [使用 Nicks 托盘设置昵称](#index8)  
• [使用 Nicks 托盘查询帐户信息](#index9)  
• [Substrate Tutorials , Substrate 教程](#index98)  
• [Contact 联系方式](#index99)  

# <span id='index1'>• 介绍</span>  
如您所见，构建本地区块链，Substrate 节点模板提供了一个工作运行时，其中包括一些默认的 FRAME 开发模块（托盘），以帮助您开始构建自定义区块链。

本教程介绍了将新托盘添加到节点模板的运行时的基本步骤。每当您想向运行时添加新的 FRAME 托盘时，这些步骤都是类似的。但是，每个托盘都需要特定的配置设置——例如，执行托盘实现的功能所需的特定参数和类型。在本教程中，您会将Nicks 托盘添加到节点模板的运行时，因此您将了解如何配置特定于 Nicks 托盘的设置。Nicks 托盘允许区块链用户支付押金，为他们控制的账户保留昵称。它实现了以下功能：

set_name收取存款并设置帐户名称（如果尚未使用该名称）的功能。
clear_name删除与帐户关联的名称并退回押金的功能。
kill_name强制删除账户名而不退还押金的功能。

# <span id='index2'>• 教程目标</span>  
通过完成本教程，您将使用 Nicks 托盘来完成以下目标：

• 了解如何更新运行时依赖项以包含新托盘。

• 了解如何配置特定于托盘的 Rust 特征。

• 通过使用前端模板与新托盘交互来查看运行时的更改。

# <span id='index3'>• 添加 Nicks 托盘依赖项</span>  
在您可以使用新托盘之前，您必须将一些关于它的信息添加到编译器用来构建运行时二进制文件的配置文件中。

对于 Rust 程序，您使用该Cargo.toml文件来定义配置设置和依赖项，这些设置和依赖项决定了在生成的二进制文件中编译的内容。因为 Substrate 运行时编译为包含标准库函数的原生 Rust 二进制文件和不包含标准库的WebAssembly (Wasm)二进制文件，所以该Cargo.toml文件控制两个重要信息：

要作为运行时依赖项导入的托盘，包括要导入的托盘的位置和版本。
编译原生 Rust 二进制文件时应启用的每个托盘中的功能。通过从每个托盘启用标准 ( std) 功能集，您可以编译运行时以包含在构建 WebAssembly 二进制文件时会丢失的函数、类型和原语。

要将 Nicks 托盘的依赖项添加到运行时：

• 打开终端外壳并切换到节点模板的根目录。

• runtime/Cargo.toml在文本编辑器中打开配置文件。

• 通过将 crate 添加到依赖项列表中，导入pallet-nickscrate 以使其可用于节点模板运行时。
```
[dependencies.pallet-nicks]
default-features = false
git = 'https://github.com/paritytech/substrate.git'
tag = 'monthly-2021-10'
version = '4.0.0-dev'
```

• 在编译运行时将功能添加pallet-nicks/std到要启用的列表中。features
```
[features]
default = ['std']
std = [
  ...
  'pallet-aura/std',
  'pallet-balances/std',
  'pallet-nicks/std',    # add this line
  ...
]
```

• 通过运行以下命令检查新依赖项是否正确解析：
```
cargo check -p node-template-runtime
```

# <span id='index4'>• 查看托盘的配置特征</span>  
每个托盘都有一个名为Config. 该Config特征用于识别托盘执行其功能所需的参数和类型。

添加托盘所需的大多数特定于托盘的代码都是使用Configtrait 实现的。您可以通过参考其 Rust 文档或托盘的源代码来查看您需要为任何托盘实现的内容。例如，要查看您需要为nicks托盘实现什么，您可以参考 Rust 文档或Nicks 托盘源代码pallet_nicks::Config中的特征定义。

对于本教程，您可以看到托盘中的Configtraitnicks声明了以下类型：
```
pub trait Config: frame_system::Config {
	/// The overarching event type.
	type Event: From<Event<Self>> + IsType<<Self as frame_system::Config>::Event>;

	/// The currency trait.
	type Currency: ReservableCurrency<Self::AccountId>;

	/// Reservation fee.
	#[pallet::constant]
	type ReservationFee: Get<BalanceOf<Self>>;

	/// What to do with slashed funds.
	type Slashed: OnUnbalanced<NegativeImbalanceOf<Self>>;

	/// The origin account that can forcibly set or remove a name. Root can always do this.
	type ForceOrigin: EnsureOrigin<Self::Origin>;

	/// The minimum length for a name.
	#[pallet::constant]
	type MinLength: Get<u32>;

	/// The maximum length for a name.
	#[pallet::constant]
	type MaxLength: Get<u32>;
}
```

确定您的托盘所需的类型后，您需要将代码添加到运行时以实现该Config特征。要了解如何Config为托盘实现特征，您可以使用已在节点模板运行时中实现的Balances托盘作为示例。

要查看ConfigBalances 托盘的特征：

• runtime/src/lib.rs在文本编辑器中打开文件。

• 找到Balances托盘部分。

• 请注意，Balances托盘的实现由两部分组成：
parameter_types!定义常量值的块。
```
parameter_types! {
	// The u128 constant value 500 is aliased to a type named ExistentialDeposit.
	pub const ExistentialDeposit: u128 = 500;
	// A heuristic that is used for weight estimation.
	pub const MaxLocks: u32 = 50;
}
```

配置接口impl定义的类型和值的块。Config
```
impl pallet_balances::Config for Runtime {
	// The previously defined parameter_type is used as a configuration parameter.
	type MaxLocks = MaxLocks;
	// The "Balance" that appears after the equal sign is an alias for the u128 type.
	type Balance = Balance;
	// The empty value, (), is used to specify a no-op callback function.
	type DustRemoval = ();
	// The previously defined parameter_type is used as a configuration parameter.
	type ExistentialDeposit = ExistentialDeposit;
	// The FRAME runtime system is used to track the accounts that hold balances.
	type AccountStore = System;
	// Weight information is supplied to the Balances pallet by the node template runtime.
	// type WeightInfo = (); // old way
	type WeightInfo = pallet_balances::weights::SubstrateWeight<Runtime>;
	// The ubiquitous event type.
	type Event = Event;
}
```

# <span id='index5'>• 为托盘实现 Config 特征</span>  
现在您已经看到了如何Config为 Balances 托盘实现特征的示例，您已准备好Config为 Nicks 托盘实现特征。

要nicks在运行时实现托盘：

• runtime/src/lib.rs在文本编辑器中打开文件。

• 找到 Balances 代码块的最后一行。

• 为 Nicks 托盘添加以下代码块：
```
/// Add this code block to your template for Nicks:
parameter_types! {
	// Choose a fee that incentivizes desireable behavior.
	pub const NickReservationFee: u128 = 100;
	pub const MinNickLength: u32 = 8;
	// Maximum bounds on storage are important to secure your chain.
	pub const MaxNickLength: u32 = 32;
}

impl pallet_nicks::Config for Runtime {
	// The Balances pallet implements the ReservableCurrency trait.
	// `Balances` is defined in `construct_runtime!` macro. See below.
	// https://paritytech.github.io/substrate/master/pallet_balances/index.html#implementations-2
	type Currency = Balances;

	// Use the NickReservationFee from the parameter_types block.
	type ReservationFee = NickReservationFee;

	// No action is taken when deposits are forfeited.
	type Slashed = ();

	// Configure the FRAME System Root origin as the Nick pallet admin.
	// https://paritytech.github.io/substrate/master/frame_system/enum.RawOrigin.html#variant.Root
	type ForceOrigin = frame_system::EnsureRoot<AccountId>;

	// Use the MinNickLength from the parameter_types block.
	type MinLength = MinNickLength;

	// Use the MaxNickLength from the parameter_types block.
	type MaxLength = MaxNickLength;

	// The ubiquitous event type.
	type Event = Event;
}
```

• 将尼克斯添加到construct_runtime!宏。
```
construct_runtime!(
pub enum Runtime where
    Block = Block,
    NodeBlock = opaque::Block,
    UncheckedExtrinsic = UncheckedExtrinsic
  {
    /* --snip-- */
    Balances: pallet_balances::{Pallet, Call, Storage, Config<T>, Event<T>},

    /*** Add This Line ***/
    Nicks: pallet_nicks::{Pallet, Call, Storage, Event<T>},
  }
);
```

• 通过运行以下命令检查新依赖项是否正确解析：
```
cargo check -p node-template-runtime
```

• 通过运行以下命令以发布模式编译节点：
```
cargo build --release
```

# <span id='index6'>• 启动区块链节点</span>  
在您的节点编译后，您就可以启动已通过Nicks 托盘中的昵称功能增强的节点，并使用前端模板与其交互。

启动本地 Substrate 节点：

• 如有必要，打开终端外壳。

• 切换到 Substrate 节点模板的根目录。

• 通过运行以下命令以开发模式启动节点：
```
./target/release/node-template --dev
```
在这种情况下，该--dev选项指定节点使用预定义的development链规范在开发者模式下运行。默认情况下，当您按 Control-c 停止节点时，此选项还会删除所有活动数据，例如密钥、区块链数据库和网络信息。使用该--dev选项可确保您在任何时候停止和重新启动节点时都处于干净的工作状态。

• 通过查看终端中显示的输出来验证您的节点是否已启动并成功运行。

如果finalized控制台输出中的数字在增加，则您的区块链正在生成新块并就它们描述的状态达成共识。

• 保持显示节点输出的终端打开以继续。

# <span id='index7'>• 启动前端模板</span>  
现在您已经向运行时添加了一个新托盘，您可以使用 Substrate 前端模板与节点模板交互并访问 Nicks 托盘。

启动前端模板：

• 在您的计算机上打开一个新的终端外壳。

• 在新终端中，切换到安装前端模板的根目录。

• 通过运行以下命令启动前端模板的 Web 服务器：
```
yarn start
```
• 在浏览器中打开http://localhost:8000/以查看前端模板。

# <span id='index8'>• 使用 Nicks 托盘设置昵称</span>  
启动前端模板后，您可以使用它与刚刚添加到运行时的 Nicks 托盘进行交互。

为帐户设置昵称：

检查帐户选择列表以验证当前选择了 Alice 帐户。

在 Pallet Interactor 组件中，确认选择了 Extrinsic。

nicks从可调用的托盘列表中选择。

选择可setName调度作为要从nicks托盘调用的函数。

键入一个长度超过MinNickLength（8 个字符）且不超过MaxNickLength（32 个字符）的名称。

单击Signed以执行该功能。
![image](https://user-images.githubusercontent.com/28084126/176255726-e590a471-2c85-4963-ba8a-940a06047214.png)

设置名称

观察调用状态和Nicks 托盘发出的事件。

# <span id='index9'>• 使用 Nicks 托盘查询帐户信息</span>  
接下来，您可以使用查询功能从 Nicks 托盘的运行时存储中读取 Alice 的昵称值。

要返回为 Alice 存储的信息：

• 在 Pallet Interactor 组件中，选择Query。

• nicks从可查询的托盘列表中选择。

• 选择nameOf.

• 复制并粘贴alice帐户字段中的帐户地址，然后单击查询。
![image](https://user-images.githubusercontent.com/28084126/176256083-74f480ca-f529-416f-a284-fedd41a7d3dc.png)
返回类型是一个包含两个值的元组：

Alice 帐户的十六进制编码昵称。

为保护昵称而从 Alice 的账户中保留的金额。

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
![微信公众号565 ee](https://user-images.githubusercontent.com/28084126/176256832-306f05a7-c9c0-4405-9c08-5675bbd079c6.png)
