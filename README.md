# Library Management System

This project is a simple Library Management System built using PHP and the Slim framework. It provides endpoints for user registration, authentication, and managing authors and books. The system uses JWT (JSON Web Tokens) for authentication and token management.

## Features

- User Registration and Authentication
- Author Management (Register, Show, Delete)
- Book Management (Register, Update, Delete)
- Token Management (Generate, Validate, Mark as Used)

## Prerequisites

- PHP 7.4 or higher
- Composer
- MySQL

## Installation

1. Clone the repository:

    ```bash
    git clone https://github.com/ELDRagados/Library.git
    cd library-management-system
    ```

2. Install dependencies:

    ```bash
    composer install
    ```

3. Set up the database:

    - Create a MySQL database named `library`.

    - Run the following SQL scripts to create the necessary tables:

    ```sql
    CREATE TABLE users (
        userid INT AUTO_INCREMENT PRIMARY KEY,
        username VARCHAR(255) NOT NULL,
        password VARCHAR(255) NOT NULL
    );

    CREATE TABLE authors (
        authorid INT AUTO_INCREMENT PRIMARY KEY,
        name VARCHAR(255) NOT NULL
    );

    CREATE TABLE books (
        bookid INT AUTO_INCREMENT PRIMARY KEY,
        title VARCHAR(255) NOT NULL,
        authorid INT NOT NULL,
        FOREIGN KEY (authorid) REFERENCES authors(authorid)
    );

    CREATE TABLE tokens (
        id INT AUTO_INCREMENT PRIMARY KEY,
        token VARCHAR(255) NOT NULL,
        userid INT NOT NULL,
        status ENUM('active', 'revoked', 'expired') DEFAULT 'active',
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
        used_at TIMESTAMP NULL,
        FOREIGN KEY (userid) REFERENCES users(userid)
    );
    ```

4. Configure the database connection:

    Update the database connection details in `index.php`:

    ```php
    <?php
    $servername = "localhost";
    $username = "root";
    $password = "";
    $dbname = "library";
    ?>
    ```

## Usage

### User Registration

- **Endpoint:** `/user/register`
- **Method:** `POST`
- **Payload:**

    ```json
    {
      "username": "your_username",
      "password": "your_password"
    }
    ```

### User Authentication

- **Endpoint:** `/user/auth`
- **Method:** `POST`
- **Payload:**

    ```json
    {
      "username": "your_username",
      "password": "your_password"
    }
    ```

### Author Management

#### Register Author

- **Endpoint:** `/author/register`
- **Method:** `POST`
- **Payload:**

    ```json
    {
      "token": "your_jwt_token",
      "name": "author_name"
    }
    ```

#### Show Authors

- **Endpoint:** `/author/show`
- **Method:** `GET`

#### Delete Author

- **Endpoint:** `/author/delete`
- **Method:** `DELETE`
- **Payload:**

    ```json
    {
      "token": "your_jwt_token",
      "authorid": 1
    }
    ```

### Book Management

#### Register Book

- **Endpoint:** `/book/register`
- **Method:** `POST`
- **Payload:**

    ```json
    {
      "token": "your_jwt_token",
      "title": "book_title",
      "authorid": 1
    }
    ```

#### Update Book

- **Endpoint:** `/book/update`
- **Method:** `PUT`
- **Payload:**

    ```json
    {
      "token": "your_jwt_token",
      "bookid": 1,
      "title": "new_book_title",
      "authorid": 1
    }
    ```

#### Delete Book

- **Endpoint:** `/book/delete`
- **Method:** `DELETE`
- **Payload:**

    ```json
    {
      "token": "your_jwt_token",
      "bookid": 1
    }
    ```

### Token Management

- **Generate Token:** Tokens are generated during user authentication and stored in the `tokens` table with a status of 'active'.

- **Validate Token:** Tokens are validated using the `validateToken` function, which checks the token's status and decodes it to retrieve the user ID.

- **Mark Token as Used:** Tokens are marked as used by updating their status to 'revoked' and setting the `used_at` timestamp.

## Code Excerpt

Here is an excerpt from `index.php` that shows how tokens are managed:

```php
<?php
$password = "";
try {
    $conn = new PDO("mysql:host=$servername;dbname=$dbname", $username, $password);
    $conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

    $sql = "INSERT INTO tokens (token, userid, status) VALUES (:token, :userid, 'active')";
    $stmt = $conn->prepare($sql);
    $stmt->bindParam(':token', $token);
    $stmt->bindParam(':userid', $userid);
    $stmt->execute();
} catch (PDOException $e) {
    // Handle exception
}

return $token;
}

function validateToken($token) {
    global $key;
    $servername = "localhost";
    $username = "root";
    $password = "";
    $dbname = "library";

    try {
        $conn = new PDO("mysql:host=$servername;dbname=$dbname", $username, $password);
        $conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

        $sql = "SELECT * FROM tokens WHERE token = :token AND status = 'active'";
        $stmt = $conn->prepare($sql);
        $stmt->bindParam(':token', $token);
        $stmt->execute();
        $data = $stmt->fetch(PDO::FETCH_ASSOC);

        if ($data) {
            // Decode the token to get the payload
            $decoded = JWT::decode($token, new Key($key, 'HS256'));
            return $decoded->data->userid;
        } else {
            return false;
        }
    } catch (PDOException $e) {
        // Handle exception
        return false;
    }
}

License
This project is licensed under the MIT License. See the LICENSE file for details.
