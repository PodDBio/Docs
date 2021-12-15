# 1. Architecture

POD’s two major components are the on-chain contract and the off-chain data indexer. The picture below illustrates how they interact:

![PodDB architecture](https://github.com/0xBlueCat/docs-archive/blob/master/imgs/poddb-arch.png?raw=true)

## 1.1. On-chain Contract

The on-chain contract serves as a database where data is stored after being serialized. It provides functions and interfaces to define, store and access data.

## 1.2. Off-chain Data Indexer

Data in on-chain contracts are synchronized to the off-chain indexer. dApps can query all kinds of data through the off-chain data indexer.

# 2. Data Storage Structure

In POD, each single data point is stored in a Tag. You can categorize Tags using a TagClass. At the same time, each Tag is linked to a TagObject to indicate which on-chain entity that the data point is about. The below topics in this section will explain how these elements work together.

## 2.1 TagObject <span id="tagobject"></span>

A TagObject represents an on-chain entity. Any data must be related to a TagObject. This is the structure of a TagObject:

```solidity
struct TagObject {
  address Address;
  uint256 TokenId;
}
```

A TagObject can be about an externally owned account (EOA) on Ethereum, a contract account (CA), or an ERC721 NFT. It is worth noting that it’s also possible to create a TagObject for a TagClass.

1. When you create a TagObject for an EOA address, or a CA address, fill the `Address` field with the corresponding address and fill the `TokenId` field with 0;

2. When you create a TagObject for an ERC721 NFT, fill the `Address` field with the  ERC721 contract address and fill the `TokenId` field with the NFT TokenId. Please note that the TokenId of an NFT cannot be 0, otherwise the TagObject will be regarded as an EOA, or a CA address;

3. When you create a TagObject for a TagClass, convert the TagClassId to an address and fill the `Address` field, then fill the `TokenId` field with 0.

## 2.2 TagClass & TagClassInfo <span id="tagclass"></span>

A TagClass is equivalent to a table in a relational database. A collection of Tags forms a TagClass.

You can define field names, field types, and write permissions of a TagClass.

TagClassInfo provides additional information to describe a TagClass, such as the Tag name, descriptions, etc.

This is the structure of TagClass and TagClassInfo:

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

| Field | Type | Description |
| ------------- | ------------- |------------- |
| ClassId | `bytes20` | The ID of this TagClass |
| Version |`uint8`| The version of this TagClass|
| Owner |`address`| TagClass Owner's address|
| FieldTypes |`bytes`| Hexadecimal numbers representing field types supported by this TagClass|
| Flags |`uint8`| "0" or "0x80", see details below|
| Agent |`TagAgent`| TagClass Agent's Type and Address |
| TagName |`string`| The name of this TagClass |
| FieldNames |`string`| Names of fields allowed to store in this TagClass|
| Desc |`string`| Descriptions of this TagClass|

**Owner**

The TagClass Owner is the address that created the TagClass. By default, the Owner has write-permissions to manage the TagClass, modify or create Tags, and so on. The Owner can transfer permissions to another address.

**FieldTypes**

The data stored in a Tag must conform to the data type defined in the TagClass that this Tag belongs to. POD supports the following data types:

<span id="fieldtype"></span>
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
For example:

1. `0x0103` represents two values, a `bool` and a `UInt16`; 
2. `0x1507` represents an `Int8Array`;

Please note:

1. Arrays must be followed by an underlying data type, and cannot be nested, which means you cannot define a multidimensional array in POD.

2. The `bytes` type and `string` type are to store byte arrays. Because `bytes` and `string` are more commonly used, they are defined separately for ease of processing. The only difference between `bytes` and `string` is that `bytes` is used to represent raw byte arrays and `string` is used to represent human-readable UTF-8 encoded strings.

3. If the type of field is `bytes`, `string` or `array`, you need to add an Uint16 value to indicate the length before this data when encoding it, so that the decoder can parse the data correctly. For example, the encoded `Hello World` string is: `0x000b48656c6c6f20576f726c64`.

4. Since a UInt16 value takes 2 bytes, the length of `bytes`, `string`, and `number` values should not exceed 65535.

5. You can define multiple data types for a TagClass, which means this field can hold multiple values. The number of values of this field, and the number of values in the `FieldNames` should be the same. You don't need to separate these values.

**FieldNames**

You can define multiple field names for a TagClass, which means this field can hold multiple values. The number of values of this field, and the number of values in the `FieldTypes` should be the same.
Please separate these values with commas, for example: "Name,Age".

**Flags**

POD defines TagClass flag values for changing some default actions in POD. Technically there can be up to 8 default flag values. For now only 1 flag has been defined: the Deprecated flag.

Once created, the TagClass cannot be deleted, but it can be deprecated using the Deprecated flag. You can view a deprecated TagClass, and view or delete the Tags of the TagClass. But you cannot create a new Tag and modify existing Tags.

Include `0x80` in this field if you want to deprecate the TagClass, otherwise include 0.

## 2.3 TagAgent  <span id="tagagent"></span>

A TagClass Owner can delegate write permissions to a TagAgent without affecting the Owner’s permissions. However, a TagAgent can only create and modify Tags in this TagClass, but cannot modify the TagClass.

There’s no restriction on reading data. Any contract can read any data written in POD. Only data writing is restricted with different levels of permissions.

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

1.  Address: an EOA, or a CA address;
2.  TagClass: an address (EOA or CA) that owns the Tag in this TagClass.

## 2.4 Tag

A Tag is a data record contained in a TagClass, which is equivalent to a row in a relational database.

```
CODE
```
| Field | Type | Description |
| ------------- | ------------- |------------- |
| TagId | `bytes20` | An ID generated using the TagClass ID and TagObject which this Tag belongs to |
| Version | `uint8` | Version of the Tag |
| ClassId | `bytes20` | ClassId of the TagClass that this Tag belongs to|
| Data | `bytes` | The serialized data point |
| ExpiredAt | `uint32` | A 4-byte value indicating expiration time; or "0" meaning never expires|

**Flags**

When creating a Tag, it’s possible to set a flag variable for the Tag. For now only 1 flag has been defined by POD out of 8: the **Wildcard flag**.

By default, a Tag can be related to only one TagObject, such as an NFT. By using the Wildcard flag you can relate a Tag to multiple TagObjects which represent all NFTs of one single contract. This is particularly helpful when you store data of NFTs generated using a contract factory pattern.

Include `0x01` if you want to set the Wildcard Tag, otherwise include `0`.

# 3. Data serialization  <span id="serialize"></span>

POD offers the `WriteBuffer` and `ReadBuffer` to implement data serialization and deserialization. WriteBuffer is a bytes buffer that can be dynamically expanded.

For example, there is a TagClass containing user identity data. The `FieldNames` is "Id,Name,RegisterTime", and the `FieldTypes` is "bytes32,String,Uint32".

The code for serialization and deserialization is:

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
      .WriteBytes32(id)
      .writeString(name)
      .writeUint32(registerTime); //serialize id //serialize name //serialize registerTime
    Bytes memory data = wBuf.getBytes();
    return data;
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
    return(id, name, registerTime);
  }
}
```
