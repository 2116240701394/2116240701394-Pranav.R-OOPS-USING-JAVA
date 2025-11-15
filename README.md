import java.sql.*;
import java.util.*;

public class FlightBookingSystem {

    static final String URL = "jdbc:mysql://localhost:3306/flightdb";
    static final String USER = "root";          // your MySQL username
    static final String PASSWORD = "password";  // your MySQL password

    static Scanner sc = new Scanner(System.in);

    public static void main(String[] args) {
        int choice;
        do {
            System.out.println("\n==== FLIGHT BOOKING SYSTEM ====");
            System.out.println("1. Admin Login");
            System.out.println("2. User Login");
            System.out.println("3. Exit");
            System.out.print("Enter your choice: ");
            choice = sc.nextInt();
            sc.nextLine();

            switch (choice) {
                case 1:
                    adminLogin();
                    break;
                case 2:
                    userMenu();
                    break;
                case 3:
                    System.out.println("Exiting... Thank you!");
                    break;
                default:
                    System.out.println("Invalid choice!");
            }
        } while (choice != 3);
    }

    // ---------- ADMIN MODULE ----------
    static void adminLogin() {
        System.out.print("Enter Admin Username: ");
        String username = sc.nextLine();
        System.out.print("Enter Password: ");
        String password = sc.nextLine();

        if (username.equals("admin") && password.equals("1234")) {
            adminMenu();
        } else {
            System.out.println("Invalid credentials!");
        }
    }

    static void adminMenu() {
        int choice;
        do {
            System.out.println("\n--- ADMIN MENU ---");
            System.out.println("1. Add Flight");
            System.out.println("2. View Flights");
            System.out.println("3. Delete Flight");
            System.out.println("4. Logout");
            System.out.print("Enter choice: ");
            choice = sc.nextInt();
            sc.nextLine();

            switch (choice) {
                case 1:
                    addFlight();
                    break;
                case 2:
                    viewFlights();
                    break;
                case 3:
                    deleteFlight();
                    break;
                case 4:
                    System.out.println("Admin logged out.");
                    break;
                default:
                    System.out.println("Invalid option!");
            }
        } while (choice != 4);
    }

    static void addFlight() {
        try (Connection con = DriverManager.getConnection(URL, USER, PASSWORD)) {
            System.out.print("Enter Flight Number: ");
            String fn = sc.nextLine();
            System.out.print("Enter Source: ");
            String src = sc.nextLine();
            System.out.print("Enter Destination: ");
            String dest = sc.nextLine();
            System.out.print("Enter Seats: ");
            int seats = sc.nextInt();
            System.out.print("Enter Price: ");
            double price = sc.nextDouble();
            sc.nextLine();

            String query = "INSERT INTO flights VALUES (?, ?, ?, ?, ?)";
            PreparedStatement ps = con.prepareStatement(query);
            ps.setString(1, fn);
            ps.setString(2, src);
            ps.setString(3, dest);
            ps.setInt(4, seats);
            ps.setDouble(5, price);

            ps.executeUpdate();
            System.out.println("Flight added successfully!");

        } catch (SQLIntegrityConstraintViolationException e) {
            System.out.println("Flight already exists!");
        } catch (Exception e) {
            System.out.println("Error: " + e.getMessage());
        }
    }

    static void viewFlights() {
        try (Connection con = DriverManager.getConnection(URL, USER, PASSWORD);
             Statement st = con.createStatement()) {

            ResultSet rs = st.executeQuery("SELECT * FROM flights");

            System.out.println("\nAvailable Flights:");
            System.out.println("----------------------------------------------------------");
            System.out.printf("%-10s %-15s %-15s %-10s %-10s\n",
                    "FlightNo", "Source", "Destination", "Seats", "Price");
            System.out.println("----------------------------------------------------------");

            while (rs.next()) {
                System.out.printf("%-10s %-15s %-15s %-10d %-10.2f\n",
                        rs.getString("flight_no"),
                        rs.getString("source"),
                        rs.getString("destination"),
                        rs.getInt("seats"),
                        rs.getDouble("price"));
            }

        } catch (Exception e) {
            System.out.println("Error: " + e.getMessage());
        }
    }

    static void deleteFlight() {
        try (Connection con = DriverManager.getConnection(URL, USER, PASSWORD)) {
            System.out.print("Enter Flight Number to delete: ");
            String fn = sc.nextLine();

            String query = "DELETE FROM flights WHERE flight_no = ?";
            PreparedStatement ps = con.prepareStatement(query);
            ps.setString(1, fn);

            int rows = ps.executeUpdate();
            if (rows > 0)
                System.out.println("Flight deleted successfully!");
            else
                System.out.println("Flight not found!");

        } catch (Exception e) {
            System.out.println("Error: " + e.getMessage());
        }
    }

    // ---------- USER MODULE ----------
    static void userMenu() {
        int choice;
        do {
            System.out.println("\n--- USER MENU ---");
            System.out.println("1. View Flights");
            System.out.println("2. Book a Flight");
            System.out.println("3. Logout");
            System.out.print("Enter choice: ");
            choice = sc.nextInt();
            sc.nextLine();

            switch (choice) {
                case 1:
                    viewFlights();
                    break;
                case 2:
                    bookFlight();
                    break;
                case 3:
                    System.out.println("User logged out.");
                    break;
                default:
                    System.out.println("Invalid option!");
            }
        } while (choice != 3);
    }

    static void bookFlight() {
        try (Connection con = DriverManager.getConnection(URL, USER, PASSWORD)) {
            System.out.print("Enter Flight Number to book: ");
            String fn = sc.nextLine();

            String checkQuery = "SELECT * FROM flights WHERE flight_no = ?";
            PreparedStatement checkPs = con.prepareStatement(checkQuery);
            checkPs.setString(1, fn);
            ResultSet rs = checkPs.executeQuery();

            if (rs.next()) {
                int seats = rs.getInt("seats");
                double price = rs.getDouble("price");

                if (seats > 0) {
                    String updateQuery = "UPDATE flights SET seats = seats - 1 WHERE flight_no = ?";
                    PreparedStatement ps = con.prepareStatement(updateQuery);
                    ps.setString(1, fn);
                    ps.executeUpdate();

                    System.out.println("Booking confirmed for Flight " + fn);
                    System.out.println("Ticket Price: ₹" + price);
                } else {
                    System.out.println("No seats available!");
                }
            } else {
                System.out.println("Flight not found!");
            }

        } catch (Exception e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
