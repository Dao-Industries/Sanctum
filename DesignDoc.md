# Sanctum Design Document
## Table of Contents
- [Sanctum Design Document](#sanctum-design-document)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
    - [Goals](#goals)
    - [Motivation](#motivation)
      - [Inspiration-based token?](#inspiration-based-token)
    - [Definitions](#definitions)
  - [Requirements](#requirements)
    - [Functional Requirements](#functional-requirements)
    - [User Stories](#user-stories)
      - [User Account Management](#user-account-management)
      - [Campaign Management](#campaign-management)
      - [Item and Collection Management](#item-and-collection-management)
      - [Foundry Integration](#foundry-integration)
      - [Gacha (Item Rolling)](#gacha-item-rolling)
    - [Use Cases](#use-cases)
      - [User Account Management](#user-account-management-1)
      - [Campaign Management](#campaign-management-1)
      - [Item and Collection Management](#item-and-collection-management-1)
      - [Foundry Integration](#foundry-integration-1)
      - [Gacha (Item Rolling)](#gacha-item-rolling-1)
    - [Non-Functional Requirements](#non-functional-requirements)
  - [Design Details](#design-details)
    - [High Level Design](#high-level-design)
    - [Data Models and API](#data-models-and-api)
    - [Data Storage](#data-storage)
  - [User Interfaces](#user-interfaces)
    - [User Account Management](#user-account-management-2)
    - [Campaign Management](#campaign-management-2)
    - [Item and Collection Management](#item-and-collection-management-2)
    - [Foundry Integration](#foundry-integration-2)
    - [Gacha (Item Rolling)](#gacha-item-rolling-2)
  - [Testing](#testing)
    - [Unit Tests](#unit-tests)
    - [Functional tests](#functional-tests)
  - [Deployment](#deployment)
  - [Maintenance](#maintenance)

## Overview
### Goals
Sanctum aims to be an all-in-one service for Game Masters (GMs) to manage and distribute homebrew items to their players. It is not my goal to create a second [5e.tools](https://5e.tools), or to create a 1:1 mapping of their DM tools for loot. The goal of Sanctum is twofold, a modern homebrew item manager and its corresponding gacha system.

### Motivation
In tabletop RPGs (TTRPGs), progression for players is typically tracked in two ways - via character level, and via their equipment. Oftentimes, players tend to be more “excited” for equipment changes since these don’t happen in an expected manner, unlike levels.

How do we tackle this? Players can already visit the shop during their downtime and pick up new equipment, but in these cases, only standard equipment is available. Most players are more interested in the more unique items, whether that be weapons, armour, or accessories. These are usually given out at the discretion of the GM, which may not be as frequent as players would like.

How do we provide players access to a variety of unique items? I think we can take some inspiration from Gacha games, which allows players to acquire a random set of items at the cost of some currency. Then, the next question is, what currency should be used? I propose that Players can use Gold (or some standard currency) or an inspiration-based token.

#### Inspiration-based token?
In the games where I am the GM, players often do not find too much of a use for inspiration, and here is where I'm proposing an alternative use for inspiration. When players gain inspiration, it is often for things either roleplay or creativity, which is a good way to give players positive feedback.

Instead of just using these inspiration points for re-rolling some bad rolls, why not let them use it for our gacha system? This should create a system where players are more incentivized to roleplay or be more creative with their approach to the game.

### Definitions
Below is a list of words and their definitions. While not comprehensive, they should serve as a starting point to understand some of the context throughout the rest of this document.

| Word | Abbreviation | Definition |
| ---- | ------------ | ---------- |
| Game Master | GM | ... |
| Player | Player | ... |
| Inspiration | Inspiration | ... |
| 5th Edition System Reference Document | 5E SRD | ... |
| Items | Items | ... |
| Collections | Collections | ... |
| Gacha | Gacha | ... |
| Roll | Roll | ... |
| Banner | Banner | ... |

## Requirements
### Functional Requirements
NOTE: Sanctum has two types of users, Players and GMs. Unless specified, “Users” would refer to both types of users.

- Users should be able to create, login, and manage their accounts
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

### User Stories
#### User Account Management
- As a user, I want to create an account using my email to use Sanctum
- As a user, I want to log into my account to use Sanctum
- As a user, I want to reset my password because I forgot my password
- As a user, I want to manage my account details so I can update my information
- As a user, I want to have the ability to delete my account at any given time

#### Campaign Management
- As a GM, I want to create a new campaign so I can organize and manage my games
- As a GM, I want to edit the details about my campaign so that the details are correct as the campaign progresses
- As a GM, I want to enable or disable specific collections for any given campaign so that each campaign only has items relevant to its own setting
- As a player, I want to join a campaign using an unique code so I can start using Sanctum and rolling for items

#### Item and Collection Management
- As a user, I want to create, edit, view, and delete items and collections so that I can manage my inventory
- As a user, I want to export items and collections to JSON so I can share or backup my data
- As a user, I want to import items and collections from JSON so I can add new items
- As a player, I want to trade items with other players within the same campaign

#### Foundry Integration
- As a player, I want to link my player entity to my character in FoundryVTT so that game data is synchronized
- As a player, I want Sanctum to import and update inventory from FoundryVTT without changing other details

#### Gacha (Item Rolling)
- As a player, I want to roll for items from the available banners

### Use Cases
#### User Account Management

#### Campaign Management

#### Item and Collection Management

#### Foundry Integration

#### Gacha (Item Rolling)

### Non-Functional Requirements
NOTE: The MVP for Sanctum is for one campaign with 4 - 5 users, being my personal D&D campaign.

Scalability: MVP needs to handle 4 - 5 concurrent connections.. However, it should be designed to scale up for outside campaigns and their users.

Performance: MVP needs to have > 1s response time for any of the usual use-cases, minus batch import / export. Every other operation should be performant. Similarly, this needs to scale for future campaigns as well.

Replication: MVP will not have data replication, but will be backed up using AWS S3. However, the data should be designed such that we can easily shard and horizontally scale services and databases.

Consistency: MVP needs to have data states consistent. When a user rolls for some items, there should be no uncertainty regarding what items were rolled and in what quantities.

## Design Details
### High Level Design
![Rough High Level Design](/_images/High_LeveL_Design.png)

In this design, we will be using the micro-service architecture, where each service is responsible for a subset of operations. We should be dealing with a relatively read-heavy system, especially with campaigns, items, and collections. The gacha service is an independent service that serves as the "core" of Sanctum, and acts as both a data consumer and producer, ensuring data consistency.

Here, it is important to note that Sanctum is split into two distinct projects, the Client (user-facing) and services. This GitHub repo (Sanctum) will be for anything user-facing, and the various other services will be split into other repos. [Below](#user-interfaces), you will find mock-ups of User Interfaces for the Client, which will outline several of the above use-cases.

### Data Models and API
In this high-level design document, we will not go into specific details about our data models and API, but it is important to talk about them.

At a high-level, we can visualize the following tables and relations:

![Rough Data Models](/_images/Data_Models.png)

- Items and Collections are deliberately kept separate
- Campaigns operate on Collections only
- Users database only keep user and authentication information
- Inventory is split apart from Users to reduce operations performed on the Users database

### Data Storage
As noted above, since we will be pursuing strong consistency, I'll be using PostgreSQL as the main form of storage. AWS S3 will be used as Object storage for backups for the MVP and serve static images for items.

Some considerations were made to potentially use a NoSQL database like Cassandra or MongoDB, but:

1. Personally, I wanted to get more experience with SQL and PostgreSQL
2. Strong consistency for item states was defined above and PostgreSQL fits that requirement

## User Interfaces
### User Account Management
...

### Campaign Management
...

### Item and Collection Management
...

### Foundry Integration
...

### Gacha (Item Rolling)
...

## Testing
### Unit Tests
Unit tests will be written using Pytes

### Functional tests

## Deployment
Containers and Docker

## Maintenance
Monitoring
Backup and Recovery
