<template lang="pug">
  q-card.q-mt-lg
    q-card-title.q-ml-md(class="q-subheading") {{room.name}}
        <span slot="subtitle"> {{spareCount}}个课时空闲</span>
    q-card-separator
    q-card-main
      div.row
      | 教室预约情况:
      div.row.justify-between(style="overflow-x:scroll;height:5rem;")
        div
        div
          div.row.no-wrap(flat)
            div.justify-start(v-for="i in 13" :key="i" style="color:grey;height:1rem;width:3rem;") |{{schedule[i-1].start}}
          q-btn-group.row.no-wrap(flat)
            q-btn(flat
            :class="{'bg-blue-grey':roomSchedule[i-1]===1,'bg-deep-orange-4':roomSchedule[i-1]===0,'bg-green':roomSchedule[i-1]===-1}"
            v-for="i in 13" :key="i" color="white" style="height:2rem;width:3rem;" :label="i")
        div
      div.row
        q-select.col-6(
            float-label="选择开始时间"
            v-model="start"
            :options="startOptions")
        q-select.col-6(
            float-label="选择结束时间"
            v-model="end"
            :options="endOptions")
    q-card-actions(align="end")
        q-btn(flat icon="event" label="重置" @click="onResetClick()")
        q-btn(flat wait-for-ripple icon="event" label="预约" @click="onReservationClick()")
</template>
<script>
// import { schedule } from '/utils/index'
const schedule = []
const rawStartOptions = [
  {
    label: '第一节 (8：00)',
    value: 1
  },
  {
    label: '第二节 (8：55)',
    value: 2
  },
  {
    label: '第三节 (10：00)',
    value: 3
  },
  {
    label: '第四节 (10：55)',
    value: 4
  },
  {
    label: '第五节 (12：10)',
    value: 5
  },
  {
    label: '第六节 (13：05)',
    value: 6
  },
  {
    label: '第七节 (14：10)',
    value: 7
  },
  {
    label: '第八节 (15：05)',
    value: 8
  },
  {
    label: '第九节 (16：00)',
    value: 9
  },
  {
    label: '第十节 (16：55)',
    value: 10
  },
  {
    label: '第十一节 (18：00)',
    value: 11
  },
  {
    label: '第十二节 (18：55)',
    value: 12
  },
  {
    label: '第十三节 (19：50)',
    value: 13
  }
]
export default {
  props: {
    index: Number,
    room: {
      type: Object,
      default() {
        return {}
      }
    },
    date: Date
  },
  data() {
    return {
      schedule: schedule,
      start: null,
      end: null
    }
  },
  computed: {
    startOptions: function() {
      let options = []
      options = rawStartOptions.filter(x => {
        return this.room.schedule[x.value - 1] === 1
      })
      return options
    },
    endOptions: function() {
      let options = []
      if (this.start === null) {
        return options
      } else {
        let start = this.start - 1
        let i = this.start - 1
        while (this.room.schedule[i]) {
          options.push({
            label: `第${i + 1}节（${schedule[i].end}）`,
            value: i + 1
          })
          if (i - start + 1 >= 4) {
            break
          }
          i += 1
        }
        return options
      }
    },
    roomSchedule: function() {
      let schedule = this.room.schedule.slice()
      if (this.start !== null && this.end !== null) {
        for (let i = this.start - 1; i < this.end; i++) {
          if (schedule[i] === 1) {
            schedule[i] = -1
          }
        }
      }
      // console.log(schedule)
      return schedule
    },
    spareCount: function() {
      return this.room.schedule.reduce((total, num) => {
        return total + num
      })
    }
  },
  watch: {
    start: function(newStart, oleStart) {
      this.end = null
    },
    room: function() {
      this.onResetClick()
    }
  },
  methods: {
    onResetClick() {
      this.start = null
      this.end = null
    },
    onReservationClick() {
      if (!this.start || !this.end) {
        return
      }
      this.$q
        .dialog({
          title: '确认',
          message: `确认预约吗`,
          ok: '确认',
          cancel: '取消'
        })
        .then(() => {
          this.postReservationForm()
          // Picked "OK"
        })
        .catch(() => {
          // Picked "Cancel" or dismissed
        })
    },
    postReservationForm() {
      this.$http
        .post('/api/room-orders/', {
          roomID: this.room.id,
          start: this.start,
          end: this.end,
          date: this.date.valueOf() / 1000
        })
        .then(resp => {
          // console.log(resp)
          this.$emit('reservateSuccess')
          this.$q.notify('预约成功！')
        })
        .catch(err => {
          console.log(err)
          this.$q.notify(err.response.data.msg)
        })
    }
  }
}
</script>
