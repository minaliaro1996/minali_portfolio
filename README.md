# Project : Monitor revenue generation from courses, subscriptions, and partnerships.
Analytics Portfolio
CREATE TABLE Courses (
    course_id INT PRIMARY KEY,
    course_name VARCHAR(255),
    category VARCHAR(100),
    price DECIMAL(10,2)
);

CREATE TABLE Sales (
    sale_id INT PRIMARY KEY AUTO_INCREMENT,
    course_id INT,
    user_id INT,
    purchase_date DATE,
    amount DECIMAL(10,2),
    payment_method VARCHAR(50),
    FOREIGN KEY (course_id) REFERENCES Courses(course_id)
);

CREATE TABLE Refunds (
    refund_id INT PRIMARY KEY AUTO_INCREMENT,
    sale_id INT,
    refund_date DATE,
    refund_amount DECIMAL(10,2),
    refund_reason TEXT,
    FOREIGN KEY (sale_id) REFERENCES Sales(sale_id)
);

CREATE TABLE Instructors (
    instructor_id INT PRIMARY KEY AUTO_INCREMENT,
    instructor_name VARCHAR(255),
    course_id INT,
    earnings DECIMAL(10,2),
    FOREIGN KEY (course_id) REFERENCES Courses(course_id)
);

SELECT c.category, c.course_name, COUNT(s.sale_id) AS total_sales, 
       SUM(s.amount) AS total_revenue
FROM Sales s
JOIN Courses c ON s.course_id = c.course_id
GROUP BY c.category, c.course_name
ORDER BY total_revenue DESC;

SELECT DATE_FORMAT(purchase_date, '%Y-%m') AS month, 
       SUM(amount) AS monthly_revenue
FROM Sales
GROUP BY month
ORDER BY month;

SELECT r.refund_reason, COUNT(r.refund_id) AS total_refunds, 
       SUM(r.refund_amount) AS refund_loss
FROM Refunds r
GROUP BY r.refund_reason
ORDER BY total_refunds DESC;

SELECT i.instructor_name, c.course_name, 
       SUM(i.earnings) AS total_earnings
FROM Instructors i
JOIN Courses c ON i.course_id = c.course_id
GROUP BY i.instructor_name, c.course_name
ORDER BY total_earnings DESC;

import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import mysql.connector

# Database Connection
conn = mysql.connector.connect(
    host="localhost", user="root", password="yourpassword", database="edtech_db"
)

# Fetch Data
query = "SELECT category, course_name, SUM(amount) AS revenue FROM Sales JOIN Courses ON Sales.course_id = Courses.course_id GROUP BY category, course_name;"
df = pd.read_sql(query, conn)

# Close Connection
conn.close()

# Visualizing Sales by Course
plt.figure(figsize=(12, 6))
sns.barplot(data=df, x="course_name", y="revenue", hue="category")
plt.xticks(rotation=45)
plt.title("Revenue by Course & Category")
plt.show()


