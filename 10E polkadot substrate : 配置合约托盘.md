• [介绍](#index1)  
• [添加托盘依赖项](#index2)  
• [实现 Contracts 配置特征](#index3)  
• [公开合约 API](#index4)  
• [更新外部节点](#index5)  
• [启动本地 Substrate 节点](#index6)  
• [Substrate Tutorials , Substrate 教程](#index98)  
• [Contact 联系方式](#index99) 

# <span id='index1'>• 介绍</span>  
如果您完成了构建本地区块链教程，那么您已经知道 Substrate节点模板提供了一个工作运行时，其中包括一些供您入门的托盘。在将托盘添加到运行时中，您了解了将新托盘添加到运行时的基本常用步骤。但是，每个托盘都需要您配置特定的参数和类型。为了了解这意味着什么，本教程演示了向运行时添加更复杂的托盘。在本教程中，您将添加Contracts 托盘，以便您可以支持区块链的智能合约开发。

# <span id='index2'>• 添加托盘依赖项</span>  
任何时候将托盘添加到运行时，都需要导入适当的 crate 并更新运行时的依赖项。对于 Contracts 托盘，您需要导入pallet-contractscrate。

• runtime/Cargo.toml在文本编辑器中打开配置文件。

• 通过将 crate 添加到依赖项列表中，导入pallet-contractscrate 以使其可用于节点模板运行时。
```
[dependencies.pallet-contracts]
default-features = false
git = 'https://github.com/paritytech/substrate.git'
tag = 'latest'
version = '4.0.0-dev'

[dependencies.pallet-contracts-primitives]
default-features = false
git = 'https://github.com/paritytech/substrate.git'
tag = 'latest'
version = '4.0.0-dev'
```

• 将 Contracts 托盘添加到std功能列表中，以便在将运行时构建为原生 Rust 二进制文件时包含其功能。
```
[features]
default = ['std']
std = [
	#--snip--
	'pallet-contracts/std',
	'pallet-contracts-primitives/std',
	#--snip--
 ]
```

• 保存更改并关闭runtime/Cargo.toml文件。

# <span id='index3'>• 实现 Contracts 配置特征</span>  
现在您已成功导入 Contracts 托盘箱，您可以将其添加到运行时。如果您浏览过其他教程，您可能已经知道每个托盘都有一个配置特征 - 称为Config- 运行时必须实现。

要Config在运行时实现 Contracts 托盘的特征：

• runtime/src/lib.rs在文本编辑器中打开文件。

• 找到pub use frame_support块并添加Nothing到特征列表中。

```
pub use frame_support::{
	construct_runtime, parameter_types,
	traits::{KeyOwnerProofSystem, Randomness, StorageInfo, Nothing},
	weights::{
	constants::{BlockExecutionWeight, ExtrinsicBaseWeight, RocksDbWeight, WEIGHT_PER_SECOND},
	IdentityFee, Weight,
},
StorageValue,
};
```

• 将WeightInfo属性添加到运行时。

```
/* After this line */
use pallet_transaction_payment::CurrencyAdapter;
/*** Add this line ***/
use pallet_contracts::weights::WeightInfo;
```

• 将 Contracts 托盘所需的常量添加到运行时。
```
/* After this block */
// Time is measured by number of blocks.
pub const MINUTES: BlockNumber = 60_000 / (MILLISECS_PER_BLOCK as BlockNumber);
pub const HOURS: BlockNumber = MINUTES * 60;
pub const DAYS: BlockNumber = HOURS * 24;

/* Add this block */
// Contracts price units.
pub const MILLICENTS: Balance = 1_000_000_000;
pub const CENTS: Balance = 1_000 * MILLICENTS;
pub const DOLLARS: Balance = 100 * CENTS;

const fn deposit(items: u32, bytes: u32) -> Balance {
	items as Balance * 15 * CENTS + (bytes as Balance) * 6 * CENTS
}

/// Assume ~10% of the block weight is consumed by `on_initialize` handlers.
/// This is used to limit the maximal weight of a single extrinsic.
const AVERAGE_ON_INITIALIZE_RATIO: Perbill = Perbill::from_percent(10);
/*** End Added Block ***/
```

• 将 Config 特征的参数类型和实现添加到运行时。
```
impl pallet_timestamp::Config for Runtime {
/* --snip-- */
}

/*** Add this block ***/
parameter_types! {
	pub TombstoneDeposit: Balance = deposit(
		1,
		<pallet_contracts::Pallet<Runtime>>::contract_info_size()
	);
	pub DepositPerContract: Balance = TombstoneDeposit::get();
	pub const DepositPerStorageByte: Balance = deposit(0, 1);
	pub const DepositPerStorageItem: Balance = deposit(1, 0);
	pub RentFraction: Perbill = Perbill::from_rational(1u32, 30 * DAYS);
	pub const SurchargeReward: Balance = 150 * MILLICENTS;
	pub const SignedClaimHandicap: u32 = 2;
	pub const MaxValueSize: u32 = 16 * 1024;
	// The lazy deletion runs inside on_initialize.
	pub DeletionWeightLimit: Weight = AVERAGE_ON_INITIALIZE_RATIO *
	 BlockWeights::get().max_block;
	// The weight needed for decoding the queue should be less or equal than a fifth
	// of the overall weight dedicated to the lazy deletion.
	pub DeletionQueueDepth: u32 = ((DeletionWeightLimit::get() / (
		<Runtime as pallet_contracts::Config>::WeightInfo::on_initialize_per_queue_item(1) -
		<Runtime as pallet_contracts::Config>::WeightInfo::on_initialize_per_queue_item(0)
	 )) / 5) as u32;

	pub Schedule: pallet_contracts::Schedule<Runtime> = Default::default();
}

impl pallet_contracts::Config for Runtime {
	type Time = Timestamp;
	type Randomness = RandomnessCollectiveFlip;
	type Currency = Balances;
	type Event = Event;
	type RentPayment = ();
	type SignedClaimHandicap = SignedClaimHandicap;
	type TombstoneDeposit = TombstoneDeposit;
	type DepositPerContract = DepositPerContract;
	type DepositPerStorageByte = DepositPerStorageByte;
	type DepositPerStorageItem = DepositPerStorageItem;
	type RentFraction = RentFraction;
	type SurchargeReward = SurchargeReward;
	type WeightPrice = pallet_transaction_payment::Module<Self>;
	type WeightInfo = pallet_contracts::weights::SubstrateWeight<Self>;
	type ChainExtension = ();
	type DeletionQueueDepth = DeletionQueueDepth;
	type DeletionWeightLimit = DeletionWeightLimit;
	type Call = Call;
	/// The safest default is to allow no calls at all.
	///
	/// Runtimes should whitelist dispatchables that are allowed to be called from contracts
	/// and make sure they are stable. Dispatchables exposed to contracts are not allowed to
	/// change because that would break already deployed contracts. The `Call` structure itself
	/// is not allowed to change the indices of existing pallets, too.
	type CallFilter = Nothing;
	type Schedule = Schedule;
	type CallStack = [pallet_contracts::Frame<Self>; 31];
}
/*** End added block ***/
```

• 将合同托盘中公开的类型添加到construct_runtime!宏中。
```
construct_runtime!(
	pub enum Runtime where
	 Block = Block,
	 NodeBlock = opaque::Block,
	 UncheckedExtrinsic = UncheckedExtrinsic
	{
	 /* --snip-- */
	 /*** Add this ine ***/
	 Contracts: pallet_contracts::{Pallet, Call, Storage, Event<T>},
	}
);
```

• 保存更改并关闭runtime/src/lib.rs文件。

• 通过运行以下命令检查您的运行时是否正确编译：
```
cargo check -p node-template-runtime
```
尽管运行时应该编译，但您还不能编译整个节点。

# <span id='index4'>• 公开合约 API</span>  
一些托盘（包括合同托盘）公开了自定义运行时 API 和 RPC 端点。您无需启用 Contracts 托盘上的 RPC 调用即可在链上使用它。但是，公开 Contracts 托盘的 API 和端点很有用，因为这样做可以让您执行以下任务：

从链下读取合约状态。

在不进行事务的情况下调用节点存储。

要公开 Contracts RPC API：

• 在文本编辑器中打开runtime/Cargo.toml文件并添加依赖项部分以导入 Contracts RPC 端点运行时 API。
```
[dependencies.pallet-contracts-rpc-runtime-api]
default-features = false
git = 'https://github.com/paritytech/substrate.git'
tag = 'latest'
version = '4.0.0-dev'
```

• 将 Contracts RPC API 添加到std功能列表中，以便在将运行时构建为原生 Rust 二进制文件时包含其功能。
```
[features]
default = ['std']
std = [
	#--snip--
	'pallet-contracts-rpc-runtime-api/std',
]
```

• 保存更改并关闭runtime/Cargo.toml文件。

• 打开文件并在运行时文件末尾附近的宏runtime/src/lib.rs中实现合约运行时 API 。impl_runtime_apis!lib.rs
```
impl_runtime_apis! {
	/* --snip-- */
	/*** Add this block ***/
	impl pallet_contracts_rpc_runtime_api::ContractsApi<Block, AccountId, Balance, BlockNumber, Hash>
	for Runtime {
	 fn call(
			origin: AccountId,
			dest: AccountId,
			value: Balance,
			gas_limit: u64,
			input_data: Vec<u8>,
	 ) -> pallet_contracts_primitives::ContractExecResult {
			let debug = true;
			Contracts::bare_call(origin, dest, value, gas_limit, input_data, debug)
	}

		fn instantiate(
			origin: AccountId,
			endowment: Balance,
			gas_limit: u64,
			code: pallet_contracts_primitives::Code<Hash>,
			data: Vec<u8>,
			salt: Vec<u8>,
	 ) -> pallet_contracts_primitives::ContractInstantiateResult<AccountId, BlockNumber> {
			let compute_rent_projection = true;
			let debug = true;
			Contracts::bare_instantiate(origin, endowment, gas_limit, code, data, salt, compute_rent_projection, debug)
	 }

	 fn get_storage(
			address: AccountId,
			key: [u8; 32],
	 ) -> pallet_contracts_primitives::GetStorageResult {
			Contracts::get_storage(address, key)
	 }

	 fn rent_projection(
			address: AccountId,
	 ) -> pallet_contracts_primitives::RentProjectionResult<BlockNumber> {
			Contracts::rent_projection(address)
	 }
	}
	/*** End added block ***/
}
```

• 保存更改并关闭runtime/src/lib.rs文件。

• 通过运行以下命令检查您的运行时是否正确编译：
```
cargo check -p node-template-runtime
```

# <span id='index5'>• 更新外部节点</span>  
至此，您已完成将 Contracts 托盘添加到运行时。现在，您需要考虑外部节点是否需要任何相应的更新。要让 Contracts 托盘利用 RPC 端点 API，您需要将自定义 RPC 端点添加到节点配置中。

要将 RPC API 扩展添加到外部节点：

打开node/Cargo.toml文件并添加依赖项部分以导入 Contracts 和 Contracts RPC crates。
```
[dependencies]
jsonrpc-core = '15.1.0'
structopt = '0.3.8'
#--snip--
# *** Add the following lines ***

[dependencies.pallet-contracts]
git = 'https://github.com/paritytech/substrate.git'
tag = 'latest'
version = '4.0.0-dev'

[dependencies.pallet-contracts-rpc]
git = 'https://github.com/paritytech/substrate.git'
tag = 'latest'
version = '4.0.0-dev'
```

• node/src/rpc.rs在文本编辑器中打开文件。

Substrate 提供了一个 RPC 来与节点交互。但是，默认情况下它不包含对合同托盘的访问。要与合同托盘交互，您必须扩展现有的 RPC 并添加合同托盘及其 API。
```
use node_template_runtime::{opaque::Block, AccountId, Balance, Index, BlockNumber, Hash}; // NOTE THIS IS AN ADJUSTMENT TO AN EXISTING LINE
use pallet_contracts_rpc::{Contracts, ContractsApi};
   /* --snip-- */

/// Instantiate all full RPC extensions.
pub fn create_full<C, P>(
   deps: FullDeps<C, P>,
) -> jsonrpc_core::IoHandler<sc_rpc::Metadata> where
   /* --snip-- */
   C: Send + Sync + 'static,
   C::Api: substrate_frame_rpc_system::AccountNonceApi<Block, AccountId, Index>,
   /*** Add This Line ***/
   C::Api: pallet_contracts_rpc::ContractsRuntimeApi<Block, AccountId, Balance, BlockNumber, Hash>,
   /* --snip-- */
{
   /* --snip-- */
   io.extend_with(
		TransactionPaymentApi::to_delegate(TransactionPayment::new(client.clone()))
   );
   /*** Add this block ***/
   // Contracts RPC API extension
   io.extend_with(
		ContractsApi::to_delegate(Contracts::new(client.clone()))
   );
   /*** End added block ***/
   io
}
```

• 保存更改并关闭node/src/rpc.rs文件。

• 通过运行以下命令检查您的运行时是否正确编译：
```
cargo check -p node-template-runtime
```

• 通过运行以下命令以发布模式编译节点：
```
cargo build --release
```

# <span id='index6'>• 启动本地 Substrate 节点</span>  
在您的节点编译后，您就可以启动已通过Contracts 托盘中的智能合约功能增强的 Substrate 节点，并使用前端模板与其交互。

启动本地节点：

• 如有必要，打开终端外壳。

• 切换到 Substrate 节点模板的根目录。

• 通过运行以下命令以开发模式启动节点：
```
./target/release/node-template --dev
```

• 在新终端中，切换到安装前端模板的根目录。

• 通过运行以下命令启动前端模板的 Web 服务器：
```
yarn start
```

• 在浏览器中打开http://localhost:8000/以查看前端模板。

• 在 Pallet Interactor 组件中，确认选择了 Extrinsic。

• contracts从可调用的托盘列表中选择。
![image](https://user-images.githubusercontent.com/28084126/177032033-3368be4f-a608-41c5-9471-124796add60f.png)

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
![微信公众号565 ee](https://user-images.githubusercontent.com/28084126/177032090-535f3bf7-db13-4875-a4a0-357887c91b5c.png)
