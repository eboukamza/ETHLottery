# ETHLottery

The Ethereum Block Hash Lottery

ETHLottery is a 100% fair lottery based on Ethereum blockchain technology, where the result is chosen based on a certain Ethereum Block Hash created in the future, therefore impossible to precalculate and temper with.

ETHLottery was designed to spare as much ether gas as possible keeping interactions to a minimum and a tidy contract code with simplistic logic.

## How it works

Each round starts with an Ethereum smart contract with empty balance, unless previous round has accumulated, during each round participants can make bets by sending the minimal fee and trying to guess which is the exact last byte of a certain Ethereum Block Hash generated only and **really only** after the lottery is closed.

The Ethereum Block Hash is stored in hexadecimal format therefore the last byte are the exact last two characters of the hash. (Eg.: 0x00, 0xca, 0xf1, 0x3d). This gives 1 chance in 256 with possible byte values ranging from 0x00 to 0xff.

Example Ethereum Block Hash:
**0x5a801d8cf81a8ad942d5fa1b47f25b0b1507b7531b93f13ef9764e287bc51800**

From the example above the winning result byte would be **0x00** as the hash ends in 00.

The Ethereum Block Hash used as lottery result belongs to the 10th block, after the block containing the only transaction made from the ETHLottery smart contract to the lottery owner's Ethereum wallet with the payment of the owner fee, this action happens only once and only after the lottery is closed.

The lottery closes exactly when the contract balance which holds the prize, reaches or surpasses the value of the jackpot.

The lottery fee, the owner fee and the jackpot are values set when the contract is created and are 100% public and immutable in the blockchain.

In case there are **no** winning bets with the exact last byte guess, the lottery jackpot prize accumulates to next lottery round.

The lottery prize jackpot is divided equally among all winning participants.

All lotteries have owner fee set during lottery creation which is a percentage of the jackpot.

## How to play with geth console on CLI

### Preparations

Before sending a bet, make the preparations by creating the lottery instance, pointing to the ABI and already deployed contract address.

```shell
// set the ETHLottery contract ABI
var abi = [{"constant":true,"inputs":[],"name":"name","outputs":[{"name":"","type":"bytes32"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"owner_fee","outputs":[{"name":"","type":"uint256"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"manager_address","outputs":[{"name":"","type":"address"}],"payable":false,"type":"function"},{"constant":false,"inputs":[],"name":"register","outputs":[],"payable":false,"type":"function"},{"constant":false,"inputs":[],"name":"destruct","outputs":[],"payable":false,"type":"function"},{"constant":false,"inputs":[],"name":"withdraw","outputs":[],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"result_block","outputs":[{"name":"","type":"uint256"}],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"_result_hash","type":"bytes32"}],"name":"manual_lottery","outputs":[],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"result","outputs":[{"name":"","type":"bytes1"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"jackpot","outputs":[{"name":"","type":"uint256"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"owner","outputs":[{"name":"","type":"address"}],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"_lottery","type":"address"}],"name":"accumulate","outputs":[],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"create_block","outputs":[{"name":"","type":"uint256"}],"payable":false,"type":"function"},{"constant":false,"inputs":[],"name":"lottery","outputs":[],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"_byte","type":"bytes1"}],"name":"play","outputs":[],"payable":true,"type":"function"},{"constant":true,"inputs":[],"name":"result_hash","outputs":[{"name":"","type":"bytes32"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"fee","outputs":[{"name":"","type":"uint256"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"winners_count","outputs":[{"name":"","type":"uint256"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"open","outputs":[{"name":"","type":"bool"}],"payable":false,"type":"function"},{"inputs":[{"name":"_manager","type":"address"},{"name":"_fee","type":"uint256"},{"name":"_jackpot","type":"uint256"},{"name":"_owner_fee","type":"uint256"}],"payable":false,"type":"constructor"},{"anonymous":false,"inputs":[{"indexed":false,"name":"_balance","type":"uint256"}],"name":"Balance","type":"event"},{"anonymous":false,"inputs":[{"indexed":false,"name":"_result","type":"bytes1"}],"name":"Result","type":"event"},{"anonymous":false,"inputs":[{"indexed":false,"name":"_open","type":"bool"}],"name":"Open","type":"event"},{"anonymous":false,"inputs":[{"indexed":true,"name":"_sender","type":"address"},{"indexed":false,"name":"_byte","type":"bytes1"},{"indexed":false,"name":"_time","type":"uint256"}],"name":"Play","type":"event"},{"anonymous":false,"inputs":[{"indexed":true,"name":"_sender","type":"address"},{"indexed":false,"name":"_amount","type":"uint256"},{"indexed":false,"name":"_time","type":"uint256"}],"name":"Withdraw","type":"event"},{"anonymous":false,"inputs":[],"name":"Destroy","type":"event"},{"anonymous":false,"inputs":[{"indexed":false,"name":"_amount","type":"uint256"}],"name":"Accumulate","type":"event"}];

// set the ETHLottery contract address
var address = "0xa55e4642e1630d6b49d6727b700dc15cac19fe72";

// set the participant address (to use gas and fee from)
var participant = "0xa2a069c92c8f41e8433adf1609e335af035eb17b";

// instantiate the ETHLottery contract
var ETHLottery = eth.contract(abi).at(address);
```

