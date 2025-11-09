# Purpose 
Cloak Me is a privacy-first companion for WhatsApp group messaging. It surfaces incoming group messages in a private reading interface where reading does not automatically send read receipts, typing indicators, or online presence back to WhatsApp. Users choose when or whether to send a read receipt manually. Cloak Me integrates with the WhatsApp Cloud API and is permission-driven, reversible, and secure
## High level overview
### What system does
•	Ingests WhatsApp group messages via the WhatsApp Cloud API.
•	Stores minimal, permissioned data and provides a private, distraction-free UI to read messages without triggering presence signals on WhatsApp.
•	Lets users manually decide when to reveal that they read messages.
## Architecture style
•	Client-server with microservices-friendly boundaries. Core services are separated (API, Auth, Messaging sync, Scheduling, Notification) so we can scale parts independently.
'''
flowchart LR
  subgraph Client
    MobileApp[Mobile / Web App]
    NotificationService[Local Notification Handler]
  end

  subgraph Backend
    APIGateway[API Gateway]
    AuthService[Auth Service (OAuth + Tokens)]
    WhatsAppSync[WhatsApp Sync Service]
    MessageStore[(Encrypted Message Store)]
    Scheduling[Read Receipt Scheduler]
    Delivery[Delivery Service (Reveal / Send Receipt)]
    UserService[User Profile & Preferences]
    Audit[(Audit / Event Log)]
  end
  subgraph Infra
    CDN[CDN]
    Push[Push Notification Provider]
    DB[(Primary DB)]
    Cache[(Encrypted Cache / Redis)]
    Queue[(Message / Task Queue)]
  end


  MobileApp -->|API requests| APIGateway
  APIGateway --> AuthService
  APIGateway --> WhatsAppSync
  WhatsAppSync --> MessageStore
  WhatsAppSync --> Queue
  Queue --> Scheduling
  Scheduling --> Delivery
  Delivery -->|WhatsApp Cloud API| WhatsAppSync
  MobileApp --> NotificationService
  NotificationService --> Push
  UserService --> MessageStore
  Audit --> MessageStore
  '''
  ## Technology Stack
  Frontend: React Native (mobile), React (web) — single codebase patterns optional.
  API / Backend: Node.js (TypeScript) or Python (FastAPI); small microservices.
  Database: PostgreSQL for relational data (users, preferences, audit). Use column-level encryption for sensitive fields.
  Message store / queue: Redis for ephemeral cache; RabbitMQ / Amazon SQS for reliable task queue.
  
  
