<script setup>

import FullCalendar from '@fullcalendar/vue3'
import dayGridPlugin from '@fullcalendar/daygrid'
import interactionPlugin from '@fullcalendar/interaction'
import { ref, onMounted } from 'vue'
import axios from 'axios'
import Navbar from '~/pages/navbar.vue' 

const events = ref([])
const dataInicio = ref('')
const dataFim = ref('')
const ordem = ref('crescente')

// Estilo dos eventos no calendário
const renderEventContent = (arg) => {
  const linhas = arg.event.title.split('\n')
  return {
    html: `
      <div class="bg-red-300 text-black rounded p-1 text-xs leading-tight">
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
    const [guardasRes, usuariosRes, feriadosRes] = await Promise.all([
      axios.get('http://localhost:8000/Guarda/'), // Guardas
      axios.get('http://localhost:8000/UsuarioGuarda/'), // Usuários
     
    ]);

    const guardas = guardasRes.data.results;
    const usuarios = usuariosRes.data.results;
    

    // Formatar eventos de guardas
    const eventosGuardas = guardas.map(guarda => {
      const usuariosDaGuarda = usuarios.filter(u => u.id_guarda === guarda.id);

      const atiradores = usuariosDaGuarda
        .filter(u => !u.comandante)
        .map(u => `#${u.numero_atirador}`)
        .join('\n ');

      const comandante = usuariosDaGuarda.find(u => u.comandante);

      const titulo = `Atiradores:\n ${atiradores}\nComandante: #${comandante?.numero_atirador || '---'}`;

      

      return {
        title: titulo,
        start: guarda.data_guarda.split('T')[0],
        color: '#007bff' // Azul para eventos de guardas
      };
    });

  

    // Combinar eventos de guardas e feriados
    events.value = [...eventosGuardas];
    calendarOptions.value.events = events.value;

  } catch (error) {
    console.error('Erro ao carregar os eventos:', error);
  }
};
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
  <div>
    <Navbar />
    <NuxtPage />
  </div>
  <div class="flex flex-col items-center justify-center p-6">
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

    
    <div class="w-full max-w-6xl">
      <FullCalendar :options="calendarOptions" />
    </div>
  </div>
</template>




