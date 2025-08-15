import collections

def display_catalog(catalog):
    """
    Displays the product catalog with product names and prices.
    """
    print("\n--- Product Catalog ---")
    if not catalog:
        print("No products available in the catalog.")
        return

    print(f"{'Product':<25} {'Price':>12}")
    print("-" * 40)
    for product, price in catalog.items():
        print(f"{product:<25} GH₵{price:>10.2f}")
    print("-" * 40)

def add_to_cart(cart, catalog):
    """
    Allows the user to add products to their cart.
    Handles invalid product names and quantities.
    """
    while True:
        product_name = input("Enter product name to add (or 'done' to finish): ").strip().title()
        if product_name == 'Done':
            break

        if product_name not in catalog:
            print(f"'{product_name}' is not in the catalog. Please choose from the available products.")
            # Show available products for reference
            print("Available products:")
            for i, product in enumerate(catalog.keys(), 1):
                print(f"  {i}. {product}")
            continue

        while True:
            try:
                quantity = int(input(f"Enter quantity for {product_name}: "))
                if quantity <= 0:
                    print("Quantity must be a positive number.")
                else:
                    break
            except ValueError:
                print("Invalid quantity. Please enter a whole number.")

        # Update cart: if product already exists, add to quantity; otherwise, add new item
        cart[product_name] = cart.get(product_name, 0) + quantity
        print(f"Added {quantity} x {product_name} to your cart.")
    print("Finished adding items to cart.")

def view_cart(cart, catalog):
    """
    Displays the current items in the shopping cart with quantities and prices.
    """
    print("\n--- Your Shopping Cart ---")
    if not cart:
        print("Your cart is empty.")
        return

    print(f"{'Product':<25} {'Qty':>5} {'Unit Price':>12} {'Subtotal':>12}")
    print("-" * 60)
    total_before_discount = 0
    for product, quantity in cart.items():
        if product in catalog:
            unit_price = catalog[product]
            subtotal = unit_price * quantity
            total_before_discount += subtotal
            print(f"{product:<25} {quantity:>5} GH₵{unit_price:>10.2f} GH₵{subtotal:>10.2f}")
        else:
            print(f"{product:<25} {quantity:>5} (Item not found in catalog!)")
    print("-" * 60)
    print(f"{'SUBTOTAL:':<43} GH₵{total_before_discount:>10.2f}")
    print("-" * 60)

def update_cart(cart, catalog):
    """
    Allows the user to update the quantity of an item or remove it from the cart.
    """
    if not cart:
        print("Your cart is empty. Nothing to update.")
        return

    view_cart(cart, catalog)  # Show current cart for user reference

    while True:
        product_name = input("Enter product name to update/remove (or 'done' to finish): ").strip().title()
        if product_name == 'Done':
            break

        if product_name not in cart:
            print(f"  '{product_name}' is not in your cart.")
            continue

        while True:
            action = input(f"Enter new quantity for {product_name} (0 to remove, or 'cancel'): ").strip().lower()
            if action == 'cancel':
                break
            try:
                new_quantity = int(action)
                if new_quantity < 0:
                    print(" Quantity cannot be negative.")
                elif new_quantity == 0:
                    del cart[product_name]
                    print(f" '{product_name}' removed from cart.")
                    break
                else:
                    old_quantity = cart[product_name]
                    cart[product_name] = new_quantity
                    print(f" Quantity for {product_name} updated from {old_quantity} to {new_quantity}.")
                    break
            except ValueError:
                print(" Invalid input. Please enter a whole number or 'cancel'.")
        
        if action != 'cancel':  # If an update/removal happened, ask if they want to continue
            continue_update = input("Update another item? (yes/no): ").strip().lower()
            if continue_update not in ['yes', 'y']:
                break
    print("Finished updating cart.")

