<template>
    <div>
        <div class="row justify-content-center">
            <div class="col-9 col-md-9 col-lg-9">
                <form class="card card-sm" onsubmit="event.preventDefault();">
                    <div class="card-body row no-gutters align-items-center p-0">
                        <div class="col-auto">
                            <i class="fas fa-search h4 text-body"></i>
                        </div>
                        <!--end of col-->
                        <div class="col">
                            <input class="form-control form-control-sm form-control-borderless" type="search"
                                placeholder="Search topics or keywords" v-model="this.c_name">
                        </div>
                        <!--end of col-->
                        <div class="col-auto">
                            <button class="btn btn-sm btn-success btn-search" @click="fncSearch">Search</button>
                        </div>
                        <!--end of col-->
                    </div>
                </form>
            </div>
            <!--end of col-->
        </div>
        <div class="row">
            <div class="float-left mt-5">
                <label class="d-inline-block mr-1" for="show_entries" style="font-size: 17px;">Show </label>
                <!-- added the amount of entries as an argument to the showEntries function below -->
                <select @change="showEntries(show_entries)" v-model="show_entries"
                    class="form-control form-control-sm d-inline-block" id="show_entries" style="width: auto;">
                    <option value="25" selected>25</option>
                    <option value="50">50</option>
                    <option value="100">100</option>
                    <option value="all">All</option>
                </select>
                <label class="d-inline-block ml-1" for="show_entries" style="font-size: 17px;">entries</label>
                <!-- {{ totalshow }} -->
            </div>
        </div>
        <br>
        <table class="table table-bordered">
            <thead class="table-header">
                <tr>
                    <th class="table-bg" width="10">#. </th>
                    <th class="table-bg" width="10">Flags </th>
                    <th class="table-bg" width="30" v-on:click="doSort('name.official')">Country Name
                        <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor"
                            class="bi bi-arrow-down-up" viewBox="0 0 16 16">
                            <path fill-rule="evenodd"
                                d="M11.5 15a.5.5 0 0 0 .5-.5V2.707l3.146 3.147a.5.5 0 0 0 .708-.708l-4-4a.5.5 0 0 0-.708 0l-4 4a.5.5 0 1 0 .708.708L11 2.707V14.5a.5.5 0 0 0 .5.5m-7-14a.5.5 0 0 1 .5.5v11.793l3.146-3.147a.5.5 0 0 1 .708.708l-4 4a.5.5 0 0 1-.708 0l-4-4a.5.5 0 0 1 .708-.708L4 13.293V1.5a.5.5 0 0 1 .5-.5" />
                        </svg> ({{ sort.desc ? 'desc' : 'asc' }})
                    </th>
                    <th class="table-bg" width="30">2 character Country Code</th>
                    <th class="table-bg" width="30">3 character Country Code</th>
                    <th class="table-bg" width="30">Native Country Name</th>
                    <th class="table-bg" width="30">Alternative Country Name</th>
                    <th class="table-bg" width="30">Country Calling Codes</th>
                </tr>
            </thead>
            <tbody v-if="apiData != null">
                <!--  v-if="index >= this.startIndex && index < this.endIndex" -->
                <tr v-for="(item, idx) in sortedData" class="text-justify text-wrap" >
                    <td>{{ idx + 1 }}</td>
                    <th scope="text-center"><img :src="item.flags.png" alt="" width="50"></th>
                    <td @click="fncShowPopup(item.name.official)">{{ item.name.official }}</td>
                    <td>{{ item.cca2 }}</td>
                    <td>{{ item.cca3 }}</td>
                    <td>{{ item.name.nativeName }}</td>
                    <td>{{ item.altSpellings }}</td>
                    <td>{{ item.idd.root }}{{ item.idd.suffixes }}</td>
                </tr>
            </tbody>
            <tbody v-else>
                <tr>
                    <td colspan="8" class="text-center">No data to show</td>
                </tr>
            </tbody>
        </table>
        <div class="row">
            <div class="col-lg-6 col-md-6 col-sm-6">
                <div class="pagination float-left mt-1">
                    <p>Showing {{ startIndex + 1 }} to {{ endIndex > totalshow ? totalshow : endIndex }}
                        of {{ totalshow }} entries</p>
                </div>
            </div>

            <div class="col-lg-6 col-md-6 col-sm-6">

                <nav aria-label="Page navigation example" class="pagination float-right mt-1" style="float: right;">
                    <ul class="pagination">
                        <li class="page-item"><a class="page-link btn-sm" @click="previous()">Previous</a></li>
                        <li class="page-item"><a class="page-link bg-primary text-light btn-sm"
                                v-for="num in Math.ceil(totalshow / 25)" @click="pagination(num)"
                                style="display: inline-block">{{ num }}</a></li>
                        <li class="page-item"><a class="page-link btn-sm" @click="nextCategory()">Next</a></li>
                    </ul>
                </nav>
            </div>
        </div>


        <!-- <transition name="modal"> -->
        <div v-if="isOpen">
            <div class="overlay" @click.self="isOpen = false;">
                <div class="moda">
                    <h1>Show all single record by country name</h1>
                    <table class="table table-bordered">
                        <thead class="table-header">
                            <tr>
                                <th class="table-bg-p" width="10">#. </th>
                                <th class="table-bg-p" width="10">Flags </th>
                                <th class="table-bg-p" width="30" >Country Name
                                    <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor"
                                        class="bi bi-arrow-down-up" viewBox="0 0 16 16">
                                        <path fill-rule="evenodd"
                                            d="M11.5 15a.5.5 0 0 0 .5-.5V2.707l3.146 3.147a.5.5 0 0 0 .708-.708l-4-4a.5.5 0 0 0-.708 0l-4 4a.5.5 0 1 0 .708.708L11 2.707V14.5a.5.5 0 0 0 .5.5m-7-14a.5.5 0 0 1 .5.5v11.793l3.146-3.147a.5.5 0 0 1 .708.708l-4 4a.5.5 0 0 1-.708 0l-4-4a.5.5 0 0 1 .708-.708L4 13.293V1.5a.5.5 0 0 1 .5-.5" />
                                    </svg> ({{ sort.desc ? 'desc' : 'asc' }})
                                </th>
                                <th class="table-bg-p" width="30">2 character Country Code</th>
                                <th class="table-bg-p" width="30">3 character Country Code</th>
                                <th class="table-bg-p" width="30">Native Country Name</th>
                                <th class="table-bg-p" width="30">Alternative Country Name</th>
                                <th class="table-bg-p" width="30">Country Calling Codes</th>
                            </tr>
                        </thead>
                        <tbody v-if="singlerow.length >0">
                            <!--  v-if="index >= this.startIndex && index < this.endIndex" -->
                            <tr v-for="(item, idex) in singlerow" class="text-justify text-wrap">
                                <td>{{ idex + 1 }}</td>
                                <th scope="text-center"><img :src="item.flags.png" alt="" width="50"></th>
                                <td>{{ item.name.official }}</td>
                                <td>{{ item.cca2 }}</td>
                                <td>{{ item.cca3 }}</td>
                                <td>{{ item.name.nativeName }}</td>
                                <td>{{ item.altSpellings }}</td>
                                <td>{{ item.idd.root }}{{ item.idd.suffixes }}</td>
                            </tr>
                        </tbody>
                        <tbody v-else>
                            <tr>
                                <td colspan="8" class="text-center">No data to show</td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>
        </div>
        <!-- </transition> -->
    </div>
