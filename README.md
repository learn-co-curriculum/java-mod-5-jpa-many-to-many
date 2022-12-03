# Many-to-Many Relationship

## Learning Goals

- Create many-to-many associations between entities with JPA convention.

## Introduction

In this lesson, we will implement the following many-to-many relationship between
students and subjects:

![Many to many relationship table](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-jpa-manytomany/student_subject_nojointable.png)

Each student can take many subjects and each subject is taken by multiple students.

Recall that many-to-many relationships require a join table, so we need another
entity `STUDENT_DATA_SUBJECT` to keep track of the relationships. Note the
entity has a composite primary key `(student, subject)`. The updated ER model
that contains the new entity is shown below:

![Updated many to many relationship join table](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-jpa-manytomany/student_subject_withjointable.png)

## Create Subject Entity and Define Relationship

The `STUDENT_DATA_SUBJECT` table will be automatically created by JPA, so we only
need to create the `Subject` entity and define its relationship with the
`Student` entity.

Create a `Subject` class in the `model` package and add the following code:

```java
package org.example.model;

import javax.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
public class Subject {
    @Id
    @GeneratedValue
    private int id;

    private String title;

    @ManyToMany(mappedBy = "subjects")
    private List<Student> students = new ArrayList<>();

    public void addStudent(Student student) {
        students.add(student);
    }

    public List<Student> getStudents() {
        return students;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    @Override
    public String toString() {
        return "Subject{" +
                "id=" + id +
                ", title='" + title +
                '}';
    }
}
```

The project directory structure should look like this:

![student subject directory structure](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-jpa-manytomany/student_subject_project_structure.png)




Edit the `Student` class and add the following `subjects` field, getter, and setter:

```java
// Student.java

@ManyToMany
private List<Subject> subjects = new ArrayList<>();

public void addSubject(Subject subject) {
    subjects.add(subject);
}

public List<Subject> getSubjects() {
    return subjects;
}

// other getters/setters/toString
```

We have added a `List` to both classes because both of them have to be aware of
each other. For many-to-many relationships, the `mappedBy` property can be added
to either class since the join table is the same for both and keeps track of
the relationships between the two classes.  We added the `mappedBy` property
to the `Subject` class, but it could have been placed in the `Student` class instead.

Let’s modify the `JpaCreate` class to add `Subject` instances and define
instance relationships. Edit  the `JpaCreate` class and add the following
(place before the creation of the EntityManagerFactory and EntityManager):

```java
// create subjects
Subject subject1 = new Subject();
subject1.setTitle("Arts and Crafts");
Subject subject2 = new Subject();
subject2.setTitle("Reading");
Subject subject3 = new Subject();
subject3.setTitle("Math");

// create students >-< subjects association
student1.addSubject(subject1);
student1.addSubject(subject2);

student2.addSubject(subject2);
student2.addSubject(subject3);
```

Remember to persist the subjects as part of the transaction:

```java
//persist the subjects
entityManager.persist(subject1);
entityManager.persist(subject2);
entityManager.persist(subject3);
```

1. Edit `persistence.xml` to set the `hibernate.hbm2ddl.auto` property to `create`.
2. Run the `JpaCreate.main` method to create the new tables and insert the data. 
3. Use **pgAdmin** to query the new `SUBJECT` table and `STUDENT_DATA_SUBJECT` join table:


`SELECT * FROM SUBJECT;`

| ID  | TITLE           |
|-----|-----------------|
| 10  | Arts and Crafts |
| 11  | Reading         |
| 12  | Math            |




`SELECT * FROM STUDENT_DATA_SUBJECT;`

| STUDENTS_ID  | SUBJECTS_ID |
|--------------|-------------|
| 1            | 10          |
| 1            | 11          |
| 2            | 11          |
| 2            | 12          |



Notice the code established the relationship by adding a subject to a
student's list of subjects:

```java
// create students >-< subjects association
student1.addSubject(subject1);
student1.addSubject(subject2);

student2.addSubject(subject2);
student2.addSubject(subject3);
```

We could alternatively have established the relationship by adding a student to a
subject's list of students:

```java
// create students >-< subjects association
subject1.addStudent(student1);
subject2.addStudent(student1);

subject2.addStudent(student2);
subject3.addStudent(student2);
```

We only need to add to one list to add the association in the database.

## Fetching Data

Data fetching works exactly the same way as one-to-one and many-to-many. The
fetch type is also `LAZY` by default.

