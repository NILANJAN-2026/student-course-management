# Student Course Management System - Full Spring Boot Project

## 1. Project Overview

This project is a Spring Boot web application for managing Students and Courses using:

- Spring Boot
- Spring MVC
- Spring Data JPA
- MySQL
- JSP
- JSTL
- Bootstrap
- JUnit & Mockito

Operations Implemented:

- Create Student
- Read Student List
- Update Student
- Custom Inner Join Query
- Exception Handling
- Unit Testing

---

# 2. Project Structure

```plaintext
student-course-management
│
├── src/main/java/com/example/demo
│   ├── controller
│   ├── entity
│   ├── repository
│   ├── service
│   └── DemoApplication.java
│
├── src/main/resources
│   └── application.properties
│
├── src/main/webapp/WEB-INF/views
│   ├── index.jsp
│   ├── add-student.jsp
│   └── update-student.jsp
│
├── src/test/java/com/example/demo
│   └── StudentRepositoryTest.java
│
└── pom.xml
```

---

# 3. pom.xml

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>student-course-management</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.5</version>
    </parent>

    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <scope>runtime</scope>
        </dependency>

        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-jasper</artifactId>
        </dependency>

        <dependency>
            <groupId>jakarta.servlet.jsp.jstl</groupId>
            <artifactId>jakarta.servlet.jsp.jstl-api</artifactId>
        </dependency>

        <dependency>
            <groupId>org.glassfish.web</groupId>
            <artifactId>jakarta.servlet.jsp.jstl</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

    </dependencies>

</project>
```

---

# 4. application.properties

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/studentdb
spring.datasource.username=root
spring.datasource.password=yourpassword

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true

spring.mvc.view.prefix=/WEB-INF/views/
spring.mvc.view.suffix=.jsp
```

---

# 5. Entity Classes

## Student.java

```java
package com.example.demo.entity;

import jakarta.persistence.*;
import java.util.List;

@Entity
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String email;

    @ManyToMany
    @JoinTable(
            name = "student_course",
            joinColumns = @JoinColumn(name = "student_id"),
            inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private List<Course> courses;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public List<Course> getCourses() {
        return courses;
    }

    public void setCourses(List<Course> courses) {
        this.courses = courses;
    }
}
```

---

## Course.java

```java
package com.example.demo.entity;

import jakarta.persistence.*;
import java.util.List;

@Entity
public class Course {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String courseName;

    @ManyToMany(mappedBy = "courses")
    private List<Student> students;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getCourseName() {
        return courseName;
    }

    public void setCourseName(String courseName) {
        this.courseName = courseName;
    }

    public List<Student> getStudents() {
        return students;
    }

    public void setStudents(List<Student> students) {
        this.students = students;
    }
}
```

---

# 6. Repository Layer

## StudentRepository.java

```java
package com.example.demo.repository;

import com.example.demo.entity.Student;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import java.util.List;

public interface StudentRepository extends JpaRepository<Student, Long> {

    @Query("SELECT s FROM Student s JOIN s.courses c")
    List<Student> getStudentsWithCourses();
}
```

---

## CourseRepository.java

```java
package com.example.demo.repository;

import com.example.demo.entity.Course;
import org.springframework.data.jpa.repository.JpaRepository;

public interface CourseRepository extends JpaRepository<Course, Long> {
}
```

---

# 7. Service Layer

## StudentService.java

```java
package com.example.demo.service;

import com.example.demo.entity.Student;
import com.example.demo.repository.StudentRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class StudentService {

    @Autowired
    private StudentRepository repository;

    public Student saveStudent(Student student) {
        return repository.save(student);
    }

    public List<Student> getAllStudents() {
        return repository.findAll();
    }

    public Student getStudentById(Long id) {
        return repository.findById(id).orElse(null);
    }

    public Student updateStudent(Student student) {
        return repository.save(student);
    }
}
```

---

# 8. Controller Layer

## StudentController.java

