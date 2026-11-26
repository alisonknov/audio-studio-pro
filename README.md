# Audio Studio Pro

[![Ask DeepWiki](https://devin.ai/assets/askdeepwiki.png)](https://deepwiki.com/alisonknov/audio-studio-pro)

Audio Studio Pro is a web-based digital audio workstation (DAW) that leverages the powerful Demucs source separation model to split any song into its constituent instrumental stems. It features a modern, intuitive multitrack player where you can mix, pan, and adjust the volume of each track in real-time.

## Features

-   **AI-Powered Stem Separation**: Upload an audio file (WAV, FLAC, MP3) and automatically separate it into vocals, drums, bass, guitar, piano, and other instruments using the `htdemucs_6s` model.
-   **Interactive Multitrack Mixer**: A user-friendly interface displays a waveform for each separated stem.
-   **Real-Time Mixing Controls**: Adjust volume, pan (left/right balance), mute, and solo for each individual track.
-   **Synchronized Playback**: Play, pause, and seek through all tracks simultaneously.
-   **High-Quality Export**: Download your final mix or individual isolated stems in WAV, FLAC, or MP3 format.
-   **Drag & Drop Interface**: Easily upload your audio files by dragging them into the browser window.

## Tech Stack

The project is a monorepo composed of a Python backend and a Next.js frontend.

**Backend:**
- **Framework**: FastAPI
- **AI Model**: Meta's Demucs (`htdemucs_6s`) for audio source separation.
- **Audio Processing**: Pydub for mixing, format conversion, and exporting.
- **Server**: Uvicorn
- **Containerization**: Docker

**Frontend:**
- **Framework**: Next.js (with App Router)
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **State Management**: Zustand
- **Waveform Visualization**: `wavesurfer.js`
- **API Communication**: Axios
- **Icons**: Lucide React

## Project Structure

```
.
├── docker-compose.yml    # Defines a Redis service (not used by the current app logic)
├── backend/              # FastAPI application for audio processing
│   ├── Dockerfile
│   ├── requirements.txt
│   └── app/
│       ├── audio_engine.py # Core logic for running Demucs
│       └── main.py         # API endpoints for upload, status, and download
└── frontend/             # Next.js web interface
    ├── public/
    └── src/
        ├── app/          # Main application pages
        └── components/   # React components, including the MultiTrackPlayer
```

## Getting Started

Follow these instructions to get a copy of the project up and running on your local machine for development and testing purposes.

### Prerequisites

-   [Docker](https://www.docker.com/get-started) and Docker Compose
-   [Node.js](https://nodejs.org/) (v20 or later) and npm

### Installation

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/alisonknov/audio-studio-pro.git
    cd audio-studio-pro
    ```

2.  **Run the Backend (using Docker):**

    The backend is containerized for easy setup. It handles all Python dependencies, including PyTorch and Demucs.

    *   Build the Docker image:
        ```bash
        docker build -t audio-studio-pro-backend ./backend
        ```
    *   Run the container. This command maps port `8000` on your machine to the container's port and creates a volume to persist the separated audio files.
        ```bash
        docker run -d -p 8000:7860 -v ./backend/separated:/app/separated --name audio-pro-be audio-studio-pro-backend
        ```
    *   The backend API will now be available at `http://localhost:8000`.

3.  **Run the Frontend:**

    *   Navigate to the frontend directory:
        ```bash
        cd frontend
        ```
    *   Install the dependencies:
        ```bash
        npm install
        ```
    *   Run the development server:
        ```bash
        npm run dev
        ```
    *   Open your browser and navigate to `http://localhost:3000` to use the application.

## How It Works

1.  **Upload**: The user uploads an audio file through the Next.js web interface.
2.  **Processing**: The file is sent to the FastAPI backend. The backend assigns a task ID and starts a background process to separate the audio using the `demucs` command-line tool.
3.  **Polling**: The frontend periodically checks a status endpoint using the task ID to see if the separation is complete.
4.  **Mixing**: Once complete, the backend provides URLs to the separated stem files (e.g., `vocals.wav`, `drums.wav`). The frontend loads these stems into the `MultiTrackPlayer` component, which uses `wavesurfer.js` to render the waveforms.
5.  **Download**: When the user clicks "Exportar Mix", the frontend sends the current state of all tracks (volumes, pans, mutes) to the backend. The backend uses Pydub to create the final audio mix and sends the file back to the user for download.

## API Endpoints

The core API is served by the FastAPI backend.

-   `POST /api/upload`: Accepts an audio file upload and initiates the stem separation process.
-   `GET /api/status/{task_id}`: Returns the current status of a processing task (`"Processando na IA"`, `"Concluído"`, `"Falhou"`).
-   `POST /api/download`: Accepts mixing parameters (volumes, pans, mutes, format) and returns the final mixed audio file.
-   `/audio/*`: Serves the static separated audio files from the `separated` directory.
