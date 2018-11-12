# sql
The `REFERENCES` keyword is part of a foreign key constraint and it causes MySQL to require that the value(s) in the specified column(s) of the referencing table are also present in the specified column(s) of the referenced table.

This prevents foreign keys from referencing ids that do not exist or were deleted, and it can optionally prevent you from deleting rows whilst they are still referenced.

A specific example is if every employee must belong to a department then you can add a foreign key constraint from `employee.departmentid` referencing `department.id`.

Run the following code to create two test tables `tablea` and `tableb` where the column `a_id` in `tableb` references the primary key of `tablea`. `tablea` is populated with a few rows.

```sql
CREATE TABLE tablea (
  id INT PRIMARY KEY,
  foo VARCHAR(100) NOT NULL
) Engine = InnoDB;

INSERT INTO tablea (id, foo) VALUES
(1, 'foo1'),
(2, 'foo2'),
(3, 'foo3');

CREATE TABLE tableb (
  id INT PRIMARY KEY,
  a_id INT NOT NULL,
  bar VARCHAR(100) NOT NULL,
  FOREIGN KEY fk_b_a_id (a_id) REFERENCES tablea (id)
) Engine = InnoDB;
```

Now try these commands:

```sql
INSERT INTO tableb (id, a_id, bar) VALUES (1, 2, 'bar1');
-- succeeds because there is a row in tablea with id 2

INSERT INTO tableb (id, a_id, bar) VALUES (2, 4, 'bar2');
-- fails  because there is not a row in tablea with id 4

DELETE FROM tablea WHERE id = 1;
-- succeeds because there is no row in tableb which references this row

DELETE FROM tablea WHERE id = 2;
-- fails because there is a row in tableb which references this row
```

Important note: Both tables must be InnoDB tables or the constraint is ignored.

```sql
CREATE TABLE artists (
  id    INTEGER PRIMARY KEY, 
  name  TEXT
);

CREATE TABLE tracks (
  traid     INTEGER, 
  title   TEXT, 
  artist INTEGER,
  FOREIGN KEY(artist) REFERENCES artists(id)
);

INSERT INTO artists VALUES (1, "Tom Chapin");
INSERT INTO artists VALUES (2, "Harry Chapin");

/* Tom's songs */
INSERT INTO tracks VALUES (1, "Great Big Words", 1);
INSERT INTO tracks VALUES (1, "This Pretty Planet", 1);
/* Harry's songs */
INSERT INTO tracks VALUES (2, "Cat's in the Cradle", 2);

/* It won't let us do this, because 3 isn't an artist id:
INSERT INTO tracks VALUES (2, "Cat's in the Cradle", 3);
*/

SELECT * FROM artists;
SELECT * FROM tracks;
```

*One-to-one*: Use a foreign key to the referenced table:
```
student: student_id, first_name, last_name, address_id
address: address_id, address, city, zipcode, student_id # you can have a
                                                        # "link back" if you need
```

*One-to-many*: Use a foreign key on the many side of the relationship linking back to the "one" side:
```
teachers: teacher_id, first_name, last_name # the "one" side
classes:  class_id, class_name, teacher_id  # the "many" side
```

*Many-to-many*: Use a junction table
```
student: student_id, first_name, last_name
classes: class_id, name, teacher_id
student_classes: class_id, student_id     # the junction table
```

Example queries:
```sql
-- Getting all students for a class:

SELECT s.student_id, last_name
  FROM student_classes sc 
INNER JOIN students s ON s.student_id = sc.student_id
  WHERE sc.class_id = X

-- Getting all classes for a student: 

SELECT c.class_id, name
  FROM student_classes sc 
INNER JOIN classes c ON c.class_id = sc.class_id
  WHERE sc.student_id = Y
```