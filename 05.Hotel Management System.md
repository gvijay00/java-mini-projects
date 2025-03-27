

# **Hotel Management System**  

- This project builds a **Hotel Management System** using Java and a database, allowing users to manage hotel rooms, guests, staff, bookings, and payments efficiently while ensuring data integrity and security.  

## **Project Structure**  
```
com.example.hotelmanagement/
├── dao/
│   ├── RoomDAO.java
│   ├── GuestDAO.java
│   ├── StaffDAO.java
│   ├── BookingDAO.java
│   ├── PaymentDAO.java
├── model/
│   ├── Room.java
│   ├── Guest.java
│   ├── Staff.java
│   ├── Booking.java
│   ├── Payment.java
├── util/
│   └── DatabaseConnection.java
├── service/
│   ├── RoomService.java
│   ├── GuestService.java
│   ├── StaffService.java
│   ├── BookingService.java
│   ├── PaymentService.java
├── exception/
│   └── DatabaseException.java
└── main/
    └── HotelManagementApp.java
```

## **Database Schema with Relationships**  
```sql
-- Rooms table
CREATE TABLE rooms (
    room_id INT PRIMARY KEY AUTO_INCREMENT,
    room_number VARCHAR(10) UNIQUE NOT NULL,
    type VARCHAR(50) NOT NULL,
    price_per_night DECIMAL(8,2) NOT NULL,
    status VARCHAR(20) DEFAULT 'Available'
);

-- Guests table
CREATE TABLE guests (
    guest_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    phone VARCHAR(15),
    checkin_date DATE,
    checkout_date DATE
);

-- Staff table
CREATE TABLE staff (
    staff_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    role VARCHAR(50) NOT NULL,
    phone VARCHAR(15),
    hire_date DATE
);

-- Bookings table
CREATE TABLE bookings (
    booking_id INT PRIMARY KEY AUTO_INCREMENT,
    guest_id INT NOT NULL,
    room_id INT NOT NULL,
    checkin_date DATE NOT NULL,
    checkout_date DATE NOT NULL,
    status VARCHAR(20) DEFAULT 'Booked',
    FOREIGN KEY (guest_id) REFERENCES guests(guest_id),
    FOREIGN KEY (room_id) REFERENCES rooms(room_id)
);

-- Payments table
CREATE TABLE payments (
    payment_id INT PRIMARY KEY AUTO_INCREMENT,
    booking_id INT NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    payment_date DATE NOT NULL,
    payment_method VARCHAR(50),
    FOREIGN KEY (booking_id) REFERENCES bookings(booking_id)
);
```

## **Relationship Types Implemented**  
1. **One-to-Many**:  
   - A guest can book multiple rooms over time.  
   - A staff member can handle multiple bookings.  

2. **Many-to-Many**:  
   - Guests and rooms are linked through the **Bookings** table.  
   - Payments are linked to bookings, ensuring a structured financial record.  

## **Updated Package Details**  

### **1. Model Classes**  
- **Room.java**: Stores room details like type, price, and status.  
- **Guest.java**: Stores guest information and check-in/check-out dates.  
- **Staff.java**: Represents hotel staff members, such as receptionists and housekeeping.  
- **Booking.java**: Tracks room bookings with status updates.  
- **Payment.java**: Manages payments linked to bookings.  

### **2. DAO Layer**  
- **RoomDAO.java**: CRUD operations for rooms.  
- **GuestDAO.java**: CRUD operations for guests.  
- **StaffDAO.java**: CRUD operations for staff.  
- **BookingDAO.java**: Manages room bookings, including:  
  - `bookRoom(int guestId, int roomId, Date checkinDate, Date checkoutDate)`  
  - `updateBookingStatus(int bookingId, String status)`  
  - `getBookingsForGuest(int guestId)`  
  - `getAvailableRooms()`  

- **PaymentDAO.java**: Manages guest payments, including:  
  - `processPayment(int bookingId, double amount, String paymentMethod)`  
  - `getPaymentDetails(int guestId)`  

### **3. Service Layer**  
- **RoomService.java**: Manages room availability and pricing.  
- **GuestService.java**: Handles guest check-in, check-out, and information updates.  
- **StaffService.java**: Manages staff roles and assignments.  
- **BookingService.java**: Enforces booking rules, including:  
  - Checking room availability.  
  - Preventing duplicate bookings.  
  - Managing check-in and check-out processes.  
- **PaymentService.java**: Handles financial transactions and ensures proper billing.  

## **Implementation Details for Relationships**  

### **Join Operations**  
- Implements SQL JOIN operations to fetch related data:  
  ```java
  // Example methods in DAO classes
  List<Booking> getBookingsWithGuests();
  List<Room> getAvailableRoomsWithPricing();
  ```  

### **Cascading Operations**  
- Ensures proper management of guest stays and payments:  
  - Prevent deletion of a room with active bookings.  
  - Prevent deletion of guests with unpaid bookings.  

### **Transaction Management**  
- Ensures data consistency when booking a room:  
  ```java
  // Example booking transaction
  connection.setAutoCommit(false);
  try {
      // Insert booking record
      // Update room status
      connection.commit();
  } catch (SQLException e) {
      connection.rollback();
      throw new DatabaseException("Booking transaction failed", e);
  } finally {
      connection.setAutoCommit(true);
  }
  ```  

### **Reporting Queries**  
- Implementation of complex queries spanning multiple tables:  
  - List most booked rooms.  
  - Find highest-paying guests.  
  - Track pending and completed payments.  

