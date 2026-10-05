import requests
from bs4 import BeautifulSoup
import pandas as pd
import time

# Website from which we will collect the data
base_url = "https://books.toscrape.com/catalogue/page-{}.html"

# This list will store all the books
all_books = []

print("Starting web scraping...\n")

# Scrape the first 5 pages
for page in range(1, 6):

    url = base_url.format(page)

    print(f"Scraping page {page}...")

    # Send request to the website
    response = requests.get(url)

    # Check whether the website responded successfully
    if response.status_code != 200:
        print(f"Could not open page {page}")
        continue

    # Read the HTML content
    soup = BeautifulSoup(response.text, "html.parser")

    # Find all book containers
    books = soup.find_all("article", class_="product_pod")

    # Extract information from each book
    for book in books:

        # Book title
        title = book.find("h3").find("a")["title"]

        # Book price
        price = book.find("p", class_="price_color").text.strip()

        # Availability
        availability = book.find(
            "p", class_="instock availability"
        ).text.strip()

        # Rating
        rating_tag = book.find("p", class_="star-rating")

        # The rating is stored inside the class name
        rating = rating_tag.get("class")[1]

        # Convert rating words into numbers
        rating_values = {
            "One": 1,
            "Two": 2,
            "Three": 3,
            "Four": 4,
            "Five": 5
        }

        rating = rating_values.get(rating, 0)

        # Book page URL
        book_link = book.find("h3").find("a")["href"]

        # Make the URL complete
        book_url = "https://books.toscrape.com/catalogue/" + book_link

        # Store the information
        all_books.append({
            "Book Title": title,
            "Price": price,
            "Availability": availability,
            "Rating": rating,
            "Book URL": book_url
        })

    # Small delay between requests
    time.sleep(1)


# Convert collected data into a DataFrame
df = pd.DataFrame(all_books)

# Remove duplicate books if any
df.drop_duplicates(inplace=True)

# Save the dataset
df.to_csv("books_dataset.csv", index=False)

print("\nScraping completed successfully!")
print(f"Total books collected: {len(df)}")

print("\nFirst 10 records:")
print(df.head(10))

print("\nDataset saved as: books_dataset.csv")
