# Microservices-Based Online Event Booking Platform

This project implements a microservices-based online event booking platform with the following components:

1. **User Service** - Manages user authentication and profiles
2. **Event Service** - Manages event listings and details
3. **Booking Service** - Handles ticket bookings and payment processing
4. **Notification Service** - Sends email/SMS notifications for confirmations

## Technology Stack

| Microservice | Technology | Database | Communication |
|--------------|------------|----------|---------------|
| User Service | FastAPI | PostgreSQL | REST API (Sync) |
| Event Service | Express.js | MongoDB | REST API (Sync) |
| Booking Service | Flask | PostgreSQL | REST API (Sync), RabbitMQ (Async) |
| Notification Service | Flask | MongoDB | RabbitMQ (Async) |

## Service Communication

1. **User Service → Event Service (Sync via REST API)**
   - Users retrieve available events from the Event Service
   - Example API Call: `GET /events`

2. **User Service → Booking Service (Sync via REST API)**
   - Users create a booking by calling the Booking Service
   - Example API Call: `POST /bookings` with `{ user_id, event_id, tickets }`

3. **Booking Service → Notification Service (Async via RabbitMQ)**
   - When a booking is confirmed, the Booking Service publishes an event to RabbitMQ
   - The Notification Service consumes this event and sends a confirmation email
   - Example RabbitMQ Event: `{ booking_id, user_email, status: "CONFIRMED" }`

4. **Booking Service → Event Service (Sync via REST API)**
   - Before confirming a booking, the Booking Service checks event availability
   - Example API Call: `GET /events/{event_id}/availability`

## Setup Instructions

### Prerequisites

- Python 3.8+
- Node.js 14+
- PostgreSQL
- MongoDB
- RabbitMQ

### Environment Setup

1. **Clone the repository**
   ```
   git clone <repository-url>
   cd event-booking-platform
   ```

2. **Set up PostgreSQL databases**
   - Create `user_db` and `booking_db` databases

3. **Set up MongoDB databases**
   - Create `event_db` and `notification_db` databases

4. **Set up RabbitMQ**
   - You can use Docker to run RabbitMQ:
   ```
   docker-compose up -d
   ```

### Installing Dependencies and Running Services

#### 1. User Service
```bash
cd user-service
pip install -r requirements.txt
python main.py
```

#### 2. Event Service
```bash
cd event-service
npm install
npm start
```

#### 3. Booking Service
```bash
cd booking-service
pip install -r requirements.txt
python app.py
```

#### 4. Notification Service
```bash
cd notification-service
pip install -r requirements.txt
python app.py
```

### Testing the Application

1. **Seed Test Data**
   ```bash
   cd test
   python seed_events.py
   ```

2. **Run Integration Test**
   ```bash
   cd test
   python test_booking_flow.py
   ```

## API Documentation

### User Service (Port 8080)

- `POST /users` - Register a new user
- `POST /token` - Login and get access token
- `GET /users/me` - Get current user profile
- `GET /events` - Get all events (proxies to Event Service)
- `POST /bookings` - Create a new booking (proxies to Booking Service)

### Event Service (Port 8081)

- `GET /events` - Get all events
- `GET /events/{id}` - Get a specific event
- `GET /events/{id}/availability` - Check ticket availability
- `POST /events` - Create a new event
- `PUT /events/{id}/book` - Update ticket availability after booking
- `PUT /events/{id}` - Update an event
- `DELETE /events/{id}` - Delete an event

### Booking Service (Port 8082)

- `POST /bookings` - Create a new booking
- `GET /bookings/{id}` - Get a specific booking
- `GET /bookings/user/{user_id}` - Get all bookings for a user
- `PUT /bookings/{id}/cancel` - Cancel a booking

### Notification Service (Port 8083)

- `GET /notifications` - Get all notifications
- `GET /notifications/test` - Send a test notification

## Implementation Details

This implementation follows these best practices:

1. **Error Handling**
   - Comprehensive try-catch blocks to handle exceptions
   - Proper HTTP status codes and error messages

2. **Logging**
   - Each microservice logs activities to its own log file
   - Includes service name, function, and timestamp

3. **Environment Variables**
   - Configuration settings are loaded from environment variables
   - Sensitive information like database credentials is not hardcoded

4. **API Documentation**
   - Well-defined endpoints with clear request/response formats

5. **Asynchronous Communication**
   - RabbitMQ for reliable asynchronous notifications

6. **Resilience**
   - Services handle unavailability of dependent services
   - Transactions are rolled back if dependent operations fail
