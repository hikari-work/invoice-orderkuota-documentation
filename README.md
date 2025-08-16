# Invoice Management System - OrderKuota Integration

## 📋 Overview

This application is an invoice management system integrated with OrderKuota, built using Java Spring Boot 3. It simplifies invoice management for transactions made through OrderKuota.

## ✨ Features

- **🔐 Login**: Secure authentication using OrderKuota registered accounts
- **💳 Transaction History**: View and manage transaction mutations
- **📄 Invoice Management**: Create, manage, and view invoices from transactions
- **📱 QRIS Generator**: Generate QRIS codes for easy invoice payments
- **🏦 Bank Account Verification**: Check bank account validity
- **🔔 Notifications**: Receive status updates via callback URLs
- **⚙️ Transaction Settings**: Configure unique reference IDs for transactions

---

## 🚀 Deployment Guide

### Prerequisites

- **Java**: OpenJDK 21 or equivalent (Oracle JDK 21, Amazon Corretto 21)
- **Build Tool**: Apache Maven 3.9.0 or newer
- **Database**: MySQL 8.0 or newer
- **Server**: VPS with Java and Maven compatibility
- **API Access**: OrderKuota account with API Key and Secret Key
- **Optional**: Web server for callback data reception

### Installation Steps

#### 1. Get the Code
```bash
# Clone repository or download as ZIP
git clone <repository-url>
cd invoice-management-system
```

#### 2. Configure Application
Edit `src/main/resources/application.properties`:

```properties
# Application Settings
spring.application.name=Integerasi Order Kuota
server.port=8080

# Database Configuration
spring.datasource.url=jdbc:mysql://localhost:3306/your_database_name
spring.datasource.username=your_username
spring.datasource.password=your_password

# JPA Settings
spring.jpa.hibernate.ddl-auto=update
spring.web.resources.add-mappings=false

# Connection Pool
spring.datasource.hikari.minimum-idle=10
spring.datasource.hikari.maximum-pool-size=100

# Application Config
application.config.random.reff.id=false
```

#### Configuration Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `server.port` | Application port | 8080 |
| `spring.datasource.url` | MySQL connection URL | - |
| `spring.datasource.username` | Database username | - |
| `spring.datasource.password` | Database password | - |
| `spring.jpa.hibernate.ddl-auto` | Database schema management | update |
| `spring.datasource.hikari.minimum-idle` | Minimum idle connections | 10 |
| `spring.datasource.hikari.maximum-pool-size` | Maximum connections | 100 |
| `application.config.random.reff.id` | Randomize reference IDs | false |

#### 3. Build and Run

```bash
# Install dependencies
mvn clean install

# Build JAR file
mvn package

# Run application
java -jar target/invoice-management-system.jar
```

**Docker Alternative:**
```bash
docker build -t invoice-management-system .
docker run -p 8080:8080 invoice-management-system
```

---

## 📚 API Documentation

> **⚠️ Note**: Direct API usage is not recommended. Please use the official application instead.

### Base URL
```
http://your-server:8080/api/v2
```

### Authentication Flow

#### 1. User Authentication
**Endpoint:** `POST /users/auth`

**Request:**
```json
{
  "username": "string",
  "password": "string"
}
```

**Responses:**

✅ **Success** (200):
```json
{
  "status": null,
  "data": {
    "email": {
      "status": "Success",
      "data": {
        "email": "user@example.com"
      }
    }
  }
}
```

❌ **Invalid Credentials** (200):
```json
{
  "status": null,
  "data": {
    "email": {
      "status": "Error",
      "data": {
        "error": "Username or password Wrong"
      }
    }
  }
}
```

#### 2. OTP Verification
**Endpoint:** `POST /users/auth/otp`

**Request:**
```json
{
  "username": "string",
  "otp": "string"
}
```

**Responses:**

✅ **Success** (200):
```json
{
  "status": "OK",
  "data": {
    "token": "d52dae4a-615b-4d72-ac7b-d63b4677a38c",
    "username": "viandrastefani",
    "email": "user@example.com"
  }
}
```

❌ **Invalid OTP** (400):
```json
{
  "status": "Error",
  "data": {
    "error": "OTP was wrong"
  }
}
```

### User Management

#### 3. Get User Details
**Endpoint:** `GET /users/details`

**Headers:**
```http
Authorization: Bearer <token>
```

**Response:**
```json
{
  "username": "viandrastefani",
  "email": "user@example.com",
  "token": "d52dae4a-615b-4d72-ac7b-d63b4677a38c",
  "qrcode": "data:image/png;base64,iVBO....",
  "callback_url": null,
  "qris_string": "000201010...6304A0D7"
}
```

