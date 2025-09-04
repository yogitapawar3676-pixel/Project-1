# Project-1

CREATE DATABASE Studentresult_db;

USE Studentresult_db;

CREATE TABLE Students(
   student_id INT AUTO_INCREMENT PRIMARY KEY,
   name VARCHAR(100),
   dob INT,
   department VARCHAR(100)
);


CREATE TABLE Course(
  course_id INT PRIMARY KEY,
  course_name VARCHAR(100),
  credits INT
);

CREATE TABLE Semesters(
  semester_id INT PRIMARY KEY,
  semester_name VARCHAR(100)
);

CREATE TABLE Grades(
  grade_id INT AUTO_INCREMENT PRIMARY KEY,
  student_id INT,
  course_id INT,
  semester_id INT,
  marks INT,
  grade CHAR(2),
  gpa DECIMAL(3,2),
  FOREIGN KEY (student_id) REFERENCES Students(student_id),
  FOREIGN KEY (course_id) REFERENCES Course(course_id),
  FOREIGN KEY (semester_id) REFERENCES Semesters(semester_id)
);
DROP TABLE Grades;
INSERT INTO Students(name, dob, department) VALUES
('Alice',2002-08-12,'CSE'),
('Rim',2003-06-05,'ECE'),
('Charlie',2003-03-11,'IT');

INSERT INTO Course(course_id,course_name, credits) VALUES
(1,'Database Sysrem',4),
(2,'Computer Network',3),
(3,'Operating System',4),
(4,'Mathematics',3);

INSERT INTO Semesters(semester_id,semester_name)VALUES
(01,'sem1'),
(02,'sem2');

INSERT INTO Grades(student_id,course_id,semester_id,marks,grade,gpa) VALUES
(1,1,1,86,'A',4.0),
(1,2,1,78,'B',3.0),
(2,1,1,65,'C',2.0),
(2,3,1,92,'A',4.0),
(3,2,2,55,'D',1.0),
(3,4,2,87,'A',4.0);

SELECT 
    s.student_id,
    s.name,
    sem.semester_id,
    ROUND(SUM(g.gpa * c.credits) / SUM(c.credits),2)AS Semester_GPA
FROM Grades g
JOIN Students s ON g.student_id = s.student_id
JOIN Course  c ON g.course_id = c.course_id
JOIN Semesters sem ON g.semester_id = sem.semester_id
GROUP BY s.student_id, s.name, sem.semester_id;


SELECT s.name, count(*)
AS Total_subjects,
SUM(CASE WHEN g.mark >=40 THEN 1 ELSE 0 END)
AS Failed FROM Grades g
JOIN Students s ON g.students_id = s.student_id
GROUP BY s.student_id;
   
   
SELECT s.student_id,s.name,
ROUND(SUM(g.gpa*c.credits)/SUM(c.credits),2)
AS CGPA,
RANK() OVER (ORDER BY ROUND(SUM(g.gpa*c.credits)/SUM(c.credits),2)DESC)
AS Rank_Position 
FROM Grades g 
JOIN Students s ON g.student_id = s.student_id
JOIN Course c ON g.course_id = c.course_id
GROUP BY s.student_id;
   
   DELIMITER $$
   
   CREATE TRIGGER calculate_gpa
   BEFORE INSERT ON Grades
   FOR EACH ROW
   BEGIN
	IF NEW.marks >= 85 THEN 
        SET NEW.grade = 'A';
        SET NEW.gpa = 4.0;
	ELSEIF NEW.marks >= 70 THEN
        SET NEW.grade = 'B';
        SET NEW.gpa = 3.0;
	ELSEIF NEW.marks >= 55 THEN
        SET NEW.grade = 'C';
        SET NEW.gpa = 2.0;
	ELSEIF NEW.marks >= 40 THEN 
        SET NEW.grade = 'D';
        SET NEW.gpa = 1.0;
	ELSE
        SET NEW.grade = 'F';
        SET NEW.gpa = 0.0;
	END IF;
    END$$
    DELIMITER ;
    
    
  CREATE VIEW Semester_Result
  AS SELECT s.student_id,s.name,sem.semester_id,
  ROUND(SUM(g.gpa*c.credits)/SUM(c.credits),2)
  AS GPA 
  FROM Grades g
  JOIN Students s ON g.student_id = s.student_id
  JOIN Course c ON g.course_id = c.course_id
  JOIN Semesters sem ON g.semester_id = sem.semester_id
  GROUP BY s.student_id, sem.semester_id;
  
  SELECT*FROM Semester_Result
  INTO OUTFILE'/tmp/semester_result.csv'
  FIELDS TERMINATED BY','LINES TERMINATED BY '\n';
 
 



