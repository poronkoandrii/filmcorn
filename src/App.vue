<template>
  <div id="app">
    <header
      class="header"
      id="info"
      :style="{
        'background-image': 'url(' + getUrl + ')'
      }"
    >
      <div class="mainer">
        <div class="header__auto">
          <a href="/" class="header__auto-logo">
            <b>film</b>corn
          </a>
          <div class="header__auto-log">
            <div class="header__auto-log-search" :class="{active: isSearch}">
              <input
                type="search"
                class="header__auto-log-search-input"
                v-model="search"
                :placeholder="isLang ? 'Поиск' : 'Search'"
                @keyup.enter="search ? searchMovie() : ''"
              />
              <i
                class="fas fa-search header__auto-log-search-icon"
                @click="search ? searchMovie() : isSearch = !isSearch"
              ></i>
            </div>
            <div
              class="header__auto-log-btn"
              :class="{ active: !isLang }"
              @click="changeLang('en-US')"
            >English</div>
            <div
              class="header__auto-log-btn"
              :class="{ active: isLang }"
              @click="changeLang('ru-RU')"
            >Русский</div>
          </div>
        </div>
        <div class="header__info">
          <div class="header__info-left">
            <div class="header__info-left-name">{{ oneFilm.title }}</div>
            <div class="header__info-left-genre">{{ getGenreNames(oneFilm.genres) }}</div>
            <div class="header__info-left-raiting">
              <div class="header__info-left-raiting-stars">
                <span
                  class="movies__item-card-raiting-stars-bg"
                  :style="{ width: oneFilm.vote_average * 10 + '%' }"
                ></span>
              </div>
              <div class="header__info-left-raiting-points">{{ numRound(oneFilm.vote_average, 1) }}</div>
            </div>
          </div>
          <div class="header__info-right">
            <transition name="fade">
              <div class="header__info-right-about" v-if="isOverview">
                <div class="header__info-right-about-caption">
                  {{isLang ? 'Оригинальное название:' : 'Original name:'}}
                  <span
                    class="header__info-right-about-caption-desc"
                  >
                    {{
                    oneFilm.original_title
                    }}
                  </span>
                </div>
                <div class="header__info-right-about-caption">
                  {{isLang ? 'Дата выхода:' : 'Release date:'}}
                  <span
                    class="header__info-right-about-caption-desc"
                  >
                    {{
                    oneFilm.release_date
                    }}
                  </span>
                </div>
                <div class="header__info-right-about-caption">
                  {{isLang ? 'Бюджет:' : 'Budget:'}}
                  <span
                    class="header__info-right-about-caption-desc"
                  >{{ oneFilm.budget }}$</span>
                </div>
                <div class="header__info-right-about-caption">
                  {{isLang ? 'Описание:' : 'Description:'}}
                  <span
                    class="header__info-right-about-caption-desc header__info-right-about-caption-desc--f"
                  >{{ oneFilm.overview }}</span>
                </div>
              </div>
            </transition>
            <div class="header__info-right-block">
              <button
                class="header__info-right-block-btn"
                @click="showHideVideo = true"
                v-text="isLang ? 'Смотреть трейлер' : 'Watch trailer'"
              ></button>
              <button
                class="header__info-right-block-btn"
                @click="isOverview = !isOverview"
                v-text="isLang ? 'Описание' : 'View Info'"
              ></button>
              <i
                class="header__info-right-block-icon fas fa-heart"
                @click="addToFavorite(oneFilm.id)"
                :class="{
                  active: checkFavorite(oneFilm.id)
                }"
              ></i>
            </div>
          </div>
        </div>
        <div class="header__watch" v-if="showHideVideo" @keydown.esc="showHideVideoo()">
          <div class="header__watch-close" @click="showHideVideo = false">
            <i class="fas fa-times"></i>
          </div>
          <iframe
            class="header__watch-iframe"
            :src="youtube + oneFilm.key"
            frameborder="0"
            allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture"
            allowfullscreen
          ></iframe>
        </div>
      </div>
    </header>
    <div class="mainer">
      <ul class="categories">
        <li
          class="categories__item"
          :class="{ active: activeId === 1 }"
          @click="getTrand(1)"
          v-text="isLang ? 'В тренде' : 'Trending'"
        ></li>
        <li
          class="categories__item"
          :class="{ active: activeId === 2 }"
          @click="getTopRated(2)"
          v-text="isLang ? 'Топ рейтинг' : 'Top Rated'"
        ></li>
        <li
          class="categories__item"
          :class="{ active: activeId === 3 }"
          @click="getPopular(3)"
          v-text="isLang ? 'Популярное' : 'Popular'"
        ></li>
        <li
          class="categories__item"
          :class="{ active: activeId === 4 }"
          @click="getFavorite(4, favoriteList)"
          v-text="isLang ? 'Избранное' : 'Favorite'"
        ></li>
        <li
          class="categories__item"
          :class="{ active: activeId === 5 }"
          @click="getComingSoon(5)"
          v-text="isLang ? 'Уже скоро' : 'Coming Soon'"
        ></li>
        <li
          class="categories__item categories__item--genre"
          :class="{active: isGenre}"
          @click.self="isGenre = !isGenre"
        >
          {{isLang ? 'Жанр' : 'Genre'}}
          <ul class="categories__item-ul">
            <li
              class="categories__item-list"
              v-for="genre of genresList"
              :key="genre.id"
              @click="changeGenre(genre.id)"
            >{{ genre.name }}</li>
          </ul>
        </li>
      </ul>

      <ul class="movies" id="movies">
        <li class="movies__item" v-for="movie of moviesList" :key="movie.id">
          <div class="movies__item-card">
            <div class="movies__item-card-label" v-if="movie.popularity > 75">Popular</div>
            <div class="movies__item-card-picture">
              <img
                class="movies__item-card-picture-img"
                v-if="movie.poster_path"
                :src="imageUrl + movie.poster_path"
                :alt="movie.title"
              />
              <img
                v-else
                class="movies__item-card-picture-img"
                src="./assets/poster.jpg"
                :alt="movie.title"
              />
              <div class="movies__item-card-picture-back">
                <div
                  class="movies__item-card-picture-back-rait"
                >{{ numRound(movie.vote_average, 1) }}</div>
                <i
                  class="movies__item-card-picture-back-icon fas fa-heart"
                  @click="addToFavorite(movie.id)"
                  :class="{
                  active: checkFavorite(movie.id)
                }"
                ></i>
                <div class="movies__item-card-picture-back-title">{{ movie.title }}</div>
                <div
                  class="movies__item-card-picture-back-genres"
                >{{ getGenreNames(movie.genre_ids) }}</div>
                <div class="movies__item-card-picture-back-description">{{ movie.overview }}</div>
                <a
                  href="#info"
                  class="movies__item-card-picture-back-btn movies__item-card-picture-back-btn--pos"
                  @click="getOneMovie(movie.id)"
                  v-text="!isLang ? 'More details' : 'Подробнее'"
                ></a>
              </div>
            </div>
            <div class="movies__item-card-raiting">
              <div class="movies__item-card-raiting-stars">
                <span
                  class="movies__item-card-raiting-stars-bg"
                  :style="{ width: movie.vote_average * 10 + '%' }"
                ></span>
              </div>
              <div class="movies__item-card-raiting-points">{{ numRound(movie.vote_average, 1) }}</div>
            </div>
            <div class="movies__item-card-genres">{{ getGenreNames(movie.genre_ids) }}</div>
            <div class="movies__item-card-title">{{ movie.title }}</div>
          </div>
        </li>
        <li class="movies__item empty"></li>
        <li class="movies__item empty"></li>
        <li class="movies__item empty"></li>
      </ul>

      <div class="stepper" v-if="genreActiveId">
        <ul class="stepper__ul" @click="moreMovies">
          <li class="stepper__ul-li"></li>
          <li class="stepper__ul-li"></li>
          <li class="stepper__ul-li"></li>
        </ul>
      </div>

      <footer class="footer">
        <div class="footer__col">
          <div
            class="footer__col-lang"
            v-text="!isLang ? 'English' : 'Русский'"
            @click="isLang ? changeLang('en-US') : changeLang('ru-RU')"
          ></div>
        </div>
        <div class="footer__col">
          <div class="footer__col-logo">
            <a class="footer__col-logo-link" href="/">
              <b>film</b>corn
            </a>
          </div>
          <div class="footer__col-copy">
            Copyright © 2020
            <span class="footer__col-copy-up">
              <b>film</b>corn
            </span>. All Rights Reserved.
          </div>
        </div>
        <div class="footer__col">
          <div class="footer__col-social">
            <a
              href="https://www.facebook.com/"
              target="blank"
              class="footer__col-social-icons footer__col-social-icons--facebook"
            >
              <i class="fab fa-facebook-f"></i>
            </a>
            <a
              href="https://www.instagram.com/"
              target="blank"
              class="footer__col-social-icons footer__col-social-icons--instagram"
            >
              <i class="fab fa-instagram"></i>
            </a>
            <a
              href="https://twitter.com/explore"
              target="blank"
              class="footer__col-social-icons footer__col-social-icons--twitter"
            >
              <i class="fab fa-twitter"></i>
            </a>
            <a
              href="https://www.youtube.com/"
              target="blank"
              class="footer__col-social-icons footer__col-social-icons--youtube"
            >
              <i class="fab fa-youtube"></i>
            </a>
          </div>
        </div>
      </footer>
    </div>
  </div>
