# manel-CTA
import json
from datetime import datetime, date
import os  # Added for file checking

# ===================== GLOBAL DATA =====================
# Dictionary of all train stations with their properties
stations = {
    "DTC": {"name": "Downtown Central", "zone": 1, "active": True},
    "UNI": {"name": "University Park", "zone": 2, "active": True},
    "TEC": {"name": "Tech District", "zone": 2, "active": True},
    "NOR": {"name": "Northgate", "zone": 3, "active": True},
    "SOU": {"name": "Southpoint", "zone": 3, "active": True},
    "EAS": {"name": "Eastwood", "zone": 4, "active": True},
    "WES": {"name": "Westfield", "zone": 4, "active": True}
}

# Base price for each travel zone
ZONE_PRICES = {1: 2.50, 2: 3.50, 3: 4.50, 4: 5.50}

# Different ticket types and their price multipliers
TICKET_TYPES = {
    "single": 1.0,  # Normal one-way ticket
    "return": 1.8,  # Return ticket (80% more than single)
    "weekly": 10.0  # Weekly unlimited pass
}

# Global variables to store all system data
tickets = []  # List to store all purchased tickets
next_ticket_id = 1001  # Starting ID for new tickets (CTA1001)
total_revenue = 0.0  # Total money earned from ticket sales


# ----------------- CORE FUNCTIONS ----------------------

def show_menu():
    """Display the main menu with all available options"""
    print("\n" + "=" * 50)
    print("        CTA TICKET BOOKING SYSTEM")
    print("=" * 50)
    print("1. Buy a Ticket")
    print("2. View All Stations")
    print("3. View Ticket Statistics")
    print("4. Calculate Price Only")
    print("5. Add New Station")
    print("6. Save Data to File")
    print("7. Load Data from File")
    print("8. Exit")
    print("-" * 50)


def show_stations():
    """Display all stations in a formatted table"""
    print("\n" + "=" * 50)
    print("STATION LIST")
    print("=" * 50)
    print(f"{'Code':<6} {'Name':<20} {'Zone':<6} {'Status':<10}")
    print("-" * 50)

    active_count = 0
    for code, info in stations.items():
        status = "ACTIVE" if info["active"] else "INACTIVE"
        if info["active"]:
            active_count += 1
        print(f"{code:<6} {info['name']:<20} {info['zone']:<6} {status:<10}")

    print("-" * 50)
    print(f"Total: {len(stations)} stations ({active_count} active)")


def calculate_price(origin_zone, dest_zone, ticket_type="single"):
    """Calculate the price of a ticket between two zones"""
    if ticket_type not in TICKET_TYPES:
        return None  # Invalid ticket type

    # Calculate base price (average of both zone prices)
    if origin_zone == dest_zone:
        base_price = ZONE_PRICES[origin_zone]
    else:
        base_price = (ZONE_PRICES[origin_zone] + ZONE_PRICES[dest_zone]) / 2

    # Apply ticket type multiplier and round to 2 decimal places
    final_price = base_price * TICKET_TYPES[ticket_type]
    return round(final_price, 2)


def validate_date(date_str):
    """Check if a date string is valid and not in the past"""
    try:
        # Convert string to date object
        travel_date = datetime.strptime(date_str, "%Y-%m-%d").date()
        today = date.today()

        # Check if date is in the past
        if travel_date < today:
            print(" Error: Travel date cannot be in the past!")
            return None

        # Optional: Check if date is too far in future (1 year max)
        max_date = date(today.year + 1, today.month, today.day)
        if travel_date > max_date:
            print(" Error: Cannot book more than 1 year in advance!")
            return None

        return travel_date
    except ValueError:
        # If date format is wrong
        print(" Error: Invalid date format! Use YYYY-MM-DD")
        return None


