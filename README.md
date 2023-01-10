# Many-to-Many Relationship

## Learning Goals

- Create many-to-many associations between entities with JPA convention.

## Code Along

We will continue with the `Student`, `IdCard`, and `Project` entities from the previous lessons.

## Introduction

Let's update the model to add a `Subject` entity to represent the subjects studied by students.

- Each subject has an id and title. 
- A subject is studied by many students. A student may study many subjects.

In this lesson, we will implement the following many-to-many relationship between
students and subjects:

![Many to many relationship table](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-jpa-manytomany/student_subject_nojointable.png)

The many-to-many relationship would be stored in a database using a join table `Student_Subject`
that has a composite primary key `(student, subject)`. The updated ER model
that contains an entity `Student_Subject` that models the join table is shown below.
The many-to-many relationship between `Student` and `Subject` is replaced with
one-to-many relationships with the new entity `Student_Subject`.

![Updated many to many relationship join table](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-jpa/student_subject_join_table.png)

However, the join table representing the `Student_Subject` entity is automatically
created by JPA, so we only need to create the `Subject` entity class in Java and then
define the many-to-many relationship between `Student` and `Subject` using the `@ManyToMany`
annotation based on model defined in the original ERD:

![Many to many relationship table](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-jpa-manytomany/student_subject_nojointable.png)


## Create Subject Entity and Instances

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

Edit the `JpaCreate` class to create three new subjects and persist them in the database:

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

        // create subjects
        Subject subject1 = new Subject();
        subject1.setTitle("Arts and Crafts");
        Subject subject2 = new Subject();
        subject2.setTitle("Reading");
        Subject subject3 = new Subject();
        subject3.setTitle("Math");

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

        // create students <> subjects association


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

1. Edit `persistence.xml` to set the `hibernate.hbm2ddl.auto` property to `create`.
2. Run the `JpaCreate.main` method to create the new table and insert the data.
3. Use **pgAdmin** to query the new `SUBJECT` table:


`SELECT * FROM SUBJECT;`

| ID  | TITLE           |
|-----|-----------------|
| 10  | Arts and Crafts |
| 11  | Reading         |
| 12  | Math            |

## Implementing the Many-To-Many Relationship

A many-to-many relationship between two entities is
not directly stored as a foreign key in either entity's table.  
As we've seen in prior lessons, the many-to-many relationship requires
a new join table that has a composite key consisting
of the two entity's primary keys.

In terms of the JPA implementation, the many-to-many
relationship is stored in each entity class as a collection
such as a list or set.

We will still pick one of the entities as the "owning"
side of the relationship to establish the join table.
The non-owning side entity will be
defined using the `mappedBy` attribute.
We will create an association between two objects
by adding to the collection on the owning side
of the relationship.

Here are the steps we have to follow to implement the many-to-many relationship with JPA:

- Pick one entity to be the owner of the relationship.
- The owning side Java class adds a new field with the annotation `@ManyToMany`.
    - The field stores a collection of references to non-owning side entities.
    - The optional `@JoinTable` annotation may be used to specify the join table name and the composite key column names.
    - Implement a method to add an entity to the collection.
- The non-owning side Java class adds a new field with the annotation `@ManyToMany`.
    - The field stores a collection of references to owning side entities.
    - The `mappedBy` property references the `@ManyToMany` field that was added to the owning side class.
    - If a method adds an entity to the non-owning side collection, the method should also add to the owning side collection.
- A new join table is generated containing a composite primary key.


Let's go through the steps to implement the relationship between `Student` and `Subject`.
We will pick `Student` as the owning side in the many-to-many
relationship between `Student` and `Subject`:

![many to many relationship student owning side](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-jpa/many_to_many_student_owning_side.png)


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

Edit the `Subject` class and add the following `students` field, getter, and setter:

```java
@ManyToMany(mappedBy = "subjects")
private List<Student> students = new ArrayList<>();

public void addStudent(Student student) {
    students.add(student);
    student.addSubject(this);  //Student is owning side
}

public List<Student> getStudents() {
    return students;
}
```

NOTE: Since `Student` is the owner of the relationship, the `addStudent()` method
calls `student.addSubject(this)` to ensure the owning side properly established the
relationship.  We can also omit the `addStudent()` method from `Subject`, forcing the
relationship to always be established through the owning entity `Student`.


We have added a `List` to both classes because both of them have to be aware of
each other. For many-to-many relationships, the `mappedBy` property can be added
to either class since the join table is the same for both and keeps track of
the relationships between the two classes.  We added the `mappedBy` property
to the `Subject` class, but it could have been placed in the `Student` class instead.

