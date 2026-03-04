# Car Dealer E-commerce System
Damian D'Souza | Kamil Marszałek | Michał Suski | Michał Szwejk | Piotr Szkoda

## Table of Contents
1. [Demo](#demo)
2. [Stages of the Project](#stages-of-the-project)
3. [System Overview](#system-overview)
4. [System Architecture](#system-architecture)
5. [Technology Stack](#technology-stack)
6. [Database Design](#database-design)
7. [Backend](#backend)
8. [Frontend](#frontend)
9. [API Documentation](#api-documentation)
10. [Real-time Features](#real-time-features)

# Demo

## Main Page
![Offers page 1](images/offers-1.png)
![Offers page 2](images/offers-2.png)
![Mazda showcase](images/Mazda.png)

## Adding an Offer
![New offer 1](images/new-1.png)
![New offer 2](images/new-2.png)
![New offer 3](images/new-3.png)

## User Account
![Account page](images/account.png)
![Reviews page](images/reviews.png)
![My car](images/my-car.png)
![Bought cars](images/bought.png)


# Stages of the Project

1. Development of a conceptual (E-R) model ✅
2. Development of a relational logical data model based on the conceptual model ✅
3. Design of application functionality: operational part and analytical/reporting part ✅
4. Optimization of the logical model (especially denormalization) to maximize system performance ✅
5. Development of functional database-level elements (triggers, stored procedures) ✅
6. Selection of database technology, environment installation, and configuration ✅
7. Selection of application technology, development environment installation, and configuration ✅
8. Development, implementation, and optimization of the physical model ✅
9. Preparation of test scenarios and test data ✅
10. Preparation of analytical/design documentation (especially data model diagrams with descriptions) ✅
11. Preparation of user documentation for the application ✅



## System Overview

This application allows users to register, create and manage car sale offers, participate in auctions, and purchase vehicles.

### **Key Features**

- **Car Catalog**: Comprehensive car database with manufacturers, models, and detailed specifications
- **Auction System**: Real-time bidding
- **Sale Offers**: Sellers can create and manage car sale listings
- **Image Management**: Cloud-based storage and management of car photos
- **Real-time Notifications**: WebSocket-based live updates for offers and auction events
- **Search and Filtering**: Advanced search capabilities for cars and auctions


## System Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │    Backend      │    │    Database     │
│   (Next.js)     │◄──►│   (Go/Gin)      │◄──►│  (PostgreSQL)   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                              │
                              ▼
                       ┌─────────────────┐
                       │      Redis      │
                       │(Message Broker) │
                       └─────────────────┘
                              │
                              ▼
                       ┌─────────────────┐
                       │   Cloudinary    │
                       │ (Image Storage) │
                       └─────────────────┘
```

### **Architecture Principles**

- **Modularity**: Division into domains and modules for better code organization
- **RESTful API**: Standard REST endpoints with comprehensive Swagger documentation
- **Real-time Communication**: WebSocket integration for live updates
- **Message Broker**: Redis for message passing
- **Cloud Storage**: External image storage for scalability
- **Container-based Deployment**: Containerization for consistent environments

## Technology Stack

### **Backend Technologies**
- **Go 1.21+**: Main backend language
- **Gin Framework**: HTTP web framework for Go
- **GORM**: Object-relational mapping library
- **Redis**: Message broker
- **JWT**: JSON Web Tokens for authentication
- **WebSocket**: Real-time communication via gorilla/websocket
- **Swagger**: API documentation with gin-swagger
- **Cloudinary**: Cloud image storage
- **bcrypt**: Password hashing

### **Frontend Technologies**
- **Next.js 14**: React framework with App Router
- **TypeScript**: Type-safe JavaScript
- **React 18**: UI library with the latest features
- **NextAuth.js**: Authentication framework
- **TailwindCSS**: Utility-first CSS framework

### **Database Technologies**
- **PostgreSQL 15+**: Main database
- **pg_ivm**: Incremental View Maintenance extension
- **Custom Triggers**: Business logic at the database level
- **Views**: Materialized views for query optimization

### **DevOps and Deployment**
- **Docker**: Containerization platform
- **Git**: Version control system

## Database Design

The database is designed using PostgreSQL with normalization principles and data integrity in mind.

### **Conceptual Model**
![er.png](images/er.png)

### **Relational Model**

![relational.png](images/relational.png)

### **Database File Structure**

```
database/
├── schema.sql          # Basic table and data type definitions
├── triggers.sql        # Database triggers for business logic
├── init-pg-ivm.sql     # Materialized views with pg_ivm extension
├── init-test-db.sql    # Test database initialization
└── Dockerfile          # Database container configuration
```


### **Data Types and Enums**

The database uses custom PostgreSQL enums for type safety:

- **SELECTOR**: User type (`'P'` for Person, `'C'` for Company)
- **OFFER_STATUS**: Sale offer states (`'pending'`, `'ready'`, `'published'`, `'sold'`, `'expired'`)
- **COLOR**: Car colors (20 predefined colors plus `'other'`)
- **FUEL_TYPE**: Engine fuel types (`'diesel'`, `'petrol'`, `'electric'`, `'hybrid'`, etc.)
- **TRANSMISSION**: Gearbox types (`'manual'`, `'automatic'`, `'cvt'`, `'dual_clutch'`)
- **DRIVE**: Drive systems (`'fwd'`, `'rwd'`, `'awd'`)
- **DOCUMENT_TYPE**: File classifications (`'invoice'`, `'receipt'`, `'other'`)

### **Advanced Database Features**

#### Triggers (triggers.sql)
- **Auction Creation Trigger**: Automatically sets `is_auction = TRUE` when an auction record is created
- **User Deletion Trigger**: Cleans up unsold offers when a user is deleted

#### Performance Optimizations
- **Unique Indexes**: Optimized constraints on materialized views
- **Composite Indexes**: Multi-column indexes for frequent queries
- **Foreign Key Constraints**: Referential integrity with cascade deletes
- **Check Constraints**: Database-level data validation

### **Denormalization**

The design uses the **pg_ivm (Incremental View Maintenance)** extension to create smart views that automatically keep data consistent:

#### Materialized Views:

- **offer_creators**: Fast lookup of offer creators
- **offer_bidders**: Active bidders in published auctions
- **offer_likers**: Users who liked published offers
- **user_offer_interactions**: Union of all user-offer relationships
- **regular_sale_offer_view**: Comprehensive view of non-auction offers
- **auction_sale_offer_view**: Comprehensive view of auction offers
- **sale_offer_view**: Combined view of all sale offers with car details

#### Performance
- **Elimination of complex JOINs**: Materialized views already store joined data from multiple tables
- **Precomputed aggregates**: Offer counts, average prices, bidding statistics
- **Incremental updates**: pg_ivm updates only changed parts of views

#### Scalability
- **Reduced CPU load**: No need to execute costly JOINs on every query
- **Parallel access**: Reading from materialized views does not block transactional operations

#### Consistency
- **Automatic synchronization**: pg_ivm guarantees real-time data consistency
- **Integrity**: View updates without additional operations

#### Advantages
- Significant read query performance improvement
- Simplified application logic
- Better scalability with large datasets

#### Drawbacks
- Increased disk space usage
- Potential view update latency

### **Data Integrity Features**

- **Cascade Deletes**: Related records are cleaned up appropriately
- **Default Values**: Soft deletion using a default user (ID: 1) for referential integrity
- **Range Constraints**: Validated ranges for numeric fields (doors: 1-6, seats: 2-100, etc.)
- **Unique Constraints**: Duplicate prevention where required
- **Deferrable Constraints**: Support for complex transaction scenarios

## Backend

The backend is built in Go, using the Gin framework and GORM for interaction with the PostgreSQL database.

### **Domain Structure**

```
internal/domains/
├── auction/          # Auction management
├── auth/             # Authentication and authorization
├── bid/              # Bidding operations
├── car/              # Car catalog management
├── generic/          # Functionality shared across domains
├── image/            # Image handling
├── liked_offer/      # Liked offers management
├── manufacturer/     # Car manufacturer management
├── model/            # Car model management
├── notification/     # Real-time notifications
├── purchase/         # Purchase and transaction management
├── refresh_token/    # Refresh token management
├── review/           # Review system
├── sale_offer/       # Sale offer management
├── scheduler/        # Scheduled and recurring tasks
├── user/             # User management
└── ws/               # WebSocket connection management
```


### **Key Components**

#### Model Layer
- **Database Models**: GORM-based model definitions
- **Validation**: Input validation and sanitization
- **Relationships**: Complex relationships between entities

#### Domain Layer
- **Business Logic**: Core business rules and operations
- **Services**: Domain-specific service implementations
- **Repositories**: Data access abstraction

#### API Layer
- **Routes**: RESTful endpoint definitions
- **Middleware**: Authentication, CORS, logging
- **Handlers**: HTTP request/response processing

### **Authentication Flow**

1. **Registration**: User creates an account (person or company)
2. **Login**: Credential validation and JWT token issuance
3. **Token Refresh**: Automatic token renewal
4. **Authorization**: Access control for protected resources

### **Real-time Features**

- **WebSocket Connections**: Persistent connections for live updates
- **Notification System**: Real-time offer notifications
- **Connection Management**: User session tracking

## Frontend

The frontend is built in Next.js, using TypeScript and TailwindCSS to create a responsive user interface.

### **Application Structure**

```
app/
├── (home)/              # Application homepage
├── login/               # User login
├── signup/              # New user registration
├── account/             # User account management
│   ├── activity/        # User activity history
│   ├── favorites/       # Liked offers
│   ├── listings/        # User listings
│   ├── notifications/   # Notifications
│   ├── reviews/         # Reviews and feedback
│   └── settings/        # Profile settings
├── offer/               # Sale offer management
│   ├── add/             # Add new offers
│   └── [id]/            # Details of a specific offer
├── seller/              # Seller profiles
│   └── [sellerId]/      # Specific seller profile
├── actions/             # Server Actions for Next.js
├── api/                 # API Routes (API route handlers)
├── lib/                 # Libraries and utilities
├── test/                # Test components
└── ui/                  # User interface components
```

### **Key Features**

#### Authentication Integration
- **NextAuth.js**: Secure authentication with JWT
- **Session Management**: Automatic token refresh
- **Route Protection**: Middleware-based access control

#### User Interface
- **Component Library**: Reusable UI components
- **Form Handling**: Type-safe form validation
- **Image Upload**: Cloudinary integration

#### State Management
- **Server Components**: Optimized data fetching
- **Client Components**: Interactive UI elements
- **API Integration**: Axios-based HTTP client


## API Documentation

### **Main Endpoints**


#### Authentication
```
POST /auth/register          - User registration
POST /auth/login             - User authentication
POST /auth/refresh           - Token refresh
PUT  /auth/change-password   - Password change
POST /logout                 - User logout
```

#### User Management
```
GET    /users/              - List all users
PUT    /users/              - Update user profile
GET    /users/id/{id}       - Get user by ID
GET    /users/email/{email} - Get user by email
DELETE /users/{id}          - Delete user
```

#### Sale Offers
```
POST /sale-offer/                  - Create sale offer
PUT  /sale-offer/                  - Update sale offer
PUT  /sale-offer/publish/{id}      - Publish sale offer
POST /sale-offer/filtered          - Filter sale offers
POST /sale-offer/my-offers         - My sale offers
POST /sale-offer/liked-offers      - Liked offers
GET  /sale-offer/id/{id}           - Offer details by ID
GET  /sale-offer/offer-types       - Offer types
GET  /sale-offer/order-keys        - Sorting keys
POST /sale-offer/buy/{id}          - Buy now
```

#### Auctions
```
POST /auction/              - Create auction
PUT  /auction/              - Update auction
POST /auction/buy-now/{id}  - Buy now in auction
```

#### Bidding
```
POST /bid/                                              - Place bid
GET  /bid/                                              - List all bids
GET  /bid/{id}                                          - Bid details by ID
GET  /bid/bidder/{id}                                   - Bids by bidder ID
GET  /bid/auction/{id}                                  - Bids in auction
GET  /bid/highest/{id}                                  - Highest bid
GET  /bid/highest/auction/{auctionID}/bidder/{bidderID} - User's highest bid
```


### **API Features**

- **Swagger Documentation**: Interactive API explorer at `/swagger/`
- **Request/Response Models**: Comprehensive data schemas
- **Error Handling**: Standardized error responses
- **Pagination**: Cursor-based pagination for large datasets
- **Filtering**: Advanced query parameters for data filtering
- **Sorting**: Flexible sorting options

## Real-time Features

### **WebSocket Implementation**

The system implements real-time features using WebSocket connections:

#### Event Types

```json
{
  "type": "bid_placed",
  "auction_id": "123",
  "bid_amount": 25000,
  "bidder": "user456",
  "timestamp": "2024-01-15T10:30:00Z"
}
```

```json
{
  "type": "auction_ended",
  "auction_id": "123",
  "winning_bid": 30000,
  "winner": "user789",
  "timestamp": "2024-01-15T11:00:00Z"
}
```
