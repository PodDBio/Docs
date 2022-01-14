# 1. Architecture

POD’s two major components are the on-chain contract and the off-chain data indexer. The picture below illustrates how they interact:

![POD architecture](https://raw.githubusercontent.com/PodDBio/Docs/master/pic/pod_inforgraphic.png)

## 1.1. On-chain Contract

The on-chain contract serves as a database where data is stored after being serialized. It provides functions and interfaces to define, store and access data.

## 1.2. Off-chain Data Indexer

Data in on-chain contracts are synchronized to the off-chain indexer. Customer dApps can query all kinds of data through the off-chain data indexer.

# 2. Data Storage Structure

In POD, each single data point is stored in a Tag. You can categorize Tags using a TagClass. At the same time, each Tag is linked to a TagObject to indicate which on-chain entity that the data point is about. The below topics in this section will explain how these elements work together.

## 2.1 TagObject <span id="tagobject"></span>

A TagObject represents an on-chain entity. Any data must be related to a TagObject. This is the structure of a TagObject:

```solidity
enum ObjectType {
    Address,
    NFT,
    TagClass
}

struct TagObject {
    ObjectType Type;
    bytes20 Address;
    uint256 TokenId;
}
```

A TagObject can be about an externally owned account (EOA) on Ethereum, a contract account (CA), or an ERC721 NFT. It is worth noting that it’s also possible to create a TagObject for a TagClass.

1. When you create a TagObject for an EOA address, or a CA address, fill the `Address` field with the corresponding address and fill the `TokenId` field with 0;

2. When you create a TagObject for an ERC721 NFT, fill the `Address` field with the  ERC721 contract address and fill the `TokenId` field with the NFT TokenId;

3. When you create a TagObject for a TagClass, convert the TagClassId to an address and fill the `Address` field, then fill the `TokenId` field with 0.

## 2.2 TagClass & TagClassInfo <span id="tagclass"></span>

A TagClass is equivalent to a table in a relational database. A collection of Tags forms a TagClass.

You can define field names, field types, and write permissions of a TagClass.

TagClassInfo provides additional information to describe a TagClass, such as the Tag name, descriptions, etc.

This is the structure of TagClass and TagClassInfo:

```solidity
struct TagClass {
   bytes18 ClassId;
   uint8 Version;
   address Owner;
   bytes FieldTypes;
   uint8 Flags;
   TagAgent agent;
   address LogicContract;
}

struct TagClassInfo {
   bytes18 ClassId;
   uint8 Version;
   string TagName;
   string FieldNames;
   string Desc;
}
```

| Field | Type | Description |
| ------------- | ------------- |------------- |
| ClassId | `bytes18` | The ID of this TagClass |
| Version |`uint8`| The version of this TagClass|
| Owner |`address`| TagClass Owner's address|
| FieldTypes |`bytes`| Hexadecimal numbers representing field types supported by this TagClass|
| Flags |`uint8`| "0" or "0x80", see details below|
| Agent |`TagAgent`| TagClass Agent's Type and Address |
| LogicContract |`address`| Contract address of logic tagClass |
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

1. `0x0003` represents two values, a `bool` and a `UInt16`; 
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

## 2.3 ValueTag & LogicTag

There are two types of Tags in POD: ValueTag and LogicTag.

**ValueTag**

A ValueTag contains a definite data record which is stored in a POD contract. You can manage a ValueTag using functions `SetTag` and `DeleteTag`.

**LogicTag**

A LogicTag contains a smart contact. The resulting value is not stored in POD, it's returned from the contract defined in the LogicTagClass. The data stored in a LogicTag is also managed by this contract.

The programmability of a LogicTag enables POD to provide an even stronger support. Apart from values, users can also store algorithms that return required values.

What the LogicTag contract need to do is to implement the ILogicTag interface.

```solidity
interface ILogicTag {
    function getLogicTagData(bytes18 classId, TagObject calldata object) external view returns (bytes memory data, bool exist);
}
```

## 2.4 TagAgent  <span id="tagagent"></span>

Only owner has write-permissions to TagClass. But a TagClass Owner can delegate write permissions to a TagAgent without affecting the Owner’s permissions. However, a TagAgent can only create and modify Tags in this TagClass, but cannot modify the TagClass.

There’s no restriction on reading data. Any contract can read any data written in POD. Only data writing is restricted with different levels of permissions.

```solidity
enum AgentType {
  Address, // user address or contract address,
  Tag //address which had this tag
}

struct TagAgent {
  AgentType Type;
  bytes20 Address;
}
```

There are two types of TagAgent:

1.  Address: an EOA, or a CA address;
2.  TagClass: an address (EOA or CA) that owns the Tag in this TagClass.

## 2.5 Tag

A Tag is a data record contained in a TagClass, which is equivalent to a row in a relational database.

```
struct Tag {
    bytes20 TagId;
    uint8 Version;
    bytes18 ClassId;
    uint32 ExpiredAt; 
    bytes Data;
}
```

| Field | Type | Description |
| ------------- | ------------- |------------- |
| TagId | `bytes20` | An ID generated using the TagClass ID and TagObject which this Tag belongs to |
| Version | `uint8` | Version of the Tag |
| ClassId | `bytes18` | ClassId of the TagClass that this Tag belongs to|
| ExpiredAt | `uint32` | A 4-byte value indicating expiration time; or "0" meaning never expires|
| Data | `bytes` | The serialized data point |

# 3. Data serialization  <span id="serialize"></span>
Data must be serialized before it’s stored in a Tag, and the data type must conform to what has been defined in the related TagClass.

**Basic Types**

1. `UInt<M>`: Unsigned integers. “M” can be 8, 16, 32, 64 or 256. The length of the values is fixed, you need to insert leading zeros when there are not enough bits. For example, the Uint32 value represents the number 123 is `0x0000007b`.

2. `Bool`: `True` is represented by the UInt8 value `0x01` (equals to the integer 1). `False` is represented by the UInt8 value `0x00` (equals to the integer 0).

3. `Bytes<M>`: Fixed-length byte arrays. “M” can be 1, 2, 3, 4, 8, 20, 32. You need to insert zeros to the end when there are not enough bits. For example, `0x010203 ` is encoded as `0x0102030000000000`.

4. `Address`: ETH addresses. The encoding is the same as the `bytes20` type.

5. `Bytes`: Variable-length byte arrays. When encoding, you first write an UInt16 value to indicate the length and then write the actual encoded data. For example, `0x01020304` is encoded as `0x000401020304`.

6. `String`: UTF-8 encoded byte arrays. The encoding is the same as the `bytes` type. For example, the encoded string of "Hello,World" is `0x000b48656c6c6f2c576f726c64`.

**Compound Types**

Only array are supported currently. Array nesting is not supported. The encoded result consists of an UInt16 value to indicate the length of the array, and the encoded array itself. For example, the Uint8 array [1,2,3] is encoded as `0x0003010203`.
