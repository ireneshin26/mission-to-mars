# Mission to Mars

## **Overview**
The purpose of this project is to build a Web App that will scrape data about the mission to Mars from all over the web and display it in one central location every time it scrapes new data with a click of a button. The extracted data is stored in a NoSQL MongoDB database. Using flask, an HTML page is created with the data findings.

## **Resources**
The following websites were scraped:
* Mars News: https://redplanetscience.com/
* Featured Image: https://spaceimages-mars.com
* Mars Facts: https://galaxyfacts-mars.com
* Hemisphere Images: https://marshemispheres.com/

## **Results** 

### Deliverable 1: Scrape Full-Resolution Mars Hemisphere Images and Titles 

  * Cerberus: <img height="350" alt="Cerberus" src="https://user-images.githubusercontent.com/110875578/197118123-87b82912-8ea8-4bb6-a852-dbc0da55dcd2.jpeg">
  * Schiaparelli: <img height="350" alt="Schiaparelli" src="https://user-images.githubusercontent.com/110875578/197118126-5fc5cece-c13a-4e26-9bfe-d74ea84ad280.jpeg">
  * Syrtis Major: <img height="350" alt="Syrtis Major" src="https://user-images.githubusercontent.com/110875578/197118130-1b21343b-7022-4364-859f-13a204d46a41.jpeg">
  * Valles Marineris: <img height="350" alt="Valles Marineris" src="https://user-images.githubusercontent.com/110875578/197118137-8b87bd43-293b-4baf-8286-eea25e2190bb.jpeg">

### Deliverable 2: Update the Web App with Mars’s Hemisphere Images and Titles
* The scraping.py file contains code that retrieves the full-resolution image URL and title for each hemisphere image:
  ```
  # Import Splinter, BeautifulSoup, and Pandas
  from bs4 import BeautifulSoup as soup
  import pandas as pd
  from splinter import Browser
  from selenium import webdriver
  from webdriver_manager.chrome import ChromeDriverManager
  import datetime as dt

  def scrape_all():
      # Initiate headless driver for deployment
      executable_path = {'executable_path': ChromeDriverManager().install()}
      browser = Browser('chrome', **executable_path, headless=True)
      news_title, news_paragraph = mars_news(browser)

     # Run all scraping functions and store results in dictionary
      data = {
          "news_title": news_title,
          "news_paragraph": news_paragraph,
          "featured_image": featured_image(browser),
          "facts": mars_facts(),
          "last_modified": dt.datetime.now(),
          "hemispheres": hemispheres(browser)}

      # Stop webdriver and return data
      browser.quit()
      return data

  def mars_news(browser):

      # Scrape Mars News
      # Visit the mars nasa news site
      url = 'https://redplanetscience.com/'
      browser.visit(url)

      # Optional delay for loading the page
      browser.is_element_present_by_css('div.list_text', wait_time=1)

      # Convert the browser html to a soup object and then quit the browser
      html = browser.html
      news_soup = soup(html, 'html.parser')

      # Add try/except for error handling
      try:
          slide_elem = news_soup.select_one('div.list_text')
          # Use the parent element to find the first 'a' tag and save it as 'news_title'
          news_title = slide_elem.find('div', class_='content_title').get_text()
          # Use the parent element to find the paragraph text
          news_p = slide_elem.find('div', class_='article_teaser_body').get_text()

      except AttributeError:
          return None, None

      return news_title, news_p

  def featured_image(browser):
      # Visit URL
      url = 'https://spaceimages-mars.com'
      browser.visit(url)

      # Find and click the full image button
      full_image_elem = browser.find_by_tag('button')[1]
      full_image_elem.click()

      # Parse the resulting html with soup
      html = browser.html
      img_soup = soup(html, 'html.parser')

      # Add try/except for error handling
      try:
          # Find the relative image url
          img_url_rel = img_soup.find('img', class_='fancybox-image').get('src')

      except AttributeError:
          return None

      # Use the base url to create an absolute url
      img_url = f'https://spaceimages-mars.com/{img_url_rel}'

      return img_url

  def mars_facts():
      # Add try/except for error handling
      try:
          # Use 'read_html' to scrape the facts table into a dataframe
          df = pd.read_html('https://galaxyfacts-mars.com')[0]

      except BaseException:
          return None

      # Assign columns and set index of dataframe
      df.columns=['Description', 'Mars', 'Earth']
      df.set_index('Description', inplace=True)

      # Convert dataframe into HTML format, add bootstrap
      return df.to_html()

  # Scrape High-Resolution Mars’ Hemisphere Images and Titles

  def hemispheres(browser):
      # Visit url
      url = 'https://marshemispheres.com/'
      browser.visit(url)
      # collect four hemisphere images and titles
      hemisphere_image_urls = []
      for i in range(4):
          hemispheres = {}
          browser.find_by_css('a.product-item h3')[i].click()
          element = browser.links.find_by_text('Sample').first
          img_url = element['href']
          title = browser.find_by_css("h2.title").text
          hemispheres["img_url"] = img_url
          hemispheres["title"] = title
          hemisphere_image_urls.append(hemispheres)
          browser.back()
      return hemisphere_image_urls


  if __name__ == "__main__":
      # If running as script, print scraped data
      print(scrape_all())
  ```
* The Mongo database is updated to contain the full-resolution image URL and title for each hemisphere image (10 pt)
  * ![mongodb_hemispheres](https://user-images.githubusercontent.com/110875578/197118074-bb915f6f-3e81-4d7f-b598-4e7d2b563e18.jpg)

* Scraped desktop index.html:
  * ![html_desktop](https://user-images.githubusercontent.com/110875578/197116711-ebca1670-dfde-4125-a04a-6d207c3f7917.jpg)


## Deliverable 3: Add Bootstrap 3 Components
* The webpage is mobile-responsive
  * Per deliverable requirements, we can confirm that the webpage is mobile-responsive: 
  <img height="500" alt="Mobile1" src="https://user-images.githubusercontent.com/110875578/197116824-2a6714eb-8931-4449-bac3-51e9de35c2b6.jpg">

  <img height="500" alt="Mobile 2" src="https://user-images.githubusercontent.com/110875578/197116828-592608d1-fab5-49b2-92b6-55b0d1d65407.jpg">

  <img height="500" alt="Mobile 3" src="https://user-images.githubusercontent.com/110875578/197116830-2db022c7-9180-4c5f-b782-1ea469cbcccc.jpg">

  <img height="500" alt="Mobile 4" src="https://user-images.githubusercontent.com/110875578/197116836-cc17a277-29a4-4457-8421-8ba9b2ec8fce.jpg">

  <img height="500" alt="Mobile 5" src="https://user-images.githubusercontent.com/110875578/197116837-01fa8389-7f91-4767-ae08-804d6c3d089c.jpg">



