# Medicine-purchase-record
I created 3 packages
MedicinePurchaseRecord/
├── src/
│   ├── model/
│   │   ├── Medicine.java
│   │   ├── Customer.java
│   │   └── Purchase.java
│   ├── service/
│   │   └── PharmacyService.java
│   ├── util/
│   │   └── InputValidator.java
│   └── Main.java

In file called medicine

package model;

public class Medicine {
    private String id;
    private String name;
    private double price;
    private int stock;

    public Medicine(String id, String name, double price, int stock) {
        this.id = id;
        this.name = name;
        this.price = price;
        this.stock = stock;
    }

    public String getId() { return id; }
    public String getName() { return name; }
    public double getPrice() { return price; }
    public int getStock() { return stock; }

    public void setStock(int stock) { this.stock = stock; }
}
Customer

package model;

public class Customer {
    private String name;
    private String contact;

    public Customer(String name, String contact) {
        this.name = name;
        this.contact = contact;
    }

    public String getName() { return name; }
    public String getContact() { return contact; }
}
purchase

package model;

public class Purchase {
    private Customer customer;
    private Medicine medicine;
    private int quantity;

    public Purchase(Customer customer, Medicine medicine, int quantity) {
        this.customer = customer;
        this.medicine = medicine;
        this.quantity = quantity;
    }

    public Customer getCustomer() { return customer; }
    public Medicine getMedicine() { return medicine; }
    public int getQuantity() { return quantity; }

    public double getTotalPrice() {
        return medicine.getPrice() * quantity;
    }
}

service/InsufficientStockException.java

package service;

public class InsufficientStockException extends Exception {
    public InsufficientStockException(String message) {
        super(message);
    }
}
service/PharmacyService.java
package service;

import model.Customer;
import model.Medicine;
import model.Purchase;

import java.util.ArrayList;
import java.util.List;

public class PharmacyService {
    private final List<Medicine> inventory = new ArrayList<>();
    private final List<Purchase> purchases = new ArrayList<>();

    public void addMedicine(Medicine m) {
        inventory.add(m);
    }

    public Medicine findMedicineById(String id) {
        for (Medicine m : inventory) {
            if (m.getId().equalsIgnoreCase(id)) return m;
        }
        return null;
    }

    public void recordPurchase(Customer customer, String medicineId, int quantity)
            throws InsufficientStockException {
        Medicine m = findMedicineById(medicineId);
        if (m == null) throw new InsufficientStockException("Medicine not found.");
        if (m.getStock() < quantity)
            throw new InsufficientStockException("Not enough stock available.");
        m.setStock(m.getStock() - quantity);
        purchases.add(new Purchase(customer, m, quantity));
    }

    public void showAllPurchases() {
        System.out.println("\n--- Purchase History ---");
        for (Purchase p : purchases) {
            System.out.println(p);
        }
    }

    public void showInventory() {
        System.out.println("\n--- Inventory ---");
        for (Medicine m : inventory) {
            System.out.println(m);
        }
    }
}
util/InputValidator.java
package util;

public class InputValidator {
    public static boolean isValidPhone(String phone) {
        return phone != null && phone.matches("\\d{10}");
    }

    public static boolean isValidQuantity(int qty) {
        return qty > 0;
    }
}

Main.java
import model.Customer;
import model.Medicine;
import service.PharmacyService;
import service.InsufficientStockException;
import util.InputValidator;

import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        PharmacyService service = new PharmacyService();
        Scanner scanner = new Scanner(System.in);

        // Sample data
        service.addMedicine(new Medicine("M001", "Paracetamol", 1.5, 100));
        service.addMedicine(new Medicine("M002", "Amoxicillin", 2.0, 50));
        service.addMedicine(new Medicine("M003", "Ibuprofen", 2.5, 30));

        System.out.println("Welcome to Medicine Purchase Record System");

        service.showInventory();

        try {
            System.out.print("Enter your name: ");
            String name = scanner.nextLine();

            System.out.print("Enter contact (10 digits): ");
            String contact = scanner.nextLine();
            if (!InputValidator.isValidPhone(contact)) {
                throw new IllegalArgumentException("Invalid phone number!");
            }

            System.out.print("Enter Medicine ID to purchase: ");
            String medId = scanner.nextLine();

            System.out.print("Enter quantity: ");
            int qty = scanner.nextInt();
            if (!InputValidator.isValidQuantity(qty)) {
                throw new IllegalArgumentException("Invalid quantity!");
            }

            Customer customer = new Customer(name, contact);
            service.recordPurchase(customer, medId, qty);
            System.out.println("Purchase recorded successfully.");

        } catch (InsufficientStockException e) {
            System.out.println("Error: " + e.getMessage());
        } catch (IllegalArgumentException e) {
            System.out.println("Input Error: " + e.getMessage());
        } catch (Exception e) {
            System.out.println("Unexpected error occurred.");
        }

        service.showAllPurchases();
        scanner.close();
    }
}