```java
package com.example.demo.controller;

import com.example.demo.entity.Student;
import com.example.demo.service.StudentService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

@Controller
public class StudentController {

    @Autowired
    private StudentService service;

    @GetMapping("/")
    public String home(Model model) {
        model.addAttribute("students", service.getAllStudents());
        return "index";
    }

    @GetMapping("/add")
    public String addForm(Model model) {
        model.addAttribute("student", new Student());
        return "add-student";
    }

    @PostMapping("/save")
    public String saveStudent(@ModelAttribute Student student) {

        try {
            service.saveStudent(student);
        }
        catch (Exception e) {
            e.printStackTrace();
        }

        return "redirect:/";
    }

    @GetMapping("/edit/{id}")
    public String editStudent(@PathVariable Long id, Model model) {

        Student student = service.getStudentById(id);

        model.addAttribute("student", student);

        return "update-student";
    }

    @PostMapping("/update")
    public String updateStudent(@ModelAttribute Student student) {

        service.updateStudent(student);

        return "redirect:/";
    }
}
```

---

# 9. JSP Pages

## index.jsp

```jsp
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>

<html>
<head>
    <title>Students</title>

    <style>
        body {
            font-family: Arial;
            padding: 30px;
            background-color: #f4f4f4;
        }

        table {
            width: 100%;
            border-collapse: collapse;
            background: white;
        }

        th, td {
            border: 1px solid black;
            padding: 10px;
            text-align: center;
        }

        a {
            text-decoration: none;
            color: blue;
        }
    </style>
</head>

<body>

<h2>Student List</h2>

<a href="/add">Add Student</a>

<br><br>

<table>

<tr>
    <th>ID</th>
    <th>Name</th>
    <th>Email</th>
    <th>Action</th>
</tr>

<c:forEach var="s" items="${students}">
<tr>
    <td>${s.id}</td>
    <td>${s.name}</td>
    <td>${s.email}</td>
    <td>
        <a href="/edit/${s.id}">Edit</a>
    </td>
</tr>
</c:forEach>

</table>

</body>
</html>
```

---

## add-student.jsp

```jsp
<html>
<head>
    <title>Add Student</title>
</head>
<body>

<h2>Add Student</h2>

<form action="/save" method="post">

    Name:
    <input type="text" name="name">

    <br><br>

    Email:
    <input type="email" name="email">

    <br><br>

    <button type="submit">Save</button>

</form>

</body>
</html>
```

---

## update-student.jsp

```jsp
<html>
<head>
    <title>Update Student</title>
</head>
<body>

<h2>Update Student</h2>

<form action="/update" method="post">

    <input type="hidden" name="id" value="${student.id}">

    Name:
    <input type="text" name="name" value="${student.name}">

    <br><br>

    Email:
    <input type="email" name="email" value="${student.email}">

    <br><br>

    <button type="submit">Update</button>

</form>

</body>
</html>
```

---

# 10. Main Application Class

## DemoApplication.java

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

---

# 11. Sample SQL Data

```sql
INSERT INTO course(course_name) VALUES
('Java'),
('Python'),
('DBMS'),
('C Programming'),
('Spring Boot'),
('Data Structures'),
('Machine Learning'),
('Cloud Computing'),
('Cyber Security'),
('AI Fundamentals');
```

---

# 12. Unit Testing

## StudentRepositoryTest.java

```java
package com.example.demo;

import com.example.demo.repository.StudentRepository;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import static org.junit.jupiter.api.Assertions.assertFalse;

@SpringBootTest
public class StudentRepositoryTest {

    @Autowired
    private StudentRepository repository;

    @Test
    void testFindAll() {

        assertFalse(repository.findAll().isEmpty());
    }
}
```

---

# 13. ER Diagram

```plaintext
Student
--------
id
name
email

Many-To-Many

Course
--------
id
courseName
```

---

# 14. How to Run the Project

1. Install MySQL
2. Create database:

```sql
CREATE DATABASE studentdb;
```

3. Open project in IntelliJ IDEA or VS Code
4. Configure application.properties
5. Run DemoApplication.java
6. Open browser:

```plaintext
http://localhost:8080/
```

---

# 15. Screenshots Required

Take screenshots of:

- Project Structure
- Database Tables
- Add Student Form
- Student List
- Update Operation
- SQL Tables
- Unit Test Result
- Browser Output

---

# 16. Challenges Faced

- JSP configuration issues
- Database connection setup
- Handling Many-to-Many relationship
- Fixing Maven dependencies

Solutions:

- Added correct dependencies
- Configured application.properties correctly
- Used JPA annotations properly

---

# 17. Github Upload

