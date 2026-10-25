# Lunar News Geolocation

This project reads news articles from `nasa_news.txt`, identifies lunar place
names using spaCy and a USGS lunar gazetteer, and plots the matched coordinates
over a lunar surface image in `geo_pars.ipynb`.

## Required project files

Keep these files in the same project folder:

- `geo_pars.ipynb` — notebook and mapping code.
- `nasa_news.txt` — UTF-8 text containing one article per entry. Each entry is
  a single line of text, and entries are separated by a line containing only
  `#`.

  Example structure:

  ```text
  VIPER rover is scheduled to land near the Nobile Crater at the South Pole.
  #
  Artemis III scouts targets near Shackleton and Mare Serenitatis.
  #
  Blue Ghost left Earth on 15 January, after being launched by US firm Firefly Aerospace with the intention of exploring the Sea of Crises, a huge basin visible from Earth.
  #
  ```
- `moon_geolocation.csv` — USGS planetary nomenclature export for the Moon.
  The notebook reads feature names, feature types, center latitudes, and center
  longitudes from this file. Download the lunar gazetteer from the
  (https://planetarynames.wr.usgs.gov/SearchResults?Target=16_Moon&approvalDateFrom=01%2F01%2F1973&approvalDateTo=12%2F31%2F1973&pageSize=500&displayAll=true&output=csv)

  [USGS Astrogeology Gazetteer](https://planetarynames.wr.usgs.gov/).
- `lunar_surface.jpg` — equirectangular lunar surface texture used as the map
  background. The current image is the NASA GSFC Scientific Visualization
  Studio LROC color map (`lroc_color_2k.jpg`). Source:
  [NASA CGI Moon Kit](https://svs.gsfc.nasa.gov/4720/).

The Compose configuration mounts the project folder into `/workspace`, so
these files remain on the host and the notebook can read them by their
project-relative filenames.

## Coordinate system

The notebook plots the USGS gazetteer's center latitude and longitude as
selenographic coordinates: longitude is the horizontal axis and latitude is
the vertical axis. The lunar surface image is displayed as an equirectangular
map, with longitude mapped to image x and latitude to image y. No coordinate
conversion is performed, so the map assumes the gazetteer and image use
matching coordinate conventions.

## Python requirements

The Docker image uses Python 3.13 and installs the packages listed in
`req_nb.txt`:

- `ipykernel`
- `notebook`
- `matplotlib`
- `plotly`
- `numpy`
- `pandas`
- `spacy`
- `spacy-transformers`

The `Dockerfile` also installs CPU-only PyTorch and spaCy's
`en_core_web_trf` transformer model. These are needed by the notebook's
transformer pipeline; model installation is part of the image build.

## Build and run with Docker

Install and start Docker Desktop, then open PowerShell in this project folder
and build and start the notebook server:

```powershell
docker compose up --build -d
```

Get the Jupyter access URL, including its token, from the container logs:

```powershell
docker compose logs geo-moon
```

Open the `http://127.0.0.1:8888/tree?token=...` URL from the logs in a
browser. Open `geo_pars.ipynb` in Jupyter and run the notebook cells in order.
The `nasa_news.txt` file and the generated notebook outputs are stored in the
mounted project folder.

To stop the server:

```powershell
docker compose down
```

To execute the notebook from PowerShell without opening the Jupyter UI:

```powershell
docker compose run --rm geo-moon jupyter nbconvert --to notebook --execute --inplace geo_pars.ipynb --ExecutePreprocessor.timeout=600
```