def calculate_discount_for_product(quantity, buy_n, get_m_free):
    """
    Calculate discount for a single product.
    Simple logic: for every buy_n items purchased, get_m_free items free
    """
    if quantity < buy_n:
        return 0
    
    # How many complete discount sets can we apply?
    discount_sets = quantity // buy_n
    free_items = discount_sets * get_m_free
    
    # Don't give more free items than total quantity
    return min(free_items, quantity)

def calculate_total(cart, catalog, discounts):
    """
    Calculates the total cost of items in the cart, applying discounts.
    Returns total cost and a dictionary of applied discounts.
    """
    total_cost = 0.0
    applied_discounts = {}
    discount_details = []

    for product, quantity in cart.items():
        if product in catalog:
            unit_price = catalog[product]
            item_total_before_discount = unit_price * quantity
            
            # Check for discounts
            free_items = 0
            if product in discounts:
                discount_info = discounts[product]
                if discount_info["type"] == "buy_n_get_m_free":
                    buy_n = discount_info["buy"]
                    get_m_free = discount_info["get_free"]
                    free_items = calculate_discount_for_product(quantity, buy_n, get_m_free)
            
            # Calculate the actual cost after discount
            paid_items = quantity - free_items
            item_total_after_discount = paid_items * unit_price
            discount_amount = free_items * unit_price
            
            total_cost += item_total_after_discount
            
            if free_items > 0:
                applied_discounts[product] = {
                    'free_items': free_items,
                    'discount_amount': discount_amount,
                    'buy_n': buy_n,
                    'get_m_free': get_m_free
                }
                discount_details.append(f"Buy {buy_n} Get {get_m_free} Free - {product}: {free_items} free items")
        else:
            print(f"Warning: Product '{product}' in cart not found in catalog. Skipping.")

    return total_cost, applied_discounts, discount_details

def print_invoice(cart, catalog, discounts):
    """
    Prints the final invoice listing all items, prices, and the total.
    Includes details about applied discounts.
    """
    print("\n" + "=" * 70)
    print("                    IPHONE DEALERS SHOP")
    print("                       GROUP 22")
    print("                     FINAL INVOICE")
    print("=" * 70)
    
    if not cart:
        print("Your cart is empty. No invoice to generate.")
        return

    # Calculate totals and discounts
    total_cost, applied_discounts, discount_details = calculate_total(cart, catalog, discounts)

    print(f"{'Product':<25} {'Qty':>5} {'Unit Price':>12} {'Line Total':>12} {'Discount':>12}")
    print("-" * 70)
    
    subtotal_before_discount = 0
    total_discount_amount = 0

    for product, quantity in cart.items():
        if product in catalog:
            unit_price = catalog[product]
            line_total_before_discount = unit_price * quantity
            subtotal_before_discount += line_total_before_discount
            
            discount_amount = 0
            if product in applied_discounts:
                discount_amount = applied_discounts[product]['discount_amount']
                total_discount_amount += discount_amount
            
            line_total_after_discount = line_total_before_discount - discount_amount

            print(f"{product:<25} {quantity:>5} GH₵{unit_price:>10.2f} GH₵{line_total_after_discount:>10.2f} GH₵{discount_amount:>10.2f}")
        else:
            print(f"{product:<25} {quantity:>5} (Item not found in catalog!)")
    
    print("-" * 70)
    print(f"{'SUBTOTAL:':<58} GH₵{subtotal_before_discount:>10.2f}")
    
    if discount_details:
        print(f"\n{'DISCOUNTS APPLIED:':<20}")
        for detail in discount_details:
            print(f"  • {detail}")
        print(f"{'TOTAL SAVINGS:':<58} GH₵{total_discount_amount:>10.2f}")
    
    print(f"{'GRAND TOTAL:':<58} GH₵{total_cost:>10.2f}")
    print("=" * 70)
    print("            Thank you for shopping with us!")
    print("=" * 70)

