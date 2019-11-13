<template>
    <div id="app">
        <form action="#" @submit.prevent="getIssues">
            <div class="form-group">
                <p>
                    <span>Selected Employee: </span>
                    <b-dropdown id="ddEmployee"
                                name="ddEmployee"
                                v-model="ddEmployeeVm.ddSelectedOption"
                                text="Select Item"
                                variant="primary"
                                class="m-md-2" v-on:change="ddEmployeeVmChangeItem">
                        <b-dropdown-item disabled value="0">Select a Profile</b-dropdown-item>
                        <b-dropdown-item v-for="option in ddEmployeeVm.options"
                                         :key="option.value"
                                         :value="option.value"
                                         @click="ddEmployeeVm.ddSelectedOption = option.value">
                            {{option.text}}
                        </b-dropdown-item>
                    </b-dropdown>
                </p>
                <p>
                    <span>Language: </span>
                    <b-dropdown id="ddLanguage" text="Language" class="m-md-2">
                        <b-dropdown-item>Java</b-dropdown-item>
                        <b-dropdown-item>Kotlin</b-dropdown-item>
                        <b-dropdown-item>Scala</b-dropdown-item>
                        <b-dropdown-divider></b-dropdown-divider>
                        <b-dropdown-item>Python</b-dropdown-item>
                        <b-dropdown-divider></b-dropdown-divider>
                        <b-dropdown-item>JavaScript</b-dropdown-item>
                    </b-dropdown>
                </p>

                <p>
                    <span>Selected Profile: </span>
                    <b-dropdown id="ddProfile"
                                name="ddProfile"
                                v-model="ddProfileVm.ddSelectedOption"
                                text="Select Item"
                                variant="primary"
                                class="m-md-2" v-on:change="ddProfileVmChangeItem">
                        <b-dropdown-item disabled value="0">Select a Profile</b-dropdown-item>
                        <b-dropdown-item v-for="option in ddProfileVm.options"
                                         :key="option.value"
                                         :value="option.value"
                                         @click="ddProfileVm.ddSelectedOption = option.value">
                            {{option.text}}
                        </b-dropdown-item>
                    </b-dropdown>
                </p>

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
                },
                ddEmployeeVm: {
                    originalValue: [],
                    ddSelectedOption: "chaim_turkel",
                    disabled: false,
                    readonly: false,
                    visible: true,
                    color: "",
                    options: [
                        {
                            "value": "chaim_turkel",
                            "text": "Chaim Turkel"
                        },
                        {
                            "value": "yoav_nordman",
                            "text": "Yoav Nordman"
                        },
                        {
                            "value": "haim_cohen",
                            "text": "Haim Cohen"
                        }
                    ]
                }
            };
        },
        methods: {
            ddProfileVmChangeItem: async function () {
                //grab some remote data
                try {
                    let response = await this.$http.get('https://www.example.com/api/' + this.ddTestVm.ddTestSelectedOption + '.json');
                    console.log(response.data);
                    this.someOtherProperty = response.data;
                } catch (error) {
                    console.log(error)
                }
            },
            ddEmployeeVmChangeItem: async function () {
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
                        {params: {per_page: 100}}
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