## Little Lemon Restaurant Database Project Overview:

Database Setup:

Establish the initial parameters and configurations for the database.
Information to be Stored:

Define the data that needs to be stored in the database, including entities such as customers, orders, menus, menu items, reservations, etc.
Main Entities and Related Attributes:

Identify the primary entities in the database (e.g., Customers, Orders, Menus) and their related attributes.
Normalization Process:

Apply the normalization process to efficiently organize information and reduce redundancy in the database.
Final Database Design:

Create the final design of the database, including tables, relationships, and constraints.
Implementation in MySQL Server:

Set up the database and tables according to the final design on a MySQL server.
Report Generation:

Generate reports that include information about restaurant orders, customers with specific orders, and the best-selling menu items.
Query Optimization:

Improve query efficiency, such as displaying the maximum ordered quantity in the Orders table and providing detailed information about a specific order.
Delete a Specific Order:

Develop a query to delete a specific order based on its identifier.
Check Table Booking Status:

Create queries to check the booking status of a table in the restaurant.
Verify and Decline Conflicting Bookings:

Implement queries to verify reservations and decline those that are already booked under another name.
Update Existing Bookings:

Develop queries to update information in the reservations table, such as changing the associated name for an existing reservation.
Cancel a Reservation:

Create queries to cancel a reservation in the database.
Database Client Creation:

Implement and configure database clients capable of interacting with the database, executing queries, and performing specific operations.

## Information Required to be Stored:

Bookings:

Store details about booked tables, including booking ID, date, and table number.
Orders:

Store information about each order, including order date, quantity, and total cost.
Order Delivery Status:

Store details about the delivery status of each order, including delivery date and status.
Menu:

Store information about various menu items, including cuisines, starters, courses, drinks, and desserts.
Customer Details:

Store customer information, including names and contact details.
Staff Information:

Store details about staff members, including their roles and salaries.

## Main entites and related attributes
Customers:

PK → CustomerID
Attributes → CustomerName, ContactDetails
Staff:

PK → Staff ID
Attributes → StaffName, Role, Salary
Mene Items:

PK → MenuID
Attributes → Cuisine, Starters, Courses, Drinks, Desserts

PK → Booking ID
Attributes → Date, TableNumber, CustomerID, StaffInformation_StaffID

PK → OrderID
Attributes → OrderDate, Quantity, TotalCost, MenuID, CustomerID

PK → DeliveryID
Attributes → DeliveryDate, Status, Orders_OrderID

