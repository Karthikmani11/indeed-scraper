import os
import csv
from datetime import datetime
from selenium import webdriver
from selenium.webdriver.edge.service import Service
from selenium.webdriver.edge.options import Options
from selenium.webdriver.common.by import By
from selenium.common.exceptions import NoSuchElementException, TimeoutException
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import logging

# Configure logging
logging.basicConfig(filename='scraper_errors.log', level=logging.ERROR)

def get_url(position, location):
    """Generate URL from position and location"""
    template = 'https://in.indeed.com/jobs?q={}&l={}&from=searchOnHP'
    return template.format(position.replace(' ', '%20'), location.replace(' ', '%20'))

def save_data_to_file(records):
    """Save data to CSV file"""
    file_exists = os.path.isfile('results.csv')
    with open('results.csv', 'a', newline='', encoding='utf-8') as f:
        writer = csv.writer(f)
        if not file_exists:  # Write header only if file is new
            writer.writerow(['JobTitle', 'Company', 'Location', 'PostDate', 'ExtractDate', 'Summary', 'JobUrl'])
        writer.writerows(records)

def get_record(card):
    """Extract job data from a single card"""
    job_title = card.find_element(By.CSS_SELECTOR, 'h2.jobTitle').text
    company = card.find_element(By.CSS_SELECTOR, '.companyName').text
    location = card.find_element(By.CSS_SELECTOR, '.companyLocation').text
    post_date = card.find_element(By.CSS_SELECTOR, '.date').text
    extract_date = datetime.today().strftime('%Y-%m-%d')
    
    # Summary extraction
    try:
        summary = card.find_element(By.CSS_SELECTOR, '.summary').text
    except NoSuchElementException:
        summary = "No summary available"
    
    job_url = card.find_element(By.CSS_SELECTOR, 'h2.jobTitle a').get_attribute('href')
    return (job_title, company, location, post_date, extract_date, summary, job_url)

def main(position, location, pages=1):
    """Run the main program routine"""
    scraped_jobs = []
    url = get_url(position, location)

    # Setup WebDriver
    service = Service(r"C:\Users\gokul karthik\Music\New folder\msedgedriver.exe")  # Update path as needed
    options = Options()
    options.use_chromium = True
    driver = webdriver.Edge(service=service, options=options)
    driver.implicitly_wait(15)  # Increased wait time

    try:
        for page in range(pages):
            driver.get(f"{url}&start={page * 10}")
            WebDriverWait(driver, 10).until(EC.presence_of_element_located((By.CSS_SELECTOR, 'a.tapItem')))
            
            cards = driver.find_elements(By.CSS_SELECTOR, 'a.tapItem')
            if not cards:
                print(f"No job cards found on page {page + 1}.")
                continue  # Skip to next page if no cards are found
            
            for card in cards:
                record = get_record(card)
                scraped_jobs.append(record)
    
    except (NoSuchElementException, TimeoutException) as e:
        logging.error(f"Error extracting job data: {e}")

    finally:
        driver.quit()

    # Save records
    save_data_to_file(scraped_jobs)
    print(f"Scraped {len(scraped_jobs)} job(s) and saved to results.csv.")

# Example usage
if __name__ == "__main__":
    main('python developer', 'chennai', pages=2)  # Scraping 2 pages as an example
