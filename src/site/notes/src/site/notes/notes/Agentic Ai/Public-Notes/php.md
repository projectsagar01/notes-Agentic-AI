---
{"dg-publish":true,"permalink":"/src/site/notes/notes/Agentic Ai/Public-Notes/php/","dg-note-properties":{}}
---

Absolutely. 👍 If your test is tomorrow, let’s learn PHP CRUD in the simplest possible way.

CRUD =

C → Create (insert data)
R → Read (show data)
U → Update (edit data)
D → Delete (remove data)

I can teach you by making one small Student CRUD project that you can practice and reproduce in your test.

1. Create the database

In phpMyAdmin → SQL, run:

CREATE DATABASE school;

USE school;

CREATE TABLE students (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL,
    course VARCHAR(100) NOT NULL
);

2. Create your project

If you're using XAMPP, create:

htdocs/
└── crud/
    ├── db.php
    ├── index.php
    ├── add.php
    ├── edit.php
    └── delete.php

3. Database connection — db.php
<?php

$conn = mysqli_connect("localhost", "root", "", "school");

if (!$conn) {
    die("Connection failed: " . mysqli_connect_error());
}
?>


Remember this pattern:

mysqli_connect(host, username, password, database)

4. Create — add.php
<?php
include 'db.php';

if (isset($_POST['submit'])) {

    $name = $_POST['name'];
    $email = $_POST['email'];
    $course = $_POST['course'];

    $sql = "INSERT INTO students (name, email, course)
            VALUES ('$name', '$email', '$course')";

    mysqli_query($conn, $sql);

    header("Location: index.php");
}
?>

<!DOCTYPE html>
<html>
<head>
    <title>Add Student</title>
</head>
<body>

<h2>Add Student</h2>

<form method="POST">

    Name:
    <input type="text" name="name" required>
    <br><br>

    Email:
    <input type="email" name="email" required>
    <br><br>

    Course:
    <input type="text" name="course" required>
    <br><br>

    <button type="submit" name="submit">Add Student</button>

</form>

</body>
</html>


The important part for Create is:

$sql = "INSERT INTO students (name, email, course)
        VALUES ('$name', '$email', '$course')";

mysqli_query($conn, $sql);

5. Read — index.php
<?php
include 'db.php';

$sql = "SELECT * FROM students";
$result = mysqli_query($conn, $sql);
?>

<!DOCTYPE html>
<html>
<head>
    <title>Student CRUD</title>
</head>
<body>

<h2>Student List</h2>

<a href="add.php">Add Student</a>

<br><br>

<table border="1" cellpadding="10">

<tr>
    <th>ID</th>
    <th>Name</th>
    <th>Email</th>
    <th>Course</th>
    <th>Action</th>
</tr>

<?php while ($row = mysqli_fetch_assoc($result)) { ?>

<tr>

    <td><?php echo $row['id']; ?></td>

    <td><?php echo $row['name']; ?></td>

    <td><?php echo $row['email']; ?></td>

    <td><?php echo $row['course']; ?></td>

    <td>
        <a href="edit.php?id=<?php echo $row['id']; ?>">Edit</a>

        <a href="delete.php?id=<?php echo $row['id']; ?>">Delete</a>
    </td>

</tr>

<?php } ?>

</table>

</body>
</html>


The important part for Read:

$sql = "SELECT * FROM students";
$result = mysqli_query($conn, $sql);

while ($row = mysqli_fetch_assoc($result)) {
    echo $row['name'];
}

6. Update — edit.php
<?php
include 'db.php';

$id = $_GET['id'];

$sql = "SELECT * FROM students WHERE id=$id";
$result = mysqli_query($conn, $sql);
$row = mysqli_fetch_assoc($result);

if (isset($_POST['update'])) {

    $name = $_POST['name'];
    $email = $_POST['email'];
    $course = $_POST['course'];

    $sql = "UPDATE students SET
            name='$name',
            email='$email',
            course='$course'
            WHERE id=$id";

    mysqli_query($conn, $sql);

    header("Location: index.php");
}
?>

<!DOCTYPE html>
<html>
<body>

<h2>Edit Student</h2>

<form method="POST">

    Name:
    <input type="text" name="name"
           value="<?php echo $row['name']; ?>">
    <br><br>

    Email:
    <input type="email" name="email"
           value="<?php echo $row['email']; ?>">
    <br><br>

    Course:
    <input type="text" name="course"
           value="<?php echo $row['course']; ?>">
    <br><br>

    <button type="submit" name="update">Update</button>

</form>

</body>
</html>


The important part:

UPDATE students SET
name='$name',
email='$email',
course='$course'
WHERE id=$id

7. Delete — delete.php
<?php

include 'db.php';

$id = $_GET['id'];

$sql = "DELETE FROM students WHERE id=$id";

mysqli_query($conn, $sql);

header("Location: index.php");

?>


That's the whole CRUD. 🎯

🧠 Memorize this for your test
CRUD	SQL	PHP idea
Create	INSERT	Add new record
Read	SELECT	Display records
Update	UPDATE	Edit record
Delete	DELETE	Remove record

The basic flow you should remember is:

FORM
 ↓
$_POST / $_GET
 ↓
SQL QUERY
 ↓
mysqli_query()
 ↓
DATABASE
 ↓
DISPLAY RESULT


If you want, I can also give you a complete PHP CRUD test question + answer, exactly like something a teacher might give you tomorrow, and walk you through it step-by-step from creating the database to running it in XAMPP.