1. Edit `persistence.xml` to set the `hibernate.hbm2ddl.auto` property to `none`.
2. Edit `JpaRead` to get and print the subject's students and the student's subjects as shown below.
3. Run the `JpaRead.main` method to query the database.


```java
package org.example;

import org.example.model.IdCard;
import org.example.model.Project;
import org.example.model.Student;
import org.example.model.Subject;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.Persistence;
import java.util.List;

public class JpaRead {
    public static void main(String[] args) {
        // create EntityManager
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("example");
        EntityManager entityManager = entityManagerFactory.createEntityManager();

        // get student data using primary key id=1
        Student student1 = entityManager.find(Student.class, 1);
        System.out.println(student1);
        // get the student's id card
        IdCard card1 = student1.getCard();
        System.out.println(card1);
        // get the student's projects
        List<Project> projects = student1.getProjects();
        System.out.println(projects);
        // get the subject's students
        Subject subject = entityManager.find(Subject.class, 11);
        System.out.println(subject.getStudents());
        // get the student's subjects
        System.out.println(student1.getSubjects());

        // close entity manager and factory
        entityManager.close();
        entityManagerFactory.close();
    }
}
```

Mixed in with the Hibernate output is the list of students taking the subject with id 11,
and the list of subjects for the student with id 1:

```text
[Student{id=1, name='Jack', dob=2000-01-01, studentGroup=ROSE}, Student{id=2, name='Lee', dob=1999-01-01, studentGroup=DAISY}]  
[Subject{id=10, title='Arts and Crafts}, Subject{id=11, title='Reading}]
```

## Code Check

### Subject

```java
package org.example.model;

import javax.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
public class Subject {
    @Id
    @GeneratedValue
    private int id;

    private String title;

    @ManyToMany(mappedBy = "subjects")
    private List<Student> students = new ArrayList<>();

    public void addStudent(Student student) {
        students.add(student);
    }

    public List<Student> getStudents() {
        return students;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    @Override
    public String toString() {
        return "Subject{" +
                "id=" + id +
                ", title='" + title +
                '}';
    }
}
```

### Student

```java
package org.example.model;

import javax.persistence.*;
import java.time.LocalDate;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "STUDENT_DATA")
public class Student {
    @Id
    @GeneratedValue
    private int id;

    private String name;

    private LocalDate dob;

    @Enumerated(EnumType.STRING)
    private StudentGroup studentGroup;

    @OneToOne(fetch = FetchType.LAZY)
    private IdCard card;

    @OneToMany(mappedBy = "student")
    private List<Project> projects = new ArrayList<>();

    @ManyToMany
    private List<Subject> subjects = new ArrayList<>();

    public void addSubject(Subject subject) {
        subjects.add(subject);
    }

    public List<Subject> getSubjects() {
        return subjects;
    }

    public void addProject(Project project) {
        projects.add(project);
    }

    public List<Project> getProjects() {
        return projects;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public LocalDate getDob() {
        return dob;
    }

    public void setDob(LocalDate dob) {
        this.dob = dob;
    }

    public StudentGroup getStudentGroup() {
        return studentGroup;
    }

    public void setStudentGroup(StudentGroup studentGroup) {
        this.studentGroup = studentGroup;
    }

    public IdCard getCard() {
        return card;
    }

    public void setCard(IdCard card) {
        this.card = card;
    }

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", dob=" + dob +
                ", studentGroup=" + studentGroup +
                '}';
    }
}
```

### JpaCreate

