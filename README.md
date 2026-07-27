# Kalkulator czasu półmaratonu

Aplikacja szacująca czas ukończenia półmaratonu na podstawie płci, wieku i czasu
na 5 km. Dane wejściowe użytkownik podaje w formie swobodnego tekstu — wyciąga
z niego potrzebne informacje model językowy (OpenAI + `instructor`).

## Jak to działa

1. **Dane** — wyniki półmaratonu wrocławskiego 2023/2024 (`halfmarathon_wroclaw_*.csv`)
   wgrane do Digital Ocean Spaces (`scripts/upload_data_to_spaces.py`).
2. **Trening modelu** — `training_pipeline.ipynb`: wczytuje dane ze Spaces, czyści je,
   ogranicza cechy do tych dostępnych w aplikacji (`gender`, `age`, `time_5k_s`) żeby
   uniknąć data leakage, trenuje regresję (PyCaret) i zapisuje model lokalnie
   oraz do Spaces (`models/polmaraton_model.pkl`).
3. **Aplikacja** — `app.py` (Streamlit): czat, w którym użytkownik opisuje siebie,
   LLM wyciąga płeć/wiek/czas na 5 km do struktury (`ExtractedInfo`), appka dopytuje
   o brakujące lub nierealne dane (poza zakresem, na którym trenował model), a na
   koniec pobiera model ze Spaces i pokazuje predykcję. Wywołania LLM są monitorowane
   przez Langfuse (`@observe()` + `langfuse.openai.OpenAI`).

## Struktura repo

```
app.py                     # aplikacja Streamlit
spaces_client.py           # klient S3-compatible do Digital Ocean Spaces
training_pipeline.ipynb    # notebook treningowy
scripts/upload_data_to_spaces.py
requirements.txt
Procfile                   # komenda startowa dla Digital Ocean App Platform
.python-version            # przypięty Python 3.11 (pycaret nie ma wheeli dla 3.14)
```

## Uruchomienie lokalne

```bash
pip install -r requirements.txt
cp .env.example .env   # uzupełnij klucze (DO Spaces, OpenAI, Langfuse)
streamlit run app.py
```

Zmienne środowiskowe (`.env`, patrz `.env.example`):

| Zmienna | Do czego |
|---|---|
| `DO_SPACES_KEY` / `DO_SPACES_SECRET` / `DO_SPACES_REGION` / `DO_SPACES_BUCKET` | Digital Ocean Spaces (dane + model) |
| `OPENAI_API_KEY` | ekstrakcja danych z tekstu użytkownika |
| `LANGFUSE_PUBLIC_KEY` / `LANGFUSE_SECRET_KEY` / `LANGFUSE_HOST` | monitoring wywołań LLM |

## Deploy

Aplikacja wdrożona przez Digital Ocean App Platform, podłączony do tego repo
(branch `main`, auto-deploy). Zmienne środowiskowe ustawione bezpośrednio w
panelu App Platform (nie ma tam pliku `.env` — `spaces_client.py` czyta je
z `os.environ`).
