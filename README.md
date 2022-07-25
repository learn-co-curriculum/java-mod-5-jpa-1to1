# One-to-One Relationship

## Learning Goals

- Create one-to-one associations between models with JPA convention.
- Fetch related data from the database.

## Introduction

Relational database tables can be built such that they associate with one
another via primary and foreign keys. JPA provides us with annotations that
streamline model association in our program.

In this lesson, we will be building out the following relationship:

![One to One relationship table](https://curriculum-content.s3.amazonaws.com/java-spring-1/one-to-one-table.png)

Each student has an ID card and each ID card belongs to a student. We will only
use JPA annotations to create the relationship.

## Create ID Card Entity and Instances

Create the `IdCard` class in the `models` package and add the following code:

```java
package org.example.models;

import javax.persistence.*;

@Entity
@Table(name = "ID_CARD")
public class IdCard {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private int id;

    private boolean isActive;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public boolean isActive() {
        return isActive;
    }

    public void setActive(boolean active) {
        isActive = active;
    }

    @Override
    public String toString() {
        return "IdCard{" +
                "id=" + id +
                ", isActive=" + isActive +
                '}';
    }
}
```

Now, open the `JpaCreate` class, create two new ID cards, and save them in the
database (`hibernate.hbm2ddl.auto` should be `create`):

```java
package org.example;

import org.example.enums.StudentGroup;
import org.example.models.IdCard;
import org.example.models.Student;

import javax.persistence.Persistence;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityManager;
import javax.persistence.EntityTransaction;
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
        transaction.commit();

        // close entity manager
        entityManager.close();
        entityManagerFactory.close();
    }
}
```

Your directory structure should look like this:

```java
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── org
    │   │       └── example
    │   │           ├── JpaCreate.java
    │   │           ├── JpaDelete.java
    │   │           ├── JpaRead.java
    │   │           ├── JpaUpdate.java
    │   │           ├── enums
    │   │           │   └── StudentGroup.java
    │   │           └── models
    │   │               ├── IdCard.java
    │   │               └── Student.java
    │   └── resources
    │       └── META-INF
    │           └── persistence.xml
    └── test
        └── java
```

## Modeling the Relationship

Here is the relationship we are modeling for reference:

![One to one relationship table](https://curriculum-content.s3.amazonaws.com/java-spring-1/one-to-one-table.png)

Both the `Student` and `IdCard` classes have to be modified to model the
relationship. Here are the steps we have to follow:

1. Create fields in both classes to store the associated instance.
2. Annotate fields with the `@OneToOne` annotation.
3. Define which table will reference the foreign key by passing the `mappedBy`
   value to the `@OneToOne` annotation.

```java
package org.example.models;

import org.example.enums.StudentGroup;
import javax.persistence.*;
import java.util.Date;

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

    @OneToOne
    private IdCard card;

		// getters and setters
}
```

```java
package org.example.models;

import javax.persistence.*;

@Entity
@Table(name = "ID_CARD")
public class IdCard {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private int id;

    private boolean isActive;

    @OneToOne(mappedBy = "card")
    private Student student;

		// getters and setters
}
```

We have added the `card` property to the `Student` class and the `student`
property to the `IdCard` class. If we simply put a `@OneToOne` annotation on
both the properties, the database will create foreign keys in both tables.

`STUDENT_DATA` Table:

| ID  | DOB        | NAME   | STUDENTGROUP | CARD_ID |
| --- | ---------- | ------ | ------------ | ------- |
| 1   | 2022-06-14 | Jack   | LOTUS        | null    |
| 2   | 2022-06-14 | Leslie | ROSE         | null    |

`ID_CARD` Table:

| ID  | ISACTIVE | STUDENT_ID |
| --- | -------- | ---------- |
| 3   | TRUE     | null       |
| 4   | FALSE    | null       |

We need to define that the `card` and `student` property in the two models are
connected to each other, i.e., they are the same relationship definition. The
`mappedBy` property used in the `IdCard` class tells JPA that the `student`
property in the class has a one-to-one relationship with the `card` property in
the `Student` class.

Modify the `JpaCreate` class to associate the data:

```java
package org.example;

import org.example.enums.StudentGroup;
import org.example.models.IdCard;
import org.example.models.Project;
import org.example.models.Student;

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
        transaction.commit();

        // close entity manager
        entityManager.close();
        entityManagerFactory.close();
    }
}
```

If we run the `main` method in the `JpaCreate` class, the tables will look like
the following:

| ID  | DOB        | NAME   | STUDENTGROUP | CARD_ID |
| --- | ---------- | ------ | ------------ | ------- |
| 1   | 2022-06-14 | Jack   | LOTUS        | 3       |
| 2   | 2022-06-14 | Leslie | ROSE         | 4       |

| ID  | ISACTIVE |
| --- | -------- |
| 3   | TRUE     |
| 4   | FALSE    |

## Fetching Data with Relationships

JPA performs join operations automatically to get related data from an entity.
For one-to-one relationships, it performs the join operation on the first query
but we can defer this operation to when the data is accessed in the program
using the `fetch` property on the `@OneToOne` annotation.

Change the `hibernate.hbm2ddl.auto` property in the `persistence.xml` to `none`
before performing read operations. We will be querying data in the `JpaRead`
class.

Open the `JpaRead` class and add the following code:

```java
package org.example;

import org.example.models.IdCard;
import org.example.models.Student;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.Persistence;

public class JpaRead {
    public static void main(String[] args) {
        // create EntityManager
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("example");
        EntityManager entityManager = entityManagerFactory.createEntityManager();

        // get records
        Student student1 = entityManager.find(Student.class, 1);
        IdCard card1 = student1.getCard();
        System.out.println(card1);

        // close entity manager
        entityManager.close();
        entityManagerFactory.close();
    }
}
```

When we run the `main` method, the program will print out the `IdCard`
associated with the student with ID `1`. If you look at the terminal, you can
see the query that Hibernate used. Notice that the join operation was performed
when the `find` method was called.

```
Hibernate:
    select
        student0_.id as id1_1_0_,
        student0_.card_id as card_id5_1_0_,
        student0_.dob as dob2_1_0_,
        student0_.name as name3_1_0_,
        student0_.studentGroup as studentg4_1_0_,
        idcard1_.id as id1_0_1_,
        idcard1_.isActive as isactive2_0_1_
    from
        STUDENT_DATA student0_
    left outer join
        ID_CARD idcard1_
            on student0_.card_id=idcard1_.id
    where
        student0_.id=?
```

### Defer Related Data Fetching

We can modify the behavior such that the join is performed when the `getCard()`
method is called on a `Student` instance. This is beneficial if a model has lots
of associations which would make the initial join operation slow.

Open the `Student` class and add the `(fetch = FetchType.LAZY)` property to the
`@OneToOne` annotation:

```java
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
}
```

A one-to-one association uses the `FetchType.EAGER` value if no value is
specified. By setting it to `FetchType.LAZY`, we can defer the join query. Run
the `main` method of the `JpaRead` class again to see how the Hibernate query
changed. You should see two queries this time where the first query only fetches
the `Student` data and the second query fetches the associated card data.

```
Hibernate:
    select
        student0_.id as id1_1_0_,
        student0_.card_id as card_id5_1_0_,
        student0_.dob as dob2_1_0_,
        student0_.name as name3_1_0_,
        student0_.studentGroup as studentg4_1_0_
    from
        STUDENT_DATA student0_
    where
        student0_.id=?
```

```
Hibernate:
    select
        idcard0_.id as id1_0_0_,
        idcard0_.isActive as isactive2_0_0_,
        student1_.id as id1_1_1_,
        student1_.card_id as card_id5_1_1_,
        student1_.dob as dob2_1_1_,
        student1_.name as name3_1_1_,
        student1_.studentGroup as studentg4_1_1_
    from
        ID_CARD idcard0_
    left outer join
        STUDENT_DATA student1_
            on idcard0_.id=student1_.card_id
    where
        idcard0_.id=?
```

## Conclusion

We have learned how to create one-to-one relationships between entities and
fetch associated data efficiently in this lesson.

## References

- [Java Persistence/Relationships](https://en.wikibooks.org/wiki/Java_Persistence/Relationships)
- [Understanding JPA, Part 2: Relationships the JPA Way](https://www.infoworld.com/article/2077819/understanding-jpa-part-2-relationships-the-jpa-way.html)
