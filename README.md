# 🎶 Shared Music Player (YouTube Jukebox) 🎶

A web application that allows multiple users to join a virtual room and collectively manage and listen to music from YouTube in real-time.

## Project Goals

To create a shared music platform where users can create or join virtual listening rooms. One user acts as the host, controlling playback, while others can contribute by adding songs to a shared playlist. The application aims for real-time synchronization of the listening experience across all participants in a room.

## Features

-   **Join/Create Room:** Users enter a username and a room name to join. A new room is created if it doesn't exist.
-   **Role Management:** The first user to join a room automatically becomes the host.
-   **Shared Playlist:** All users in a room see and contribute to a common playlist.
-   **Add Music:**
    -   Search for songs from YouTube.
    -   Add selected songs to the end of the playlist.
    -   Prevent adding duplicate songs (based on YouTube Video ID).
    -   Track and display the user who added each song.
-   **Playback Control (Host Only):**
    -   Play the current song.
    -   Skip to the next song.
    -   Go back to the previous song.
    -   Select and play any song from the playlist by clicking on it.
    -   Pause/Play toggle.
    -   Seek within the currently playing song.
-   **Player Modes:** Toggle between displaying the full YouTube video player or just the thumbnail and song information.
-   **Real-time Synchronization:** Updates to the playlist, playback state (play/pause, current time, current song), and users in the room are synced to all connected clients in real-time using WebSockets.
-   **Seamless Joining:** When a new user joins a room, the currently playing song or playback state is not interrupted. The new user receives the current room state upon connection.
-   **Persistence:** Room data (playlist, users, host status) is stored in MongoDB to allow state recovery and longer-term room existence.

## Technology Stack

-   **Frontend:**
    -   React / Next.js: Building the user interface and managing client-side state.
    -   YouTube Iframe Player API: Embedding and controlling the YouTube player.
    -   Socket.IO Client: Real-time communication with the backend.
-   **Backend:**
    -   Node.js / Express: Building the API and WebSocket server.
    -   Socket.IO: Managing WebSocket connections and broadcasting real-time events.
    -   Mongoose / MongoDB Driver: Interacting with the MongoDB database for data persistence.
    -   axios / node-fetch: Making external API calls (e.g., to YouTube Data API).
    -   Google APIs Client Library (optional): For easier interaction with YouTube Data API.
-   **Database:**
    -   MongoDB: Storing persistent room data.
-   **API:**
    -   YouTube Data API v3: Searching and retrieving YouTube video information.

## Architecture

The application follows a client-server architecture with real-time communication:

1.  **Client (Frontend):**
    -   Handles the user interface, interacts with the embedded YouTube player (via Iframe API).
    -   Sends user actions (joining a room, adding music, controlling playback - host only) to the backend via HTTP requests or Socket.IO events.
    -   Receives real-time room state updates (playlist changes, new song playing, new user joined) from the backend via Socket.IO and updates the UI accordingly.
    -   Each client runs its own YouTube player instance, synchronizing its state (play, pause, seek, current time) based on data received from the server.

2.  **Server (Backend):**
    -   Acts as the **single source of truth** for the state of all active rooms.
    -   Uses Socket.IO to manage client connections and group clients into Socket.IO "rooms" based on the user-provided room name.
    -   Receives requests from clients, validates permissions (e.g., only the host can skip songs).
    -   Performs operations like adding songs to the playlist or updating the current song state.
    -   Interacts with MongoDB to save/update room states.
    -   Uses the YouTube Data API to search for videos requested by users.
    -   Broadcasts room state changes (playlist updates, song playing, current time) to all connected clients *within that specific room* via Socket.IO. When a new user joins, the server sends the current state of their target room to them.

3.  **Database (MongoDB):**
    -   Stores information about rooms (e.g., room name, current users list, playlist items, current song index, host status).
    -   Allows rooms to persist even if no one is currently in them or if the server restarts (depending on specific implementation logic).

**Why a Backend is Necessary:**

-   **State Synchronization:** A frontend-only approach cannot reliably synchronize the playlist, current song, playback position, and user presence across multiple disconnected clients in real-time. The server is the central point that holds the shared state and pushes updates to everyone.
-   **Real-time Communication:** WebSocket (Socket.IO) technology requires a server to establish and manage persistent, bi-directional connections.
-   **API Key Security:** Exposing your YouTube Data API Key directly in frontend code is a security risk, making it vulnerable to misuse. The backend acts as a secure proxy to make API calls.
-   **Persistence:** Storing room data beyond a single user's browser session (e.g., in a database like MongoDB) requires a server to interact with the database.

## Setup

To set up and run this project, you need to configure and start both the backend and frontend.

**1. Clone the Repository:**

```bash
git clone <YOUR_REPOSITORY_URL>
cd <your_project_directory_name>
```

**2. Backend Setup:**

```bash
# Navigate into the backend directory (e.g., ./backend)
cd backend # Or your specific backend folder name

# Install dependencies
npm install # or yarn install
```

**3. Backend Configuration:**

