• [介绍](#index1)  
• [准备工作](#index2)  
• [目标](#index3)  
• [设计应用程序](#index4)  
• [为您的托盘搭建脚手架](#index5)  
• [配置托盘以发出事件](#index6)  
• [实施托盘事件](#index7)  
• [托盘错误](#index8)  
• [为存储的项目实现存储映射](#index9)  
• [实现可调用函数](#index10)  
• [包括MaxBytesInHash运行时配置](#index11)  
• [使用新托盘构建运行时](#index12)  
• [添加您的自定义反应组件](#index13)  
• [提交证明](#index14)  
• [Substrate Tutorials , Substrate 教程](#index98)  
• [Contact 联系方式](#index99) 

# <span id='index1'>• 介绍</span>  
本教程说明如何使用 Substrate 区块链开发框架和FRAME库创建自定义存在证明 (PoE) 服务。

存在证明是一种通过使用存储在区块链上的对象信息来验证数字对象的真实性和所有权的方法。因为区块链将时间戳​​和签名与对象相关联，所以区块链记录可用于验证（作为证明）特定对象在特定日期和时间存在。它还可以验证记录的所有者在该日期和时间是谁。

区块链不是使用单个文件，而是使用加密哈希存储数字记录。哈希使区块链能够通过使用小而唯一的哈希值有效地存储任意大小的文件。因为对文件的任何更改都会导致不同的哈希值，用户可以通过计算哈希值并将该哈希值与存储在链上的哈希值进行比较来证明文件的有效性。

区块链使用公钥将数字身份映射到拥有私钥的账户。区块链记录您用于存储数字对象哈希的帐户，作为交易的一部分。由于账户信息是作为交易的一部分存储的，因此账户的控制者可以稍后证明所有权是最初上传文件的人。

# <span id='index2'>• 准备工作</span>  
通过安装Rust 和 Rust 工具链，您已经为 Substrate 开发配置了环境。

您已完成 创建您的第一个 Substrate 区块链并安装了节点和 前端模板。

您通常熟悉软件开发并使用命令行界面。

# <span id='index3'>• 目标</span>  
了解定制托盘的基本结构。

查看 Rust 宏如何简化您需要编写的代码的示例。

启动一个包含自定义托盘的区块链节点。

添加暴露存在证明托盘的前端代码。

# <span id='index4'>• 设计应用程序</span>  
存在证明应用程序公开了以下可调用函数：

create_claim()允许用户通过上传哈希来声明文件的存在。

revoke_claim()允许索赔的当前所有者撤销所有权。

这些功能只需要您存储有关已声明的证明以及提出这些声明的人的信息。

# <span id='index5'>• 为您的托盘搭建脚手架</span>  
Substrate 节点模板有一个基于 FRAME 的运行时。 FRAME是一个代码库，允许您通过组合称为“pallets”的模块来构建 Substrate 运行时。您可以将托盘视为定义区块链可以做什么的单独逻辑片段。Substrate 为您提供了许多用于基于 FRAME 的运行时的预构建托盘。

本教程演示如何从头开始创建自定义托盘。因此，第一步是从节点模板目录中的文件中删除一些文件和内容。

• 打开终端外壳并导航到节点模板的根目录。

• pallets/template/src通过运行以下命令切换到目录：
```
cd pallets/template/src
```

• 删除以下文件：
```
benchmarking.rs
mock.rs
tests.rs
```

• pallets/template/src/lib.rs在文本编辑器中打开文件。

此文件包含可用作新托盘模板的代码。您不会在本教程中使用模板代码。但是，您可以在删除模板代码之前查看它提供的内容。

•  的所有内容替换pallets/template/src/lib.rs为以下框架代码，其中包含FRAME V2 的一些最小宏：
```
  #![cfg_attr(not(feature = "std"), no_std)]

  pub use pallet::*;

  #[frame_support::pallet]
  pub mod pallet {
      use frame_support::pallet_prelude::*;
      use frame_system::pallet_prelude::*;

      // The struct on which we build all of our Pallet logic.
      #[pallet::pallet]
      #[pallet::generate_store(pub(super) trait Store)]
      pub struct Pallet<T>(_);

      /* Placeholder for defining custom types. */

      // TODO: Update the `config` block below
      #[pallet::config]
      pub trait Config: frame_system::Config {
          type Event: From<Event<Self>> + IsType<<Self as frame_system::Config>::Event>;
      }

      // TODO: Update the `event` block below
      #[pallet::event]
      #[pallet::generate_deposit(pub(super) fn deposit_event)]
      pub enum Event<T: Config> {}

      // TODO: Update the `error` block below
      #[pallet::error]
      pub enum Error<T> {}

      // TODO: add #[pallet::storage] block

      // TODO: Update the `call` block below
      #[pallet::call]
      impl<T: Config> Pallet<T> {}
  }
```

• 保存您的更改。

• （可选）通过运行以下命令检查您的代码是否编译：
```
cargo check -p node-template-runtime
cargo build -r
```

# <span id='index6'>• 配置托盘以发出事件</span>  
每个托盘都有一个名为Config. 此特征用于设置 FRAME 系统的接口，并将所需的关联类型设置为在包含此托盘的运行时中具体定义。对于本教程，配置设置仅使托盘能够发出事件，就像几乎每个托盘一样。

要定义Config存在证明托盘的特征，请pallets/template/src/lib.rs在文本编辑器中打开文件并更新#[pallet::config]块以匹配以下代码块：
```
	#[pallet::config]
	pub trait Config: frame_system::Config {
		type Event: From<Event<Self>> + IsType<<Self as frame_system::Config>::Event>;
		type MaxBytesInHash: Get<u32>;
	}
```

# <span id='index7'>• 实施托盘事件</span>  
要实现托盘事件，请更新#[pallet::event]块以匹配以下代码块：
```
	#[pallet::event]
	#[pallet::generate_deposit(pub(super) fn deposit_event)]
	pub enum Event<T: Config> {
		ClaimCreated(T::AccountId, BoundedVec<u8, T::MaxBytesInHash>),
		ClaimRevoked(T::AccountId, BoundedVec<u8, T::MaxBytesInHash>),
	}
```

# <span id='index8'>• 托盘错误</span>  
要实现存在证明托盘的错误，请将// TODO: add #[pallet::error] block行替换为以下代码块：
```
	#[pallet::error]
	pub enum Error<T> {
		ProofAlreadyClaimed,
		NoSuchProof,
		NotProofOwner,
	}
```

• [为存储的项目实现存储映射](#index9)  
要实现存在证明托盘的存储，请将// TODO: add #[pallet::storage] block行替换为以下代码块：
```
	#[pallet::storage]
	pub(super) type Proofs<T: Config> = StorageMap<
		_,
		Blake2_128Concat,
		BoundedVec<u8, T::MaxBytesInHash>,
		(T::AccountId, T::BlockNumber),
		OptionQuery,
	>;
```

# <span id='index10'>• 实现可调用函数</span>  
要在存在证明托盘中实现此逻辑，请将// TODO: add #[pallet::call] block行替换为以下代码块：
```
	#[pallet::call]
	impl<T: Config> Pallet<T> {
		#[pallet::weight(1_000)]
		pub fn create_claim(
			origin: OriginFor<T>,
			proof: BoundedVec<u8, T::MaxBytesInHash>,
		) -> DispatchResult {
			let sender = ensure_signed(origin)?;

			ensure!(!Proofs::<T>::contains_key(&proof), Error::<T>::ProofAlreadyClaimed);

			let current_block = <frame_system::Pallet<T>>::block_number();

			Proofs::<T>::insert(&proof, (&sender, current_block));

			// Emit an event that the claim was created.
			Self::deposit_event(Event::ClaimCreated(sender, proof));

			Ok(())
		}

		#[pallet::weight(10_000)]
		pub fn revoke_claim(
			origin: OriginFor<T>,
			proof: BoundedVec<u8, T::MaxBytesInHash>,
		) -> DispatchResult {
			// Check that the extrinsic was signed and get the signer.
			// This function will return an error if the extrinsic is not signed.
			// https://docs.substrate.io/v3/runtime/origins
			let sender = ensure_signed(origin)?;

			// Verify that the specified proof has been claimed.
			ensure!(Proofs::<T>::contains_key(&proof), Error::<T>::NoSuchProof);

			// Get owner of the claim.
			// Panic condition: there is no way to set a `None` owner, so this must always unwrap.
			let (owner, _) = Proofs::<T>::get(&proof).expect("All proofs must have an owner!");

			// Verify that sender of the current call is the claim owner.
			ensure!(sender == owner, Error::<T>::NotProofOwner);

			// Remove claim from storage.
			Proofs::<T>::remove(&proof);

			// Emit an event that the claim was erased.
			Self::deposit_event(Event::ClaimRevoked(sender, proof));
			Ok(())
		}
	}
```

至此，您已经完成了一个托盘！ 现在要使用托盘，我们必须在您的运行时正确配置它。

# <span id='index11'>• 包括MaxBytesInHash运行时配置</span>  
您应该好奇存在证明托盘使用BoundedVec<u8, T::MaxBytesInHash>类型作为证明，但到目前为止我们还没有具体的概念MaxBytesInHash。这个常量应该在运行时设置为可以在你的区块链中使用的合理值。许多 web3 应用程序中使用的一种非常典型的哈希类型是CID，并且这些类型的 V1 实例通常小于64 bytes长度。所以在这里我们MaxBytesInHash在运行时指定为这个长度（或更小）：

• runtime/src/lib.rs在文本编辑器中打开文件。

• 更新pallet_template::Config块以包括：
```
impl pallet_template::Config for Runtime {
	type Event = Event;
	type MaxBytesInHash = frame_support::traits::ConstU32<64>;
}
```

• 保存更改并关闭文件。

• （可选）通过运行以下命令检查您的代码是否编译：
```
cargo check -p node-template-runtime
```

# <span id='index12'>• 使用新托盘构建运行时</span>  
将存在证明托盘的所有部分复制到pallets/template/lib.rs文件中后，就可以编译并启动节点了。

编译并启动更新的 Substrate 节点：

• 打开终端外壳。

• 切换到节点模板的根目录。

• 通过运行以下命令编译节点模板：
```
cargo build --release
```

• 通过运行以下命令以开发模式启动节点：
```
./target/release/node-template --dev
```
该选项使用预定义的链规范--dev启动节点。development使用该--dev选项可确保您在任何时候停止和重新启动节点时都处于干净的工作状态。

• 验证节点产生块。

# <span id='index13'>• 添加您的自定义反应组件</span>  
• 在您的计算机上打开一个新的终端 shell，然后切换到安装前端模板的根目录。

• src/TemplateModule.js在文本编辑器中打开文件。

• 删除该文件的全部内容。

• 将以下代码复制并粘贴到src/TemplateModule.js文件中：
```
import React, { useEffect, useState } from 'react'
import { Form, Input, Grid, Message } from 'semantic-ui-react'

// Pre-built Substrate front-end utilities for connecting to a node
// and making a transaction.
import { useSubstrateState } from './substrate-lib'
import { TxButton } from './substrate-lib/components'

// Polkadot-JS utilities for hashing data.
import { blake2AsHex } from '@polkadot/util-crypto'

// Main Proof Of Existence component
function Main(props) {
  // Establish an API to talk to the Substrate node.
  const { api, currentAccount } = useSubstrateState()
  // React hooks for all the state variables we track.
  // Learn more at: https://reactjs.org/docs/hooks-intro.html
  const [status, setStatus] = useState('')
  const [digest, setDigest] = useState('')
  const [owner, setOwner] = useState('')
  const [block, setBlock] = useState(0)

  // Our `FileReader()` which is accessible from our functions below.
  let fileReader;
  // Takes our file, and creates a digest using the Blake2 256 hash function
  const bufferToDigest = () => {
    // Turns the file content to a hexadecimal representation.
    const content = Array.from(new Uint8Array(fileReader.result))
      .map(b => b.toString(16).padStart(2, '0'))
      .join('');
    const hash = blake2AsHex(content, 256);
    setDigest(hash);
  };

  // Callback function for when a new file is selected.
  const handleFileChosen = file => {
    fileReader = new FileReader();
    fileReader.onloadend = bufferToDigest;
    fileReader.readAsArrayBuffer(file);
  };

  // React hook to update the owner and block number information for a file
  useEffect(() => {
    let unsubscribe;
    // Polkadot-JS API query to the `proofs` storage item in our pallet.
    // This is a subscription, so it will always get the latest value,
    // even if it changes.
    api.query.templateModule
      .proofs(digest, result => {
        // Our storage item returns a tuple, which is represented as an array.
        if (result.inspect().inner) {
          let [tmpAddress, tmpBlock] = result.toHuman()
          setOwner(tmpAddress)
          setBlock(tmpBlock)
        } else {
          setOwner('')
          setBlock(0)
        }
      })
      .then(unsub => {
        unsubscribe = unsub;
      });
    return () => unsubscribe && unsubscribe();
    // This tells the React hook to update whenever the file digest changes
    // (when a new file is chosen), or when the storage subscription says the
    // value of the storage item has updated.
  }, [digest, api.query.templateModule])

  // We *assume* a file digest is claimed if the stored block number is not 0
  function isClaimed() {
    return block !== 0
  }

  // The actual UI elements which are returned from our component.
  return (
    <Grid.Column>
      <h1>Proof of Existence</h1>
      {/* Show warning or success message if the file is or is not claimed. */}
      <Form success={!!digest && !isClaimed()} warning={isClaimed()}>
        <Form.Field>
          {/* File selector with a callback to `handleFileChosen`. */}
          <Input
            type="file"
            id="file"
            label="Your File"
            onChange={e => handleFileChosen(e.target.files[0])}
          />
          {/* Show this message if the file is available to be claimed */}
          <Message success header="File Digest Unclaimed" content={digest} />
          {/* Show this message if the file is already claimed. */}
          <Message
            warning
            header="File Digest Claimed"
            list={[digest, `Owner: ${owner}`, `Block: ${block}`]}
          />
        </Form.Field>
        {/* Buttons for interacting with the component. */}
        <Form.Field>
          {/* Button to create a claim. Only active if a file is selected, and not already claimed. Updates the `status`. */}
          <TxButton
            label="Create Claim"
            type="SIGNED-TX"
            setStatus={setStatus}
            disabled={isClaimed() || !digest}
            attrs={{
              palletRpc: 'templateModule',
              callable: 'createClaim',
              inputParams: [digest],
              paramFields: [true]
            }}
          />
          {/* Button to revoke a claim. Only active if a file is selected, and is already claimed. Updates the `status`. */}
          <TxButton
            label="Revoke Claim"
            type="SIGNED-TX"
            setStatus={setStatus}
            disabled={!isClaimed() || owner !== currentAccount.address}
            attrs={{
              palletRpc: 'templateModule',
              callable: 'revokeClaim',
              inputParams: [digest],
              paramFields: [true]
            }}
          />
        </Form.Field>
        {/* Status message about the transaction. */}
        <div style={{ overflowWrap: 'break-word' }}>{status}</div>
      </Form>
    </Grid.Column>
  );
}

export default function TemplateModule(props) {
  const { api } = useSubstrateState()
  return api.query.templateModule ? <Main {...props} /> : null

}
```

• 保存更改并关闭文件。

• 通过运行以下命令启动前端模板：
```
nvm install # use the correct node version
yarn        # instal deps
yarn start  # start a dev server
```

# <span id='index14'>• 提交证明</span>  
要使用新的前端组件测试存在证明托盘：

• 在页面底部找到组件。

• 单击选择文件并选择计算机上的任何文件。

存在证明托盘为所选文件生成哈希并将其显示在“文件摘要”字段中。

由于该文件没有所有者或块号，因此可以声明它。

• 单击创建声明以获取文件的所有权。
![poe-component](https://user-images.githubusercontent.com/28084126/172207509-65515ab2-1bfc-4d5d-ae5b-7fa48f33331c.png)

单击创建声明调用create_claim自定义存在证明托盘中的函数。前端组件显示已完成交易的文件摘要、帐户标识符和块号。

• 验证声明是否成功并且新claimCreated事件出现在事件组件中。
![poe-claimed](https://user-images.githubusercontent.com/28084126/172208034-8d080858-d323-4196-8288-d9ffc75eb7bb.png)

前端组件识别出该文件现在已被声明，并为您提供撤销声明的选项。

请记住，只有所有者才能撤销索赔。如果您选择另一个用户帐户，则撤销选项将被禁用。

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

![微信公众号565 ee](https://user-images.githubusercontent.com/28084126/172208901-cc3d754f-5305-451c-a789-7df3784508be.png)
