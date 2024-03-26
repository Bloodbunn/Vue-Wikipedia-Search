<template >
    <div class="pt-10" :class="{'bg-black text-white': isDarkTheme}">
        <div class="w-[90%] mx-auto p-3" >
        <div class="flex justify-between mb-5 w-[500px] max-[590px]:w-full mx-auto" >
        <h1 class="text-4xl max-[410px]:text-xl font-bold  text-blue-600 text-center ">Search Wikipedia</h1>
            <button @click="toggleTheme" class="p-1 bg-gray-300 font-bold rounded-xl max-[410px]:text-sm" :class="{'bg-gray-600 text-white': isDarkTheme}">{{ isDarkTheme ?
                'Light' : 'Dark' }}</button>
        </div>

        <form @submit.prevent="submitSearch" class="border border-gray-300 shadow-md p-3 w-[500px] max-[590px]:w-full mx-auto text-center">
            <input class="border border-blue-300 p-2 mb-3 outline-blue-400 w-full max-[410px]:p-1" :class="{'bg-black text-white': isDarkTheme}" type="text" v-model="searchQuery">
            <button class="bg-red-600 hover:bg-red-800 text-white font-bold rounded-md p-3 max-[410px]:p-2 max-[410px]:text-sm" >Search</button>


        </form>

        <div class="mt-5 w-max mx-auto text-2xl font-bold max-[410px]:text-sm" v-if="isLoading" :class="{'bg-black text-white': isDarkTheme}" >
            Loading content...
        </div>
        <div class="text-red-700" v-if="error" :class="{'bg-black text-white': isDarkTheme}">
            {{ error }}
        </div>

        <div v-if="searchResult.length" >
            <div v-for="result in searchResult" :key="result.pageid" class="p-3">
                <h3 class="text-xl mb-2 font-bold underline text-blue-600 text-left max-[410px]:text-base" :class="{'bg-black text-white': isDarkTheme}"><a target="_blank" rel="noopener"
                        :href="`https://en.wikipedia.org/?curid=${result.pageid}`">{{
                result.title }}</a></h3>
                <p class="text-gray-700 text-sm text-left max-[410px]:text-sm" v-html="`${result.snippet}...`" :class="{'bg-black text-white': isDarkTheme}"></p>
            </div>
        </div>

    </div>
    </div>
    

</template>

<script setup>
import { ref } from 'vue'

const searchQuery = ref('')
const searchResult = ref([])
const error = ref(null)
const isLoading = ref(false)
const isDarkTheme = ref(false)

const submitSearch = () => {
    if (searchQuery.value.trim() !== '') {
        searchWikipedia(searchQuery.value)
    } else {
        searchResult.value = []
        error.value = "Please enter a valid search term"
    }
}

const searchWikipedia = async (query) => {
    const encodedQuery = encodeURIComponent(query) // this method ensures our search query is suitable to be put inside a url below at the end
    const endPoint = `https://en.wikipedia.org/w/api.php?action=query&list=search&prop=info&inprop=url&utf8=&format=json&origin=*&srlimit=10&srsearch=${encodedQuery}`

    try {
        isLoading.value = true
        const response = await fetch(endPoint)
        const data = await response.json()

        if (data.query && data.query.search) {
            searchResult.value = data.query.search
            error.value = null
        } else {
            searchResult.value = []
            error.value = "No data found"
        }

    } catch (err) {
        console.error("Error Fetching Data", err)
        error.value = "An error occured while fetching data"
        searchResult.value = []
    }
    finally {
        isLoading.value = false //weather try or catch is successful at the end we want isLoading message in the template to disappear
    }


}

const toggleTheme = () => {
    isDarkTheme.value = !isDarkTheme.value
}
</script>