<template>
  <div class="calendar-sidebar">

      <section class="instructions">
        <el-avatar :size="100" src="https://cube.elemecdn.com/0/88/03b0d39583f48206768a7534e55bcpng.png"></el-avatar>
        <h1>Hsu,Tzu-Chien</h1>
      </section>

      <section class="quick-toggles">
        <h2>
          顯示週末
           <el-switch
          v-model="weekendsVisibleCheckbox"
          active-color="rgb(59, 178, 227)"
          inactive-color="#2C3E50">
        </el-switch>
        </h2>
      </section>

      <section class="events-list">
          <h2>全部活動 ({{ events.length }})</h2>
          <ul>
              <li v-for="event in events" :key="event.id">
                  <el-row type="flex">
                    <el-col span="2">
                       <i class="el-icon-close" @click="handleDelete(event.id)"></i>
                    </el-col>
                    <el-col>
                       <div><b>{{ event.title }}</b></div>
                       <div>{{ getFormattedDate(event) }}</div>
                    </el-col>
                  </el-row>          
              </li>
          </ul>
      </section>
  </div>
</template>

<script>
import moment from 'moment'


export default {
  props: {
    events: {
      type: Array,
      required: true
    },
    weekendsVisible: {
      type: Boolean,
      required: true
    },
    calendarRef: {
      type: Object,
      required: true
    }
  },
  computed: {
    weekendsVisibleCheckbox: {
      get () {
        return this.weekendsVisible
      },
      set (value) {
        return this.$emit('set-weekends-visible', value)
      }
    }
  },
  methods: {
    isAllDay (event) {
      return (event.allDay !== undefined) ? event.allDay : false
    },
    getFormattedDate (event) {
      const date = event.date || event.start
      const { allDay } = event
      if (date === undefined) {
        return ''
      }
      return allDay ? moment(date).format('YYYY-MM-DD') : moment(date).format('YYYY-MM-DD HH:mm:ss')
    },
    handleDelete(id){
      if(this.calendarRef){
        let event = this.calendarRef.getEventById(id)
        event.remove();
      }
    }
  }
}
</script>

<style scoped>
    .calendar-sidebar {
        min-height: 100vh;
        width: 300px;
        line-height: 1.5;
        background: #eaf9ff;
        border-right: 1px solid #d3e2e8;
    }
    .instructions{
      text-align: center;
      margin-bottom: 1em;
    }
    ul {
        margin: 0;
        padding: 0 0 0 1em;
        list-style: none;
    }

    ul li {
      margin: 1.5em 0 0 0;
      padding: 0;
    }

    section {
        padding: 1em 2em;
    }

    h2 {
        margin: 0;
        font-size: 16px;
    }
    .el-icon-close{
      color: red;
      font-size:16px;
      margin: 2px 2px 0px 0px;
      cursor: pointer;
    }
</style>