## Final Design 
![image](https://github.com/FranciscoRubioF/db-capstone-project/assets/148911432/29ead0e8-1dde-4427-9f2d-c60358534023)
## Forward Engineering

DROP TABLE IF EXISTS `bookings`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!50503 SET character_set_client = utf8mb4 */;
CREATE TABLE `bookings` (
  `BookingID` int NOT NULL,
  `Date` date NOT NULL,
  `TableNumber` int NOT NULL,
  `CustomerID` int NOT NULL,
  `StaffInformation_StaffID` int NOT NULL,
  PRIMARY KEY (`BookingID`),
  KEY `Customer_idx` (`CustomerID`),
  KEY `fk_Bookings_StaffInformation1_idx` (`StaffInformation_StaffID`),
  CONSTRAINT `Customer` FOREIGN KEY (`CustomerID`) REFERENCES `customerdetails` (`CustomerID`) ON DELETE CASCADE ON UPDATE CASCADE,
  CONSTRAINT `fk_Bookings_StaffInformation1` FOREIGN KEY (`StaffInformation_StaffID`) REFERENCES `staffinformation` (`StaffID`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb3;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `bookings`
--

LOCK TABLES `bookings` WRITE;
/*!40000 ALTER TABLE `bookings` DISABLE KEYS */;
/*!40000 ALTER TABLE `bookings` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `customerdetails`
--

DROP TABLE IF EXISTS `customerdetails`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!50503 SET character_set_client = utf8mb4 */;
CREATE TABLE `customerdetails` (
  `CustomerID` int NOT NULL,
  `CustomerName` varchar(255) NOT NULL,
  `ContactDetails` varchar(255) NOT NULL,
  PRIMARY KEY (`CustomerID`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb3;
/*!40101 SET character_set_client = @saved_cs_client */;

This is just a part.

## Creating reports
### Task 1.  Create a virtual table called OrdersView that focuses on OrderID, Quantity and Cost columns within the Orders table for all orders with a quantity greater than 2. 

CREATE VIEW OrdersView AS
SELECT OrderID, Quantity, TotalCost
FROM Orders
WHERE Quantity > 2;

![image](https://github.com/FranciscoRubioF/db-capstone-project/assets/148911432/16ca4a65-934e-4fcf-93ab-f830626a0980)


### Task 2. Little Lemon need information from four tables on all customers with orders that cost more than $150.
SELECT
    c.CustomerID,
    c.CustomerName,
    o.OrderID,
    o.TotalCost,
    m.menu_name,
    mi.Courses,
    mi.Starters
FROM
    Customers c
JOIN
    Orders o ON c.customer_id = o.customer_id
JOIN
    menu m ON o.menu_id = m.MenuID
JOIN
    MenuItems mi ON m.menu_id = mi.menu_id
WHERE
    o.TotalCost > 150
ORDER BY
    o.cost;

![image](https://github.com/FranciscoRubioF/db-capstone-project/assets/148911432/957ad5fd-aebe-429a-ba3c-dd6954b16740)
### Task 3. Little Lemon need you to find all menu items for which more than 2 orders have been placed. You can carry out this task by creating a subquery that lists the menu names from the menus table for any order quantity with more than 2.
SELECT
    menu_name
FROM
    Menus
WHERE
    menu_id IN (
        SELECT
            menu_id
        FROM
            Orders
        GROUP BY
            menu_id
        HAVING
            COUNT(*) > 2
    );
![image](https://github.com/FranciscoRubioF/db-capstone-project/assets/148911432/36682d70-cb75-4aac-a904-334a2ef2b6ba)
### task 4. Little Lemon need you to create a procedure that displays the maximum ordered quantity in the Orders table. 

DELIMITER //

CREATE PROCEDURE GetMaxQuantity(OUT max_quantity INT)
BEGIN
    SELECT MAX(quantity) INTO max_quantity
    FROM Orders;
END //

DELIMITER ;
![image](https://github.com/FranciscoRubioF/db-capstone-project/assets/148911432/ab8ed34b-79dc-419f-8db6-8bbf5f4e0137)

### task 5. Little Lemon need you to help them to create a prepared statement called GetOrderDetail. This prepared statement will help to reduce the parsing time of queries. It will also help to secure the database from SQL injections.

DELIMITER //

CREATE PROCEDURE GetOrderDetail(IN customer_id_value INT)
BEGIN
    SET @sql_query = CONCAT('
        SELECT
            order_id,
            quantity,
            cost
        FROM
            Orders
        WHERE
            customer_id = ?'
    );

    -- Prepare and execute the dynamic SQL statement
    PREPARE stmt FROM @sql_query;
    EXECUTE stmt USING customer_id_value;
    DEALLOCATE PREPARE stmt;
END //

DELIMITER ;

The GetOrderDetail procedure is created to accept an input parameter customer_id_value.
The CONCAT function is used to dynamically construct the SQL query with the provided customer ID.
The PREPARE statement prepares the dynamic SQL query.
The EXECUTE statement then executes the prepared statement using the input parameter.
Finally, the DEALLOCATE statement is used to release the prepared statement.

![image](https://github.com/FranciscoRubioF/db-capstone-project/assets/148911432/2b5e6733-0d70-4ae5-b6d1-2b57788d30be)

### Task 6. Create a stored procedure called CancelOrder. Little Lemon want to use this stored procedure to delete an order record based on the user input of the order id.

DELIMITER //

CREATE PROCEDURE CancelOrder(IN order_id_value INT)
BEGIN
    DECLARE message VARCHAR(255);

    DELETE FROM Orders WHERE order_id = order_id_value;

    IF ROW_COUNT() > 0 THEN
        SET message = CONCAT('Order ', order_id_value, ' is cancelled.');
    ELSE
        SET message = CONCAT('Order ', order_id_value, ' not found.');
    END IF;

    SELECT message AS cancellation_message;
END //
![image](https://github.com/FranciscoRubioF/db-capstone-project/assets/148911432/6810cfb1-e317-4d58-8a98-194cdc8dda5d)

DELIMITER ;
### Task 7. Little Lemon wants to populate the Bookings table of their database with some records of data.

-- Inserting record 1
INSERT INTO Bookings (BookingID, BookingDate, TableNumber, CustomerID)
VALUES (1, '2022-10-10', 5, 1);

-- Inserting record 2
INSERT INTO Bookings (BookingID, BookingDate, TableNumber, CustomerID)
VALUES (2, '2022-11-12', 3, 3);

-- Inserting record 3
INSERT INTO Bookings (BookingID, BookingDate, TableNumber, CustomerID)
VALUES (3, '2022-10-11', 2, 2);

-- Inserting record 4
INSERT INTO Bookings (BookingID, BookingDate, TableNumber, CustomerID)
VALUES (4, '2022-10-13', 2, 1);

![image](https://github.com/FranciscoRubioF/db-capstone-project/assets/148911432/0c479948-bb57-4dd6-bfdf-118627c4095e)

### task 8. Little Lemon need you to create a stored procedure called CheckBooking to check whether a table in the restaurant is already booked. Creating this procedure helps to minimize the effort involved in repeatedly coding the same SQL statements. The procedure should have two input parameters in the form of booking date and table number. You can also create a variable in the procedure to check the status of each table.

DELIMITER //

CREATE PROCEDURE CheckBooking(IN booking_date_value DATE, IN table_number_value INT)
BEGIN
    DECLARE booking_status VARCHAR(50);

    -- Check if the table is booked on the given date
    SELECT
        CASE
            WHEN COUNT(*) > 0 THEN 'Table is already booked'
            ELSE 'Table is available'
        END INTO booking_status
    FROM Bookings
    WHERE BookingDate = booking_date_value AND TableNumber = table_number_value;

    -- Output the result
    SELECT booking_status AS BookingStatus;
END //

DELIMITER ;

![image](https://github.com/FranciscoRubioF/db-capstone-project/assets/148911432/346d53ea-0ccf-4ad0-b227-e8489b7c629d)

 ### task 9.For your third and final task, Little Lemon need to verify a booking, and decline any reservations for tables that are already booked under another name. 
 DELIMITER //

CREATE PROCEDURE AddValidBooking(IN booking_date_value DATE, IN table_number_value INT)
BEGIN
    DECLARE booking_status INT;

    -- Start the transaction
    START TRANSACTION;

    -- Add a new booking record
    INSERT INTO Bookings (BookingDate, TableNumber)
    VALUES (booking_date_value, table_number_value);

    -- Check if the table is already booked on the given date
    SELECT COUNT(*) INTO booking_status
    FROM Bookings
    WHERE BookingDate = booking_date_value AND TableNumber = table_number_value;

    -- If the table is already booked, rollback the transaction
    IF booking_status > 1 THEN
        ROLLBACK;
        SELECT 'Booking declined. Table is already booked.' AS BookingStatus;
    ELSE
        -- If the table is available, commit the transaction
        COMMIT;
        SELECT 'Booking successful.' AS BookingStatus;
    END IF;
END //

DELIMITER ;
![image](https://github.com/FranciscoRubioF/db-capstone-project/assets/148911432/71a4f58f-3a8f-4a65-9478-7e08b2c65731)

### Task 10.create a new procedure called AddBooking to add a new table booking record.
DELIMITER //

CREATE PROCEDURE AddBooking(
    IN booking_id_value INT,
    IN customer_id_value INT,
    IN table_number_value INT,
    IN booking_date_value DATE
)
BEGIN
    -- Add a new booking record
    INSERT INTO Bookings (BookingID, CustomerID, TableNumber, BookingDate)
    VALUES (booking_id_value, customer_id_value, table_number_value, booking_date_value);

    -- Display confirmation message
    SELECT 'New booking added' AS Confirmation;
END //

DELIMITER ;
![image](https://github.com/FranciscoRubioF/db-capstone-project/assets/148911432/4d0f511a-84e2-4dce-bf00-1b4384a61b05)

### Task 11. create a new procedure called UpdateBooking that they can use to update existing bookings in the booking table.

DELIMITER //

CREATE PROCEDURE UpdateBooking(
    IN booking_id_value INT,
    IN new_booking_date_value DATE
)
BEGIN
    -- Update existing booking record
    UPDATE Bookings
    SET BookingDate = new_booking_date_value
    WHERE BookingID = booking_id_value;

    -- Display confirmation message
    SELECT CONCAT('Booking ', booking_id_value, ' updated') AS Confirmation;
END //

DELIMITER ;
![image](https://github.com/FranciscoRubioF/db-capstone-project/assets/148911432/a28a8699-c097-4b14-bea1-ec2e44fdf369)

### Task 12.create a new procedure called CancelBooking that they can use to cancel or remove a booking.
DELIMITER //

CREATE PROCEDURE CancelBooking(
    IN booking_id_value INT
)
BEGIN
    -- Delete booking record
    DELETE FROM Bookings
    WHERE BookingID = booking_id_value;

    -- Display confirmation message
    SELECT CONCAT('Booking ', booking_id_value, ' cancelled') AS Confirmation;
END //

DELIMITER ;
![image](https://github.com/FranciscoRubioF/db-capstone-project/assets/148911432/b6498f35-ac36-4998-ba8f-50e13390b1e2)

## Tableau

### task 1. create a bar chart that shows customers sales and filter data based on sales with at least $70.
![image](https://github.com/FranciscoRubioF/db-capstone-project/assets/148911432/bc2ea856-d36d-45c9-be90-ac782994795a)

### task 2. create a line chart to show the sales trend from 2019 to 2022. 
![image](https://github.com/FranciscoRubioF/db-capstone-project/assets/148911432/9411b7fa-8b14-43cc-bf78-3934ac470f42)


### task 3. create a Bubble chart of sales for all customers.
![image](https://github.com/FranciscoRubioF/db-capstone-project/assets/148911432/870d239f-bf1d-4e21-9d33-d404fa11414b)

### task 4. compare the sales of the three different cuisines sold at Little Lemon.
![image](https://github.com/FranciscoRubioF/db-capstone-project/assets/148911432/980b42cd-34ad-4b5c-abfe-ad016e4bec55)

### task 5. create an interactive dashboard that combines the Bar chart called Customers sales and the Sales Bubble Chart.
![image](https://github.com/FranciscoRubioF/db-capstone-project/assets/148911432/b563c975-e1f9-42ee-b5ea-77d1f160911e)
