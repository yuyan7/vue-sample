<template>
  <div>
    <infinite-loading v-if="hasNext" @infinite="infiniteHandler" spinner="spiral" direction="top">
      <div slot="no-more">No more</div>
      <div slot="no-results">No results</div>
    </infinite-loading>

    <ul class="list">
      <li class="item" v-for="(item, index) in items" :key="index">{{ item }} </li>
    </ul>
  </div>
</template>

<script>
import InfiniteLoading from 'vue-infinite-loading';

export default {
  name: 'WareHouse',
  components: {
    InfiniteLoading
  },
  data() {
    return {
      items: [],
      startPage: 0,
      endPage: 0,
      totalPages: 0,
      pageSize: 20,
      initialized: false
    }
  },
  computed: {
    hasNext() {
      return this.initialized && this.totalPages > this.endPage;
    }
  },
  mounted() {
    window.addEventListener('scroll', () => this.scroll());

    const urlParams = new URLSearchParams(window.location.search);
    const page = urlParams.get('page');

    if (page) {
      // URLパラメータでページ番号が指定された場合、指定ページから表示
      this.startPage = parseInt(page, 10);
      this.endPage = parseInt(page, 10);
    } else {
      this.startPage = 1;
      this.endPage = 1;
    }

    this.getItems(null, this.startPage, false);
  },
  methods: {
    infiniteHandler($state) {
      if (this.endPage >= this.totalPages) {
        $state.complete();
      } else {
        this.getItems($state, this.endPage + 1, true);
      }
    },
    getItems($state, page) {
      setTimeout(() => {
        let data = [];
        for (let i = 1; i <= this.pageSize; i++) {
          data.push(`item${page}-${i}`);
        }
        this.totalPages = 10;

        this.items = this.items.concat(data);
        this.endPage = page;

        if ($state) $state.loaded();

        this.$nextTick(() => {
          this.initialized = true;
        });
      }, 1000);
    },
    scroll() {
      let scrollPos = window.pageYOffset || document.documentElement.scrollTop;
      let windowHeight = window.outerHeight;
      let page = Math.ceil((scrollPos + 0.5 * windowHeight) / 40 / this.pageSize) + (this.startPage - 1);
      window.history.replaceState(null, null, `/?page=${page}`);
    }
  }
};
</script>
