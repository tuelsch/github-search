<template>
  <div class="user-search">
    <div class="user-search__input-wrapper">
      <label for="user-search-input" class="sr-only">search a github user by name</label>
      <img class="user-search__icon" src="@/assets/search-icon.svg" alt="search icon">
      <input
        id="user-search-input"
        class="user-search__input"
        type="text"
        placeholder="type a github username"
        v-model="query"
        @input="fetchUsers"
        @keydown.up.prevent="selectUpper"
        @keydown.down.prevent="selectLower"
        @keydown.enter="enterKey"
      >
      <div class="user-search__autocomplete">
        <span class="user-search__input-mirror">{{query}}</span>
        <span class="user-search__ghost-text"></span>
      </div>
    </div>

    <p v-if="errorMessage" class="error">Error: {{ errorMessage }}</p>

    <div
      class="user-search__dropdown"
      :class="{ show: showDropdown }"
    >
      <header>github users</header>
      <ul
        class="user-search__results"
        ref="resultsContainer"
      >
        <li
          class="user-search__result"
          v-for="(user, index) in users"
          :key="user.id"
          :class="{ 'active': isActive(index) }"
          @mouseenter="select(index)"
        >
          <a
            class="user-search__profile-link"
            :href="user.html_url"
          >
            <img
              class="user-search__avatar"
              alt="github user profile picture"
              :src="user.avatar_url"
            >
            <span class="user-search__username">
              <span class="bold"></span>
              {{user.login}}
            </span>
          </a>
        </li>
      </ul>
    </div>
  </div>
</template>

<script lant="ts">
  import { Component, Vue } from 'vue-property-decorator';

  @Component
  export default class UserSearch extends Vue {
    query = "";
    users = [];
    errorMessage = "";
    activeItem = 0;

    async fetchUsers () {
      // Exit conditions
      if (this.query.length < 2) {
        this.users = [];
        this.errorMessage = '';
        this.activeItem = 0;
        return;
      }

      const url = `https://api.github.com/search/users?q=${this.query}`;

      try {
        this.errorMessage = '';
        const res = await fetch(url);
        if (!res.ok) { throw new Error(res.statusText); }
        const jsonResponse = await res.json();
        this.users = jsonResponse.items;
      } catch (error) {
        this.users = [];
        this.errorMessage = error.message;
      }
    }

    isActive (index) {
      return index === this.activeItem;
    }

    get showDropdown () {
      return !!this.users.length;
    }

    selectUpper () {
      if (this.activeItem - 1 >= 0) {
        this.activeItem = this.activeItem - 1;
        this.scrollToItem();
      }
    }

    selectLower () {
      if (this.activeItem + 1 < this.users.length) {
        this.activeItem = this.activeItem + 1;
        this.scrollToItem();
      }
    }

    select (index) {
      this.activeItem = index;
    }

    scrollToItem () {
      const scrollTop = this.$refs.resultsContainer.scrollTop;
      const height = this.$refs.resultsContainer.offsetHeight;
      const item = this.$refs.resultsContainer.children[this.activeItem];

      if (scrollTop > item.offsetTop) {
        this.$refs.resultsContainer.scrollTop = item.offsetTop;
      } else if (scrollTop < item.offsetTop - height + item.offsetHeight) {
        this.$refs.resultsContainer.scrollTop = item.offsetTop - height + item.offsetHeight;
      }
    }

    enterKey () {
      if (this.users.length) {
        window.location.href = this.users[this.activeItem].html_url;
      }
    }
  }
</script>

<style lang="scss" scoped>
  $base-space: 8px;
  input[type="text"] {
    font: inherit;
  }

  ul {
    list-style: none;
    margin: 0;
    padding-left: 0;
  }

  .bold {
    font-weight: bold;
  }

  .active {
    background: lavender;
  }

  .error {
    color: red;
  }

  .user-search {
    position: relative;

    &__input-wrapper {
      position: relative;
      flex-grow: 1;
    }

    &__icon {
      position: absolute;
      left: $base-space * 1.5;
      top: 50%;
      transform: translate(0, -50%);
      pointer-events: none;
    }

    &__input {
      border: 1px solid lightgrey;
    }

    &__input,
    &__input-mirror {
      display: block;
      width: 100%;
      padding: $base-space $base-space * 1.5 $base-space $base-space * 3 + 24;
    }

    &__autocomplete {
      position: absolute;
      top: 0;
      left: 0;
      pointer-events: none;
    }

    &__input-mirror {
      opacity: 0;
    }

    &__dropdown {
      position: absolute;
      top: 100%;
      width: 100%;
      font-size: 0.8em;
      opacity: 0;
      visibility: hidden;
      transition: opacity 0.3s, visibility 0s 0.3s;
      box-shadow: 0 5px 10px rgba(0,0,0,0.2);

      &.show {
        opacity: 1;
        visibility: visible;
        transition: opacity 0.3s, visibility 0s 0s;
      }

      header {
        background: whitesmoke;
        color: darkgrey;
        text-transform: uppercase;
        font-weight: bold;
        text-align: left;
        padding: $base-space * 1.5;
      }
    }

    &__results {
      position: relative;
      max-height: 300px;
      overflow: auto;
    }

    &__profile-link {
      display: block;
      display: flex;
      align-items: center;
      padding: $base-space $base-space * 1.5;
      border-bottom: 1px solid lightgrey;
      color: inherit;
      text-decoration: none;
    }

    &__avatar {
      width: 45px;
    }

    &__username {
      margin-left: $base-space * 2;
    }
  }

  .sr-only {
    border: 0 !important;
    clip: rect(1px, 1px, 1px, 1px) !important; /* 1 */
    -webkit-clip-path: inset(50%) !important;
      clip-path: inset(50%) !important;  /* 2 */
    height: 1px !important;
    margin: -1px !important;
    overflow: hidden !important;
    padding: 0 !important;
    position: absolute !important;
    width: 1px !important;
    white-space: nowrap !important;            /* 3 */
  }
</style>
