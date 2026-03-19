# tmdb-eda-assignment
import requests
import pandas as pd
import sqlite3

# --- CONFIGURATION ---
# Replace 'YOUR_API_KEY_HERE' with your actual TMDB API Key
API_KEY = 'YOUR_API_KEY_HERE' 
BASE_URL = "https://api.themoviedb.org/3/discover/movie"

# --- TASK 1: Build the Pipeline ---

def fetch_movie_data(api_key):
    params = {
        'api_key': api_key,
        'language': 'en-US',
        'sort_by': 'popularity.desc',
        'include_adult': 'false',
        'page': 1
    }
    response = requests.get(BASE_URL, params=params)
    
    if response.status_code == 200:
        return response.json()['results']
    else:
        print(f"Error: {response.status_code}")
        return []

# Fetch data (at least 20 movies)
movies_list = fetch_movie_data(API_KEY)
df_raw = pd.DataFrame(movies_list)

# Save to SQLite
conn = sqlite3.connect('tmdb_data.db')
df_raw.to_sql('movies', conn, if_exists='replace', index=False)
print("Data successfully saved to SQLite database.")

# --- TASK 2: Perform EDA ---

# Load data back from SQLite
df = pd.read_sql_query("SELECT * FROM movies", conn)

# 1. Display the first 5 rows
print("\n--- First 5 Rows ---")
print(df.head())

# 2. Summary statistics
print("\n--- Summary Statistics ---")
print(df.describe())

# 3. Count movies per genre 
# Note: TMDB returns genre_ids as a list. We will explode them to count.
print("\n--- Count of Movies per Genre ID ---")
genre_counts = df.explode('genre_ids')['genre_ids'].value_counts()
print(genre_counts)

# 4. Identify columns with missing values
print("\n--- Missing Values Per Column ---")
print(df.isnull().sum())

# Close connection
conn.close()