### Public lottery methods (read-only = FREE)

Its always possible to see the current lottery status by calling the read-only methods.

```shell
// lottery name
ETHLottery.name();

// lottery open status, (true or false)
ETHLottery.open();

// jackpot amount in WEI 
ETHLottery.jackpot();

// bet minimum fee amount in WEI
ETHLottery.fee();

// owner fee amount in percentage of balance
ETHLottery.owner_fee();

// block number containing winning result hash, only when open=false
ETHLottery.result_block();

// block hash containing winning result byte, only when open=false
ETHLottery.result_hash();

// lottery result byte, the result, only when open=false
ETHLottery.result();

// number of winners, only when open=false and result is available
ETHLottery.winners_count();
```

### Sending bets

To send a bet, create a transaction to the contract address by calling the function play() passing the bet guess and the minimal fee payment as value.

```shell
// set the bet guess to a byte represented in hexadecimal (from 0x00 to 0xff)
var guess = '0x3d';

// set the amount to the minimum fee from contract value
var amount = ETHLottery.fee().toString(10);

// unlock the participant account / wallet
web3.personal.unlockAccount(participant, "YOUR_VERY_SECRET_PASSWORD");

// call the play function passing the bet guess and the value
ETHLottery.play(guess, {from: participant, value: amount});

// make sure you lock the account again
web3.personal.lockAccount(participant);

// send more bets
ETHLottery.play('0xbc', {from: participant, value: amount});
ETHLottery.play('0xca', {from: participant, value: amount});
ETHLottery.play('0x78', {from: participant, value: amount});
ETHLottery.play('0xf1', {from: participant, value: amount});
```

### Withdraw prize

In case the bet sent was a winning bet, right after the lottery result is known it is possible to withdraw the prize by calling the withdraw() function for the winning participant.

```shell
// unlock the participant account / wallet
web3.personal.unlockAccount(participant, "YOUR_VERY_SECRET_PASSWORD");

// withdraw the prize, after ETHLottery is closed, only for winners
ETHLottery.withdraw({from: participant});

// make sure you lock the account again
web3.personal.lockAccount(participant);
```

## How to play with js GUI

It is also possible to play using the javascript GUI in the file index.html 

If you decide to test the index.html make sure to update ABI and run your own Ethereum node to attach.

### Install dependency
```shell
npm install web3 bignumber
```

## How to run your own ETHLottery

### Create a new lottery by deploying contract

First step is to create a new lottery deploying the contract to Ethereum blockchain network.

At this step, its possible to set the minimum bet fee, the jackpot amount and the owner fee.

