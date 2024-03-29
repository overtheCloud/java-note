# Solidity

智能合约开发语言。类似Js。

先从一段示例代码开始。示例代码来自 `https://cryptozombies.io/en`

```javascript
// 指定版本，因合约不同版本间差异可能过大导致无法正确编译，此处指定版本大于等于 0.5.0 且小于 0.6.0 
pragma solidity >=0.5.0 <0.6.0;
// contract 关键字定义了一个合约，语法： contract [name]
contract ZombieFactory {

    // 声明一个事件，可以被js监听
    event NewZombie(uint id, string name, uint dna);
	// 声明变量, uint 无符号数字，默认指 uint256，还有 uint8 | uint16 | uint 32...
    // stirng 字符串
    // address 地址
    // mapping k-v组，同 map
    uint dnaDigits = 16;
    // 算术运算符，支持 +, -, *, /, %, **(平方)
    uint dnaModulus = 10 ** dnaDigits;
	// 声明一个结构体，包含多个属性
    struct Zombie {
        string name;
        uint dna;
    }
    // [] 声明数组，push()(返回索引) 和 get() 从数组中添加和读取数据,索引从 0 开始
    // 范围标识 public(可以在外部调用) | private(仅当前合约内可调用)
    Zombie[] public zombies;
	// 定义函数, 
    // memory 表示变量存储在内存中,数组、结构体、mapping、字符串都需要用 memory 修饰
    function _createZombie(string memory _name, uint _dna) private {
        uint id = zombies.push(Zombie(_name, _dna)) - 1;
        // 推送事件给监听器
        emit NewZombie(id, _name, _dna);
    }

    // view 表示此函数只查看合约数据，不会修改
    // pure 表示此函数不涉及合约数据
    // returns 指定返回值类型
    function _generateRandomDna(string memory _str) private view returns (uint) {
        // 方法调用
        // uint() 强制转换类型为 uint
        // abi.encoedePacked() 自带函数，编码
        uint rand = uint(keccak256(abi.encodePacked(_str)));
        return rand % dnaModulus;
    }

    // 对外提供的方法，创建一个 Zombie 并推送消息
    function createRandomZombie(string memory _name) public {
        uint randDna = _generateRandomDna(_name);
        _createZombie(_name, randDna);
    }
}
```

对应 js

```javascript
// Here's how we would access our contract:
var abi = /* abi generated by the compiler */
var ZombieFactoryContract = web3.eth.contract(abi)
var contractAddress = /* our contract address on Ethereum after deploying */
var ZombieFactory = ZombieFactoryContract.at(contractAddress)
// `ZombieFactory` has access to our contract's public functions and events

// some sort of event listener to take the text input:
$("#ourButton").click(function(e) {
  var name = $("#nameInput").val()
  // Call our contract's `createRandomZombie` function:
  ZombieFactory.createRandomZombie(name)
})

// Listen for the `NewZombie` event, and update the UI
var event = ZombieFactory.NewZombie(function(error, result) {
  if (error) return
  let zombie = generateZombie(result.zombieId, result.name, result.dna)
  console.log(zombie);
})

// take the Zombie dna, and update our image
function generateZombie(id, name, dna) {
  let dnaStr = String(dna)
  // pad DNA with leading zeroes if it's less than 16 characters
  while (dnaStr.length < 16)
    dnaStr = "0" + dnaStr

  let zombieDetails = {
    // first 2 digits make up the head. We have 7 possible heads, so % 7
    // to get a number 0 - 6, then add 1 to make it 1 - 7. Then we have 7
    // image files named "head1.png" through "head7.png" we load based on
    // this number:
    headChoice: dnaStr.substring(0, 2) % 7 + 1,
    // 2nd 2 digits make up the eyes, 11 variations:
    eyeChoice: dnaStr.substring(2, 4) % 11 + 1,
    // 6 variations of shirts:
    shirtChoice: dnaStr.substring(4, 6) % 6 + 1,
    // last 6 digits control color. Updated using CSS filter: hue-rotate
    // which has 360 degrees:
    skinColorChoice: parseInt(dnaStr.substring(6, 8) / 100 * 360),
    eyeColorChoice: parseInt(dnaStr.substring(8, 10) / 100 * 360),
    clothesColorChoice: parseInt(dnaStr.substring(10, 12) / 100 * 360),
    zombieName: name,
    zombieDescription: "A Level 1 CryptoZombie",
  }
  return zombieDetails
}
```

#### 基础语法

`mapping` key-value ，键值对结构的数据类型

```js
mapping (uint => string) userIdToName;
```

`Msg.sender` 方法调用者的地址

`Msg.value` 调方法的交易的金额

```js
mapping (address => uint) favoriteNumber;

function setMyNumber(uint _myNumber) public {
  favoriteNumber[msg.sender] = _myNumber;
}
```

