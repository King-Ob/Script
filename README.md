# Script

import os
import time
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

# Set up webdriver for Chrome
driver = webdriver.Chrome()
driver.maximize_window()

# Login to Box.com
driver.get("https://www.box.com/login")
username = driver.find_element_by_id("login-email")
password = driver.find_element_by_id("login-password")
username.send_keys("your_username")
password.send_keys("your_password")
password.send_keys(Keys.RETURN)
time.sleep(5)

# Navigate to the folder to download
driver.get("https://app.box.com/folder/folder_id")
time.sleep(5)

# Create a directory to store downloaded files and folders
if not os.path.exists('downloads'):
    os.makedirs('downloads')

# Find all files and folders in the current directory
elements = driver.find_elements_by_css_selector('li.item')
for element in elements:
    # Check if the item is a folder or a file
    icon = element.find_element_by_css_selector('span.item_icon')
    if 'folder' in icon.get_attribute('class'):
        # If it's a folder, navigate to the folder and download its contents
        link = element.find_element_by_css_selector('a.item_link')
        link.click()
        time.sleep(2)
        driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
        time.sleep(2)
        download_all_button = driver.find_element_by_css_selector('button[title="Download All"]')
        download_all_button.click()
        time.sleep(5)
    else:
        # If it's a file, check if it has been updated and download the file
        link = element.find_element_by_css_selector('a.item_link')
        link.click()
        time.sleep(2)
        # Wait for the "Download" button to become clickable
        download_button = WebDriverWait(driver, 10).until(EC.element_to_be_clickable((By.CSS_SELECTOR, 'button[title="Download"]')))
        # Get the download link and last modified time
        download_link = download_button.get_attribute('href')
        modified_time = element.find_element_by_css_selector('span.modification-time').get_attribute('data-created')
        # Check if the file has been updated
        if not os.path.exists(f"downloads/{element.text}"):
            # If the file does not exist in the downloads folder, download it
            download_button.click()
            time.sleep(5)
        else:
            # If the file already exists in the downloads folder, check if it has been updated
            local_modified_time = os.path.getmtime(f"downloads/{element.text}")
            if local_modified_time < int(modified_time):
                # If the file has been updated, download it
                download_button.click()
                time.sleep(5)
                os.remove(f"downloads/{element.text}")
        # Rename the file to match the original filename
        downloaded_file = max([f"downloads/{f}" for f in os.listdir('downloads')], key=os.path.getctime)
        os.rename(downloaded_file, f"downloads/{element.text}")

# Close the webdriver
driver.quit()