</template>

<script>
import { ref, onMounted } from 'vue'
import axios from 'axios'

export default {
    data() {
        return {
            isOpen: false,
            show_entries: "all",
            currentPage: 1,
            startIndex: 0,
            endIndex: 25,
            perPage: 25,
            apiData: null,
            singlerow: null,
            totalshow: 0,
            c_name: '',
            sort: {
                field: '',
                desc: true
            },
        };
    },
    computed: {
        sortedData() {
            if (!this.sort.field) {
                return this.apiData;
            }

            return this.apiData.concat().sort((a, b) => {
                if (this.sort.desc) {
                    return a[this.sort.field] > b[this.sort.field] ? -1 : 1
                } else {
                    return a[this.sort.field] > b[this.sort.field] ? 1 : -1
                }
            })
        },
        totalCategory: function () {
            return Math.ceil(this.apiData.length / this.perPage)
        }
    },
    mounted() {
        // const apiUrl = 'https://restcountries.com/v3.1/all'; // Replace with your API endpoint URL

        this.fncCountryList();
    },
    methods: {
        doSort(field) {
            if (field == this.sort.field) {
                this.sort.desc = !this.sort.desc
            } else {
                this.sort.field = field;
                this.sort.desc = true;
            }
        },
        fncCountryList() {
            axios.get("https://restcountries.com/v3.1/all")
                .then((response) => {
                    this.apiData = response.data;
                    // console.log(this.apiData.length);
                    this.totalshow = this.apiData.length;
                })
                .catch((error) => {
                    console.error('Error fetching data:', error);
                    this.apiData = null;
                });
        },
        fncSearch() {
            if (this.c_name == '') {
                this.fncCountryList();
            } else {

                axios.get("https://restcountries.com/v3.1/name/" + this.c_name)
                    .then((response) => {
                        this.apiData = response.data;
                    })
                    .catch((error) => {
                        console.error('Error fetching data:', error);
                        this.apiData = null;
                    });
            }
        },
        pagination: function (activePage) {
            this.currentPage = activePage;
            this.startIndex = (this.currentPage * 25) - 25;
            this.endIndex = this.startIndex + 25;
        },
        previous: function () {
            if (this.currentPage > 1) {
                this.pagination(this.currentPage - 1);
            }

        },
        nextCategory: function () {
            if (this.currentPage < this.totalCategory) {
                this.pagination(this.currentPage + 1);
            }
        },
        //mutated this function
        showEntries: function (value) {
            this.endIndex = value;
        },
        fncShowPopup(cname) {
            // alert(cname);
            this.isOpen = true;
            axios.get("https://restcountries.com/v3.1/name/" + cname)
                .then((response) => {
                    this.singlerow = response.data;
                    // console.log(this.singlerow);
                })
                .catch((error) => {
                    alert(error);
                });
        }

    }
};
</script>

<style scoped>
.form-control-borderless {
    border: none;
}

.form-control-borderless:hover,
.form-control-borderless:active,
.form-control-borderless:focus {
    border: none;
    outline: none;
    box-shadow: none;
}

.table-bg {
    background: #feee07;
    color: #333;
}
.table-bg-p {
    background: #179fe8;
    color: #ffffff;
}

.btn-search {
    border-radius: 0px 5px 5px 0px;
}

.modal {
    width: 500px;
    margin: 0px auto;
    padding: 20px;
    background-color: #fff;
    border-radius: 2px;
    box-shadow: 0 2px 8px 3px;
    transition: all 0.2s ease-in;
    font-family: Helvetica, Arial, sans-serif;
}

.fadeIn-enter {
    opacity: 0;
}

.fadeIn-leave-active {
    opacity: 0;
    transition: all 0.2s step-end;
}

.fadeIn-enter .modal,
.fadeIn-leave-active.modal {
    transform: scale(1.1);
}


.overlay {
    position: fixed;
    top: 0;
    left: 0;
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 100%;
    background: #00000094;
    z-index: 999;
    transition: opacity 0.2s ease;
}

.moda {
    background: #fff;
    width: 50%;
    padding: 5px;
    border-radius: 10px;
}
</style>