</template>
<script>
export default {
  name: "app",
  data() {
    return {
      imageUrl: "https://image.tmdb.org/t/p/w500",
      imageOriginalUrl: "https://image.tmdb.org/t/p/original",
      apiUrl: "https://api.themoviedb.org",
      apiKey: "4ae255cceb48db051b0e98f3c6cb5cba",
      lang: "ru-RU",
      genresList: [],
      genreActiveId: null,
      moviesList: [],
      currentPage: 1,
      oneFilm: { key: null },
      favoriteList: [],
      activeId: 1,
      isOverview: false,
      youtube: "https://www.youtube.com/embed/",
      showHideVideo: false,
      search: "",
      isSearch: false,
      isLang: true,
      isGenre: false
    };
  },
  methods: {
    getGenres() {
      fetch(
        this.apiUrl +
          "/3/genre/movie/list?api_key=" +
          this.apiKey +
          "&language=" +
          this.lang
      )
        .then(result => {
          return result.json();
        })
        .then(result => {
          this.genresList = result.genres;

          this.getTrand();
          this.changeId(1);
        });
    },
    getMovies(genreId) {
      fetch(
        this.apiUrl +
          "/3/discover/movie?api_key=" +
          this.apiKey +
          "&language=" +
          this.lang +
          "&sort_by=popularity.desc&include_adult=false&include_video=false&page=" +
          this.currentPage +
          "&with_genres=" +
          genreId
      )
        .then(result => {
          return result.json();
        })
        .then(result => {
          this.moviesList = this.moviesList.concat(result.results);
        });
    },
    numRound(num, point) {
      num = num / 2;
      return num.toFixed(point);
    },
    getGenreNames(ids) {
      let result = "";
      if (ids) {
        for (let id of ids) {
          const index = this.genresList.findIndex(genre => genre.id === id);
          result += this.genresList[index].name + ", ";
        }
        result = result.slice(0, -2);
      }
      return result;
    },
    moreMovies() {
      this.currentPage++;
      this.getMovies(this.genreActiveId);
    },
    changeGenre(id) {
      this.changeId(0);
      this.currentPage = 1;
      this.moviesList = [];
      this.genreActiveId = id;
      this.getMovies(id);
    },
    changeId(activeId) {
      this.activeId = activeId;
      this.isGenre = false;
    },
    getOneMovie(id) {
      this.isOverview = false;
      fetch(
        this.apiUrl +
          "/3/movie/" +
          id +
          "?api_key=" +
          this.apiKey +
          "&language=" +
          this.lang
      )
        .then(result => {
          return result.json();
        })
        .then(result => {
          let genre = [];
          for (let i of result.genres) {
            genre.push(i.id);
          }
          this.oneFilm = {
            title: result.title,
            genres: genre,
            vote_average: result.vote_average,
            poster: result.backdrop_path,
            id: result.id,
            original_title: result.original_title,
            budget: result.budget,
            release_date: result.release_date,
            overview: result.overview,
            id: id,
            key: this.watchNow(id)
          };
        });
    },
    addToFavorite(id) {
      if (this.favoriteList.length) {
        if (this.favoriteList.indexOf(id) < 0) {
          this.favoriteList.push(id);
        } else {
          this.favoriteList.splice(this.favoriteList.indexOf(id), 1);
        }
      } else {
        this.favoriteList.push(id);
      }
      localStorage.setItem("favorites", JSON.stringify(this.favoriteList));
    },
    checkFavorite(id) {
      return this.favoriteList.indexOf(id) > -1;
    },
    getTrand(activeId) {
      this.genreActiveId = "";
      fetch(
        this.apiUrl +
          "/3/trending/movie/day?api_key=" +
          this.apiKey +
          "&language=" +
          this.lang
      )
        .then(result => {
          return result.json();
        })
        .then(result => {
          this.moviesList = [];
          this.moviesList = result.results;
        });
      this.changeId(activeId);
    },
    getPopular(activeId) {
      this.genreActiveId = "";
      fetch(
        this.apiUrl +
          "/3/movie/popular?api_key=" +
          this.apiKey +
          "&language=" +
          this.lang +
          "&sort_by=popularity.desc&page=" +
          this.currentPage
      )
        .then(result => {
          return result.json();
        })
        .then(result => {
          this.moviesList = [];
          this.moviesList = this.moviesList.concat(result.results);
        });
      this.changeId(activeId);
    },
    getTopRated(activeId) {
      this.genreActiveId = "";
      fetch(
        this.apiUrl +
          "/3/movie/top_rated?api_key=" +
          this.apiKey +
          "&page=1" +
          "&language" +
          this.lang
      )
        .then(result => {
          return result.json();
        })
        .then(result => {
          this.moviesList = [];
          this.moviesList = result.results;
        });
      this.changeId(activeId);
    },
    getComingSoon(activeId) {
      this.genreActiveId = "";
      fetch(
        this.apiUrl +
          "/3/movie/upcoming?api_key=" +
          this.apiKey +
          "&language" +
          this.lang +
          "&page=1"
      )
        .then(result => {
          return result.json();
        })
        .then(result => {
          this.moviesList = [];
          this.moviesList = result.results;
        });
      this.changeId(activeId);
    },
    getFavorite(activeId, list) {
      this.genreActiveId = "";
      this.moviesList = [];
      this.changeId(activeId);
      for (let movie of list) {
        fetch(
          this.apiUrl +
            "/3/movie/" +
            movie +
            "?api_key=" +
            this.apiKey +
            "&language=" +
            this.lang
        )
          .then(result => {
            return result.json();
          })
          .then(result => {
            let genre = [];
            for (let i of result.genres) {
              genre.push(i.id);
            }
            result.genre_ids = genre;
            this.moviesList = this.moviesList.concat(result);
          });
      }
    },
    watchNow(id) {
      fetch(
        this.apiUrl +
          "/3/movie/" +
          id +
          "/videos?api_key=" +
          this.apiKey +
          "&language=" +
          this.lang
      )
        .then(result => {
          return result.json();
        })
        .then(result => {
          if (result.results.length) {
            this.oneFilm.key = result.results[0].key;
          } else {
            this.oneFilm.key = "L1Snj1Pt-Hs";
          }
        });
    },
    searchMovie() {
      fetch(
        this.apiUrl +
          "/3/search/movie?api_key=" +
          this.apiKey +
          "&language=" +
          this.lang +
          "&query=" +
          this.search +
          "&page=1" +
          "&include_adult=false"
      )
        .then(result => {
          return result.json();
        })
        .then(result => {
          this.moviesList = [];
          this.moviesList = result.results;
          this.moviesList.sort((a, b) =>
            a.popularity > b.popularity ? -1 : 1
          );
        });
      this.isSearch = false;
      this.search = "";
      const toMovies = this.$el.querySelector("#movies");
      toMovies.scrollIntoView({ behavior: "smooth" });
    },
    changeLang(lang) {
      this.lang = lang;
      this.getGenres();
      this.isLang = !this.isLang;
      this.getOneMovie(this.oneFilm.id);
    },
    showHideVideoo() {
      this.showHideVideo = false;
    }
  },
  created: function() {
    this.getGenres();
    this.getOneMovie(68735);
    this.favoriteList = JSON.parse(localStorage.getItem("favorites")) || [];
    document.addEventListener("keydown", this.showHideVideoo);
  },
  beforeDestroy: function() {
    document.removeEventListener("keydown", this.showHideVideoo);
  },
  computed: {
    getUrl: function() {
      if (this.oneFilm.poster) {
        return this.imageOriginalUrl + this.oneFilm.poster;
      } else {
        return "./dist/header-bg.jpg";
      }
    }
  }
};
</script>

<style lang="scss">
@import url("https://fonts.googleapis.com/css2?family=Montserrat:wght@200;400;500;600;700&display=swap");
@import "./scss/style";
</style>
