# PodDB-EVM-SDK-TS

PodDB-EVM-SDK-TS is development tools for PodDB that deployed in EVM-compatible blockchain.

Adapt to PodDB Version: 2

## Installation

Yarn

```shell
yarn add poddb-evm-sdk-ts
```

## Contract Address
```typescript
const EthRinkebyPodDBContractAddress = '0x702A6B19F15c754052AfB1166b44Fa04265EB140';
const BSCTestnetPodDBContractAddress = '0x0039b621d28B83141744c86457AC75A844f7F139';
const AVAXTestnetPodDBContractAddress = '0xDAD5901E4Ff49b22daFa2cDcC27D6bc595D51DDA';
const FTMTestnetPodDBContractAddress = '0x50B0610AF513D361390F61986E023c881Be2c21D';
```

## Usage

### Connect PodDB contract.

Connect PodDB contract needs an ethereum provider object. In nodejs environment, you can get by connecting a JsonRpc node:

```typescript
const provider = new ethers.providers.JsonRpcProvider(
'http://127.0.0.1:8545', //ethereum node rpc url
);
```

For some chains, such as BSC, AVAX, etc., the official RPC URL is built in:

```typescript
const BSCMainnetDefaultRpcUrl = 'https://bsc-dataseed1.binance.org/';
const BSCTestnetDefaultRpcUrl = 'https://data-seed-prebsc-1-s1.binance.org:8545/';
const AVAXMainnetDefaultRpcUrl = 'https://api.avax.network/ext/bc/C/rpc';
const AVAXTestnetDefaultRpcUrl = 'https://api.avax-test.network/ext/bc/C/rpc';
const FTMMainnetDefaultRpcUrl = 'https://rpcapi.fantom.network';
const FTMTestnetDefaultRpcUrl = 'https://rpc.testnet.fantom.network/';
```

In the browser environment, you can obtain by requesting wallet plugins such as MetaMask:

```typescript
const provider = new ethers.providers.Web3Provider(window.ethereum);
```

Connect PodDB contract code:

```typescript
const podDBContract = await PodDBContract.getPodDBContractV2(provider, 'PodDBContractAddress');
```

If you need to send transactions, such as creating TagClass, SetTag, etc., you also need to link a signer to sign the transaction.
In the nodejs environment, it can be obtained directly through the private key.

```typescript
const signer = new ethers.Wallet(
'0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80', //private key
provider,
);
```

In the browser environment, you can obtain by requesting wallet plugins such as MetaMask:

```typescript
const provider = new ethers.providers.Web3Provider(window.ethereum);
await window.ethereum.request({ method: 'eth_requestAccounts' });
const signer = provider.getSigner();
```

PodDBContract connect signer code:

```typescript
podDBContract.connectSigner(signer);
```

### Create TagClass
#### Field define

The field definition in TagClass consists of two parts: FieldTypes and FieldName.

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

Fields in TagClass currently support the above types.
Array types are supported (not nested), but structure types are not supported.

FieldTypes and FieldNames can be empty if TagClass does not need to store data.

You can use the TagClassFieldBuilder class to define field types.

To create a TagClass example with 2 fields:

```typescript
const fieldBuilder = new TagClassFieldBuilder();
fieldBuilder.put("f1", TagFieldType.String);
fieldBuilder.put("f2", TagFieldType.Uint32);
const fieldTypes = fieldBuilder.getFieldTypes();
const fieldNames = fieldBuilder.getFieldNames();
```

#### Set Agent
Only the owner of TagClass has write-permission，but owner can set an agent to proxy the owner's write-permission.

Agent consists of AgentType and String.

```typescript
export enum AgentType {
    Address,
    Tag,
}
```

When the AgentType is an Address, the Agent is an address, externally owned address (EOA), or contract address (CA);

When the AgentType is a Tag, the Agent is a TagClass, any address (EOA, or CA) that has the Tag under this TagClass has write-permission.

Defined Agents can use the TagAgentBuilder class.

