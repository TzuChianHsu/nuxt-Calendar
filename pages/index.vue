<script>
import FullCalendar from '@fullcalendar/vue'
import dayGridPlugin from '@fullcalendar/daygrid'
import timeGridPlugin from '@fullcalendar/timegrid'
import interactionPlugin from '@fullcalendar/interaction'
import listPlugin from '@fullcalendar/list'
import { INITIAL_EVENTS, createEventId } from './event-utils'
import CalendarSidebar from '../components/CalendarSidebar.vue'
import CalendarModal from '../components/CalendarModal.vue'
import moment from 'moment'


const INIT_FORM = {
    title: '',
    allDay: true,
    dateRange: [],
    dateTime: '',
}

export default {
  components: {
    FullCalendar,
    CalendarSidebar,
    CalendarModal
  },
  data: function() {
    return {
      calendarOptions: {
        plugins: [ dayGridPlugin, interactionPlugin, timeGridPlugin, listPlugin ],
        initialView: 'dayGridMonth',
        firstDay: 1, // 設定一週中顯示的第一天是哪天，週日是0，週一是1，類推
        locale: 'zh-cn', // 切換語言，當前為中文
        eventColor: '#3BB2E3', // 全部日曆日程背景色
        initialDate: moment().format('YYYY-MM-DD'), // 自定義設定背景顏色時一定要初始化日期時間
        aspectRatio: 1.65, // 設定日曆單元格寬度與高度的比例。
        displayEventTime: true, // 是否顯示時間
        allDaySlot: false, // 周，日檢視時，all-day 不顯示
        headerToolbar: { // 日曆頭部按鈕位置
          left: 'prevYear,prev next,nextYear',
          center: 'title',
          right: 'today dayGridMonth,timeGridWeek,timeGridDay,timeGridFourDay'
        },
        views: {
          timeGridFourDay: {
            type: 'timeGrid',
            duration: { days: 4 },
            buttonText: '4天'
          }
        },
         buttonText: {
          today: '今天',
          month: '月',
          week: '周',
          day: '日',
        },
        slotLabelFormat: {
          hour: '2-digit',
          minute: '2-digit',
          meridiem: false,
          hour12: false // 設定時間為24小時
        },
        initialView: 'dayGridMonth',
        initialEvents: INITIAL_EVENTS,
        editable: true, // 是否可以進行（拖動、縮放）修改
        selectable: true, // 是否可以選中日曆格
        selectMirror: true,
        dayMaxEvents: true,
        weekends: true,
        eventClick: this.handleEventClick,
        eventsSet: this.handleEvents,   
        select: this.handleDateSelect, // 選中日曆格事件
        eventStartEditable: true, // Event日程開始時間可以改變，預設true，如果是false其實就是指日程塊不能隨意拖動，只能上下拉伸改變他的endTime
        eventDurationEditable: true, // Event日程的開始結束時間距離是否可以改變，預設true，如果是false則表示開始結束時間範圍不能拉伸，只能拖拽
        selectMirror: true,
        selectMinDistance: 0, // 選中日曆格的最小距離
        navLinks: true, // 貼連結
        slotEventOverlap: false // 相同時間段的多個日程視覺上是否允許重疊，預設true允許

      },
      currentEvents: [],
      dialogStatus: false,
      selectInfo: {},
      title: '',
      form: INIT_FORM,
      calendarApi: {}
    }
  },
  mounted() {
      if(this.$refs.fullCalendar){
        this.calendarApi =  this.$refs.fullCalendar.getApi()
      }
  },
  methods: {
    handleWeekendsToggle() {
      this.calendarOptions.weekends = !this.calendarOptions.weekends // update a property
    },
    handleDateSelect(selectInfo) {
      this.dialogStatus = 'add'
      this.selectInfo = selectInfo
      this.form.dateRange = [moment(selectInfo.startStr)] 
    },
    handleAddSubmit(){
      this.calendarApi.unselect()
      const { title, allDay, dateRange, dateTime }  = this.form
       if( this.dialogStatus === 'edit'){ this.selectInfo.event.remove()}
       if (title && dateRange) {
        dateRange.map((date)=>{
          const dateFormat = moment(date).format("YYYY-MM-DD")
          const start = allDay ? dateFormat : dateFormat + moment(dateTime).format("[T]HH:mm:ss")
          this.calendarApi.addEvent({
              id: createEventId(),
              title,
              start,
              // end:  moment(date).add(1,'days'),
              allDay: allDay,
            })
        })
      }
     
      this.closeDialog('calendarModal')
      this.form = INIT_FORM
    },

    handleEventClick(selectInfo) {
      this.selectInfo = selectInfo
      this.dialogStatus = 'edit';
      const { title, start, allDay } = selectInfo.event
      this.form = {
        title,
        allDay,
        dateRange: [moment(start)],
        dateTime: moment(start)
      }
 
    },
    handleDelete(){
      this.$confirm(`確定刪除活動 '${this.selectInfo.event.title}'？`,'' , {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
        }).then(() => {
          this.selectInfo.event.remove()
          this.$message({
            type: 'success',
            message: '删除成功!'
          });
          
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          });
        }).finally(() => {
         this.closeDialog('calendarModal')
        });
    },
    handleEvents(events) {
      this.currentEvents = events
    },
    handleClose(){
      this.closeDialog('calendarModal')
    },
    closeDialog(formName){
      this.dialogStatus = false
      this.$refs[formName].$refs['form'].resetFields()
      this.form = INIT_FORM
    }
  }
}
</script>

<template>
  <div class='demo-app'>
    <div class='demo-app-sidebar'>
       <calendar-sidebar
            :events="this.currentEvents"
            :weekends-visible="this.calendarOptions.weekends"
            :calendar-ref="this.calendarApi"
            @set-weekends-visible="handleWeekendsToggle"
        />
    </div>
    <div class='demo-app-main'>
      <FullCalendar
        ref="fullCalendar"
        class='demo-app-calendar'
        :options='calendarOptions'
      >
        <template v-slot:eventContent='arg'>
          <b>{{ arg.timeText }}</b>
          <i>{{ arg.event.title }}</i>
        </template>
      </FullCalendar>
    </div>
        <calendar-modal 
            ref="calendarModal"
            :form="this.form"
            :dialog-status="dialogStatus" 
            @handle-close="this.handleClose" 
            @handle-add-submit="this.handleAddSubmit"
            @handle-delete="this.handleDelete"
            :select-info="this.selectInfo"
        />
  </div>
</template>

<style lang='css'>

h2 {
  margin: 0;
  font-size: 16px;
}


b { /* used for event dates/times */
  margin-right: 3px;
}

.demo-app {
  display: flex;
  min-height: 100%;
  font-family: Arial, Helvetica Neue, Helvetica, sans-serif;
  font-size: 14px;
}

.demo-app-sidebar {
  width: 300px;
  line-height: 1.5;
  background: #eaf9ff;
  border-right: 1px solid #d3e2e8;
}

.demo-app-sidebar-section {
  padding: 2em;
}

.demo-app-main {
  flex-grow: 1;
  padding: 3em;
}

.fc { /* the calendar root */
  max-width: 1100px;
  margin: 0 auto;
}

.el-date-editor.el-input{
  width: 100%;
}

</style>
