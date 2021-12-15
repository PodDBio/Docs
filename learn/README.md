# Lite Paper

## Introduction

POD is a database built across various blockchain networks that serves as a common data environment to store on-chain entities’ metadata. 

POD aims to provide separate metadata storage spaces for each on-chain entity, a public space for storing descriptions of various metadata, and a data-centric middleware to ensure the consistency of data from different data environments. 

We hope to establish an infrastructure for blockchain data collaboration and become the engine that drives the growth of the ecosystem.

## Current Landscape

### Background 

The year 2021 marks an era of metaverse. Facebook renamed itself “Meta”, Microsoft and Nvidia also announced their metaverse strategies. 

The metaverse boom triggers the growth of activities in entertainment, social networking, e-commerce, content creation, etc., which brings the increase of blockchain users, on-chain assets, businesses. In the blockchain world, these are represented by on-chain entities. An on-chain entity refers to a specific existence on the blockchain that can be identified via its address or asset ID, such as an address, an NFT or a contract. The data providing information about these entities are their metadata. 

Metaverse participants can include not only humans but also machines like programs, smart contracts and offline machines (e.g., ticket vending machines) Hence, to ensure a smooth data collaboration experience, metadata should be not only human-readable, but also machine-readable. Therefore, it is important to decide how we store, organize, manage and use the metadata.

### Obstacles 

There are a few obstacles that interfere metadata utilization. 



#### No separated storage spaces 

 

1. On-chain entities do not have their own storage space. The smallest unit for on-chain data management is the contract, and the relationship between objects and data in the contract is ambiguous. There is no universal method to relate data to an entity.

2. Metadata are scattered in contracts. It is hard to collect all metadata about a certain entity. 

3. Metadata does not have their own security boundaries. Business logics of a contract decide the publication and modification of metadata.

4. Apps cannot interact directly with metadata freely.

 

#### Hard to distinguish

It is not possible to tell the difference between various metadata of the same entity, what meanings they convey and if there are metadata about one entity can be regarded as in the same metric system. 

 

#### No consistency of metadata 

Consistency refers to the consistency in the method of defining metadata types, interpreting metadata, and interacting with the metadata. It is not possible to guarantee the consistency among different contracts, apps and blockchains.


Due to the above obstacles, metadata are not yet interoperable. Interoperability and collaboration based on metadata is rarely seen, which hinders entity categorization, business division, and industry standardization. We need a new metadata infrastructure to solve these problems.

## POD’s Solution

We believe that we need a third-party, neutral and on-chain metadata database to tackle these obstacles. 



### Metadata storage based on entities 

 POD provides a separate data storage space for each on-chain entity. Using this storage space, POD implements a direct and non-transferable bondage between entities and metadata at the low level, providing a uniform metadata encoding method, clear security boundaries, and a unified way to interact with metadata.

 At the high level, POD provides UGC-level metadata interaction and management tools, which allow anyone to add any metadata description to any object on the blockchain.

For example, it will be low-cost and easy to add stories to NFTs, add identities, and other metadata attributes.



### Metadata Classification 

POD provides a space for binding descriptions to metadata, enabling the ability to define and categorize metadata, through which users can define 3 types of metadata:

 

1. Data categories

   Let’s say we want to define some NFTs as “Spaceships”. We can define metadata with certain properties to have the attribute "spaceship", then give these NFTs the tag so we know these NFTs are in the “spaceship” category.

2. Data attributes 

   For example, we can define metadata with values in a certain range to have the “Flight speed” attribute so all users can align their understanding regarding these values.

3. Data Relationships

   POD allows users to define and establish relationships between entities. For example, the relationship between an address and an NFT avatar with the name of "My Avatar" is established through (Address <-> (My Avatar) <-> NFT). Any dApp can display avatars for users based on this relationship. Users can define similar relationships such as “My Shoes” and “My identity”.

The public space allows users to define, classify, and identify metadata by their meanings and data structures.

Data in one category can be considered as one type of metadata expressing the same meaning. For instance, data in the “Avatar” category represent NFTs with the attribute of “User current avatar”. It is also possible to configure an attribute of “Immutable” for metadata in POD. Through these functionalities, we can classify data and create attributes by any matric for them.

### Cross-platform Consistency

First, POD database is on the blockchain. Metadata classification and access are enabled by deploying contracts on different blockchain networks to ensure data definition methods and low-level data structures are following the same standard, so as to ensure the consistency of metadata using in different contracts and apps. Second, POD offers a POD-Bridge to ensure the interoperability of metadata and the consistency of their classifications. 

## Benefits

 

### For Data Providers

- A new, non-transferable data tagging model
- A public space to define data across all networks
- A public space to access data across all networks
- Increased data reusability and brought by the public space that can prevent repeated data cleansing and storage
- A data toolchain to save costs for development and data interaction
- An infrastructure connecting platforms and networks, offering users more choices, and reducing implementation costs on different chains
- A negotiation platform for data abstraction and standardization throughout the industry

### For Data Consumers

- A public and trusted third-party data collaboration platform
- Metadata sources that are convenient to access and validate
- Accurately identify users and precisely divide on-chain entities 
- The possibility of business division and specialization
- Custom programming empowered by data operation and management based on data classification and aggregation


## Look to the Future

POD enables a layer that stores and classifies on-chain data and their metadata by allowing users to store data as objects. This layer bridges together different contracts and blockchains, hence data defined and stored in POD are available to contracts and users across all networks. As we gradually establish cooperation with more blockchain networks, POD will evolve into an open and trustworthy unified data infrastructure which empowers data communication throughout the blockchain world. It is expected to see more UGC and data interactions. 

Looking ahead, we hope to bring different parties together and establish a collaborative dialogue, so as to inspire the decentralized community to arrive at an agreed data collaboration standard in a bottom-up manner. The standard potentially includes but is not limited to major data categories, major data attributes, data storage solutions for different purposes, transmission interface design, and so on. One the one hand, users can leverage this standard to seamlessly work with data from any network. On the other hand, they still have the freedom to tailor the standard practice according to their own businesses and needs. Thereby, we expect to standardize the metadata environment through POD to drive metadata sorting and on-chain entity categorization to allow for more focused business models, more specialized segmentations, and data standards for high-level businesses to bring about broader, more flexible, and more segmented business collaboration opportunities for the metaverse.