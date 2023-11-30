<template>
  <div id="app">
    <h1>Movies</h1>
    <div>
      <input
        type="text"
        v-model="selectedMovieTitle"
        placeholder="Movie title"
      />
      <select v-model="selectedDistributor">
        <option
          v-for="distributor in distributors"
          :key="distributor.distributor_id"
          :value="distributor.distributor_id"
        >
          {{ distributor.name }}
        </option>
      </select>

      <select v-model="selectedGenre">
        <option
          v-for="genre in genres"
          :key="genre.genre_id"
          :value="genre.genre_id"
        >
          {{ genre.name }}
        </option>
      </select>

      <select v-model="selectedMpaa">
        <option
          v-for="mpaa in mpaas"
          :key="mpaa.mpaa_rating_id"
          :value="mpaa.mpaa_rating_id"
        >
          {{ mpaa.rating }}
        </option>
      </select>

      <button @click="fetchFilteredMovies">Filter Movies</button>
      <button @click="resetFilters">Reset Filters</button>
    </div>
    <div>
      <table>
        <thead>
          <tr>
            <th>Movie ID</th>
            <th>Release Date</th>
            <th>Title</th>
            <th>Production Budget</th>
            <th>Domestic Gross</th>
            <th>Worldwide Gross</th>
            <th>Genre ID</th>
            <th>MPAA Rating ID</th>
            <th>Distributor ID</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="movie in movies" :key="movie.movie_id">
            <td>{{ movie.movie_id }}</td>
            <td>{{ movie.release_date }}</td>
            <td>{{ movie.title }}</td>
            <td>{{ movie.production_budget }}</td>
            <td>{{ movie.domestic_gross }}</td>
            <td>{{ movie.worldwide_gross }}</td>
            <td>{{ genreName(movie.genre_id) }}</td>
            <td>{{ mpaaRating(movie.mpaa_rating_id) }}</td>
            <td>{{ distributorName(movie.distributor_id) }}</td>
          </tr>
        </tbody>
      </table>
      <button @click="changePage(-1)" :disabled="currentPage <= 1">
        Previous
      </button>
      <span>Page {{ currentPage }}</span>
      <button
        class="next-button"
        @click="changePage(1)"
        :disabled="currentPage == totalPages"
      >
        Next
      </button>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "App",
  data() {
    return {
      movies: [],
      distributors: [],
      distributorMap: {},
      genres: [],
      genreMap: {},
      mpaas: [],
      mpaaMap: {},
      currentPage: 1,
      selectedMovieTitle: "",
      selectedDistributor: null,
      selectedGenre: null,
      selectedMpaa: null,
      totalPages: null,
      totalItems: null,
    };
  },

  methods: {
    async fetchFilteredMovies() {
      try {
        const response = await axios.get(
          `https://x101g9s6lj.execute-api.us-east-1.amazonaws.com/dev/movies`,
          {
            params: {
              page: this.currentPage,
              distributor: this.selectedDistributor,
              genre: this.selectedGenre,
              mpaa: this.selectedMpaa,
              title: this.selectedMovieTitle,
            },
          }
        );
        this.movies = response.data.movies;
        this.totalPages = response.data.totalPages;
        this.totalItems = response.data.totalItems;
        console.log("totalPages " + this.totalPages);
      } catch (error) {
        console.error("Error fetching movies:", error);
      }
    },
    async fetchDistributors() {
      try {
        const response = await axios.get(
          "https://x101g9s6lj.execute-api.us-east-1.amazonaws.com/dev/distributors"
        );
        this.distributors = response.data;
        this.createDistributorMap();
      } catch (error) {
        console.error("Error fetching distributors:", error);
      }
    },
    async fetchGenres() {
      try {
        const response = await axios.get(
          "https://x101g9s6lj.execute-api.us-east-1.amazonaws.com/dev/genres"
        );
        this.genres = response.data;
        this.createGenreMap();
      } catch (error) {
        console.error("Error fetching genres:", error);
      }
    },
    async fetchMpaas() {
      try {
        const response = await axios.get(
          "https://x101g9s6lj.execute-api.us-east-1.amazonaws.com/dev/mpaa-ratings"
        );
        this.mpaas = response.data;
        this.createMpaaMap();
      } catch (error) {
        console.error("Error fetching genres:", error);
      }
    },
    createGenreMap() {
      this.genreMap = this.genres.reduce((map, genre) => {
        map[genre.genre_id] = genre.name;
        return map;
      }, {});
    },
    genreName(id) {
      return this.genreMap[id] || "Unknown"; // Return the genre name or 'Unknown' if not found
    },
    createMpaaMap() {
      this.mpaaMap = this.mpaas.reduce((map, mpaa) => {
        map[mpaa.mpaa_rating_id] = mpaa.rating;
        return map;
      }, {});
    },
    mpaaRating(id) {
      return this.mpaaMap[id] || "Unknown"; // Return the genre name or 'Unknown' if not found
    },
    createDistributorMap() {
      this.distributorMap = this.distributors.reduce((map, distributor) => {
        map[distributor.distributor_id] = distributor.name;
        return map;
      }, {});
    },
    distributorName(id) {
      return this.distributorMap[id] || "Unknown"; // Return the genre name or 'Unknown' if not found
    },
    changePage(increment) {
      this.currentPage += increment;
      this.fetchFilteredMovies();
    },
    resetFilters() {
      this.selectedDistributor = null;
      this.selectedGenre = null;
      this.selectedMpaa = null;
      this.currentPage = 1;
      this.selectedMovieTitle = "";
      this.fetchFilteredMovies();
    },
  },
  mounted() {
    this.fetchFilteredMovies();
    this.fetchDistributors();
    this.fetchGenres();
    this.fetchMpaas();
  },
};
</script>
<style >
body {
  font-family: "Arial", sans-serif;
  margin: 0;
  padding: 0;
  background-color: #f4f4f4;
}

#app {
  max-width: 1200px;
  margin: auto;
  padding: 20px;
  background-color: white;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

h1 {
  color: #333;
  text-align: center;
}

select {
  width: 150px;
  padding: 10px;
  margin-right: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  background-color: white;
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap;
}

option {
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap;
}

button {
  background-color: #4caf50;
  color: white;
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s;
  margin-right: 10px;
}

button:hover {
  background-color: #45a049;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
  margin-bottom: 30px;
}

th,
td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}

th {
  background-color: #f0f0f0;
}

tr:nth-child(even) {
  background-color: #f2f2f2;
}

tr:hover {
  background-color: #ddd;
}
.next-button {
  margin-left: 10px; /* Add left margin to the Next button */
}

.pagination {
  text-align: center;
  margin-top: 20px;
}

.pagination span {
  margin: 0 10px;
}
</style>
