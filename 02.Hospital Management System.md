# **Hospital Management System**  

- This project builds a **Hospital Management System** using Java and a database, allowing users to manage patients, doctors, appointments, and medical records while ensuring data integrity and security.  

## **Project Structure**  
```
com.example.hospitalmanagement/
├── dao/
│   ├── PatientDAO.java
│   ├── DoctorDAO.java
│   ├── AppointmentDAO.java
│   ├── MedicalRecordDAO.java
├── model/
│   ├── Patient.java
│   ├── Doctor.java
│   ├── Appointment.java
│   ├── MedicalRecord.java
├── util/
│   └── DatabaseConnection.java
├── service/
│   ├── PatientService.java
│   ├── DoctorService.java
│   ├── AppointmentService.java
│   ├── MedicalRecordService.java
├── exception/
│   └── DatabaseException.java
└── main/
    └── HospitalManagementApp.java
```

## **Database Schema with Relationships**  

```sql
-- Patients table
CREATE TABLE patients (
    patient_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    phone VARCHAR(15),
    date_of_birth DATE,
    address VARCHAR(200)
);

-- Doctors table
CREATE TABLE doctors (
    doctor_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    specialization VARCHAR(100),
    experience INT NOT NULL
);

-- Appointments table (junction table for many-to-many relationship)
CREATE TABLE appointments (
    appointment_id INT PRIMARY KEY AUTO_INCREMENT,
    patient_id INT NOT NULL,
    doctor_id INT NOT NULL,
    appointment_date TIMESTAMP NOT NULL,
    status ENUM('Scheduled', 'Completed', 'Cancelled') DEFAULT 'Scheduled',
    FOREIGN KEY (patient_id) REFERENCES patients(patient_id),
    FOREIGN KEY (doctor_id) REFERENCES doctors(doctor_id),
    UNIQUE KEY (patient_id, doctor_id, appointment_date) -- Prevents duplicate appointments
);

-- Medical Records table
CREATE TABLE medical_records (
    record_id INT PRIMARY KEY AUTO_INCREMENT,
    patient_id INT NOT NULL,
    doctor_id INT NOT NULL,
    diagnosis TEXT NOT NULL,
    treatment TEXT NOT NULL,
    date TIMESTAMP NOT NULL,
    FOREIGN KEY (patient_id) REFERENCES patients(patient_id),
    FOREIGN KEY (doctor_id) REFERENCES doctors(doctor_id)
);
```

## **Relationship Types Implemented**  

1. **One-to-Many**: Doctor to Medical Records  
   - One doctor can create multiple medical records  
   - Each record belongs to one doctor  

2. **Many-to-Many**: Patients to Doctors (through Appointments Table)  
   - Patients can book appointments with multiple doctors  
   - Doctors can have multiple patients  
   - The Appointments table serves as a junction table  

## **Updated Package Details**  

### **1. Model Classes**  
- **Patient.java**: Stores patient details including contact information and medical history  
- **Doctor.java**: Represents doctors with their specialization and experience  
- **Appointment.java**: Tracks scheduled appointments between patients and doctors  
- **MedicalRecord.java**: Stores medical history of patients and treatments provided  

### **2. DAO Layer**  
- **PatientDAO.java**: CRUD operations for patient data  
- **DoctorDAO.java**: CRUD operations for doctor data  
- **AppointmentDAO.java**: Manages appointment scheduling, including:  
  - `bookAppointment(int patientId, int doctorId, Timestamp appointmentDate)`  
  - `cancelAppointment(int appointmentId)`  
  - `getAppointmentsForPatient(int patientId)`  
  - `getAppointmentsForDoctor(int doctorId)`  
- **MedicalRecordDAO.java**: Manages medical records, including:  
  - `addMedicalRecord(int patientId, int doctorId, String diagnosis, String treatment, Timestamp date)`  
  - `getMedicalHistory(int patientId)`  

### **3. Service Layer**  
- **PatientService.java**: Handles patient registration and appointment history  
- **DoctorService.java**: Manages doctor availability and specialization details  
- **AppointmentService.java**: Enforces appointment scheduling policies, including:  
  - Prevents double booking  
  - Allows cancellation before a set deadline  
- **MedicalRecordService.java**: Handles secure medical record management and retrieval  

## **Implementation Details for Relationships**  

### **Join Operations**  
- Implements SQL JOIN operations to fetch related data:  
  ```java
  // Example methods in DAO classes
  List<Appointment> getAppointmentsWithDoctors();
  List<MedicalRecord> getMedicalRecordsWithDoctorInfo();
  ```

### **Cascading Operations**  
- Implements cascading delete for certain relationships:  
  - When deleting a patient, ensure medical records are archived  
  - When deleting a doctor, reassign active appointments  

### **Transaction Management**  
- Ensures data integrity across related tables:  
  ```java
  // Example appointment transaction
  connection.setAutoCommit(false);
  try {
      // Insert appointment record
      // Update doctor's availability
      connection.commit();
  } catch (SQLException e) {
      connection.rollback();
      throw new DatabaseException("Appointment booking failed", e);
  } finally {
      connection.setAutoCommit(true);
  }
  ```

### **Reporting Queries**  

- Implementation of complex queries spanning multiple tables:  
  - List doctors with the highest number of appointments  
  - Find patients with the most frequent visits  
  - Identify common diagnoses across patients  
