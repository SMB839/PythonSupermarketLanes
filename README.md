# importing random number libary ans time library

import random

import time


# creating a customer class with customer information

class Customer:

    def __init__(self, name, basket_size):

        self.name = name

        self.basket_size = basket_size

        self.checkout_time = 0

        self.won_lottery_ticket = False

    def checkout(self, checkout_lane):

        # Calculate checkout time based on basket size

        checkout_time = checkout_lane.calculate_checkout_time(self.basket_size)

        self.checkout_time = checkout_time

        # Display checkout information and check for lottery ticket

        print(f"{self.name} checked out with {self.basket_size} items in {checkout_time:.2f} seconds.", end=' ')

        if self.basket_size > 10:

            self.won_lottery_ticket = True

            print("- Congratulations, you won a lottery ticket!")

        else:

            print()

        checkout_lane.remove_customer(self)


# creates a checkoutstatus class and inside the class it defines whether a lane should close or not

class CheckoutStatus:

    def __init__(self, max_customers):
        self.max_customers = max_customers

    def should_close(self, current_customers):
        # Check if the checkout lane should be closed based on the amount of customers in the lane

        return current_customers == 0 or current_customers >= self.max_customers


# creates a checkoutlane class where it defines the lane number and the max customers for the lane

class CheckoutLane:

    def __init__(self, lane_number, max_customers, lane_type='Regular', cashier_name=None, num_self_service_tills=None):

        self.lane_number = lane_number

        self.is_open = False

        self.customers = []

        self.status_checker = CheckoutStatus(max_customers)

        self.lane_type = lane_type

        self.max_customers = max_customers

        self.item_processing_time = 4 if lane_type == 'Regular' else 8

        self.cashier_name = cashier_name if lane_type == 'Regular' else None

        self.num_self_service_tills = num_self_service_tills if lane_type == 'SelfService' else None

    def open(self):

        # Open the checkout lane and display information

        self.is_open = True

        print(
            f"Lane {self.lane_number} {'(Regular) ' if self.lane_type == 'Regular' else '(Self-Service) '} is now open.")

    def close(self):

        # Close the checkout lane and process remaining customers

        if not self.is_open:
            return

        self.is_open = False

        for customer in self.customers:
            customer.checkout(self)

        self.customers = []

        # Display information based on the type of checkout lane

        if self.lane_type == 'Regular':

            print(f"Lane {self.lane_number} (Regular) - Cashier: {self.cashier_name} is now closed.")

        elif self.lane_type == 'SelfService':

            print(
                f"Lane {self.lane_number} (Self-Service) - Unmanned Tills: {self.num_self_service_tills} is now closed.")

    def calculate_checkout_time(self, basket_size):

        # Calculate checkout time by multiplying item processing time and basket size

        return self.item_processing_time * basket_size

    def add_customer(self, customer, lane_manager):

        # Add a customer to a checkout lane, but make sure that only customers with less than 10 items are added to the self service lane

        if len(self.customers) < self.max_customers:

            if self.lane_type == 'SelfService' and (
                    customer.basket_size > 10 or len(self.customers) >= self.num_self_service_tills):

                print(f"{customer.name} with {customer.basket_size} items cannot join Self-Service Lane.")

                # the customer will be sent to the lane with the shortest que

                regular_lanes = [lane for lane in lane_manager if lane.is_open and lane.lane_type == 'Regular']

                if regular_lanes:

                    shortest_regular_lane = min(regular_lanes, key=lambda lane: len(lane.customers))

                    print(f"{customer.name} is being moved to Lane {shortest_regular_lane.lane_number} (Regular).")

                    shortest_regular_lane.add_customer(customer, lane_manager)

                else:

                    print(f"No open regular lanes. Cannot move {customer.name} to another lane.")

                return

            # Add the customer to the lane and display the lane number, customer basket

            self.customers.append(customer)

            print(
                f"{customer.name} joined Lane {self.lane_number} {'(Regular) ' if self.lane_type == 'Regular' else '(Self-Service) '} with {customer.basket_size} items.")

            print(
                f"Processing time for {customer.name}: {self.calculate_checkout_time(customer.basket_size):.2f} seconds",
                end=' ')

            if customer.basket_size > 10:
                print("- Awarded a lottery ticket!")

                customer.won_lottery_ticket = True

                print("Congratulations, you won a lottery ticket!")

        else:

            # if a lane is full output that information

            print(
                f"{customer.name} cannot join Lane {self.lane_number} {'(Regular) ' if self.lane_type == 'Regular' else '(Self-Service) '}, it's full.")

            if self.lane_type == 'Regular':

                # Try to find another open lane that is not at max capacity

                open_lanes_with_space = [lane for lane in lane_manager if
                                         lane.is_open and lane.lane_type == 'Regular' and len(
                                             lane.customers) < lane.max_customers]

                if open_lanes_with_space:

                    target_lane = min(open_lanes_with_space, key=lambda lane: len(lane.customers))

                    print(f"{customer.name} is being diverted to Lane {target_lane.lane_number} (Regular).")

                    target_lane.add_customer(customer, lane_manager)

                else:

                    # Open a closed lane if all open lanes are full

                    open_lanes = [lane for lane in lane_manager if lane.is_open and lane.lane_type == 'Regular']

                    closed_lanes = [lane for lane in lane_manager if not lane.is_open and lane.lane_type == 'Regular']

                    if open_lanes and all(len(lane.customers) >= lane.max_customers for lane in open_lanes):

                        if closed_lanes:

                            shortest_queue_lane = min(closed_lanes, key=lambda lane: len(lane.customers))

                            print(
                                f"Opening Lane {shortest_queue_lane.lane_number} (Regular) as all open lanes are full.")

                            shortest_queue_lane.open()

                            shortest_queue_lane.add_customer(customer, lane_manager)

                        else:

                            print(f"No closed lanes to open for {customer.name}.")

                    else:

                        # Check and close the lane if there are no customers

                        self.check_and_close_empty_lane()

    def check_and_close_empty_lane(self):

        if self.status_checker.should_close(len(self.customers)):
            self.close()

            print(
                f"Lane {self.lane_number} {'(Regular) ' if self.lane_type == 'Regular' else '(Self-Service) '} is closed with 0 customers.")

    def remove_customer(self, customer):

        # Remove a customer after checkout and display their basket size, checkout time and if they won a lottery ticket or not

        self.customers.remove(customer)

        print(
            f"{customer.name} has been removed from Lane {self.lane_number} {'(Regular) ' if self.lane_type == 'Regular' else '(Self-Service) '} after checkout.")

        if not self.customers:
            self.check_and_close_empty_lane()

    def display_lane_info(self):

        # Display information about the checkout lane

        if self.lane_type == 'Regular':

            print(
                f"Regular Lane {self.lane_number} - Cashier: {self.cashier_name} - {'Open' if self.is_open else 'Closed'}")

        elif self.lane_type == 'SelfService':

            print(
                f"Self-Service Lane {self.lane_number} - Unmanned Tills: {self.num_self_service_tills} - {'Open' if self.is_open else 'Closed'}")


def simulate_checkout():
    lane_manager = []

    # a list of all the lanes that the supermarket has

    regular_lane_1 = CheckoutLane(1, max_customers=5, lane_type='Regular', cashier_name="Cashier 1")

    regular_lane_2 = CheckoutLane(2, max_customers=5, lane_type='Regular', cashier_name="Cashier 2")

    regular_lane_3 = CheckoutLane(3, max_customers=5, lane_type='Regular', cashier_name="Cashier 3")

    regular_lane_4 = CheckoutLane(4, max_customers=5, lane_type='Regular', cashier_name="Cashier 4")

    regular_lane_5 = CheckoutLane(5, max_customers=5, lane_type='Regular', cashier_name="Cashier 5")

    self_service_lane = CheckoutLane(6, max_customers=8, lane_type='SelfService', num_self_service_tills=8)

    # opening the lanes that should be open at the start of the simulation

    regular_lane_1.open()

    regular_lane_2.close()

    regular_lane_3.close()

    regular_lane_4.close()

    regular_lane_5.close()

    self_service_lane.open()

    lane_manager.extend(
        [regular_lane_1, regular_lane_2, regular_lane_3, regular_lane_4, regular_lane_5, self_service_lane])

    while True:

        # Generate a new set of customers every 30 seconds

        time.sleep(30)

        # Check if all lanes are closed

        if all(not lane.is_open for lane in lane_manager):
            print("All lanes are closed. The supermarket is closed.")

            break

        # make sure that the random number of customers generated is between 1 and 10, every 30 seconds

        num_customers = random.randint(1, 10)

        for _ in range(num_customers):

            random_customer = Customer(name=f"Customer{random.randint(1, 10)}", basket_size=random.randint(1,
                                                                                                           30))  # make sure customer basketsize is a random number between 1 and 30

            open_lanes = [lane for lane in lane_manager if lane.is_open]

            eligible_lanes = [lane for lane in open_lanes if isinstance(lane, CheckoutLane)]

            if eligible_lanes:

                # Find the lane with the shortest queue

                shortest_queue_lane = min(eligible_lanes, key=lambda lane: len(lane.customers))

                shortest_queue_lane.add_customer(random_customer, lane_manager)

            else:

                # Try to find another open lane that is not at max capacity  or open a closed lane

                open_lanes_with_space = [lane for lane in lane_manager if
                                         lane.is_open and lane.lane_type == 'Regular' and len(
                                             lane.customers) < lane.max_customers]

                if open_lanes_with_space:

                    target_lane = min(open_lanes_with_space, key=lambda lane: len(lane.customers))

                    print(f"{random_customer.name} is being diverted to Lane {target_lane.lane_number} (Regular).")

                    target_lane.add_customer(random_customer, lane_manager)

                else:

                    # Open a closed regular lane if all open lanes are full

                    closed_regular_lanes = [lane for lane in lane_manager if
                                            not lane.is_open and lane.lane_type == 'Regular']

                    if closed_regular_lanes:

                        shortest_closed_regular_lane = min(closed_regular_lanes, key=lambda lane: len(lane.customers))

                        print(
                            f"{random_customer.name} is being diverted to Lane {shortest_closed_regular_lane.lane_number} (Regular).")

                        shortest_closed_regular_lane.open()

                        shortest_closed_regular_lane.add_customer(random_customer, lane_manager)

                    else:

                        print(f"No open or closed regular lanes for {random_customer.name}.")

                        self_service_lane.check_and_close_empty_lane()  # Check and close empty Self-Service lanes


# Run the simulation

simulate_checkout()
