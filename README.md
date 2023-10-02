import csv
import datetime
import os  # Import the 'os' module for file path handling

# Define the file paths
users_file = 'users.txt'
ppe_file = 'ppe.txt'
suppliers_file = 'suppliers.txt'
transactions_file = 'transactions.txt'  # Add transactions file path

def admin_add_user():
    user_id = input("Enter new User ID: ")
    user_name = input("Enter user's name: ")
    password = input("Enter new password: ")
    user_type = input("Enter user type (admin or staff): ")

    # Check if the user ID is unique
    with open('users.txt', 'r') as file:
        for line in file:
            parts = line.strip().split(',')
            if parts[0] == user_id:
                print("User ID already exists. Please choose a different one.\n")
                return

    # Add the new user to users.txt
    with open('users.txt', 'a') as file:
        file.write(f"{user_id},{user_name},{password},{user_type}\n")

    print("New user added!\n")

# Function to delete an existing user by the admin (Admin Only)
def admin_delete_user():
    user_id = input("Enter the User ID to delete: ")

    # Check if the user exists
    with open('users.txt', 'r') as file:
        lines = file.readlines()

    found = False
    with open('users.txt', 'w') as file:
        for line in lines:
            parts = line.strip().split(',')
            if parts[0] == user_id:
                found = True
            else:
                file.write(line)

    if found:
        print("User deleted!\n")
    else:
        print("User not found.\n")

# Function to edit information of an existing user by the admin (Admin Only)
def admin_edit_user():
    user_id = input("Enter the User ID to edit: ")

    # Check if the user exists
    with open('users.txt', 'r') as file:
        lines = file.readlines()

    found = False
    with open('users.txt', 'w') as file:
        for line in lines:
            parts = line.strip().split(',')
            if parts[0] == user_id:
                user_name = input("Enter new user's name: ")
                password = input("Enter new password: ")
                user_type = input("Enter user type (admin or staff): ")
                file.write(f"{user_id},{user_name},{password},{user_type}\n")
                found = True
            else:
                file.write(line)

    if found:
        print("User information updated!\n")
    else:
        print("User not found.\n")

# Function to create the initial users.txt file with an admin and a staff user
def create_initial_users_file():
    with open('users.txt', 'w') as file:
        file.write("admin123,Admin User,adminpassword,admin,Administrator\n")
        file.write("staff123,Staff User,staffpassword,staff,Staff\n")



# Function to create the initial ppe.txt file with 6 PPE items
def create_initial_ppe_file():
    with open(ppe_file, 'w') as file:
        file.write("HC,Supplier1,100\n")
        file.write("FS,Supplier1,100\n")
        file.write("MS,Supplier2,100\n")
        file.write("GL,Supplier2,100\n")
        file.write("GW,Supplier4,100\n")
        file.write("SC,Supplier5,100\n")

# Function to create the initial suppliers.txt file with sample supplier data
def create_initial_suppliers_file():
    with open(suppliers_file, 'w') as file:
        file.write("Supplier1,HC,123-456-7890\n")
        file.write("Supplier1,FS,123-456-7890\n")
        file.write("Supplier2,MS,555-555-5555\n")
        file.write("Supplier2,GL,555-555-5555\n")
        file.write("Supplier4,GW,888-888-8888\n")
        file.write("Supplier5,SC,999-999-9999\n")

# Function to create transactions.txt and record transactions
def create_transactions_file():
    with open(transactions_file, 'w') as file:
        pass  # Create an empty transactions file

# Function to display current stock for all items
def display_current_stock():
    with open(ppe_file, 'r') as file:
        print("Current Stock:")
        print("{:<5} {:<15} {:<10}".format("Item", "Supplier", "Quantity"))
        for line in file:
            item_code, supplier_code, quantity = line.strip().split(',')
            print("{:<5} {:<15} {:<10}".format(item_code, supplier_code, quantity))

# Function to search and print details of items distributed for a particular item (Admin and Staff)
def search_item_distribution():
    item_code = input("Enter Item Code to search for distribution details: ")
    with open(transactions_file, 'r') as file:
        print(f"Distribution Details for Item Code: {item_code}")
        print("{:<20} {:<15} {:<10}".format("Date-Time", "Supplier", "Quantity Change"))
        for line in file:
            timestamp, code, supplier_code, quantity_change = line.strip().split(',')
            if code == item_code:
                print("{:<20} {:<15} {:<10}".format(timestamp, supplier_code, quantity_change))

# Function to record a transaction in transactions.txt
def record_transaction(item_code, supplier_code, quantity_change):
    timestamp = datetime.datetime.now()
    transaction_data = f"{timestamp},{item_code},{supplier_code},{quantity_change}\n"
    with open(transactions_file, 'a') as file:
        file.write(transaction_data)

