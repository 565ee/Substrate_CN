• [介绍](#index1)  
• [ERC-20 标准的基础知识](#index2)  
• [创建代币供应](#index3)  
• [上传并实例化合约](#index4)  
• [转移代币](#index5)  
• [添加转会事件](#index6)  
• [发出事件](#index7)  
• [添加审批逻辑](#index8)  
• [添加从逻辑转移](#index9)   
• [Substrate Tutorials , Substrate 教程](#index98)  
• [Contact 联系方式](#index99) 

# <span id='index1'>• 介绍</span>  
本教程说明了如何使用墨水构建 ERC-20 代币合约！语。
ERC-20 规范定义了可替代代币的通用标准。
拥有定义令牌的属性的标准使遵循规范的开发人员能够构建可以与其他产品和服务互操作的应用程序。

ERC-20 代币标准并不是唯一的代币标准，但却是最常用的一种。

• 教程目标

通过完成本教程，您将实现以下目标：

- 了解 ERC-20 标准中定义的基本属性和接口。

- 创建符合 ERC-20 标准的代币。

- 在合约之间转移代币。

- 处理涉及批准或第三方的转移活动的路由。

- 创建与令牌活动相关的事件。

# <span id='index2'>• ERC-20 标准的基础知识</span>  
[ERC-20 代币标准](https://eips.ethereum.org/EIPS/eip-20) 定义了运行在以太坊区块链上的大多数智能合约的接口。
这些标准接口允许个人在现有的智能合约平台之上部署自己的加密货币。

如果您查看该标准，您会注意到定义了以下核心功能。

```javascript
// ----------------------------------------------------------------------------
// ERC Token Standard #20 Interface
// https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md
// ----------------------------------------------------------------------------

contract ERC20Interface {
    // Storage Getters
    function totalSupply() public view returns (uint);
    function balanceOf(address tokenOwner) public view returns (uint balance);
    function allowance(address tokenOwner, address spender) public view returns (uint remaining);

    // Public Functions
    function transfer(address to, uint tokens) public returns (bool success);
    function approve(address spender, uint tokens) public returns (bool success);
    function transferFrom(address from, address to, uint tokens) public returns (bool success);

    // Contract Events
    event Transfer(address indexed from, address indexed to, uint tokens);
    event Approval(address indexed tokenOwner, address indexed spender, uint tokens);
}
```

用户余额映射到账户地址，界面允许用户转移他们拥有的代币或允许第三方代表他们转移代币。
最重要的是，必须实施智能合约逻辑以确保资金不会被无意创建或销毁，并且用户的资金不会受到恶意行为者的影响。

请注意，所有公共函数都返回一个“bool”，仅指示调用是否成功。
在 Rust 中，这些函数通常会返回一个 `Result`。

# <span id='index3'>• 创建代币供应</span>  
处理 ERC-20 代币的智能合约类似于 Incrementer 合约，它使用映射来存储 [使用映射存储值](/tutorials/smart-contracts/use-mapping/) 中的值。
对于本教程，ERC-20 合约由固定供应的代币组成，这些代币在部署合约时全部存入与合约所有者关联的账户。
然后合约所有者可以将代币分发给其他用户。

您在本教程中创建的简单 ERC-20 合约并不代表您铸造和分发代币的唯一方式。
但是，这份 ERC-20 合约为扩展您在其他教程中学到的内容以及如何使用墨水提供了良好的基础！用于构建更强大的智能合约的语言。

对于 ERC-20 代币合约，初始存储包括：

- `total_supply` 代表合约中代币的总供应量。
- `balances` 代表每个账户的个人余额。

首先，让我们用一些模板代码创建一个新项目。

要构建 ERC-20 代币智能合约：

•  在您的本地计算机上打开一个终端外壳，如果您还没有打开的话。

•  通过运行以下命令创建一个名为 `erc20` 的新项目：

   ```bash
   cargo contract new erc20
   ```

•  通过运行以下命令切换到新项目目录：

   ```bash
   cd erc20/
   ```

•  在文本编辑器中打开 `lib.rs` 文件。

•  将默认模板源代码替换为新的 [erc20](https://docs.substrate.io/assets/tutorials/ink-workshop/erc-template-lib-0.rs ) 源代码。

•  保存对 `lib.rs` 文件的更改，然后关闭该文件。

•  在文本编辑器中打开 `Cargo.toml` 文件并查看合约的依赖项。

•  在 `[dependencies]` 部分，如有必要，修改 `scale` 和 `scale-info` 设置。

   ```toml
   scale = { package = "parity-scale-codec", version = "3", default-features = false, features = ["derive"] }
   scale-info = { version = "2", default-features = false, features = ["derive"], optional = true }
   ```

•  保存对 `Cargo.toml` 文件的更改，然后关闭该文件。

•  通过运行以下命令验证程序是否编译并通过了简单的测试

   ```bash
   cargo +nightly test
   ```

该命令应显示类似于以下内容的输出，以指示测试成功完成：

   ```text
   running 2 tests
   test erc20::tests::new_works ... ok
   test erc20::tests::balance_works ... ok

   test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
   ```

•  通过运行以下命令验证您是否可以为合约构建 WebAssembly：

   ```bash
   cargo +nightly contract build
   ```

如果程序编译成功，您就可以以当前状态上传它或开始向合约添加功能。

# <span id='index4'>• 上传并实例化合约</span>  
如果你想测试你目前拥有的东西，你可以使用 [Contracts UI](https://contracts-ui.substrate.io) 上传合约。

在添加新功能之前测试 ERC-20 合约：

• 启动本地合约节点。

• 上传`erc20.contract`文件。

•  为 `new` 构造函数指定初始令牌供应。

•  在运行的本地节点上实例化合约。

•  选择 `totalSupply` 作为要发送的消息，然后单击 **Read** 以验证代币的总供应量与初始供应量相同。

•  选择“balanceOf”作为要发送的消息。

•  选择用于实例化合约的账户的`AccountId`，然后点击**Read**。

如果您选择任何其他 `AccountId`，然后单击 **Read**，余额为零，因为所有代币都归合约所有者所有。

# <span id='index5'>• 转移代币</span>  
此时，ERC-20 合约有一个用户账户，拥有该合约的total_supply 代币。
为了使该合约有用，合约所有者必须能够将代币转移到其他账户。

对于这个简单的 ERC-20 合约，您将添加一个公共的“转移”函数，使您作为合约调用者能够将您拥有的代币转移给另一个用户。

公共 `transfer` 函数调用私有 `transfer_from_to()` 函数。
因为这是一个内部函数，所以可以在没有任何授权检查的情况下调用它。
但是，转移的逻辑必须能够确定 `from` 帐户是否有可转移到接收 `to` 帐户的代币。
`transfer_from_to()` 函数使用合约调用者（`self.env().caller()`）作为`from` 账户。
在这种情况下，`transfer_from_to()` 函数会执行以下操作：

- 获取 `from` 和 `to` 账户的当前余额。

- 检查 `from` 余额是否小于要发送的代币的 `value` 数量。

  ```rust
  let from_balance = self.balance_of(from);
    if from_balance < value {
    return Err(Error::InsufficientBalance)
  }
  ```

- 从转账账户中减去“价值”，并将“价值”添加到接收账户。

  ```rust
  self.balances.insert(from, &(from_balance - value));
  let to_balance = self.balance_of(to);
  self.balances.insert(to, &(to_balance + value));
  ```

将转账函数添加到智能合约：

•  在您的本地计算机上打开一个终端外壳，如果您还没有打开的话。

•  验证您是否在 `erc20` 项目目录中。

•  在文本编辑器中打开 `lib.rs`。

账户中没有足够的token来完成转账，则返回错误。

   ```rust
   /// Specify ERC-20 error type.
   #[derive(Debug, PartialEq, Eq, scale::Encode, scale::Decode)]
   #[cfg_attr(feature = "std", derive(scale_info::TypeInfo))]
   pub enum Error {
   /// Return if the balance cannot fulfill a request.
       InsufficientBalance,
   }
   ```

•  添加`Result`返回类型，返回`InsufficientBalance`错误。

   ```rust
   /// Specify the ERC-20 result type.
   pub type Result<T> = core::result::Result<T, Error>;
   ```

•  添加`transfer()`公共函数，使合约调用者可以将代币转移到另一个账户。

   ```rust
   #[ink(message)]
   pub fn transfer(&mut self, to: AccountId, value: Balance) -> Result<()> {
       let from = self.env().caller();
       self.transfer_from_to(&from, &to, value)
   }
   ```

•  添加`transfer_from_to()`私有函数，将代币从合约调用者关联的账户转移到接收账户。

   ```rust
   fn transfer_from_to(
       &mut self,
       from: &AccountId,
       to: &AccountId,
       value: Balance,
   ) -> Result<()> {
        let from_balance = self.balance_of_impl(from);
        if from_balance < value {
            return Err(Error::InsufficientBalance)
        }

        self.balances.insert(from, &(from_balance - value));
        let to_balance = self.balance_of_impl(to);
        self.balances.insert(to, &(to_balance + value));
        Ok(())
   }
   ```

此代码片段使用 `balance_of_impl()` 函数。
`balance_of_impl()` 函数与 `balance_of` 函数相同，只是它使用引用在 WebAssembly 中以更有效的方式查找帐户余额。
将以下函数添加到智能合约中以使用此功能：

   ```rust
   #[inline]
   fn balance_of_impl(&self, owner: &AccountId) -> Balance {
       self.balances.get(owner).unwrap_or_default()
   }
   ```

•  通过运行以下命令验证程序是否编译并通过了测试用例：

   ```bash
   cargo +nightly test
   ```

该命令应显示类似于以下内容的输出，以指示测试成功完成：

   ```text
   running 3 tests
   test erc20::tests::new_works ... ok
   test erc20::tests::balance_works ... ok
   test erc20::tests::transfer_works ... ok

   test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
   ```

# <span id='index6'>• 添加转会事件</span>  
在本教程中，您将声明一个 `Transfer` 事件以提供有关已完成传输操作的信息。
`Transfer` 事件包含以下信息：

- `Balance` 类型的值。
- 用于 `from` 帐户的选项包装的 `AccountId` 变量。
- 用于 `to` 帐户的选项包装的 `AccountId` 变量。

为了更快地访问事件数据，他们可以拥有_indexed fields_。
您可以通过在该字段上使用 `#[ink(topic)]` 属性标记来执行此操作。

添加 `Transfer` 事件：

•  在文本编辑器中打开 `lib.rs` 文件。

•  使用 `#[ink(event)]` 属性宏声明事件。

   ```rust
   #[ink(event)]
   pub struct Transfer {
       #[ink(topic)]
       from: Option<AccountId>,
       #[ink(topic)]
       to: Option<AccountId>,
       value: Balance,
     }
   ```

您可以使用 `.unwrap_or()` 函数检索 `Option<T>` 变量的数据

# <span id='index7'>• 发出事件</span>  
现在您已经声明了事件并定义了事件包含的信息，您需要添加发出事件的代码。
您可以通过使用事件名称作为调用的唯一参数调用 `self.env().emit_event()` 函数来执行此操作。

在这个 ERC-20 合约中，您希望在每次发生转移时发出一个“转移”事件。
代码中有两个地方出现这种情况：

- 在初始化合约的`new`调用期间。

- 每次调用 `transfer_from_to` 时。

`from` 和 `to` 字段的值是 `Option<AccountId>` 数据类型。
但是，在令牌的初始转移期间，为初始*供应*设置的值
不是来自任何其他帐户。
在这种情况下，Transfer 事件的 `from` 值为 `None`。

要发出 Transfer 事件：

•  在文本编辑器中打开 `lib.rs` 文件。

•  在 `new` 构造函数中的 `new_init()` 函数中添加 `Transfer` 事件。

   ```rust
   fn new_init(&mut self, initial_supply: Balance) {
       let caller = Self::env().caller();
       self.balances.insert(&caller, &initial_supply);
       self.total_supply = initial_supply;
       Self::env().emit_event(Transfer {
           from: None,
           to: Some(caller),
           value: initial_supply,
         });
       }
   ```

•  将 `Transfer` 事件添加到 `transfer_from_to()` 函数中。

   ```rust
   self.balances.insert(from, &(from_balance - value));
   let to_balance = self.balance_of_impl(to);
   self.balances.insert(to, &(to_balance + value));
   self.env().emit_event(Transfer {
       from: Some(*from),
       to: Some(*to),
       value,
   });
   ```

请注意，`value` 不需要 `Some()`，因为该值未存储在 `Option` 中。

•  添加一个将代币从一个账户转移到另一个账户的测试。

   ```rust
   #[ink::test]
   fn transfer_works() {
       let mut erc20 = Erc20::new(100);
       assert_eq!(erc20.balance_of(AccountId::from([0x0; 32])), 0);
       assert_eq!(erc20.transfer((AccountId::from([0x0; 32])), 10), Ok(()));
       assert_eq!(erc20.balance_of(AccountId::from([0x0; 32])), 10);
   }

   ```

•  通过运行以下命令验证程序是否编译并通过所有测试：

   ```bash
   cargo +nightly test
   ```

该命令应显示类似于以下内容的输出，以指示测试成功完成：

   ```text
   running 3 tests
   test erc20::tests::new_works ... ok
   test erc20::tests::balance_works ... ok
   test erc20::tests::transfer_works ... ok

   test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
   ```

# <span id='index8'>• 添加审批逻辑</span>  
批准另一个账户来使用你的代币是第三方转账过程的第一步。
作为代币所有者，您可以指定指定账户可以代表您转移的任意账户和任意数量的代币。
您没有批准您帐户中的所有代币，您可以指定一个已批准帐户允许转移的最大数量。

当您多次调用 `approve` 时，您会用新值覆盖先前批准的值。
默认情况下，任何两个帐户之间的批准值为 `0`。
如果您想撤销对您帐户中代币的访问权限，您可以调用值为 0 的 `approve` 函数。

要在 ERC-20 合约中存储批准，您需要使用稍微复杂的“映射”键。

由于每个帐户可以有不同的金额可供任何其他帐户使用，因此您需要使用元组作为映射到余额值的键。
例如：

```rust
pub struct Erc20 {
 /// Balances that can be transferred by non-owners: (owner, spender) -> allowed
 allowances: ink_storage::Mapping<(AccountId, AccountId), Balance>,
}
```

元组使用`（所有者，支出者）`来标识允许代表`所有者`访问令牌的`支出者`帐户，直到指定的`允许`。

将审批逻辑添加到智能合约：

•  在文本编辑器中打开 `lib.rs` 文件。

•  使用 `#[ink(event)]` 属性宏声明 `Approval` 事件。

```rust
#[ink(event)]
pub struct Approval {
    #[ink(topic)]
    owner: AccountId,
    #[ink(topic)]
    spender: AccountId,
    value: Balance,
}
```

• 添加`Error`声明，如果转账请求超过账户限额，则返回错误。

   ```rust
   #[derive(Debug, PartialEq, Eq, scale::Encode, scale::Decode)]
   #[cfg_attr(feature = "std", derive(scale_info::TypeInfo))]
   pub enum Error {
       InsufficientBalance,
       InsufficientAllowance,
   }
   ```

•  将所有者和非所有者组合的存储映射添加到帐户余额。

   ```rust
   allowances: Mapping<(AccountId, AccountId), Balance>,
   ```

•  增加approve()函数，授权spender账户从调用者账户提取token，最高可达最大值。

   ```rust
   #[ink(message)]
   pub fn approve(&mut self, spender: AccountId, value: Balance) -> Result<()> {
       let owner = self.env().caller();
       self.allowances.insert((&owner, &spender), &value);
       self.env().emit_event(Approval {
         owner,
         spender,
         value,
       });
       Ok(())
   }
   ```

•  添加一个`allowance()`函数返回一个`spender`被允许从`owner`账户中提取的代币数量。

   ```rust
   #[ink(message)]
   pub fn allowance(&self, owner: AccountId, spender: AccountId) -> Balance {
       self.allowance_impl(&owner, &spender)
   }
   ```

此代码片段使用 `allowance_impl()` 函数。
`allowance_impl()` 函数与 `allowance` 函数相同，只是它使用引用在 WebAssembly 中以更有效的方式查找令牌余量。
将以下函数添加到智能合约中以使用此功能：

   ```rust
   #[inline]
   fn allowance_impl(&self, owner: &AccountId, spender: &AccountId) -> Balance {
       self.allowances.get((owner, spender)).unwrap_or_default()
   }
   ```

# <span id='index9'>• 添加从逻辑转移</span>  
现在您已经为一个帐户设置了代表另一个帐户转移代币的批准，您需要创建一个 `transfer_from` 函数以使批准的用户能够转移代币。
`transfer_from` 函数调用私有 `transfer_from_to` 函数来完成大部分传输逻辑。
授权非所有者转移代币有一些要求：

- `self.env().caller()` 合约调用者必须被分配在 `from` 账户中可用的代币。

- 存储为“津贴”的分配必须大于要转移的价值。

如果满足这些要求，合约会将更新后的配额插入到 `allowance` 变量中，并使用指定的 `from` 和 `to` 账户调用 `transfer_from_to()` 函数。

记住在调用 `transfer_from` 时，`self.env().caller()` 和 `from` 账户是用来查找当前配额的，但是在`from` 和 `to 之间调用了 `transfer_from` 函数` 指定的帐户。

每当调用 `transfer_from` 时，都会使用三个帐户变量，您需要确保正确使用它们。

将 `transfer_from` 逻辑添加到智能合约：

•  在文本编辑器中打开 `lib.rs` 文件。

•  添加`transfer_from()`函数，将代币的`value`数量转入`from`账户到`to`账户。

```rust
/// Transfers tokens on the behalf of the `from` account to the `to account
#[ink(message)]
pub fn transfer_from(
    &mut self,
    from: AccountId,
    to: AccountId,
    value: Balance,
) -> Result<()> {
    let caller = self.env().caller();
    let allowance = self.allowance_impl(&from, &caller);
    if allowance < value {
        return Err(Error::InsufficientAllowance)
    }
    self.transfer_from_to(&from, &to, value)?;
    self.allowances
        .insert((&from, &caller), &(allowance - value));
    Ok(())
    }
}
```

•  添加对`transfer_from()`函数的测试。

   ```rust
   #[ink::test]
   fn transfer_from_works() {
    let mut contract = Erc20::new(100);
    assert_eq!(contract.balance_of(AccountId::from([0x1; 32])), 100);
    contract.approve(AccountId::from([0x1; 32]), 20);
    contract.transfer_from(AccountId::from([0x1; 32]), AccountId::from([0x0; 32]), 10);
    assert_eq!(contract.balance_of(AccountId::from([0x0; 32])), 10);
   }
   ```

•  添加对`allowance()`函数的测试。

   ```rust
   #[ink::test]
   fn allowances_works() {
    let mut contract = Erc20::new(100);
    assert_eq!(contract.balance_of(AccountId::from([0x1; 32])), 100);
    contract.approve(AccountId::from([0x1; 32]), 200);
    assert_eq!(contract.allowance(AccountId::from([0x1; 32]), AccountId::from([0x1; 32])), 200);

    contract.transfer_from(AccountId::from([0x1; 32]), AccountId::from([0x0; 32]), 50);
    assert_eq!(contract.balance_of(AccountId::from([0x0; 32])), 50);
    assert_eq!(contract.allowance(AccountId::from([0x1; 32]), AccountId::from([0x1; 32])), 150);

    contract.transfer_from(AccountId::from([0x1; 32]), AccountId::from([0x0; 32]), 100);
    assert_eq!(contract.balance_of(AccountId::from([0x0; 32])), 50);
    assert_eq!(contract.allowance(AccountId::from([0x1; 32]), AccountId::from([0x1; 32])), 150);
   }
   ```

以下命令验证程序是否编译并通过所有测试：

   ```bash
   cargo +nightly test
   ```

该命令应显示类似于以下内容的输出，以指示测试成功完成：

   ```text
   running 5 tests
   test erc20::tests::new_works ... ok
   test erc20::tests::balance_works ... ok
   test erc20::tests::transfer_works ... ok
   test erc20::tests::transfer_from_works ... ok
   test erc20::tests::allowances_works ... ok

   test result: ok. 5 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
   ```

•  通过运行以下命令验证您是否可以为合约构建 WebAssembly

   ```bash
   cargo +nightly contract build
   ```

为合约构建 WebAssembly 后，您可以使用合约 UI 上传并实例化它，如 [上传并实例化合约](#upload-and-instantiate-the-contract) 中所述。

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

![微信公众号565 ee](https://user-images.githubusercontent.com/28084126/179358584-3409af45-44ad-4f01-88f3-101dbb556ea6.png)