-   Create a `.env` file in the backend directory.
-   Configure the following environment variables:

    ```env
    PORT=5000 # The port the backend server will run on (or any preferred port)
    MONGODB_URI=mongodb://localhost:27017/shared_music # Your MongoDB connection string
    YOUTUBE_API_KEY=YOUR_YOUTUBE_DATA_API_KEY # Replace with your actual YouTube Data API Key
    ```

    -   **Getting `MONGODB_URI`:** If using local MongoDB, it's typically `mongodb://localhost:27017/<your_database_name>`. If using MongoDB Atlas, you'll get a connection string from their dashboard.
    -   **Getting `YOUTUBE_API_KEY`:** You need to create a project in Google Cloud Platform, enable the YouTube Data API v3, and create an API Key. Details can be found in the [Google Cloud Console](https://console.cloud.google.com/). Be sure to keep this key secure.

**4. Frontend Setup:**

```bash
# Navigate into the frontend directory (e.g., ../frontend)
cd ../frontend # Or your specific frontend folder name

# Install dependencies
npm install # or yarn install
```

**5. Frontend Configuration:**

-   If using Next.js API Routes for API calls (like YouTube search), configure environment variables in `.env.local` at the frontend root.
-   Crucially, configure the URL of your backend server for Socket.IO connections.

    ```env
    # In the root directory of your frontend (for Next.js)
    NEXT_PUBLIC_BACKEND_URL=http://localhost:5000 # The URL of your backend server
    # If you were to expose YouTube API Key on frontend (less secure)
    # NEXT_PUBLIC_YOUTUBE_API_KEY=YOUR_YOUTUBE_DATA_API_KEY
    ```

    -   Variables prefixed with `NEXT_PUBLIC_` in Next.js are made available to the browser.

## How to Run

**1. Start the Backend Server:**

In the backend directory:

```bash
npm start # Or the command to start your main server file (e.g., node dist/server.js)
```

The server will run on the configured port (default is 5000).

**2. Start the Frontend Development Server:**

In the frontend directory:

```bash
npm run dev # For Next.js development mode
# or npm start # For a standard Create React App production build
```

The frontend development server will typically run on Next.js's default port (usually 3000).

**3. Access the Application:**

Open your web browser and navigate to: `http://localhost:3000`

## Usage

1.  **Access the page:** Enter your desired **Username** and **Room Name** to join or create.
2.  **Join/Create:** Click the **Join Room** button.
3.  **Inside the Room:**
    *   **Search:** Type a song title or YouTube link into the search bar and press Enter or click Search.
    *   **Add Music:** From the search results, click the **Add to Playlist** button to add a song.
    *   **Playlist:** View the list of songs in the room's shared playlist. The name of the user who added each song will be displayed.
    *   **Player:**
        *   If you are the **Host**, you will see playback control buttons (Play/Pause, Next, Previous, Seek bar, buttons to play specific songs in the playlist). Use these to control the music for everyone in the room.
        *   If you are a **Guest**, you will primarily see the currently playing song and the playlist.
    *   **Display Mode:** There will be a button to toggle between showing the full video player or just the song's thumbnail and information.
    *   **Room Members:** See a list of users currently in the room (feature to be implemented).

## Project Structure (Suggested)

```
/
├── backend/
│   ├── src/
│   │   ├── models/       # Mongoose schemas (Room, User, Song)
│   │   ├── services/     # Business logic (YouTube API calls, DB operations)
│   │   ├── routes/       # HTTP routes (if any, e.g., auth)
│   │   ├── socket/       # Socket.IO event handlers and room logic
│   │   └── server.ts     # Main server setup (Express, Socket.IO, DB connection)
│   ├── .env              # Environment variables
│   ├── package.json
│   └── tsconfig.json     # If using TypeScript
│
├── frontend/
│   ├── src/
│   │   ├── components/   # Reusable React components (Player, Playlist, Search, etc.)
│   │   ├── pages/        # Next.js pages (index.tsx, room/[roomId].tsx)
│   │   ├── hooks/        # Custom React hooks (e.g., useSocket, usePlayer)
│   │   ├── utils/        # Utility functions
│   │   ├── contexts/     # React Context for global state (User, Room, Socket)
│   │   └── styles/       # CSS or styling solution
│   ├── public/           # Static assets
│   ├── .env.local        # Environment variables (Next.js)
│   ├── package.json
│   ├── tsconfig.json     # If using TypeScript
│   └── next.config.js
│
├── .gitignore
├── README.md             # This file
└── package.json          # Optional: Monorepo package.json
```

## Potential Future Enhancements

-   **In-room Chat:** Add a chat feature for users within the same room.
-   **Improved Sync:** Enhance synchronization of playback position and volume across clients (can be challenging).
-   **Playlist Reordering:** Allow users (maybe only host or via voting) to reorder the playlist.
-   **Room/Playlist History:** Store a history of joined rooms or listened playlists.
-   **Authentication:** Implement user registration/login for personalized features (favorite playlists, profiles).
-   **Other Music Sources:** Support music from sources other than YouTube (requires checking APIs and licensing).
-   **Advanced Host Controls:** Implement features like queue mode, muting, or kicking users.

## Contributing

Contributions are welcome! Please fork the repository and submit a Pull Request with your features or bug fixes.

## License

This project is licensed under the [Choose a License, e.g., MIT License](LICENSE).
