# Astral Map Explorer

Astral Map Explorer is a Svelte application that visualizes stars and constellations on an interactive map. It allows users to explore different constellations, adjust star brightness, and view detailed information about selected stars.

## Features

- **Interactive Star Map**: Visualize stars and constellations on a dynamic map.
- **Constellation Filtering**: Filter stars by their constellations.
- **Brightness Adjustment**: Adjust the visibility of stars based on their magnitude.
- **Star Search**: Search for stars by their name.
- **Detailed Star Information**: View detailed information about selected stars, including their coordinates, distance, spectral type, and more.
- **Meteors Animation**: Enjoy a meteor shower animation on the star map.
- **Navigation**: Easily navigate between the home page and detailed star information pages.

## Pages

### Home Page

The home page displays the star map and provides various interactive features to explore the stars.

#### Components

- **Constellation Filter**: A dropdown menu to filter stars by their constellations.
- **Brightness Slider**: A slider to adjust the brightness of the stars.
- **Star Map**: An SVG-based star map displaying stars and constellations. Users can click on stars to view detailed information.
- **Star Search**: An input field to search for stars by name.
- **Star List**: A list of stars based on the current filter and search query. Each star in the list has a button to view more details.

### Detail Page

The detail page provides in-depth information about a selected star.

#### Components

- **Back Button**: A button to navigate back to the home page.
- **Star Information**: Displays detailed information about the selected star, including its name, constellation, magnitude, coordinates, distance, spectral type, color index, and radial velocity.
- **Synopsis**: A brief description of the star fetched from Wikipedia (if available).
- **Star Image**: An image of the star fetched from Wikipedia (if available).
- **Star Map**: An SVG-based star map showing the position of the selected star, the sun, and the Earth's orbit.

## Usage

1. **Explore Stars**: Use the constellation filter and brightness slider to explore different stars and constellations.
2. **Search for Stars**: Use the search input to find stars by name.
3. **View Star Details**: Click on a star on the map or in the list to view detailed information on the detail page.
4. **Navigate**: Use the back button on the detail page to return to the home page.

## Setup and Installation

1. Clone the repository.
2. Install dependencies using `npm install`.
3. Run the development server using `npm run dev`.
4. Open your browser and navigate to `http://localhost:5173` to view the application.

## Recommended IDE Setup

- [VS Code](https://code.visualstudio.com/) with the [Svelte](https://marketplace.visualstudio.com/items?itemName=svelte.svelte-vscode) extension.

## Technical Considerations

- This project uses Svelte with Vite for fast development and hot module replacement.
- The star data is fetched from a local JSON file (`/data/stardata.json`).
- The application uses `axios` for HTTP requests and `svelte-spa-router` for navigation.
- State management is handled using Svelte's built-in stores.

## Contribution

Contributions are welcome! Please open an issue or submit a pull request with your improvements.

## License

This project is licensed under the MIT License.
