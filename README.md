
# AudiobookBay Automated

AudiobookBay Automated is a lightweight web application designed to simplify audiobook management. It allows users to search [**AudioBook Bay**](https://audiobookbay.lu/) for audiobooks and send magnet links directly to a designated **Deluge, qBittorrent, Transmission, or rdtClient** client.

## How It Works
- **Search Results**: Users search for audiobooks. The app grabs results from AudioBook Bay and displays results with the **title** and **cover image**, along with two action links:
  1. **More Details**: Opens the audiobook's page on AudioBook Bay for additional information.
  2. **Download to Server**: Sends the audiobook to your configured torrent client for downloading.

- **Magnet Link Generation**: When a user selects "Download to Server," the app generates a magnet link from the infohash displayed on AudioBook Bay and sends it to the torrent client. Along with the magnet link, the app assigns:
  - A **category label** for organizational purposes.
  - A **save location** for downloaded files.


> **Note**: This app does not download or move any material itself (including torrent files). It only searches AudioBook Bay and facilitates magnet link generation for torrent.

**With rdtClient:** The flow is the same: search → "Download to Server" sends the magnet to rdtClient with the configured category and save path. The **Status** tab lists torrents in that category; only with rdtClient you can **Remove**, **Pause**, and **Resume** torrents from the app. Categories are created in rdtClient automatically if they don’t exist.

## Features
- **Search Audiobook Bay**: Easily search for audiobooks by title or keywords.
- **View Details**: Displays book titles and covers with quickly links to the full details on AudioBook Bay.
- **Basic Download Status Page**: Monitor the download status of items in your torrent client that share the specified category assigned.
- **No AudioBook Bay Account Needed**: The app automatically generates magnet links from the displayed infohashes and push them to your torrent client for downloading.
- **Automatic Folder Organization**: Once the download is complete, torrent will automatically move the downloaded audiobook files to your save location. Audiobooks are organized into subfolders named after the AudioBook Bay title, making it easy for [**Audiobookshelf**](https://www.audiobookshelf.org/) to automatically add completed downloads to its library.



## Why Use This?
AudiobookBay Downloader provides a simple and user-friendly interface for users to download audiobooks without on their own and import them into your libary. 

---

## Installation

### Prerequisites
- **Deluge, qBittorrent, Transmission, or rdtClient** (with the Web UI / API enabled)
- **Docker** (optional, for containerized deployments)

### Environment Variables
The app uses environment variables to configure its behavior. Set these in a `.env` file (local) or in your container environment (Docker).

**Required (pick one client):**
- `DOWNLOAD_CLIENT` — One of: `qbittorrent`, `transmission`, `delugeweb`, or `rdtclient`.

**Download client connection** — use either (a) or (b):
- **(a)** `DL_SCHEME`, `DL_HOST`, `DL_PORT` — e.g. `http`, `192.168.1.227`, `6500`
- **(b)** `DL_URL` — full URL, e.g. `http://192.168.1.07:6500` (overrides scheme/host/port if set)

**Auth and paths (same for all clients):**
- `DL_USERNAME` / `DL_PASSWORD` — login for the torrent client (rdtClient uses API auth)
- `DL_CATEGORY` — category/label for downloads (e.g. `Audiobooks`; used for Status tab filter with rdtclient)
- `SAVE_PATH_BASE` — root save path for downloads (from the torrent client’s perspective)

**Optional:**
- `ABB_HOSTNAME` — Audiobook Bay host (default `audiobookbay.lu`). If the hostname causes issues, try without quotes.
- `ABB_VERIFY_SSL` — set to `false` to disable SSL verification for Audiobook Bay requests (default `true`).
- `PAGE_LIMIT` — max search result pages (default `5`; higher may hit rate limits).
- `FLASK_PORT` or `PORT` — app port (default `5078`).
- **rdtClient only:** `RDTCLIENT_API_PREFIX` — API path (default `/api/v2`). Only needed if your rdtClient uses a different prefix.
- **Dashboard message:** `DASHBOARD_MESSAGE_ENABLED` — `true`/`false` (default `true`). `DASHBOARD_MESSAGE` — custom text; use `\n` for new lines.

**Optional nav link** (adds a link in the nav bar, e.g. to your audiobook player):
- `NAV_LINK_NAME` — link text (e.g. `Open Audiobook Player`)
- `NAV_LINK_URL` — URL (e.g. `https://audiobooks.yourdomain.com/`)

**Example `.env` for rdtClient:**
```env
DOWNLOAD_CLIENT=rdtclient
DL_SCHEME=http
DL_HOST=192.168.1.227
DL_PORT=6500
DL_USERNAME=admin
DL_PASSWORD=your_password
DL_CATEGORY=Audiobooks
SAVE_PATH_BASE=/audiobooks
ABB_HOSTNAME=audiobookbay.is
NAV_LINK_NAME=audiobookshelf
NAV_LINK_URL=https://audio.yourdomain.com
# Optional: ABB_VERIFY_SSL=false, DASHBOARD_MESSAGE_ENABLED=true, DASHBOARD_MESSAGE=Line1\nLine2
```

### Using Docker

1. Use `docker-compose` for quick deployment. Example `docker-compose.yml` (use your own image or the upstream one):

   ```yaml
   version: '3.8'

   services:
     audiobookbay-rdtclient:
       image: ghcr.io/KenGrinder/audiobookbay-rdtclient:latest
       ports:
         - "5078:5078"
       container_name: audiobookbay-rdtclient
       env_file:
         - .env
       # Or list env vars explicitly, e.g. DOWNLOAD_CLIENT=rdtclient, DL_HOST=..., DL_PORT=..., etc.
   ```

2. **Start the Application**:
   ```bash
   docker-compose up -d
   ```
   Ensure your `.env` contains at least `DOWNLOAD_CLIENT`, connection (e.g. `DL_HOST`, `DL_PORT`), `DL_USERNAME`, `DL_PASSWORD`, `DL_CATEGORY`, and `SAVE_PATH_BASE` as described above.

### Running Locally
1. **Install Dependencies**:
   Ensure you have Python installed, then install the required dependencies:
   ```bash
   pip install -r requirements.txt
   
2. Create a .env file in the project directory to configure your application. Below is an  example of the required variables:
    ```
    # Torrent Client Configuration
    DOWNLOAD_CLIENT=transmission # Change to delugeweb, transmission or qbittorrent
    DL_SCHEME=http
    DL_HOST=192.168.1.123
    DL_PORT=8080
    DL_USERNAME=admin
    DL_PASSWORD=pass
    DL_CATEGORY=abb-downloader
    SAVE_PATH_BASE=/audiobooks
    
    # AudiobookBar Hostname
    ABB_HOSTNAME='audiobookbay.is' #Default
    # ABB_HOSTNAME='audiobookbay.lu' #Alternative

    PAGE_LIMIT=5 #Default
    FLASK_PORT=5078 #Default

    # Optional Navigation Bar Entry
    NAV_LINK_NAME=Open Audiobook Player
    NAV_LINK_URL=https://audiobooks.yourdomain.com/
    ```

3. Start the app:
   ```bash
   python app.py
   ```

---

## Notes
- **This app does NOT download any material**: It simply generates magnet links and sends them to your configured torrent client (qBittorrent, Transmission, rdtClient, etc.) for handling.

- **Folder Mapping**: __The `SAVE_PATH_BASE` is based on the perspective of your torrent client__, not this app. This app does not move any files; all file handling and organization are managed by the torrent client. Ensure that the `SAVE_PATH_BASE` in your torrent client aligns with your audiobook library (e.g., for Audiobookshelf). Using a path relative to where this app is running, instead of the torrent client, will cause issues.


---

## Feedback and Contributions
This project is a work in progress, and your feedback is welcome! Feel free to open issues or contribute by submitting pull requests.

---

## Screenshots
### Search Results
![screenshot-2025-01-13-19-59-03](https://github.com/user-attachments/assets/8a30fd4e-a289-49d0-83ab-67a3bcfc9745)

### Download Status
![screenshot-2025-01-13-19-59-25](https://github.com/user-attachments/assets/19cc74de-51fc-422f-9cab-fe69e30c74b9)

---
