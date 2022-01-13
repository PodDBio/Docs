# POD: The Web3 Infrastructure for Data Collaboration

## Introduction

POD is an on-chain data middleware built across various blockchain networks that aims to create a secure and trustworthy third-party data collaboration platform.

In this paper, we review the current situation of on-chain data usage and explain how POD can revolutionize data collaboration in the Metaverse.

## Current Landscape

### Background 

The year 2021 marks an era of metaverse. Facebook renamed itself “Meta”, Microsoft and Nvidia also announced their metaverse strategies.

The metaverse boom triggers the growth of activities in entertainment, social networking, e-commerce, content creation, etc., which brings the increase of blockchain users, on-chain assets and businesses. In the blockchain world, these are represented by on-chain entities. An on-chain entity refers to a specific existence on the blockchain that can be identified via its address or asset ID, such as an NFT or a contract. The data providing information about these entities are their metadata.

Data consumers in the metaverse can include not only humans but also programs, smart contracts, and physical machines (e.g., ticket vending machines). To ensure a smooth data collaboration experience, metadata should be structured and relational, so they are easy to be searched and used by both humans and software. 

Therefore, the time has come to design a new approach to store, organize, manage, and use on-chain metadata.

### Obstacles 

There are a few obstacles that interfere data collaboration on the level of metadata. 

#### Hard to Find

On-chain data are scattered in contracts on different networks. It is not yet possible for contracts to obtain all data from one single common data environment. To access a data element, a system must establish peer-to-peer communication directly with the contract that stores it.

#### Hard to Relate

Due to the non-relational storage, it is hard to trace the relationship between two entities or associate entities with their metadata. As a result, it is a challenge to classify and make sense out of on-chain metadata.

#### Hard to Utilize

Each network or app is unique in the way it transmits data, which means its APIs, data encoding methods, and access control may not be compatible with those of others. Thus, the process of data management and usage is not yet automated. Data from diverse sources must be pre-processed before they can work in tandem.

## POD’s Solution

As a neutral third-party data middleware, POD will tackle these obstacles by providing the key functionalities listed below and realize the full potential of metadata. 

### I. Entity-based Metadata Storage

In POD, a metadata element is stored in a tag, which must be associated with an on-chain entity. Such design manifests the relationship between the entity and the metadata, and allows the relationship to be non-transferable. When users look for certain metadata of an entity, what they need to do is to search for the tag and entity pair by the tag name and the entity address. which will facilitate the verification and acquisition of the entity data.

In addition,  the metadata in POD can be in the form of values or logic. Values are definite, like tags, certifications, descriptions, etc. Logic refers to programming logic that can retrieve a value or store a value based on a given condition. Examples include an algorithm that calculates real-time reputation scores based on user status data; and an NFT with lottery scripts for NFT components. 

