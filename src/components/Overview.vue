<template>
  <div class="overview">
    <div class="loader" v-if="loading">
      <div>
        <p>Finding what country are you in...</p>
        <div class="loader1">
          <span></span>
          <span></span>
          <span></span>
          <span></span>
          <span></span>
        </div>
      </div>
    </div>
    <div class="main" v-else>
      <div class="top">
        <div class="logo">
          <h1>Holidays</h1>
        </div>
        <div class="select">
          <div class="select-container country">
            <select v-model="countryCode" @change="setCountry">
              <option value="" selected data-default>Select Country</option>
              <option
                v-for="(item, index) in countries"
                :key="index"
                :value="item['iso-3166']"
              >
                {{ item.country_name }}
              </option>
            </select>
          </div>
          <div class="select-container language">
            <select v-model="year">
              <option value="" selected data-default>Select Year</option>
              <option v-for="(item, index) in years" :key="index" :value="item">
                {{ item }}
              </option>
            </select>
            <div class="select-btn">
              <button
                @click="countryCode !== '' && year !== '' ? search() : null"
              >
                Search
              </button>
            </div>
          </div>
        </div>
      </div>

      <div class="search-loader" v-if="searching">
        <div class="loader1">
          <span></span>
          <span></span>
          <span></span>
          <span></span>
          <span></span>
        </div>
      </div>

      <div class="body" v-else>
        <div class="header">
          <span><img :src="data.$$flag" alt="" /></span>
          <h2>{{ data.$$country }} {{ selectedYear }}</h2>
        </div>
        <div
          class="holidays"
          v-for="(item, index) in data.holidays"
          :key="index"
        >
          <h2>{{ item.$$month }}</h2>
          <div class="dates" v-for="(holiday, index) in item" :key="index">
            <section class="left">
              <span class="number">
                {{ holiday.date.datetime.day }}
              </span>
              <span class="day">
                {{ holiday.$$day }}
              </span>
            </section>
            <section class="right">
              <div
                class="content-header"
                :style="{ background: holiday.$$color }"
              >
                {{ holiday.name }}
              </div>
              <div class="description">
                <p>{{ holiday.description }}</p>
                <ul>
                  <li>{{ holiday.locations }}</li>
                  <li>{{ holiday.type[0] }}</li>
                </ul>
              </div>
            </section>
          </div>
        </div>
      </div>
      <center>
        <button class="scroll-top" @click="backToTop">^ Back to top</button>
      </center>
    </div>
  </div>
</template>

<script>
const axios = require('axios');

export default {
  name: 'Overview',
  data() {
    return {
      calendarKey: process.env.VUE_APP_CALENDARKEY,
      ipDataKey: process.env.VUE_APP_IPDATAKEY,
      loading: true,
      searching: false,
      data: [],
      countries: null,
      countryCode: '',
      country: '',
      year: '',
      selectedYear: '2020',
      years: [
        '2015',
        '2016',
        '2017',
        '2018',
        '2019',
        '2020',
        '2021',
        '2022',
        '2023',
        '2024',
        '2025',
        '2026',
        '2027',
        '2028',
        '2029',
        '2030',
      ],
    };
  },
  created() {
    this.countries = JSON.parse(localStorage.getItem('countries')) || [];
    this.getData();
    if (this.countries.length < 1) {
      this.getCountries();
    }
  },
  methods: {
    setCountry() {
      this.country = this.countries.find(
        (country) => country['iso-3166'] === this.countryCode
      );
    },
    search() {
      this.selectedYear = this.year;
      this.searching = true;
      this.getData();
    },
    getCountries() {
      axios
        .get(
          `https://calendarific.com/api/v2/countries?&api_key=${this.calendarKey}`
        )
        .then((response) => {
          this.countries = response.data.response.countries;
          localStorage.setItem('countries', JSON.stringify(this.countries));
        });
    },
    getData() {
      axios
        .get(`https://api.ipdata.co?api-key=${this.ipDataKey}`)
        .then((response) => {
          axios
            .get(
              this.searching
                ? `https://calendarific.com/api/v2/holidays?&api_key=${this.calendarKey}&country=${this.countryCode}&year=${this.year}`
                : `https://calendarific.com/api/v2/holidays?&api_key=${this.calendarKey}&country=${response.data.country_code}&year=2020`
            )
            .then((res) => {
              res.data.response['$$country'] = this.searching
                ? this.country.country_name
                : response.data.country_name;
              res.data.response['$$flag'] = this.searching
                ? `https://www.countryflags.io/${this.countryCode}/flat/64.png`
                : response.data.flag;

              const holidays = res.data.response.holidays
                .map((value) => {
                  const month = this.dateBuilder(value.date.iso, 'month');

                  value['$$day'] = this.dateBuilder(value.date.iso, 'day');
                  value['$$month'] = month;

                  value['$$color'] = this.setColor(month);
                  return value;
                })
                .reduce((arr, obj) => {
                  arr[obj.$$month] = arr[obj.$$month] || [];
                  arr[obj.$$month]['$$month'] = obj.$$month;
                  arr[obj.$$month]['$$holidays'] = [];

                  arr[obj.$$month].push(obj);

                  arr[obj.$$month].forEach((element) => {
                    arr[obj.$$month]['$$holidays'].push(element);
                  });

                  return arr;
                }, {});

              res.data.response.holidays = holidays;

              this.data = res.data.response;
              this.loading = false;
              this.searching = false;
            });
        })
        .catch(() => {
          this.loding = false;
        });
    },
    backToTop() {
      window.scrollTo({ top: 0, behavior: 'smooth' });
    },
    setColor(val) {
      switch (val) {
        case 'January':
          return '#12cad6';
        case 'February':
          return '#ff00c8';
        case 'March':
          return '#f5e965';
        case 'April':
          return '#ff3e6d';
        case 'May':
          return '#00faac';
        case 'June':
          return '#6b48ff';
        case 'July':
          return '#dcaee8';
        case 'August':
          return '#e41749';
        case 'September':
          return '#43c0ac';
        case 'October':
          return '#fc6b3f';
        case 'November':
          return '#4ef037';
        case 'December':
          return '#f85959';
        default:
          return 'white';
      }
    },
    dateBuilder(value, type) {
      const actiondate = new Date(value);

      let months = [
        'January',
        'February',
        'March',
        'April',
        'May',
        'June',
        'July',
        'August',
        'September',
        'October',
        'November',
        'December',
      ];
      let days = ['sun', 'mon', 'tue', 'wed', 'thu', 'fri', 'sat'];
      let day = days[actiondate.getDay()];
      let month = months[actiondate.getMonth()];

      return type === 'day' ? day : month;
    },
  },
};
</script>

