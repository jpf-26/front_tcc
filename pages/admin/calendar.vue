<script setup>
import FullCalendar from '@fullcalendar/vue3'
import dayGridPlugin from '@fullcalendar/daygrid'
import interactionPlugin from '@fullcalendar/interaction'
import { ref, onMounted } from 'vue'
import axios from 'axios'

const events = ref([])
const dataInicio = ref('')
const dataFim = ref('')
const ordem = ref('crescente')

// Estilo dos eventos no calendário
const renderEventContent = (arg) => {
  const linhas = arg.event.title.split('\n')
  return {
    html: `
      <div class="bg-green-300 text-black rounded p-1 text-xs leading-tight">
        ${linhas.map(l => `<div>${l}</div>`).join('')}
      </div>
    `
  }
}

const calendarOptions = ref({
  plugins: [dayGridPlugin, interactionPlugin],
  initialView: 'dayGridMonth',
  events: events.value,
  eventContent: renderEventContent,
  eventClick(info) {
    alert(`Guarda em ${info.event.start.toLocaleDateString()}\n\n${info.event.title}`)
  }
})

// Buscar eventos existentes
const fetchEvents = async () => {
  try {
    const [guardasRes, usuariosRes] = await Promise.all([
      axios.get('http://localhost:8000/Guarda/'),
      axios.get('http://localhost:8000/UsuarioGuarda/')
    ])

    const guardas = guardasRes.data.results
    const usuarios = usuariosRes.data.results

    const eventosFormatados = guardas.map(guarda => {
      const usuariosDaGuarda = usuarios.filter(u => u.id_guarda === guarda.id)

      const atiradores = usuariosDaGuarda
        .filter(u => !u.comandante)
        .map(u => `#${u.numero_atirador}`)
        .join(', ')

      const comandante = usuariosDaGuarda.find(u => u.comandante)

      const titulo = `Atiradores: ${atiradores}\nComandante: #${comandante?.numero_atirador || '---'}`

      return {
        title: titulo,
        start: guarda.data_guarda.split('T')[0]
      }
    })

    events.value = eventosFormatados
    calendarOptions.value.events = events.value

  } catch (error) {
    console.error('Erro ao carregar os eventos:', error)
  }
}

// Sorteio de guardas
const sortearGuardas = async () => {
  if (!dataInicio.value || !dataFim.value) {
    alert('Informe a data de início e fim!')
    return
  }

  try {
    await axios.post('http://localhost:8000/sortear_guardas/', {
      data_inicio: dataInicio.value,
      data_fim: dataFim.value,
      ordem: ordem.value
    })

    alert('Sorteio realizado com sucesso!')
    fetchEvents()

  } catch (error) {
    console.error('Erro ao sortear guardas:', error)
    alert('Erro ao sortear guardas.')
  }
}

// Apagar registros de guardas
const apagarGuardas = async () => {
  const confirmacao = confirm('Tem certeza que deseja apagar todas as guardas?')

  if (!confirmacao) return

  try {
    await axios.delete('http://localhost:8000/apagar_guardas/')
    alert('Guardas apagadas com sucesso!')
    fetchEvents()
  } catch (error) {
    console.error('Erro ao apagar guardas:', error)
    alert('Erro ao apagar guardas.')
  }
}

onMounted(fetchEvents)
</script>



<template>
  <div class="flex flex-col items-center justify-center p-6">
    <!-- Campos para datas e ordem -->
    <div class="mb-4 flex flex-col sm:flex-row items-center gap-4">
      <label class="flex flex-col text-sm">
        Data Início:
        <input v-model="dataInicio" type="date" class="border px-3 py-1 rounded" />
      </label>

      <label class="flex flex-col text-sm">
        Data Fim:
        <input v-model="dataFim" type="date" class="border px-3 py-1 rounded" />
      </label>

      <label class="flex flex-col text-sm">
        Ordem:
        <select v-model="ordem" class="border px-3 py-1 rounded">
          <option value="crescente">Crescente</option>
          <option value="decrescente">Decrescente</option>
        </select>
      </label>
    </div>

    <!-- Botões -->
    <div class="mb-6 flex gap-4">
      <button
        @click="sortearGuardas"
        class="bg-green-500 hover:bg-green-600 text-white font-semibold px-4 py-2 rounded shadow"
      >
        Sortear Guardas
      </button>

      <button
        @click="apagarGuardas"
        class="bg-red-500 hover:bg-red-600 text-white font-semibold px-4 py-2 rounded shadow"
      >
        Apagar Guardas
      </button>
    </div>

    <!-- Calendário -->
    <div class="w-full max-w-6xl">
      <FullCalendar :options="calendarOptions" />
    </div>
  </div>
</template>

<style scoped>
/* Cabeçalho dos dias da semana */
.fc .fc-col-header-cell {
  background-color: #ffa07a; /* Exemplo: laranja claro */
  color: white;
  font-weight: bold;
  text-transform: uppercase;
  padding: 5px 0;
}

/* Cores individuais para cada dia da semana */
.fc .fc-day-sun {
  background-color: #fdd;
}
.fc .fc-day-mon {
  background-color: #ffefcc;
}
.fc .fc-day-tue {
  background-color: #e4ffcc;
}
.fc .fc-day-wed {
  background-color: #ccfff4;
}
.fc .fc-day-thu {
  background-color: #cce0ff;
}
.fc .fc-day-fri {
  background-color: #e6ccff;
}
.fc .fc-day-sat {
  background-color: #fcd5ff;
}

/* Estilo para os eventos */
.fc-event {
  background-color: #7fffd4 !important;
  border: none !important;
  color: black !important;
  padding: 4px;
  border-radius: 12px;
  font-weight: bold;
  font-size: 0.85rem;
  white-space: normal !important;
  line-height: 1.2;
  text-align: center;
}

/* Redimensionamento da altura das células para acomodar mais eventos */
.fc .fc-daygrid-day-frame {
  min-height: 80px;
}

/* Estilo geral do calendário */
.fc {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

/* Estilo dos botões de navegação */
.fc-button {
  background-color: #007bff;
  color: white;
  border: none;
  padding: 6px 12px;
  border-radius: 5px;
}
.fc-button:hover {
  background-color: #0056b3;
}
</style>


