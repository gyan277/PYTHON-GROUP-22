import collections

def display_catalog(catalog):
    """
    Displays the product catalog with product names and prices.
    """
    print("\n--- Product Catalog ---")
    if not catalog:
        print("No products available in the catalog.")
        return

    print(f"{'Product':<20} {'Price':>10}")
    print("-" * 32)
    for product, price in catalog.items():
        print(f"{product:<20} GHC{price:>9.2f}")
    print("-" * 32)

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

    print(f"{'Product':<20} {'Qty':>5} {'Unit Price':>12} {'Subtotal':>12}")
    print("-" * 50)
    for product, quantity in cart.items():
        if product in catalog:
            unit_price = catalog[product]
            subtotal = unit_price * quantity
            print(f"{product:<20} {quantity:>5} GHC{unit_price:>11.2f} GHC{subtotal:>11.2f}")
        else:
            # This case should ideally not happen if add_to_cart is robust
            print(f"{product:<20} {quantity:>5} (Item not found in catalog!)")
    print("-" * 50)

def update_cart(cart, catalog):
    """
    Allows the user to update the quantity of an item or remove it from the cart.
    """
    if not cart:
        print("Your cart is empty. Nothing to update.")
        return

    view_cart(cart, catalog) # Show current cart for user reference

    while True:
        product_name = input("Enter product name to update/remove (or 'done' to finish): ").strip().title()
        if product_name == 'Done':
            break

        if product_name not in cart:
            print(f"'{product_name}' is not in your cart.")
            continue

        while True:
            action = input(f"Enter new quantity for {product_name} (0 to remove, or 'cancel'): ").strip().lower()
            if action == 'cancel':
                break
            try:
                new_quantity = int(action)
                if new_quantity < 0:
                    print("Quantity cannot be negative.")
                elif new_quantity == 0:
                    del cart[product_name]
                    print(f"'{product_name}' removed from cart.")
                    break
                else:
                    cart[product_name] = new_quantity
                    print(f"Quantity for {product_name} updated to {new_quantity}.")
                    break
            except ValueError:
                print("Invalid input. Please enter a whole number or 'cancel'.")
        if action != 'cancel': # If an update/removal happened, ask if they want to continue
            continue_update = input("Update another item? (yes/no): ").strip().lower()
            if continue_update != 'yes':
                break
    print("Finished updating cart.")


def calculate_total(cart, catalog, discounts):
    """
    Calculates the total cost of items in the cart, applying discounts.
    Returns total cost and a list of applied discounts.
    """
    total_cost = 0.0
    applied_discounts = collections.defaultdict(float) # To store discount amount per product

    for product, quantity in cart.items():
        if product in catalog:
            unit_price = catalog[product]
            effective_quantity = quantity
            discount_amount_for_item = 0.0

            # Apply discount logic
            if product in discounts:
                discount_info = discounts[product]
                if discount_info["type"] == "buy_n_get_m_free":
                    buy_n = discount_info["buy"]
                    get_m_free = discount_info["get_free"]
                    
                    if quantity >= buy_n:
                        # Calculate how many free items are earned
                        num_sets = quantity // (buy_n + get_m_free) # Number of full sets of (buy+free)
                        free_items_from_sets = num_sets * get_m_free
                        
                        # Calculate how many free items are earned from just 'buy_n' groups
                        # This is an alternative interpretation for "buy N get M free"
                        # For "buy 3 get 1 free", if you buy 4, you get 1 free. If you buy 6, you get 2 free.
                        # This means for every N items, M are free.
                        # The cost is for (N / (N+M)) of the items.
                        
                        # Let's use the interpretation: for every 'buy_n' items, 'get_m_free' are free.
                        # The user pays for 'buy_n' items for every 'buy_n + get_m_free' items.
                        
                        # Example: buy 3 get 1 free.
                        # If quantity = 4, pay for 3. (1 free)
                        # If quantity = 7, pay for 6. (1 free from first 4, 1 free from next 3. Total 2 free)
                        
                        # Number of items to pay for:
                        paid_items_per_set = buy_n
                        total_items_per_set = buy_n + get_m_free
                        
                        num_full_sets = quantity // total_items_per_set
                        remaining_items = quantity % total_items_per_set
                        
                        effective_quantity = (num_full_sets * paid_items_per_set) + min(remaining_items, paid_items_per_set)
                        
                        # Calculate the actual items that are free
                        free_items = quantity - effective_quantity
                        discount_amount_for_item = free_items * unit_price
                        applied_discounts[product] += discount_amount_for_item
                        
                        print(f"DEBUG: {product}: Qty={quantity}, Effective Qty={effective_quantity}, Free Items={free_items}, Discount=${discount_amount_for_item:.2f}")

            cost_for_item = effective_quantity * unit_price
            total_cost += cost_for_item
        else:
            print(f"Warning: Product '{product}' in cart not found in catalog. Skipping.")

    return total_cost, applied_discounts

