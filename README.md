package myPackage;
import java.util.Scanner;

public class Bankmanagementsystem {

    public static void main(String[] args) {
        Scanner scan = new Scanner(System.in);
        double balance = 1000.0; // Initial personal account balance
        double groupBalance = 0.0; // Initial group account balance
        boolean groupCreated = false; // Tracks whether a group account has been created
        String currentCurrency = "PHP"; // Default currency
        int choice; // User's choice in the main menu

        do {
            // Main menu display
            System.out.println("\nWelcome to Online Banking");
            System.out.println("Current Personal Balance: " + formatCurrency(balance, currentCurrency));
            if (groupCreated) {
                System.out.println("Current Group Balance: " + formatCurrency(groupBalance, currentCurrency));
            }
            System.out.println("1. Cash In");
            System.out.println("2. Cash Out");
            System.out.println("3. Pay or Split Bills");
            System.out.println("4. Switch Currency");
            System.out.println("5. Create Group Account");
            System.out.println("6. Exit");
            System.out.print("Enter your choice: ");
            choice = scan.nextInt();

            // Switch case to handle user selection
            switch (choice) {
                case 1:
                    balance = cashIn(scan, balance); // Add money to personal account
                    break;
                case 2:
                    balance = cashOut(scan, balance); // Withdraw money from personal account
                    break;
                case 3:
                    if (!groupCreated) {
                        balance = payBills(scan, balance); // Pay bills using personal balance
                    } else {
                        balance = payOrSplitBills(scan, balance, groupBalance); // Split bills if group account exists
                    }
                    break;
                case 4:
                    // Switch between currencies
                    String[] result = switchCurrency(scan, balance, groupBalance, currentCurrency);
                    currentCurrency = result[0];
                    balance = Double.parseDouble(result[1]);
                    groupBalance = Double.parseDouble(result[2]);
                    break;
                case 5:
                    if (!groupCreated) {
                        groupBalance = createGroupAccount(scan); // Create a group account
                        groupCreated = true;
                    } else {
                        System.out.println("Group account already exists.");
                    }
                    break;
                case 6:
                    // Exit the application
                    System.out.println("Thank you for using Online Banking. Goodbye!");
                    break;
                default:
                    // Handle invalid input
                    System.out.println("Invalid choice. Please try again.");
            }
        } while (choice != 6); // Repeat until user chooses to exit

        scan.close(); // Close scanner resource
    }

    // Method to cash in (add money to personal account)
    public static double cashIn(Scanner scan, double balance) {
        System.out.print("\nEnter the amount to cash in: ");
        double amount = scan.nextDouble();
        if (amount > 0) {
            balance += amount;
            System.out.printf("You have successfully cashed in %.2f. Your new balance is %.2f\n", amount, balance);
        } else {
            System.out.println("Invalid amount. Please enter a positive value.");
        }
        return balance;
    }

    // Method to cash out (withdraw money from personal account)
    public static double cashOut(Scanner scan, double balance) {
        System.out.print("\nEnter the amount to cash out: ");
        double amount = scan.nextDouble();
        if (amount > 0 && amount <= balance) {
            balance -= amount;
            System.out.printf("You have successfully cashed out %.2f. Your new balance is %.2f\n", amount, balance);
        } else if (amount > balance) {
            System.out.println("Insufficient balance.");
        } else {
            System.out.println("Invalid amount. Please enter a positive value.");
        }
        return balance;
    }

    // Method to pay bills using personal account balance
    public static double payBills(Scanner scan, double balance) {
        System.out.println("\n--- Pay Bills ---");
        System.out.println("1. Electricity");
        System.out.println("2. Water");
        System.out.println("3. Internet");
        System.out.println("4. Others");
        System.out.print("Choose the type of bill: ");
        int billType = scan.nextInt();
        scan.nextLine(); // Consume newline

        String billName = ""; // Name of the bill
        switch (billType) {
            case 1:
                billName = "Electricity";
                break;
            case 2:
                billName = "Water";
                break;
            case 3:
                billName = "Internet";
                break;
            case 4:
                System.out.print("Enter the name of the other bill: ");
                billName = scan.nextLine();
                break;
            default:
                System.out.println("Invalid choice. Returning to main menu.");
                return balance;
        }

        System.out.print("Enter the bill amount for " + billName + ": ");
        double billAmount = scan.nextDouble();

        if (billAmount > 0 && billAmount <= balance) {
            balance -= billAmount;
            System.out.printf("You have successfully paid %.2f for %s. Your remaining balance is %.2f\n", billAmount, billName, balance);
        } else if (billAmount > balance) {
            System.out.println("Insufficient balance to pay the bill.");
        } else {
            System.out.println("Invalid bill amount.");
        }

        return balance;
    }

