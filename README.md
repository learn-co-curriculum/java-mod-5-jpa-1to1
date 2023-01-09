# One-to-One Relationship

## Learning Goals

- Create one-to-one associations between entities with JPA annotations.
- Pick one entity to be the relationship owner and establish the relationship as bidirectional.
- Perform eager and lazy fetching of related data from the database.

## Code Along

We will continue with the `Student` entity from the previous lessons.

## Introduction

Relational database tables can be built such that they associate with one
another via primary and foreign keys. JPA provides us with annotations that
streamline model association in our program.

In this lesson, we will  implement the following one-to-one relationship:

![One to One relationship erd](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-jpa-1to1/student_idcard_erd.png)

- Each `Student` has one `IdCard` and each `IdCard` belongs to one `Student`. 
- We will use the JPA `@OneToOne` annotation to create the one-to-one relationship.  
- The `Student` entity will be chosen as the relationship owner, meaning a foreign key reference to
  the associated `IdCard` entity will be stored in the student database table.

## Create ID Card Entity and Instances

Create the `IdCard` class in the `model` package and add the following code.
The `@Table` annotation names the corresponding database table `ID_CARD`.

```java
package org.example.model;

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
The project structure should now look like this:

![idcard project structure](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-jpa-1to1/idcard_projectstructure.png)

Let's remove the `@Table` annotation from the `Student` class.
JPA will use the class name `Student` as the default name of the database table
(PostgreSQL names the table with uppercase letters as `STUDENT`).

```java
@Entity
public class Student {
    @Id
    @GeneratedValue
    private int id;

    private String name;

    private LocalDate dob;

    @Enumerated(EnumType.STRING)
    private StudentGroup studentGroup;

    //getters, setters, toString
    ...
}
```

### Persisting `IdCard` objects to the database

1. Edit the `JpaCreate` class, create three new ID cards, and save them in the
database as shown in the code below.
2. Edit `hibernate.hbm2ddl.auto` in `persistence.xml` to set the property to `create`.
3. Run `JpaCreate.main` to recreate the database schema based on the `Student` and `IdCard` entity classes.


```java
package org.example;

import org.example.model.IdCard;
import org.example.model.Student;
import org.example.model.StudentGroup;

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
      transaction.commit();

      //close entity manager and factory
      entityManager.close();
      entityManagerFactory.close();
   }
}
```

The Hibernate output shows the creation of the `ID_CARD` table and the `STUDENT` table:

```text
Hibernate: 
    
    create table ID_CARD (
       id int4 not null,
        isActive boolean not null,
        primary key (id)
    )
Hibernate: 
    
    create table STUDENT (
       id int4 not null,
        dob date,
        name varchar(255),
        studentGroup varchar(255),
        primary key (id)
    )
```

We can also see the primary key auto-generation and insert statements for the three students
and three id card entities:

```text
Hibernate: 
    select
        nextval ('hibernate_sequence')
Hibernate: 
    select
        nextval ('hibernate_sequence')
Hibernate: 
    select
        nextval ('hibernate_sequence')
Hibernate: 
    select
        nextval ('hibernate_sequence')
Hibernate: 
    select
        nextval ('hibernate_sequence')
Hibernate: 
    select
        nextval ('hibernate_sequence')
Hibernate: 
    insert 
    into
        STUDENT
        (dob, name, studentGroup, id) 
    values
        (?, ?, ?, ?)
Hibernate: 
    insert 
    into
        STUDENT
        (dob, name, studentGroup, id) 
    values
        (?, ?, ?, ?)
Hibernate: 
    insert 
    into
        STUDENT
        (dob, name, studentGroup, id) 
    values
        (?, ?, ?, ?)
Hibernate: 
    insert 
    into
        ID_CARD
        (isActive, id) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        ID_CARD
        (isActive, id) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        ID_CARD
        (isActive, id) 
    values
        (?, ?)
