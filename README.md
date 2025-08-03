import java.math.BigDecimal;
import java.math.RoundingMode;
import java.util.Scanner;

public class ATMInterface {

    /* ------------ MODEL: Bank Account ------------ */
    static class BankAccount {
        private final String accountNumber;
        private final String holderName;
        private final int pin;
        private BigDecimal balance;

        public BankAccount(String accountNumber, String holderName, int pin, BigDecimal openingBalance) {
            this.accountNumber = accountNumber;
            this.holderName = holderName;
            this.pin = pin;
            this.balance = openingBalance.setScale(2, RoundingMode.HALF_UP);
        }

        public boolean validatePin(int entered) {
            return this.pin == entered;
        }

        public BigDecimal getBalance() {
            return balance;
        }

        public boolean deposit(BigDecimal amount) {
            if (amount.signum() <= 0) return false;
            balance = balance.add(amount);
            return true;
        }

        public boolean withdraw(BigDecimal amount) {
            if (amount.signum() <= 0) return false;
            if (balance.compareTo(amount) < 0) return false;
            balance = balance.subtract(amount);
            return true;
        }

        @Override
        public String toString() {
            return holderName + " (A/C: " + accountNumber + ")";
        }
    }

    /* ------------ CONTROLLER / UI: ATM ------------ */
    static class ATM {
        private final Scanner sc = new Scanner(System.in);
        private final BankAccount account;

        public ATM(BankAccount account) {
            this.account = account;
        }

        public void start() {
            System.out.println("===== Welcome to the ATM =====");
            if (!authenticate()) {
                System.out.println("Too many failed attempts. Card retained. Bye!");
                return;
            }
            System.out.println("Hello, " + account + "!");
            mainMenu();
            System.out.println("Thank you for using the ATM. Goodbye!");
        }

        private boolean authenticate() {
            int attempts = 3;
            while (attempts-- > 0) {
                int pin = readInt("Enter PIN: ");
                if (account.validatePin(pin)) return true;
                System.out.println("Incorrect PIN. Attempts left: " + attempts);
            }
            return false;
        }

        private void mainMenu() {
            while (true) {
                System.out.println("\n===== Main Menu =====");
                System.out.println("1. Withdraw");
                System.out.println("2. Deposit");
                System.out.println("3. Check Balance");
                System.out.println("0. Exit");
                System.out.print("Choose: ");

                String choice = sc.nextLine().trim();
                switch (choice) {
                    case "1": doWithdraw(); break;
                    case "2": doDeposit(); break;
                    case "3": doCheckBalance(); break;
                    case "0": return;
                    default:  System.out.println("Invalid choice. Try again.");
                }
            }
        }

        private void doWithdraw() {
            BigDecimal amt = readAmount("Enter amount to withdraw: ");
            if (amt == null) return;

            if (account.withdraw(amt)) {
                System.out.println("Withdrawal of ₹" + amt + " successful.");
            } else {
                if (amt.signum() <= 0) {
                    System.out.println("Amount must be positive.");
                } else {
                    System.out.println("Insufficient balance. Transaction failed.");
                }
            }
            doCheckBalance();
        }

        private void doDeposit() {
            BigDecimal amt = readAmount("Enter amount to deposit: ");
            if (amt == null) return;

            if (account.deposit(amt)) {
                System.out.println("Deposit of ₹" + amt + " successful.");
            } else {
                System.out.println("Amount must be positive. Transaction failed.");
            }
            doCheckBalance();
        }

        private void doCheckBalance() {
            System.out.println("Current Balance: ₹" + account.getBalance());
        }

        // -------- helpers --------
        private int readInt(String prompt) {
            while (true) {
                System.out.print(prompt);
                String s = sc.nextLine().trim();
                try {
                    return Integer.parseInt(s);
                } catch (NumberFormatException e) {
                    System.out.println("Please enter a valid integer.");
                }
            }
        }

        private BigDecimal readAmount(String prompt) {
            System.out.print(prompt);
            String s = sc.nextLine().trim();
            try {
                BigDecimal amt = new BigDecimal(s).setScale(2, RoundingMode.HALF_UP);
                return amt;
            } catch (NumberFormatException e) {
                System.out.println("Invalid number format.");
                return null;
            }
        }
    }

    /* ------------ MAIN ------------ */
    public static void main(String[] args) {
        // Demo data: one account. You can wire this to a DB/file easily.
        BankAccount acc = new BankAccount(
                "1234567890",
                "John Doe",
                4321,
                new BigDecimal("10000.00")
        );
        new ATM(acc).start();
    }
}
