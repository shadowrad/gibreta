<template>
  <q-page padding>
    <q-card>
      <q-card-section>
        <q-input v-model="note_name" label="Nombre" />
        <q-btn color="primary" label="Cargar" @click="loadData" />
      </q-card-section>
      <q-card-section>
        <q-editor v-model="description" placeholder="Escribe tu descripción aquí..." />
      </q-card-section>

      <q-card-section>
        <q-file v-model="file" label="Subir fotos" multiple @update:model-value="onFileUpload" />
        <div class="row q-mt-md">
          <q-img v-for="(photo, index) in photos" :key="index" :src="photo" class="thumb" />
        </div>
      </q-card-section>

      <q-card-section>
        <div id="map" style="height: 300px"></div>
      </q-card-section>

      <q-card-section>
        <q-btn color="primary" label="Guardar" @click="saveData" />
        <q-btn
          color="secondary"
          label="Volver a posición original"
          @click="resetPosition"
          class="q-ml-md"
        />
      </q-card-section>
    </q-card>
  </q-page>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useQuasar } from 'quasar'
import L from 'leaflet'

const $q = useQuasar()
const note_name = ref('')
const description = ref('')
const photos = ref([])
const file = ref(null)
const originalPosition = ref({ lat: 41.3851, lng: 2.1734 }) // Barcelona por defecto
const position = ref({ ...originalPosition.value })
const map = ref(null)
const marker = ref(null)

onMounted(() => {
  const savedData = JSON.parse(localStorage.getItem('noteData_' + note_name.value))
  if (savedData) {
    description.value = savedData.description
    photos.value = savedData.photos
    position.value = savedData.position
    originalPosition.value = { ...savedData.position }
  }
  initMap()
})

const initMap = () => {
  map.value = L.map('map').setView([position.value.lat, position.value.lng], 13)
  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map.value)

  marker.value = L.marker([position.value.lat, position.value.lng], { draggable: true }).addTo(
    map.value,
  )

  marker.value.on('dragend', (e) => {
    position.value = e.target.getLatLng()
  })

  // Evento para setear la posición al hacer click en el mapa
  map.value.on('click', (e) => {
    position.value = e.latlng
    marker.value.setLatLng(e.latlng)
  })
}

const setPosition = () => {
  marker.value.setLatLng(position.value)
  map.value.setView(position.value, 13)
  $q.notify({ message: 'Posición seteada!', type: 'info' })
}

const loadData = () => {
  const savedData = JSON.parse(localStorage.getItem('noteData_' + note_name.value))
  if (savedData) {
    description.value = savedData.description
    photos.value = savedData.photos
    position.value = savedData.position
    originalPosition.value = { ...savedData.position }
  }
  setPosition()
  $q.notify({ message: 'Datos levantados!', type: 'positive' })
}

const saveData = () => {
  localStorage.setItem(
    'noteData_' + note_name.value,
    JSON.stringify({
      description: description.value,
      photos: photos.value,
      position: position.value,
    }),
  )
  originalPosition.value = { ...position.value } // Actualizar posición original después de guardar
  $q.notify({ message: 'Datos guardados!', type: 'positive' })
}

// Volver a la posición original
const resetPosition = () => {
  position.value = { ...originalPosition.value }
  marker.value.setLatLng(originalPosition.value)
  map.value.setView(originalPosition.value, 13)
  $q.notify({ message: 'Posición restablecida!', type: 'info' })
}

const onFileUpload = (files) => {
  if (!files) return
  photos.value = []
  for (let file of files) {
    const reader = new FileReader()
    reader.onload = (e) => photos.value.push(e.target.result)
    reader.readAsDataURL(file)
  }
}
</script>

<style scoped>
.thumb {
  width: 100px;
  height: 100px;
  margin: 5px;
  object-fit: cover;
}
</style>
