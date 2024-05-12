import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.Scanner;

/**
 * A simple Hotel Reservation System that allows users to reserve, view, update, and delete reservations.
 */
public class HotelReservationSystem {

    // Database connection parameters
    private static final String url = "jdbc:mysql://localhost:3306/hotel_db";
    private static final String username = "root";
    private static final String password = "Ap@123456";

    /**
     * The main method that serves as the entry point for the program.
     *
     * @param args The command-line arguments.
     */
    public static void main(String[] args) {
        try {
            // Establish a connection to the database
            Connection connection = DriverManager.getConnection(url, username, password);
            Scanner scanner = new Scanner(System.in);

            // Main menu loop
            while (true) {
                System.out.println();
                System.out.println("HOTEL MANAGEMENT SYSTEM");
                System.out.println("1. Reserve a room");
                System.out.println("2. View Reservations");
                System.out.println("3. Get Room Number");
                System.out.println("4. Update Reservations");
                System.out.println("5. Delete Reservations");
                System.out.println("0. Exit");
                System.out.print("Choose an option: ");
                int choice = scanner.nextInt();
                scanner.nextLine(); // consume newline left-over

                // Perform action based on user's choice
                switch (choice) {
                    case 1:
                        reserveRoom(connection, scanner);
                        break;
                    case 2:
                        viewReservations(connection);
                        break;
                    case 3:
                        getRoomNumber(connection, scanner);
                        break;
                    case 4:
                        updateReservation(connection, scanner);
                        break;
                    case 5:
                        deleteReservation(connection, scanner);
                        break;
                    case 0:
                        exit();
                        scanner.close();
                        return;
                    default:
                        System.out.println("Invalid choice. Try again.");
                }
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    /**
     * Allows a user to reserve a room by entering guest details.
     *
     * @param connection The database connection.
     * @param scanner    The scanner object for user input.
     */
    private static void reserveRoom(Connection connection, Scanner scanner) {
        try {
            System.out.print("Enter guest name: ");
            String guestName = scanner.nextLine();
            System.out.print("Enter room number: ");
            int roomNumber = scanner.nextInt();
            scanner.nextLine(); // consume newline left-over
            System.out.print("Enter contact number: ");
            String contactNumber = scanner.next();

            // Prepare SQL statement for inserting reservation
            String sql = "INSERT INTO reservations (guest_name, room_number, contact_number) VALUES (?, ?, ?)";
            try (PreparedStatement statement = connection.prepareStatement(sql)) {
                statement.setString(1, guestName);
                statement.setInt(2, roomNumber);
                statement.setString(3, contactNumber);
                int affectedRows = statement.executeUpdate();

                // Check if reservation was successful
                if (affectedRows > 0) {
                    System.out.println("Reservation successful!");
                } else {
                    System.out.println("Reservation failed.");
                }
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    /**
     * Displays all existing reservations in a formatted table.
     *
     * @param connection The database connection.
     */
    private static void viewReservations(Connection connection) {
        String sql = "SELECT reservation_id, guest_name, room_number, contact_number, reservation_date FROM reservations";

        try (Statement statement = connection.createStatement();
             ResultSet resultSet = statement.executeQuery(sql)) {

            // Print table header
            System.out.println("Current Reservations:");
            System.out.println("+----------------+-----------------+---------------+----------------------+-------------------------+");
            System.out.println("| Reservation ID | Guest           | Room Number   | Contact Number      | Reservation Date        |");
            System.out.println("+----------------+-----------------+---------------+----------------------+-------------------------+");

            // Iterate over each row of the result set
            while (resultSet.next()) {
                // Retrieve reservation details from the result set
                int reservationId = resultSet.getInt("reservation_id");
                String guestName = resultSet.getString("guest_name");
                int roomNumber = resultSet.getInt("room_number");
                String contactNumber = resultSet.getString("contact_number");
                String reservationDate = resultSet.getTimestamp("reservation_date").toString();

                // Format and display the reservation data in a table-like format
                System.out.printf("| %-14d | %-15s | %-13d | %-20s | %-19s   |\n",
                        reservationId, guestName, roomNumber, contactNumber, reservationDate);
            }

            // Print table footer
            System.out.println("+----------------+-----------------+---------------+----------------------+-------------------------+");
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    /**
     * Retrieves the room number associated with a guest name.
     *
     * @param connection The database connection.
     * @param scanner    The scanner object for user input.
     */
    private static void getRoomNumber(Connection connection, Scanner scanner) {
        try {
            System.out.print("Enter guest name: ");
            String guestName = scanner.nextLine();

            String sql = "SELECT room_number FROM reservations WHERE guest_name = ?";
            try (PreparedStatement statement = connection.prepareStatement(sql)) {
                statement.setString(1, guestName);
                try (ResultSet resultSet = statement.executeQuery()) {
                    if (resultSet.next()) {
                        System.out.println("Room Number: " + resultSet.getInt("room_number"));
                    } else {
                        System.out.println("No reservation found for " + guestName);
                    }
                }
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    /**
     * Updates an existing reservation with a new room number.
     *
     * @param connection The database connection.
     * @param scanner    The scanner object for user input.
     */
    private static void updateReservation(Connection connection, Scanner scanner) {
        try {
            System.out.print("Enter guest name: ");
            String guestName = scanner.nextLine();
            System.out.print("Enter new room number: ");
            int roomNumber = scanner.nextInt();
            scanner.nextLine(); // consume newline left-over

            String sql = "UPDATE reservations SET room_number = ? WHERE guest_name = ?";
            try (PreparedStatement statement = connection.prepareStatement(sql)) {
                statement.setInt(1, roomNumber);
                statement.setString(2, guestName);
                int affectedRows = statement.executeUpdate();

                if (affectedRows > 0) {
                    System.out.println("Reservation updated successfully!");
                } else {
                    System.out.println("No reservation found for " + guestName);
                }
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    /**
     * Deletes a reservation based on the guest name.
     *
     * @param connection The database connection.
     * @param scanner    The scanner object for user input.
     */
    private static void deleteReservation(Connection connection, Scanner scanner) {
        try {
            System.out.print("Enter guest name: ");
            String guestName = scanner.nextLine();

            String sql = "DELETE FROM reservations WHERE guest_name = ?";
            try (PreparedStatement statement = connection.prepareStatement(sql)) {
                statement.setString(1, guestName);
                int affectedRows = statement.executeUpdate();

                if (affectedRows > 0) {
                    System.out.println("Reservation deleted successfully!");
                } else {
                    System.out.println("No reservation found for " + guestName);
                }
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    /**
     * Checks if a reservation exists in the database based on the reservation ID.
     *
     * @param connection    The database connection.
     * @param reservationId The reservation ID to check.
     * @return True if the reservation exists, false otherwise.
     */
    private static boolean reservationExists(Connection connection, int reservationId) {
        try {
            String sql = "SELECT reservation_id FROM reservations WHERE reservation_id = ?";
            try (PreparedStatement statement = connection.prepareStatement(sql)) {
                statement.setInt(1, reservationId);
                try (ResultSet resultSet = statement.executeQuery()) {
                    return resultSet.next(); // If there's a result, the reservation exists
                }
            }
        } catch (SQLException e) {
            e.printStackTrace();
            return false; // Handle database errors as needed
        }
        return false;
    }

    /**
     * Provides a graceful exit message and waits for a few seconds before terminating the program.
     *
     * @throws InterruptedException If the thread sleep is interrupted.
     */
    public static void exit() throws InterruptedException {
        System.out.print("Exiting System");
        int i = 5;
        while (i != 0) {
            System.out.print(".");
            Thread.sleep(1000);
            i--;
        }
        System.out.println();
        System.out.println("Thank you For Using Hotel Reservation System!!!");
    }
}