<style lang="scss" scoped>
.main {
  padding: 15px;
  background: #f2f2f2;

  .top {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    margin-bottom: 20px;
    background: #ffffff;
    padding: 30px;
    border-radius: 15px;

    .logo {
      color: #f35588;
      text-align: center;
    }

    .select {
      display: flex;
      justify-content: flex-end;
      flex-direction: column;

      .country {
        margin-bottom: 10px;
      }

      .select-container {
        select {
          width: 100%;
          padding: 7px;
          border: 1px solid #e6e6e6;
          border-radius: 5px;
          font-family: 'Poppins', sans-serif;
        }
      }

      .language {
        display: flex;
      }
    }
  }

  .body {
    .header {
      display: flex;
      align-items: center;

      span {
        img {
          width: 35px;
          padding-top: 7px;
          margin-right: 10px;
        }
      }
    }

    .holidays {
      margin: 15px 0;

      h2 {
        margin-bottom: 15px;
      }

      .dates {
        display: flex;

        .left {
          width: 80px;
          text-align: center;
          line-height: 1.2em;
          display: flex;
          flex-direction: column;
          margin-right: 25px;
          border-right: 2px solid #e6e6e6;

          .number {
            font-weight: bold;
          }

          .day {
            font-size: 14px;
          }
        }

        .right {
          width: 100%;
          margin-bottom: 15px;

          .content-header,
          .description {
            padding: 10px;
          }

          .content-header {
            color: #ffffff;
            border-top-left-radius: 5px;
            border-top-right-radius: 5px;
          }

          .description {
            background: #ffffff;
            font-size: 14px;
            border-bottom-left-radius: 5px;
            border-bottom-right-radius: 5px;

            ul {
              margin-top: 10px;
              margin-left: 18px;
            }
          }
        }
      }
    }
  }
}

button {
  height: 37px;
  width: 120px;
  margin-left: 10px;
  background: #f35588;
  border: 0;
  border-radius: 5px;
  color: #ffffff;
  font-size: 14px;
  padding: 7px;
  font-family: 'Poppins', sans-serif;
  cursor: pointer;

  &:hover {
    background: #f02868;
  }

  &:focus {
    outline: none;
  }
}

.loader,
.search-loader {
  display: flex;
  justify-content: center;
  background: #f2f2f2;
  align-items: center;
  text-align: center;
}

.search-loader {
  height: 75vh;
}

.loader {
  height: 100vh;
}

.loader1 {
  display: inline-block;
  font-size: 0px;
  padding: 0px;
}
.loader1 span {
  vertical-align: middle;
  border-radius: 100%;

  display: inline-block;
  width: 10px;
  height: 10px;
  margin: 3px 2px;
  -webkit-animation: loader1 0.8s linear infinite alternate;
  animation: loader1 0.8s linear infinite alternate;
}
.loader1 span:nth-child(1) {
  -webkit-animation-delay: -1s;
  animation-delay: -1s;
  background: #6b48ff;
}
.loader1 span:nth-child(2) {
  -webkit-animation-delay: -0.8s;
  animation-delay: -0.8s;
  background: #12cad6;
}
.loader1 span:nth-child(3) {
  -webkit-animation-delay: -0.26666s;
  animation-delay: -0.26666s;
  background: #f02868;
}
.loader1 span:nth-child(4) {
  -webkit-animation-delay: -0.8s;
  animation-delay: -0.8s;
  background: #f5e965;
}
.loader1 span:nth-child(5) {
  -webkit-animation-delay: -1s;
  animation-delay: -1s;
  background: #4ef037;
}

@keyframes loader1 {
  from {
    transform: scale(0, 0);
  }
  to {
    transform: scale(1, 1);
  }
}
@-webkit-keyframes loader1 {
  from {
    -webkit-transform: scale(0, 0);
  }
  to {
    -webkit-transform: scale(1, 1);
  }
}

@keyframes loader5 {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(720deg);
  }
}
@-webkit-keyframes loader5 {
  from {
    -webkit-transform: rotate(0deg);
  }
  to {
    -webkit-transform: rotate(720deg);
  }
}

@media (min-width: 400px) {
  h2 {
    font-size: 30px;
  }

  .left {
    font-size: 23px;
  }

  .main .holidays .dates .right {
    .content-header {
      font-size: 19px;
    }

    .description {
      font-size: 17px;
    }
  }
}

@media (min-width: 768px) {
  .main {
    padding: 15px 150px;
  }

  .main .holidays .dates .right {
    .content-header,
    .description {
      padding: 15px;
    }
  }
}

@media (min-width: 992px) {
  .main .top {
    flex-direction: row;

    .select {
      flex-direction: row;
      align-items: center;

      .country {
        margin-bottom: 0;
        margin-right: 10px;
      }
    }
  }
}
</style>