`require(conditon)` 校验代币，表达式结果是 true 才继续执行

```js
// require(condition) condition 值范围[true|false] ，true则继续执行，false则停止执行
require(1==1)
uint a;
require(1!=1)
uint b;// 此行不会执行
```

`is` 继承，表示一个合约继承另一个合约

```js
contract Doge {
  function catchphrase() public returns (string memory) {
    return "So Wow CryptoDoge";
  }
}

contract BabyDoge is Doge {
  function anotherCatchphrase() public returns (string memory) {
    return "Such Moon BabyDoge";
  }
}
```

`import` 引入外部合约

```js
import "./someothercontract.sol";

contract newContract {}
```

`storage` 指定变量持久化到合约

`memory` 指定变量仅存在于内存，不会持久化到合约

```js
Sandwich[] sandwiches;
function eatSandwich(uint _index) public {
    // 此处编译器会告警，需要显示指定存储类型[storage|memory]，仅结构体和数组会有这个告警
    // Sandwich mySandwich = sandwiches[_index];
    
    // 对 storage 修饰的变量做的修改操作会持久化到合约
    Sandwich storage mySandwich = sandwiches[_index];
    mySandwich.status = "Eaten!";

    // 对 memory 修饰的变量做的修改操作不会持久化到合约，仅存在于内存
    Sandwich memory anotherSandwich = sandwiches[_index + 1];
    anotherSandwich.status = "Eaten!";
    // 可以通过直接赋值的方式修改合约数据
    sandwiches[_index + 1] = anotherSandwich;
}
```

`internal` 类似 `private`,不同之处在于引用范围扩大到子类，即 `A is B` 中的 A可以调用 B 中 `internal ` 修饰的变量和方法

`external` 类似 `public` 不同之处在于引用范围缩小到仅外部合约允许调用，即 `A is B` 中的 A 可以调用 B 中 `external` 修饰的变量和方法，而 B 中其他方法不允许调用

```js
contract Sandwich {
  uint private sandwichesEaten = 0;

  function eat() internal {
    sandwichesEaten++;
    // run() 编译器告警
  }
    function run() external {
    }
}

contract BLT is Sandwich {
  uint private baconSandwichesEaten = 0;

  function eatWithBacon() public returns (string memory) {
    baconSandwichesEaten++;
    eat();
    run();
  }
}
```

`调用其他合约`

```js
// 已存在的合约
contract LuckyNumber {
  mapping(address => uint) numbers;

  function setNum(uint _num) public {
    numbers[msg.sender] = _num;
  }

  function getNum(address _myAddress) public view returns (uint) {
    return numbers[_myAddress];
  }
}

// 定义一个合约接口
contract NumberInterface {
    // 合约接口的方法无方法体
  function getNum(address _myAddress) public view returns (uint);
}

contract MyContract {
  // 合约地址
  address NumberInterfaceAddress = 0xab38... 
  // 定义合约引用变量，numberContract 指向了链上的合约
  NumberInterface numberContract = NumberInterface(NumberInterfaceAddress);

  function someFunction() public {
    // 通过 numberContract.getNum() 实现对 LuckyNumber.getNum 的调用
    uint num = numberContract.getNum(msg.sender);
  }
}
```

`多返回值` 

```js
function multipleReturns() internal returns(uint a, uint b, uint c) {
  return (1, 2, 3);
}

function processMultipleReturns() external {
  uint a;
  uint b;
  uint c;
  // 接收全部变量
  (a, b, c) = multipleReturns();
}

function getLastReturnValue() external {
  uint c;
  // 接收部分参数
  (,,c) = multipleReturns();
}
```

`if(condition){}`  condition 值 [true|false] ， true的情况执行代码块

```js
if(1==1) {
    uint a = 1;
} 
```

`constructor`  构造器，仅在合约初始化时调用一次

```js
 constructor() internal {
    _owner = msg.sender;
  }
```

`modifier` 定义一类用于其他方法的关键字，通常用于执行其他方法前的检查

```js
// 如果非当前拥有者调用，抛出异常,onlyOwner 可以用于修饰其他方法，以阻止非当前拥有者调用
modifier onlyOwner() {
    // 判断调用者是否当前 owner
    require(isOwner());
    // 返回被 renounceOwnership 方法继续执行
    _;
}
// renounceOwnership 被调用时，首先执行 onlyOwner 的代码
 function renounceOwnership() public onlyOwner {
    emit OwnershipTransferred(_owner, address(0));
    _owner = address(0);
  }
```

`modifyer` 也可用于参数检查

```js
// 检查参数
modifier olderThan(uint _age, uint _userId) {
  require(age[_userId] >= _age);
  _;
}

// 检查参数是否大于 16
function driveCar(uint _userId) public olderThan(16, _userId) {

}
```



