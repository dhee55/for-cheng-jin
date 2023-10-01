# for-cheng-jin
import csv
import datetime

# Function to create the initial users.txt file with an admin and a staff user
def create_initial_users_file():
    with open('users.txt', 'w') as file:
        file.write("admin123,Admin User,adminpassword,admin,Administrator\n")
        file.write("staff123,Staff User,staffpassword,staff,Staff\n")

# Function to create the initial ppe.txt file with 6 PPE items
def create_initial_ppe_file():
    with open('ppe.txt', 'w') as file:
        file.write("HC,Supplier1,100\n")
        file.write("FS,Supplier1,100\n")
        file.write("MS,Supplier2,100\n")
        file.write("GL,Supplier3,100\n")
        file.write("GW,Supplier4,100\n")
        file.write("SC,Supplier5,100\n")

# Function to create the initial suppliers.txt file with sample supplier data
def create_initial_suppliers_file():
    with open('suppliers.txt', 'w') as file:
        file.write("Supplier1,HC,123-456-7890\n")
        file.write("Supplier1,FS,123-456-7890\n")
        file.write("Supplier3,MS,555-555-5555\n")
        file.write("Supplier4,GL,777-777-7777\n")
        file.write("Supplier5,GW,888-888-8888\n")
        file.write("Supplier6,SC,999-999-9999\n")

# Function to display current stock for all items
def display_current_stock():
    with open('ppe.txt', 'r') as file:
        print("Current Stock:")
        print("{:<5} {:<15} {:<10}".format("Item", "Supplier", "Quantity"))
        for line in file:
            item_code, supplier_code, quantity = line.strip().split(',')
            print("{:<5} {:<15} {:<10}".format(item_code, supplier_code, quantity))


# Function to search and print details of items distributed for a particular item (Admin and Staff)
def search_item_distribution():
    item_code = input("Enter Item Code to search for distribution details: ")
    with open('transactions.txt', 'r') as file:
        print(f"Distribution Details for Item Code: {item_code}")
        print("{:<20} {:<15} {:<10}".format("Date-Time", "Supplier", "Quantity Change"))
        for line in file:
            timestamp, code, supplier_code, quantity_change = line.strip().split(',')
            if code == item_code:
                print("{:<20} {:<15} {:<10}".format(timestamp, supplier_code, quantity_change))



# Function to create transactions.txt and record transactions

def record_transaction(item_code, supplier_code, quantity_change):
    timestamp = datetime.datetime.now()
    transaction_data = f"{timestamp},{item_code},{supplier_code},{quantity_change}\n"
    with open('transactions.txt', 'a') as file:
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
        print("1. Edit/Add User Information (Admin Only)")
        print("2. Edit Supplier Information (Admin Only)")
        print("3. Update PPE Item Quantities (Admin and Staff)")
        print("4. Search Item Distribution")
        print("5. Exit")

        choice = input("Enter your choice (1, 2, 3, 4, or 5): ")

        if choice == '1':
            admin_add_user()
        elif choice == '2':
            admin_update_supplier()
        elif choice == '3':
            update_ppe_quantity()
        elif choice == '4':
            search_item_distribution()
        elif choice == '5':
            break
        else:
            print("Invalid choice. Please enter 1, 2, 3,4 or 5.\n")
    else:
        print("Welcome, Regular User!\n")
        display_current_stock()  # Display stock for staff
        print("Options:")
        print("1. Update PPE Item Quantities (Regular User)")
        print("2. Search Item Distribution")
        print("3. Exit")

        choice = input("Enter your choice (1,2 or 3): ")

        if choice == '1':
            update_ppe_quantity()
        elif choice == '2':
            search_item_distribution()
        elif choice == '3':
            break
        else:
            print("Invalid choice. Please enter 1 or 2.\n")

print("Thank you and slay always.")
