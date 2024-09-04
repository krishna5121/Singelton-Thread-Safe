# Singelton-Thread-Safe

## Explanation

### Singleton Pattern
- `BankAccount.shared` provides the single instance of the `BankAccount` class.

### Serial Queue for Thread Safety
- The `serialQueue` ensures that all operations on the account balance are performed sequentially, avoiding race conditions. This is crucial in a banking context where operations like deposits and withdrawals need to be atomic.

### Thread-Safe Methods
- **`deposit(amount:)`**: Adds the specified amount to the balance. The operation is performed asynchronously on the serial queue to ensure thread safety.
- **`withdraw(amount:)`**: Deducts the specified amount from the balance if there are sufficient funds. It also uses the serial queue to prevent race conditions.
- **`getBalance()`**: Synchronously retrieves the current balance. Using `sync` ensures that the method waits for any ongoing operations to complete before returning the balance.

### Real-Life Scenario
- Multiple threads (representing different banking transactions) may try to deposit or withdraw money simultaneously. The serial queue ensures that each operation is completed before the next one starts, maintaining the integrity of the account balance.

```swift
import Foundation

class BankAccount {
    // Singleton instance
    static let shared = BankAccount()
    
    // Serial queue for thread safety
    private let serialQueue = DispatchQueue(label: "com.example.BankAccountSerialQueue")
    
    // Private property to hold the account balance
    private var balance: Double = 0.0
    
    // Private initializer to prevent external instantiation
    private init() {}
    
    // Thread-safe method to deposit money
    func deposit(amount: Double) {
        serialQueue.async {
            self.balance += amount
            print("Deposited \(amount). Current balance: \(self.balance)")
        }
    }
    
    // Thread-safe method to withdraw money
    func withdraw(amount: Double) {
        serialQueue.async {
            if self.balance >= amount {
                self.balance -= amount
                print("Withdrew \(amount). Current balance: \(self.balance)")
            } else {
                print("Insufficient funds. Tried to withdraw \(amount), but current balance is \(self.balance)")
            }
        }
    }
    
    // Thread-safe method to check the balance
    func getBalance() -> Double {
        return serialQueue.sync {
            return balance
        }
    }
}

// Example usage
let account = BankAccount.shared

// Perform multiple operations
account.deposit(amount: 1000.0)
account.withdraw(amount: 200.0)
account.withdraw(amount: 500.0)
account.deposit(amount: 300.0)
account.withdraw(amount: 700.0)

// Check final balance
let finalBalance = account.getBalance()
print("Final balance: \(finalBalance)")
