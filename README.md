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