def validate_system(catalog, discounts):
    """
    Validate that the system configuration is correct
    """
    print("Validating system configuration...")
    
    # Check for discount products that don't exist in catalog
    invalid_discount_products = []
    for product in discounts:
        if product not in catalog:
            invalid_discount_products.append(product)
    
    if invalid_discount_products:
        print(f"Warning: These discount products are not in the catalog: {invalid_discount_products}")
        print("   Discounts for these products will not apply.")
    
    # Check for trailing spaces in product names
    products_with_spaces = []
    for product in catalog:
        if product != product.strip():
            products_with_spaces.append(product)
    
    if products_with_spaces:
        print(f"Warning: These products have leading/trailing spaces: {products_with_spaces}")
        print("   This may cause matching issues.")
    
    if not invalid_discount_products and not products_with_spaces:
        print("System configuration is valid!")

def main_menu():
    """
    Main function to run the command-line shopping system.
    """
    # Initialize product catalog (cleaned up - removed trailing spaces and unrealistic models)
    catalog = {
        "Iphone 11": 2100.99,
        "Iphone 11 Pro": 2333.99,
        "Iphone 12": 2476.99,
        "Iphone 12 Mini": 2356.99,
        "Iphone 12 Pro": 2645.99,
        "Iphone 12 Pro Max": 2766.99,
        "Iphone 13": 2900.99,
        "Iphone 13 Pro": 2997.99,
        "Iphone 13 Pro Max": 3150.99,
        "Iphone 13 Mini": 2807.99,
        "Iphone 14": 3203.99,
        "Iphone 14 Pro": 3355.99,
        "Iphone 14 Pro Max": 3590.99,
        "Iphone 15": 4999.99,
        "Iphone 15 Pro": 5233.99,
        "Iphone 15 Pro Max": 5633.99,
    }

    # Initialize shopping cart
    cart = {}

    # Define discount rules (using products that actually exist in catalog)
    discounts = {
        "Iphone 11": {"type": "buy_n_get_m_free", "buy": 3, "get_free": 1},
        "Iphone 12": {"type": "buy_n_get_m_free", "buy": 2, "get_free": 1},
        "Iphone 13": {"type": "buy_n_get_m_free", "buy": 3, "get_free": 1}
    }

    print("Welcome to IPHONE DEALERS SHOP (GROUP 22)")
    print("=" * 50)
    
    # Validate system configuration
    validate_system(catalog, discounts)

    while True:
        print("\nSHOPPING MAIN MENU")
        print("-" * 30)
        print("1. View Products")
        print("2. Add Item to Cart")
        print("3. View Cart")
        print("4. Update/Remove Item from Cart")
        print("5. Checkout (Print Invoice)")
        print("6. Exit")
        print("-" * 30)

        choice = input("Enter your choice (1-6): ").strip()

        if choice == '1':
            display_catalog(catalog)
        elif choice == '2':
            display_catalog(catalog)  # Show catalog before adding
            add_to_cart(cart, catalog)
        elif choice == '3':
            view_cart(cart, catalog)
        elif choice == '4':
            update_cart(cart, catalog)
        elif choice == '5':
            print_invoice(cart, catalog, discounts)
            # Ask if user wants to continue shopping or clear cart
            continue_shopping = input("\nDo you want to continue shopping? (y/n): ").strip().lower()
            if continue_shopping not in ['y', 'yes']:
                print("Thank you for shopping with us! Goodbye.")
                break
            else:
                # Optionally clear cart after checkout
                clear_cart = input("Clear cart for new shopping? (y/n): ").strip().lower()
                if clear_cart in ['y', 'yes']:
                    cart.clear()
                    print("Cart cleared!")
        elif choice == '6':
            if cart:
                confirm_exit = input("You have items in your cart. Are you sure you want to exit? (y/n): ").strip().lower()
                if confirm_exit in ['y', 'yes']:
                    print("Thank you for visiting IPHONE DEALERS SHOP! Goodbye.")
                    break
            else:
                print("Thank you for visiting IPHONE DEALERS SHOP! Goodbye.")
                break
        else:
            print("Invalid choice. Please enter a number between 1 and 6.")

# Run the main menu function when the script is executed
if __name__ == "__main__":
    main_menu()