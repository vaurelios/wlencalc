<script setup>
  import { computed, reactive, ref } from 'vue'
  import { DateTime, Duration } from 'luxon'
  import TimeDiffInput from '../components/TimeDiffInput.vue'

  const startDate = ref(DateTime.now().minus({ month: 1 }).toISODate())
  const endDate = ref(DateTime.now().toISODate())
  const dataByDay = reactive({})
  const workPeriods = ref(2)
  const dayList = computed(() => {
    const items = []

    const a = DateTime.fromISO(startDate.value)
    const b = DateTime.fromISO(endDate.value)
    const diff = b.diff(a, 'days')

    for (let i = 0; i <= diff.days; i++) {
      const cDate = a.plus({ days: i })

      items.push({
        isoStr: cDate.toISODate(),
        display: cDate.toLocaleString()
      })
    }

    return items
  })

  function getTotalMinutesByDay(day) {
    if (!(day in dataByDay)) return 0

    let total = 0

    dataByDay[day].forEach((val) => {
      total = total + val.diff
    })

    return total
  }

  function formatMinutes(minutes) {
    return Duration.fromObject({ minutes: minutes }).toFormat('hh:mm')
  }

  const totalTime = computed(() => {
    let total = 0

    dayList.value.forEach((day) => {
      total = total + getTotalMinutesByDay(day.isoStr)
    })

    return total
  })

  function inputChanged(day, period, val) {
    if (!(day in dataByDay)) dataByDay[day] = []

    dataByDay[day][period] = {
      diff: val
    }
  }
</script>

<template>
  <div class="columns p-5">
    <div class="column is-9 table-container">
      <table class="table is-fullwidth is-scrollable">
        <thead>
          <tr>
            <th>Date</th>
            <th v-for="n in workPeriods" :key="n">Period {{ n }}</th>
            <th>Total</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="item in dayList" :key="item.isoStr">
            <th>{{ item.display }}</th>
            <td v-for="n in workPeriods" :key="n">
              <TimeDiffInput @changed="(val) => inputChanged(item.isoStr, n, val)" />
            </td>
            <td>{{ formatMinutes(getTotalMinutesByDay(item.isoStr)) }}</td>
          </tr>
        </tbody>
      </table>
    </div>
    <div class="column is-3">
      <div class="field">
        <label class="label">Start Date:</label>
        <div class="control">
          <input class="input" type="date" v-model="startDate">
        </div>
      </div>
      <div class="field">
        <label class="label">End Date:</label>
        <div class="control">
          <input class="input" type="date" v-model="endDate">
        </div>
      </div>
      <div class="field">
        <label class="label">Work Peridos by Day:</label>
        <div class="control">
          <input class="input" type="number" v-model="workPeriods">
        </div>
      </div>
      <label class="label">Total Time</label>
      <span>{{ formatMinutes(totalTime) }}</span>
    </div>
  </div>
</template>

<style scoped>
.table.is-scrollable tbody {
  overflow-x: auto;
}
</style>