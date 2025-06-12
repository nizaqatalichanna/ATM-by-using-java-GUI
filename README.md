import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.sql.*;
import java.text.SimpleDateFormat;
import java.util.Date;

public class ATM {
    private JFrame frame;
    private CardLayout cardLayout = new CardLayout();
    private JPanel mainPanel = new JPanel(cardLayout);
    private JTextField accountField = new JTextField(15);
    private JPasswordField pinField = new JPasswordField(15);
    private Account currentAccount;
    private Connection conn;

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new ATM().createAndShowGUI());
    }

    public ATM() {
        try {
            conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/atm", "root", "889087");
        } catch (SQLException e) {
            e.printStackTrace();
            JOptionPane.showMessageDialog(null, "Database connection failed.", "Error", JOptionPane.ERROR_MESSAGE);
            System.exit(1);
        }
    }

    private void createAndShowGUI() {
        frame = new JFrame("ATM System");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setSize(500, 400);

        mainPanel.add(createLoginPanel(), "Login");
        mainPanel.add(createMenuPanel(), "Menu");
        mainPanel.add(createRegisterPanel(), "Register");

        frame.add(mainPanel);
        frame.setVisible(true);
    }

    private JPanel createLoginPanel() {
        JPanel panel = new JPanel(new BorderLayout());
        panel.setBackground(Color.BLACK);
        panel.setBorder(BorderFactory.createEmptyBorder(50, 50, 50, 50));

        JLabel title = new JLabel("ATM", JLabel.CENTER);
        title.setFont(new Font("Forte", Font.BOLD, 60));
        title.setForeground(Color.GREEN);
        panel.add(title, BorderLayout.NORTH);

        JPanel centerPanel = new JPanel();
        centerPanel.setLayout(new BoxLayout(centerPanel, BoxLayout.Y_AXIS));
        centerPanel.setBackground(Color.BLUE);

        JPanel formPanel = new JPanel();
        formPanel.setLayout(new BoxLayout(formPanel, BoxLayout.Y_AXIS));
        formPanel.setBackground(Color.GREEN);
        formPanel.setBorder(BorderFactory.createEmptyBorder(20, 40, 20, 40));
        formPanel.setMaximumSize(new Dimension(400, 350));

        JPanel accPanel = new JPanel();
        accPanel.setLayout(new BoxLayout(accPanel, BoxLayout.Y_AXIS));
        accPanel.setBackground(Color.BLUE);

        JLabel accLabel = new JLabel("Account Number");
        accLabel.setFont(new Font("Forte", Font.PLAIN, 30));
        accLabel.setAlignmentX(Component.CENTER_ALIGNMENT);
        accPanel.add(accLabel);
        accPanel.add(Box.createVerticalStrut(8));

        accountField.setMaximumSize(new Dimension(300, 30));
        accountField.setFont(new Font("Forte", Font.PLAIN, 31));
        accPanel.add(accountField);
        formPanel.add(accPanel);
        formPanel.add(Box.createVerticalStrut(15));

        JPanel pinPanel = new JPanel();
        pinPanel.setLayout(new BoxLayout(pinPanel, BoxLayout.Y_AXIS));
        pinPanel.setBackground(Color.BLUE);

        JLabel pinLabel = new JLabel("PIN:");
        pinLabel.setFont(new Font("Forte", Font.PLAIN, 25));
        pinLabel.setAlignmentX(Component.CENTER_ALIGNMENT);
        pinPanel.add(pinLabel);
        pinPanel.add(Box.createVerticalStrut(2));

        pinField.setMaximumSize(new Dimension(300, 30));
        pinField.setFont(new Font("Forte", Font.BOLD, 30));
        pinPanel.add(pinField);
        formPanel.add(pinPanel);
        formPanel.add(Box.createVerticalStrut(20));

        JPanel buttonPanel = new JPanel();
        buttonPanel.setBackground(Color.GREEN);
        buttonPanel.setLayout(new BoxLayout(buttonPanel, BoxLayout.X_AXIS));

        JButton loginBtn = new JButton("Login");
        loginBtn.setAlignmentX(Component.CENTER_ALIGNMENT);
        loginBtn.setBackground(Color.BLACK);
        loginBtn.setForeground(Color.BLUE);
        loginBtn.setFont(new Font("Forte", Font.BOLD, 23));
        loginBtn.setMaximumSize(new Dimension(150, 40));
        loginBtn.addActionListener(e -> login());
        buttonPanel.add(loginBtn);
        buttonPanel.add(Box.createHorizontalStrut(20));

        JButton registerBtn = new JButton("Register");
        registerBtn.setAlignmentX(Component.CENTER_ALIGNMENT);
        registerBtn.setBackground(Color.BLACK);
        registerBtn.setForeground(Color.BLUE);
        registerBtn.setFont(new Font("Forte", Font.BOLD, 23));
        registerBtn.setMaximumSize(new Dimension(150, 40));
        registerBtn.addActionListener(e -> cardLayout.show(mainPanel, "Register"));
        buttonPanel.add(registerBtn);

        formPanel.add(buttonPanel);
        centerPanel.add(formPanel);
        panel.add(centerPanel, BorderLayout.CENTER);
        return panel;
    }

    private JPanel createRegisterPanel() {
        JPanel panel = new JPanel(new BorderLayout());
        panel.setBackground(Color.BLACK);
        panel.setBorder(BorderFactory.createEmptyBorder(50, 50, 50, 50));

        JLabel title = new JLabel("Register", JLabel.CENTER);
        title.setFont(new Font("Forte", Font.BOLD, 60));
        title.setForeground(Color.GREEN);
        panel.add(title, BorderLayout.NORTH);

        JPanel formPanel = new JPanel();
        formPanel.setLayout(new BoxLayout(formPanel, BoxLayout.Y_AXIS));
        formPanel.setBackground(Color.GREEN);
        formPanel.setBorder(BorderFactory.createEmptyBorder(20, 40, 20, 40));
        formPanel.setMaximumSize(new Dimension(400, 350));

        JTextField accNumField = new JTextField(15);
        JPasswordField pinField = new JPasswordField(15);
        JTextField nameField = new JTextField(15);
        JTextField emailField = new JTextField(15);
        JTextField balanceField = new JTextField(15);

        formPanel.add(createLabeledField("Account Number:", accNumField));
        formPanel.add(Box.createVerticalStrut(10));
        formPanel.add(createLabeledField("PIN:", pinField));
        formPanel.add(Box.createVerticalStrut(10));
        formPanel.add(createLabeledField("Name:", nameField));
        formPanel.add(Box.createVerticalStrut(10));
        formPanel.add(createLabeledField("Email:", emailField));
        formPanel.add(Box.createVerticalStrut(10));
        formPanel.add(createLabeledField("Initial Balance:", balanceField));
        formPanel.add(Box.createVerticalStrut(20));

        JButton registerBtn = new JButton("Submit");
        registerBtn.setAlignmentX(Component.CENTER_ALIGNMENT);
        registerBtn.setBackground(Color.BLACK);
        registerBtn.setForeground(Color.BLUE);
        registerBtn.setFont(new Font("Forte", Font.BOLD, 23));
        registerBtn.setMaximumSize(new Dimension(150, 40));
        registerBtn.addActionListener(e -> {
            try {
                int accNum = Integer.parseInt(accNumField.getText());
                String pin = new String(pinField.getPassword());
                String name = nameField.getText();
                String email = emailField.getText();
                int balance = Integer.parseInt(balanceField.getText());

                if (pin.length() != 4 || !pin.matches("\\d+")) {
                    showMessage("PIN must be exactly 4 digits.", "Error");
                    return;
                }

                if (name.isEmpty() || email.isEmpty()) {
                    showMessage("All fields are required.", "Error");
                    return;
                }

                if (balance < 0) {
                    showMessage("Initial balance cannot be negative.", "Error");
                    return;
                }

                String query = "INSERT INTO Accounts (account_number, pin, name, email, balance) VALUES (?, ?, ?, ?, ?)";
                PreparedStatement stmt = conn.prepareStatement(query);
                stmt.setInt(1, accNum);
                stmt.setString(2, pin);
                stmt.setString(3, name);
                stmt.setString(4, email);
                stmt.setInt(5, balance);
                int rowsAffected = stmt.executeUpdate();

                if (rowsAffected > 0) {
                    showMessage("Registration successful! Please login.", "Success");
                    cardLayout.show(mainPanel, "Login");
                    accNumField.setText("");
                    pinField.setText("");
                    nameField.setText("");
                    emailField.setText("");
                    balanceField.setText("");
                } else {
                    showMessage("Registration failed. Please try again.", "Error");
                }
            } catch (SQLException ex) {
                if (ex.getErrorCode() == 1062) { // Duplicate entry error
                    showMessage("Account number already exists.", "Error");
                } else {
                    showMessage("Database error: " + ex.getMessage(), "Error");
                }
            } catch (NumberFormatException ex) {
                showMessage("Account number and balance must be numeric.", "Error");
            }
        });

        formPanel.add(registerBtn);
        panel.add(formPanel, BorderLayout.CENTER);
        return panel;
    }

    private JPanel createLabeledField(String labelText, JTextField field) {
        JPanel panel = new JPanel();
        panel.setLayout(new BoxLayout(panel, BoxLayout.Y_AXIS));
        panel.setBackground(Color.BLUE);

        JLabel label = new JLabel(labelText);
        label.setFont(new Font("Forte", Font.PLAIN, 25));
        label.setAlignmentX(Component.CENTER_ALIGNMENT);
        panel.add(label);
        panel.add(Box.createVerticalStrut(2));

        field.setMaximumSize(new Dimension(300, 30));
        field.setFont(new Font("Forte", Font.PLAIN, 30));
        panel.add(field);

        return panel;
    }

    private JPanel createMenuPanel() {
        JPanel panel = new JPanel(new BorderLayout());
        JPanel buttons = new JPanel(new GridLayout(8, 1, 10, 10));
        buttons.setBorder(BorderFactory.createEmptyBorder(10, 50, 10, 50));

        buttons.add(createButton("Check Balance", new Color(33, 150, 243), e -> showBalance()));
        buttons.add(createButton("Deposit", new Color(80, 15, 90), e -> getAmount("Deposit")));
        buttons.add(createButton("Withdraw", new Color(29, 1, 33), e -> getAmount("Withdraw")));
        buttons.add(createButton("Transfer", new Color(171, 71, 188), e -> getTransfer()));
        buttons.add(createButton("Transaction History", new Color(255, 87, 34), e -> showTransactionHistory()));
        buttons.add(createButton("Update Profile", new Color(76, 175, 80), e -> updateProfile()));
        buttons.add(createButton("Delete Account", new Color(244, 67, 54), e -> deleteAccount()));
        buttons.add(createButton("Logout", new Color(120, 144, 156), e -> logout()));

        panel.add(buttons, BorderLayout.CENTER);
        return panel;
    }

    private JButton createButton(String text, Color color, ActionListener action) {
        JButton button = new JButton(text);
        button.setFont(new Font("Forte", Font.BOLD, 30));
        button.setForeground(Color.GREEN);
        button.setBackground(color);

        button.addActionListener(e -> {
            try {
                action.actionPerformed(e);
            } catch (Exception ex) {
                showMessage("An error occurred: " + ex.getMessage(), "Error");
            }
        });

        button.addMouseListener(new MouseAdapter() {
            public void mouseEntered(MouseEvent e) { button.setBackground(Color.BLACK); }
            public void mouseExited(MouseEvent e) { button.setBackground(color); }
        });
        return button;
    }

    private void login() {
        try {
            int accNum = Integer.parseInt(accountField.getText());
            String pin = new String(pinField.getPassword());

            String query = "SELECT * FROM Accounts WHERE account_number = ? AND pin = ?";
            PreparedStatement stmt = conn.prepareStatement(query);
            stmt.setInt(1, accNum);
            stmt.setString(2, pin);
            ResultSet rs = stmt.executeQuery();

            if (rs.next()) {
                currentAccount = new Account(
                        rs.getInt("account_number"),
                        rs.getString("pin"),
                        rs.getInt("balance"),
                        rs.getString("name"),
                        rs.getString("email")
                );
                cardLayout.show(mainPanel, "Menu");
                accountField.setText("");
                pinField.setText("");
            } else {
                showMessage("Invalid account number or PIN.", "Error");
            }
        } catch (SQLException ex) {
            showMessage("Database error: " + ex.getMessage(), "Error");
        } catch (NumberFormatException ex) {
            showMessage("Account number must be numeric.", "Error");
        }
    }

    private void logout() {
        currentAccount = null;
        cardLayout.show(mainPanel, "Login");
    }

    private void showBalance() {
        try {
            String query = "SELECT balance FROM Accounts WHERE account_number = ?";
            PreparedStatement stmt = conn.prepareStatement(query);
            stmt.setInt(1, currentAccount.getAccountNumber());
            ResultSet rs = stmt.executeQuery();
            if (rs.next()) {
                currentAccount = new Account(
                        currentAccount.getAccountNumber(),
                        currentAccount.getPin(),
                        rs.getInt("balance"),
                        currentAccount.getName(),
                        currentAccount.getEmail()
                );
            }
            JOptionPane.showMessageDialog(frame,
                    String.format("<html><center><h2>Account Balance</h2><h1>$%,d</h1></center></html>",
                            currentAccount.getBalance()), "Balance", JOptionPane.PLAIN_MESSAGE);
        } catch (SQLException ex) {
            showMessage("Error fetching balance: " + ex.getMessage(), "Error");
        }
    }

    private void getAmount(String action) {
        String amount = JOptionPane.showInputDialog(frame, "Enter amount to " + action.toLowerCase() + ":", action, JOptionPane.PLAIN_MESSAGE);
        if (amount != null) {
            try {
                int amt = Integer.parseInt(amount);
                boolean success = action.equals("Deposit") ? currentAccount.deposit(amt) : currentAccount.withdraw(amt);
                if (success) {
                    updateBalanceInDB();
                    logTransaction(action, amt, currentAccount.getAccountNumber());
                    showReceipt(action, amt);
                } else {
                    showMessage("Invalid amount or insufficient funds.", "Error");
                }
            } catch (NumberFormatException ex) {
                showMessage("Please enter a valid number.", "Error");
            } catch (SQLException ex) {
                showMessage("Database error: " + ex.getMessage(), "Error");
            }
        }
    }

    private void getTransfer() {
        JPanel panel = new JPanel(new GridLayout(2, 2, 5, 5));
        JTextField recipient = new JTextField();
        JTextField amount = new JTextField();
        panel.add(new JLabel("Recipient Account:"));
        panel.add(recipient);
        panel.add(new JLabel("Amount:"));
        panel.add(amount);

        if (JOptionPane.showConfirmDialog(frame, panel, "Transfer", JOptionPane.OK_CANCEL_OPTION) == JOptionPane.OK_OPTION) {
            try {
                int acc = Integer.parseInt(recipient.getText());
                int amt = Integer.parseInt(amount.getText());

                String query = "SELECT * FROM Accounts WHERE account_number = ?";
                PreparedStatement stmt = conn.prepareStatement(query);
                stmt.setInt(1, acc);
                ResultSet rs = stmt.executeQuery();

                if (!rs.next()) {
                    showMessage("Recipient account not found.", "Error");
                } else if (acc == currentAccount.getAccountNumber()) {
                    showMessage("Cannot transfer to yourself.", "Error");
                } else if (amt <= 0) {
                    showMessage("Amount must be positive.", "Error");
                } else {
                    Account recipientAcc = new Account(
                            rs.getInt("account_number"),
                            rs.getString("pin"),
                            rs.getInt("balance"),
                            rs.getString("name"),
                            rs.getString("email")
                    );
                    if (currentAccount.transfer(amt, recipientAcc)) {
                        updateBalanceInDB();
                        updateBalanceInDB(recipientAcc);
                        logTransaction("Transfer", amt, acc);
                        showReceipt("Transfer", amt);
                    } else {
                        showMessage("Transfer failed. Check your balance.", "Error");
                    }
                }
            } catch (SQLException ex) {
                showMessage("Database error: " + ex.getMessage(), "Error");
            } catch (NumberFormatException ex) {
                showMessage("Please enter valid numbers.", "Error");
            }
        }
    }

    private void showTransactionHistory() {
        try {
            String query = "SELECT * FROM Transactions WHERE account_number = ? ORDER BY transaction_date DESC";
            PreparedStatement stmt = conn.prepareStatement(query);
            stmt.setInt(1, currentAccount.getAccountNumber());
            ResultSet rs = stmt.executeQuery();

            JTextArea historyArea = new JTextArea(15, 40);
            historyArea.setEditable(false);
            historyArea.setFont(new Font("Monospaced", Font.PLAIN, 14));
            StringBuilder history = new StringBuilder("Transaction History\n--------------------------------\n");
            while (rs.next()) {
                history.append(String.format("Type: %s\nAmount: $%,d\nTo/From: %d\nDate: %s\n--------------------------------\n",
                        rs.getString("transaction_type"),
                        rs.getInt("amount"),
                        rs.getInt("recipient_account"),
                        rs.getString("transaction_date")));
            }
            historyArea.setText(history.toString());

            JOptionPane.showMessageDialog(frame, new JScrollPane(historyArea), "Transaction History", JOptionPane.PLAIN_MESSAGE);
        } catch (SQLException ex) {
            showMessage("Error fetching transaction history: " + ex.getMessage(), "Error");
        }
    }

    private void updateProfile() {
        JPanel panel = new JPanel(new GridLayout(2, 2, 5, 5));
        JTextField nameField = new JTextField(currentAccount.getName());
        JTextField emailField = new JTextField(currentAccount.getEmail());
        panel.add(new JLabel("Name:"));
        panel.add(nameField);
        panel.add(new JLabel("Email:"));
        panel.add(emailField);

        if (JOptionPane.showConfirmDialog(frame, panel, "Update Profile", JOptionPane.OK_CANCEL_OPTION) == JOptionPane.OK_OPTION) {
            try {
                String name = nameField.getText();
                String email = emailField.getText();
                if (name.isEmpty() || email.isEmpty()) {
                    showMessage("All fields are required.", "Error");
                    return;
                }

                String query = "UPDATE Accounts SET name = ?, email = ? WHERE account_number = ?";
                PreparedStatement stmt = conn.prepareStatement(query);
                stmt.setString(1, name);
                stmt.setString(2, email);
                stmt.setInt(3, currentAccount.getAccountNumber());
                int rowsAffected = stmt.executeUpdate();

                if (rowsAffected > 0) {
                    currentAccount.setName(name);
                    currentAccount.setEmail(email);
                    showMessage("Profile updated successfully.", "Success");
                } else {
                    showMessage("Failed to update profile.", "Error");
                }
            } catch (SQLException ex) {
                showMessage("Error updating profile: " + ex.getMessage(), "Error");
            }
        }
    }

    private void deleteAccount() {
        int confirm = JOptionPane.showConfirmDialog(frame, "Are you sure you want to delete your account?", "Confirm Deletion", JOptionPane.YES_NO_OPTION);
        if (confirm == JOptionPane.YES_OPTION) {
            try {
                String query = "DELETE FROM Transactions WHERE account_number = ?";
                PreparedStatement stmt = conn.prepareStatement(query);
                stmt.setInt(1, currentAccount.getAccountNumber());
                stmt.executeUpdate();

                query = "DELETE FROM Accounts WHERE account_number = ?";
                stmt = conn.prepareStatement(query);
                stmt.setInt(1, currentAccount.getAccountNumber());
                int rowsAffected = stmt.executeUpdate();

                if (rowsAffected > 0) {
                    showMessage("Account deleted successfully.", "Success");
                    logout();
                } else {
                    showMessage("Failed to delete account.", "Error");
                }
            } catch (SQLException ex) {
                showMessage("Error deleting account: " + ex.getMessage(), "Error");
            }
        }
    }

    private void updateBalanceInDB() throws SQLException {
        updateBalanceInDB(currentAccount);
    }

    private void updateBalanceInDB(Account account) throws SQLException {
        String query = "UPDATE Accounts SET balance = ? WHERE account_number = ?";
        PreparedStatement stmt = conn.prepareStatement(query);
        stmt.setInt(1, account.getBalance());
        stmt.setInt(2, account.getAccountNumber());
        int rowsAffected = stmt.executeUpdate();
        if (rowsAffected == 0) {
            throw new SQLException("Failed to update balance for account " + account.getAccountNumber());
        }
    }

    private void logTransaction(String type, int amount, int recipient) throws SQLException {
        String query = "INSERT INTO Transactions (account_number, transaction_type, amount, recipient_account, transaction_date) VALUES (?, ?, ?, ?, ?)";
        PreparedStatement stmt = conn.prepareStatement(query);
        stmt.setInt(1, currentAccount.getAccountNumber());
        stmt.setString(2, type);
        stmt.setInt(3, amount);
        stmt.setInt(4, recipient);
        stmt.setString(5, new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date()));
        int rowsAffected = stmt.executeUpdate();
        if (rowsAffected == 0) {
            throw new SQLException("Failed to log transaction.");
        }
    }

    private void showMessage(String msg, String title) {
        JOptionPane.showMessageDialog(frame, msg, title, title.equals("Error") ? JOptionPane.ERROR_MESSAGE : JOptionPane.INFORMATION_MESSAGE);
    }

    private void showReceipt(String type, int amount) {
        JTextArea receiptArea = new JTextArea(10, 30);
        receiptArea.setEditable(false);
        receiptArea.setFont(new Font("Monospaced", Font.PLAIN, 14));

        String receipt = String.format(
                "BANK OF DADU\n" +
                        "--------------------------------\n" +
                        "Transaction: %s\n" +
                        "Account: ****%04d\n" +
                        "Amount: $%,d\n" +
                        "New Balance: $%,d\n" +
                        "Date/Time: %s\n" +
                        "--------------------------------\n" +
                        "Thank you for banking with us!",
                type, currentAccount.getAccountNumber() % 10000, amount,
                currentAccount.getBalance(), new SimpleDateFormat("dd/MM/yyyy HH:mm:ss").format(new Date())
        );
        receiptArea.setText(receipt);

        JOptionPane.showMessageDialog(frame, new JScrollPane(receiptArea), "Receipt", JOptionPane.PLAIN_MESSAGE);
    }

    static class Account {
        private int accountNumber, balance;
        private String pin, name, email;

        public Account(int acc, String pin, int bal, String name, String email) {
            this.accountNumber = acc;
            this.pin = pin;
            this.balance = bal;
            this.name = name;
            this.email = email;
        }

        public boolean validatePin(String pin) { return this.pin.equals(pin); }
        public int getBalance() { return balance; }
        public int getAccountNumber() { return accountNumber; }
        public String getName() { return name; }
        public String getEmail() { return email; }
        public String getPin() { return pin; }
        public void setName(String name) { this.name = name; }
        public void setEmail(String email) { this.email = email; }

        public boolean withdraw(int amount) {
            if (amount >= 500 && amount <= 25000 && amount % 500 == 0 && amount <= balance) {
                balance -= amount;
                return true;
            }
            return false;
        }

        public boolean deposit(int amount) {
            if (amount >= 500 && amount <= 25000 && amount % 500 == 0) {
                balance += amount;
                return true;
            }
            return false;
        }

        public boolean transfer(int amount, Account recipient) {
            if (withdraw(amount)) {
                recipient.deposit(amount);
                return true;
            }
            return false;
        }
    }
}

