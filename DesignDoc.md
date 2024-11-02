# Sanctum Design Document

- [Sanctum Design Document](#sanctum-design-document)
  - [Overview](#overview)
  - [Motivation](#motivation)
  - [Inspiration-based token?](#inspiration-based-token)
  - [Requirements](#requirements)
    - [Functional Requirements](#functional-requirements)
    - [Non-Functional Requirements](#non-functional-requirements)
  - [Design Details](#design-details)
    - [High Level Design](#high-level-design)
    - [Data Models and API](#data-models-and-api)
    - [Data Storage](#data-storage)

## Overview
Sanctum is an item gacha system, designed as a companion to FoundryVTT. It will be an individual website, similar to Roll20, where Game Masters (GMs) and Players are able to interact with the overall Sanctum system.

## Motivation
In tabletop RPGs (TTRPGs), progression for playersis tracked in two ways - via character level, and via their equipment. Oftentimes, Players tend to be more “excited” for equipment changes since these don’t happen in an expected manner, unlike levels.

How do we tackle this? Players can already visit the shop during their downtime and pick up new equipment, but in these cases, only standard equipment is available. Most players are more interested in the more unique items, whether that be weapons, armour, or accessories. These are usually given out at the discretion of the GM, which may not be as frequent as Players would like.

How do we provide players access to a variety of unique items? I think we can take some inspiration from Gacha games, which allows players to acquire a random set of items at the cost of some currency. Then, the next question is, what currency should be used? I propose that Players can use Gold (or some standard currency) or an inspiration-based token.

## Inspiration-based token?
In the games where I am the GM, players often do not find too much of a use for inspiration, and here is where I'm proposing an alternative use for inspiration. When players gain inspiration, it is often for things either roleplay or creativity, which is a good way to give players positive feedback.

Instead of just using these inspiration points for re-rolling some bad rolls, why not let them use it for our gacha system? This should create a system where players are more incentivized to roleplay or be more creative with their approach to the game.

<div style="page-break-after: always;"></div>

## Requirements
### Functional Requirements
NOTE: Sanctum has two types of users, Players and GMs. Unless specified, “Users” would refer to both types of users.

- Users should be able to create, login, and manage their accounts
  - Email based and OAuth
- GMs should be able to create a campaign
  - GMs should be able to edit details about the campaign
- Players should be able to join a campaign based on a unique code
- Users should be able to create, edit, view, and delete items and collections
  - 5E SRD will exist as a default collection for any 5E campaign (MVP)
  - GMs should be able to select which collections are available for a given campaign (Campaign settings)
- Users can import and export items and or collections
  - Users can only export for items they have ownership over
- Players should be able to link a player entity to their player character in Foundry
  - When importing from Foundry, Sanctum will import character information
  - When exporting to Foundry, Sanctum will update inventory (items)
  - Sanctum will not edit anything other than inventory (and currency)
- Players should be able to roll for items
  - Rolls are made against collections (banners)
  - Certain collections can be made active or inactive
- Players should be able to trade items with each other
  - Items can only be traded between players in the same campaign

For the purpose of the MVP, 5E will be the only supported game. Future support for PF2E and Lancer is in the timeline, as those are games I have personal interest in.

<div style="page-break-after: always;"></div>

### Non-Functional Requirements
NOTE: The MVP for Sanctum is for one campaign with 4 - 5 users, being my personal D&D campaign.

Scalability: MVP needs to handle 4 - 5 concurrent connections.. However, it should be designed to scale up for outside campaigns and their users.

Performance: MVP needs to have > 1s response time for any of the usual use-cases, minus batch import / export. Every other operation should be performant. Similarly, this needs to scale for future campaigns as well.

Replication: MVP will not have data replication, but will be backed up using AWS S3. However, the data should be designed such that we can easily shard and horizontally scale services and databases.

Consistency: MVP needs to have data states consistent. When a user rolls for some items, there should be no uncertainty regarding what items were rolled and in what quantities.

<div style="page-break-after: always;"></div>

## Design Details
### High Level Design
![Rough High Level Design](/_images/High_LeveL_Design.png)

In this design, we will be using the micro-service architecture, where each service is responsible for a subset of operations. We should be dealing with a relatively read-heavy system, especially with campaigns, items, and collections. The gacha service is an independent service that serves as the "core" of Sanctum, and acts as both a data consumer and producer, ensuring data consistency.

<div style="page-break-after: always;"></div>

### Data Models and API
In this high-level design document, we will not go into specific details about our data models and API, but it is important to talk about them.

At a high-level, we can visualize the following tables and relations:

![Rough Data Models](/_images/Data_Models.png)

- Items and Collections are deliberately kept separate
- Campaigns operate on Collections only
- Users database only keep user and authentication information
- Inventory is split apart from Users to reduce operations performed on the Users database

<div style="page-break-after: always;"></div>

### Data Storage
As noted above, since we will be pursuing strong consistency, I'll be using PostgreSQL as the main form of storage. AWS S3 will be used as Object storage for backups for the MVP and serve static images for items.

Some considerations were made to potentially use a NoSQL database like Cassandra or MongoDB, but:

1. Personally, I wanted to get more experience with SQL and PostgreSQL
2. Strong consistency for item states was defined above and PostgreSQL fits that requirement

<div style="page-break-after: always;"></div>
