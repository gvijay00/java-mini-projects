# **Transport Management System**  

- This project builds a **Transport Management System** using Java and a database, allowing users to manage vehicles, drivers, routes, and trip records efficiently while ensuring data integrity and security.  

## **Project Structure**  
```
com.example.transportmanagement/
├── dao/
│   ├── VehicleDAO.java
│   ├── DriverDAO.java
│   ├── RouteDAO.java
│   ├── TripDAO.java
├── model/
│   ├── Vehicle.java
│   ├── Driver.java
│   ├── Route.java
│   ├── Trip.java
├── util/
│   └── DatabaseConnection.java
├── service/
│   ├── VehicleService.java
│   ├── DriverService.java
│   ├── RouteService.java
│   ├── TripService.java
├── exception/
│   └── DatabaseException.java
└── main/
    └── TransportManagementApp.java
```

## **Database Schema with Relationships**  
```sql
-- Vehicles table
CREATE TABLE vehicles (
    vehicle_id INT PRIMARY KEY AUTO_INCREMENT,
    registration_number VARCHAR(20) UNIQUE NOT NULL,
    type VARCHAR(50) NOT NULL,
    capacity INT NOT NULL,
    status VARCHAR(50) DEFAULT 'Available'
);

-- Drivers table
CREATE TABLE drivers (
    driver_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    license_number VARCHAR(50) UNIQUE NOT NULL,
    phone VARCHAR(15),
    experience INT
);

-- Routes table
CREATE TABLE routes (
    route_id INT PRIMARY KEY AUTO_INCREMENT,
    start_location VARCHAR(100) NOT NULL,
    end_location VARCHAR(100) NOT NULL,
    distance_km DECIMAL(5,2) NOT NULL
);

-- Trips table
CREATE TABLE trips (
    trip_id INT PRIMARY KEY AUTO_INCREMENT,
    vehicle_id INT NOT NULL,
    driver_id INT NOT NULL,
    route_id INT NOT NULL,
    trip_date DATE NOT NULL,
    status VARCHAR(50) DEFAULT 'Scheduled',
    FOREIGN KEY (vehicle_id) REFERENCES vehicles(vehicle_id),
    FOREIGN KEY (driver_id) REFERENCES drivers(driver_id),
    FOREIGN KEY (route_id) REFERENCES routes(route_id)
);
```

## **Relationship Types Implemented**  
1. **One-to-Many**:  
   - A driver can handle multiple trips.  
   - A vehicle can be used for multiple trips.  

2. **Many-to-Many**:  
   - Drivers and vehicles are assigned dynamically for each trip.  
   - Routes are used by multiple vehicles in different trips.  

## **Updated Package Details**  

### **1. Model Classes**  
- **Vehicle.java**: Stores vehicle details like registration number, type, and status.  
- **Driver.java**: Stores driver details, including license number and experience.  
- **Route.java**: Represents predefined routes with start and end locations.  
- **Trip.java**: Tracks vehicle, driver, and route details for a specific journey.  

### **2. DAO Layer**  
- **VehicleDAO.java**: CRUD operations for vehicles.  
- **DriverDAO.java**: CRUD operations for drivers.  
- **RouteDAO.java**: CRUD operations for routes.  
- **TripDAO.java**: Manages trip scheduling, including:  
  - `scheduleTrip(int vehicleId, int driverId, int routeId, Date tripDate)`  
  - `updateTripStatus(int tripId, String status)`  
  - `getTripsForDriver(int driverId)`  
  - `getTripsForVehicle(int vehicleId)`  

### **3. Service Layer**  
- **VehicleService.java**: Manages vehicle assignments and availability.  
- **DriverService.java**: Handles driver allocation and licensing requirements.  
- **RouteService.java**: Manages route definitions and distances.  
- **TripService.java**: Enforces business rules for trip scheduling, including:  
  - Ensuring vehicle and driver availability.  
  - Preventing trip duplication.  
  - Managing trip completion updates.  

## **Implementation Details for Relationships**  

### **Join Operations**  
- Implements SQL JOIN operations to fetch related data:  
  ```java
  // Example methods in DAO classes
  List<Trip> getTripsWithDetails();
  List<Driver> getDriversWithTrips();
  ```  

### **Cascading Operations**  
- Ensures data integrity when deleting records:  
  - Prevent deletion of vehicles assigned to active trips.  
  - Prevent deletion of drivers with scheduled trips.  

### **Transaction Management**  
- Ensures data consistency during trip scheduling:  
  ```java
  // Example transaction
  connection.setAutoCommit(false);
  try {
      // Assign vehicle and driver
      // Insert trip record
      connection.commit();
  } catch (SQLException e) {
      connection.rollback();
      throw new DatabaseException("Trip scheduling failed", e);
  } finally {
      connection.setAutoCommit(true);
  }
  ```  

### **Reporting Queries**  
- Implementation of complex queries spanning multiple tables:  
  - List most frequently used vehicles.  
  - Find drivers with the most completed trips.  
  - Track pending and completed trips.  
