# **Vehicle Rental System**  

- This project builds a **Vehicle Rental System** using Java and a database, allowing users to rent vehicles, manage customers, track payments, and handle vehicle availability efficiently.  

## **Project Structure**  
```
com.example.vehiclerental/
├── dao/
│   ├── VehicleDAO.java
│   ├── CustomerDAO.java
│   ├── RentalDAO.java
│   ├── PaymentDAO.java
├── model/
│   ├── Vehicle.java
│   ├── Customer.java
│   ├── Rental.java
│   ├── Payment.java
├── util/
│   └── DatabaseConnection.java
├── service/
│   ├── VehicleService.java
│   ├── CustomerService.java
│   ├── RentalService.java
│   ├── PaymentService.java
├── exception/
│   └── DatabaseException.java
└── main/
    └── VehicleRentalApp.java
```

## **Database Schema with Relationships**  
```sql
-- Vehicles table
CREATE TABLE vehicles (
    vehicle_id INT PRIMARY KEY AUTO_INCREMENT,
    model VARCHAR(100) NOT NULL,
    type VARCHAR(50) NOT NULL,
    registration_number VARCHAR(20) UNIQUE NOT NULL,
    daily_rental_price DECIMAL(8,2) NOT NULL,
    status VARCHAR(20) DEFAULT 'Available'
);

-- Customers table
CREATE TABLE customers (
    customer_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    phone VARCHAR(15),
    license_number VARCHAR(50) UNIQUE NOT NULL
);

-- Rentals table
CREATE TABLE rentals (
    rental_id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT NOT NULL,
    vehicle_id INT NOT NULL,
    rental_date DATE NOT NULL,
    return_date DATE,
    status VARCHAR(20) DEFAULT 'Rented',
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
    FOREIGN KEY (vehicle_id) REFERENCES vehicles(vehicle_id)
);

-- Payments table
CREATE TABLE payments (
    payment_id INT PRIMARY KEY AUTO_INCREMENT,
    rental_id INT NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    payment_date DATE NOT NULL,
    payment_method VARCHAR(50),
    FOREIGN KEY (rental_id) REFERENCES rentals(rental_id)
);
```

## **Relationship Types Implemented**  
1. **One-to-Many**:  
   - A customer can rent multiple vehicles over time.  
   - A vehicle can have multiple rentals (over time).  

2. **Many-to-Many**:  
   - Customers and vehicles are linked through the **Rentals** table.  
   - Payments are linked to rentals for transaction tracking.  

## **Updated Package Details**  

### **1. Model Classes**  
- **Vehicle.java**: Stores vehicle details like model, type, registration, and status.  
- **Customer.java**: Stores customer details including license number.  
- **Rental.java**: Tracks vehicle rentals with status updates.  
- **Payment.java**: Manages payments linked to rentals.  

### **2. DAO Layer**  
- **VehicleDAO.java**: CRUD operations for vehicles.  
- **CustomerDAO.java**: CRUD operations for customers.  
- **RentalDAO.java**: Manages rental operations, including:  
  - `rentVehicle(int customerId, int vehicleId, Date rentalDate, Date returnDate)`  
  - `returnVehicle(int rentalId)`  
  - `getRentalsForCustomer(int customerId)`  
  - `getAvailableVehicles()`  

- **PaymentDAO.java**: Manages payments, including:  
  - `processPayment(int rentalId, double amount, String paymentMethod)`  
  - `getPaymentDetails(int customerId)`  

### **3. Service Layer**  
- **VehicleService.java**: Manages vehicle availability and rental rates.  
- **CustomerService.java**: Handles customer registration and validation.  
- **RentalService.java**: Enforces rental rules, including:  
  - Checking vehicle availability.  
  - Preventing duplicate rentals.  
  - Managing returns.  
- **PaymentService.java**: Handles financial transactions and ensures proper billing.  

## **Implementation Details for Relationships**  

### **Join Operations**  
- Implements SQL JOIN operations to fetch related data:  
  ```java
  // Example methods in DAO classes
  List<Rental> getRentalsWithCustomers();
  List<Vehicle> getAvailableVehicles();
  ```  

### **Cascading Operations**  
- Ensures proper handling of rentals and payments:  
  - Prevent deletion of a vehicle with active rentals.  
  - Prevent deletion of customers with unpaid rentals.  

### **Transaction Management**  
- Ensures data consistency when renting a vehicle:  
  ```java
  // Example rental transaction
  connection.setAutoCommit(false);
  try {
      // Insert rental record
      // Update vehicle status
      connection.commit();
  } catch (SQLException e) {
      connection.rollback();
      throw new DatabaseException("Rental transaction failed", e);
  } finally {
      connection.setAutoCommit(true);
  }
  ```  

### **Reporting Queries**  
- Implementation of complex queries spanning multiple tables:  
  - List most rented vehicles.  
  - Find top customers by total rentals.  
  - Track pending and completed payments.  