```typescript
const agent =
new TagAgentBuilder(
        AgentType.Address,
        "Address",
    ).build();
```

Create TagClass sample code

```typescript
const podDBContract = (await PodDBContract.getPodDBContractV2(provider, 'PodDBContractAddress')).connectSigner(signer);
//build tagClassField
const fieldBuilder = new TagClassFieldBuilder();
//Put field number, as you need
fieldBuilder.put('field1', TagFieldType.String);
fieldBuilder.put('field2', TagFieldType.Uint8);

const newTagClassTx = await podDBContract.newTagClass(
    'TagClassName',
    fieldBuilder.getFieldNames(),
    fieldBuilder.getFieldTypes(),
    'TagClassDesc',
);
newTagClassTx.wait(); //Waiting for transaction confirmation
```

### SetTag

The basic unit of data in PodDB is Tag, and the operation of writing data is called SetTag.

If the same TagClass is repeatedly written to the same user, the following will overwrite the previous data.

#### Create TagObject
PodDB is an object database. All data has an object, which is called TagObject.

TagObject consists of two parts: Address and TokenId. Objects that can be described include address (EOA, or CA), and TagClass.

TagObject can be built by the buildTagObject function.

Example:
```typescript
const nftObject = buildTagObject("ContractAddress", 1);
const addrObject = buildTagObject("Address");
```

#### Build TagData
The data written to PodDB must be serialized and conform to the field definitions in TagClass.

You can use  WriteBuffer to build the data, according the field types defined inTagClass.

Example:
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

#### Using TagFlags

TagFlags are special flags that can be used to change some default behavior.

TagFlags is an Uint8 number that can have up to 8 flags. Currently, only the 1 flag with the value 1 is used: Wildcard Flag.

**Wildcard Flag**

Wildcard Flag is a wildcard flag. SetTag creates a Tag that by default belongs to only one TagObject.

For NFT objects, you can use Wildcard Flag if you want to create a Tag for all NFTs under a contract.

The TagObject at this time is also called Wildcard Object.

You can construct WildcardFlag using TagClassFlagsBuilder.

Example:
```typescript
const tagFlags = new TagFlagsBuilder().setWildcardFlag().build();
```

#### ExpiredTime
When SetTag, you can set an expired time for Tag. This expired time is an Uint32 number in seconds.

If it is 0, it means it will never expire. If it is not 0, then the actual expired time is the block time when the transaction is sent and add the expiration time value.

SetTag sample code

```typescript
const podDBContract = (await PodDBContract.getPodDBContractV2(provider, 'PodDBContractAddress')).connectSigner(signer);

const tagClassId = 'xxx';
const tagObject = buildTagObject('address');
const data = new WriteBuffer().writeString('Hello').writeUint8(16).getBytes();
const setTagTx = await podDBContract.setTag(tagClassId, tagObject, data);

setTagTx.wait();
````

### GetTag
Tag data can be read through hasTag, getTagData, and getTagByObject.

#### HasTag
HasTag is used to check whether a TagObject has a Tag under a specific TagClass, regardless of what the data in the Tag is.

If the TagObject has no Tag, or Tag and expired, hasTag returns false.

HasTag is commonly used in access systems to check whether a user has some kind of permission.

#### getTagData
getTagData is used to obtain Tag data. If the user does not have this Tag, or this Tag expires, an empty byte array is returned.

#### getTagByObject
Compared with getTagData, getTagByObject can obtain more metadata about Tags than getTagData, such as expiration time.

#### Parse TagData
TagData is serialized data. To parse the fields in TagData, you need to know the field type definition of this TagClass.

Parsing TagData can use the TagDataParser.

GetTag sample code
```typescript
const podDBContract = (await PodDBContract.getPodDBContractV2(provider, 'PodDBContractAddress')).connectSigner(signer);

//get tag
const tag = await podDBContract.getTagByObject(tagClassId, tagObject);
//get tagClass
const tagClass = await podDBContract.getTagClass(tagClassId);
//get tagClassInfo
const tagClassInfo = await podDBContract.getTagClassInfo(tagClassId);

