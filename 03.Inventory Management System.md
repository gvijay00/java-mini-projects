# **Inventory Management System**  

- This project builds an **Inventory Management System** using Java and a database, allowing users to manage products, suppliers, stock levels, and sales transactions while ensuring data integrity and security.  

## **Project Structure**  
```
com.example.inventorymanagement/
├── dao/
│   ├── ProductDAO.java
│   ├── SupplierDAO.java
│   ├── StockDAO.java
│   ├── SaleDAO.java
├── model/
│   ├── Product.java
│   ├── Supplier.java
│   ├── Stock.java
│   ├── Sale.java
├── util/
│   └── DatabaseConnection.java
├── service/
│   ├── ProductService.java
│   ├── SupplierService.java
│   ├── StockService.java
│   ├── SaleService.java
├── exception/
│   └── DatabaseException.java
└── main/
    └── InventoryManagementApp.java
```

## **Database Schema with Relationships**  

```sql
-- Products table
CREATE TABLE products (
    product_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    category VARCHAR(100),
    price DECIMAL(10,2) NOT NULL,
    supplier_id INT,
    FOREIGN KEY (supplier_id) REFERENCES suppliers(supplier_id)
);

-- Suppliers table
CREATE TABLE suppliers (
    supplier_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    contact VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL
);

-- Stock table
CREATE TABLE stock (
    stock_id INT PRIMARY KEY AUTO_INCREMENT,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

-- Sales table
CREATE TABLE sales (
    sale_id INT PRIMARY KEY AUTO_INCREMENT,
    product_id INT NOT NULL,
    quantity_sold INT NOT NULL,
    sale_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    total_price DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```

## **Relationship Types Implemented**  

1. **One-to-Many**: Supplier to Products  
   - One supplier can provide multiple products  
   - Each product belongs to one supplier  

2. **One-to-One**: Product to Stock  
   - Each product has one stock record  
   - Each stock record belongs to one product  

3. **Many-to-Many**: Products to Sales  
   - Products can be sold multiple times  
   - Sales transactions contain multiple products  

## **Updated Package Details**  

### **1. Model Classes**  
- **Product.java**: Stores product details including category, price, and supplier reference  
- **Supplier.java**: Represents suppliers providing inventory products  
- **Stock.java**: Tracks product availability and last updated timestamp  
- **Sale.java**: Records sales transactions, including quantity sold and total price  

### **2. DAO Layer**  
- **ProductDAO.java**: CRUD operations for product data  
- **SupplierDAO.java**: CRUD operations for supplier data  
- **StockDAO.java**: Manages stock levels, including:  
  - `updateStock(int productId, int newQuantity)`  
  - `getStockForProduct(int productId)`  
- **SaleDAO.java**: Manages sales transactions, including:  
  - `recordSale(int productId, int quantity, double totalPrice)`  
  - `getSalesForProduct(int productId)`  
  - `getTotalRevenue()`  

### **3. Service Layer**  
- **ProductService.java**: Handles product catalog and supplier assignments  
- **SupplierService.java**: Manages supplier registration and products supplied  
- **StockService.java**: Ensures stock levels are updated after sales and restocking  
- **SaleService.java**: Handles sales processing, including:  
  - Prevents selling products with insufficient stock  
  - Updates stock levels after each sale  

## **Implementation Details for Relationships**  

### **Join Operations**  
- Implements SQL JOIN operations to fetch related data:  
  ```java
  // Example methods in DAO classes
  List<Product> getProductsWithSuppliers();
  List<Sale> getSalesWithProductDetails();
  ```

### **Cascading Operations**  
- Implements cascading delete for certain relationships:  
  - When deleting a supplier, ensure related products are reassigned  
  - When deleting a product, manage existing stock and sales records  

### **Transaction Management**  
- Ensures data integrity across related tables:  
  ```java
  // Example sales transaction
  connection.setAutoCommit(false);
  try {
      // Insert sale record
      // Update stock quantity
      connection.commit();
  } catch (SQLException e) {
      connection.rollback();
      throw new DatabaseException("Sale transaction failed", e);
  } finally {
      connection.setAutoCommit(true);
  }
  ```

### **Reporting Queries**  

- Implementation of complex queries spanning multiple tables:  
  - List top-selling products  
  - Find suppliers providing the most products  
  - Calculate total revenue over a given period  

