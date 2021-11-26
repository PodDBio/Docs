# Lite Paper

## What is PodDB
POD is short for Public On-chain Database, it is a tool of the data infrastructure, which allows easy sharing of data between smart contracts. The goal of POD is to be the on-chain data collaboration infrastructure and promote the development of the data ecology.

## Issues we found
We found several problems in data interaction between smart contracts

### Hard to find
There is no unified entry point for real-time access to data on the chain. At present, data is fragmented, they are scattered in different contracts and chains, there is no index, it is difficult to find, associate, which makes it hard to collaborate on data.

### Hard to store
For data service providers, smart contracts need to be developed, and even knowledge of different chains needs to be learned in order to provide multi-chain data services. This should not be the entry cost of the data source.
For data collaboration, building a multilateral and credible data environment is the cost of collaboration.

### Hard to collaboration
The current data environment is complex, such as store and query interfaces, data encoding methods, permission control methods, notification methods, etc., which will bring application costs to adopters.

## Vision and Structure
In response to the above problems, we believe that the best product form is a native, neutral third-party database on the chain. Aggregate data, integrate storage systems, and provide UGC(User Generated Content)-level tools. Let POD users can find the data they want, make data storage simpler and cheaper, and make collaboration convenient and easy to adopt.

The mission of POD is to be the infrastructure for data collaboration on the chain.

POD mainly provides two functions.

- [Data storage]User or smart contract can storage/query/manage data on POD. 
- [Data definition]User or smart contract can define/classify/verify/manage a kind of data on POD.

### Where is POD

#### By Layer

Compare with the other layers

[**TheGraph**] On-chain data aggregation services to off-chain applications

[**ChainLink**] Off-chain data aggregation services to on-chain contracts

[**POD**] On-chain data aggregation services to on-chain contracts

Through the POD layer, the unification of data definitions between different chains can be realized. 

![layer](https://raw.githubusercontent.com/PodDBio/Docs/master/pic/layer.jpg)

#### By data eco system

POD is a layer is between the object layer and the application layer. It is like a configuration file, provides a universal data manage method for the entire encryption ecology. It can allow any user or smart contract to add any data or description to any object on the chain, and to query. This will bring cross-application and unified, accurate identification, business segmentation, and abstract layer collaboration capabilities to upper-level applications.

![eco_structure](https://raw.githubusercontent.com/PodDBio/Docs/master/pic/eco_structure.png)



## Features
### Database
POD is a brand new blockchain database model designed according to the characteristics of the blockchain. Through the POD database, users can define datasets, manage datasets, publish data, update data, query data, etc.

### Object-oriented
The data in pod must belong to an object. This object could be an Externally Owned Account(EOA), a smart contract, or an NFT, or any only thing on the chain. By this, the associated data of the object can also be retrieved through the POD.

### Neutral Third-party Platform
POD is a community-driven organization. POD core only builds the infrastructure and tool chain for POD ecosystem while having no control and no effect over the data.
- Permissionless: 
  + [Write]Anyone or any smart contract can define or publish any data of any on-chain object.
  + [Read]Anyone or any smart contract can reference any data of any on-chain object.
- Customizable permission management: 
  + When user define a certain type of data, The dataset owner can customize the data management rules. Such as who can issue, when can issue, how to issue, whether can modify, who can modify, how to modify etc., If necessary, dataset owner also can set the rules to be editable. But all of these are public, verifiable, and traceable on the chain.

## Benefits

### Aggregate Data and Categorized

Users can define a dataset on the POD

- The data service can publish data to this dataset.

- Upstream applications can find the dataset in the POD category by the name of the dataset.

### Simpler and cheaper

- Data tool chains
  POD will provide various tool chains to support the entire life cycle of data. Providing data services or using data no longer requires professional smart contract or chain knowledge.

- Multi-chain and unified data interface

  Through the abstraction and interface unification of the underlying chain through POD, users no longer need to care about the technical details of the underlying chain but pay more attention to their own business.

- More flexible data management methods
  POD allows any data to be bound to any object at any time. Through the isolation of third-party databases, data design patterns and management methods are more flexible. Data changes only affect the database, without asset security.

### Easy to collaboration

- Through the standardization of the data environment, such as interactive interfaces, encoding methods, data schema definition and storage methods, permission management methods, notification methods, etc., data adopters obtain data and use data in the same way, and further can negotiate data standards.
- As an open source, transparent, and Trusted third-party database, collaborative data can be safely placed, and the security boundary will be very clear. Through this physical isolation, data changes only affect the content of the database, with no asset security.
