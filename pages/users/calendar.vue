<template>
  <div>
    <FullCalendar
      ref="calendarRef"
      :options="calendarOptions"
    />
    <button @click="addEvent">Adicionar Evento Manual</button>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import FullCalendar from '@fullcalendar/vue3'
import dayGridPlugin from '@fullcalendar/daygrid'
import interactionPlugin from '@fullcalendar/interaction'
import type { CalendarOptions, EventInput } from '@fullcalendar/core';

const calendarRef = ref()

const calendarOptions = ref<CalendarOptions>({
  plugins: [dayGridPlugin, interactionPlugin],
  initialView: 'dayGridMonth',
  editable: true,
  selectable: true,
  events: [],
  selectMirror: true,
  select: (info) => {
    const title = prompt('Digite o título do evento:')
    if (title) {
      const calendarApi = calendarRef.value.getApi()
      calendarApi.addEvent({
        title,
        start: info.startStr,
        end: info.endStr,
        allDay: info.allDay,
      })
    }
  },
})

function addEvent() {
  const calendarApi = calendarRef.value.getApi()
  calendarApi.addEvent({
    title: 'Evento Manual',
    start: new Date().toISOString().split('T')[0],
    allDay: true,
  })
}
</script>

<style scoped>
/* Estilo opcional */
</style>