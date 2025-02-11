# Event Definitions

## Summary

This document provides a comprehensive list of events used in the Welz platform, detailing their payloads and the modules that publish and subscribe to them. It ensures a clear understanding of the event-driven architecture and facilitates communication between modules.

## Events

### AccountConnected
- **Publisher**: Command Module
- **Subscribers**: Query Module, Data Aggregation Module
- **Description**: Emitted when a user successfully connects a new bank account or investment account to their profile
- **Payload**:
  ```json
  {
    "userId": "string",
    "provider": "string",
    "accountId": "string",
    "timestamp": "string"
  }
  ```

### DataSyncStarted
- **Publisher**: Data Aggregation Module
- **Subscribers**: Query Module
- **Description**: Emitted when the system begins synchronizing data from a connected financial institution
- **Payload**:
  ```json
  {
    "userId": "string",
    "provider": "string",
    "timestamp": "string"
  }
  ```

### DataSyncCompleted
- **Publisher**: Data Aggregation Module
- **Subscribers**: Query Module, Financial Insights Module
- **Description**: Emitted when data synchronization finishes successfully, including the count of new transactions found
- **Payload**:
  ```json
  {
    "userId": "string",
    "provider": "string",
    "newTransactionCount": "number",
    "timestamp": "string"
  }
  ```

### TransactionCreated
- **Publisher**: Command Module
- **Subscribers**: Query Module, Financial Insights Module
- **Description**: Emitted when a new transaction is received from a bank or manually created by a user
- **Payload**:
  ```json
  {
    "transactionId": "string",
    "userId": "string",
    "amount": "number",
    "timestamp": "string"
  }
  ```

### TransactionUpdated
- **Publisher**: Command Module
- **Subscribers**: Query Module, Financial Insights Module
- **Description**: Emitted when transaction details (amount, date, description) are modified
- **Payload**:
  ```json
  {
    "transactionId": "string",
    "userId": "string",
    "amount": "number",
    "timestamp": "string"
  }
  ```

### TransactionDeleted
- **Publisher**: Command Module
- **Subscribers**: Query Module, Financial Insights Module
- **Description**: Emitted when a transaction is marked as deleted (soft delete)
- **Payload**:
  ```json
  {
    "transactionId": "string",
    "userId": "string",
    "timestamp": "string"
  }
  ```

### TransactionCategorized
- **Publisher**: Command Module
- **Subscribers**: Query Module, Financial Insights Module, Categorization Module
- **Description**: Emitted when a transaction receives a category, either automatically by AI or manually by the user
- **Payload**:
  ```json
  {
    "transactionId": "string",
    "userId": "string",
    "category": "string",
    "previousCategory": "string | null",
    "source": "ai" | "user",
    "timestamp": "string"
  }
  ```

### ProfileShared
- **Publisher**: Command Module
- **Subscribers**: Query Module, Financial Insights Module
- **Description**: Emitted when a user grants another user access to view their financial profile
- **Payload**:
  ```json
  {
    "userId": "string",
    "sharedWithUserId": "string",
    "timestamp": "string"
  }
  ```