def buy_ticket():
    """Handle the complete ticket purchase process"""
    global next_ticket_id, total_revenue  # We'll modify these global variables

    print("\n" + "=" * 50)
    print("           BUY TICKET")
    print("=" * 50)

    # Show only active stations
    print("\nActive Stations:")
    print("-" * 40)
    for code, info in stations.items():
        if info["active"]:
            print(f"{code}: {info['name']} (Zone {info['zone']})")

    # Get departure station with validation
    while True:
        from_code = input("\nEnter FROM station code: ").upper().strip()
        if from_code in stations:
            if stations[from_code]["active"]:
                break  # Valid station found
            else:
                print(f" Station {from_code} is not active!")
        else:
            print(f" Station {from_code} not found!")
            print("Available codes:", ", ".join(stations.keys()))

    # Get destination station with validation
    while True:
        to_code = input("Enter TO station code: ").upper().strip()
        if to_code in stations:
            if stations[to_code]["active"]:
                if to_code != from_code:
                    break  # Valid and different station
                else:
                    print(" FROM and TO stations cannot be the same!")
            else:
                print(f" Station {to_code} is not active!")
        else:
            print(f" Station {to_code} not found!")

    # Get ticket type selection
    print("\nTicket Types Available:")
    for i, (ttype, multiplier) in enumerate(TICKET_TYPES.items(), 1):
        print(f"  {i}. {ttype.title()} Ticket")

    while True:
        type_choice = input("\nSelect ticket type (1-3): ").strip()
        if type_choice == "1":
            ticket_type = "single"
            break
        elif type_choice == "2":
            ticket_type = "return"
            break
        elif type_choice == "3":
            ticket_type = "weekly"
            break
        else:
            print(" Please enter 1, 2, or 3")

    # Get travel date with validation
    while True:
        date_str = input("\nEnter travel date (YYYY-MM-DD): ").strip()
        travel_date = validate_date(date_str)
        if travel_date:
            break  # Valid date found

    # Calculate the price
    from_zone = stations[from_code]["zone"]
    to_zone = stations[to_code]["zone"]
    price = calculate_price(from_zone, to_zone, ticket_type)

    # Show ticket summary before purchase
    print("\n" + "=" * 50)
    print("TICKET SUMMARY")
    print("=" * 50)
    print(f"From: {stations[from_code]['name']} (Zone {from_zone})")
    print(f"To: {stations[to_code]['name']} (Zone {to_zone})")
    print(f"Date: {travel_date}")
    print(f"Type: {ticket_type.title()} Ticket")
    print(f"Price: ${price:.2f}")
    print("=" * 50)

    # Ask for final confirmation
    confirm = input("\nConfirm purchase? (yes/no): ").lower().strip()

    if confirm == "yes" or confirm == "y":
        # Create ticket dictionary with all details
        ticket = {
            "id": f"CTA{next_ticket_id}",  # Generate unique ID
            "from": from_code,
            "to": to_code,
            "date": str(travel_date),
            "type": ticket_type,
            "price": price,
            "purchase_date": str(date.today())  # Today's date
        }

        # Add ticket to system and update totals
        tickets.append(ticket)
        total_revenue += price
        next_ticket_id += 1  # Prepare for next ticket

        print("\n TICKET PURCHASED SUCCESSFULLY!")
        print(f"   Ticket ID: {ticket['id']}")
        print(f"   Please keep this for your records.")
    else:
        print("\n Purchase cancelled.")


def view_statistics():
    """Display statistics about ticket sales"""
    if not tickets:
        print("\nNo tickets sold yet.")
        return

    print("\n" + "=" * 50)
    print("TICKET STATISTICS")
    print("=" * 50)

    # Basic totals
    print(f"Total Tickets Sold: {len(tickets)}")
    print(f"Total Revenue: ${total_revenue:.2f}")

    # Count tickets by type
    type_counts = {"single": 0, "return": 0, "weekly": 0}
    for ticket in tickets:
        type_counts[ticket["type"]] += 1

    print("\nTickets by Type:")
    for ttype, count in type_counts.items():
        if count > 0:
            print(f"  {ttype.title()}: {count}")

    # Find most popular stations
    print("\nMost Popular Stations:")
    departures = {}
    arrivals = {}

    for ticket in tickets:
        departures[ticket["from"]] = departures.get(ticket["from"], 0) + 1
        arrivals[ticket["to"]] = arrivals.get(ticket["to"], 0) + 1

    print("  Departures:")
    for station, count in sorted(departures.items(), key=lambda x: x[1], reverse=True)[:3]:
        print(f"    {station}: {count} tickets")

    print("  Arrivals:")
    for station, count in sorted(arrivals.items(), key=lambda x: x[1], reverse=True)[:3]:
        print(f"    {station}: {count} tickets")


def price_calculator():
    """Calculate ticket price without actually buying"""
    print("\n" + "=" * 50)
    print("PRICE CALCULATOR")
    print("=" * 50)

    try:
        print("\nZone numbers: 1=$2.50, 2=$3.50, 3=$4.50, 4=$5.50")

        # Get zone inputs
        from_zone = int(input("\nEnter FROM zone (1-4): "))
        to_zone = int(input("Enter TO zone (1-4): "))

        # Validate zones
        if from_zone not in range(1, 5) or to_zone not in range(1, 5):
            print(" Zones must be between 1 and 4!")
            return

        # Get ticket type
        print("\nTicket Types:")
        print("1. Single (one-way)")
        print("2. Return (round trip)")
        print("3. Weekly pass")

        choice = input("\nSelect type (1-3): ")
        if choice == "1":
            ticket_type = "single"
        elif choice == "2":
            ticket_type = "return"
        elif choice == "3":
            ticket_type = "weekly"
        else:
            print(" Invalid choice, using 'single'")
            ticket_type = "single"

        # Calculate and show price
        price = calculate_price(from_zone, to_zone, ticket_type)

        print("\n" + "=" * 30)
        print(f"Estimated Price: ${price:.2f}")
        print("=" * 30)

    except ValueError:
        print(" Please enter valid numbers!")


