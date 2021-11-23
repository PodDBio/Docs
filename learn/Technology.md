# PodDB

## 1. What is PodDB?

PodDB is a public database on the blockchain, which makes it easy to share data between contracts. At the same time, PodDB is also object-oriented.

All data in the storage pod must belong to an object. This object can be an external account address(EOA), a contract address, or an NFT, or even another TagClass (more about TagClass will be described later).

## 2. What is the mission of PodDB?

PodDB's mission is to be the data layer infrastructure for Web 3.0, empowering contract-to-contract, chain-to-chain collaboration.

## 3. PodDB technology architecture

PodDB mainly contains two components: on-chain contract and off-chain data indexer;

![PodDB architecture](https://github.com/0xBlueCat/docs-archive/blob/master/imgs/poddb-arch.png?raw=true)

### 3.1. Chain contract part

PodDB is an on-chain database where data is serialized and stored in contracts.

Therefore, the on-chain contract provides functions such as data definition, data storage and data external access interface;

### 3.2. Off-chain data indexer

The data of PodDB will be stored off-chain at the same time, and various indexes will be established according to the logical relationship of the data.

dApp can query all kinds of data through the data indexer under the chain.

## 4. The core data structure of PodDB

### 4.1 TagObject

TagObject is the data body object in PodDB, and any data in PodDB must belong to a TagObject.

```solidity
struct TagObject {
  address Address;
  uint256 TokenId;
}

```

A TagObject can be an external address (EOA) in an Eth, a contract address, or an ERC721 NFT. It is worth noting that a TagClass is also a TagObject.

1. If the TagObject is an EOA address, or a contract address, just set the Address field in the TagObject and keep the TokenId at 0;

2. If the TagObject is an ERC721 NFT, you need to set the ERC721 contract address on the Address field and the TokenId of the NFT on the TokenId.

   The main requirement is that the TokenId of the NFT cannot be 0, otherwise the TagObject will be regarded as an EOA, a contract address;

3. If the TagObject is a TagClass, you only need to cast the TagClassId to an address, set the TagObject Address field, and keep the TokenId field at 0;

### 4.2 TagAgent

The TagAgent can proxy the write-permissions of the TagClass Owner.

```solidity
enum AgentType {
  Address, // user address or contract address,
  Tag //address which had this tag
}

struct TagAgent {
  AgentType Type;
  bytes20 Agent;
}

```

There are two types of TagAgent:

1. Address types, including EOA and contract addresses;
2. TagClass type, that is, the address (EOA or contract) that owns the Tag under the TagClass;

### 4.3 TagClass & TagClassInfo

TagClass is equivalent to a table in a relational database, representing a certain type of data.

TagClass defines the field name, the field type, the write-permission and the expired time, etc.

TagClassInfo is some additional description information used to describe TagClass, such as Tag name, description, etc.

```solidity
struct TagClass {
  bytes20 ClassId;
  uint8 Version;
  address Owner;
  bytes FieldTypes;
  uint8 Flags;
  TagAgent agent;
}

struct TagClassInfo {
  bytes20 ClassId;
  uint8 Version;
  string TagName;
  string FieldNames;
  string Desc;
}

```

#### 4.3.1. Field definition

The data in PodDB is structured, and all data must conform to the field definitions in its TagClass.
Currently, the following field types are supported in PodDB:

```solidity
enum TagFieldType {
  Bool, //= 0
  Uint256, //= 1
  Uint8, //= 2
  Uint16, //= 3
  Uint32, //= 4
  Uint64, //= 5
  Int256, //= 6
  Int8, //= 7
  Int16, //= 8
  Int32, //= 9
  Int64, //= 10
  Bytes1, //= 11
  Bytes2, //= 12
  Bytes3, //= 13
  Bytes4, //= 14
  Bytes8, //= 15
  Bytes20, //= 16
  Bytes32, //= 17
  Address, //= 18
  Bytes, //= 19
  String, //= 20
  Array //= 21
}

```

TagClass supports basic types such as Bool, uint256, int256, bytes, string, and Array composite types.

Array types must be followed by an underlying data type when defining array types, and array types cannot be nested, that is, multidimensional array types cannot be defined.

Bytes and String are actually Byte array types. Because Bytes and String are more commonly used, they are specially defined for ease of processing.

There is no essential difference between Bytes and String, where Bytes used to represent bytecode arrays and String is used to represent human-readable UTF-8 encoded strings.

**Example**
`0x0103 represents two fields, the type is Bool type and Uint16 type;`
`0x1507 represents an Int8 array type;`

Note that if a field is of type Bytes, String, and Array, when encoding specific data,
you need to add an Uint16 type of length data in front of the data so that the decoder can correctly parse the data.

`For example, the string "Hello World" needs to be encoded as: 0x000b48656c6c6f20576f726c64`

The length of two bytes indicates that the length of Bytes, String, and number types cannot exceed 65535.

The number of fields defined by FieldTypes in TagClass must correspond to the number of field names defined by FieldNames in TagClaasInfo.

The field names defined by TagFieldNames are separated by commas, such as: "Name,Age".

The data stored in the Pod DB can be parsed according to the FieldTypes definition in its TagClass.

#### 4.3.2. Read and write permissions

There are no restrictions on the read-permissions in PodDB. Any contract can read any data written in PodDB.

PodDB only controls the write-permissions of the data.

In PodDB, each TagClass has an Owner, and only this Owner has permission to write or modify data under the tables defined by the TagClass.

At the same time, the Owner can also modify some definitions of some TagClass.

Owner can set an agent to write. The agent can be an external address(EOA), or a contract, or even another TagClassId, any address that has the Tag under this TagClassId has write-permission.

![PodDB-write-permission](https://github.com/0xBlueCat/docs-archive/blob/master/imgs/poddb-write-auth.png?raw=true)

#### 4.3.3. special flags

The Flags field of the TagClass defines some tags that can change some default behavior of PodDB.

The Flags field is an Uint8 type and can have up to 8 tags. Currently, there is only 1 tags used:

`1.0X01 is the Multi-Issuer flag bit;`

By, or ("or") can be combined with '|' operator.

**Multi-Issue flag**

By default, a TagObject can only have one Tag under a TagClass. For example, TagClass defines a user permission.

If a user (TagObject) owns this Tag, it means that the user has a TagClass-defined permission.

At the same time, the contract can obtain this Tag through TagClassId and TagObject.

However, in some cases, a TagObject needs multiple Tag data under the same TagClass, such as a TagClass that can define a diary, and users can submit multiple diaries under this TagClass.

In this case, you can set the Multi-Issue flag when defining TagClass;

It should be noted that if a TagClass sets the Multi-Issue tag bit, the Tag under this TagClass will not be stored on the chain,

it is also impossible to check whether this TagObject has this Tag in the contract or obtain this Tag in the contract.

Tag with Multi-Issue flag set can only be queried and used off-chain.

### 4.4 Tag

Tag is the data under the definition of TagClass, which is equivalent to rows in a relational database.

```solidity
struct Tag {
  bytes20 TagId;
  uint8 Version;
  bytes20 ClassId;
  bytes Data;
  uint32 ExpiredAt;
}

```

TagId is the unique flag of the Tag, generated by the TagClass ID and TagObject.

ClassId is TagClassId;

Data is the data actually stored by Tag, and the encoding of the data conforms to the specification defined by the FieldTypes field in TagClass;

ExpiredAt is the expired time of the data in second. Zero means never expires.

Note that the modifier and deleter of the Tag must have the data write permissions defined by the TagClass.

#### 4.4.1 Expired time

When creating a Tag, you can set an expiration time in seconds. If it is not updated before it expires, the Tag will automatically expire.

ExpiredTime is a time value in seconds and has a length of 4 bytes. If the ExpiredTime is 0, it means will never expire.

#### 4.4.2 Wildcard

You can use wildcard flag to create the same tag for all NFTs under a contract instead of creating them individually, which is very friendly for some kind of factory NFT contract.

## 5. The core interface of PodDB

```solidity
interface PodDB {
  //crate a new TagClass
  function newTagClass(
    string calldata tagName,
    string calldata fieldNames,
    bytes calldata fieldTypes,
    string calldata desc,
    uint8 flags,
    TagAgent calldata agent
  ) external returns (bytes20);

  //set tag
  //flags 1:wildcard
  function setTag(
    bytes20 tagClassId,
    TagObject calldata object,
    bytes calldata data,
    uint32 expiredTime,
    uint8 flags
  ) external returns (bytes20);

  //get tags via TagId
  function getTagById(bytes20 tagId)
    external
    view
    returns (Tag memory tag, bool valid);

  //get tags via TagObject
  function getTagByObject(bytes20 tagClassId, TagObject calldata object)
    external
    view
    returns (Tag memory tag, bool valid);

  //check whether TagObject has a tag of TagClass
  function hasTag(bytes20 tagClassId, TagObject calldata object)
    external
    view
    returns (bool valid);
}

```

## 6. Data serialization

PodDB provides two Buffers to implement data serialization and deserialization operations, namely WriteBuffer and ReadBuffer.

For example, there is a TagClass that defines a certain user identity.

The field name is "Id,Name,RegisterTime", and the field type is "bytes32,String,Uint32".

The serialization and deserialization codes are as follows:

```solidity
contract Serialization {
  using WriteBuffer for WriteBuffer.buffer;
  using ReadBuffer for ReadBuffer.buffer;

  function serialize(
    bytes32 id,
    string calldata name,
    uint32 registerTime
  ) public pure returns (bytes memory) {
    WriteBuffer.buffer memory wBuf;
    wBuf
    .Init(32 + 4 + bytes(name).length) //init buffer capacity
    .WriteBytes32(id) //serialize id
    .writeString(name).writeUint32(registerTime); //serialize name //serialize registerTime
    Bytes memory data = wBuf.getBytes();
    Return data;
  }

  function deserialize(bytes calldata data)
    public
    pure
    returns (
      bytes32 id,
      string memory name,
      uint32 registerTime
    )
  {
    ReadBuffer.buffer memory rBuf = ReadBuffer.fromBytes(data);
    id = rBuf.readBytes32(); //deserialize id;
    name = rBuf.readString(); //deserialize name;
    registerTime = rBuf.readUint32(); //deserialize registerTime;
    Return(id, name, registerTime);
  }
}

```

## 7. What value can PodDB bring?

### 7.1 Data Abstraction

PodDB can provide a kind of data abstraction through TagClass.

It can verify whether TagObject owns this Tag to pass certain functions, such as operation rights, user rights, etc.;

### 7.2 Separation of Computing and Storage

Using PodDB, you can define a TagClass in one contract, generate a Tag, and then read the Tag in another contract.

This kind of putting changeable data generation logic in one contract and stable data reading logic in another contract can make the upgrade of the contract simpler and more adaptable to external changes.

### 7.3. User-generated data

With PodDB's client tools, users can generate and manage some kind of structured contract data without contract programming, which provides great convenience.
