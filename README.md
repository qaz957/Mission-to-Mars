# Mission-to-Mars
We used web scraping techniques to pull data and images from websites to create a comprehensive site for NASA's planned mission to Mars.

### Resources
- Python
- Jupyter Notebook
- HTML
- Splinter
- Flask
- Beautiful Soup
- Chrome WebDriver
- Mongo DB

## Method

### Python Code

      # Import Splinter, BeautifulSoup, and Pandas
      from splinter import Browser
      from bs4 import BeautifulSoup as soup
      import pandas as pd
      from webdriver_manager.chrome import ChromeDriverManager


      # Set the executable path and initialize Splinter
      executable_path = {'executable_path': ChromeDriverManager().install()}
      browser = Browser('chrome', **executable_path, headless=False)

      # ### Visit the NASA Mars News Site
      # Visit the mars nasa news site
      url = 'https://redplanetscience.com/'
      browser.visit(url)

      # Optional delay for loading the page
      browser.is_element_present_by_css('div.list_text', wait_time=1)

      # Convert the browser html to a soup object and then quit the browser
      html = browser.html
      news_soup = soup(html, 'html.parser')

      slide_elem = news_soup.select_one('div.list_text')

      slide_elem.find('div', class_='content_title')

      # Use the parent element to find the first a tag and save it as `news_title`
      news_title = slide_elem.find('div', class_='content_title').get_text()
      news_title

      # Use the parent element to find the paragraph text
      news_p = slide_elem.find('div', class_='article_teaser_body').get_text()
      news_p

      # ### JPL Space Images Featured Image
      # Visit URL
      url = 'https://spaceimages-mars.com'
      browser.visit(url)

      # Find and click the full image button
      full_image_elem = browser.find_by_tag('button')[1]
      full_image_elem.click()

      # Parse the resulting html with soup
      html = browser.html
      img_soup = soup(html, 'html.parser')

      # find the relative image url
      img_url_rel = img_soup.find('img', class_='fancybox-image').get('src')

      # Use the base url to create an absolute url
      img_url = f'https://spaceimages-mars.com/{img_url_rel}'

      # ### Mars Facts
      df = pd.read_html('https://galaxyfacts-mars.com')[0]
      df.columns=['Description', 'Mars', 'Earth']
      df.set_index('Description', inplace=True)
      df.to_html()

      # # D1: Scrape High-Resolution Mars’ Hemisphere Images and Titles
      # ### Hemispheres

      # 1. Use browser to visit the URL 
      url = 'https://marshemispheres.com/'

      browser.visit(url)

      links = browser.links.find_by_partial_text('Hemisphere')
      len(links)

      # 2. Create a list to hold the images and titles.
      hemisphere_image_urls = []

      links_list = browser.links.find_by_partial_text('Hemisphere')

      # 3. Write code to retrieve the image urls and titles for each hemisphere.
      # For loop to iterate through the links
      for link in range(len(links_list)):

          # Browse through each link
          links_list[link].click()

          # Parse the HTML
          html = browser.html
          img_soup = soup(html,'html.parser')

          # Title
          title = img_soup.find('h2', class_='title').text

          # Image
          img_url = img_soup.find('li').a.get('href')

          # Add into dictionary
          hemis = {}
          hemis['img_url'] = f'https://marshemispheres.com/{img_url}'
          hemis['title'] = title
          hemisphere_image_urls.append(hemis)

          # Go back a page
          browser.back()

      print(hemisphere_image_urls)

      # 4. Print the list that holds the dictionary of each image url and title.
      hemisphere_image_urls

      # 5. Quit the browser
      browser.quit()
      
### Flask App

      from flask import Flask, render_template, redirect, url_for
      from flask_pymongo import PyMongo
      import scraping

      app = Flask(__name__)

      # Use flask_pymongo to set up mongo connection
      app.config["MONGO_URI"] = "mongodb://localhost:27017/mars_app"
      mongo = PyMongo(app)

      @app.route("/")
      def index():
         mars = mongo.db.mars.find_one()
         return render_template("index.html", mars=mars)

      @app.route("/scrape")
      def scrape():
         mars = mongo.db.mars
         mars_data = scraping.scrape_all()
         mars.update_one({}, {"$set":mars_data}, upsert=True)
         return redirect('/', code=302)

      if __name__ == "__main__":
          app.run()