> 执行合约需要消耗 gas， 在合约中定义 uint,uint8,uint16,uint32... 通常没有区别，合约都会以 uint256 存储。但是在结构体中则不同，结构体的数个 uint 会被打包到一个 uint256 结构中存储。通常在结构体中使用合适的 uint 类型可以节约 gas。

```js
struct NormalStruct {
  uint a;
  uint b;
  uint c;
}

struct MiniMe {
  uint32 a;
  uint32 b;
  uint c;
}

// `mini` will cost less gas than `normal` because of struct packing
NormalStruct normal = NormalStruct(10, 20, 30);
MiniMe mini = MiniMe(10, 20, 30); 
```

`now` 返回当前最有一个块的时间戳，从 `1970-07-01` 开始计数。

合约默认使用32位存储时间，也就是默认时间最大只能表示到 2038 年，可以指定使用 64位 表示更多时间。

`seconds` `minuters` `hours` `days` `weeks` `yeas` 这些时间单位都可以转换成以秒位单位的数字，如 1 minuters = 60, 1hour = 3600。

`定义一个数组`

```js
uint[] memory values = new uint[](3);

values[0] = 1;
values[1] = 2;
values[2] = 3;
```

`调用合约时发送eth` 需要调用方在提交交易时发送 eth

```js
contract OnlineStore {
    // payable 指定该方法允许接收 eth
  function buySomething() external payable {
    require(msg.value == 0.001 ether);
    transferThing(msg.sender);
  }
}
```

`调用合约提取eth` 将 eth 转移到指定地址

```js
contract GetPaid is Ownable {
  function withdraw() external onlyOwner {
      // 发送合约地址上的 eth 到 owner 对应的地址
    address payable _owner = address(uint160(owner()));
    _owner.transfer(address(this).balance);
  }
}
```

`产生不安全的随机数` 

```js
uint randNonce = 0;
// _modulus 随机数的位数
function randMod(uint _modulus) internal returns(uint) {
    randNonce++;
    return uint(keccak256(abi.encodePacked(now, msg.sender, randNonce))) % _modulus;
}
```

### 合约标准

实现相同标准的合约可以交互。

#### ERC20 可分隔的代币，可交易，如 USDC|USDTERC20

#### ERC721  每个 token 不可分割，且有唯一标识，可交易

```js
pragma solidity >=0.5.0 <0.6.0;

contract ERC721 {
  event Transfer(address indexed _from, address indexed _to, uint256 indexed _tokenId);
  event Approval(address indexed _owner, address indexed _approved, uint256 indexed _tokenId);

  function balanceOf(address _owner) external view returns (uint256);
  function ownerOf(uint256 _tokenId) external view returns (address);
  function transferFrom(address _from, address _to, uint256 _tokenId) external payable;
  function approve(address _approved, uint256 _tokenId) external payable;
}
```

### 内置library

`SafeMath` 提供了安全的计算

`assert` 类似 `require` ，区别是 `require` 执行失败时会返回用户剩余的费用，而 `assert` 不会，严重代码错误时，如内存溢出可用 `assert` ，其他地方建议使用 `require`

```js
pragma solidity >=0.5.0 <0.6.0;

/**
 * @title SafeMath
 * @dev Math operations with safety checks that throw on error
 */
library SafeMath {

  /**
  * @dev Multiplies two numbers, throws on overflow.
  */
  function mul(uint256 a, uint256 b) internal pure returns (uint256) {
    if (a == 0) {
      return 0;
    }
    uint256 c = a * b;
    assert(c / a == b);
    return c;
  }

  /**
  * @dev Integer division of two numbers, truncating the quotient.
  */
  function div(uint256 a, uint256 b) internal pure returns (uint256) {
    // assert(b > 0); // Solidity automatically throws when dividing by 0
    uint256 c = a / b;
    // assert(a == b * c + a % b); // There is no case in which this doesn't hold
    return c;
  }

  /**
  * @dev Subtracts two numbers, throws on overflow (i.e. if subtrahend is greater than minuend).
  */
  function sub(uint256 a, uint256 b) internal pure returns (uint256) {
    assert(b <= a);
    return a - b;
  }

  /**
  * @dev Adds two numbers, throws on overflow.
  */
  function add(uint256 a, uint256 b) internal pure returns (uint256) {
    uint256 c = a + b;
    assert(c >= a);
    return c;
  }
}
```

`using` 将 library 的全部方法赋予一个数据类型

```js
using SafeMath for uint256;

uint256 a = 5;
uint256 b = a.add(3); // 5 + 3 = 8
uint256 c = a.mul(2); // 5 * 2 = 10
```



