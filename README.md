CREATE TABLE IF NOT EXISTS Department (
    DepartmentID INT AUTO_INCREMENT PRIMARY KEY,
    DepartmentName VARCHAR(50) NOT NULL
);

CREATE TABLE IF NOT EXISTS Employee (
    EmployeeID INT AUTO_INCREMENT PRIMARY KEY,
    Name VARCHAR(100) NOT NULL,
    Email VARCHAR(100) UNIQUE NOT NULL,
    DepartmentID INT,
    Designation VARCHAR(50),
    DateOfJoining DATE,
    FOREIGN KEY (DepartmentID) REFERENCES Department(DepartmentID)
        ON DELETE SET NULL
        ON UPDATE CASCADE
);


CREATE TABLE IF NOT EXISTS Salary (
    SalaryID INT AUTO_INCREMENT PRIMARY KEY,
    EmployeeID INT NOT NULL,
    BasicPay DECIMAL(10,2) NOT NULL,
    HRA DECIMAL(10,2) NOT NULL,
    Allowances DECIMAL(10,2) NOT NULL,
    Deductions DECIMAL(10,2) NOT NULL,
    NetSalary DECIMAL(10,2) AS (BasicPay + HRA + Allowances - Deductions) STORED,
    SalaryMonth INT NOT NULL CHECK (SalaryMonth BETWEEN 1 AND 12),
    SalaryYear INT NOT NULL,
    FOREIGN KEY (EmployeeID) REFERENCES Employee(EmployeeID)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);


CREATE TABLE IF NOT EXISTS Bonus (
    BonusID INT AUTO_INCREMENT PRIMARY KEY,
    EmployeeID INT NOT NULL,
    BonusAmount DECIMAL(10,2) NOT NULL,
    BonusDate DATE NOT NULL,
    FOREIGN KEY (EmployeeID) REFERENCES Employee(EmployeeID)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);



INSERT INTO Department (DepartmentName) VALUES 
('IT'), 
('HR'), 
('Finance');

INSERT INTO Employee (Name, Email, DepartmentID, Designation, DateOfJoining) 
VALUES 
('John Doe', 'john@example.com', 1, 'Software Engineer', '2023-01-10'),
('Jane Smith', 'jane@example.com', 2, 'HR Manager', '2022-05-20'),
('Mike Johnson', 'mike@example.com', 3, 'Accountant', '2021-08-15');

INSERT INTO Salary (EmployeeID, BasicPay, HRA, Allowances, Deductions, SalaryMonth, SalaryYear)
VALUES
(1, 50000, 10000, 5000, 2000, 8, 2025),
(2, 60000, 12000, 6000, 3000, 8, 2025),
(3, 40000, 8000, 4000, 1000, 8, 2025);


INSERT INTO Bonus (EmployeeID, BonusAmount, BonusDate)
VALUES
(1, 2000, '2025-08-31'),
(2, 3000, '2025-08-31');


SELECT e.EmployeeID, e.Name, e.Email, d.DepartmentName, e.Designation, e.DateOfJoining
FROM Employee e
LEFT JOIN Department d ON e.DepartmentID = d.DepartmentID;


SELECT e.Name, s.BasicPay, s.HRA, s.Allowances, s.Deductions, s.NetSalary, s.SalaryMonth, s.SalaryYear
FROM Salary s
JOIN Employee e ON s.EmployeeID = e.EmployeeID;


INSERT INTO Salary (EmployeeID, BasicPay, HRA, Allowances, Deductions, SalaryMonth, SalaryYear)
VALUES (1, 52000, 10500, 5000, 2500, 9, 2025);


UPDATE Salary
SET BasicPay = 55000, HRA = 11000, Deductions = 3000
WHERE EmployeeID = 1 AND SalaryMonth = 9 AND SalaryYear = 2025;


DELETE FROM Salary
WHERE EmployeeID = 3 AND SalaryMonth = 8 AND SalaryYear = 2025;