# Function to add/update supplier information (Admin Only)
def admin_update_supplier():
    supplier_code = input("Enter Supplier Code: ")
    item_code = input("Enter Item Code supplied: ")
    phone_number = input("Enter Phone Number: ")

    # Check if the supplier already exists in suppliers.txt
    with open('suppliers.txt', 'r') as file:
        lines = file.readlines()

    updated = False
    with open('suppliers.txt', 'w') as file:
        for line in lines:
            parts = line.strip().split(',')
            if parts[0] == supplier_code and parts[1] == item_code:
                # Update the phone number for the existing supplier-item combination
                file.write(f"{supplier_code},{item_code},{phone_number}\n")
                updated = True
            else:
                file.write(line)

    if not updated:
        # Add a new supplier-item combination
        with open('suppliers.txt', 'a') as file:
            file.write(f"{supplier_code},{item_code},{phone_number}\n")

    print("Supplier information updated!\n")

# Function to update PPE item quantities (Admin and Staff)
def update_ppe_quantity():
    item_code = input("Enter Item Code: ")
    supplier_code = input("Enter Supplier Code: ")
    quantity_change = int(input("Enter Quantity Change (positive for addition, negative for removal): "))

    # Check if the quantity change is valid (not removing more than available)
    current_quantity = get_ppe_quantity(item_code, supplier_code)
    if quantity_change < 0 and current_quantity < abs(quantity_change):
        print("Stock is insufficient. Cannot remove more than available.\n")
        return

    # Update the quantity in ppe.txt
    with open('ppe.txt', 'r') as file:
        lines = file.readlines()

    updated = False
    with open('ppe.txt', 'w') as file:
        for line in lines:
            parts = line.strip().split(',')
            if parts[0] == item_code and parts[1] == supplier_code:
                new_quantity = int(parts[2]) + quantity_change
                if new_quantity < 0:
                    new_quantity = 0
                file.write(f"{item_code},{supplier_code},{new_quantity}\n")
                updated = True
                record_transaction(item_code, supplier_code, quantity_change)
            else:
                file.write(line)

    if not updated:
        print("Item and Supplier combination not found. Please check the details.\n")
    else:
        print("PPE quantity updated!\n")

# Function to get the current quantity of a PPE item
def get_ppe_quantity(item_code, supplier_code):
    with open('ppe.txt', 'r') as file:
        for line in file:
            parts = line.strip().split(',')
            if parts[0] == item_code and parts[1] == supplier_code:
                return int(parts[2])
    return 0

# Function to authenticate a user
def authenticate_user():
    user_id = input("Enter User ID: ")
    password = input("Enter Password: ")

    with open('users.txt', 'r') as file:
        for line in file:
            parts = line.strip().split(',')
            if parts[0] == user_id and parts[2] == password:
                return parts[3]  # Return user type

    return None

# Main program
if not os.path.exists(transactions_file):  # Check if transactions file exists
    create_transactions_file()  # If not, create it
try:
    with open('users.txt', 'r') as file:
        pass
except FileNotFoundError:
    create_initial_users_file()

try:
    with open('ppe.txt', 'r') as file:
        pass
except FileNotFoundError:
    create_initial_ppe_file()

try:
    with open('suppliers.txt', 'r') as file:
        pass
except FileNotFoundError:
    create_initial_suppliers_file()

while True:
    user_type = authenticate_user()

    if user_type is None:
        print("Invalid User ID or Password. Please try again.\n")
    elif user_type == 'admin':
        print("Welcome, Admin User!\n")
        display_current_stock()  # Display stock for admin
        print("Options:")
        print("1. Add User Information (Admin Only)")
        print("2. Delete user information (Admin Only)")
        print("3. Edit user information")
        print("4. Update PPE Item Quantities (Admin and Staff)")
        print("5. Search PPE Item Distribution")
        print("6. End")

        choice = input("Enter your choice (1, 2, 3, 4,5 or 6): ")

        if choice == '1':
            admin_add_user()
        elif choice == '2':
            admin_delete_user()
        elif choice == '3':
            admin_edit_user()
        elif choice == '4':
            update_ppe_quantity()
        elif choice == '5':
            search_item_distribution()
        elif choice == '6':
            break
        else:
            print("Invalid choice. Please enter 1, 2, 3, 4, 5, or 6.\n")
    elif user_type == 'staff':
        print("Welcome, Staff User!\n")
        display_current_stock()  # Display stock for staff
        print("Options:")
        print("1. Update PPE Item Quantities (Admin & Staff Only)")
        print("2. Search Item Distribution")
        print("3. Exit")

        choice = input("Enter your choice (1, 2, or 3): ")

        if choice == '1':
            update_ppe_quantity()
        elif choice == '2':
            search_item_distribution()
        elif choice == '3':
            break
        else:
            print("Invalid choice. Please enter 1, 2, or 3.\n")

print("Thank you and slay always.")
