• [介绍](#index1)  
• [创建一个新的智能合约项目](#index2)  
• [存储简单值](#index3)  
• [构造函数](#index4)  
• [添加一个函数来获取一个存储值](#index5)  
• [添加修改存储值的功能](#index6)  
• [为合约构建 WebAssembly](#index7)  
• [部署和测试智能合约](#index8)  
• [Substrate Tutorials , Substrate 教程](#index98)  
• [Contact 联系方式](#index99) 

# <span id='index1'>• 介绍</span>  
在准备你的第一个合约中，你学习了使用默认的第一个项目在基于 Substrate 的区块链上构建和部署智能合约的基本步骤。在本教程中，您将开发一个新的智能合约，每次执行函数调用时都会增加一个计数器值。

通过完成本教程，您将实现以下目标：

了解如何使用智能合约模板。
使用智能合约存储简单值。
使用智能合约增加和检索存储的值。
向智能合约添加公共和私有功能。

# <span id='index2'>• 创建一个新的智能合约项目</span>  
在 Substrate 上运行的智能合约从项目开始。cargo contract您使用命令创建项目。

在本教程中，您将为incrementer智能合约创建一个新项目。创建新项目会将新项目目录和默认启动文件（也称为模板文件）添加到项目目录中。您将修改这些起始模板文件以构建incrementer项目的智能合约逻辑。

为您的智能合约创建一个新项目：

• 如果您还没有打开终端外壳，请在本地计算机上打开终端外壳。
• incrementer通过运行以下命令创建一个名为的新项目：
```
cargo contract new incrementer
```

• 通过运行以下命令切换到新项目目录：
```
cd incrementer/
```

• lib.rs在文本编辑器中打开文件。

默认情况下，模板lib.rs文件包含flipper智能合约的源代码，flipper合约名称的实例重命名为incrementer.

• 用新的增量器源代码替换默认模板源代码。
• 保存对lib.rs文件的更改，然后关闭文件。
• 在文本编辑器中打开Cargo.toml文件并查看合同的依赖关系。
• 保存对Cargo.toml文件的更改，然后关闭文件。
• 通过运行以下命令来验证程序是否编译并通过了简单的测试：
```
cargo +nightly test
```

• 通过运行以下命令，验证您是否可以为合约构建 WebAssembly：
```
cargo +nightly contract build
```
如果程序编译成功，您就可以开始编程了。

# <span id='index3'>• 存储简单值</span>  
现在您已经有了一些incrementer智能合约的入门源代码，您可以引入一些新功能。例如，这个智能合约需要存储简单的值。以下代码说明了如何使用#[ink(storage)]属性宏存储此合约的简单值：
```
#[ink(storage)]
pub struct MyContract {
	// Store a bool
	my_bool: bool,
	// Store a number
	my_number: u32,
}
```

# <span id='index4'>• 构造函数</span>  
每一种墨水！智能合约必须至少有一个在创建合约时运行的构造函数。但是，如果需要，智能合约可以有多个构造函数。以下代码说明了使用多个构造函数：
```
use ink_lang as ink;

#[ink::contract]
mod mycontract {

	#[ink(storage)]
	pub struct MyContract {
		number: u32,
	}

	impl MyContract {
		/// Constructor that initializes the `u32` value to the given `init_value`.
		#[ink(constructor)]
		pub fn new(init_value: u32) -> Self {
			Self {
				number: init_value,
			}
		}

		/// Constructor that initializes the `u32` value to the `u32` default.
		///
		/// Constructors can delegate to other constructors.
		#[ink(constructor)]
		pub fn default() -> Self {
			Self {
				number: Default::default(),
			}
		}
	/* --snip-- */
	}
}
```

# <span id='index5'>• 添加一个函数来获取一个存储值</span>  
现在您已经创建并初始化了一个存储值，您可以使用公共和私有函数与它进行交互。在本教程中，您将添加一个公共函数来获取存储值。请注意，所有公共函数都必须使用#[ink(message)]属性宏。

将公共功能添加到智能合约：

• lib.rs在文本编辑器中打开文件。
• • 更新get公共函数以返回value具有该i32数据类型的存储项的数据。
```
#[ink(message)]
pub fn get(&self) -> i32 {
   self.value
   }
}
```

• • • 用代码替换Test Your Contract私有default_works函数中的注释来测试get函数。
```
fn default_works() {
   let contract = Incrementer::default();
   assert_eq!(contract.get(), 0);
}
```

• 保存更改并关闭文件。
• 通过运行以下命令，使用test子命令和工具链来测试您的工作：nightly
```
cargo +nightly test
```

# <span id='index6'>• 添加修改存储值的功能</span>  
此时，智能合约不允许用户修改存储。要使用户能够修改存储项，您必须显式标记value为可变变量。

要添加用于递增存储值的函数：

• • lib.rs在文本编辑器中打开文件。
• 添加一个新的inc公共函数以value使用by数据类型为i32.
```
#[ink(message)]
pub fn inc(&mut self, by: i32) {
   self.value += by;
   }
}
```

• 在源代码中添加一个新的测试来验证这个功能。
```
#[ink::test]
   fn it_works() {
       let mut contract = Incrementer::new(42);
       assert_eq!(contract.get(), 42);
       contract.inc(5);
       assert_eq!(contract.get(), 47);
       contract.inc(-50);
       assert_eq!(contract.get(), -3);
}
```

• 保存更改并关闭文件。
• 通过运行以下命令，使用test子命令和工具链来测试您的工作：nightly
```
cargo +nightly test
```

# <span id='index7'>• 为合约构建 WebAssembly</span>  
测试incrementer合约后，您就可以将此项目编译为 WebAssembly。将智能合约编译为 WebAssembly 后，您可以使用Contracts UI在本地合约节点上部署和测试智能合约。

为这个智能合约构建 WebAssembly：

• 如果需要，在您的计算机上打开终端外壳。
• 验证您是否位于incrementer项目文件夹中。
• incrementer通过运行以下命令编译智能合约：
```
cargo +nightly contract build
```

# <span id='index8'>• 部署和测试智能合约</span>  
如果您在substrate-contracts-node本地安装了节点，您可以为您的智能合约启动一个本地区块链节点，然后使用Contracts UI来部署和测试智能合约。

在本地节点上部署：

• 如果需要，在您的计算机上打开终端外壳。
• 通过运行以下命令以本地开发模式启动contracts节点：
```
substrate-contracts-node --dev
```

• 打开Contracts UI并验证它是否已连接到本地节点。
• 单击添加新合同。
• 点击上传新合同代码。
• 选择incrementer.contract文件，然后单击Next。
• 单击上传并实例化。
• 使用合约 UI 探索智能合约并与之交互。

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
![微信公众号565 ee](https://user-images.githubusercontent.com/28084126/178544666-2d97f040-811c-4fdf-9e7f-992e89e0b08a.png)