//parse data
const dataParser = new TagDataParser(tagClassInfo!.FieldNames, tagClass!.FieldTypes, tag!.Data);
const field1 = dataParser.get('field1')!.getString(); //or get other types
const field2 = dataParser.get('field2')!.getNumber(); //or get other types
```

### Delete Tag
The created Tag can be deleted. When you delete a Tag, you need to specify TagId, TagClassId, and TagObject.

Delete Tag sample code:

```typescript
const podDBContract = (await PodDBContract.getPodDBContractV2(provider, 'PodDBContractAddress')).connectSigner(signer);

const tagId = 'xxx';
const tagClassId = 'xxx';
const tagObject = buildTagObject('address');
const deleteTx = await podDBContract.deleteTag(tagId, tagClassId, tagObject);
deleteTx.wait();
````

### Update TagClass
After TagClass is created, everything can be modified except the field type and field name.

The TagClass Name and Desc can be modified through the UpdateTagClassInfo function.

The Owner, Agent, and TagClassFlags can be modified through the UpdateTagClass function.

Example:

```typescript
const podDBContract = (await PodDBContract.getPodDBContractV2(provider, 'PodDBContractAddress')).connectSigner(signer);

const tagClassId = 'xxx';
const newOwner = 'xxx';
const newAgent = new TagAgentBuilder(AgentType.Address, 'agentAddress').build();
const updateTagClassTx = await podDBContract.updateTagClass(tagClassId, newOwner, newAgent, DefaultTagClassFlags);
updateTagClassTx.wait();

const newName = 'newName';
const newDesc = 'newDesc';
const updateTagClassInfoTx = await podDBContract.updateTagClassInfo(tagClassId, newName, newDesc);
updateTagClassInfoTx.wait();
```

### Deprecated TagClass

The created TagClass cannot be deleted, but can be Deprecated.

The Tag of deprecated TagClass can be viewed and deleted, but you cannot create a new Tag and modify the old Tag data.

#### TagClassFlags

TagClassFlags are special flags that can change the default behavior of TagClass.

TagClassFlags is an Uint8 number that can represent up to 8 flags.

Currently, used is the value of 128 tag: Deprecated Flag.

The TagClass with deprecated Flag is deprecated TagClass.

TagClassFlags can be constructed through the TagClassFlagsBuilder class.

Example:

```typescript
const tagClassFlags = new TagClassFlagsBuilder().setDeprecatedFlag().build();
```

Deprecated TagClass sample code:

```typescript
const podDBContract = (await PodDBContract.getPodDBContractV2(provider, 'PodDBContractAddress')).connectSigner(signer);

const tagClassId = 'xxx';
let tagClass = await podDBContract.getTagClass(tagClassId);
tagClass!.Flags = new TagClassFlagsBuilder(tagClass?.Flags).setDeprecatedFlag().build();

const updateTagClassTx = await podDBContract.updateTagClass(
    tagClassId,
    tagClass!.Owner,
    tagClass!.Agent,
    tagClass!.Flags,
);
updateTagClassTx.wait();
```

### Parse Log

```typescript
const podDBContract = (await PodDBContract.getPodDBContractV2(provider, 'PodDBContractAddress')).connectSigner(signer); 

const tagClassId = 'xxx';
const tagObject = buildTagObject('address');
const data = new WriteBuffer().writeString('Hello').writeUint8(16).getBytes();
const setTagTx = await podDBContract.setTag(tagClassId, tagObject, data, {
    expiredTime: 0, //Expiration time of tag in seconds, 0 means never expires
    flags: 0, //tag Fags: 1 represents a wildcard, and the NFT sent under the target contract will have the Tag
});

setTagTx.wait();

const setTagReceipt = await provider.getTransactionReceipt(setTagTx.hash);
//parseSetTagLog
const setTagLog = await podDBContract.parseSetTagLog(setTagReceipt.logs[0]);

console.log(JSON.stringify(setTagLog, undefined, 2));
```