```


Query the `STUDENT` and `ID_CARD` tables using the **pgAdmin** query tool:

`SELECT * FROM STUDENT;`

| ID  | DOB        | NAME | STUDENTGROUP |
|-----|------------|------|--------------|
| 1   | 2000-01-01 | Jack | ROSE         |
| 2   | 1999-01-01 | Lee  | DAISY        |
| 3   | 1980-01-01 | Amal | LOTUS        |

`SELECT * FROM ID_CARD;`

| ID  | ISACTIVE |
|-----|----------|
| 4   | true     |
| 5   | false    |
| 6   | false    |

Notice the `STUDENT` primary key values are 1,2 and 3, while the `ID_CARD`
primary key values are 4, 5, and 6.  PostgreSQL appears to use a single
sequence generator for both tables.  Other DBMS software may use a separate
sequence generator for each table.

## Modeling the Relationship

Here is the one-to-one relationship we are modeling for reference:

![One to One relationship erd](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-jpa-1to1/student_idcard_erd.png)

Both the `Student` and `IdCard` classes have to be modified to model the
relationship. Here are the steps we have to follow:

1. Create fields in both classes annotated with `@OneToOne` to store the associated instance.
   - Class `Student` will have a new instance variable `IdCard card` to reference the associated id card object.   
    ```java
    @OneToOne
    private IdCard card;
    ```
   - Class `IdCard` will have a new instance variable `Student student` to reference the associated student object.    
    ```java
    @OneToOne
    private Student student;
    ```
2. Generate getter/setter methods for the new fields in both classes.
3. Define the entity on the owning side of the relationship (Student). 
   Add the `mappedBy` attribute in the `@OneToOne` annotation to the entity on the non-owing side (IdCard).
   We will do this step later in the lesson.
    ```java
    @OneToOne(mappedBy = "card")
    private Student student;
    ```

First, let's edit `Student` to add the `card` field and getter/setter methods as shown:

```java
package org.example.model;

import javax.persistence.*;
import java.time.LocalDate;

@Entity
public class Student {
   @Id
   @GeneratedValue
   private int id;

   private String name;

   private LocalDate dob;

   @Enumerated(EnumType.STRING)
   private StudentGroup studentGroup;

   @OneToOne
   private IdCard card;

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

Next, edit `IdCard` to add the new `student` field and getter/setter methods
as shown below. Initially, we will omit the `mappedBy` property to see what happens when both
classes have `@OneToOne` annotations without mapping. 

```java
package org.example.model;

import javax.persistence.*;

@Entity
@Table(name = "ID_CARD")
public class IdCard {
   @Id
   @GeneratedValue(strategy = GenerationType.AUTO)
   private int id;

   private boolean isActive;

   @OneToOne
   private Student student;

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

   public Student getStudent() {
      return student;
   }

   public void setStudent(Student student) {
      this.student = student;
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

We have added the `card` field to the `Student` class and the `student`
field to the `IdCard` class, along with getter/setter methods for the new fields. 

Run `JpaCreate.main` again to recreate the tables based on the new
fields.  Query the tables in **pgAdmin** to confirm there is
a new column with null values for the relationship in each table.
The fields are null because we did not assign the relationships
between objects in `JpaCreate`.

`SELECT * FROM STUDENT;`

| ID   | DOB         | NAME  | STUDENTGROUP  | CARD_ID  |
|------|-------------|-------|---------------|----------|
| 1    | 2000-01-01  | Jack  | ROSE          | null     |
| 2    | 1999-01-01  | Lee   | DAISY         | null     |
| 3    | 1980-01-01  | Amal  | LOTUS         | null     |

`SELECT * FROM ID_CARD;`

| ID   | ISACTIVE  | STUDENT_ID  |
|------|-----------|-------------|
| 4    | true      | null        |
| 5    | false     | null        |
| 6    | false     | null        |


## Assigning an owner to the relationship

If we simply put a `@OneToOne` annotation on
both the properties, the database will create foreign keys in both tables, but
it might be possible to set the fields incorrectly and not maintain the one-to-one relationship.

For example, we could set `card_id` value to `4` for the student with id `1`
(updating the first row in the `STUDENT` table), and accidentally set the
`student_id` value to `3` instead of `1` for the card with id `4`
(updating the first row in the `ID_CARD` table).
This would violate the one-to-one relationship.

Instead, we need to enforce that the `card` and `student` property in the two entities are
connected to each other, i.e., they are the same relationship. The best way to do this
is to store the relationship in one place in the database.


When defining a relationship between two entities,
a common task is identifying the **owning side** of the relationship.

- The entity on the owning side is referred to as the  **relationship owner**
  and a foreign key referencing the entity on the **non-owning**
  side is stored in their database table.
- The entity on the non-owning side will **not** have a foreign key column in their table.

![one to one relationship student owning side](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-jpa-1to1/one_to_one_student_owning_side.png)

In the one-to-one relationship between `Student` and `IdCard`,
we will pick `Student` as the owning side of the relationship.

- The `Student` entity will own the relationship, which means
  the `STUDENT` table has a foreign key column `card_id`
  containing the id of the associated `IdCard` entity.
- The `IdCard` entity is on the non-owning side. The  `IdCard` class
  assigns the `mappedBy` attribute to `card` to establish the relationship as bidirectional.
  - The `ID_CARD` table does not store a foreign key reference to `STUDENT`.
  - JPA will use the `card` field found in the `Student` entity to figure out the student
    associated with a given `IdCard` entity.
      

Edit `IdCard` to set the `mappedBy` property of the one-to-one relationship to `card`:

```java
package org.example.model;

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

