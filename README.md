# RPA to collect data from Website and fill out excel 

Documentation

Book Store Automation Script
This script automates comprehensive testing and data collection for an online book store. It performs the following tasks:

Data Collection: Extracts book titles and prices from the website.
Validation Tests: Tests cart functionality, price sorting, and search features.
Data Export: Saves the collected data into an Excel file for further analysis.
Prerequisites
Python Modules:

robocorp.tasks
RPA.Excel.Files
RPA.Browser.Selenium
Browser Configuration:

The script uses a browser automation library to interact with the website. Ensure you have Selenium installed and properly configured.
Functionality Overview
book_store_automation
Main function that orchestrates the workflow:

Opens the book store website.
Collects book titles and prices.
Performs validation tests:
Cart functionality.
Price sorting.
Search feature.
Exports collected data to an Excel file.
Logs the results of all validation tests.
open_website
Navigates to the book store's website.

collect_book_data
Extracts book titles and prices from the website:

Retrieves all book titles.
Converts price text to numerical values.
validate_cart_functionality
Tests cart addition and verifies if the cart count updates correctly:

Checks the initial state of the cart.
Adds a book to the cart and verifies the cart count.
validate_sort_by_price
Tests the price sorting functionality:

Sorts books by price.
Validates if the displayed prices are sorted correctly.
validate_search_functionality
Tests the search feature:

Enters a search query.
Checks if relevant results are returned.
Prints results if found.
export_book_data
Exports collected book data (titles and prices) to an Excel file:

Adds a header row and the data rows.
Applies styling to the Excel sheet.
Saves and closes the file.
print_validation_results
Logs the results of validation tests in a human-readable format.

Example Output
Upon execution, the script:

Opens the book store website.
Collects book titles and prices, such as:
Titles: "AI Foundations", "Python for Beginners"
Prices: $25.99, $33.99
Validates cart functionality, sorting, and search features.
Exports book data to an Excel file (BookStore_Inventory.xlsx).
Logs validation results, e.g.:
yaml
Copy code
Validation Results:
Cart Functionality: Passed
Price Sorting: Failed
Search Functionality: Passed
Error Handling
The script includes exception handling for:

Website navigation issues.
Data collection errors.
Validation test failures.
Excel export errors.
Usage
Run the script using the command:

bash
Copy code
python book_store_automation.py
Dependencies
Selenium WebDriver for browser interactions.
RPA.Excel.Files for Excel operations.
Ensure that the target website is accessible and matches the structure expected by the script.
Link of how it works : https://youtu.be/94EN3-SDrZo?si=EuyI6PMCznxmztVu 
This is the code: 

```Python
from robocorp.tasks import task
from robocorp import browser
from RPA.Excel.Files import Files
from RPA.Browser.Selenium import Selenium

@task
def book_store_automation():
    """
    Comprehensive book store website testing automation:
    - Collect book titles and prices
    - Validate cart functionality
    - Test sorting and search features
    - Export data to Excel
    """
    browser.configure(slowmo=50)
    try:
        open_website()
        titles, prices = collect_book_data()
        
        # Perform various validations
        validation_results = {
            'cart_functionality': validate_cart_functionality(),
            'price_sorting': validate_sort_by_price(),
            'search_functionality': validate_search_functionality()
        }
        # Export results to Excel
        export_book_data(titles, prices)
        # Log validation results
        print_validation_results(validation_results)
    
    except Exception as e:
        print(f"An error occurred during automation: {e}")

def open_website():
    """Navigate to the book store website"""
    browser.goto("https://fspacheco.github.io/rpa-challenge/kirjakauppa.html")

def collect_book_data():
    """Extract book titles and prices"""
    page = browser.page()
    
    # More robust data collection
    try:
        titles = page.locator("div.kirjan-nimi").all_text_contents()
        prices_text = page.locator("div.hinta").all_text_contents()
        prices = [float(p.replace('$', '').strip()) for p in prices_text]
        return titles, prices
    
    except Exception as e:
        print(f"Error collecting book data: {e}")
        return [], []

def validate_cart_functionality():
    """Test cart addition and count update"""
    page = browser.page()
    
    #try:
        # Check initial cart state
    initial_cart_count = (page.locator("#cart-info").text_content())
    initial_cart_count = page.locator("#cart-info").text_content()
    initial_cart_count = initial_cart_count.replace('Ostoskorin tuotteet::','')
    initial_cart_count= int(initial_cart_count)    
    if initial_cart_count != 0:
        print("Warning: Initial cart not empty")
        
        # Add first book to cart
    page.locator("text=Lisää Ostoskoriin").first.click()
        
    initial_cart_count = page.locator("#cart-info").text_content()
    initial_cart_count = initial_cart_count.replace('Ostoskorin tuotteet::','')
    initial_cart_count= int(initial_cart_count)
    if initial_cart_count != 1:
        print("Warning: Error of cart count")

def validate_sort_by_price():
    """Test price sorting functionality"""
    page = browser.page()
    try:
        # Trigger price sorting
        page.locator("#sortSelect").select_option("price")
        
        # Collect sorted prices
        sorted_prices_text = page.locator("div.hinta").all_text_contents()
        sorted_prices = [float(p.replace('$', '').strip()) for p in sorted_prices_text]
        
        if sorted_prices_text != 33.99:
            print("Warning: Error of sort of prices")
            
        return sorted_prices == sorted(sorted_prices)
    
    except Exception as e:
        print(f"Price sorting validation error: {e}")
        return False

def validate_search_functionality():
    """Test book search feature"""
    page = browser.page()
    
    try:
        search_query = "AI Foundations"
        page.locator("#searchInput").fill(search_query)
        
        # Press Enter or click search button to trigger search
        page.keyboard.press("Enter")
        # Or if there's a specific search button:
        # page.locator("#search-button").click()
        # Wait for search results to load
        # Get search results
        search_results = page.locator("#searchInput").all_text_contents()
        # Check if any results found
        if not search_results:
            print(f"No books found for search query: '{search_query}'")
            return False
        
        # Print results if found
        print(f"Warning: Search results for '{search_query}' was not found.\nPlease fix the website")
        for result in search_results:
            print(result)
        
        return True
    except Exception as e:
        # Handle cases where search element is not found or other errors
        if "Timeout" in str(e):
            print(f"Warning: No search results found for '{search_query}'")
        else:
            print(f"Search functionality error: {e}")
        return False
    
def export_book_data(titles, prices):
    """Save book data to Excel with enhanced error handling"""
    try:
        excel = Files()
        excel.create_workbook("BookStore_Inventory.xlsx", sheet_name="Books")
        
        # Add header
        excel.append_rows_to_worksheet([("Title", "Price")])
        
        # Add book data
        for title, price in zip(titles, prices):
            excel.append_rows_to_worksheet([(title, price)])
        # Style spreadsheet
        excel.auto_size_columns("A", width=55)
        excel.auto_size_columns("B", width=15)
        excel.set_styles("B12", bold=True, font_name="Arial", size=14, cell_fill="Yellow")
        excel.save_workbook()
        excel.close_workbook()
        
        print("Book data successfully exported to Excel")
    
    except Exception as e:
        print(f"Excel export error: {e}")
def print_validation_results(results):
    """Print detailed validation results"""
    print("\nValidation Results:")
    for test, passed in results.items():
        status = "Passed" if passed else "Failed"
        print(f"{test.replace('_', ' ').title()}: {status}")

if __name__ == "__main__":
    book_store_automation()
```
Done






# RPA-with-python 12-11-2024
Using RPA with python to autamte repetitive tasks

Automated Form Submission and Email Verification
This project automates the process of filling a form with user data, submitting it, and verifying the email confirmation for all submitted entries. It utilizes the Robocorp and RPA.HTTP libraries to streamline repetitive form handling tasks.
Features
•	- Downloads a sample data file containing form inputs.
•	- Automates the form submission process for multiple entries.
•	- Logs into a specified email account to verify email confirmations.
•	- Captures screenshots of email confirmations for reference.
Dependencies
Ensure the following dependencies are installed in your environment:
•	- Robocorp Tasks: Automation framework for Python.
•	- Robocorp Browser: For browser automation.
•	- RPA.HTTP: For HTTP requests.
•	- A config.py file containing:
  - EMAIL: Your email address.
  - EMAIL_PASSWORD: Your email password.
How It Works
•	- Download Sample Data: The script downloads a text file containing sample data for form submission.
•	- Process Data: The data file is read and parsed into multiple entries. Each entry is used to fill a web form.
•	- Automated Form Submission: Opens the website, selects the service type, and fills the form fields (name, email, date, etc.). Submits the form for all entries.
•	- Email Confirmation: Logs into an email account, waits for email confirmations, and captures screenshots.
•	- Output: A screenshot of the form confirmation is saved in the output/ directory.
Functions
•	- handle_form_submission(): The main task that orchestrates the entire process: downloads the sample file, reads the file and processes each entry, submits the form, and verifies email confirmations.
•	- download_sample_file(): Downloads the sample file from a specified URL.
•	- read_sample_file(): Parses the downloaded text file and returns a list of data entries.
•	- open_website(): Navigates to the form submission website.
•	- select_service(service_type): Selects the service type (e.g., Maintenance, Program Modification).
•	- fill_contact_form(data): Fills the form fields using the provided data: additional notes, name, email, phone, date, and payment details. Submits the form.
•	- log_in(): Automates login to an email account for confirmation.
•	- collect_results(): Captures a screenshot of the email confirmation page.
Configuration
Create a config.py file with the following variables:

EMAIL = "your_email@example.com"
EMAIL_PASSWORD = "your_password"

Usage
1. Clone this repository and navigate to the project directory.
2. Install the required dependencies.
3. Run the script using:
   robocorp run
