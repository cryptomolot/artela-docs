---
sidebar_position: 2
---

# Operation Interface

## Introduction

Operation is a special interface. Unlike other join points, which are triggered during transaction execution, this
interface can be called directly by an operation transaction signed by EoA. The operation transaction will trigger the
execution of the Aspect.

![op.png](op.png)

In this interface execution, Aspect is only able to access the Aspect state. Runtime context accessing is unsupported due to it isn't triggered by the transaction lifecycle join point.

You can use this interface to manage the Aspect state. For example, there is a whitelist Aspect that will be triggered pre-transaction execution; you can insert and update the whitelist by operation interface.

## Interface

```
interface IAspectOperation {
  operation(input: OperationInput): Uint8Array;
}
```

* **Parameter**
    * input: OperationInput; The base layer will deliver the OperationInput object to Aspect in this join point.
        - `input.block.number`: current block number.
        - `input.tx.from`: caller of the transaction.
        - `input.tx.to`: to address of the transaction.
        - `input.tx.hash`: hash of the transaction.
* **Returns**
    * Uint8Array; operation result.

## Example

To implement a Operation Aspect, you can implement the `IAspectOperation` interface

```typescript

import {
    allocate,
    entryPoint,
    execute,
    IAspectOperation,
    OperationInput,
    stringToUint8Array,
    sys,
} from '@artela/aspect-libs';

class AspectTest implements IAspectOperation {
    operation(input: OperationInput): Uint8Array {
        sys.require(input.callData.length > 0, 'data is lost');
        sys.aspect.mutableState.get<string>("k2").set<string>("v2")
        const val = sys.aspect.mutableState.get<string>("k2").unwrap()!
        sys.require(val == "v2", val + "mutableState get fail")
        return stringToUint8Array('success');
    }
}

// 2.register aspect Instance
const aspect = new AspectTest();
// Note: This is very important and requires set to work.
entryPoint.setOperationAspect(aspect);

// 3.must export it
export {execute, allocate};

```

## Programming Guide

There are two programming modes that can be used in this method:

1. By utilizing the 'input' input argument, it provides essential insights into transactions and block processing.
   see [how to use input](#how-to-use-input).

2. Using the 'sys' namespace, it provides both hight level API and low-level API access to system data and contextual
   information generated during blockchain runtime, including details about the environment, blocks, transactions, and
   utility classes such as crypto and ABI encoding/decoding. see [more details](#how-to-use-apis).

## Host APIs

For a comprehensive overview of all APIs and their usage
see [API References](/develop/reference/aspect-lib/components/overview).

Each join-point has access to different host APIs, and the host APIs available within the current breakpoint can be
found at the following table.

| System APIs                                                                                                                 | Availability | Description                                                                                                                                              |
|-----------------------------------------------------------------------------------------------------------------------------|--------------|----------------------------------------------------------------------------------------------------------------------------------------------------------|
| [sys.revert](/develop/reference/aspect-lib/components/sys#1-revert)                                                         | ✅            | Forces the current transaction to fail.                                                                                                                  |
| [sys.require](/develop/reference/aspect-lib/components/sys#2-require)                                                       | ✅            | Checks if certain conditions are met; if not, forces the entire transaction to fail.                                                                     |
| [sys.log](/develop/reference/aspect-lib/components/sys#3-log)                                                               | ✅            | A wrapper for `sys.hostApi.util.log`, prints log messages to Artela output for debugging on the localnet.                                                |
| [sys.aspect.id](/develop/reference/aspect-lib/components/sys-aspect#1-sysaspectid)                                          | ✅            | Retrieves the ID of the aspect.                                                                                                                          |
| [sys.aspect.version ](/develop/reference/aspect-lib/components/sys-aspect#2-sysaspectversion)                               | ✅            | Retrieves the version of the aspect.                                                                                                                     |
| [sys.aspect.mutableState](/develop/reference/aspect-lib/components/sys-aspect#4-sysaspectmutablestate)                      | ✅            | A wrapper for `sys.hostApi.aspectState` that facilitates easier reading or writing of values of a specified type to aspect state.                        |
| [sys.aspect.property](/develop/reference/aspect-lib/components/sys-aspect#5-sysaspectproperty)                              | ✅            | A wrapper for `sys.hostApi.aspectProperty` that facilitates easier reading of values of a specified type from aspect property.                           |
| [sys.aspect.readonlyState](/develop/reference/aspect-lib/components/sys-aspect#3-sysaspectreadonlystate)                    | ✅            | A wrapper for `sys.hostApi.aspectState` that facilitates easier reading of values of a specified type from aspect state.                                 |
| [sys.aspect.transientStorage](/develop/reference/aspect-lib/components/sys-aspect#6-sysaspecttransientstorage)              | ❌            | A wrapper for `sys.hostApi.aspectTransientStorage` that facilitates easier reading or writing of values of a specified type to aspect transient storage. |
| [sys.hostApi.aspectProperty](/develop/reference/aspect-lib/components/sys-hostapi#syshostapiaspectproperty)                 | ✅            | Retrieves the property of the aspect as written in aspect deployment.                                                                                    |
| [sys.hostApi.aspectState](/develop/reference/aspect-lib/components/sys-hostapi#syshostapiaspectstate)                       | ✅            | Retrieves or writes the state of the aspect.                                                                                                             |
| [sys.hostApi.aspectTransientStorage](/develop/reference/aspect-lib/components/sys-hostapi#syshostapiaspecttransientstorage) | ❌            | Retrieves or writes to the transient storage of the aspect. This storage is only valid within the current transaction lifecycle.                         |
| [sys.hostApi.crypto.ecRecover](/develop/reference/aspect-lib/components/sys-hostapi#4-ecrecover)                            | ✅            | Calls crypto methods `ecRecover`.                                                                                                                        |
| [sys.hostApi.crypto.keccak](/develop/reference/aspect-lib/components/sys-hostapi#1-keccak)                                  | ✅            | Calls crypto methods `keccak`.                                                                                                                           |
| [sys.hostApi.crypto.ripemd160](/develop/reference/aspect-lib/components/sys-hostapi#3-ripemd160)                            | ✅            | Calls crypto methods `ripemd160`.                                                                                                                        |
| [sys.hostApi.crypto.sha256](/develop/reference/aspect-lib/components/sys-hostapi#2-sha256)                                  | ✅            | Calls crypto methods `sha256`.                                                                                                                           |
| [sys.hostApi.runtimeContext](/develop/reference/aspect-lib/components/sys-hostapi#1-get-context)                            | ✅            | Retrieves runtime context by the key.  Refer to the  [runtime context](#runtime-context)  to see which keys can be accessed by the current join point.   |
| [sys.hostApi.stateDb.balance](/develop/reference/aspect-lib/components/sys-hostapi#1-balance)                               | ✅            | Gets the balance of the specified address from the EVM state database.                                                                                   |
| [sys.hostApi.stateDb.codeHash](/develop/reference/aspect-lib/components/sys-hostapi#4-codehash)                             | ✅            | Gets the hash of the code from the EVM state database.                                                                                                   |
| [sys.hostApi.stateDb.codeSize](/develop/reference/aspect-lib/components/sys-hostapi#6-codesize)                             | ✅            | Gets the size of the code from the EVM state database.                                                                                                   |
| [sys.hostApi.stateDb.hasSuicided](/develop/reference/aspect-lib/components/sys-hostapi#3-hassuicided)                       | ✅            | Gets the codehash from the EVM state database.                                                                                                           |
| [sys.hostApi.stateDb.nonce](/develop/reference/aspect-lib/components/sys-hostapi#5-nonce)                                   | ✅            | Checks if the contract at the specified address is suicided in the current transactions.                                                                 |
| [sys.hostApi.stateDb.stateAt](/develop/reference/aspect-lib/components/sys-hostapi#2-stateat)                               | ✅            | Gets the state at a specific point.                                                                                                                      |
| [sys.hostApi.evmCall.jitCall](/develop/reference/aspect-lib/components/sys-hostapi#2-jitcall)                               | ❌            | Creates a contract call and executes it immediately.                                                                                                     |
| [sys.hostApi.evmCall.staticCall](/develop/reference/aspect-lib/components/sys-hostapi#1-staticcall)                         | ✅            | Creates a static call and executes it immediately.                                                                                                       |
| [sys.hostApi.trace.queryCallTree](/develop/reference/aspect-lib/components/sys-hostapi#2-querycalltree )                    | ❌            | Returns the call tree of EVM execution.                                                                                                                  |
| [sys.hostApi.trace.queryStateChange](/develop/reference/aspect-lib/components/sys-hostapi#1-querystatechange)               | ❌            | Returns the state change in EVM execution for the specified key.                                                                                         |

## Runtime context

The Aspect Runtime Context encapsulates data generated through the consensus process. With the acquired Runtime Context
object, retrieve specific data by specifying the relevant Context Key. Each Context Key is associated with a particular
type of data or information.

### Usage

```javascript

const aspectId = sys.hostApi.runtimeContext.get('aspect.id');
// decode BytesData
const aspectIdData = Protobuf.decode<BytesData>(aspectId, BytesData.decode);
sys.log( 'aspect.id' + ' ' + uint8ArrayToHex(aspectIdData.data));

const aspectVer = sys.hostApi.runtimeContext.get('aspect.version');
// decode UintData
const aspectVerData = Protobuf.decode<UintData>(aspectVer, UintData.decode);
sys.log( 'aspect.version' + ' ' + aspectVerData.data.toString(10));

``` 

### Key table

| Context key                                  | Value type                                                                                 | Description                                                                                                                                                                               |
|----------------------------------------------|--------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| isCall                                       | <a href="/api/docs/classes/proto.BoolData.html" target="_blank">BoolData</a>               | Get the current transaction is **Call** or **Send**. If it is **Call**, return true.                                                                                                      |
| tx.type                                      | <a href="/api/docs/classes/proto.UintData.html" target="_blank">UintData</a>               | Returns the transaction type id. LegacyTxType=0x00 AccessListTxType=0x01 DynamicFeeTxType=0x02 BlobTxType=0x03                                                                            |
| tx.chainId                                   | <a href="/api/docs/classes/proto.BytesData.html" target="_blank">BytesData</a>             | Returns the EIP155 chain ID of the transaction. The return value will always be non-nil. For legacy transactions which are not replay-protected, the return value is zero.                |
| tx.accessList                                | <a href="/api/docs/classes/proto.EthAccessList.html" target="_blank">EthAccessList</a>     | AccessListTx is the data of EIP-2930 access list transactions.                                                                                                                            |
| tx.nonce                                     | <a href="/api/docs/classes/proto.UintData.html" target="_blank">UintData</a>               | Returns the sender account nonce of the transaction.                                                                                                                                      |
| tx.gasPrice                                  | <a href="/api/docs/classes/proto.BytesData.html" target="_blank">BytesData</a>             | Returns the gas price of the transaction.                                                                                                                                                 |
| tx.gas                                       | <a href="/api/docs/classes/proto.UintData.html" target="_blank">UintData</a>               | Returns the gas limit of the transaction.                                                                                                                                                 |
| tx.gasTipCap                                 | <a href="/api/docs/classes/proto.BytesData.html" target="_blank">BytesData</a>             | Returns the gasTipCap per gas of the transaction.                                                                                                                                         |
| tx.gasFeeCap                                 | <a href="/api/docs/classes/proto.BytesData.html" target="_blank">BytesData</a>             | Returns the fee cap per gas of the transaction.                                                                                                                                           |
| tx.to                                        | <a href="/api/docs/classes/proto.BytesData.html" target="_blank">BytesData</a>             | Returns the recipient address of the transaction. For contract-creation transactions, To returns nil.                                                                                     |
| tx.value                                     | <a href="/api/docs/classes/proto.BytesData.html" target="_blank">BytesData</a>             | Returns the ether amount of the transaction.                                                                                                                                              |
| tx.data                                      | <a href="/api/docs/classes/proto.BytesData.html" target="_blank">BytesData</a>             | Returns the input data of the transaction.                                                                                                                                                |
| tx.bytes                                     | <a href="/api/docs/classes/proto.BytesData.html" target="_blank">BytesData</a>             | Returns the transaction marshal binary.                                                                                                                                                   |
| tx.hash                                      | <a href="/api/docs/classes/proto.BytesData.html" target="_blank">BytesData</a>             | Returns the transaction hash.                                                                                                                                                             |
| tx.unsigned.bytes                            | <a href="/api/docs/classes/proto.BytesData.html" target="_blank">BytesData</a>             | Returns the unsigned transaction marshal binary.                                                                                                                                          |
| tx.unsigned.hash                             | <a href="/api/docs/classes/proto.BytesData.html" target="_blank">BytesData</a>             | Returns the unsigned transaction hash.                                                                                                                                                    |
| tx.sig.v                                     | <a href="/api/docs/classes/proto.BytesData.html" target="_blank">BytesData</a>             | Returns the V signature values of the transaction.                                                                                                                                        |
| tx.sig.r                                     | <a href="/api/docs/classes/proto.BytesData.html" target="_blank">BytesData</a>             | Returns the R signature values of the transaction.                                                                                                                                        |
| tx.sig.s                                     | <a href="/api/docs/classes/proto.BytesData.html" target="_blank">BytesData</a>             | Returns the S signature values of the transaction.                                                                                                                                        |
| tx.from                                      | <a href="/api/docs/classes/proto.BytesData.html" target="_blank">BytesData</a>             | Returns the from address of the transaction.                                                                                                                                              |
| tx.index                                     | <a href="/api/docs/classes/proto.UintData.html" target="_blank">UintData</a>               | Returns the transaction index of current block.                                                                                                                                           |
| block.header.parentHash                      | <a href="/api/docs/classes/proto.BytesData.html" target="_blank">BytesData</a>             | Get the current block header parent hash.                                                                                                                                                 |
| block.header.miner                           | <a href="/api/docs/classes/proto.BytesData.html" target="_blank">BytesData</a>             | Get the current block header miner.                                                                                                                                                       |
| block.header.transactionsRoot                | <a href="/api/docs/classes/proto.BytesData.html" target="_blank">BytesData</a>             | Get the current block TransactionsRoot hash.                                                                                                                                              |
| block.header.number                          | <a href="/api/docs/classes/proto.UintData.html" target="_blank">UintData</a>               | Get the current block number.                                                                                                                                                             |
| block.header.timestamp                       | <a href="/api/docs/classes/proto.UintData.html" target="_blank">UintData</a>               | Get the current block header timestamp.                                                                                                                                                   |
| env.extraEIPs                                | <a href="/api/docs/classes/proto.IntArrayData.html" target="_blank">IntArrayData</a>       | Retrieve the EVM module parameters for the '**extra_eips**': defines the additional EIPs for the vm.Config.                                                                               |
| env.enableCreate                             | <a href="/api/docs/classes/proto.BoolData.html" target="_blank">BoolData</a>               | Retrieve the EVM module parameters for the '**enable_create**': toggles states transitions that use the vm.Create function.                                                               |
| env.enableCall                               | <a href="/api/docs/classes/proto.BoolData.html" target="_blank">BoolData</a>               | Retrieve the EVM module parameters for the '**enable_call**': toggles states transitions that use the vm.Call function.                                                                   |
| env.allowUnprotectedTxs                      | <a href="/api/docs/classes/proto.BoolData.html" target="_blank">BoolData</a>               | Retrieve the EVM module parameters for the '**allow_unprotected_txs**': defines if replay-protected (i.e non EIP155 // signed) transactions can be executed on the states machine.        |
| env.chain.chainId                            | <a href="/api/docs/classes/proto.UintData.html" target="_blank">UintData</a>               | Retrieve the Ethereum chain config id.                                                                                                                                                    |
| env.chain.homesteadBlock                     | <a href="/api/docs/classes/proto.UintData.html" target="_blank">UintData</a>               | Retrieve the Ethereum chain configuration for the '**homestead_block**': switch (nil no fork, 0 = already homestead)                                                                      |
| env.chain.daoForkBlock                       | <a href="/api/docs/classes/proto.UintData.html" target="_blank">UintData</a>               | Retrieve the Ethereum chain configuration for the '**dao_fork_block**': corresponds to TheDAO hard-fork switch block (nil no fork)                                                        |
| env.chain.daoForkSupport                     | <a href="/api/docs/classes/proto.BoolData.html" target="_blank">BoolData</a>               | Retrieve the Ethereum chain configuration for the '**dao_fork_support**': defines whether the nodes supports or opposes the DAO hard-fork                                                 |
| env.chain.eip150Block                        | <a href="/api/docs/classes/proto.UintData.html" target="_blank">UintData</a>               | Retrieve the Ethereum chain configuration for the '**eip150_block**': EIP150 implements the Gas price changes (https://github.com/ethereum/EIPs/issues/150) EIP150 HF block (nil no fork) |
| env.chain.eip155Block                        | <a href="/api/docs/classes/proto.UintData.html" target="_blank">UintData</a>               | Retrieve the Ethereum chain configuration for the '**eip155_block**'.                                                                                                                     |
| env.chain.eip158Block                        | <a href="/api/docs/classes/proto.UintData.html" target="_blank">UintData</a>               | Retrieve the Ethereum chain configuration for the '**eip158_block**'.                                                                                                                     |
| env.chain.byzantiumBlock                     | <a href="/api/docs/classes/proto.UintData.html" target="_blank">UintData</a>               | Retrieve the Ethereum chain configuration for the '**byzantium_block**': Byzantium switch block (nil no fork, 0 =already on byzantium)                                                    |
| env.chain.constantinopleBlock                | <a href="/api/docs/classes/proto.UintData.html" target="_blank">UintData</a>               | Retrieve the Ethereum chain configuration for the '**constantinople_block**': Constantinople switch block (nil no fork, 0 = already activated)                                            |
| env.chain.petersburgBlock                    | <a href="/api/docs/classes/proto.UintData.html" target="_blank">UintData</a>               | Retrieve the Ethereum chain configuration for the '**petersburg_block**': Petersburg switch block (nil no fork, 0 = already activated)                                                    |
| env.chain.istanbulBlock                      | <a href="/api/docs/classes/proto.UintData.html" target="_blank">UintData</a>               | Retrieve the Ethereum chain configuration for the '**istanbul_block**': Istanbul switch block (nil no fork, 0 = already on istanbul)                                                      |
| env.chain.muirGlacierBlock                   | <a href="/api/docs/classes/proto.UintData.html" target="_blank">UintData</a>               | Retrieve the Ethereum chain configuration for the '**muir_glacier_block**': Eip-2384 (bomb delay) switch block ( nil no fork, 0 = already activated).                                     |
| env.chain.berlinBlock                        | <a href="/api/docs/classes/proto.UintData.html" target="_blank">UintData</a>               | Retrieve the Ethereum chain configuration for the '**berlin_block**': Berlin switch block (nil = no fork, 0 = already on berlin)                                                          |
| env.chain.londonBlock                        | <a href="/api/docs/classes/proto.UintData.html" target="_blank">UintData</a>               | Retrieve the Ethereum chain configuration for the '**london_block**': London switch block (nil = no fork, 0 = already on london)                                                          |
| env.chain.arrowGlacierBlock                  | <a href="/api/docs/classes/proto.UintData.html" target="_blank">UintData</a>               | Retrieve the Ethereum chain configuration for the '**arrow_glacier_block**': Eip-4345 (bomb delay) switch block (nil = no fork, 0 = already activated)                                    |
| env.chain.grayGlacierBlock                   | <a href="/api/docs/classes/proto.UintData.html" target="_blank">UintData</a>               | Retrieve the Ethereum chain configuration for the '**gray_glacier_block**': EIP-5133 (bomb delay) switch block (nil = no fork, 0 = already activated)                                     |
| env.chain.mergeNetSplitBlock                 | <a href="/api/docs/classes/proto.UintData.html" target="_blank">UintData</a>               | Retrieve the Ethereum chain configuration for the '**merge_netsplit_block**': Virtual fork after The Merge to use as a network splitter.                                                  |
| env.chain.shanghaiTime                       | <a href="/api/docs/classes/proto.UintData.html" target="_blank">UintData</a>               | Retrieve the Ethereum chain configuration for the '**shanghaiTime**': Shanghai switch time (nil = no fork, 0 = already on shanghai).                                                      |
| env.chain.cancunTime                         | <a href="/api/docs/classes/proto.UintData.html" target="_blank">UintData</a>               | Retrieve the Ethereum chain configuration for the '**CancunTime**': Cancun switch time (nil = no fork, 0 = already on cancun).                                                            |
| env.chain.pragueTime                         | <a href="/api/docs/classes/proto.UintData.html" target="_blank">UintData</a>               | Retrieve the Ethereum chain configuration for the '**PragueTime**': Prague switch time (nil = no fork, 0 = already on prague).                                                            |
| env.consensusParams.block.maxGas             | <a href="/api/docs/classes/proto.IntData.html" target="_blank">IntData</a>                 | Retrieve the max gas per block.                                                                                                                                                           |
| env.consensusParams.block.maxBytes           | <a href="/api/docs/classes/proto.IntData.html" target="_blank">IntData</a>                 | Retrieve the max block size, in bytes.                                                                                                                                                    |
| env.consensusParams.evidence.maxAgeDuration  | <a href="/api/docs/classes/proto.IntData.html" target="_blank">IntData</a>                 | Retrieve the max age duration.It should correspond with an app's "unbonding period" or other similar mechanism for handling.                                                              |
| env.consensusParams.evidence.maxAgeNumBlocks | <a href="/api/docs/classes/proto.IntData.html" target="_blank">IntData</a>                 | The basic formula for calculating this is: MaxAgeDuration / {average block time}.                                                                                                         |
| env.consensusParams.evidence.maxBytes        | <a href="/api/docs/classes/proto.IntData.html" target="_blank">IntData</a>                 | Retrieve the maximum size of total evidence in bytes that can be committed in a single block.                                                                                             |
| env.consensusParams.validator.pubKeyTypes    | <a href="/api/docs/classes/proto.StringArrayData.html" target="_blank">StringArrayData</a> | Restrict the public key types validators can use.                                                                                                                                         |
| env.consensusParams.appVersion               | <a href="/api/docs/classes/proto.UintData.html" target="_blank">UintData</a>               | Get the ABCI application version.                                                                                                                                                         |
| aspect.id                                    | <a href="/api/docs/classes/proto.BytesData.html" target="_blank">BytesData</a>             | Returns current aspect id.                                                                                                                                                                |
| aspect.version                               | <a href="/api/docs/classes/proto.UintData.html" target="_blank">UintData</a>               | Returns current aspect version.                                                                                                                                                           |

Here is a case demonstration on how to [create an operation call](/develop/guides/operation-aspect).
Through this example, you can gain a clearer understanding of how to utilize Operation Interface.
