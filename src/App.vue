<template>
  <div class="wrapper">

    <main>
      <div class="items" v-for="stickers in thirties">
        <div class="item" v-for="sticker in stickers">
          {{sticker[0]}}<br>{{sticker[1]}}<br>&nbsp;
        </div>
      </div>
    </main>

    <aside>
      <h1>Timecard Labels</h1>

      <div class="form-group">
        <label for="newPerson">People:&nbsp;</label>
        <input-tag :tags="people" :on-change="savePeople"></input-tag>
      </div>

      <div class="form-group">
        <label>Start:</label>
        <date-picker
          v-model="start"
          @input="validateSelection"
          :disabled="startDisabled"
          format="D - MMM d, yyyy"
          :bootstrap-styling="true">
        </date-picker>
      </div>

      <div class="form-group" v-if="start">
        <label>End:</label>
        <date-picker
          v-model="end"
          :disabled="endDisabled"
          format="D - MMM d, yyyy"
          :bootstrap-styling="true"
          :disabled-picker="!start"
          :open-date="nextWeek">
        </date-picker>
      </div>

      <div class="form-group">
        <input type="checkbox" id="sortByDate" v-model="sortByDate">
        <label for="sortByDate">Sort by date</label>
      </div>
    </aside>

    <footer>
      <span>
        Made with &hearts; by
        <a href="//github.com/braden337">
          Braden
        </a>
      </span>
    </footer>
  </div>
</template>


<style lang="scss" scoped>

@import './style';

.wrapper {
  display: grid;
  grid-gap: 2rem;
  grid-template-areas:
  "aside main main"
  "aside main main"
  "footer footer footer";
  grid-template-columns: repeat(3, 1fr);

  @include mq(md) {
    grid-template-areas:
      "aside aside aside"
      "main main main"
      "footer footer footer";
  }

  main {
    grid-area: main;
    display: grid;
    grid-gap: 1rem;
    
    @include mq(print) {
      display: block;
    }

    .items {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(6.67cm, 1fr));
      grid-auto-rows: 2.54cm;
      grid-gap: 1rem;

      @include mq(print) {
        background-color: none;
        grid-template-columns: 6.67cm 6.67cm 6.67cm;
        grid-gap: 0 0.32cm;
        padding: 1.263cm 0.48cm;
      }

      .item {
        display: flex;
        justify-content: center;
        align-items: center;
        text-align: center;
        border-radius: 10px;
        background-color: $gray-200;
        color: $dark;

        @include mq(print) {
          background: none;
        }
      }
    }
  }

  aside {
    grid-area: aside;
  }

  footer {
    grid-area: footer;
    display: flex;

    span {
      $greenish: lightseagreen;
      padding: 1rem;
      background-color: $greenish;
      border: 1px solid darken($greenish, 10%);
      border-radius: 3px;
      color: darken($greenish, 30%);

      a {
        color: inherit;
      }
    }
  }
}


@media print {
  .wrapper {
    display: block;
  }
  aside, footer, footer > span {
    display: none;
  }
}

</style>

<script>

import InputTag from 'vue-input-tag'
import DatePicker from 'vuejs-datepicker'
import { chunk } from 'lodash'
import moment from 'moment'

const ONE_WEEK = 604800000
const ONE_DAY = 86400000

const byNum = (a, b) => a-b

function compareStickers(a, b) {
  if (a[1] == b[1]) {
    return 0
  }
  else if (a[1] < b[1]) {
    return -1
  }
  else {
    return 1
  }
}

function mergeNamesWithDates(names, dates, sort) {
  let merged = []
  for (let name of names) {
    for (let date of dates) {
      merged.push([name, date])
    }
  }
  return sort ? merged.sort(compareStickers) : merged
}

function nextMonday() {
  let today = moment()
  let day = today.day()
  let monday = null

  if (day == 1) {
    monday = today
  }
  else if (day < 1) {
    monday = today.add(1, 'day')
  }
  else {
    monday = today.endOf('week').add(2, 'days')
  }

  return monday
}
// new Date(nextMonday())
// new Date(nextMonday().add(6, 'days'))
module.exports = {
  data: function() {
    return {
      greeting: 'Hello world',
      people: JSON.parse(localStorage.getItem('people')) || [],
      sortByDate: false,
      start: JSON.parse(localStorage.getItem('start')) || null,
      end: JSON.parse(localStorage.getItem('end')) || null,
      startDisable: null
    }
  },
  // created() {
  //   this.startDisabled = {
  //     days: [0,2,3,4,5,6],
  //     // to: new Date(Date.now() - 86400000)
  //   }
  // },
  mounted() {
    document.querySelector('.new-tag').focus()
  },
  computed: {
    nextWeek() {
      return new Date(this.start.valueOf() + ONE_WEEK)
    },
    startDisabled() {
      return {
        days: [0,2,3,4,5,6],
        from: this.end
      }
    },
    endDisabled() {
      return {
        days: [1,2,3,4,5,6],
        to: this.start
      }
    },
    thirties() {
      return chunk(mergeNamesWithDates(this.people, this.weeks, this.sortByDate), 30)
    },
    weeks() {
      let weeks = []
      if (this.start && this.end) {
        let current = moment(this.start)
        let end = moment(this.end)
        while(current.isBefore(end)) {
          let week = current.format('MMM DD - ')
          current = current.add(6, 'days')
          week += current.format('MMM DD, YYYY')
          current = current.add(1, 'day')
          weeks.push(week)
        }
      }
      return weeks;
    }
  },
  methods: {
    displayDate(date) {
      return date ? moment(date).format('ll') : ''
    },
    savePeople(people) {
      localStorage.setItem('people', JSON.stringify(people))
    },
    validateSelection(e) {

    }
  },
  components: {
    InputTag, DatePicker
  }
}
</script>