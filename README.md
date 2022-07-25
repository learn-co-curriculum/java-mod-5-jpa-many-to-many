# Many-to-Many Relationship

## Learning Goals

- Create many-to-many associations between models with JPA convention.

## Introduction

In this lesson, we will be modeling the following many-to-many relationship:

![Many to many relationship table](https://curriculum-content.s3.amazonaws.com/java-spring-1/many-to-many-table.png)

Each student can have subjects and each subject is taken by multiple students.
Recall that many-to-many relationships require a join table so we are using
`STUDENT_DATA_SUBJECT` to keep track of the relationships.

## Create Subject Entity and Define Relationship

The `STUDENT_DATA_SUBJECT` table will be automatically created by JPA so we only
need to create the `Subject` entity and define its relationship with the
`Student` entity.

Create a `Subject` class in the `models` package and add the following code:

```java
package org.example.models;

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

    // other getters, setters, and toString
}
```

And in the `Student` class add the following property, getter, and setter:

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
```

We have added a `List` to both classes because both of them have to be aware of
each other. For many-to-many relationships, the `mappedBy` property can be added
to either classes since the join table is the same for both and keeps track of
the relationships between the two classes.

Let’s modify the `JpaCreate` class to add `Subject` instances and define
instance relationships. Open the `JpaCreate.java` file and add the following:

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
subject1.addStudent(student1);
subject2.addStudent(student1);

student2.addSubject(subject2);
student2.addSubject(subject3);
subject2.addStudent(student2);
subject3.addStudent(student2);
```

And remember to persist the subjects:

```java
entityManager.persist(subject1);
entityManager.persist(subject2);
entityManager.persist(subject3);
```

Now run the `main` method in the `JpaCreate` class to create the new tables and
insert the data. Make sure to change the `hibernate.hbm2ddl.auto` property to
`create` before running the `main` method. It should create a
`STUDENT_DATA_SUBJECT` table in the database.

| STUDENTS_ID | SUBJECTS_ID |
| ----------- | ----------- |
| 1           | 7           |
| 1           | 8           |
| 2           | 8           |
| 2           | 9           |

## Fetching Data

Data fetching works exactly the same way as one-to-one and many-to-many. The
fetch type is also `LAZY` by default.

Open up the `JpaRead` class and add the following:

```java
package org.example;

import org.example.models.Student;
import org.example.models.Subject;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.Persistence;
import java.util.List;

public class JpaRead {
    public static void main(String[] args) {
        // create EntityManager
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("example");
        EntityManager entityManager = entityManagerFactory.createEntityManager();

        // get records
        Subject subject = entityManager.find(Subject.class, 8);
        System.out.println(subject.getStudents());

        // close entity manager
        entityManager.close();
        entityManagerFactory.close();
    }
}
```

This will print out all the students who takes `subject` when you run the `main`
method (`hibernate.hbm2ddl.auto` is set to `none`)

```java
[
	Student{id=1, name='Jack', dob=2022-07-25, studentGroup=LOTUS},
	Student{id=2, name='Leslie', dob=2022-07-25, studentGroup=ROSE}
]
```

## Code Check

### Subject.java

```java
package org.example.models;

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
                ", title='" + title + '\'' +
                ", students=" + students +
                '}';
    }
}
```

### Student.java

```java
package org.example.models;

import org.example.enums.StudentGroup;

import javax.persistence.*;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

@Entity
@Table(name = "STUDENT_DATA")
public class Student {
    @Id
    @GeneratedValue
    private int id;

    private String name;

    @Temporal(TemporalType.DATE)
    private Date dob;

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

    public void addProjct(Project project) {
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

    public Date getDob() {
        return dob;
    }

    public void setDob(Date dob) {
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

### JpaCreate.java

```java
package org.example;

import org.example.enums.StudentGroup;
import org.example.models.IdCard;
import org.example.models.Project;
import org.example.models.Student;
import org.example.models.Subject;

import javax.persistence.Persistence;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityManager;
import javax.persistence.EntityTransaction;
import java.time.LocalDate;
import java.util.Date;

public class JpaCreate {
    public static void main(String[] args) {
        // create student instances
        Student student1 = new Student();
        student1.setName("Jack");
        student1.setDob(new Date());
        student1.setStudentGroup(StudentGroup.LOTUS);

        Student student2 = new Student();
        student2.setName("Leslie");
        student2.setDob(new Date());
        student2.setStudentGroup(StudentGroup.ROSE);

        // create id cards
        IdCard card1 = new IdCard();
        card1.setActive(true);

        IdCard card2 = new IdCard();
        card2.setActive(false);

        // create student-card associations
        student1.setCard(card1);
        student2.setCard(card2);

        // create projects
        Project project1 = new Project();
        project1.setTitle("Ant Hill Diorama");
        project1.setSubmissionDate(LocalDate.of(2022, 7, 20));
        project1.setScore(85);

        Project project2 = new Project();
        project2.setTitle("Saturn V Poster Presentation");
        project2.setSubmissionDate(LocalDate.of(2022, 7, 25));
        project2.setScore(90);

        // create student -< projects associations
        project1.setStudent(student1);
        project2.setStudent(student1);

        student1.addProjct(project1);
        student1.addProjct(project2);

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
        subject1.addStudent(student1);
        subject2.addStudent(student1);

        student2.addSubject(subject2);
        student2.addSubject(subject3);
        subject2.addStudent(student2);
        subject3.addStudent(student2);

        // create EntityManager
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("example");
        EntityManager entityManager = entityManagerFactory.createEntityManager();

        // access transaction object
        EntityTransaction transaction = entityManager.getTransaction();

        // create and use transactions
        transaction.begin();
        entityManager.persist(student1);
        entityManager.persist(student2);

        entityManager.persist(card1);
        entityManager.persist(card2);

        entityManager.persist(project1);
        entityManager.persist(project2);

        entityManager.persist(subject1);
        entityManager.persist(subject2);
        entityManager.persist(subject3);
        transaction.commit();

        // close entity manager
        entityManager.close();
        entityManagerFactory.close();
    }
}
```

## Conclusion

We have learned how to create many-to-many relationships in this lesson. Now
that we have looked at the common relationships, we can design and build complex
applications. There are frameworks available that provide another layer of
abstraction which makes it even easier to work with databases which we will
check out in later lessons.
