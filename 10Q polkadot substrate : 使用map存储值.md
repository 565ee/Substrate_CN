• [介绍](#index1)  
• [初始化映射](#index2)  
• [识别合约调用者](#index3)  
• [使用合约调用者](#index4)  
• [向智能合约添加映射](#index5)  
• [插入、更新或删除值](#index6)  
• [Substrate Tutorials , Substrate 教程](#index98)  
• [Contact 联系方式](#index99) 

# <span id='index1'>• 介绍</span>  
在开发智能合约中，您开发了一个用于存储和检索单个数值的智能合约。本教程说明了如何扩展智能合约的功能以管理每个用户的一个号码。要添加此功能，您将使用该Mapping类型。

以下代码说明了将用户映射到数字：
```
#[ink(storage)]
pub struct MyContract {
	// Store a mapping from AccountIds to a u32
	my_number_map: ink_storage::Mapping<AccountId, u32>,
}
```

使用Mapping数据类型，您可以为每个键存储一个唯一的存储值实例。对于本教程，每个都AccountId代表一个键，它映射到一个且只有一个存储的 numeric my_value。每个用户只能存储、递增和检索与他或她自己的相关联的值AccountId。

# <span id='index2'>• 初始化映射</span>  
第一步是初始化一个AccountId和一个存储值之间的映射。在合约中使用映射之前，您必须始终初始化映射，以避免映射错误和不一致。要初始化映射，您需要执行以下操作：

SpreadAllocate在存储结构上添加特征。
• 指定映射键和映射到它的值。
• 调用该ink_lang::utils::initalize_contract函数来初始化合约的映射。
以下示例说明了如何初始化映射和检索值：
```
#![cfg_attr(not(feature = "std"), no_std)]

use ink_lang as ink;

#[ink::contract]
mod mycontract {
    use ink_storage::traits::SpreadAllocate;

    #[ink(storage)]
    #[derive(SpreadAllocate)]
    pub struct MyContract {
        // Store a mapping from AccountIds to a u32
        map: ink_storage::Mapping<AccountId, u32>,
    }

    impl MyContract {
        #[ink(constructor)]
        pub fn new(count: u32) -> Self {
            // This call is required to correctly initialize the
            // Mapping of the contract.
            ink_lang::utils::initialize_contract(|contract: &mut Self| {
                let caller = Self::env().caller();
                contract.map.insert(&caller, &count);
            })
        }

        #[ink(constructor)]
        pub fn default() -> Self {
            ink_lang::utils::initialize_contract(|_| {})
        }

        // Get the number associated with the caller's AccountId, if it exists
        #[ink(message)]
        pub fn get(&self) -> u32 {
            let caller = Self::env().caller();
            self.map.get(&caller).unwrap_or_default()
        }
    }
}
```

# <span id='index3'>• 识别合约调用者</span>  
在前面的示例中，您可能已经注意到self.env().caller()函数调用。此函数在整个合约逻辑中都可用，并且始终返回合约调用者。重要的是要注意合约调用者与原始调用者不同。如果用户访问一个合约然后调用后续合约，self.env().caller()则第二个合约中的地址是第一个合约的地址，而不是原始用户的地址。

# <span id='index4'>• 使用合约调用者</span>  
在许多情况下，让合约调用者可用是有用的。例如，您可以使用self.env().caller()创建一个仅允许用户访问自己的值的访问控制层。您还可以self.env().caller()在合约部署期间使用来保存合约所有者。例如：
```
#![cfg_attr(not(feature = "std"), no_std)]

use ink_lang as ink;

#[ink::contract]
mod mycontract {

	#[ink(storage)]
	pub struct MyContract {
		// Store a contract owner
		owner: AccountId,
	}

	impl MyContract {
		#[ink(constructor)]
		pub fn new() -> Self {
			Self {
				owner: Self::env().caller();
			}
		}
		/* --snip-- */
	}
}
```

# <span id='index5'>• 向智能合约添加映射</span>  
您现在已准备好向incrementer合约中引入存储映射。
向incrementer合约中添加存储映射：

• 如果需要，在您的计算机上打开终端外壳。
• 验证您是否位于incrementer项目文件夹中。
• lib.rs在文本编辑器中打开文件。
• 导入SpreadAllocate特征并为您的合同派生它。
```
#[ink::contract
mod incrementer {
   use ink_storage::traits::SpreadAllocate;

   #[ink(storage)]
   #[derive(SpreadAllocate)]
```

• 将映射键 from 添加AccountId到存储为 的i32数据类型my_value。
```
pub struct Incrementer {
   value: i32,
   my_value: ink_storage::Mapping<AccountId, i32>,
}
```

• 使用该函数为合约中的函数initialize_contract设置一个初始值。valuemy_valuenew
```
#[ink(constructor)]
pub fn new(init_value: i32) -> Self {
   ink_lang::utils::initialize_contract(|contract: &mut Self| {
       contract.value = init_value;
       let caller = Self::env().caller();
       contract.my_value.insert(&caller, &0);
   })
}
```

• 使用该函数为合约中的initialize_contract函数设置一个初始value值。default
```
#[ink(constructor)]
pub fn default() -> Self {
   ink_lang::utils::initialize_contract(|contract: &mut Self| {
       contract.value = Default::default();
   })
}
```

• 保存更改并关闭文件。
• 通过运行以下命令，使用test子命令和工具链来测试您的工作：nightly
```
cargo +nightly test
```

# <span id='index6'>• 插入、更新或删除值</span>  
Incrementer 合约的最后一步是允许用户更新自己的值。您可以使用对 Mapping API 的调用在智能合约中提供此功能。

映射提供对存储项目的ink_storage直接访问。例如，您可以通过Mapping::insert()使用现有键调用来替换为存储项保存的先前值。您还可以通过首先使用从存储中读取值来更新值Mapping::get()，然后使用 更新值Mapping::insert()。如果给定键没有现有值，则Mapping::get()返回 None。

由于 Mapping API 提供对存储的直接访问，因此您可以使用该 Mapping::remove()方法从存储中删除给定键的值。

向合约添加插入和删除功能：
• • 验证您是否位于incrementer项目文件夹中。
• lib.rs在文本编辑器中打开文件。
• 添加一个inc_mine()函数，允许合约调用者获取my_value存储项并将增量value插入到映射中。
```
#[ink(message)]
pub fn inc_mine(&mut self, by: i32) {
   let caller = self.env().caller();
   let my_value = self.get_mine();
   self.my_value.insert(caller, &(my_value + by));
}
```

• 添加一个remove_mine()函数，允许合约调用者my_value从存储中清除存储项。
```
#[ink(message)]
pub fn remove_mine(&self) {
   self.my_value.remove(&self.env().caller())
}
```

• 添加新测试以验证inc_mine()功能是否按预期工作。
```
#[ink::test]
fn inc_mine_works() {
   let mut contract = Incrementer::new(11);
   assert_eq!(contract.get_mine(), 0);
   contract.inc_mine(5);
   assert_eq!(contract.get_mine(), 5);
   contract.inc_mine(5);
   assert_eq!(contract.get_mine(), 10);
}
```

• 添加新测试以验证remove_mine()功能是否按预期工作。
```
#[ink::test]
fn remove_mine_works() {
   let mut contract = Incrementer::new(11);
   assert_eq!(contract.get_mine(), 0);
   contract.inc_mine(5);
   assert_eq!(contract.get_mine(), 5);
   contract.remove_mine();
   assert_eq!(contract.get_mine(), 0);
}
```

• 通过运行以下命令，使用test子命令和工具链来测试您的工作：nightly
```
cargo +nightly test
```

该命令应显示类似于以下内容的输出，以指示测试成功完成：
```
running 5 tests
test incrementer::tests::default_works ... ok
test incrementer::tests::it_works ... ok
test incrementer::tests::remove_mine_works ... ok
test incrementer::tests::inc_mine_works ... ok
test incrementer::tests::my_value_works ... ok

test result: ok. 5 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

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
![微信公众号565 ee](https://user-images.githubusercontent.com/28084126/178929112-e1ed07f7-c7c8-4db3-88ba-2e99d952562e.png)
