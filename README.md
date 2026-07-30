

# ME204 Final Project: Movie Runtime Trends by Genre (2015–2025)

| GitHub username         | LSE ID       |
| ----------------------- | ------------ |
| `domenicapalominoh-lang`| `[LSE ID]`   |

## Overview

This project asks: **how has the average runtime of top-grossing movies changed across 2015–2025, and does that trend differ by genre?** For each year, I collect the highest-revenue films from The Movie Database (TMDB), fetch each film's full details, and compare runtimes over time and across genres.

## Data source

- **The Movie Database (TMDB) API** — <https://www.themoviedb.org/>
  - `GET /discover/movie` — top films per year, sorted by `revenue.desc` (5 pages ≈ 100 films/year, 2015–2025).
  - `GET /movie/{movie_id}` — full details (runtime, genres, budget, revenue, dates) for each film found above.

All data is collected by code in NB01. No static or manually downloaded files are used.

## How to reproduce

**1. Get a TMDB API token.** Create a free account at TMDB, then go to Settings → API and copy your **API Read Access Token** (v4 bearer token).

**2. Create a `.env` file** in the project root with:

```
API_ACCESS_TOKEN=your_token_here
```

This file is git-ignored and must never be committed. See `.env.example` for the variable name.

**3. Install dependencies:**

```
pip install requests python-dotenv pandas plotly
```

**4. Run the notebooks in order** from inside the `notebooks/` folder (paths are relative, e.g. `../data/raw/`):

| Order | File | Reads | Does | Writes |
| ----- | ---- | ----- | ---- | ------ |
| 1 | `NB01-Data-Collection.ipynb` | TMDB API | Fetches top-revenue films per year (2015–2025), then fetches full details for each film | `data/raw/2015.json … 2025.json`, `data/raw/movie_details.json` |
| 2 | `NB02-Data-Transformation.ipynb` | `data/raw/movie_details.json` | Normalises movie details into a tidy table, drops films with `runtime = 0`, splits out a genre table and a movie–genre link table, loads all three into SQLite | `data/processed/movies.csv`, `data/processed/genres.csv`, `data/processed/genres_movie.csv`, `data/movies.db` |
| 3 | `NB03-Data-Analysis.ipynb` | `data/movies.db` | Explores runtime by year and by genre with `plotly.express` (histograms, box plots, line charts) | charts for the public page |

## Repository structure

```
me204-final-project/
├── README.md
├── .gitignore
├── .env.example
├── data/
│   ├── raw/              # per-year JSON + movie_details.json from TMDB
│   ├── processed/        # movies.csv, genres.csv, genres_movie.csv
│   └── movies.db         # SQLite: movies, genres, movie_genres tables
├── notebooks/
│   ├── NB01-Data-Collection.ipynb
│   ├── NB02-Data-Transformation.ipynb
│   └── NB03-Data-Analysis.ipynb
└── docs/
    └── index.html
```

## Tidy tables (what one row means)

- **`movies.csv`** — one row per film: `id`, `title`, `original_title`, `release_date`, `budget`, `revenue`, `popularity`, `vote_average`, `vote_count`, `runtime`, `original_language`, `year`.
- **`genres.csv`** — one row per unique genre: `genre_id`, `genre_name`.
- **`genres_movie.csv`** — one row per film–genre pair, linking `genre_id` to movie `id`.

The same three tables exist in `data/movies.db` (`movies`, `genres`, `movie_genres`), which NB03 reads from.