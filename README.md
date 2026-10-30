# Book Reviews

A book review website built with Flask and PostgreSQL. Users create an account, search a catalog of 5,000 books by ISBN, title or author, and leave a 1 to 5 star rating with a written review. The app also has a small JSON API for each book. It was my Project 1 for Harvard's CS50 Web Programming with Python and JavaScript.

![Signing up, logging in, searching for "asimov" and by part of an ISBN, then viewing the JSON API for "I, Robot" after a review was added](docs/demo.gif)

The book detail page no longer loads, because it calls the Goodreads API, which has been shut down. The demo shows the parts that still work. See [Goodreads API shutdown](#goodreads-api-shutdown) below.

## Features

- **Accounts:** sign up with a username and password (asked twice to confirm). Passwords are stored as hashes with Werkzeug, and usernames are saved in title case, so "tenzin" and "Tenzin" are the same account.
- **Book search:** search by any part of an ISBN, title or author. Title and author matching ignores case.
- **Book page:** shows the title, author, year and ISBN, a cover image from Open Library, Goodreads rating data and all reviews, each shown with its star rating.
- **Reviews:** each user can review a book once, with a 1 to 5 rating and text. A second review of the same book shows a "You already reviewed this book!" message.
- **JSON API:** `GET /api/<isbn>` returns the book's title, author, year, ISBN, review count and average score, or a 404 error if the ISBN isn't in the database.
- **CSV import:** `import.py` loads the 5,000 books in `books.csv` into the database.

## How it works

1. Flask handles the pages, and SQLAlchemy runs raw SQL queries against PostgreSQL. Sessions are stored on the server's filesystem with Flask-Session.
2. `POST /signup` checks that the passwords match and the username is free, then saves the user. `POST /` checks the password hash and stores the user's id in the session.
3. `POST /search` runs a `LIKE` query over the `isbn`, `title` and `author` columns and lists the matching books.
4. `GET /book/<isbn>` reads the book and its reviews from the database, asks the Goodreads API for that ISBN's rating data, and renders the page. `POST /review/<isbn>` saves a review.
5. `GET /api/<isbn>` counts and averages the book's reviews in SQL and returns JSON.

## Project structure

```
application.py      Flask routes: login, sign up, search, book page, reviews, JSON API, log out
helpers.py          login_required decorator
import.py           Loads books.csv into the books table
books.csv           5,000 books (ISBN, title, author, year)
requirements.txt    Python packages (unpinned)
templates/          layout, login (index), sign up, search, results, book and error pages
static/             style.css and the Goodreads logo
docs/demo.gif       Demo recording
```

## Running it locally

`requirements.txt` doesn't pin versions, and the code uses SQLAlchemy 1.3-style raw SQL strings, so install 2020-era versions. I tested this with **Python 3.8** and **PostgreSQL 16**:

```bash
git clone https://github.com/sangyetenphel/cs50-books.git
cd cs50-books
python3.8 -m venv venv
source venv/bin/activate
pip install "Flask==1.1.2" "Werkzeug==1.0.1" "Jinja2==2.11.3" "MarkupSafe==1.1.1" \
    "itsdangerous==1.1.0" "click==7.1.2" "Flask-Session==0.3.2" \
    "SQLAlchemy==1.3.24" "psycopg2-binary==2.8.6" "requests==2.24.0"
```

Create a PostgreSQL database and point `DATABASE_URL` at it:

```bash
export DATABASE_URL=postgresql://USER:PASSWORD@localhost/DBNAME
```

The repository doesn't include a schema file. These tables match the queries in `application.py` and `import.py`:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR NOT NULL UNIQUE,
    password VARCHAR NOT NULL
);
CREATE TABLE books (
    isbn VARCHAR PRIMARY KEY,
    title VARCHAR NOT NULL,
    author VARCHAR NOT NULL,
    year INTEGER NOT NULL
);
CREATE TABLE reviews (
    id SERIAL PRIMARY KEY,
    rating INTEGER NOT NULL,
    review TEXT,
    isbn VARCHAR NOT NULL REFERENCES books (isbn),
    user_id INTEGER NOT NULL REFERENCES users (id)
);
```

Save that as `schema.sql`, then create the tables, import the books and start the server:

```bash
psql "$DATABASE_URL" -f schema.sql
python import.py
export FLASK_APP=application.py
flask run
```

Then open http://127.0.0.1:5000. The pages load Bootstrap and Font Awesome from public CDNs.

## Goodreads API shutdown

The book page called the Goodreads `review_counts` API to show each book's average rating and number of ratings. Goodreads shut down its public API in December 2020, and the app doesn't handle the failed request, so `GET /book/<isbn>` now returns a 500 Internal Server Error. Because the review form is on that page, the redirect after posting a review also ends on the error, although the review itself is saved first. The demo above added a review by sending the same form data to `POST /review/<isbn>`, then showed it through the JSON API.

Sign up, log in, search, saving reviews and the JSON API all work. The JSON API also returns a 500 error for books that have no reviews yet, because it tries to round an empty average.

## Built with

- **Backend:** Python, Flask, Flask-Session, SQLAlchemy and Werkzeug
- **Database:** PostgreSQL
- **Frontend:** HTML, Jinja templates, Bootstrap 4 and Font Awesome
- **External data:** [Open Library Covers API](https://openlibrary.org/dev/docs/api/covers) for cover images, and the Goodreads API (now shut down) for rating data

## Course credit

Built for Project 1 of [CS50's Web Programming with Python and JavaScript](https://cs50.harvard.edu/web/) (Harvard University, 2020 edition). The original course submission video is on [YouTube](https://youtu.be/QHyDgRNN70w).

Created by Sangye Tenphel, 2020.
