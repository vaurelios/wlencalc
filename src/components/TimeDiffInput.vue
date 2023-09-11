<script setup>
    import { computed, reactive, watch } from 'vue';
    import { DateTime } from 'luxon';

    const emit = defineEmits(['changed'])

    const first = reactive({
        hours: '',
        minutes: '',
        value: computed(() => first.hours + ':' + first.minutes)
    })
    const last = reactive({
        hours: '',
        minutes: '',
        value: computed(() => last.hours + ':' + last.minutes)
    })

    function validateAndPadding(newVal, oldVal, min = 0, max=59) {
        const intVal = parseInt(newVal)

        if (isNaN(intVal)) return oldVal

        if (intVal < min || intVal > max)
            return oldVal

        return intVal.toString().padStart(2, '0')
    }

    function calcDiff() {
        const exp = /^\d{2}:\d{2}$/

        if (!exp.test(first.value) && !exp.test(last.value)) return

        const diff = DateTime.fromISO(last.value).diff(DateTime.fromISO(first.value), 'minutes')

        emit('changed', diff.minutes)
    }

    watch(() => first.hours, (newVal, oldVal) => first.hours = validateAndPadding(newVal, oldVal, 0, 23))
    watch(() => first.minutes, (newVal, oldVal) => first.minutes = validateAndPadding(newVal, oldVal, 0, 59))
    watch(() => last.hours, (newVal, oldVal) => last.hours = validateAndPadding(newVal, oldVal, 0, 23))
    watch(() => last.minutes, (newVal, oldVal) => last.minutes = validateAndPadding(newVal, oldVal, 0, 59))

    watch(() => first.value, (val) => {
        if (!(/^\d{2}:\d{2}$/.test(val))) return

        calcDiff()
    })
    watch(() => last.value, (val) => {
        if (!/^\d{2}:\d{2}$/.test(val)) return

        calcDiff()
    })
</script>
<template>
    <div class="is-flex is-flex-direction-row is-flex-wrap-wrap is-justify-content-space-evenly">
        <div class="is-flex is-flex-wrap-nowrap is-align-content-center">
            <input type="text" class="input is-small" v-model="first.hours">
            <span>:</span>
            <input type="text" class="input is-small" v-model="first.minutes">
        </div>
        <span>to</span>
        <div class="is-flex is-flex-wrap-nowrap is-align-content-center">
            <input type="text" class="input is-small" v-model="last.hours">
            <span>:</span>
            <input type="text" class="input is-small" v-model="last.minutes">
        </div>
    </div>
</template>
<style scoped>
input {
    width: 3rem;
}
</style>