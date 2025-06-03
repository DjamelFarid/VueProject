<template>
  <div class="contenedor-tareas">
    <h1>Lista de Tareas</h1>

    <div class="agregar-tarea">
      <h2>Agregar Nueva Tarea</h2>
      <input
        type="text"
        v-model="nuevaTarea"
        placeholder="Nueva tarea"
        @keyup.enter="agregarTarea"
      />
      <button @click="agregarTarea">Agregar</button>
    </div>

    <div class="lista-tareas">
      <h2>Lista de Tareas</h2>
      <div class="filtro">
        <label>
          <input type="checkbox" v-model="mostrarCompletadas" />
          Mostrar tareas completadas
        </label>
      </div>
      <table>
        <thead>
          <tr>
            <th>ID</th>
            <th>Tarea</th>
            <th>Estado</th>
            <th>Acciones</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="tarea in tareasFiltradas" :key="tarea.id">
            <td>{{ tarea.id }}</td>
            <td>{{ tarea.titulo }}</td>
            <td>
              <input type="checkbox" :checked="tarea.completada" @change="cambiarEstado(tarea)" />
            </td>
            <td>
              <button @click="eliminarTarea(tarea.id)">Eliminar</button>
            </td>
          </tr>
        </tbody>
      </table>
      <br />
      <button @click="eliminarCompletadas">Eliminar Tareas Completadas</button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'

interface Tarea {
  id: number
  titulo: string
  completada: boolean
}

const URL_API = 'http://localhost:3000/items'
const tareas = ref<Tarea[]>([])
const nuevaTarea = ref('')
const mostrarCompletadas = ref(false)

const tareasFiltradas = computed(() => {
  return tareas.value.filter((tarea) => mostrarCompletadas.value || !tarea.completada)
})

const cargarTareas = async () => {
  try {
    const respuesta = await fetch(URL_API)
    if (!respuesta.ok) throw new Error('Error al cargar las tareas')
    tareas.value = await respuesta.json()
  } catch (error) {
    console.error('Error al cargar las tareas:', error)
  }
}

const agregarTarea = async () => {
  if (nuevaTarea.value.trim() === '') return

  try {
    const nuevaTareaItem = {
      titulo: nuevaTarea.value,
      completada: false,
    }
    const respuesta = await fetch(URL_API, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(nuevaTareaItem),
    })
    if (!respuesta.ok) throw new Error('Error al agregar la tarea')
    const tareaAgregada = await respuesta.json()
    tareas.value.push(tareaAgregada)
    nuevaTarea.value = ''
  } catch (error) {
    console.error('Error al agregar la tarea:', error)
  }
}

const cambiarEstado = async (tarea: Tarea) => {
  try {
    const tareaActualizada = { ...tarea, completada: !tarea.completada }
    const respuesta = await fetch(`${URL_API}/${tarea.id}`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(tareaActualizada),
    })
    if (!respuesta.ok) throw new Error('Error al actualizar la tarea')
    const indice = tareas.value.findIndex((t) => t.id === tarea.id)
    if (indice !== -1) {
      tareas.value[indice] = tareaActualizada
    }
  } catch (error) {
    console.error('Error al actualizar la tarea:', error)
  }
}

const eliminarTarea = async (id: number) => {
  try {
    const respuesta = await fetch(`${URL_API}/${id}`, {
      method: 'DELETE',
    })
    if (!respuesta.ok) throw new Error('Error al eliminar la tarea')
    tareas.value = tareas.value.filter((tarea) => tarea.id !== id)
  } catch (error) {
    console.error('Error al eliminar la tarea:', error)
  }
}

const eliminarCompletadas = async () => {
  try {
    const tareasCompletadas = tareas.value.filter((tarea) => tarea.completada)
    await Promise.all(
      tareasCompletadas.map((tarea) =>
        fetch(`${URL_API}/${tarea.id}`, {
          method: 'DELETE',
        }),
      ),
    )
    tareas.value = tareas.value.filter((tarea) => !tarea.completada)
  } catch (error) {
    console.error('Error al eliminar las tareas completadas:', error)
  }
}

onMounted(() => {
  cargarTareas()
})
</script>

<style scoped>
.contenedor-tareas {
  width: 90%;
  margin: 2rem auto;
  padding: 2.5rem;
  background-color: #f0f7f0;
  border-radius: 15px;
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.1);
  min-height: 80vh;
  display: flex;
  flex-direction: column;
}

h1 {
  color: #1a472a;
  text-align: center;
  margin-bottom: 2.5rem;
  font-size: 2.8rem;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
}

h2 {
  color: #2d5a27;
  margin-bottom: 1.5rem;
  font-size: 1.8rem;
}

.agregar-tarea {
  background-color: white;
  padding: 2rem;
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  margin-bottom: 2.5rem;
}

.agregar-tarea input {
  padding: 15px;
  border: 2px solid #2ecc71;
  border-radius: 8px;
  width: 70%;
  margin-right: 15px;
  font-size: 1.1rem;
  transition: all 0.3s ease;
}

.agregar-tarea input:focus {
  outline: none;
  border-color: #27ae60;
  box-shadow: 0 0 8px rgba(46, 204, 113, 0.5);
}

.lista-tareas {
  background-color: white;
  padding: 2rem;
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  flex-grow: 1;
}

.filtro {
  margin: 1.5rem 0;
  padding: 1.2rem;
  background-color: #e8f5e9;
  border-radius: 8px;
}

.filtro label {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #1a472a;
  font-weight: 500;
}

table {
  width: 100%;
  border-collapse: separate;
  border-spacing: 0;
  margin: 1.5rem 0;
  border-radius: 10px;
  overflow: hidden;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

th, td {
  padding: 1.2rem;
  text-align: left;
  border: 1px solid #e0e0e0;
  font-size: 1.1rem;
}

th {
  background-color: #2ecc71;
  color: white;
  font-weight: 600;
  text-transform: uppercase;
  font-size: 1rem;
  padding: 1.5rem;
}

tr:nth-child(even) {
  background-color: #f0f7f0;
}

tr:hover {
  background-color: #e8f5e9;
}

button {
  padding: 12px 24px;
  background-color: #2ecc71;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s ease;
  text-transform: uppercase;
  font-size: 1rem;
  min-width: 120px;
}

button:hover {
  background-color: #27ae60;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

button:active {
  transform: translateY(0);
}

input[type="checkbox"] {
  width: 20px;
  height: 20px;
  cursor: pointer;
  accent-color: #2ecc71;
}

@media (max-width: 768px) {
  .contenedor-tareas {
    width: 95%;
    padding: 1.5rem;
  }
  
  .agregar-tarea input {
    width: 100%;
    margin-bottom: 1rem;
  }
  
  button {
    width: 100%;
    margin-top: 0.5rem;
  }
}
</style>
