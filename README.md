# Web Scraping with Python

## Introduction

This project demonstrates web scraping using Python, focusing on two distinct sources: YouTube and IMDb. Using Selenium and BeautifulSoup, data is extracted from these websites and saved into separate CSV files for further analysis.

## Libraries Used

### Selenium

Selenium is a powerful tool for controlling web browsers through programs and performing browser automation. It is widely used for testing web applications but also serves as an excellent tool for web scraping, especially when dealing with dynamic content.

### BeautifulSoup

BeautifulSoup is a Python library for parsing HTML and XML documents. It creates parse trees from page source code that can be used to extract data easily. It is particularly useful for web scraping static content.

## Installation

To get started, install the necessary libraries:

```bash
!pip install bs4 selenium requests pandas
!apt install chromium-chromedriver
```

## YouTube Scraping with Selenium

The YouTube scraping script extracts video details from the [Unfold Data Science YouTube channel](https://www.youtube.com/@UnfoldDataScience/videos).

### Code Explanation

1. **Setup Selenium and Chrome WebDriver**:
   ```python
   from selenium import webdriver
   from selenium.webdriver.common.by import By
   from selenium.webdriver.support.ui import WebDriverWait
   from selenium.webdriver.support import expected_conditions as EC
   import pandas as pd

   options = webdriver.ChromeOptions()
   options.add_argument('--ignore-certificate-errors')
   options.add_argument('--incognito')
   options.add_argument('--headless')
   options.add_argument('--no-sandbox')
   options.add_argument('--disable-dev-shm-usage')

   driver = webdriver.Chrome(options=options)
   ```

2. **Navigate to the YouTube Channel**:
   ```python
   urlPath = 'https://www.youtube.com/@UnfoldDataScience/videos'
   driver.get(urlPath)
   ```

3. **Extract Video Information**:
   ```python
   videos = driver.find_elements(By.CLASS_NAME, "style-scope ytd-rich-item-renderer")

   titles, views, dates, likesData, commentsData = [], [], [], [], []
   wait = WebDriverWait(driver, 15)

   for video in videos:
       titles.append(video.find_element(By.XPATH, './/*[@id="video-title"]').text)
       views.append(video.find_element(By.XPATH, './/*[@id="metadata-line"]/span[1]').text)
       dates.append(video.find_element(By.XPATH, './/*[@id="metadata-line"]/span[2]').text)
       
       # Navigate to video page to get likes and comments
       video.find_element(By.XPATH, '//*[@id="video-title-link"]').click()
       try:
           likes = wait.until(EC.presence_of_element_located((By.XPATH, '//*[@id="segmented-like-button"]/ytd-toggle-button-renderer/yt-button-shape/button'))).text
           likesData.append(likes)
           comments = wait.until(EC.presence_of_element_located((By.XPATH, '//div[@id="title"]//*[@id="count"]//span[1]'))).text
           commentsData.append(comments)
       except:
           likesData.append('')
           commentsData.append('')
       driver.back()

   driver.quit()
   ```

4. **Save Data to CSV**:
   ```python
   df = pd.DataFrame({
       'title': titles,
       'views': views,
       'dates': dates,
       'likes': likesData,
       'comments': commentsData
   })
   df.to_csv('Youtube.csv', index=False)
   ```

### Analysis

Load the CSV file and perform various analyses, such as calculating average views, finding the highest likes-to-views ratio, and plotting the correlation between likes and comments.

```python
df = pd.read_csv('Youtube.csv')
# Perform analyses here...
```

## IMDb Scraping with BeautifulSoup

The IMDb scraping script extracts details about the top-rated movies from the [IMDb Top 1000](https://www.imdb.com/search/title/?groups=top_1000&sort=user_rating,desc&count=100&start=1&ref_=adv_nxt).

### Code Explanation

1. **Setup and Send Request**:
   ```python
   import requests
   from bs4 import BeautifulSoup
   import pandas as pd

   URL = 'https://www.imdb.com/search/title/?groups=top_1000&sort=user_rating,desc&count=100&start=1&ref_=adv_nxt'
   headers = {"Accept-Language": "en-US,en;q=0.8"}
   response = requests.get(URL, headers=headers)
   soup = BeautifulSoup(response.content, 'html.parser')
   ```

2. **Extract Movie Information**:
   ```python
   MovieTitle, ReleaseYear, IMDbRating, Genre, Director = [], [], [], [], []

   movies = soup.find_all('div', class_='lister-item mode-advanced')

   for movie in movies:
       title = movie.find('h3', class_='lister-item-header').find('a').text.strip()
       year = movie.find('span', class_='lister-item-year').text.strip('()')
       genre = movie.find('span', class_='genre').text.strip()
       rating = movie.find('div', class_='inline-block ratings-imdb-rating').text.strip()
       director = movie.find('p', class_='').find('a').text

       MovieTitle.append(title)
       ReleaseYear.append(year)
       IMDbRating.append(rating)
       Genre.append(genre)
       Director.append(director)
   ```

3. **Save Data to CSV**:
   ```python
   df = pd.DataFrame({
       'Movie Titles': MovieTitle,
       'Release Year': ReleaseYear,
       'IMDb Rating': IMDbRating,
       'Directors': Director,
       'Genre': Genre
   })
   df.to_csv('IMDB.csv', index=False)
   ```

### Analysis

Load the CSV file and perform various analyses, such as calculating average IMDb rating, finding the most common genre, and identifying the director with the highest average rating.

```python
df = pd.read_csv('IMDB.csv')
# Perform analyses here...
```

## Output Files

1. **YouTube Data**: [Youtube.csv](Youtube.csv)
2. **IMDb Data**: [IMDB.csv](IMDB.csv)

## Conclusion

This project showcases the use of Selenium for dynamic content scraping and BeautifulSoup for static content scraping, providing a comprehensive guide to web scraping in Python. The collected data is stored in CSV files and can be further analyzed to extract meaningful insights.