```java
package org.example;

import org.example.model.*;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;
import javax.persistence.Persistence;
import java.time.LocalDate;

public class JpaCreate {
    public static void main(String[] args) {
        // create a new student instance
        Student student1 = new Student();
        student1.setName("Jack");
        student1.setDob(LocalDate.of(2000,1,1));
        student1.setStudentGroup(StudentGroup.ROSE);

        // create a new student instance
        Student student2 = new Student();
        student2.setName("Lee");
        student2.setDob(LocalDate.of(1999,1,1));
        student2.setStudentGroup(StudentGroup.DAISY);

        // create a new student instance
        Student student3 = new Student();
        student3.setName("Amal");
        student3.setDob(LocalDate.of(1980,1,1));
        student3.setStudentGroup(StudentGroup.LOTUS);


        // create id cards
        IdCard card1 = new IdCard();
        card1.setActive(true);
        IdCard card2 = new IdCard();
        IdCard card3 = new IdCard();

        // create projects
        Project project1 = new Project();
        project1.setTitle("Ant Hill Diorama");
        project1.setSubmissionDate(LocalDate.of(2022, 7, 20));
        project1.setScore(85);

        Project project2 = new Project();
        project2.setTitle("Saturn V Poster Presentation");
        project2.setSubmissionDate(LocalDate.of(2022, 7, 25));
        project2.setScore(90);

        Project project3 = new Project();
        project3.setTitle("Mars Rover Presentation");
        project3.setSubmissionDate(LocalDate.of(2022, 7, 30));
        project3.setScore(87);


        // create student-card associations
        student1.setCard(card1);
        student2.setCard(card2);
        student3.setCard(card3);

        // create student -< projects associations
        project1.setStudent(student1);
        project2.setStudent(student1);
        project3.setStudent(student2);

        student1.addProject(project1);
        student1.addProject(project2);
        student2.addProject(project3);

        // create subjects
        Subject subject1 = new Subject();
        subject1.setTitle("Arts and Crafts");
        Subject subject2 = new Subject();
        subject2.setTitle("Reading");
        Subject subject3 = new Subject();
        subject3.setTitle("Math");

        // create students >-< subjects association
        student1.addSubject(subject1);
        student1.addSubject(subject2);

        student2.addSubject(subject2);
        student2.addSubject(subject3);

        // create EntityManager
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("example");
        EntityManager entityManager = entityManagerFactory.createEntityManager();

        // access transaction object
        EntityTransaction transaction = entityManager.getTransaction();

        // create and use transactions
        transaction.begin();

        //persist the students
        entityManager.persist(student1);
        entityManager.persist(student2);
        entityManager.persist(student3);

        //persist the id cards
        entityManager.persist(card1);
        entityManager.persist(card2);
        entityManager.persist(card3);

        //persist the projects
        entityManager.persist(project1);
        entityManager.persist(project2);
        entityManager.persist(project3);

        //persist the subjects
        entityManager.persist(subject1);
        entityManager.persist(subject2);
        entityManager.persist(subject3);

        transaction.commit();

        //close entity manager and factory
        entityManager.close();
        entityManagerFactory.close();
    }
}
```


### JpaRead

```java
package org.example;

import org.example.model.IdCard;
import org.example.model.Project;
import org.example.model.Student;
import org.example.model.Subject;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.Persistence;
import java.util.List;

public class JpaRead {
    public static void main(String[] args) {
        // create EntityManager
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("example");
        EntityManager entityManager = entityManagerFactory.createEntityManager();

        // get student data using primary key id=1
        Student student1 = entityManager.find(Student.class, 1);
        System.out.println(student1);
        // get the student's id card
        IdCard card1 = student1.getCard();
        System.out.println(card1);
        // get the student's projects
        List<Project> projects = student1.getProjects();
        System.out.println(projects);
        // get the subject's students
        Subject subject = entityManager.find(Subject.class, 11);
        System.out.println(subject.getStudents());
        // get the student's subjects
        System.out.println(student1.getSubjects());

        // close entity manager and factory
        entityManager.close();
        entityManagerFactory.close();
    }
}
```

## Conclusion

We have learned how to create many-to-many relationships in this lesson. 

A student is associated with many subjects, thus the `Student` class stores
the relationship as:

```java
public class Student {
    ...

    @ManyToMany
    private List<Subject> subjects = new ArrayList<>();
    
    ...
}
```


A subject is associated with many students, thus the `Subject` class stores
the relationship as:

```java
public class Project {
    ...

    @ManyToMany(mappedBy = "subjects")
    private List<Student> students = new ArrayList<>();
    
    ...
}
```

The `mappedBy` property uses the `subjects` field of the `Student`
class to establish the many-to-many association between `Subject` and `Student`.

We establish the many-to-many association by adding a subject to a student's list of subjects:

```java
// create students >-< subjects association
student1.addSubject(subject1);
student1.addSubject(subject2);

student2.addSubject(subject2);
student2.addSubject(subject3);
```

We could alternatively establish the association by adding a student to a subject's list of students:

```java
// create students >-< subjects association
subject1.addStudent(student1);
subject2.addStudent(student1);

subject2.addStudent(student2);
subject3.addStudent(student2);
```

We can design and build complex applications now that we have looked
at the common relationships one-to-one, one-to-many, and many-to-many.
There are frameworks available that provide another layer of
abstraction which makes it even easier to work with databases which we will
check out in later lessons.


You can [fork and clone](https://github.com/learn-co-curriculum/java-mod-5-jpa-many-to-many)
the final version of the project.