SELECT e.Name, d.DepartmentName, s.BasicPay, s.HRA, s.Allowances, s.Deductions, s.NetSalary
FROM Salary s
JOIN Employee e ON s.EmployeeID = e.EmployeeID
JOIN Department d ON e.DepartmentID = d.DepartmentID
WHERE s.SalaryMonth = 8 AND s.SalaryYear = 2025;

SELECT e.Name, b.BonusAmount, b.BonusDate
FROM Bonus b
JOIN Employee e ON b.EmployeeID = e.EmployeeID;

SELECT SUM(NetSalary) AS TotalPayroll
FROM Salary
WHERE SalaryMonth = 8 AND SalaryYear = 2025;

SELECT e.Name, s.NetSalary + IFNULL(b.BonusAmount,0) AS TotalPaid
FROM Salary s
JOIN Employee e ON s.EmployeeID = e.EmployeeID
LEFT JOIN Bonus b 
    ON s.EmployeeID = b.EmployeeID 
    AND MONTH(b.BonusDate) = s.SalaryMonth 
    AND YEAR(b.BonusDate) = s.SalaryYear;
    -- 1. Find employees who joined in a specific year
SELECT Name, DateOfJoining
FROM Employee
WHERE YEAR(DateOfJoining) = 2022;

-- 2. Get the total number of employees in each department
SELECT d.DepartmentName, COUNT(e.EmployeeID) AS EmployeeCount
FROM Department d
LEFT JOIN Employee e ON d.DepartmentID = e.DepartmentID
GROUP BY d.DepartmentName;

-- 3. Find employees with a net salary above a certain amount
SELECT e.Name, s.NetSalary
FROM Salary s
JOIN Employee e ON s.EmployeeID = e.EmployeeID
WHERE s.NetSalary > 60000;

-- 4. List all employees with their department and bonus details (if any)
SELECT e.Name, d.DepartmentName, b.BonusAmount, b.BonusDate
FROM Employee e
LEFT JOIN Department d ON e.DepartmentID = d.DepartmentID
LEFT JOIN Bonus b ON e.EmployeeID = b.EmployeeID;

-- 5. Get average salary by department
SELECT d.DepartmentName, AVG(s.NetSalary) AS AverageSalary
FROM Salary s
JOIN Employee e ON s.EmployeeID = e.EmployeeID
JOIN Department d ON e.DepartmentID = d.DepartmentID
GROUP BY d.DepartmentName;

-- 6. Find the highest paid employee in each department
SELECT d.DepartmentName, e.Name, s.NetSalary
FROM Salary s
JOIN Employee e ON s.EmployeeID = e.EmployeeID
JOIN Department d ON e.DepartmentID = d.DepartmentID
WHERE s.NetSalary = (
    SELECT MAX(s2.NetSalary)
    FROM Salary s2
    JOIN Employee e2 ON s2.EmployeeID = e2.EmployeeID
    WHERE e2.DepartmentID = d.DepartmentID
);

-- 7. Show employees who have not received any bonus
SELECT e.Name
FROM Employee e
LEFT JOIN Bonus b ON e.EmployeeID = b.EmployeeID
WHERE b.EmployeeID IS NULL;

-- 8. Get the total bonus amount paid in a specific month and year
SELECT SUM(BonusAmount) AS TotalBonus
FROM Bonus
WHERE MONTH(BonusDate) = 8 AND YEAR(BonusDate) = 2025;

-- 9. List employees with their full salary details for a specific month and year
SELECT e.Name, d.DepartmentName, s.BasicPay, s.HRA, s.Allowances, s.Deductions, s.NetSalary
FROM Salary s
JOIN Employee e ON s.EmployeeID = e.EmployeeID
JOIN Department d ON e.DepartmentID = d.DepartmentID
WHERE s.SalaryMonth = 9 AND s.SalaryYear = 2025;

-- 10. Find employees who joined before a specific date
SELECT Name, DateOfJoining
FROM Employee
WHERE DateOfJoining < '2022-01-01';

