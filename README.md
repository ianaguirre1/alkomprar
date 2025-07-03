# --- Constants ---
# Archivos ya no son necesarios si no se guardan los datos
# PRODUCTS_FILE = "products.json"
# SALES_FILE = "sales.json"

# --- Data Structures ---
products = {}  # Stores product data: {product_id: {title, author, category, price, stock}}
sales = []     # Stores sales data: [{sale_id, client, product_id, quantity, date, discount_percentage}]

# --- Helper Functions ---

def load_initial_data():
    """Initializes with sample data directly in memory."""
    global products, sales
    print("Initializing with sample products (data will not be saved).")
    products = {
        "P001": {"title": "The Lord of the Rings", "author": "J.R.R. Tolkien", "category": "Fantasy", "price": 25.00, "stock": 50},
        "P002": {"title": "Pride and Prejudice", "author": "Jane Austen", "category": "Romance", "price": 15.50, "stock": 30},
        "P003": {"title": "1984", "author": "George Orwell", "category": "Dystopian", "price": 12.00, "stock": 40},
        "P004": {"title": "To Kill a Mockingbird", "author": "Harper Lee", "category": "Fiction", "price": 18.75, "stock": 25},
        "P005": {"title": "The Great Gatsby", "author": "F. Scott Fitzgerald", "category": "Classic", "price": 10.00, "stock": 60}
    }
    sales = [] # Start with no sales for a clean simulation each run

def get_next_product_id():
    """Generates a new unique product ID."""
    if not products:
        return "P001"
    # Ensure all product IDs are strings before attempting int conversion for max
    max_id = max(products.keys(), key=lambda x: int(x[1:]))
    next_num = int(max_id[1:]) + 1
    return f"P{next_num:03d}"

def get_next_sale_id():
    """Generates a new unique sale ID."""
    if not sales:
        return "S001"
    # Ensure all sale IDs are strings before attempting int conversion for max
    max_id = max(sales, key=lambda x: int(x["sale_id"][1:]))["sale_id"]
    next_num = int(max_id[1:]) + 1
    return f"S{next_num:03d}"

def validate_positive_number(prompt, data_type=int):
    """Validates if user input is a positive number."""
    while True:
        try:
            value = data_type(input(prompt))
            if value <= 0:
                print("Input must be a positive number. Please try again.")
            else:
                return value
        except ValueError:
            print("Invalid input. Please enter a valid number.")

def validate_string_input(prompt, allow_empty=False):
    """Validates if user input for a string is not empty."""
    while True:
        value = input(prompt).strip()
        if not value and not allow_empty:
            print("This field cannot be empty. Please try again.")
        else:
            return value

# --- Inventory Management ---

def register_product():
    """Registers a new product into the inventory."""
    print("\n--- Register New Product ---")
    product_id = get_next_product_id()
    title = validate_string_input("Enter product title: ")
    author = validate_string_input("Enter product author: ")
    category = validate_string_input("Enter product category: ")
    price = validate_positive_number("Enter product price: ", float)
    stock = validate_positive_number("Enter product stock quantity: ")

    products[product_id] = {
        "title": title,
        "author": author,
        "category": category,
        "price": price,
        "stock": stock
    }
    print(f"Product '{title}' (ID: {product_id}) registered successfully.")

def consult_product():
    """Consults and displays details of a product."""
    print("\n--- Consult Product ---")
    product_id = validate_string_input("Enter product ID to consult: ").upper()
    product = products.get(product_id)
    if product:
        print(f"\nProduct ID: {product_id}")
        for key, value in product.items():
            print(f"  {key.replace('_', ' ').title()}: {value}")
    else:
        print(f"Product with ID '{product_id}' not found.")

def update_product():
    """Updates details of an existing product."""
    print("\n--- Update Product ---")
    product_id = validate_string_input("Enter product ID to update: ").upper()
    if product_id not in products:
        print(f"Product with ID '{product_id}' not found.")
        return

    product = products[product_id]
    print(f"Updating product: {product['title']} (ID: {product_id})")

    new_title = validate_string_input(f"Enter new title (current: {product['title']}): ", allow_empty=True)
    if new_title:
        product["title"] = new_title

    new_author = validate_string_input(f"Enter new author (current: {product['author']}): ", allow_empty=True)
    if new_author:
        product["author"] = new_author

    new_category = validate_string_input(f"Enter new category (current: {product['category']}): ", allow_empty=True)
    if new_category:
        product["category"] = new_category

    while True:
        price_input = input(f"Enter new price (current: {product['price']}) (leave empty to keep current): ")
        if not price_input:
            break
        try:
            new_price = float(price_input)
            if new_price <= 0:
                print("Price must be a positive number. Please try again.")
            else:
                product["price"] = new_price
                break
        except ValueError:
            print("Invalid input. Please enter a valid number for price.")

    while True:
        stock_input = input(f"Enter new stock quantity (current: {product['stock']}) (leave empty to keep current): ")
        if not stock_input:
            break
        try:
            new_stock = int(stock_input)
            if new_stock < 0:
                print("Stock quantity cannot be negative. Please try again.")
            else:
                product["stock"] = new_stock
                break
        except ValueError:
            print("Invalid input. Please enter a valid integer for stock.")

    print(f"Product '{product['title']}' (ID: {product_id}) updated successfully.")