Output
Screenshots of email confirmations will be saved in the output/ directory.
Feel free to contribute to this repository by submitting issues or creating pull requests.

```Python

from robocorp.tasks import task
from robocorp import browser
from RPA.HTTP import HTTP
from config import EMAIL, EMAIL_PASSWORD
import time #time.sleep(8) in colllecting results

@task
def handle_form_submission():
    """
    Downloads sample file, fills form, and verifies email confirmation for all entries
    """
    browser.configure(
        slowmo=50,
    )
    download_sample_file()
    data_lines = read_sample_file()
    
    # Fill all forms first
    for data in data_lines:
        open_website()
        select_service(data[0])
        fill_contact_form(data)
        
    
    # Login after all forms are filled
    log_in()
    collect_results()

def download_sample_file():
    """Downloads the sample file from RPA challenge"""
    http = HTTP()
    http.download("https://raw.githubusercontent.com/fspacheco/rpa-challenge/refs/heads/main/assets/contact-form.txt")

def read_sample_file():
    """Reads the downloaded file and returns all lines"""
    data_lines = []
    with open("contact-form.txt", "r") as file:
        for line in file:
            if ',' in line:
                data_lines.append(line.strip().split(','))
    return data_lines

def process_all_entries():
    """Process each entry in the file"""
    data_lines = read_sample_file()
    for data in data_lines:
        open_website()
        select_service(data[0])
        fill_contact_form(data)
        log_in()
        collect_results()

def open_website():
    """Navigates to the given URL"""
    browser.goto("https://wehost.fi/index.php/send-your-form/")
    page = browser.page()

def select_service(service_type):
    """Select the service"""
    page = browser.page()
    
    service_mapping = {
        "maintenance": "Maintenance",
        "program_mod": "Program modification",
        "new_program": "New program",
        "robot_upgrade": "Robot upgrade"
    }
    
    if service_type in service_mapping:
        page.locator(f"text={service_mapping[service_type]}").click()
    else:
        raise ValueError("Invalid service type")

def fill_contact_form(data):
    """Fills and submits the contact form"""
    page = browser.page()
    
    # Fill additional notes
    page.locator("textarea[name='textarea-1']").wait_for(state="visible")
    page.locator("textarea[name='textarea-1']").fill(data[1])

    # Fill name fields
    page.locator("input[id^='forminator-field-first-name-1_']").fill(data[3])
    page.locator("input[id^='forminator-field-middle-name-1_']").fill(data[4])
    page.locator("input[id^='forminator-field-last-name-1_']").fill(data[5])
    
    # Fill contact information
    page.locator("input[name='email-1']").wait_for(state="visible")
    page.locator("input[name='email-1']").fill(data[6])
    
    page.locator("input[name='phone-1']").wait_for(state="visible")
    page.locator("input[name='phone-1']").fill(data[7])
    
    # Fill date
    page.locator("input[name='date-1']").fill(f"{data[8]}/2024")
    
    # Fill payment
    page.locator("input[name='currency-1']").wait_for(state="visible")
    page.locator("input[name='currency-1']").fill(data[9])
    
    # Submit form
    page.locator("button.forminator-button-submit").click()
    
def log_in():
    """Fills in the login form and clicks the 'Log in' button"""
    browser.goto("https://login.live.com/login.srf?wa=wsignin1.0&rpsnv=167&ct=1733769510&rver=7.5.2211.0&wp=MBI_SSL&wreply=https%3a%2f%2foutlook.live.com%2fowa%2f%3fnlp%3d1%26cobrandid%3dab0455a0-8d03-46b9-b18b-df2f57b9e44c%26deeplink%3dowa%252f0%252f%253fstate%253d1%2526redirectTo%253daHR0cHM6Ly9vdXRsb29rLmxpdmUuY29tL21haWwvMC8%26RpsCsrfState%3d8a6ca075-99cd-e580-3461-b198e401d2de&id=292841&aadredir=1&CBCXT=out&lw=1&fl=dob%2cflname%2cwld&cobrandid=ab0455a0-8d03-46b9-b18b-df2f57b9e44c")
    page = browser.page()
    #import the email from another file 
    page.locator("input[id='i0116']").wait_for(state="visible")
    page.locator("input[id='i0116']").fill(EMAIL)
    page.locator("#idSIButton9").click()
    #import the password from another file
    page.locator("input[id='i0118']").wait_for(state="visible")
    page.locator("input[id='i0118']").fill(EMAIL_PASSWORD)
    page.locator("#idSIButton9").click()

def collect_results():
    """Take a screenshot of the page"""
    page = browser.page()
    page.locator("#acceptButton").click()
    time.sleep(9) #this is the time the system should wait before taking a screenshot of the email.
    
    page.screenshot(path="output/form_confirmation.png")
    
```
Done

Link to see how it works: https://youtu.be/5sP54LrLflI?si=kz-eHd8L8Qhdj1G8