![elf-storage](https://raw.githubusercontent.com/PodDBio/Docs/master/pic/self-storage.png)

Due to this design, any on-chain entity can acquire two new abilities.
1. Entity-based, infinite expandability

   Entity will no longer be limited to the initial business of the issuer, and logic can be added by binding new data on the POD. POD allows any number of tags to be bound to any on-chain entity, which can bring infinite evolutionary possibilities to the entity. This is especially useful where characters often have default initial stats and players can upgrade their abilities and weapons later. In theory, characters can evolve infinitely, and PODs can support endless games.

2. Entity-based, co-maintaining metadata

   POD is entity-centric, allowing people to jointly maintain (read, write, manage) the metadata of the entity, and form the possibility of multi-party cooperation around the entity and its specific attributes. For example, the wear rating of the item, production information, insurance information, brand information, combination information, etc. 

Through POD, we can make each entity to be a rich entity, which can let entities have unlimited value possibilities.

![elf-storage](https://raw.githubusercontent.com/PodDBio/Docs/master/pic/richNFT.png)

### II. Metadata Type Definition

The metadata type includes a set of rules for how to store certain metadata elements and their descriptions. It plays a key role in bringing together data from various sources and structuring them to support desired business logics. It helps users to query, verify, use and manage data like an index.

The metadata type makes it possible to define on-chain entities and store metadata from below three aspects. 

1. Species

   Users can define and classify a class of entities or concepts by giving them a species name and marking them on the chain. That is, through POD, user can easily realize the definition and classification of on-chain entities or concepts on the chain. The species can be verified , reused by anyone, and can be edited and managed according to user-defined conditions.

   For example, to addresses, Nike fan tags can be defined and bound to user addresses; Level tags can be defined and bound to DAO users. To NFTs, shoes kind, car kind, monster kind can be bounded. 

   ![catlogy](https://raw.githubusercontent.com/PodDBio/Docs/master/pic/catlogy.png)

2. Attribute

   Like mentioned in the above topic, a metadata type specifies what attributes must be included to describe a certain type of entity. For example, you can define that the metadata of spaceship NFTs must include values representing the “Flight speed” attribute. It is also possible to set an attribute to be immutable by configuring the modification permission. 

   ![attribute value](https://raw.githubusercontent.com/PodDBio/Docs/master/pic/attribute%20value.png)

3. Relationship

   It is possible to demonstrate relationships between entities through metadata types. For example, you can indicate the avatar NFT you are currently using as your avatar through the metadata type “My Current Avatar” which accordingly links your address with the NFT address. Any dApp can display your avatar based on this relationship. Similarly, you can define a "Current Logo" type for demonstrating the relationship between a logo NFT and an entity; you can use the "Current Identity Provider" type to indicate the identity service you are using; and a “Current Clothes” type can be used to specify the clothes for an NFT avatar. The functionality of defining relationships allows on-chain entities to be composable, entities from different classes can be combined into one scenario.

   ![relationship](https://raw.githubusercontent.com/PodDBio/Docs/master/pic/relationship.png)

### III. Data Conformity Across Platforms

The POD database consists of a series of contracts deployed on different blockchain networks. Such design ensures a consistent data definition and a unified data structure at the low level, so data within each network can reach conformity. At the same time, these contracts are linked together by the POD-Bridge, so one metadata type can be applied to data from different chains. In this way, data conformity will be realized across all networks, which is the prerequisite for users to work with UGC (User Generated Content) level metadata using the toolchain provided by POD.

![structure](https://raw.githubusercontent.com/PodDBio/Docs/master/pic/structure.png)

To sum up, POD enables more effective data collaboration based on a unified understanding of the same data through the middleware where users can define and classify metadata. 

## Functionality List

Entity-based Metadata Storage

- Verify entity status by checking its class, attributes, and relationships
- Bind non-transferable metadata to entities 
- Associate metadata in form of values or logic
- Empower NFT with infinite expandability
- Empower NFT with metadata co-maintaining ability

Metadata Types applied to all networks

- Define entity classes and classify entities; achieve batch management and programmability based on the classes
- Define attributes and bind attributes to entities 
- Define and establish relationships between entities 

Metadata Conformity Across Platforms

- Ensure data conformity across all systems in terms of definitions and formats
- Toolchain supporting multiple networks and UGC level metadata

## Look to the Future

POD brings together data from different contracts and blockchains, data defined and stored in POD are immediately actionable for contracts and users across all networks. As we gradually establish cooperation with more blockchain networks, POD will evolve into an open and trustworthy data infrastructure which empowers data communication in Web3. 

Looking ahead, we hope to bring different parties together and establish a collaborative dialogue to inspire the decentralized community to arrive at an agreed data collaboration standard in a bottom-up manner. The standard potentially includes but is not limited to major entity classes, frequently used metadata types, data storage solutions for different purposes, transmission interface design, and so on. One the one hand, users can leverage this standard to seamlessly work with data from any network. On the other hand, they still have the freedom to tailor the standard practice according to their own businesses and needs. 

Thereby, POD will propel the data collaboration ecosystem to develop, which will enable focused business models, precise user segmentation and standardized data format at high level. We envision more business opportunities and more flexible collaboration in the metaverse will be unlocked.
