<template>
  <v-card>
    <v-layout>
      <!-- La linea de arriba  -->
      <v-app-bar color="primary">
        <v-app-bar-nav-icon
          variant="text"
          @click.stop="drawer = !drawer"
        ></v-app-bar-nav-icon>

        <v-toolbar-title>Seccións del programa</v-toolbar-title>
      </v-app-bar>

      <!-- el menu lateral despegable -->
      <v-navigation-drawer
        v-model="drawer"
        :location="$vuetify.display.mobile ? 'bottom' : undefined"
        temporary
      >
        <v-list
          v-model:selected="seccionSeleccionada"
          :items="items"
          @click:select="drawer = false"
        ></v-list>
      </v-navigation-drawer>

      <!-- El principio -->
      <v-main style="min-height: 400px;">
        <v-container>

          <!-- MOSTRrar tabla -->
          <div v-if="seccionSeleccionada[0] === 'mostrar'">
            <h2 class="text-h5 mb-4">Llista de Tasques</h2>

            <v-data-table
              :headers="headers"
              :items="tasques"
              item-value="id"
              hide-default-footer
              class="elevation-1"
            >
              <!-- descrip -->
              <template #item.description="{ value }">
                <span v-if="value">{{ value }}</span>
                <span v-else class="text-grey-lighten-1 italic">Sense descripció</span>
              </template>

              <!-- Switch estado  -->
              <template #item.estat="{ item }">
                <v-switch
                  v-model="item.estat"
                  true-value="Completada"
                  false-value="Pendent"
                  :label="item.estat"
                  :color="item.estat === 'Completada' ? 'success' : 'warning'"
                  hide-details
                  density="compact"
                  @change="guardarTascaLocalStorage"
                ></v-switch>
              </template>

              <!--  Eliminar -->
              <template #item.eliminar="{ item }">
                <v-btn
                  icon="mdi-delete"
                  color="error"
                  variant="text"
                  density="compact"
                  @click="eliminarTasca(item)"
                >
                </v-btn>
              </template>

              <!--  Editar -->
              <template #item.editar="{ item }">
                <v-btn
                  icon="mdi-pencil"
                  color="blue"
                  variant="text"
                  density="compact"
                  @click="prepararEdicio(item)"
                >
                </v-btn>
              </template>

              <!-- si esta vacio-->
              <template #no-data>
                <div class="pa-4 text-center text-grey">
                  <p>No hi ha cap tasca per mostrar.</p>
                </div>
              </template>
            </v-data-table>
          </div>

          <!-- añadir una nueva -->
          <div v-else-if="seccionSeleccionada[0] === 'afegir'">
            <v-card class="pa-4 mx-auto" max-width="500">
              <v-card-title class="text-h5 mb-4">Afegir Nova Tasca</v-card-title>

              <v-text-field
                v-model="novaTasca.title"
                label="Títol de la tasca"
                variant="outlined"
              ></v-text-field>

              <v-textarea
                v-model="novaTasca.description"
                label="Descripció (opcional)"
                variant="outlined"
                rows="2"
              ></v-textarea>

              <v-switch
                v-model="estaCompletada"
                :label="`Estat inicial: ${estaCompletada ? 'Completada' : 'Pendent'}`"
                color="success"
                hide-details
                class="mb-4"
              ></v-switch>

              <v-btn
                color="primary"
                :disabled="!novaTasca.title.trim()"
                @click="afegirTasca"
              >
                Guardar Tasca
              </v-btn>
            </v-card>
          </div>

          <!-- para editar tasca existente -->
          <v-dialog v-model="dialogEditar" max-width="500px">
            <v-card class="pa-4">
              <v-card-title class="text-h5 mb-2">Editar Tasca</v-card-title>

              <v-card-text class="pa-0">
                <v-text-field
                  v-model="tascaAEditar.title"
                  label="Títol de la tasca"
                  variant="outlined"
                  class="mb-2"
                ></v-text-field>

                <v-textarea
                  v-model="tascaAEditar.description"
                  label="Descripció (opcional)"
                  variant="outlined"
                  rows="2"
                  class="mb-2"
                ></v-textarea>

                <v-switch
                  v-model="tascaAEditar.estat"
                  true-value="Completada"
                  false-value="Pendent"
                  :label="`Estat: ${tascaAEditar.estat}`"
                  color="success"
                  hide-details
                ></v-switch>
              </v-card-text>

              <v-card-actions class="mt-4 pa-0">
                <v-spacer></v-spacer>
                <v-btn color="grey" variant="text" @click="dialogEditar = false">
                  Cancelar
                </v-btn>
                <v-btn
                  color="primary"
                  variant="elevated"
                  :disabled="!tascaAEditar.title.trim()"
                  @click="guardarEdicio"
                >
                  Guardar Canvis
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>

        </v-container>
      </v-main>
    </v-layout>
  </v-card>