#### 4. Update User
**Endpoint:** `POST /users/update`

**Headers:**
```http
Authorization: Bearer <token>
```

**Request:**
```json
{
  "username": "string",        // Required
  "email": "string",           // Optional
  "callback_url": "string"     // Optional
}
```

### Transaction Management

#### 5. Get Transaction History
**Endpoint:** `GET /mutasi/{username}`

**Headers:**
```http
Authorization: Bearer <token>
Content-Type: application/json
```

**Query Parameters:**
- `page`: Page number (default: 0)
- `size`: Items per page (default: 10)

**Response:**
```json
{
  "page": 0,
  "size": 10,
  "data": [
    {
      "id": 162052951,
      "username": "viandrastefani",
      "debet": 0,
      "kredit": 15000,
      "keterangan": "Payment description",
      "status": "IN",
      "statement_status": "NOT_CLAIMED",
      "transfer_time": "2025-08-01T00:56:00"
    }
  ],
  "total_content": 21,
  "total_pages": 3,
  "has_next": true,
  "has_previous": false,
  "total_data": 10
}
```

### Invoice Management

#### 6. Create Invoice
**Endpoint:** `POST /invoices/create`

**Headers:**
```http
Authorization: Bearer <token>
Content-Type: application/json
```

**Request:**
```json
{
  "notes": "Payment for services",
  "amount": 10,
  "expires_at": 3600  // Expiry time in seconds
}
```

**Response:**
```json
{
  "username": "viandrastefani",
  "amount": 10,
  "status": "PENDING",
  "note": "Payment for services",
  "expires_at": 1755001134825,
  "created_at": "2025-08-12T12:18:51.825423576",
  "qris_string": "00020101021226670016COM.NOBUBANK.WWW...",
  "invoice_id": "ad411c75-f6e3-49cc-b92a-195e9afd1a7b"
}
```

#### 7. Get Invoice Details
**Endpoint:** `GET /invoices/details/{invoice_id}`

**Headers:**
```http
Authorization: Bearer <token>
```

**Response:**
```json
{
  "username": "viandrastefani",
  "amount": 10,
  "status": "PENDING",
  "note": "Payment for services",
  "expires_at": 1754820766998,
  "created_at": "2025-08-10T10:12:43.998006",
  "qris_string": "00020101021226670016COM.NOBUBANK.WWW...",
  "invoice_id": "6f467369-09d3-4d20-a70b-7ebcaed0bfb8"
}
```

**Invoice Status:**
- `PENDING`: Invoice awaiting payment
- `PAID`: Invoice has been paid
- `EXPIRED`: Invoice has expired

#### 8. Generate QRIS Code
**Endpoint:** `GET /invoices/qris/{invoice_id}`

**Headers:**
```http
Authorization: Bearer <token>
Accept: image/png
```

**Query Parameters:**
- `height`: Image height in pixels (default: 1080)
- `width`: Image width in pixels (default: 1080)

**Response:** PNG image file

### Webhook Callbacks

The system sends POST requests to your configured callback URL when invoice status changes.

#### Payment Confirmation
```json
{
  "id": "6434e9b7-7ba6-4160-b9c3-030529dda148",
  "status": "PAID",
  "amount": 101,
  "note": "Payment description",
  "created_at": "2025-08-10T01:23:37.463178",
  "paid_at": 1754763948300
}
```

#### Expiration Notice
```json
{
  "id": "6434e9b7-7ba6-4160-b9c3-030529dda148",
  "status": "EXPIRED",
  "amount": 101,
  "note": "Payment description",
  "created_at": "2025-08-10T01:23:37.463178",
  "expired_at": 1754763948300
}
```

### Error Responses

#### 401 Unauthorized
```json
{
  "message": "Token Invalid"
}
```

#### 404 Not Found
```json
{
  "timestamp": "2025-08-12T19:24:28.6993629",
  "status": 404,
  "error": "Not Found",
  "message": "Invoice Not Found",
  "path": "/api/v2/invoices/details/invalid-id"
}
```

#### 400 Bad Request
```json
{
  "status": "Error",
  "data": {
    "email": "Invalid email format"
  }
}
```

---

## 🔧 Configuration Tips

1. **Database Setup**: Ensure MySQL is running and accessible
2. **Security**: Use strong database credentials
3. **Performance**: Adjust connection pool settings based on load
4. **Monitoring**: Set up logging for production environments
5. **Backup**: Regularly backup your database

## 📞 Support

For additional support and configuration assistance, please refer to the official OrderKuota documentation or contact the development team.
