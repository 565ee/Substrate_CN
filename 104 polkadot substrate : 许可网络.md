• [介绍](#index1)  
• [学习成果](#index2)  
• [构建节点模板](#index3)  
• [添加node-authorization托盘](#index4)  
• [为我们的托盘添加创世纪存储](#index5)  
• [获取节点密钥和 PeerID](#index6)  
• [Alice 和 Bob 启动网络](#index7)  
• [添加 charlie 的节点](#index8)  
• [将 Dave 作为子节点添加到 Charlie](#index9)  
• [Substrate Tutorials , Substrate 教程](#index98)  
• [Contact 联系方式](#index99)  

# <span id='index1'>• 介绍</span>  
在本教程中，您将学习如何使用 节点授权托盘使用 Substrate 构建许可网络。本教程大约需要1 小时才能完成。

您可能已经熟悉公共或无需许可的区块链，每个人都可以通过运行节点自由加入网络。在许可网络中，仅允许授权节点执行特定活动，例如验证块和传播交易。可能需要许可区块链的一些示例：

• 私有（或联盟）网络
• 高度监管的数据环境（医疗保健、B2B 分类账等）
• 大规模测试公共网络

在您开始之前，我们希望：

• 您已完成 构建 PoE 分散式应用程序教程。
• 您在概念上熟悉 Substrate 中的P2P 网络。我们建议先完成专用网络教程 以获取相关经验。

# <span id='index2'>• 学习成果</span>  
• 了解如何在运行时使用节点授权托盘
• 了解如何创建由多个节点组成的许可网络

# <span id='index3'>• 构建节点模板</span>  
• 克隆节点模板。
```
git clone https://github.com/substrate-developer-hub/substrate-node-template
git checkout latest
```
• 构建节点模板。
```
cd substrate-node-template/
cargo build --release
```

# <span id='index4'>• 添加node-authorization托盘</span>  
• 首先，我们必须将托盘添加到我们的运行时依赖项中：
runtime/Cargo.toml

```
pallet-node-authorization = { default-features = false, git = "https://github.com/paritytech/substrate.git", branch = "polkadot-v0.9.23" }

#--snip--
[features]
default = ['std']
std = [
    #--snip--
    'pallet-node-authorization/std',
    #--snip--
]
```

• 我们需要在我们简单的区块链中模拟治理，所以我们只需要一个sudo管理规则，将托盘的接口配置为EnsureRoot. 在生产环境中，我们应该希望在这里实现基于治理的检查。
runtime/src/lib.rs
```

/* --snip-- */

use frame_system::EnsureRoot;

/* --snip-- */

parameter_types! {
	pub const MaxWellKnownNodes: u32 = 8;
	pub const MaxPeerIdLength: u32 = 128;
}

impl pallet_node_authorization::Config for Runtime {
	type Event = Event;
	type MaxWellKnownNodes = MaxWellKnownNodes;
	type MaxPeerIdLength = MaxPeerIdLength;
	type AddOrigin = EnsureRoot<AccountId>;
	type RemoveOrigin = EnsureRoot<AccountId>;
	type SwapOrigin = EnsureRoot<AccountId>;
	type ResetOrigin = EnsureRoot<AccountId>;
	type WeightInfo = ();
}

/* --snip-- */
```

• construct_runtime最后，我们准备使用以下额外的代码行将我们的托盘放入宏中：
runtime/src/lib.rs

```
construct_runtime!(
	pub enum Runtime where
		Block = Block,
		NodeBlock = opaque::Block,
		UncheckedExtrinsic = UncheckedExtrinsic
	{
		/* --snip-- */
		NodeAuthorization: pallet_node_authorization, // <-- add this line
		/* --snip-- */
	}
);
```

# <span id='index5'>• 为我们的托盘添加创世纪存储</span>  
• PeerId以 bs58 格式编码，因此我们需要在node/Cargo.toml中新建一个库bs58来对其进行解码以获取其字节。
node/cargo.toml
```
[dependencies]
#--snip--
bs58 = "0.4.0"
#--snip--
```

• 现在我们在node/src/chain_spec.rs添加一个适当的创世存储。同样，导入必要的依赖项：
节点/src/chain_spec.rs
```
/* --snip-- */
use sp_core::OpaquePeerId; // A struct wraps Vec<u8>, represents as our `PeerId`.
use node_template_runtime::NodeAuthorizationConfig; // The genesis config that serves for our pallet.
/* --snip-- */
```

• 在辅助函数中添加我们的 genesis 配置testnet_genesis，
节点/src/chain_spec.rs
```
/// Configure initial storage state for FRAME modules.
fn testnet_genesis(
	wasm_binary: &[u8],
	initial_authorities: Vec<(AuraId, GrandpaId)>,
	root_key: AccountId,
	endowed_accounts: Vec<AccountId>,
	_enable_println: bool,
) -> GenesisConfig {

		/* --snip-- */

	/*** Add This Block Item ***/
		node_authorization: NodeAuthorizationConfig {
			nodes: vec![
				(
					OpaquePeerId(bs58::decode("12D3KooWBmAwcd4PJNJvfV89HwE48nwkRmAgo8Vy3uQEyNNHBox2").into_vec().unwrap()),
					endowed_accounts[0].clone()
				),
				(
					OpaquePeerId(bs58::decode("12D3KooWQYV9dGMFoRzNStwpXztXaBUjtPqi6aU76ZgUriHhKust").into_vec().unwrap()),
					endowed_accounts[1].clone()
				),
			],
		},

	/* --snip-- */

}
```

# <span id='index6'>• 获取节点密钥和 PeerID</span>  
• 对于 Alice 的知名节点：
```
# Node Key
c12b6d18942f5ee8528c8e2baf4e147b5c5c18710926ea492d09cbd9f6c9f82a

# Peer ID, generated from node key
12D3KooWBmAwcd4PJNJvfV89HwE48nwkRmAgo8Vy3uQEyNNHBox2

# BS58 decoded Peer ID in hex:
0x0024080112201ce5f00ef6e89374afb625f1ae4c1546d31234e87e3c3f51a62b91dd6bfa57df
```

• 对于 Bob 的知名节点：
```
# Node Key
6ce3be907dbcabf20a9a5a60a712b4256a54196000a8ed4050d352bc113f8c58

# Peer ID, generated from node key
12D3KooWQYV9dGMFoRzNStwpXztXaBUjtPqi6aU76ZgUriHhKust

# BS58 decoded Peer ID in hex:
0x002408011220dacde7714d8551f674b8bb4b54239383c76a2b286fa436e93b2b7eb226bf4de7
```

• 对于查理的不知名节点：
```
# Node Key
3a9d5b35b9fb4c42aafadeca046f6bf56107bd2579687f069b42646684b94d9e

# Peer ID, generated from node key
12D3KooWJvyP3VJYymTqG7eH4PM5rN4T2agk5cdNCfNymAqwqcvZ

# BS58 decoded Peer ID in hex:
0x002408011220876a7b4984f98006dc8d666e28b60de307309835d775e7755cc770328cdacf2e
```

• 对于 Dave 的子节点
```
# Node Key
a99331ff4f0e0a0434a6263da0a5823ea3afcfffe590c9f3014e6cf620f2b19a

# Peer ID, generated from node key
12D3KooWPHWFrfaJzxPnqnAYAoRUyAHHKqACmEycGTVmeVhQYuZN

# BS58 decoded Peer ID in hex:
0x002408011220c81bc1d7057a1511eb9496f056f6f53cdfe0e14c8bd5ffca47c70a8d76c1326d
```

Alice 和 Bob 的节点已经在创世存储中配置并作为众所周知的节点。我们稍后会将 Charlie 的节点添加到众所周知的节点集中。最后，我们将在 Charlie 的节点和 Dave 的节点之间添加连接，而不使 Dave 的节点成为众所周知的节点。

# <span id='index7'>• Alice 和 Bob 启动网络</span>  
• 我们先启动 Alice 的节点：
```
./target/release/node-template \
--chain=local \
--base-path /tmp/validator1 \
--alice \
--node-key=c12b6d18942f5ee8528c8e2baf4e147b5c5c18710926ea492d09cbd9f6c9f82a \
--port 30333 \
--ws-port 9944
```

• 启动 Bob 的节点：
```
# In a new terminal, leave Alice running
./target/release/node-template \
--chain=local \
--base-path /tmp/validator2 \
--bob \
--node-key=6ce3be907dbcabf20a9a5a60a712b4256a54196000a8ed4050d352bc113f8c58 \
--port 30334 \
--ws-port 9945
```

两个节点都启动后，您应该能够在麻烦终端日志中看到创建和完成的新块。现在让我们使用 polkadot.js 应用程序 并检查我们区块链的知名节点。不要忘记切换到我们正在运行的本地节点之一：127.0.0.1:9944或127.0.0.1:9945.

然后，我们进入开发者页面，Chain State 子选项卡，查看nodeAuthorization托盘、wellKnownNodesstorage 中存储的数据。您应该能够看到 Alice 和 Bob 节点的 peer id，前缀为0x以十六进制格式显示其字节。

我们也可以通过查询存储来查询一个节点的所有者，owners输入节点的peer id，你应该得到所有者的账户地址。

# <span id='index8'>• 添加 charlie 的节点</span>  
• 让我们启动查理的节点，
```
./target/release/node-template \
--chain=local \
--base-path /tmp/validator3 \
--name charlie  \
--node-key=3a9d5b35b9fb4c42aafadeca046f6bf56107bd2579687f069b42646684b94d9e \
--port 30335 \
--ws-port=9946 \
--offchain-worker always
```

• 转到应用程序中的开发人员页面，Sudo选项卡，然后使用查理节点的十六进制对等 ID提交nodeAuthorization- 调用，当然所有者是查理。add_well_known_node注意 Alice 是此调用的有效 sudo 来源。
<img width="1257" alt="add_well_known_node" src="https://user-images.githubusercontent.com/28084126/173120742-3ef1fe10-e3a8-4621-9fcb-90631e2cb05d.png">

# <span id='index9'>• 将 Dave 作为子节点添加到 Charlie</span>  
• 让我们添加 Dave 的节点，不是作为知名节点，而是 Charlie 的“子节点”。Dave 将只能连接到 Charlie 以访问网络。这是一项安全功能：因此 Charlie 独自负责任何连接的子节点对等点。大卫有一个访问控制点，以防他们需要被删除或审核。

使用以下命令启动 Dave 的节点：
```
./target/release/node-template \
--chain=local \
--base-path /tmp/validator4 \
--name dave \
--node-key=a99331ff4f0e0a0434a6263da0a5823ea3afcfffe590c9f3014e6cf620f2b19a \
--port 30336 \
--ws-port 9947 \
--offchain-worker always
```

启动后，没有可用的连接。这是一个许可网络，所以首先，Charlie 需要配置他的节点以允许来自 Dave 节点的连接。

在Developer Extrinsics页面中，让 Charlie 提交一个addConnectionsextrinsic。第一个 PeerId 是 Charlie 节点的十六进制对等 ID。连接是 Charlie 节点允许的 peer id 列表，这里我们只添加 Dave 的。
<img width="1263" alt="charlie_add_connections" src="https://user-images.githubusercontent.com/28084126/173121185-83b8e81d-7cd2-4efc-8db4-747487330489.png">

然后，Dave 需要配置他的节点以允许来自 Charlie 节点的连接。但是在他添加这个之前，Dave 需要声明他的节点，希望还不算太晚！
<img width="1257" alt="dave_claim_node" src="https://user-images.githubusercontent.com/28084126/173121410-36f97f77-7fa4-499e-ab6f-c8014f5d2d07.png">

同样，Dave 可以从 Charlie 的节点添加连接。
<img width="1261" alt="dave_add_connections" src="https://user-images.githubusercontent.com/28084126/173121851-6ced972d-d4c5-4c77-bcfd-007678bae3b2.png">

您现在应该看到 Dave 正在追赶积木，并且只有一个属于 Charlie 的同伴！重新启动 Dave 的节点，以防它没有立即与 Charlie 连接。

您已完成本教程，并且已经了解了如何构建许可网络。您还可以使用其他可调度的调用，例如 remove_well_known_node, remove_connections。

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
![微信公众号565 ee](https://user-images.githubusercontent.com/28084126/173124217-442bc946-dda4-4d1b-b746-13c9391753bf.png)
