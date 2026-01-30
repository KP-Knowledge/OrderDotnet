# OrderDotnet

A REST API service for order management system built with .NET following Clean Architecture principles.

## Table of Contents
- [Overview](#overview)
- [Architecture](#architecture)
- [Domain Models](#domain-models)
- [Order State Transitions](#order-state-transitions)
- [Technologies](#technologies)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Running the Application](#running-the-application)
- [API Endpoints](#api-endpoints)
- [Workflow Activities](#workflow-activities)
- [Documentation](#documentation)
- [Examples](#examples)
- [Design Principles](#design-principles)

## Overview

OrderDotnet is a comprehensive order management system that handles the complete lifecycle of orders in an e-commerce environment. The system implements a sophisticated state machine for order processing, integrates with Temporal workflows for complex orchestration, and maintains strict adherence to Clean Architecture and Domain-Driven Design principles.

### Key Features
- **State Machine Pattern:** Robust order state transitions with validation (Initial → Pending → Paid → Completed/Refunded → Cancelled)
- **Idempotent Operations:** All operations are idempotent by design with reference ID tracking
- **Loyalty Program Integration:** Earn and burn loyalty points with transactions
- **Payment Processing:** Multi-method payment support (Credit Card, Debit Card, Bank Transfer, Digital Wallet, Cash)
- **Stock Management:** Stock reservation and tracking with confirmation workflows
- **Order Items:** Complete line item management with product details and pricing
- **Audit Trail:** Comprehensive state transition history via OrderJourneys and OrderLogs
- **Temporal Workflows:** Complex order processing orchestration with Temporal

## Architecture

The project follows **Clean Architecture** with clear separation of concerns:

### Layers
- **Domain** - Pure business logic with entities, aggregates, value objects, and domain services (no external dependencies)
- **Application** - Use cases, services, and DTOs orchestrating domain logic
- **Infrastructure** - Repository implementations, EF Core configurations, and external integrations
- **Api** - REST API controllers exposing application functionality
- **Workflow** - Temporal workflow activities and workflow definitions
- **DatabaseMigration** - EF Core database migrations and schema management

### Additional Components
- **Service** - Shared service extensions
- **Tests** - Comprehensive unit and integration tests
- **Examples** - Code examples and usage patterns
- **Docs** - Technical documentation and PRDs

## Domain Models

### Core Entities
- **Order** - Main order aggregate root with state management
- **OrderItem** - Order line items with product details, quantity, and pricing
- **OrderLoyalty** - Loyalty program earn/burn transactions
- **OrderPayment** - Payment records with multi-method support
- **OrderStock** - Stock reservations and confirmations
- **OrderJourneys** - State transition history and audit trail
- **OrderLogs** - Comprehensive action logging

### Relationships
- Order ↔ OrderItem (1-to-many)
- Order ↔ OrderLoyalty (1-to-many)
- Order ↔ OrderPayment (1-to-1)
- Order ↔ OrderStock (1-to-many)
- Order ↔ OrderJourneys (1-to-many)
- Order ↔ OrderLogs (1-to-many)

## Order State Transitions

### Available States
- **Initial** - Starting state when order is created
- **Pending** - Order awaiting payment or processing
- **Paid** - Payment received and confirmed
- **Completed** - Order successfully fulfilled
- **Refunded** - Payment refunded to customer
- **Cancelled** - Order cancelled and will not be processed

### Valid Transitions
```
Initial → Pending
Pending → Paid | Cancelled
Paid → Completed | Refunded
Refunded → Cancelled
Completed → Cancelled
Cancelled → Pending (reactivation)
```

### Business Rules
- **Paid State:** Requires sufficient payments
- **Completed State:** Requires full payment and confirmed stock
- **Cancelled State:** Prevents cancellation of completed orders
- **Refunded State:** Only allows refunds of paid/completed orders

## Technologies
- **Framework:** .NET 6+
- **Database:** PostgreSQL
- **ORM:** Entity Framework Core
- **Workflow Engine:** Temporal
- **Architecture Patterns:** Clean Architecture, Repository Pattern, Aggregate Pattern, State Machine Pattern, Use Case Pattern
- **Testing:** xUnit, Moq

## Getting Started

### Prerequisites
- .NET 6 SDK or later
- Docker and Docker Compose
- Temporal CLI
- PostgreSQL (via Docker or local installation)

### Installation
1. **Clone the repository:**
   ```bash
   git clone https://github.com/your-username/OrderDotnet.git
   cd OrderDotnet
   ```

2. **Start PostgreSQL:**
   ```bash
   docker-compose -f Docker/postgres.yaml up -d
   ```

3. **Run database migrations:**
   ```bash
   cd DatabaseMigration
   dotnet run
   ```

4. **Start Temporal:**
   ```bash
   temporal server start-dev
   ```

### Running the Application

1. **Run the API:**
   ```bash
   cd Api
   dotnet run
   ```

2. **Run the Workflow Worker:**
   ```bash
   cd Workflow
   dotnet run
   ```

The API will be available at `http://localhost:5000` (or as configured in [appsettings.json](Api/appsettings.json)).

## API Endpoints

### Order Management
- `POST /api/orders` - Create a new order with items
- `GET /api/orders/{orderId}` - Get order details
- `PUT /api/orders/{orderId}/state` - Transition order state
- `GET /api/orders/{orderId}/valid-states` - Get valid next states

### Order Items
- `POST /api/orders/items` - Add order item
- `PUT /api/orders/items` - Update order item
- `DELETE /api/orders/items/{id}` - Remove order item
- `GET /api/orders/{orderId}/items` - Get order items

### State Transitions
- `POST /api/orders/{orderId}/transition` - Execute state transition with validation
- `GET /api/orders/{orderId}/journeys` - Get state transition history

## Workflow Activities

The system uses Temporal workflows for complex order orchestration:

- **StartOrderWorkflow** - Initialize workflow with workflowId and orderId
- **ReserveStock** - Reserve inventory for order items
- **BurnLoyaltyTransaction** - Process loyalty point burning
- **EarnLoyaltyTransaction** - Process loyalty point earning
- **ProcessPayment** - Handle payment processing
- **CompletedCart** - Complete the order
- **GetOrderDetail** - Retrieve order details

**Main Workflow:** `OrderProcessingWorkflow`

## Documentation

Comprehensive documentation is available in the [Docs](Docs) directory:

### Product Requirements
- [PRD_Order.md](Docs/PRD_Order.md) - Main order system requirements and iterations
- [PRD_OrderWorkflow.md](Docs/PRD_OrderWorkflow.md) - Temporal workflow requirements

### Technical Documentation
- [OrderState.md](Docs/OrderState.md) - State machine implementation details
- [TransitionOrderState_Implementation.md](Docs/TransitionOrderState_Implementation.md) - OrderAggregate state transition implementation
- [TransitionOrderStateUseCase_Implementation.md](Docs/TransitionOrderStateUseCase_Implementation.md) - Application layer use case implementation
- [test_idempotency.md](Docs/test_idempotency.md) - Idempotency implementation and testing
- [NavigationPropertiesTest.md](Docs/NavigationPropertiesTest.md) - Entity relationships and navigation properties
- [4th_Iteration_Implementation.md](Docs/4th_Iteration_Implementation.md) - OrderItem entity implementation
- [CLEANUP_SUMMARY.md](Docs/CLEANUP_SUMMARY.md) - MediatR removal and architecture simplification

## Examples

Code examples demonstrating various features are available in the [Examples](Examples) directory:

- [OrderStateEndpointExample.cs](Examples/OrderStateEndpointExample.cs) - API endpoint examples
- [OrderStateTransitionExample.cs](Examples/OrderStateTransitionExample.cs) - State transition usage
- [TransitionOrderStateUseCaseExample.cs](Examples/TransitionOrderStateUseCaseExample.cs) - Use case integration
- [WorkflowEventQueryExamples.cs](Examples/WorkflowEventQueryExamples.cs) - Workflow event queries
- [WorkflowHistoryQueryExamples.cs](Examples/WorkflowHistoryQueryExamples.cs) - Workflow history queries

## Design Principles

The project strictly follows these principles:

- **SOLID Principles** - Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion
- **DRY (Don't Repeat Yourself)** - Code reusability through extension methods and shared components
- **Domain-Driven Design** - Rich domain model with aggregates, entities, and value objects
- **Idempotent Operations** - All operations can be safely retried
- **Fail-Fast Validation** - Early validation to prevent invalid states
- **Immutability** - Using records and readonly collections where appropriate
- **Clean Architecture** - Dependency rule pointing inward to the domain

## Project Structure

```
OrderDotnet/
├── Api/                    # REST API controllers and configuration
├── Application/            # Use cases, services, and DTOs
├── Domain/                 # Pure domain logic (entities, aggregates, value objects)
├── Infrastructure/         # EF Core, repositories, external integrations
├── Workflow/              # Temporal workflows and activities
├── DatabaseMigration/     # EF Core migrations
├── Tests/                 # Unit and integration tests
├── Examples/              # Code examples and usage patterns
├── Docs/                  # Technical documentation
└── Docker/                # Docker configuration files
```

## Testing

Run tests with:
```bash
dotnet test
```

The test suite includes:
- Unit tests for domain logic
- Integration tests for repositories
- Use case tests
- State transition validation tests

## Contributing

1. Follow Clean Architecture principles
2. Ensure domain layer remains pure (no external dependencies)
3. Use ILogger<T> for logging (no Console.WriteLine)
4. Write unit tests for new features
5. Follow SOLID principles and DRY
6. Implement idempotent operations
7. Add documentation for complex features

## License

[Specify your license here]

---

For more detailed information, please refer to the documentation in the [Docs](Docs) directory and code examples in [Examples](Examples).
