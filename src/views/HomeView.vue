<template>
  <div class="job-feed">
    <v-container v-if="jobListings.length">
      <div class="d-flex">
        <v-menu offset-y>
          <template v-slot:activator="{ on, attrs }">
            <v-btn text color="black" dark v-bind="attrs" v-on="on">
              Filter by Position
              <v-icon class="ml-2">mdi-arrow-down</v-icon>
            </v-btn>
          </template>
          <v-card
            height="500"
            class="pr-4"
            style="overflow-x: hidden; overflow-y: auto"
          >
            <v-treeview
              selected-color="red"
              @input="selected"
              selectable
              v-model="selectedCheckbox"
              :items="positionFunctionsArray"
            ></v-treeview>
          </v-card>
        </v-menu>
        <v-spacer></v-spacer>
        <v-menu offset-y>
          <template v-slot:activator="{ on, attrs }">
            <v-btn text color="black" dark v-bind="attrs" v-on="on">
              {{ pageSizeText }}
              <v-icon class="ml-2">mdi-arrow-down</v-icon>
            </v-btn>
          </template>
          <v-list>
            <v-list-item link @click="changePageSize(5)">
              <v-list-item-title>5 per page</v-list-item-title>
            </v-list-item>
            <v-list-item link @click="changePageSize(25)">
              <v-list-item-title>25 per page</v-list-item-title>
            </v-list-item>
            <v-list-item link @click="changePageSize(250)">
              <v-list-item-title>Display all</v-list-item-title>
            </v-list-item>
          </v-list>
        </v-menu>
      </div>
      <v-card
        v-for="(data, index) in jobListings"
        :key="index"
        class="mt-4 rounded-lg"
      >
        <v-card-title>{{ data.job.title }}</v-card-title>
        <v-card-text
          >{{ data.job.unit.display_name }} <v-icon>mdi-circle-small</v-icon>
          {{ data.job.position_function.name_en }}
          <v-icon>mdi-circle-small</v-icon> {{ data.job.due_date }}</v-card-text
        >
      </v-card>
    </v-container>
    <div v-else class="loader">Loading</div>
  </div>
</template>

<script>
export default {
  name: "HomeView",
  data: () => ({
    apiBaseUrl: "",
    pageSize: 0,
    pageSizeText: "",
    jobListings: [],
    positionFunctionsArray: [],
    selectedCheckbox: [],
  }),
  components: {},
  methods: {
    async fetchAllJobs() {
      const allJobs = await fetch(
        `${this.apiBaseUrl}job/listings/?include_open=False&page=1&page_size=${this.pageSize}&use_mojob_feed_filter=True&use_pagination=True`
      );

      const { results } = await allJobs.json();
      this.jobListings = results;
    },
    async fetchJobPositions() {
      const positionFunctions = await fetch(
        `${this.apiBaseUrl}job/position-functions/?page_size=100`
      );
      const { results } = await positionFunctions.json();
      this.positionFunctionsArray = results;
    },
    async selected() {
      const data = this.selectedCheckbox;
      const positionFunctionsQuery = data.toString();

      const filteredJobs = await fetch(
        `${this.apiBaseUrl}job/listings/?include_open=False&page=1&page_size=${this.pageSize}&use_mojob_feed_filter=True&use_pagination=True&position_functions=${positionFunctionsQuery}`
      );
      const { results } = await filteredJobs.json();
      this.jobListings = results;
    },
    async changePageSize(res) {
      this.pageSize = res;
      const url = this.apiBaseUrl;
      const allJobs = await fetch(
        `${url}job/listings/?include_open=False&page=1&page_size=${res}&use_mojob_feed_filter=True&use_pagination=True`
      );
      const { results } = await allJobs.json();
      this.jobListings = results;
      if (res >= 250) {
        this.pageSizeText = `Display All`;
      } else {
        this.pageSizeText = `${res} Per Page`;
      }
    },
  },

  mounted() {
    this.apiBaseUrl = "https://test-api.mojob.io/public/";
    this.pageSize = 5;
    this.pageSizeText = "5 Per Page";
    this.fetchAllJobs();
    this.fetchJobPositions();
  },
};
</script>

<style lang="scss">
.custom__scrollable-card {
  max-height: 500;
  overflow-x: hidden;
  overflow-y: auto;
}
.loader {
  height: 100vh;
}
.loader,
.loader:before,
.loader:after {
  background: #46bbb3;
  -webkit-animation: load1 1s infinite ease-in-out;
  animation: load1 1s infinite ease-in-out;
  width: 1em;
  height: 4em;
}
.loader {
  color: #46bbb3;
  text-indent: -9999em;
  margin: 20% auto;
  position: relative;
  font-size: 11px;
  -webkit-transform: translateZ(0);
  -ms-transform: translateZ(0);
  transform: translateZ(0);
  -webkit-animation-delay: -0.16s;
  animation-delay: -0.16s;
}
.loader:before,
.loader:after {
  position: absolute;
  top: 0;
  content: "";
}
.loader:before {
  left: -1.5em;
  -webkit-animation-delay: -0.32s;
  animation-delay: -0.32s;
}
.loader:after {
  left: 1.5em;
}
@-webkit-keyframes load1 {
  0%,
  80%,
  100% {
    box-shadow: 0 0;
    height: 4em;
  }
  40% {
    box-shadow: 0 -2em;
    height: 5em;
  }
}
@keyframes load1 {
  0%,
  80%,
  100% {
    box-shadow: 0 0;
    height: 4em;
  }
  40% {
    box-shadow: 0 -2em;
    height: 5em;
  }
}
</style>