</template>

<script setup>
  import { ref, onMounted } from 'vue'

  const drawer = ref(false)
  const seccionSeleccionada = ref(['mostrar'])

  // nav
  const items = [
    { title: 'Mostrar Taula', value: 'mostrar' },
    { title: 'Afegir una tasca nova', value: 'afegir' },
  ]

  // Columnas de la tabla de tascas
  const headers = [
    { title: 'Títol', key: 'title' },
    { title: 'Descripció', key: 'description' },
    { title: 'Estat', key: 'estat' },
    { title: 'Eliminar', key: 'eliminar' },
    { title: 'Editar', key: 'editar' },
  ]

  // array donde se guardan las tascas
  const tasques = ref([])

  // crear nuevas tascas
  const novaTasca = ref({
    title: '',
    description: '',
  })
  const estaCompletada = ref(false)

  // edit
  const dialogEditar = ref(false)
  const tascaAEditar = ref({
    id: null,
    title: '',
    description: '',
    estat: 'Pendent',
  })

  // guardar en el array las tascas con el  localStorage
  function guardarTascaLocalStorage () {
    localStorage.setItem('guardaTasques', JSON.stringify(tasques.value))
  }

  // ccargar datos del localStorage
  function cargarTasquesLocalStorage () {
    const cargarTasques = localStorage.getItem('guardaTasques')
    if (cargarTasques) {
      tasques.value = JSON.parse(cargarTasques)
    } else {
      tasques.value = [
        {
          id: 1,
          title: 'Exemple',
          description: 'A Cepeda le gusta cepear en un Cepedin',
          estat: 'Pendent',
        },
      ]
      guardarTascaLocalStorage()
    }
  }

  // añadir una nueva tasca a la array y guardar que se añadio
  function afegirTasca () {
    if (!novaTasca.value.title.trim()) return

    tasques.value.push({
      id: Date.now(),
      title: novaTasca.value.title,
      description: novaTasca.value.description,
      estat: estaCompletada.value ? 'Completada' : 'Pendent',
    })

    guardarTascaLocalStorage()

    novaTasca.value = { title: '', description: '' }
    estaCompletada.value = false
    seccionSeleccionada.value = ['mostrar']
  }

  // eliminar tasca
  function eliminarTasca (itemAEliminar) {
    tasques.value = tasques.value.filter(tasca => tasca.id !== itemAEliminar.id)
    guardarTascaLocalStorage()
  }

  // pPreparar datos para editar
  function prepararEdicio (item) {
    tascaAEditar.value = { ...item }
    dialogEditar.value = true
  }

  // Guardar cambios de la edición
  function guardarEdicio () {
    if (!tascaAEditar.value.title.trim()) return

    const index = tasques.value.findIndex(t => t.id === tascaAEditar.value.id)
    if (index !== -1) {
      tasques.value[index] = { ...tascaAEditar.value }
      guardarTascaLocalStorage()
    }

    dialogEditar.value = false
  }

  // Inicializar al montar
  onMounted(() => {
    cargarTasquesLocalStorage()
  })
</script>