```shell
// set the manager contract address
var manager_address = '0x4b8ed55eac499816532a5388373bfff0398c8203';

// set betting minimum fee amount to 0.001 ETH
var fee = 1000000000000000;

// set jackpot amount to 1 ETH
var jackpot = 1000000000000000000;

// set owner fee amount in percentage of total
var owner_fee = 2;

// set the owner address
var owner = web3.eth.accounts[0];

// set the ABI for the current contract version
// created with: solc --abi ETHLottery.sol
var abi = [{"constant":true,"inputs":[],"name":"name","outputs":[{"name":"","type":"bytes32"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"owner_fee","outputs":[{"name":"","type":"uint256"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"manager_address","outputs":[{"name":"","type":"address"}],"payable":false,"type":"function"},{"constant":false,"inputs":[],"name":"register","outputs":[],"payable":false,"type":"function"},{"constant":false,"inputs":[],"name":"destruct","outputs":[],"payable":false,"type":"function"},{"constant":false,"inputs":[],"name":"withdraw","outputs":[],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"result_block","outputs":[{"name":"","type":"uint256"}],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"_result_hash","type":"bytes32"}],"name":"manual_lottery","outputs":[],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"result","outputs":[{"name":"","type":"bytes1"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"jackpot","outputs":[{"name":"","type":"uint256"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"owner","outputs":[{"name":"","type":"address"}],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"_lottery","type":"address"}],"name":"accumulate","outputs":[],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"create_block","outputs":[{"name":"","type":"uint256"}],"payable":false,"type":"function"},{"constant":false,"inputs":[],"name":"lottery","outputs":[],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"_byte","type":"bytes1"}],"name":"play","outputs":[],"payable":true,"type":"function"},{"constant":true,"inputs":[],"name":"result_hash","outputs":[{"name":"","type":"bytes32"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"fee","outputs":[{"name":"","type":"uint256"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"winners_count","outputs":[{"name":"","type":"uint256"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"open","outputs":[{"name":"","type":"bool"}],"payable":false,"type":"function"},{"inputs":[{"name":"_manager","type":"address"},{"name":"_fee","type":"uint256"},{"name":"_jackpot","type":"uint256"},{"name":"_owner_fee","type":"uint256"}],"payable":false,"type":"constructor"},{"anonymous":false,"inputs":[{"indexed":false,"name":"_balance","type":"uint256"}],"name":"Balance","type":"event"},{"anonymous":false,"inputs":[{"indexed":false,"name":"_result","type":"bytes1"}],"name":"Result","type":"event"},{"anonymous":false,"inputs":[{"indexed":false,"name":"_open","type":"bool"}],"name":"Open","type":"event"},{"anonymous":false,"inputs":[{"indexed":true,"name":"_sender","type":"address"},{"indexed":false,"name":"_byte","type":"bytes1"},{"indexed":false,"name":"_time","type":"uint256"}],"name":"Play","type":"event"},{"anonymous":false,"inputs":[{"indexed":true,"name":"_sender","type":"address"},{"indexed":false,"name":"_amount","type":"uint256"},{"indexed":false,"name":"_time","type":"uint256"}],"name":"Withdraw","type":"event"},{"anonymous":false,"inputs":[],"name":"Destroy","type":"event"},{"anonymous":false,"inputs":[{"indexed":false,"name":"_amount","type":"uint256"}],"name":"Accumulate","type":"event"}];

// set the compiled code for the current contract version
// created with: run solc --bin ETHLottery.sol
var code = "0x60606040527f4554484c6f7474657279202d204c61737420312042797465204c6f7474657279600090600019169055341561003957600080fd5b604051608080611552833981016040528080519060200190919080519060200190919080519060200190919080519060200190919050505b33600260006101000a81548173ffffffffffffffffffffffffffffffffffffffff021916908373ffffffffffffffffffffffffffffffffffffffff1602179055506001600260146101000a81548160ff0219169083151502179055504360068190555083600160006101000a81548173ffffffffffffffffffffffffffffffffffffffff021916908373ffffffffffffffffffffffffffffffffffffffff1602179055508260048190555081600381905550806005819055507fc15f25c4eb6ac5b9dc23661b850cbdb3ee36aebed198ad870ab929bae77142de600260149054906101000a900460ff16604051808215151515815260200191505060405180910390a15b505050505b6113c9806101896000396000f30060606040523615610105576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806306fdde031461010a57806314ce40591461013b5780631800ed8e146101645780631aa3a008146101b95780632b68b9c6146101ce5780633ccfd60b146101e3578063494d93cc146101f85780634f01d77e1461022157806365372147146102485780636b31ee01146102b55780638da5cb5b146102de5780638f045bb61461033357806394e47e611461036c578063ba13a57214610395578063be3e33d5146103aa578063ca92a136146103e4578063ddca3f4314610415578063fbe6a9b21461043e578063fcfff16f14610467575b600080fd5b341561011557600080fd5b61011d610494565b60405180826000191660001916815260200191505060405180910390f35b341561014657600080fd5b61014e61049a565b6040518082815260200191505060405180910390f35b341561016f57600080fd5b6101776104a0565b604051808273ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200191505060405180910390f35b34156101c457600080fd5b6101cc6104c6565b005b34156101d957600080fd5b6101e16105c5565b005b34156101ee57600080fd5b6101f66106a8565b005b341561020357600080fd5b61020b610877565b6040518082815260200191505060405180910390f35b341561022c57600080fd5b61024660048080356000191690602001909190505061087d565b005b341561025357600080fd5b61025b610b85565b60405180827effffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff19167effffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff1916815260200191505060405180910390f35b34156102c057600080fd5b6102c8610bb7565b6040518082815260200191505060405180910390f35b34156102e957600080fd5b6102f1610bbd565b604051808273ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200191505060405180910390f35b341561033e57600080fd5b61036a600480803573ffffffffffffffffffffffffffffffffffffffff16906020019091905050610be3565b005b341561037757600080fd5b61037f610cc7565b6040518082815260200191505060405180910390f35b34156103a057600080fd5b6103a8610ccd565b005b6103e260048080357effffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff1916906020019091905050611014565b005b34156103ef57600080fd5b6103f7611327565b60405180826000191660001916815260200191505060405180910390f35b341561042057600080fd5b61042861132d565b6040518082815260200191505060405180910390f35b341561044957600080fd5b610451611333565b6040518082815260200191505060405180910390f35b341561047257600080fd5b61047a611339565b604051808215151515815260200191505060405180910390f35b60005481565b60055481565b600160009054906101000a900473ffffffffffffffffffffffffffffffffffffffff1681565b6000600260009054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff1614151561052457600080fd5b600160009054906101000a900473ffffffffffffffffffffffffffffffffffffffff1690508073ffffffffffffffffffffffffffffffffffffffff16631aa3a0086040518163ffffffff167c0100000000000000000000000000000000000000000000000000000000028152600401600060405180830381600087803b15156105ac57600080fd5b6102c65a03f115156105bd57600080fd5b5050505b5b50565b600260149054906101000a900460ff161515156105e157600080fd5b600260009054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff1614151561063d57600080fd5b7ff58fef8e187ef8dfd7bef096c1ef3e4f3c54f98d95b8ad5659349b07e61204df60405160405180910390a1600260009054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16ff5b5b5b565b6000600260149054906101000a900460ff161515156106c657600080fd5b6000600c60003373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020019081526020016000205411151561071457600080fd5b600c60003373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020019081526020016000205490506000600c60003373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001908152602001600020819055503373ffffffffffffffffffffffffffffffffffffffff166108fc829081150290604051600060405180830381858888f19350505050151561081b5780600c60003373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001908152602001600020819055505b3373ffffffffffffffffffffffffffffffffffffffff167ff279e6a1f5e320cca91135676d9cb6e44ca8a08c0b88342bcdb1144f6511b5688242604051808381526020018281526020019250505060405180910390a25b5b5b50565b60075481565b6000806000600260149054906101000a900460ff1615151561089e57600080fd5b600260009054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff161415156108fa57600080fd5b8360098160001916905550600954601f60208110151561091657fe5b1a7f010000000000000000000000000000000000000000000000000000000000000002600a60006101000a81548160ff02191690837f010000000000000000000000000000000000000000000000000000000000000090040217905550600b6000600a60009054906101000a90047f0100000000000000000000000000000000000000000000000000000000000000027effffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff19167effffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff191681526020019081526020016000209250828054905060088190555060006008541115610ad3576008543073ffffffffffffffffffffffffffffffffffffffff1631811515610a3457fe5b049150600090505b600854811015610ad25781600c60008584815481101515610a5957fe5b906000526020600020900160005b9054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001908152602001600020819055505b8080600101915050610a3c565b5b7fce2892841501496330839ba407f44943da10eb31b71db5c90df80c1ae1d7d1d3600a60009054906101000a90047f01000000000000000000000000000000000000000000000000000000000000000260405180827effffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff19167effffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff1916815260200191505060405180910390a15b5b5b50505050565b600a60009054906101000a90047f01000000000000000000000000000000000000000000000000000000000000000281565b60035481565b600260009054906101000a900473ffffffffffffffffffffffffffffffffffffffff1681565b600260149054906101000a900460ff16151515610bff57600080fd5b600260009054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff16141515610c5b57600080fd5b7f16c8ec2c882242e0bca92f0cc3b1f1e752044b71aee579d87cda6245cc71a8ca3073ffffffffffffffffffffffffffffffffffffffff16316040518082815260200191505060405180910390a18073ffffffffffffffffffffffffffffffffffffffff16ff5b5b5b50565b60065481565b6000806000600260149054906101000a900460ff16151515610cee57600080fd5b6007544310158015610d065750610100600754014311155b8015610d1f575060095460001916600754406000191614155b1515610d2a57600080fd5b600260009054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff16141515610d8657600080fd5b6007544060098160001916905550600954601f602081101515610da557fe5b1a7f010000000000000000000000000000000000000000000000000000000000000002600a60006101000a81548160ff02191690837f010000000000000000000000000000000000000000000000000000000000000090040217905550600b6000600a60009054906101000a90047f0100000000000000000000000000000000000000000000000000000000000000027effffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff19167effffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff191681526020019081526020016000209250828054905060088190555060006008541115610f62576008543073ffffffffffffffffffffffffffffffffffffffff1631811515610ec357fe5b049150600090505b600854811015610f615781600c60008584815481101515610ee857fe5b906000526020600020900160005b9054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001908152602001600020819055505b8080600101915050610ecb565b5b7fce2892841501496330839ba407f44943da10eb31b71db5c90df80c1ae1d7d1d3600a60009054906101000a90047f01000000000000000000000000000000000000000000000000000000000000000260405180827effffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff19167effffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff1916815260200191505060405180910390a15b5b5b5b505050565b6000600260149054906101000a900460ff16151561103157600080fd5b600454341015151561104257600080fd5b600b6000837effffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff19167effffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff1916815260200190815260200160002080548060010182816110ab919061134c565b916000526020600020900160005b33909190916101000a81548173ffffffffffffffffffffffffffffffffffffffff021916908373ffffffffffffffffffffffffffffffffffffffff160217905550506003543073ffffffffffffffffffffffffffffffffffffffff1631101515611238576000600260146101000a81548160ff021916908315150217905550600a430160078190555060646005543073ffffffffffffffffffffffffffffffffffffffff16310281151561116957fe5b049050600260009054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff166108fc829081150290604051600060405180830381858888f1935050505015156111ed576001600260146101000a81548160ff02191690831515021790555060006007819055505b7fc15f25c4eb6ac5b9dc23661b850cbdb3ee36aebed198ad870ab929bae77142de600260149054906101000a900460ff16604051808215151515815260200191505060405180910390a15b7fe8d947d7ebdd7b8b8fa2ad2022c7591418ac32d8c29c5f8a8fc9de52ffa540923073ffffffffffffffffffffffffffffffffffffffff16316040518082815260200191505060405180910390a13373ffffffffffffffffffffffffffffffffffffffff167fa59307e8fe3d4b654d8d230e2ca86c8cfd44a732b78879c2afed9c90c8f00c1a834260405180837effffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff19167effffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff191681526020018281526020019250505060405180910390a25b5b5b5050565b60095481565b60045481565b60085481565b600260149054906101000a900460ff1681565b815481835581811511611373578183600052602060002091820191016113729190611378565b5b505050565b61139a91905b8082111561139657600081600090555060010161137e565b5090565b905600a165627a7a72305820344308e197717bbd2dd06cfb229492fb3a13c116bd71ed8d22cf8441be52966e0029";

// unlock the owner account
web3.personal.unlockAccount(owner, "YOUR_VERY_SECRET_PASSWORD");

// create contract
var contract = web3.eth.contract(abi);
var ETHLottery = contract.new(manager_address, fee, jackpot, owner_fee, { from: owner, data: code, gas: 1000000 });

// make sure you lock the owner account
web3.personal.lockAccount(owner);
```

### Send the result and find out the winners 

After lottery is closed, by contract balance hitting the jackpot, it is possible to run the lottery to know the result.

This action need to be performed after 10 blocks have passed from the lottery close block.

```shell
// unlock the owner account
web3.personal.unlockAccount(owner, "YOUR_VERY_SECRET_PASSWORD");

// call the lottery method while the 10th block is still available 
ETHLottery.lottery();

// make sure you lock the owner account
web3.personal.lockAccount(owner);
```

## How to play using the official ETHLottery client

The official ETHLottery client is available under this [github repository](https://github.com/re2005/ETHLottery-Client)

Follow the instructions from the README page on how to install the official ETHLottery client.


### Live official ETHLottery Client demo

There is also a [live demo](https://lifelottery.github.io/) currently running against the ropsten test network

Demo requires metamask installed on latest version of google chrome browser.

There is a step by step manual on how to play using the official ETHLottery client.
