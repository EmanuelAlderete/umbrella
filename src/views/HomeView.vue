<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
        type="text"
        @input="getSearchResults"
        v-model="searchQuery"
        placeholder="Search for a city or state"
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004371]"
      />
      <ul
        v-show="searchQuery !== ''"
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
      >
        <p v-if="searchError">Hmm... algo não funcionou direito. Já estamos vendo isso!</p>
        <p v-else-if="mapboxSearchResults.length === 0">
          Nada por aqui... talvez outro termo funcione melhor!
        </p>
        <template v-else>
          <li
            v-for="(searchResult, index) in mapboxSearchResults"
            :key="index"
            class="py-2 cursor-pointer"
            @click="previewCity(searchResult)"
          >
            {{ searchResult.properties.full_address }}
          </li>
        </template>
      </ul>
    </div>
  </main>
</template>

<script setup>
import axios from 'axios'
import { ref } from 'vue'
import { useRouter } from 'vue-router'

const mapboxAPIKey =
  'pk.eyJ1IjoiZW1hbnVlbGFsZGVyZXRlIiwiYSI6ImNtOXU3MmphcjA2M3YyaXEzczRybm15OW8ifQ.aF3R9vZlJu67P2vCvHsC1g'

const searchQuery = ref('')
const queryTimeout = ref(null)
const mapboxSearchResults = ref([])
const searchError = ref(false)

const getSearchResults = () => {
  queryTimeout.value = setTimeout(async () => {
    clearTimeout(queryTimeout.value)
    if (searchQuery.value.trim() !== '') {
      try {
        const result = await axios.get(
          `https://api.mapbox.com/search/geocode/v6/forward?q=${encodeURIComponent(searchQuery.value)}&access_token=${mapboxAPIKey}&types=place`,
        )
        mapboxSearchResults.value = result.data.features || []
        searchError.value = false
      } catch {
        searchError.value = true
        mapboxSearchResults.value = []
      }
    } else {
      mapboxSearchResults.value = []
      searchError.value = false
    }
  }, 1000)
}

const router = useRouter()

const previewCity = (searchResult) => {
  const [city, state] = searchResult.properties.full_address.split(',')
  router.push({
    name: 'cityView',
    params: { state: state.replaceAll(' ', ''), city: city.replaceAll(' ', '') },
    query: {
      lat: searchResult.geometry.coordinates[1],
      lng: searchResult.geometry.coordinates[0],
      preview: true,
    },
  })
}
</script>