    // Method to handle bill payment or splitting with a group
    public static double payOrSplitBills(Scanner scan, double balance, double groupBalance) {
        System.out.println("\n--- Pay or Split Bills ---");
        System.out.println("1. Pay Bill for Yourself");
        System.out.println("2. Split Bill with Group");
        System.out.print("Choose an option: ");
        int choice = scan.nextInt();

        if (choice == 1) {
            balance = payBills(scan, balance); // Pay bills with personal balance
        } else if (choice == 2) {
            if (groupBalance > 0) {
                balance = splitBill(scan, balance, groupBalance); // Split bills using group balance
            } else {
                System.out.println("Group account is empty or doesn't exist.");
            }
        } else {
            System.out.println("Invalid option.");
        }

        return balance;
    }

    // Method to split a bill among group members
    public static double splitBill(Scanner scan, double balance, double groupBalance) {
        System.out.print("Enter the total bill amount to split: ");
        double totalBill = scan.nextDouble();

        System.out.print("Enter the number of people to split with: ");
        int numPeople = scan.nextInt();

        if (numPeople > 0) {
            double splitAmount = totalBill / numPeople;
            if (groupBalance >= splitAmount) {
                groupBalance -= splitAmount;
                balance -= splitAmount;
                System.out.printf("Each person pays %.2f. Your group balance is now %.2f and your balance is %.2f\n", splitAmount, groupBalance, balance);
            } else {
                System.out.println("Insufficient group balance to split the bill.");
            }
        } else {
            System.out.println("Invalid number of people.");
        }

        return balance;
    }

    // Method to switch currency and recalculate balances
    public static String[] switchCurrency(Scanner scan, double balance, double groupBalance, String currentCurrency) {
        System.out.println("\n--- Switch Currency ---");
        System.out.println("1. Japanese Yen (JPY)");
        System.out.println("2. Philippine Peso (PHP)");
        System.out.println("3. US Dollar (USD)");
        System.out.print("Select a currency: ");
        int currencyChoice = scan.nextInt();

        double toPHP = 1.0; // Conversion rate to PHP
        double fromPHP = 1.0; // Conversion rate from PHP
        String newCurrency = currentCurrency;

        // Determine conversion rate from the current currency to PHP
        switch (currentCurrency) {
            case "PHP":
                toPHP = 1.0;
                break;
            case "JPY":
                toPHP = 1 / 2.68; 
                break;
            case "USD":
                toPHP = 1 / 0.017; 
                break;
            default:
                System.out.println("Unknown current currency. No changes made.");
                return new String[]{currentCurrency, String.valueOf(balance), String.valueOf(groupBalance)};
        }

        // Determine conversion rate from PHP to the new currency
        switch (currencyChoice) {
            case 1: 
                newCurrency = "JPY";
                fromPHP = 2.68;
                break;
            case 2: 
                newCurrency = "PHP";
                fromPHP = 1.0; 
                break;
            case 3: 
                newCurrency = "USD";
                fromPHP = 0.017; 
                break;
            default:
                System.out.println("Invalid choice. No changes made.");
                return new String[]{currentCurrency, String.valueOf(balance), String.valueOf(groupBalance)};
        }

        // Convert balances to the new currency
        double newBalance = balance * toPHP * fromPHP;
        double newGroupBalance = groupBalance * toPHP * fromPHP;

        System.out.println("Currency switched to " + newCurrency + ".");
        System.out.println("New personal balance: " + formatCurrency(newBalance, newCurrency));
        if (groupBalance > 0) {
            System.out.println("New group balance: " + formatCurrency(newGroupBalance, newCurrency));
        }

        return new String[]{newCurrency, String.valueOf(newBalance), String.valueOf(newGroupBalance)};
    }

    // Method to create a group account with an initial deposit
    public static double createGroupAccount(Scanner scan) {
        System.out.print("\nEnter the initial deposit for the group account: ");
        double groupBalance = scan.nextDouble();
        if (groupBalance > 0) {
            System.out.printf("Group account created successfully with an initial balance of %.2f.\n", groupBalance);
        } else {
            System.out.println("Invalid amount. Group account not created.");
            groupBalance = 0.0;
        }
        return groupBalance;
    }

    // Method to format currency based on the selected type
    public static String formatCurrency(double amount, String currency) {
        switch (currency) {
            case "JPY":
                return "¥" + String.format("%.2f", amount);
            case "PHP":
                return "₱" + String.format("%.2f", amount);
            case "USD":
                return "$" + String.format("%.2f", amount);
            default:
                return String.format("%.2f", amount) + " " + currency;
        }
    }
}
