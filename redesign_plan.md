# Technical Specification: vgmstream-web UI Redesign

This document outlines the technical plan for redesigning the user interface of the vgmstream-web application. The goal is to create a more modern, user-friendly interface with enhanced functionality, including drag-and-drop support, a media player with a playlist, and batch file operations.

## 1. Proposed HTML Structure

The existing HTML structure will be updated to support the new features. The following is a proposed structure for the `<body>` of `index.html`:

```html
<body>
  <main>
    <header>
      <h2>vgmstream-web</h2>
    </header>

    <div id="drop-zone" class="drop-zone">
      <p>Drag & drop WEM files or folders here, or click to select.</p>
      <input type="file" multiple id="file-input" class="visually-hidden">
    </div>

    <div id="player-container" class="player-container">
      <div id="media-player" class="media-player">
        <custom-audio id="audio-player" controls></custom-audio>
      </div>
      <div id="playlist-container" class="playlist-container">
        <ul id="playlist" class="playlist">
          <!-- Playlist items will be dynamically added here -->
        </ul>
      </div>
    </div>

    <div id="controls" class="controls">
      <button id="download-selected-btn" class="button">Download Selected</button>
      <button id="download-all-btn" class="button">Download All</button>
      <button id="rename-selected-btn" class="button">Rename Selected</button>
      <button id="rename-all-btn" class="button">Rename All</button>
      <button id="remove-selected-btn" class="button">Remove Selected</button>
    </div>
  </main>

  <div id="footer" data-nosnippet>
    <!-- Existing footer content -->
  </div>

  <div id="fadeoverlay" data-nosnippet>...</div>

  <script src="js/soundbuffer.js"></script>
  <script src="js/custom-audio.js"></script>
  <script src="js/player.js"></script>
</body>
```

### Key Changes:

-   **`<div id="drop-zone">`**: A dedicated area for file drag-and-drop. It will also function as a large file selection button.
-   **`<input type="file" id="file-input">`**: A hidden file input that will be triggered programmatically by clicking the drop zone.
-   **`<div id="player-container">`**: A container for the media player and the playlist, allowing for a more organized layout.
-   **`<div id="media-player">`**: A wrapper for the custom audio element.
-   **`<div id="playlist-container">`**: A container for the playlist.
-   **`<ul id="playlist">`**: An unordered list that will be populated with playlist items.
-   **`<div id="controls">`**: A container for action buttons related to the playlist.

## 2. JavaScript Logic

The JavaScript logic will be updated to handle the new UI elements and features. The following is an outline of the required functions and logic, which will be implemented in `js/player.js`.

### 2.1. Drag-and-Drop

-   **`initializeDragAndDrop()`**:
    -   Add event listeners (`dragenter`, `dragover`, `dragleave`, `drop`) to the `#drop-zone` element.
    -   Prevent default browser behavior for these events.
    -   Add a visual indicator when a file is dragged over the drop zone.
-   **`handleDrop(event)`**:
    -   Get the dropped files from `event.dataTransfer.files`.
    -   Filter for WEM files.
    -   Initiate the conversion process for each valid file.

### 2.2. Playlist Management

-   **`addToPlaylist(file, convertedUrl)`**:
    -   Create a new playlist item element (`<li>`).
    -   The item should display the original filename and a remove button.
    -   Store the original filename, converted URL, and a unique ID in the element's dataset.
    -   Add an event listener to the playlist item to play the corresponding track when clicked.
    -   Add an event listener to the remove button to remove the item from the playlist.
-   **`removeFromPlaylist(event)`**:
    -   Remove the corresponding playlist item from the DOM.
    -   Clean up any associated data.
-   **`playTrack(event)`**:
    -   Get the converted URL from the clicked playlist item's dataset.
    -   Set the `src` of the `#audio-player` to the URL and play the audio.
    -   Highlight the currently playing track in the playlist.

### 2.3. WEM to OGG Conversion

-   The existing conversion logic using the `cli-worker.js` will be adapted.
-   When files are dropped or selected, the `convertFile` or `convertDir` function will be called for each file.
-   Upon successful conversion, the `addToPlaylist` function will be called with the original file and the converted file's URL.

### 2.4. Download and Rename Features

-   **`downloadSelected()`**:
    -   Get all selected playlist items.
    -   Create a zip archive of the converted OGG files.
    -   Trigger a download of the zip file.
-   **`downloadAll()`**:
    -   Get all playlist items.
    -   Create a zip archive of all converted OGG files.
    -   Trigger a download of the zip file.
-   **`renameSelected()`**:
    -   Get all selected playlist items.
    -   Prompt the user for a new name or a naming pattern (e.g., `track_#`).
    -   Rename the files in the playlist and update their display names.
-   **`renameAll()`**:
    -   Similar to `renameSelected()`, but for all files in the playlist.

## 3. CSS Styles

New CSS classes will be added to `css/player.css` to style the new UI elements.

```css
/* Drop Zone */
.drop-zone {
  border: 2px dashed #ccc;
  border-radius: 10px;
  padding: 40px;
  text-align: center;
  cursor: pointer;
  transition: background-color 0.3s;
}

.drop-zone.drag-over {
  background-color: #f0f0f0;
}

.visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  margin: -1px;
  padding: 0;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  border: 0;
}

/* Player and Playlist */
.player-container {
  display: flex;
  margin-top: 20px;
  gap: 20px;
}

.media-player {
  flex: 1;
}

.playlist-container {
  flex: 1;
  max-height: 400px;
  overflow-y: auto;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.playlist {
  list-style: none;
  padding: 0;
  margin: 0;
}

.playlist-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  border-bottom: 1px solid #eee;
  cursor: pointer;
}

.playlist-item:hover {
  background-color: #f9f9f9;
}

.playlist-item.playing {
  background-color: #e0f7fa;
}

.playlist-item .remove-btn {
  background: none;
  border: none;
  color: #f00;
  cursor: pointer;
}

/* Controls */
.controls {
  margin-top: 20px;
  display: flex;
  gap: 10px;
}