    //getters/setters/toString
```

JPA will enforce the relationship to be one-to-one by omitting the student
foreign key column in the `ID_CARD` table.  The relationship will be
stored as a column in the `STUDENT` table only.

Since `Student` owns the relationship, we update `JpaCreate`
to associate a specific `IdCard` entity for each `Student` entity
by calling the `setCard(Card card)` method.
This should be done before persisting the entities.

```java
package org.example;

import org.example.model.IdCard;
import org.example.model.Student;
import org.example.model.StudentGroup;

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

      // create student-card associations
      student1.setCard(card1);
      student2.setCard(card2);
      student3.setCard(card3);

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
      transaction.commit();

      //close entity manager and factory
      entityManager.close();
      entityManagerFactory.close();
   }
}
```

If we run `JpaCreate.main` and query the tables in **pgAdmin**, the tables will look like
the following:

`SELECT * FROM STUDENT;`

| ID  | DOB        | NAME | STUDENTGROUP | CARD_ID |
|-----|------------|------|--------------|---------|
| 1   | 2000-01-01 | Jack | ROSE         | 4       |
| 2   | 1999-01-01 | Lee  | DAISY        | 5       |
| 3   | 1980-01-01 | Amal | LOTUS        | 6       |

`SELECT * FROM ID_CARD;` 

| ID  | ISACTIVE |
|-----|----------|
| 4   | true     |
| 5   | false    |
| 6   | false    |

Notice the `ID_CARD` table no longer stores the `Student` id.
Since the relationship is one-to-one and bidirectional, we can always figure out
which student is associated with a specific id card by querying
the student table.

## Fetching Data with Relationships

JPA performs join operations automatically to get related data from an entity.
For one-to-one relationships, it performs the join operation on the first query,
but we can defer this operation until the data is accessed in the program
using the `fetch` property on the `@OneToOne` annotation.

1. Change the `hibernate.hbm2ddl.auto` property in the `persistence.xml` to `none`
   before performing read operations. We will be querying data in the `JpaRead` class.
2. Edit the `JpaRead` class and add the following code to get and print the student's id card:

```java
package org.example;

import org.example.model.IdCard;
import org.example.model.Student;
import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.Persistence;

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

      // close entity manager and factory
      entityManager.close();
      entityManagerFactory.close();
   }
}
```

Run the `JpaRead.main` method. If you look at the terminal output, you can
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
        STUDENT student0_ 
    left outer join
        ID_CARD idcard1_ 
            on student0_.card_id=idcard1_.id 
    where
        student0_.id=?
```

The print statement output shows the result of calling the `toString()`
methods for both the `Student` object and the `IdCard` object:

```text
Student{id=1, name='Jack', dob=2000-01-01, studentGroup=ROSE}
IdCard{id=4, isActive=true}
```

### Defer Related Data Fetching

We can modify the behavior such that the join is performed when the `getCard()`
method is called on a `Student` instance. This is beneficial if a model has lots
of associations which would make the initial join operation slow.

Open the `Student` class and add the `(fetch = FetchType.LAZY)` property to the
`@OneToOne` annotation:

```java
package org.example.model;

import javax.persistence.*;
import java.time.LocalDate;

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

   //getters/setters/toString
```

A one-to-one association uses the `FetchType.EAGER` value as a default if no value is
specified. By setting it to `FetchType.LAZY`, we can defer the join query
until the `getCard()` method is called. 

Run the `main` method of the `JpaRead` class again to see how the Hibernate query
changed. You should see two queries this time where the first query only fetches
the `Student` data and the second query fetches the associated card data, with the
`toString()` print output occurring between the queries.

```
Hibernate: 
    select
        student0_.id as id1_1_0_,
        student0_.card_id as card_id5_1_0_,
        student0_.dob as dob2_1_0_,
        student0_.name as name3_1_0_,
        student0_.studentGroup as studentg4_1_0_ 
    from
        STUDENT student0_ 
    where
        student0_.id=?
Student{id=1, name='Jack', dob=2000-01-01, studentGroup=ROSE}
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
        STUDENT student1_ 
            on idcard0_.id=student1_.card_id 
    where
        idcard0_.id=?
IdCard{id=4, isActive=true}
```

## Caution when working with circular references

