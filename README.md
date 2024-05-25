# php-update
<?php
// Establish connection to the database
$servername = "localhost";
$dbusername = "root"; // Replace with your database username
$dbpassword = ""; // Replace with your database password
$dbname = "javascprit"; // Replace with your database name

// Create connection
$conn = new mysqli($servername, $dbusername, $dbpassword, $dbname);

// Check connection
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}

$response = array();

if ($_SERVER['REQUEST_METHOD'] === 'GET') {
    // Handle search functionality
    $username = $_GET['searchusername'] ?? '';

    if (empty($username)) {
        $response = array('status' => 'error', 'message' => 'Please provide a username');
    } else {
        // Fetch user data
        $stmt = $conn->prepare("SELECT username, email FROM users WHERE username = ?");
        if ($stmt) {
            $stmt->bind_param("s", $username);
            $stmt->execute();
            $result = $stmt->get_result();

            if ($result->num_rows > 0) {
                $user = $result->fetch_assoc();
                $response = array('status' => 'success', 'user' => $user);
            } else {
                $response = array('status' => 'error', 'message' => 'User not found');
            }

            $stmt->close();
        } else {
            $response = array('status' => 'error', 'message' => 'Database query error');
        }
    }
} elseif ($_SERVER['REQUEST_METHOD'] === 'POST') {
    // Handle update functionality
    $data = json_decode(file_get_contents('php://input'), true);
    $username = $data['username'] ?? '';

    if (empty($username)) {
        $response = array('status' => 'error', 'message' => 'Username is required');
    } else {
        $fields = [];
        $values = [];

        foreach ($data as $key => $value) {
            if ($key !== 'username' && !empty($value)) {
                $fields[] = "$key = ?";
                $values[] = $value;
            }
        }

        if (count($fields) > 0) {
            $values[] = $username;
            $stmt = $conn->prepare("UPDATE users SET " . implode(", ", $fields) . " WHERE username = ?");
            if ($stmt) {
                $stmt->bind_param(str_repeat('s', count($values)), ...$values);
                if ($stmt->execute()) {
                    $response = array('status' => 'success', 'message' => 'Fields updated successfully');
                } else {
                    $response = array('status' => 'error', 'message' => 'Failed to update fields');
                }

                $stmt->close();
            } else {
                $response = array('status' => 'error', 'message' => 'Database query error');
            }
        } else {
            $response = array('status' => 'error', 'message' => 'No fields to update');
        }
    }
}

header('Content-Type: application/json');
echo json_encode($response);

// Close connection
$conn->close();
?>

<!DOCTYPE html>
<html>
<head>
    <link rel="stylesheet" href="C:\xampp\htdocs\scratch\style.css">
    <title>scratch</title>
</head>
<body>
    <div class="container">
        <form id="retrieveForm" onsubmit="handleFormSubmit(event)">
            <input type="text" id="searchusername" name="username" placeholder="Enter username to search">
            <input type="submit" value="Search"><br><br>

            <label for="username">Username</label><br><br>
            <input type="text" id="username" name="username" placeholder="username"><br><br>
            <label for="email">Email</label><br><br>
            <input type="text" id="email" name="email" placeholder="email"><br><br>

            <input type="button" value="Update" onclick="update()">
        </form>
    </div>
    <script>
        function handleFormSubmit(event) {
            event.preventDefault();
            var searchusername = document.getElementById("searchusername").value;

            fetch('update.php?searchusername=' + encodeURIComponent(searchusername))
            .then(response => response.json())
            .then(data => {
                if (data.status === 'success') {
                    document.getElementById('username').value = data.user.username;
                    document.getElementById('email').value = data.user.email;
                } else {
                    alert(data.message);
                }
            })
            .catch(error => console.error('Error:', error));
        }

        function update() {
            var formElements = document.querySelectorAll('#retrieveForm input');
            var formData = {};

            formElements.forEach(function(element) {
                if (element.name && element.value) {
                    formData[element.name] = element.value;
                }
            });

            fetch('update.php', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify(formData)
            })
            .then(response => response.json())
            .then(data => {
                if (data.status === 'success') {
                    alert('Fields updated successfully');
                    // Clear the text fields
                    formElements.forEach(function(element) {
                            element.value = '';
                        });
                } else {
                    alert(data.message);
                }
            })
            .catch(error => console.error('Error:', error));
        }
    </script>
</body>
</html>
