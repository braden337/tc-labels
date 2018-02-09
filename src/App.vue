<template>
  <div class="wrapper">

    <main>
      <div class="items" v-for="thirty in thirties">
        <div class="item" v-for="label in thirty">
          {{label[0]}}<br>{{label[1] | weekRange}}<br>&nbsp;
        </div>
      </div>
    </main>

    <aside>
      <h1>Timecard Labels</h1>
      <form>
        <div class="form-group">
          <label for="newPerson">People:&nbsp;</label>
          <input-tag :tags="people" :on-change="savePeople"></input-tag>
          <!--<button class="btn btn-primary" @click="sortPeople">Sort</button>-->
          <p class="form-text text-right">
            <a
              href="#"
              @click.prevent="clear">clear</a> /
            <a
              :href="peopleDataUri"
              :download="saveFileName">save</a> /
            <a
              href="#restore"
              @click.prevent="upload = !upload">{{upload ? 'close' : 'restore'}}</a>
          </p>
          <input type="file" @change="restore" class="form-control" v-if="upload" id="restore">
        </div>
  
        <div class="form-group">
          <label>Start:</label>
          <date-picker
            v-model="start"
            :disabled="startDisabled"
            :format="dateFormat"
            :bootstrap-styling="true"
            :open-date="startOpenDate"
            @input="saveDate('start', $event)">
          </date-picker>
        </div>
  
        <div class="form-group" v-if="start">
          <label>End:</label>
          <date-picker
            v-model="end"
            :disabled="endDisabled"
            :format="dateFormat"
            :bootstrap-styling="true"
            :disabled-picker="!start"
            :open-date="nextWeek"
            @input="saveDate('end', $event)">
          </date-picker>
        </div>
  
        <div class="form-group">
          <input @change="saveSort" type="checkbox" id="sortByDate" v-model="sortByDate">
          <label for="sortByDate">Sort by date</label>
        </div>
      </form>
      <div v-if="start && end && people.length" class="alert alert-primary" role="alert">
        <strong>{{weeks.length}}</strong> {{weeks.length | plural('week',' weeks')}} for <strong>{{people.length}}</strong> {{people.length | plural('person', 'people')}} will print on <strong>{{numPages}}</strong> {{numPages | plural('page', 'pages')}}
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


<style lang="scss">

@import './style';

.vue-input-tag-wrapper .new-tag {
  font-size: 1rem !important;
}

.wrapper {
  display: grid;
  grid-gap: 2rem;
  grid-template-areas:
  "aside main main"
  "aside main main";
  /*"footer footer footer";*/
  grid-template-columns: repeat(3, 1fr);

  @include mq(md) {
    grid-template-areas:
      "aside aside aside"
      "main main main";
      /*"footer footer footer";*/
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
    display: none;

    span {
      $greenish: lightseagreen;
      padding: 1rem;
      background-color: $green;
      border: 1px solid darken($green, 10%);
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
import { chunk, uniq } from 'lodash'
import moment from 'moment'

const ONE_DAY = 86400000
const ONE_WEEK = ONE_DAY * 7

const byNum = (a, b) => a-b

function compareWeeks(a, b) {
  return a[1].monday - b[1].monday
}

function mergeNamesWithDates(names, weeks, sort) {
  let merged = []
  for (let name of names) {
    for (let week of weeks) {
      merged.push([name, week])
    }
  }
  return sort ? merged.sort(compareWeeks) : merged
}

function nextMonday() {
  let today = new Date()
  let day = today.getDay()
  let monday = null

  if (day == 1) {
    monday = today
  }
  else if (day < 1) {
    monday = new Date(today.valueOf() + ONE_DAY)
  }
  else {
    monday = new Date(today.valueOf() + ONE_DAY * (8 - day))
  }

  return monday
}


module.exports = {
  data: function() {
    return {
      people: JSON.parse(localStorage.getItem('people')) || [],
      sortByDate: JSON.parse(localStorage.getItem('sort')) || false,
      start: localStorage.getItem('start') ? new Date(JSON.parse(localStorage.getItem('start'))) : 0 || null,
      end: localStorage.getItem('start') ? new Date(JSON.parse(localStorage.getItem('end'))) : 0 || null,
      startOpenDate: this.start || nextMonday(),
      upload: false,
      dateFormat: 'MMMM d, yyyy'
    }
  },
  mounted() {
    document.querySelector('.new-tag').focus()
  },
  computed: {
    numPages() {
      // return Math.ceil(this.people.length * this.weeks.length / 30)
      let pages = this.people.length * this.weeks.length / 30
      return Number.isInteger(pages) ? pages : pages.toFixed(1)
    },
    saveFileName() {
      return `people_${new Date().valueOf()}.json`
    },
    peopleDataUri() {
      return `data:application/json;base64,${btoa(JSON.stringify(this.people, null, 2))}`
    },
    nextWeek() {
      return new Date(this.start.valueOf() + ONE_WEEK)
    },
    startDisabled() {
      return {
        days: [0,2,3,4,5,6],
        to: new Date(nextMonday().valueOf() - ONE_DAY),
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
        let current = this.start.valueOf()
        let end = this.end.valueOf()
        while(current < end) {
          let week = {}
          week.monday = current.valueOf()
          week.sunday = current.valueOf() + ONE_DAY * 6
          weeks.push(week)
          current += ONE_WEEK
        }
        // let current = moment(this.start)
        // let end = moment(this.end)
        // while(current.isBefore(end)) {
        //   let week = current.format('MMM DD - ')
        //   current = current.add(6, 'days')
        //   week += current.format('MMM DD, YYYY')
        //   current = current.add(1, 'day')
        //   weeks.push(week)
        // }
      }
      return weeks
    }
  },
  methods: {
    clear() {
      this.people = []
      // this.start = null
      // this.end = null
      // this.sortByDate = false
      // this.saveDate()
      this.savePeople()
      // this.removeSort()
    },
    restore(e) {
      let reader = new FileReader()
      reader.readAsText(e.target.files[0])
      reader.addEventListener('loadend', function() {
        this.savePeople(JSON.parse(reader.result))
        this.upload = false
      }.bind(this))
    },
    displayDate(date) {
      return date ? moment(date).format('ll') : ''
    },
    saveSort(e) {
      console.log(e.target.checked)
      localStorage.setItem('sort', JSON.stringify(e.target.checked))
    },
    removeSort() {
      localStorage.removeItem('sort')
    },
    saveDate(name, date) {
      if (name) {
        localStorage.setItem(name, JSON.stringify(date.valueOf()))
      }
      else {
        ['start', 'end'].forEach(n => localStorage.removeItem(n))
      }
    },
    capitalize(word) {
      word = word.split('')
      word[0] = word[0].toUpperCase()
      return word.join('')
    },
    savePeople(people) {
      if (people) {
        let sortedPPL = uniq(people.map(x => x.split(' ').map(this.capitalize).join(' ')).sort())
        this.people = sortedPPL
        localStorage.setItem('people', JSON.stringify(sortedPPL))
      }
      else {
        localStorage.removeItem('people')
      }
    },
    validateSelection(e) {

    },
    sortPeople() {
      this.people = this.people.slice().sort()
    }
  },
  filters: {
    plural(n, singular, plural) {
      return n == 1 ? singular : plural
    },
    weekRange(week) {
      let a = moment(week.monday)
      let b = moment(week.sunday)
      return a.format('MMM D')+' - '+b.format('MMM D, YYYY')
    }
  },
  components: {
    InputTag, DatePicker
  }
}
</script>