def delete_product():
    """Deletes a product from the inventory."""
    print("\n--- Delete Product ---")
    product_id = validate_string_input("Enter product ID to delete: ").upper()
    if product_id in products:
        confirm = input(f"Are you sure you want to delete product '{products[product_id]['title']}' (ID: {product_id})? (yes/no): ").lower()
        if confirm == 'yes':
            del products[product_id]
            print(f"Product with ID '{product_id}' deleted successfully.")
        else:
            print("Product deletion cancelled.")
    else:
        print(f"Product with ID '{product_id}' not found.")

# --- Sales Management ---

def record_sale():
    """Records a new sale."""
    print("\n--- Record New Sale ---")
    client_name = validate_string_input("Enter client name: ")
    product_id = validate_string_input("Enter product ID to sell: ").upper()

    if product_id not in products:
        print(f"Product with ID '{product_id}' not found.")
        return

    product = products[product_id]
    print(f"Product selected: {product['title']} (Current Stock: {product['stock']})")

    quantity_to_sell = validate_positive_number("Enter quantity to sell: ")

    if quantity_to_sell > product["stock"]:
        print(f"Error: Insufficient stock. Only {product['stock']} units available.")
        return

    discount_percentage = 0.0
    while True:
        discount_input = input("Enter discount percentage (0-100, leave empty for no discount): ")
        if not discount_input:
            break
        try:
            discount_percentage = float(discount_input)
            if 0 <= discount_percentage <= 100:
                break
            else:
                print("Discount percentage must be between 0 and 100.")
        except ValueError:
            print("Invalid input. Please enter a valid number for discount.")

    # Update stock
    products[product_id]["stock"] -= quantity_to_sell

    sale_id = get_next_sale_id()
    # No real date without datetime module, using a placeholder
    sale_date = "Simulation Date"

    sales.append({
        "sale_id": sale_id,
        "client": client_name,
        "product_id": product_id,
        "quantity": quantity_to_sell,
        "date": sale_date,
        "discount_percentage": discount_percentage
    })
    print(f"Sale {sale_id} recorded successfully for {quantity_to_sell} units of '{product['title']}'.")

def consult_sales():
    """Displays all recorded sales."""
    print("\n--- All Recorded Sales ---")
    if not sales:
        print("No sales recorded yet.")
        return

    for sale in sales:
        product_info = products.get(sale["product_id"], {"title": "Unknown Product"})
        print(f"\nSale ID: {sale['sale_id']}")
        print(f"  Client: {sale['client']}")
        print(f"  Product: {product_info['title']} (ID: {sale['product_id']})")
        print(f"  Quantity: {sale['quantity']}")
        print(f"  Date: {sale['date']}")
        print(f"  Discount: {sale['discount_percentage']}%")

# --- Reporting Module ---

def top_selling_products():
    """Shows the top 3 most sold products."""
    print("\n--- Top 3 Most Sold Products ---")
    if not sales:
        print("No sales data available to determine top products.")
        return

    product_sales = {}
    for sale in sales:
        product_id = sale["product_id"]
        quantity = sale["quantity"]
        product_sales[product_id] = product_sales.get(product_id, 0) + quantity

    sorted_products = sorted(product_sales.items(), key=lambda item: item[1], reverse=True)

    if not sorted_products:
        print("No products have been sold yet.")
        return

    print("Rank | Product ID | Title                      | Total Units Sold")
    print("-----|------------|----------------------------|-----------------")
    for i, (product_id, total_sold) in enumerate(sorted_products[:3]):
        product = products.get(product_id, {"title": "Unknown Product"})
        print(f"{i + 1:<4} | {product_id:<10} | {product['title']:<26} | {total_sold:<16}")

def sales_report_by_author():
    """Generates a sales report grouped by author."""
    print("\n--- Sales Report by Author ---")
    if not sales:
