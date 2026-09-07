# Movie Recommendation System

A movie discovery and recommendation application built with Streamlit, FastAPI, TMDB, and a local TF-IDF similarity model.

The application lets users search for movies, browse popular and trending titles, open detailed movie pages, and view recommendations based on both TF-IDF similarity and genre.

## Live Application

Use the deployed Streamlit application here:

**[Open Movie Recommendation System](https://movierecommendation-an.streamlit.app/)**

The Streamlit frontend currently uses the deployed FastAPI backend at:

`https://movierecommendation-f6g7.onrender.com`

## Features

- Search TMDB movies by title keyword.
- Display autocomplete-style movie suggestions and poster results.
- Browse home feeds for:
  - Trending movies
  - Popular movies
  - Top-rated movies
  - Movies currently playing
  - Upcoming movies
- Open a movie details page with:
  - Poster and backdrop images
  - Release date
  - Genres
  - Overview
- Generate two recommendation groups:
  - **Similar Movies:** local TF-IDF cosine-similarity recommendations based on the movie dataset.
  - **More Like This:** TMDB recommendations discovered from the selected movie's first genre.
- Navigate between home and detail views with Streamlit query parameters.
- Use the FastAPI Swagger documentation at `/docs` when running the backend.
- Cache short-lived search requests in the Streamlit frontend to reduce repeated API calls.

## Technology Stack

- **Frontend:** Streamlit
- **Backend API:** FastAPI
- **Server:** Uvicorn
- **Movie data and images:** The Movie Database (TMDB) API
- **Recommendation model:** TF-IDF and cosine similarity with scikit-learn and SciPy
- **Data processing:** pandas and NumPy
- **HTTP clients:** requests and httpx
- **Configuration:** python-dotenv
- **Python version:** 3.11.9

## Project Structure

```text
.
├── app.py                # Streamlit user interface
├── main.py               # FastAPI backend and recommendation endpoints
├── movies_metadata.csv   # Movie metadata source dataset
├── movies.ipynb          # Notebook used for exploration/model preparation
├── df.pkl               # Serialized movie DataFrame used by the API
├── indices.pkl          # Serialized movie-title-to-index mapping
├── tfidf.pkl            # Serialized TF-IDF vectorizer
├── tfidf_matrix.pkl     # Serialized TF-IDF matrix
├── requirements.txt     # Python dependencies
├── runtime.txt          # Deployment Python runtime
└── .gitignore            # Ignored local files and secrets
```

## Prerequisites

- Python 3.11 or a compatible Python 3 version.
- A TMDB API key. Create an account at [themoviedb.org](https://www.themoviedb.org/) and obtain an API key from the API settings.
- Git, if cloning the repository.

## Clone the Repository

```bash
git clone https://github.com/anudeepnakka3-star/movierecommendation.git
cd movierecommendation
```

## Local Setup

Create and activate a virtual environment:

### Windows PowerShell

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

### macOS or Linux

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Install the dependencies:

```bash
pip install -r requirements.txt
```

Create a `.env` file in the project root:

```env
TMDB_API_KEY=your_tmdb_api_key_here
```

The `.env` file is ignored by Git and should never be committed.

## Run the Backend Locally

Start the FastAPI server from the project root:

```bash
uvicorn main:app --reload --host 127.0.0.1 --port 8000
```

The local backend will be available at:

- API root: http://127.0.0.1:8000/
- Health check: http://127.0.0.1:8000/health
- Interactive Swagger docs: http://127.0.0.1:8000/docs

The API loads `df.pkl`, `indices.pkl`, `tfidf.pkl`, and `tfidf_matrix.pkl` during startup. Keep these files in the project root when running the backend locally.

## Run the Streamlit Frontend Locally

In a second terminal, activate the virtual environment and run:

```bash
streamlit run app.py
```

Streamlit normally opens at http://localhost:8501.

By default, `app.py` calls the deployed Render backend. To use the backend running on your own computer, change the `API_BASE` value near the top of `app.py` to:

```python
API_BASE = "http://127.0.0.1:8000"
```

Run the backend before opening the frontend if you make this change.

## API Endpoints

| Method | Endpoint | Purpose |
|---|---|---|
| `GET` | `/` | Returns a basic API status message and links to docs and health checks. |
| `GET` | `/health` | Returns the backend health status. |
| `GET` | `/home` | Returns poster cards for trending, popular, top-rated, now-playing, or upcoming movies. |
| `GET` | `/tmdb/search` | Searches TMDB and returns multiple movie results. |
| `GET` | `/movie/id/{tmdb_id}` | Returns details for a movie using its TMDB ID. |
| `GET` | `/recommend/genre` | Returns TMDB movies from the selected movie's first genre. |
| `GET` | `/recommend/tfidf` | Returns local TF-IDF recommendations for a title. |
| `GET` | `/movie/search` | Returns movie details plus TF-IDF and genre recommendations. |

Example requests:

```text
http://127.0.0.1:8000/home?category=popular&limit=24
http://127.0.0.1:8000/tmdb/search?query=batman
http://127.0.0.1:8000/recommend/tfidf?title=Toy%20Story&top_n=10
http://127.0.0.1:8000/movie/search?query=Inception&tfidf_top_n=12&genre_limit=12
```

## How Recommendations Work

### TF-IDF recommendations

The backend loads the serialized movie DataFrame, title index, TF-IDF vectorizer, and TF-IDF matrix. When a title is selected, it finds the corresponding row and calculates cosine similarity against the other movie vectors. The highest-scoring titles are returned, excluding the selected movie itself.

### Genre recommendations

The backend first retrieves the selected movie's TMDB details, takes its first listed genre, and queries TMDB's discover endpoint for popular movies in that genre. The selected movie is removed from the returned list.

If TF-IDF recommendations cannot be matched to TMDB movie cards, the frontend still attempts to show genre recommendations as a fallback.

## Deployment Notes

- The Streamlit deployment is available at https://movierecommendation-an.streamlit.app/.
- The frontend is configured to call the Render FastAPI service at `https://movierecommendation-f6g7.onrender.com`.
- Backend deployments must provide the `TMDB_API_KEY` environment variable.
- Backend deployments must include all four serialized pickle files used during startup.
- `runtime.txt` pins the deployment runtime to Python 3.11.9.

## Troubleshooting

### `TMDB_API_KEY missing`

Create a `.env` file locally or configure `TMDB_API_KEY` as an environment variable in the deployment platform, then restart the backend.

### The API fails during startup

Confirm that `df.pkl`, `indices.pkl`, `tfidf.pkl`, and `tfidf_matrix.pkl` exist in the same directory as `main.py`.

### The frontend cannot load movies

Check that the backend is running and that its `/health` endpoint responds with:

```json
{"status":"ok"}
```

When developing locally, confirm that `API_BASE` in `app.py` points to the backend you intend to use.

### No TF-IDF recommendations appear

The selected title must match a title in the local dataset/index. Genre recommendations can still work through TMDB when the local title lookup does not match.

## License

No license file is currently included in this repository.
