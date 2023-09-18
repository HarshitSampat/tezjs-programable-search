<template>
  <div class="m-3 p-3 container mx-auto w-full lg:w-3/4">
    <form
      class="flex text-slate-600"
      v-on:submit.prevent="onSubmit"
      @submit="getData(1)"
    >
      <!-- <label class="mr-2 text-[#666] text-base flex text-center   hidden md:block">Search on Radixweb: </label> -->
      <span class="icon">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          fill="none"
          viewBox="0 0 24 24"
          stroke-width="1.5"
          stroke="currentColor"
          class="w-4 h-4"
        >
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            d="M21 21l-5.197-5.197m0 0A7.5 7.5 0 105.196 5.196a7.5 7.5 0 0010.607 10.607z"
          />
        </svg>
      </span>
      <input
        type="search"
        class="
          px-3
          py-2
          placeholder-slate-400
          bg-white
          text-sm
          focus:outline-none
          border border-[#dfe1e5]
          w-full
          sm:w-3/4
          md:w-1/2
          lg:w-2/5
          rounded-full
          input-field
        "
        placeholder="Search on Radixweb"
        v-model="searchInput"
        id="search"
      />
    </form>

    <div v-if="response.items" class="mt-4 relative">
      <div
        v-if="loader"
        class="bg-white opacity-50 w-full h-full z-10 absolute"
      ></div>
      <div
        v-for="result in response.items"
        :key="result.title"
        class="py-3 clear-both relative h-full"
      >
        <div>
          <a
            :href="result.formattedUrl"
            class="text-[#1155CC] text-base overflow-hidden h-auto"
            v-html="result.htmlTitle"
          ></a>
          <div class="text-[#093] text-xs">
            <span>{{
              result.formattedUrl
                .replace("https://", "")
                .replace("...", "")
                .replace(/(\/)$/, "")
                .replace("//", "/")
                .replaceAll("/", " > ")
            }}</span>
          </div>
          <div>
            <div v-if="result.pagemap">
              <div
                v-for="img in result.pagemap.cse_thumbnail"
                :key="img.src"
                class="w-16"
              >
                <img
                  v-tez-src="img.src"
                  style="max-width: 60px; max-height: 120px"
                  class="float-left py-1 block pr-2 mx-auto text-center"
                  alt="my"
                />
              </div>
            </div>
            <div class="text-xs" v-html="result.htmlSnippet"></div>
          </div>
        </div>
      </div>
    </div>
    <div
      v-if="response.queries && !response.items"
      class="
        inline-block
        bg-[#FFF4C2]
        text-[#333]
        border border-[#FFCC33]
        my-3
        p-1
        text-sm
      "
    >
      No Results
    </div>
    <!-- pagination -->
    <ul
      class="flex gap-2 justify-center m-3 text-[#666] text-sm clear-both"
      v-if="response.items"
    >
      <!-- visible button start -->
      <li v-for="page in this.pages()" :key="page.count">
        <button
          type="button"
          @click="this.onPageChange(page)"
          v-if="
            response.queries.nextPage ||
            this.response.queries.request[0].startIndex >= page.name
          "
          :class="{ active: isPageActive(page.count) }"
        >
          {{ page.count }}
        </button>
      </li>
    </ul>
    <footer>
      <div class="text-sm grid grid-cols-12" v-if="response.items">
        <div
          class="
            text-[#15c]
            flex
            gap-1
            col-span-12
            md:col-span-6
            mx-auto
            md:mx-0
            my-2
          "
        >
          <span>
            <svg
              xmlns="http://www.w3.org/2000/svg"
              fill="none"
              viewBox="0 0 24 24"
              stroke-width="1.5"
              stroke="currentColor"
              class="w-4 h-4"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                d="M21 21l-5.197-5.197m0 0A7.5 7.5 0 105.196 5.196a7.5 7.5 0 0010.607 10.607z"
              />
            </svg> </span
          ><span>Search for </span
          ><span
            class="
              text-ellipsis
              overflow-hidden
              font-bold
              whitespace-nowrap
              inline-block
              align-middle
            "
            style="max-width: 200px"
            >{{ response.queries.request[0].searchTerms }}</span
          ><span> on Radixweb</span>
        </div>
        <div
          class="
            text-[#666]
            col-span-12
            md:col-span-6
            my-2
            text-center
            md:text-end
          "
        >
          ENHANCED BY tezjs
        </div>
      </div>
      <div class="text-sm text-[#666] text-center p-4">©2022 tezjs</div>
    </footer>
  </div>
</template>
<script>
import axios from "axios";

export default {
  data() {
    return {
      searchInput: "",
      response: [],
      pageStart: 1,
      loader: false,
      currentPage: 1,
    };
  },
  mounted(){
    const params =(new URL(location)).searchParams;
    const q = params.get("q");
    if(q){
      this.onBackPage(q);
    }
  },
  methods: {
    pages() {
      const range = [];
      let count = 1;
      for (let i = 1; i <= 100; i = i + 10) {
        range.push({
          name: i,
          count,
        });
        count = count + 1;
      }
      return range;
    },
    onBackPage(params){
      this.searchInput=params;
      this.getData(1);      
    },
    onPageChange(page) {
      this.currentPage = page.count;
      this.getData(page.name);
    },
    isPageActive(page) {
      return this.currentPage === page;
    },
    async getData(start) {
      let params = {
        key: import.meta.env.VITE_KEY,
        cx: import.meta.env.VITE_CX,
        q: this.searchInput,
        start,
      };
      this.loader = true;
      let res = await axios.get("https://www.googleapis.com/customsearch/v1", {
        params,
      });
      const url = new URL(window.location);
      url.searchParams.set("q", this.searchInput);
      window.history.pushState({}, "", url);
      this.response = res.data;
      this.loader = false;
    },
  },
};
</script>
<style>
.active {
  color: #1a73e8;
}

.icon {
  padding: 10px;
  position: absolute;
}

.input-field {
  padding-left: 30px;
}
</style>