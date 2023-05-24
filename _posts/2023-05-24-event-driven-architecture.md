---
title: Event Driven Architecture The Complete Guide - 강의노트
categories:
  - Dev
tags:
  - event
  - udemy
  - architecture  
  - msa
  - cqrs
last_modified_at: 2023-05-24T11:59:00-00:00
---

Udemy 강의 [Event Driven Architecture - The Complete Guide](https://kmooc.udemy.com/course/event-driven-architecture-the-complete-guide/) 를 듣고 강의내용 정리

---
- 이벤트 정의
    - The cornetstone of event driven architecture
    - require a well-defiend definition
    - evolved from other architectures
    - Microservices Communication
        - perhaps the most important part in microservices architecture
        - dictates performance, scalability, implementation and more
        - event driven architecture handles the communication part
    - Command and Query
        - Command
            - service asks another service to do something
            - there might be a reseponse to the command, usually a success or failure indicator
            - usually synchronous
            - somethimes return a response
            - calling service needs to know who handles the command
            
            ![Command service](https://capture.dropbox.com/bRnBvXMjptcJpQfj?raw=1)
            
        - Query
            - Retrieve data
            - Almost always synchronous
            - always return a response
            - calling service needs to know who handles the query
        - Problems with Command and Query
            - performance
                - synchronous, potential for performance hit
            - coupling
                - If the called service changes - the calling service has to change too
                - more work, more maintenance
            - scalability
                - the calling service calls a single instance of a service
                - if this instance is busy - there’s a performance hit
    - event
        - something happend
        - asynchronous
        - never returns a response
        - calling service has no idea who handles the event
    - contents of event
        - two types of event data
            - complete
                - contains all the relevant data
                - usually entity data
                - no additional data is required for the event processing
            - pointer
                - contains pointer to the complete data of the entity
                - complete data usually stored in a database
                - event handler needs to access the database to retrieve complete data
        - when to use which?
            - complete
                - the better approach
                - makes the event completely autonomous
                - can get out of the system boundaries
            - pointer
                - data is large
                - need to ensure data is up-to-date
        
- 이벤트 기반 아키텍처의 기본 사항
    - a software architecture paradigm that uses events as the mean of communication between services
    - often called EDA
    - has three main components
    - producer
        - the component / service sending the event
        - offen called publisher
        - usually send event reporting something the componet done
        - ex. customer service → new customer added event
        - exact method of calling the channel depends on the channel
        - usually using a dedicated SDK developed by the channel vendor
        - utilizes some kind of network call, usually with specialized ports and proprietary protocal
        - RabbitMQ listens on port 5672 and uses the AMQP protocol
        - Producer can be developed using any development language
    - Channel
        - the most important component in the E.D.A
        - Reseponsible for distributing the events to the relevant parties
        - The channel places the event in a specializd queue, often called Topic or Fanout
        - Consumers listen to this queue and grab the event
        - Note
            - Implementation details vary wildly between channels
            - RabbitMQ works differently than kafka that works differently than webhooks
            - Always dive deep into the docs of the channel you’re using
            - we’ll use RabbitMQ and SignalR in the implementation section
        - the Channel distributes the event to the Consumers
        - The channel’s method of distribution varies between channels
        - Can be:
            - Queue:
            - Rest API call
            - Proprietary listener
    - consumer
        - the component that receives the event sent by the Producer and distributed by the Channel
        - Can be developed in any development language compatible with the Channel’s libraries
        - processed the event
        - sometimes - reports back when processing is complete (Ack)
        - consumer gets the event using either
            - push
            - pull
        - the method depends on the channel
    - Advantages of EDA
        - E.D.A has a lot of advantages over other architecture paradigms
        - As a quick refresher…
    - Problems with Command and Query
        - Performance
            - EDA is an asynchronous architecture
            - The Channel does not wait for response from consumer
            - No performance bottlenecks
        - Coupling
            - The producer sends events to the channel
            - the channel distributes events to topics / queues
            - both have no idea who’s listening to the event
            - no coupling
        - Scalability
            - Many consumers can listen to events from channel
            - More can be added as needed
            - Channel doesn’t care, producer doesn’t know
            - Fully scalable
    - EDA and Pub
        - EDA is often mentioned with Pub / Sub
        - Pub / Sub = Publish and Subscribe
        - A messaging pattern used by E.D.A
        - Main difference
            - EDA descibes the whole architecture of the system
            - Pub/Sub is a messaging pattern used by the system
    - Ordering in EDA
        - messaging engines often gurantee the order of the messages
        - with event driven architecture (especially with pub/sub) ordering is not always guranteed
        - ordering might be affected by consumer latency, code performance and more
        - If ordering is important,
            - RabbitMQ supports it
            - SignalR does not
    - Orchestration and Choreography
        - Orchestration
            - Flow of events in the system is determined by a central orchestrator
            - Orchestrator receives output from components and calls the next component in the flow
            - The next component send the output back to the orchestrator etc.
        - Choreography
            - No central “knowing all” component
            - Each component notifies about the status of events
            - Other components listen to the events and act accordingly
            - 

- 이벤트 소싱 및 CQRS
    - Event Sourcing and CQRS
        - Event Driven Architecture is mainly about services
        - Event can be used as the basic building blocks of data too
        - Event Sourcing and CQRS offer a pattern to store data as events
    - Problems with Traditional DBs
        - Traditional databases hold data about current state of entity
        - There is no way to see historical data of entities
        - Data is a “snapshot” of a point int time
        - Especially problematic with…
    
    ![Problems with Traditional DBs](/assets/images/problems-with-traditional-dbs.png)
    
    - Event Sourcing
        - A data store pattern in which every change in the data is captured and saved
        - database stores list of changes for the entity, not the entity itself
        - No updates or deletes, just inserts
        - Every row documents a change in a property/ies of the entity
        - in this pattern, the database is called Event Store
        
        ![Event Sourcing1](https://capture.dropbox.com/QCj40YAcgmstW6pe?raw=1)
        
        ![Event Sourcing2](https://capture.dropbox.com/aFwHd46a6Vo2oJGy?raw=1)
        
        - Pros.
            - Extremely easy to view historical data
            - simple database structure
            - simple database operations (no updates, no concurrency)
            - very fast inserts
        - Cons.
            - viewing current entity state is cumbersome and slow
            - large database capacity (many records per entity)
            - `CQRS to the Rescue!`
    - CQRS stans for:
        - Command and Query Responsibility Serregation
        - separating the commands (updates/ inserts/ deletes) from the queries
        - each one of them in a seperate database
        - commands database is implemented as event store to improve performance and simplicity
        - Queries database stores entities
        - Database are synced using a central synchronization mechanism
    - CQRS
        - Pros
            - combines Event sourcing pros with traditional entity query
            - No performance hit when querying entities
        - Cons
            - Entity data is not updated in real-time
            - Difficult to set up and maintain
    - When to use event sourcing & CQRS
        - when access to historical data is extremely important
            - regulation, finance, healthcare
        - when data is large and replaying events is not feasible
        - when performance is critical (inserts or queries)
        - 
- When to Use EDA
    - Scalability
        - Scalability is a non- issue in EDA
        - New consumers can be added as needed with no changes to the architecture
        - Great for fluctuating load
    - Asynchronous
        - if inter-service communication can be asynchronous, consider EDA
        - Remember: EDA is async by nature
        - Examples:
            - Send instructions to perform payment
            - write to log
        - check how many synchronous interactions there are
        - usually mainly queries
        - the more synchronous calls - the less EDA is relevant
    - Reliable Network
        - EDA utilizes a lot of traffic
        - Network shoud be reliable or performance will be slow
    - When not to Use EDA
        - EDA is not suitable for:
            - small systems with a few services
            - synchronous-oriented systems
                - ie. information system serving mainly queries from end users

- stateless and stateful EDA
    - Stateless vs Stateful EDA
        - related to the consumer behavior
        - both are legitimate, but make sure to select the right one for your scenario
        - while the concepts are similar, the reasoning is different
        - with software architecture it’s often said that:
            - “stateful is bad”
        - there is not necessarily the case with EDA
    - stateless EDA
        - each event handled by a consumer is completely autonomous and is not related to past/ future events
        - should be used when the event is an independant unit with its own outcomes
        
        ![Stateless EDA](https://capture.dropbox.com/n5MGlZy6LP71pk39?raw=1)
        
        - it doesn’t matter which consumer is handling the event
        - the outcome is always the same
        - should be used when each event is autonomous
        - note: has nothing to do with the question of what data is contained in the event and whether a call to a DB is required
    - Stateful EDA
        - events might be related to past / future events
        - should be used mainly for aggregators and time-related events
        - examples:
            - send an email if more than 5 failure events were received in a single minute
            - calculate the amount of orders submitted in an hour
    
        ![Stateful EDA](https://capture.dropbox.com/S1GvFYld07R3rqiG?raw=1)
    
    - it’s extremely important which consumer handles the event
    - current state is stored in specific consumer(s)
    - should be used when events are part of a chain of events
    - Problems with Stateful EDA
        - stateful EDA presents some problems that should be taken care of
            - Load balancing
                - since the state is stored in a specific consumer, subsequent events must be routed to the same consumer
                - no load balancing is possible
            - Scalability
    - stateless vs stateful
        - rule of thumb:
            - use stateless EDA unless the business requirements force you to use stateful

- 스트리밍
    - event streaming engines publish stream of events
        - telemetry from sensors, system logs etc
        - the events are published to a “stream”
        - consumers subscribe to specific stream
        - events are retained in a stream for a specified amount of time
        
        ![Event streaming](https://capture.dropbox.com/CZgYafvvYA8GBs2f?raw=1)
        
    - event streaming
        - consumers can retrieve events that were sent in the past (usually up to a few days)
        - streaming engine can be used as a central database
        - not all events are necessarily handled
        - usually used for events generated outside of the system
        - events are retained
        - not all events are handled
        - high load
    - when to use event streaming
        - when the system needs to handle stream of events from the outside
            - sensor data, logs, etc
        - when events should be retained for future use
        - when high load is expected
    
- 로깅 및 모니터링
    - especially challenging in event driven architecture
    - in EDA there are distributed, non-coupled components
    - getting unified, chronologically ordered logs is difficult
        - each component writes log to different destination
        - log formats are different
        - timestamps are not synchronized
    - correlation id
        - handles the following problems:
            - timestamps not synced
            - different destination
            - many records, hard to trace
        - at the beginning of each transaction, a unique identifier is attached to the message
        - this identifier is logged as part of the log record
        - allows tracing across components
        
        ![Using Correlation Id in Logging](https://capture.dropbox.com/EUI0uBbn4tA1iN4l?raw=1)
        
    - central logging engine
        - central engine used for aggregating all the log records
        - has converters that convert between log formats
        - great analytics and visualizations capabilities
        - can utilize the existing channel for log transport
    - implementing central logging engine
        - the elastic stack
    - what should be logged
        - logs should reflect the behavior of the system
        - shoud be able to replay a transaction using logs
        - event trigger
        - event contents
        - event receive
        - event handlig complete
- Advanced Topics
    - Mixing EDA with Request / Response
        - Most EDA system are not pure EDA
        - Main reason:
            - UI Clients need responsiveness and use Web API to call the backend
        - If client only asks for data, EDA will probably won’t work
    - Synchronous EDA
        - EDA is asynchronous by nature
        - Normally the producer does not wait for a response to the event
        - Sometimes it does, as a separate event
        - Quite difficult to implement
        - EDA with Response
            - send the event
            - wait for the response event
            - wait some more
            - handle the return event in a separate code segment
        - To make the process easier you can create a wrapper around the producer
        - The wrapper expose a synchronous Web API
        - Calls the producer and performs all the dirty work of sending → wating → handling response
        
        ![Synchronous EDA](https://capture.dropbox.com/js5pTbha7N7KvWKf?raw=1)
        
        - Not easy to implement
        - Do it only if you absolutely need a response from the event
        - Some channels (e.g. RabbitMQ) have built-in support for this
    - Events as Source of Truth
        - with traditional systems, the database holds the operational data and the events trigger actions in the system
        - with channels that retain events, the channel can function as the source of truth
        - Relevant only if:
            - events are retained
            - the channel has some kind of query language
            - main functionality of the system is around event streaming, no complex interactions
    - Implementing Events as Source of Truth
        - events are retained
        - has the KQL qeury language
        - designed for event streaming
    - The Saga Pattern
        - Transaction management in distributed system is difficult
        - No accepted, widely used solution
        - 2-phase commit protocal exists, but difficult to implement
        
        ![The Saga Pattern1](https://capture.dropbox.com/tOrTiGsMGyVTB5C1?raw=1)
        
        - The sage pattern strive to solve this problem
        - A sequence of service- scoped transaction, triggerd by events
        - when a transaction fails, a compensating transaction is triggered
        
        ![The Saga Pattern2](https://capture.dropbox.com/EAiBpWPoJYjaURbn?raw=1)
        
        - Not easy to implement
        - Consistency is not ensured - compensating transaction can fail
        - quite difficult to debug
        - monitoring is important
        - when to use?
            - No need to strongly - coupled transactions
            - compensating transactions can be defined
    - EDA on the Front End
        - So far we discussed Event Driven Architecture in the back end
        - It can also be implemented on the front end
        - we won’t deep-dive into it, but raise awareness to it
        - Micro Frontends
            - the page is divided to separate, independent UI component
            - Each component has its own UI, functionality, backend etc.
            - Components communicate with each other using events
            - Implementing Micro Frontends
                - the components themselves are platform agnostic and can use any fronted framework
                - the inter-component communication is done using Browser Events and Custom Elements, supported by all major browsers
        - Push Notifications
            - Events that are sent to the browser from the server
            - As opposed to the more traditional Request/ Response model
            - Not exactly EDA but involves events in the clients
            - very useful for apps require streaming info from the server
            - quite a lot of libraries and frameworks:
                - SignalR
                - Socket.IO
                - gRPC
                - And more …

- Implementing Event Driven Architecture
    - Events Approach
        - Two main approaches for implementing events:
            - events are retained
            - events are not retained
    - Retaining Events
        - The channel retains the event for future handling
        - a retention period is defined which after it expires - the event is removed
        - great for streaming events and when the channel is the source of truth
        - use a messaging / queue engine
            - RabbitMQ / Kafka
    - Not Retaining Events
        - the channel publishes the events and does not store them
        - if a consumer missed an event - it can’t be replayed
        - used mainly for in-system events
        - use an event publisher
        - specific engine depends on platform used, types of interfaces and more
        - let’s see some examples…
        - azure event grid
        - webhooks
            - a standard for publishing events using REST API
            - consumers subscribe to the webhook engine and register a REST API endpoint that will be called when an event occurs
            - the webhook will call the endpoints when event is triggered
            - easy to implement
            - supported by github, dropbox, paypal, stripe and more

        ![Webhooks Flow](https://capture.dropbox.com/1mcts5U0uEqGxMP4?raw=1)

    - HTTP Push Notification
        - send events from the server to client
        - great for chats, message notification and more
    - Implementing the producer
        - can be based on any platform
        - needs to be able to communicate with the channel
        - depends on the channel implementation
        - lets’ see some exampels…
        - RabbitMQ
        - SignalR
    - Implementing the consumer
        - can be based on any platform
        - needs to be able to communicate with the channel
        - depends on the channel implementation
        - let’s see some examples…
        - RabbitMQ
        - WebHooks

    - 실제 사례 연구
        - RabbitMQ 및 SignalR과 같은 엔진을 사용하여 이벤트 기반 아키텍처의 개념을 보여주고 향후 프로젝트의 템플릿으로 사용할 수 있는 완전한 기능을 갖춘 이벤트 기반 시스템을 구축
        - NOP - The NOise Processing system