def add_station():
    """Add a new station to the system"""
    print("\n" + "=" * 50)
    print("ADD NEW STATION")
    print("=" * 50)

    # Get station name with validation
    while True:
        name = input("\nEnter station name: ").strip()
        if name:
            # Check if name already exists
            existing = any(info["name"].lower() == name.lower() for info in stations.values())
            if not existing:
                break
            else:
                print(" Station name already exists!")
        else:
            print(" Station name cannot be empty!")

    # Get zone with validation
    while True:
        try:
            zone = int(input("Enter zone (1-4): "))
            if 1 <= zone <= 4:
                break
            else:
                print(" Zone must be 1-4!")
        except ValueError:
            print(" Please enter a number!")

    # Get station code with validation
    while True:
        code = input("Enter 3-letter station code: ").upper().strip()
        if len(code) == 3 and code.isalpha():
            if code not in stations:
                break
            else:
                print(" Station code already exists!")
        else:
            print(" Code must be exactly 3 letters!")

    # Add the new station to the system
    stations[code] = {
        "name": name,
        "zone": zone,
        "active": True
    }

    print(f"\n Station '{name}' added successfully!")
    print(f"   Code: {code}, Zone: {zone}")


def save_data():
    """Save all system data to a JSON file"""
    # Prepare all data in one dictionary
    data = {
        "stations": stations,
        "tickets": tickets,
        "next_ticket_id": next_ticket_id,
        "total_revenue": total_revenue
    }

    try:
        # Write data to file
        with open("cta_data.json", "w") as f:
            json.dump(data, f, indent=4)
        print("\n Data saved to 'cta_data.json'")
    except Exception as e:
        print(f"\n Error saving data: {e}")


def load_data():
    """Load saved data from JSON file"""
    global stations, tickets, next_ticket_id, total_revenue

    # Check if file exists first
    if not os.path.exists("cta_data.json"):
        print("\nℹ No saved data found. Starting fresh.")
        return

    try:
        # Read data from file
        with open("cta_data.json", "r") as f:
            data = json.load(f)

        # Restore all system data
        stations = data["stations"]
        tickets = data["tickets"]
        next_ticket_id = data["next_ticket_id"]
        total_revenue = data["total_revenue"]

        print("\n Data loaded from 'cta_data.json'")
        print(f"   Loaded {len(tickets)} tickets")

    except json.JSONDecodeError:
        print("\n Data file is corrupted. Starting fresh.")
    except Exception as e:
        print(f"\n Error loading data: {e}")


# ------------------------- MAIN PROGRAM -------------------------

def main():
    """Main program loop that runs the ticket booking system"""
    print("\n" + "=" * 50)
    print("   WELCOME TO CTA TRANSPORTATION SYSTEM")
    print("=" * 50)
    print("   Zone-based Ticketing System")
    print("=" * 50)

    # Try to load saved data on startup
    load_data()

    # Main program loop
    while True:
        show_menu()

        try:
            choice = input("\nEnter your choice (1-8): ").strip()

            # Handle user's menu choice
            if choice == "1":
                buy_ticket()
            elif choice == "2":
                show_stations()
            elif choice == "3":
                view_statistics()
            elif choice == "4":
                price_calculator()
            elif choice == "5":
                add_station()
            elif choice == "6":
                save_data()
            elif choice == "7":
                load_data()
            elif choice == "8":
                # Ask to save before exiting
                save_choice = input("\nSave data before exiting? (yes/no): ").lower()
                if save_choice in ["yes", "y"]:
                    save_data()
                print("\nThank you for using CTA System. Goodbye!")
                break  # Exit the loop and program
            else:
                print(" Invalid choice! Please enter 1-8.")

        except KeyboardInterrupt:
            # Handle Ctrl+C gracefully
            print("\n\nProgram interrupted.")
            break
        except Exception as e:
            # Catch any unexpected errors
            print(f"\n An error occurred: {e}")

        # Pause between operations (except when exiting)
        if choice != "8":
            input("\nPress Enter to continue...")


# Start the program
if __name__ == "__main__":
    main()