Let’s modify the `JpaCreate` class to add associations between students and subjects.
Edit  the `JpaCreate` class and add the following associations:
```java

// create students <> subjects association
student1.addSubject(subject1);
student1.addSubject(subject2);

student2.addSubject(subject2);
student2.addSubject(subject3);
```

1. Run the `JpaCreate.main` method to create the tables and insert the data. 
2. Use **pgAdmin** to query the `SUBJECT`, `STUDENT`, and `STUDENT_SUBJECT` tables.
   Notice the relationship is stored in the join table `STUDENT_SUBJECT`.

`SELECT * FROM SUBJECT;`

| ID  | TITLE           |
|-----|-----------------|
| 10  | Arts and Crafts |
| 11  | Reading         |
| 12  | Math            |

`SELECT * FROM STUDENT;`

| ID   | DOB         | NAME  | STUDENTGROUP  | CARD_ID |
|------|-------------|-------|---------------|---------|
| 1    | 2000-01-01  | Jack  | ROSE          | 4       |
| 2    | 1999-01-01  | Lee   | DAISY         | 5       |
| 3    | 1980-01-01  | Amal  | LOTUS         | 6       |


`SELECT * FROM STUDENT_SUBJECT;`

| STUDENTS_ID  | SUBJECTS_ID |
|--------------|-------------|
| 1            | 10          |
| 1            | 11          |
| 2            | 11          |
| 2            | 12          |


Notice the code established the relationship by adding a subject to a
student's list of subjects:

```java
// create students <> subjects association
student1.addSubject(subject1);
student1.addSubject(subject2);

student2.addSubject(subject2);
student2.addSubject(subject3);
```

We could alternatively have established the relationship by adding a student to a
subject's list of students:

```java
// create students <> subjects association
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
2. Edit `persistence.xml` to set the `hibernate.show_sql` property `false` to suppress the SQL output printed by Hibernate.
3. Edit `JpaRead` to get and print the subject's students and the student's subjects as shown below.
4. Run the `JpaRead.main` method to query the database.


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

The last two lines of output display students studying a particular subject, and subjects
studied by a particular student.

```text
Student{id=1, name='Jack', dob=2000-01-01, studentGroup=ROSE}
IdCard{id=4, isActive=true}
[Project{id=7, title='Ant Hill Diorama', submissionDate=2022-07-20, score=85}, Project{id=8, title='Saturn V Poster Presentation', submissionDate=2022-07-25, score=90}]
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
        student.addSubject(this);  //Student is owning side
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

        // create subjects
        Subject subject1 = new Subject();
        subject1.setTitle("Arts and Crafts");
        Subject subject2 = new Subject();
        subject2.setTitle("Reading");
        Subject subject3 = new Subject();
        subject3.setTitle("Math");

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

        // create students <> subjects association
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

- Pick one entity to be the owner of the relationship.
- The owning side Java class adds a new field with the annotation `@ManyToMany`.
  - The field stores a collection of references to non-owning side entities.
  - The optional `@JoinTable` annotation may be used to specify the join table name and the composite key column names.
  - Implement a method to add an entity to the collection.
- The non-owning side Java class adds a new field with the annotation `@ManyToMany`.
  - The field stores a collection of references to owning side entities.
  - The `mappedBy` property references the `@ManyToMany` field that was added to the owning side class.
  - If a method adds an entity to the non-owning side collection, the method should also add to the owning side collection.
- A new join table is generated containing a composite primary key.


A student is associated with many subjects.  When `Student` is on the owning side,
the relationship is implemented as:

```java
public class Student {
    ...

    @ManyToMany
    private List<Subject> subjects = new ArrayList<>();

    public void addSubject(Subject subject) {
      subjects.add(subject);
    }
    ...
}
```


A subject is associated with many students.  When `Subject` is on
the non-owning side, the relationship is implemented as:

```java
public class Subject {
    ...

    @ManyToMany(mappedBy = "subjects")
    private List<Student> students = new ArrayList<>();
    
    ...
}
```

The `mappedBy` property uses the `subjects` field of the `Student`
class to establish the many-to-many association between `Subject` and `Student`.

We establish the many-to-many association by adding to a student's list of subjects:

```java
// create students <> subjects association
student1.addSubject(subject1);
student1.addSubject(subject2);

student2.addSubject(subject2);
student2.addSubject(subject3);
```

We can design and build complex applications now that we have looked
at the common relationships one-to-one, one-to-many, and many-to-many.
There are frameworks available that provide another layer of
abstraction which makes it even easier to work with databases which we will
check out in later lessons.


You can [fork and clone](https://github.com/learn-co-curriculum/java-mod-5-jpa-many-to-many)
the final version of the project.