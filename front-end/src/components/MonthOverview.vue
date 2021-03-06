<template>
  <v-container class="full-height" fluid grid-list-xl>

    <header class="page-header">
      <v-select
        :items="years"
        v-model="selectedYear"
        label="Select"
        item-value="text"
        y-offset
        single-line
      ></v-select>

      <DatabaseInfo />
    </header>

    <v-layout row wrap>
      <v-layout v-if="loading" justify-center>
        <v-progress-circular indeterminate color="primary"></v-progress-circular>
      </v-layout>

      <v-layout v-if="!error && !loading && !months.length" justify-center>
        <p class="empty-state">No data found</p>
      </v-layout>

      <v-layout v-if="error" justify-center>
        <v-alert :value="true" type="error">
          Something went wrong
        </v-alert>  
      </v-layout>

      <v-flex v-else-if="!error && !loading" v-for="month in months" :key="month.name" xs12 sm6 md3 xl2>
        <MonthCard :month="month" v-on:removeData="onRemoveData"/>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
import axios from 'axios'

import MonthCard from './MonthCard.vue'
import DatabaseInfo from './DatabaseInfo.vue'

import { store } from '../store.js'

export default {
  components: {
    MonthCard,
    DatabaseInfo
  },
  created() {
    this.getMonths();
  },
  data() {
    return {
      loading: false,
      error: null,
      selectedYear: (new Date).getFullYear(),
      months: [],
      years: [
        { text: (new Date).getFullYear() },
        { text: (new Date).getFullYear() - 1 },
        { text: (new Date).getFullYear() - 2 },
        { text: (new Date).getFullYear() - 3 }
      ]
    }
  },
  watch: {
    selectedYear() {
      this.getMonths();
    }
  },
  methods: {
    getMonths() {
      this.loading = true;

      axios.get(`/api/months?year=${this.selectedYear}`)
        .then(res => {  
          this.months = res.data;
          this.loading = false;
        })
        .catch(err => {
          this.loading = false;
          this.error = err;
        })
    },
    onRemoveData({ monthName, weekNumber }) {
      if (weekNumber) {
        const month = this.months.find(month => month.name === monthName)

        this.$delete(month.weeks, weekNumber)

        if (!Object.keys(month.weeks).length) {
          this.months.filter(month => month.name !== monthName)
        }
      } else {
        this.months = this.months.filter(month => month.name !== monthName)
      }
    }
  }
}
</script>

<style scoped>
  .input-group--select {
    max-width: 10rem;
  }

  .full-height {
    height: 100%;
    position: relative;
  }

  .progress-circular,
  .empty-state,
  .error {
    position: absolute;
    top: 50%;
    transform: translateY(-50%)
  }

  .empty-state {
    font-size: 2.5rem;
    color: rgba(0,0,0,.54);
  }

  .page-header {
    display: flex;
    align-items: center;
  }
</style>