```bash
git init
git add .
git commit -m "Initial Commit"
git branch -M main
git remote add origin YOUR_GITHUB_URL
git push -u origin main
```

---

# 18. Final Submission

Submit:

- Source Code ZIP
- PDF Report
- Screenshots
- GitHub URL

---

# 19. Files You Must Upload to GitHub

Your final GitHub project should contain the following files and folders:

```plaintext
student-course-management/
│
├── src/
├── pom.xml
├── README.md
├── screenshots/
├── database/
│   └── sample-data.sql
│
├── report/
│   └── SpringBoot_Assignment_Report.pdf
│
└── .gitignore
```

---

# 20. README.md File

Create a file named:

```plaintext
README.md
```

Add the following content:

```md
# Student Course Management System

## Overview
This project is developed using Spring Boot, JSP, JPA, and MySQL.

Features:
- Create Student
- Read Student List
- Update Student
- Many-to-Many Relationship
- JPA Repository
- Custom Join Query
- Unit Testing

## Technologies Used
- Java
- Spring Boot
- MySQL
- JSP
- Bootstrap
- Maven

## Run Project
1. Create MySQL database:

```sql
CREATE DATABASE studentdb;
```

2. Configure application.properties
3. Run DemoApplication.java
4. Open browser:

http://localhost:8080/

## Author
Nilanjan Banerjee
```

---

# 21. sample-data.sql File

Create:

```plaintext
database/sample-data.sql
```

Add:

```sql
INSERT INTO course(course_name) VALUES
('Java'),
('Python'),
('DBMS'),
('Spring Boot'),
('Cloud Computing'),
('Machine Learning'),
('Data Structures'),
('Cyber Security'),
('AI Fundamentals'),
('Operating Systems');
```

---

# 22. .gitignore File

Create file:

```plaintext
.gitignore
```

Add:

```plaintext
/target/
.idea/
*.iml
*.log
```

---

# 23. Step-by-Step GitHub Upload Process

## Step 1: Create GitHub Account

Go to:

https://github.com

Create account if not already available.

---

## Step 2: Create New Repository

Click:

```plaintext
New Repository
```

Repository Name:

```plaintext
student-course-management
```

Select:
- Public
- Add README (optional)

Click:

```plaintext
Create Repository
```

---

## Step 3: Open Project Folder

Open terminal inside project folder.

Example:

```plaintext
student-course-management/
```

---

## Step 4: Initialize Git

Run:

```bash
git init
```

---

## Step 5: Add Files

Run:

```bash
git add .
```

---

## Step 6: Commit Files

Run:

```bash
git commit -m "Initial Spring Boot Assignment Upload"
```

---

## Step 7: Connect GitHub Repository

Copy repository URL from GitHub.

Example:

```plaintext
https://github.com/yourname/student-course-management.git
```

Run:

```bash
git remote add origin YOUR_REPOSITORY_URL
```

Example:

```bash
git remote add origin https://github.com/nilbanerjee/student-course-management.git
```

---

## Step 8: Push Project to GitHub

Run:

```bash
git branch -M main
git push -u origin main
```

---

# 24. Verify Upload

Open GitHub repository.

Verify:
- Source code visible
- README visible
- Screenshots uploaded
- SQL file uploaded
- PDF report uploaded

---

# 25. Screenshots Folder

Create folder:

```plaintext
screenshots/
```

Add screenshots:

- home-page.png
- add-student.png
- update-student.png
- mysql-table.png
- unit-test.png
- project-structure.png

---

# 26. Final ZIP File

Create final ZIP:

```plaintext
student-course-management.zip
```

Include:
- Entire Spring Boot project
- PDF Report
- Screenshots
- SQL File

---

# 27. Final Submission Checklist

✔ Spring Boot Project Running
✔ CRUD Operations Working
✔ MySQL Connected
✔ GitHub Repository Uploaded
✔ PDF Report Created
✔ Screenshots Added
✔ ZIP File Created
✔ README.md Added
✔ SQL Sample Data Added

---

# 28. Viva Preparation Questions

Prepare answers for:

1. What is Spring Boot?
2. What is JPA?
3. Explain @Entity.
4. Difference between @OneToMany and @ManyToMany.
5. What is JpaRepository?
6. What is MVC architecture?
7. What is Dependency Injection?
8. Explain CRUD operations.
9. What is Hibernate?
10. Why use JSP?

