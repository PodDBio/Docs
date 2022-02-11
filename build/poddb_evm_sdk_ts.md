# POD EVM TS SDK

This SDK is a tool for developers to interact with POD contracts deployed on EVM-compatible blockchain networks.

For more information about POD, see [this doc](https://poddbio.github.io/Docs/#/learn/Technology).

## Installation

Yarn

```shell
yarn add poddb-evm-sdk-ts
```

## Contract Address

**Mainnet**

| Chain | Contract | Address |
| ------------- | ------------- |------------- |
| Polygon|TagClass|0x2D606CedEb061AA491F4D4AeaD6f02930A9B5600|
| Polygon|Tag|0x0d2D5549102044D4B8dDa7F608F2d692C39eB467|

**Testnet**

| Chain | Contract | Address |
| ------------- | ------------- |------------- |
| ETH(Rinkeby)|TagClass|0x0EC77Da842Bd72a88378d5b0DBa7dd7fF0B525C7|
| ETH(Rinkeby)|Tag|0x78FdcF13bCaec0EaeFd225316df978E24c584E3B|
| BSC|TagClass|0x1E94Bb8168B20AaE7da2b4072c3136f7f68093c4|
| BSC|Tag|0x67b75CaC3CEEa80A68c622aAA5Ee022d4E139ff0|
| AVAX|TagClass|0x1A2b3E7945A73E782cE3cC117F8199E2030f70Eb|
| AVAX|Tag|0xEE122D2A6ECd381E312911f30bA2739a8e3841fA|
| FTM|TagClass|0x462B75b54673f0Ac5E812707DbcEe55D2C906A92|
| FTM|Tag|0x74134A463370d272E08f1bD7A0b64883fD9d62C6|
| Polygon|TagClass|0xBCC70346547A825E03F98ED6d2c5dF246AE09b5C|
| Polygon|Tag|0x9D92B04811376bB133E155c9420c91E6A7FA9B8B|

## Connect to POD Contract

### Obtain Provider

To connect to POD contracts, first you need an Ethereum provider object, which can be obtained by connecting to a JsonRpc node in a nodejs environment:

```typescript
const provider = new ethers.providers.JsonRpcProvider(
'http://127.0.0.1:8545', //ethereum node rpc url
);
```

Some chains, such as BSC and AVAX, have built in RPC URLs with their names:

```typescript
const BSCMainnetDefaultRpcUrl = 'https://bsc-dataseed1.binance.org/';
const BSCTestnetDefaultRpcUrl = 'https://data-seed-prebsc-1-s1.binance.org:8545/';
const AVAXMainnetDefaultRpcUrl = 'https://api.avax.network/ext/bc/C/rpc';
const AVAXTestnetDefaultRpcUrl = 'https://api.avax-test.network/ext/bc/C/rpc';
const FTMMainnetDefaultRpcUrl = 'https://rpcapi.fantom.network';
const FTMTestnetDefaultRpcUrl = 'https://rpc.testnet.fantom.network/';
const PolygonMainnetDefaultRpcUrl = 'https://polygon-rpc.com/';
const PolygonTestnetDefaultRpcUrl = 'https://rpc-mumbai.matic.today';
```

If you use a web browser, you can obtain the provider by sending requests to a wallet plugin such as MetaMask:

```typescript
const provider = new ethers.providers.Web3Provider(window.ethereum);
```

### Connect to Contract

Connect to a POD's tag contract and tagClass contract:

```typescript
const tagContract = await TagClassContract.getTagContractV1(provider, 'TagContractAddress');
const tagClassContract = await TagClassContract.getTagClassContractV1(provider, 'TagClassContractAddress');
```

### Connect to Signer

If you need to send transactions, for example to create a TagClass or a Tag, you also need a signer to sign the transactions. In the nodejs environment, it can be obtained directly through the private key.

```typescript
const signer = new ethers.Wallet(
'0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80', //private key
provider,
);
```

If you use a web browser, you can send a request to a wallet plugin such as MetaMask:

```typescript
const provider = new ethers.providers.Web3Provider(window.ethereum);
await window.ethereum.request({ method: 'eth_requestAccounts' });
const signer = provider.getSigner();
```

Connect POD's tag & tagClass contract to the signer:

```typescript
tagContract.connectSigner(signer);
tagClassContract.connectSigner(signer);
```

## Create TagClass

### Define Fields

When creating a TagClass, you need to define the `FieldTypes` and `FieldName`.

POD supports the following data types:

```typescript
enum TagFieldType {
    Bool,     //= 0
    Uint256,  //= 1
    Uint8,    //= 2
    Uint16,   //= 3
    Uint32,   //= 4
    Uint64,   //= 5
    Int256,   //= 6
    Int8,     //= 7
    Int16,    //= 8
    Int32,    //= 9
    Int64,    //= 10
    Bytes1,   //= 11
    Bytes2,   //= 12
    Bytes3,   //= 13
    Bytes4,   //= 14
    Bytes8,   //= 15
    Bytes20,  //= 16
    Bytes32,  //= 17
    Address,  //= 18
    Bytes,    //= 19
    String,   //= 20
    Array     //= 21
}
```

The `array` type is supported (not nested), but the `structure` type is not.

`FieldTypes` and `FieldNames` can be empty if TagClass does not need to store any data.

### Set Agent

Only the owner of TagClass has write-permission. But a TagClass Owner and can delegate write permissions to a TagAgent.

The Agent consists of `AgentType` and a string.

```typescript
export enum AgentType {
    Address,
    TagClass,
}
```

When the `AgentType` holds `Address`, the Agent is an externally owned address (EOA), or a contract address (CA);

When the `AgentType` holds `TagClass`, the Agent is a TagClass, any address (EOA, or CA) that is associated with Tag belonging to this TagClass has write-permissions.

**Example**

This is the full code for creating an example TagClass:

```typescript
const tagClassContract = (
    await TagClassContract.getTagClassContractV1(provider, 'TagClassContractAddress')
).connectSigner(signer);

const agent = TagAgent.fromAddress('address');
const newTagClassTx = await tagClassContract.newValueTagClass(
    'TagClassName',
    'TagClassDesc',
    [
        {
            fieldName: 'field1',
            fieldType: TagFieldType.String,
        },
        {
            fieldName: 'field2',
            fieldType: TagFieldType.Uint8,
        },
    ],
    {
        agent,
    },
);
newTagClassTx.wait(); //Waiting for transaction confirmation
````

## SetTag

A Tag is a basic unit of data contained in a TagClass. Each Tag must be associated with a TagObject. The operation of writing data is called `SetTag`.

If you write data for one TagObject and TagClass more than once, the new data overwrites the old one.

### Create TagObject

```typescript
export enum ObjectType {
  Address,
  NFT,
  TagClass,
}
```

A TagObject consists of three parts: `ObjectType`, `Address` and `TokenId`. A TagObject can represent an address (EOA, or CA), NFT, or a TagClass.

### Build TagData

The data in a Tag must be serialized and conform to the field definitions of the TagClass that this Tag belongs to.

You can use the `WriteBuffer` to obtain defined types and serialize data accordingly.

**Example**

```typescript
const tagData = new WriteBuffer()
.writeBool(true)
.writeUint256(
    ethers.BigNumber.from(
        "0x56ee41124a9af6b8590735ac413711e05faa6dc2b80f1e5d0cf7a5873ed36947"
    )
)
.writeArray([true, false], TagFieldType.Bool)
.getBytes();
```

### Expiration Time

When creating a Tag, you can set an expiration time for the Tag using an Uint32 number in seconds. The actual expiration time should take into account the block time when the transaction was sent. If the value is 0, the Tag will never expire.

**Example**

SetTag sample code

```typescript
const tagContract = (await TagContract.getTagContractV1(provider, 'TagContractAddress')).connectSigner(signer);

const tagClassId = 'xxx';
const tagObject = TagObject.fromAddress('address');
const data = new WriteBuffer().writeString('Hello').writeUint8(16).getBytes();
const setTagTx = await tagContract.setTag(tagClassId, tagObject, data, {
    expiredTime: 0, //Expiration time of tag in seconds, 0 means never expires
});

setTagTx.wait();
```

## GetTag

The operation of reading data from a Tag is called `GetTag`. There are three functions for reading data from POD: `hasTag`, `getTagData`, and `getTag`.

### hasTag

Calling `hasTag` is used to check whether in a specific TagClass exists a Tag that belongs to a certain TagObject, regardless of what data is stored in the Tag. If no such Tag exists, or the Tag has expired, `false` will be returned.

The `hasTag` is usually used for checking whether a user has certain permissions or rights.

### getTagData

Calling `getTagData` is used to obtain the data stored in a Tag that is related to a certain TagObject from a specific TagClass. If no such Tag exists, or the Tag has expired, an empty byte array will be returned.

### getTag

By using `getTag` you can obtain more metadata of a Tag than using getTagData, such as data expiration time and so on.

### Parse TagData

TagData is serialized data. To parse the fields in TagData, you need to know the field type definition of this TagClass.

Parsing TagData can use the `TagDataParser`.

**Example**

Below is the example code for `Parse TagData`:

```typescript
const tagContract = (await TagContract.getTagContractV1(provider, 'TagContractAddress')).connectSigner(signer);

const tagClassContract = (
    await TagClassContract.getTagClassContractV1(provider, 'TagClassContractAddress')
).connectSigner(signer);

const tagClassId = 'xxx';
const tagObject = TagObject.fromAddress('address');

//get tag
const tag = await tagContract.getTag(tagClassId, tagObject);
//get tagClass
const tagClass = await tagClassContract.getTagClass(tagClassId);
//get tagClassInfo
const tagClassInfo = await tagClassContract.getTagClassInfo(tagClassId);

//parse data
const dataParser = new TagDataParser(tagClassInfo!.fieldNames, tagClass!.fieldTypes, tag!.data);
const field1 = dataParser.get('field1')!.getString(); //or get other types
const field2 = dataParser.get('field2')!.getNumber(); //or get other types
```

## Delete Tag

You can delete unwanted Tag by passing `TagClassId` and `TagObject` to the the `deleteTag` function.

**Example**

```typescript
const tagContract = (await TagContract.getTagContractV1(provider, 'TagContractAddress')).connectSigner(signer);

const tagClassId = 'xxx';
const tagObject = TagObject.fromAddress('address');
const deleteTx = await tagContract.deleteTag(tagClassId, tagObject);
deleteTx.wait();
```

## Update TagClass

After a TagClass is created, the field types and field names are not editable. The TagClass name and description can be modified using the `UpdateTagClassInfo` function. You can change the Owner, Agent, and `TagClassFlags` using the `UpdateTagClass` function.

**Example**

```typescript
const tagClassContract = (
    await TagClassContract.getTagClassContractV1(provider, 'TagClassContractAddress')
).connectSigner(signer);

const tagClassId = 'xxx';
const newAgent = TagAgent.fromAddress('agentAddress');
const updateTagClassTx = await tagClassContract.updateValueTagClass(tagClassId, DefaultTagClassFlags, newAgent);
updateTagClassTx.wait();
```

### TagClassFlags

POD defines TagClass flags represented by Uint8 numbers for changing some default actions in POD.

**Example**

`TagClassFlags` can be constructed through the `TagClassFlagsBuilder` class:

```typescript
const tagClassFlags = new TagClassFlagsBuilder().setDeprecatedFlag().build();
```

#### Deprecated TagClass

Technically there can be up to 8 default flag values. For now only 1 flag has been defined: the Deprecated flag, represented by the Uint8 number with the value of 128.

Once created, the TagClass cannot be deleted, but it can be deprecated using the Deprecated flag. You can view a deprecated TagClass, and view or delete the Tags of the TagClass. But you cannot create a new Tag and modify existing Tags.

**Example**

Deprecate a TagClass:

```typescript
const tagClassContract = (
    await TagClassContract.getTagClassContractV1(provider, 'TagClassContractAddress')
).connectSigner(signer);

const tagClassId = 'xxx';
let tagClass = await tagClassContract.getTagClass(tagClassId);
tagClass!.flags = new TagClassFlagsBuilder(tagClass?.flags).setDeprecatedFlag().build();

const updateTagClassTx = await tagClassContract.updateValueTagClass(tagClassId, tagClass!.flags, tagClass!.agent);
updateTagClassTx.wait();
```

## Parse Log

```typescript
const tagContract = (await TagContract.getTagContractV1(provider, 'TagContractAddress')).connectSigner(signer);

const tagClassId = 'xxx';
const tagObject = TagObject.fromAddress('address');
const data = new WriteBuffer().writeString('Hello').writeUint8(16).getBytes();
const setTagTx = await tagContract.setTag(tagClassId, tagObject, data, {
    expiredTime: 0, //Expiration time of tag in seconds, 0 means never expires
});

setTagTx.wait();

const setTagReceipt = await provider.getTransactionReceipt(setTagTx.hash);
const setTagLog = await tagContract.parseSetTagLog(setTagReceipt.logs[0]);
```
