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
