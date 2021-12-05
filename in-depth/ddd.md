# DDD

## Types of Bounded Contexts

### Core Domain
Solves the main problem domain.

### Generic Subdomain
Takes care of a recurring problem that every platform has.
Example: Identity Management, User Registration

### Supporting Subdomain
Takes care of a side problem that is not an integral part of the value proposition.
This sometimes is not an extra BC but instead, if small enough, a module in the Core Domain BC.
A Monolith/BigBallOfMud is an amalgamation of Core and Supporting BCs.
Example: Model Information Pages on Buyer Side

## Context Mapping

The art of integrating 2 different Bounded Contexts. The relationship out of this integration can be categorised. The relationship can change over time and should be picked to address the issues that exist at the time.
Context mapping is usually done properly, if the “receiving” integration is able to map the external model and language into the model and language of the local domain.

Sometimes, an integration is or becomes not feasible. You might decide to go Separate Ways and implement some outside feature in your own Bounded Context (avoid turning into Big Ball of Mud!)or in a separate, in-house BC.

## Relationships

### Partnership
There is mutual alignment and frequent communication. There might be shared delivery schedules.
Example: MoVe + KCA

### Supplier/Consumer Relationship
The supplier is upstream, the consumer downstream. The supplier defines what the consumer gets and the customer adapts to changes there.
Example: VeDaH, move-event-dispatcher + Marketplaces

There are 2 subtypes: 
* Conformist is a sort of Supplier/Consumer relation, where the Supplier does not care about the consumer. The consumed concept is very complex or large, so that the conformist consumer fully adapts to it. The upstream ubiquitous language is taken in as is by the consumer, no mapping effort is made. Example: Facebook API client
* Non-Conformist with Anticorruption Layer: The most defensive way of consuming from upstream. You add a translation layer that turns the incoming model into the local model, thus keeping it pure. Anticorruption layers should always be introduced if feasible. Example: KCA, GT receiving data from the event-dispatcher

## Behaviours and Qualities

### Open Host Service (OHS) - the quality of integrating with upstream
A consumer can access upstream via a well documented API that is easy to use. The mechanism is can be pull or put.
Example: Partners using the MLG API; Event Dispatcher and its recipients
Opposite example: scraping data from an upstream website

### Published Language - a schema description exists to exchange data with upstream
There is some sort of documented schema (OpenAPI, Json Schema, DTD ..) that makes it easy to translate data between both BCs. An OHS upstream often comes with “Published Language”, helping the integration experience.

### Shared Kernel
There is a shared model (e.g. a library) or shared representation. Development of that kernel might be done by one party only, while downstream is only using the library.
Example: move-domain library (how does a listing look like, what rules are there)