Since `Student` and `IdCard` both contain a reference to the other class, we have
what is termed **circular** or **cyclic** references.


```java
@Entity
public class Student {
   ...

    @OneToOne(fetch = FetchType.LAZY)
    private IdCard card;
   
   ...

```


```java
@Entity
@Table(name = "ID_CARD")
public class IdCard {
   ...

    @OneToOne(mappedBy = "card")
    private Student student;
   
   ...

```


It is important to avoid an infinite cycle of method calls
if you have IntelliJ generate the `toString()` methods for both classes.
For example, IntelliJ generates this method for the `Student`
class, which calls the `toString()` method for the `card`  object reference on the last line
(don't update your code!):

```java
    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", dob=" + dob +
                ", studentGroup=" + studentGroup +
                ", card=" + card +
                '}';
    }
```

IntelliJ generates this method for the `Card` class, which calls the `toString()` method for
the `student` object reference on the last line (don't update your code!):

```java
@Override
    public String toString() {
        return "IdCard{" +
                "id=" + id +
                ", isActive=" + isActive +
                ", student=" + student +
                '}';
    }
```

Since the relationship is cyclic, the `toString()` methods generated
by IntelliJ include an infinite cycle of method calls.
In the final code solution, we removed the object references
from both `toString()` methods, but it would have been sufficient
to remove an object reference from just one of the methods:

The `Student` class:

```java
@Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", dob=" + dob +
                ", studentGroup=" + studentGroup +
                '}';
    }
```

The `IdCard` class:

```java
    @Override
    public String toString() {
        return "IdCard{" +
                "id=" + id +
                ", isActive=" + isActive +
                '}';
    }
```

## Code Check

Since we have modified and worked on different files, here are the final
versions of `Student`, `IdCard`, and `JpaCreate` for reference.

#### Student

```java
package org.example.model;

import javax.persistence.*;
import java.time.LocalDate;

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


#### IdCard

```java
package org.example.model;

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

    public Student getStudent() {
        return student;
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

Since `Student` owns the relationship, it is best to either remove the `setStudent` method from the `IdCard` class,
or edit `setStudent` to call the `setIdCard` on the student parameter as well.
We removed the `setStudent` method to force the relationship to always
be established through the relationship owner `Student`.

#### JpaCreate


```java
package org.example;

import org.example.model.IdCard;
import org.example.model.Student;
import org.example.model.StudentGroup;

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

        // create student-card associations
        student1.setCard(card1);
        student2.setCard(card2);
        student3.setCard(card3);

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
        transaction.commit();

        //close entity manager and factory
        entityManager.close();
        entityManagerFactory.close();
    }
}
```

## Conclusion

We have learned how to create one-to-one relationships between entities and
fetch associated data efficiently in this lesson.


To implement a one-to-one relationship:

- One entity is chosen as the relationship owner.
- The table corresponding to the owning side will store the foreign key reference.
- The owning side Java class adds a new field with the annotation `@OneToOne`.
    - The field stores a single reference to the non-owning side entity.
    - The `fetch` attribute is optional, but can be used to improve efficiency by delaying
      when the non-owning side entity is fetched.
- The non-owning side Java class adds a new field with the annotation `@OneToOne.`
    - The field stores a single reference to the owning side entity.
    - The `mappedBy` property references the `@OneToOne` field that was added to the owning side class.

![one to one relationship student owning side](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-jpa-1to1/one_to_one_student_owning_side.png)

We picked `Student` as the relationship owner.

- The owning side `Student` class has a field with the annotation `@OneToOne`.
  We also implemented lazy fetching of the associated `IdCard` entity.
  The `Student` class stores the relationship as:       
  ```java
  @Entity
  @Table(name = "STUDENT")
  public class Student {
     ...
     
     @OneToOne(fetch = FetchType.LAZY)
     private IdCard card;
     
     ...
  }
  ```
- The non-owning side `IdCard` class has a field with the annotation `@OneToOne` with the
  `mappedBy` attribute set to `card`, which is the relationship field defined
  by the owning side entity `Student`. A student is associated with one id card, and
  we use the `mappedBy` property to ensure the id card is associated with the same student.
  ```java
  @Entity
  @Table(name = "ID_CARD")
  public class IdCard {
    ...
    
    @OneToOne(mappedBy = "card")
    private Student student;
    
    ...
  }
  ```

## References

- [Java Persistence/Relationships](https://en.wikibooks.org/wiki/Java_Persistence/Relationships)  
- [Understanding JPA, Part 2: Relationships the JPA Way](https://www.infoworld.com/article/2077819/understanding-jpa-part-2-relationships-the-jpa-way.html)
