# go-coin-wallet

## 添加BSC 测试网络并领取测试币 

获取 erc20 代币需要换个地址

https://mirror.xyz/0xac8B5BF3e669DC1eD2D1596c2D261DA878816f40/icqx5vdBAVogJ4fbh06J4ELY5WvX10nOrmS0RsnXlhE

## bsc 区块浏览器

* 测试网： https://testnet.bscscan.com/
* 主网: bscscan.com

## 参考demo 目录

```golang
package main

import (
	"fmt"
	"math/big"

	coin "github.com/0xhelloweb3/go-coin-wallet/eth"
	"github.com/ethereum/go-ethereum/common"
)

var (
	rpcUrl = "https://data-seed-prebsc-1-s1.binance.org:8545"

	walletAddress = "0xB553803EE21b486BB86f2A63Bd682529Aa7FCE8D"

	privateKey = ""

	// bsc 测试网 busd 合约地址
	busdContractAddress = "0xeD24FC36d5Ee211Ea25A80239Fb8C4Cfd80f12Ee"
)


func main() {
	wallet := coin.NewEthChain()
	wallet.InitRemote(rpcUrl)

	// 获取主网代币 BNB 余额
	balance, _ := wallet.Balance(walletAddress)
	fmt.Printf("bnb balance: %v\n", balance)

	// 获取 Erc20代币 余额
	busdBalance, _ := wallet.TokenBalance(busdContractAddress, walletAddress)

	tokenDecimal, err := wallet.TokenDecimal(busdContractAddress, walletAddress)
	fmt.Printf("busdBalance balance: %v, decimal: %v \n", busdBalance, tokenDecimal)

	if err != nil {
		fmt.Printf("get busdt balance error: %v\n", err)
		return
	}
	nonce, _ := wallet.Nonce(walletAddress)

	// 构造多笔交易则nonce + 1
	callMethodOpts := &coin.CallMethodOpts{
		Nonce: nonce,
	}

	// erc20 代币转账
	buildTxResult, err := wallet.BuildCallMethodTx(
		privateKey,
		busdContractAddress,
		coin.Erc20AbiStr,
		// 调用的合约方法名
		"transfer",
		callMethodOpts,
		// 转账目标地址
		common.HexToAddress("0x4165FD787ffF9f659A3B9A239a1d70fc5B8aB6d1"),
		big.NewInt(10000000000))

	if err != nil {
		fmt.Printf("build call method tx error: %v\n", err)
	}
	// 发送交易
	sendTxResult, err := wallet.SendRawTransaction(buildTxResult.TxHex)
	if err != nil {
		fmt.Printf("send raw transaction error: %v\n", err)
	}
	// 打印交易hash
	fmt.Printf("sendTxResult: %v\n", sendTxResult)

	// 检测 transfer 事件， fromBlock 和 toBlock 参数可以不传
	eventlogs, _ := wallet.FindLogs(busdContractAddress, coin.Erc20AbiStr, "Transfer",
		big.NewInt(17174691), big.NewInt(17174691), nil)
	fmt.Printf("eventlogs: %v\n", eventlogs)
}
```
