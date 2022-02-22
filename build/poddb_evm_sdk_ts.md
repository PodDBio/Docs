# POD EVM TS SDK

This SDK is a tool for developers to interact with POD contracts deployed on EVM-compatible blockchain networks.

For more information about PodDB, see [this doc](https://poddbio.github.io/Docs/#/learn/Technology).

## Installation

Yarn

```shell
yarn add poddb-evm-sdk-ts
```

## Contract Address

**Mainnet**

| Chain | Contract | Address |
| ------------- | ------------- |------------- |
| Polygon|Storage|0xabE843DB2698BE671B79A0e4498ACB1b9B467E3b|
| Polygon|TagClass|0x9b2117F8B4801fac048b14F55162c94C1132A6DF|
| Polygon|Tag|0x3592304e2b3B8d37B0FFB23aA56C7CFF1a2f30C4|

**Testnet**

| Chain | Contract | Address |
| ------------- | ------------- |------------- |
| ETH(Rinkeby)|Storage|0x464C583F6Bc0B3FcB77d7A26534473B18EC6Eb24|
| ETH(Rinkeby)|TagClass|0x65e75e5130A2D95cDD236065C7078CAA711fAa9d|
| ETH(Rinkeby)|Tag|0xECa7E1B937d99d2764C601F34eD1EF58BEE577c4|
| BSC|Storage|0x2eF3d463da85F48F0D8A9A059087C98c31D21Bb6|
| BSC|TagClass|0xD5341cc8B1BC711CEf0802D4Da97e7D341e9B1D2|
| BSC|Tag|0xc126d23e45b7d1cAF93f3c0A2B0aC13b860d1533|
| FTM|Storage|0xE4105E2219d820C43AC3dAC39cBE74beFA54eb5C|
| FTM|TagClass|0xfabBEaB199A69F61b603D09AA9066232f2920bD9|
| FTM|Tag|0x9f1ECa2b257150Ea6F71376e0CECa2548B911Dc3|
| AVAX|Storage|0xBe583C3BB299F4eA1ed5455637E96c01035ecbf5|
| AVAX|TagClass|0x680aAf372d64b0E9c90426Ec53c663DF75d95bFf|
| AVAX|Tag|0x5dB4eC88Ac69F03F0246606C55cb9FbC6c279d61|
| Polygon(Mumbai)|Storage|0xf85893be8DbC59eAa416A5781D1b8C9F49398aa4|
| Polygon(Mumbai)|TagClass|0x6CCAa2796C4c28AA76f36A3bc8A56c7D9b8Fae47|
| Polygon(Mumbai)|Tag|0x1Af4311CE1619b1e3f4921EEb7247673d945C8f0|

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
```

If you use a web browser, you can obtain the provider by sending requests to a wallet plugin such as MetaMask:

```typescript
const provider = new ethers.providers.Web3Provider(window.ethereum);
```

### Connect to Contract

Connect to a POD's tag contract and tagClass contract:

```typescript
const tagContract = await TagContract.getTagContractV1(provider, 'TagContractAddress');
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

## SetTagWithSig

Owner and Agent can directly construct and sign a SetTag transaction. Alternatively, Pod also allows Owner or Agent to sign the data contained in a SetTag, and then users to construct and send the SetTag transaction.
This provides a solution to the situation where only Owner and Agent can construct and send a SetTag transaction. It allows for more flexibility in terms of business logics.

**Example**

```typescript
const tagContract = (await TagContract.getTagContractV1(provider, 'TagContractAddress')).connectSigner(signer);

const tagClassId = 'xxx';
const tagObject = TagObject.fromAddress('address');
const data = new WriteBuffer().writeString('Hello').writeUint8(16).getBytes();

//fetch signature from owner of agent;
const signer = new ethers.Wallet(
    "ownerOrAgent's private", //private key of owner or agent
    provider,
);
const signature = tagContract.getSignatureForSetTag(signer, tagClassId, tagObject, data,{
    expiredTime: 0, //Expiration time of tag in seconds, 0 means never expires
});

const setTagTx = await tagContract.setTag(tagClassId, tagObject, data, {
    expiredTime: 0, //Expiration time of tag in seconds, 0 means never expires
    signature
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
const dataParser =  tag!.parseData(tagClass!.fieldTypes,tagClassInfo!.fieldNames);
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

## DeleteTagWithSig

DeleteTagWith allow to delete a tag via the signature of owner or agent.

**Example**

```typescript
const tagContract = (await TagContract.getTagContractV1(provider, 'TagContractAddress')).connectSigner(signer);

const tagClassId = 'xxx';
const tagObject = TagObject.fromAddress('address');

//fetch signature from owner of agent;
const signer = new ethers.Wallet(
    "ownerOrAgent's private", //private key of owner or agent
    provider,
);
const signature = tagContract.getSignatureForDeleteTag(signer, tagClassId, tagObject);

const deleteTx = await tagContract.deleteTag(tagClassId, tagObject, {
    signature
});
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
