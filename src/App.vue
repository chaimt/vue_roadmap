<template>
  <div id="app">
    <form action="#" @submit.prevent="getIssues">
      <div class="form-group">
        <input
                type="text"
                placeholder="owner/repo Name"
                v-model="repository"
                class="col-md-2 col-md-offset-5"
        >
        <b-dropdown id="ddProfile"
                    name="ddProfile"
                    v-model="ddProfileVm.ddSelectedOption"
                    text="Select Item"
                    variant="primary"
                    class="m-md-2" v-on:change="changeItem">
          <b-dropdown-item disabled value="0">Select an Item</b-dropdown-item>
          <b-dropdown-item v-for="option in ddProfileVm.options"
                           :key="option.value"
                           :value="option.value"
                           @click="ddProfileVm.ddSelectedOption = option.value">
            {{option.text}}
          </b-dropdown-item>
        </b-dropdown>
        <span>Selected: {{ ddProfileVm.ddSelectedOption }}</span>

      </div>
    </form>
    <div class="alert alert-info" v-show="loading">Loading...</div>
    <div class="alert alert-danger" v-show="errored">An error occured</div>
    <chart :issues="issues"></chart>
  </div>
</template>

<script>
  import moment from "moment";
  import axios from "axios";
  import Chart from "./components/Chart.vue";
  export default {
    name: "app",
    components: {
      Chart
    },
    data() {
      return {
        loading: false,
        errored: false,
        issues: [],
        repository: "",
        startDate: null,
        ddProfileVm: {
          originalValue: [],
          ddSelectedOption: "Profiles",
          disabled: false,
          readonly: false,
          visible: true,
          color: "",
          options: [
            {
              "value": "tree_profile.json",
              "text": "Profiles"
            },
            {
              "value": "tree_profile_application.json",
              "text": "Profile Application"
            },
            {
              "value": "tree_seniority.json",
              "text": "Seniority Profile"
            }
          ]
        }
      };
    },
    methods: {
      changeItem: async function () {
        //grab some remote data
        try {
          let response = await this.$http.get('https://www.example.com/api/' + this.ddTestVm.ddTestSelectedOption + '.json');
          console.log(response.data);
          this.someOtherProperty = response.data;
        } catch (error) {
          console.log(error)
        }
      },
      getDateRange() {
        const startDate = moment().subtract(6, "days");
        const endDate = moment();
        const dates = [];
        while (startDate.isSameOrBefore(endDate)) {
          dates.push({
            day: startDate.format("MMM Do YY"),
            issues: 0
          });
          startDate.add(1, "days");
        }
        return dates;
      },
      getIssues() {
        this.loading = true;
        this.startDate = moment()
                .subtract(6, "days")
                .format("YYYY-MM-DD");
        axios
                .get(
                        `https://api.github.com/search/issues?q=repo:${
                                this.repository
                        }+is:issue+is:open+created:>=${this.startDate}`,
                        { params: { per_page: 100 } }
                )
                .then(response => {
                  this.loading = false;
                  const payload = this.getDateRange();
                  response.data.items.forEach(item => {
                    const key = moment(item.created_at).format("MMM Do YY");
                    const obj = payload.filter(o => o.day === key)[0];
                    obj.issues += 1;
                  });
                  this.issues = payload;
                })
                .catch(error => {
                  this.errored = true;
                  console.error(error);
                })
                .finally(() => (this.loading = false));
      }
    }
  };
</script>

<style>
  #app {
    font-family: "Avenir", Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }
  .bar {
    fill: #319bbe;
  }
</style>