def print_invoice(cart, catalog, discounts):
    """
    Prints the final invoice listing all items, prices, and the total.
    Includes details about applied discounts.
    """
    print("\n--- Final Invoice ---")
    if not cart:
        print("Your cart is empty. No invoice to generate.")
        return

    # First, calculate the total and discounts
    total_cost, applied_discounts = calculate_total(cart, catalog, discounts)

    print(f"{'Product':<20} {'Qty':>5} {'Unit Price':>12} {'Line Total':>12} {'Discount':>12}")
    print("-" * 71)
    
    overall_discount_total = 0.0

    for product, quantity in cart.items():
        if product in catalog:
            unit_price = catalog[product]
            
            # Calculate line total before discount for display
            line_total_before_discount = unit_price * quantity
            
            # Get the discount applied to this specific product
            discount_for_this_product = applied_discounts[product]
            overall_discount_total += discount_for_this_product

            # Calculate the actual cost for this line item after discount
            line_total_after_discount = line_total_before_discount - discount_for_this_product

            print(f"{product:<20} {quantity:>5} ${unit_price:>11.2f} ${line_total_after_discount:>11.2f} ${discount_for_this_product:>11.2f}")
        else:
            print(f"{product:<20} {quantity:>5} (Item not found in catalog!)")
    
    print("-" * 71)
    print(f"{'Total Savings':<58} GHC{overall_discount_total:>11.2f}")
    print(f"{'Grand Total':<58} GHC{total_cost:>11.2f}")
    print("-" * 71)
    print("Thank you for shopping with us!")

def main_menu():
    """
    Main function to run the command-line shopping system.
    """
    # Initialize product catalog
    catalog = {
        "Iphone XS MAX": 2050.99,
        "Iphone XR": 2545.99,
        "Iphone 11": 3100.99,
        "Iphone 11 pro": 3333.99,
        "Iphone 12": 3476.99,
        "Iphone 12 mini": 3456.99,
        "Iphone 12 pro": 3645.99,
        "Iphone 12 pro max": 3766.99,
        "Iphone 13": 3900.99,
        "Iphone 13 pro": 3997.99,
        "Iphone 13 pro max": 4050.99,
        "Iphone 13 mini": 4007.99,
        "Iphone 14": 4503.99,
        "Iphone 14 pro ": 4555.99,
        "Iphone 14 pro max ": 4890.99,
        "Iphone 15 ": 6999.99,
        "Iphone 15 pro max ": 7033.99,
        "Iphone 16": 7540.99,
        "Iphone 16 pro ": 7888.99,
        "Iphone 16 pro max ": 8090.99,
        "Iphone 16 Air ": 8099.99,
        "Iphone 17 ": 12044.99,
        "Iphone 17 pro max ": 12890.99,
        "Iphone 17 Air": 12907.99,
    }

    # Initialize shopping cart
    cart = {}

    # Define discount rules
    # Example: "Iphone XS MAX": buy 3 get 1 free
    discounts = {
        "Iphone XR": {"type": "buy_n_get_m_free", "buy": 3, "get_free": 1},
        "Iphone XS MAX": {"type": "buy_n_get_m_free", "buy": 2, "get_free": 1}
    }

    print("Welcome to IPHONE DEALERS SHOP (GROUP 22)")

    while True:
        print("\nDamnnn that's the main menu")
        print("1. View Products")
        print("2. Add Item to Cart")
        print("3. View Cart")
        print("4. Update/Remove Item from Cart")
        print("5. Checkout (Print Invoice)")
        print("6. Exit")

        choice = input("Enter your choice: ").strip()

        if choice == '1':
            display_catalog(catalog)
        elif choice == '2':
            add_to_cart(cart, catalog)
        elif choice == '3':
            view_cart(cart, catalog)
        elif choice == '4':
            update_cart(cart, catalog)
        elif choice == '5':
            print_invoice(cart, catalog, discounts)
            # After checkout, you might want to clear the cart or exit
            # For simplicity, we'll just go back to the menu.
            # If you want to clear the cart after checkout: cart.clear()
        elif choice == '6':
            print("Thank you for shopping! Goodbye.")
            break
        else:
            print("Invalid choice. Please enter a number between 1 and 6.")

# Run the main menu function when the script is executed
if __name__ == "__main__":
    main_menu()
