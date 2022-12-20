<template>
    <v-app>
        <v-app-bar>
            <v-app-bar-title>Mona Lisa</v-app-bar-title>

            <v-spacer></v-spacer>
            <v-text-field label="Search by NPC name" density="compact" variant="solo" append-inner-icon="mdi-magnify"
                single-line hide-details v-model="nameSearch" clear-icon="mdi-close-circle" clearable>
            </v-text-field>

        </v-app-bar>
        <v-main>

            <v-row>
                <v-col>
                    <v-card variant="outlined" class="mx-auto" max-width="400" v-for="npc in searchResults">
                        <v-img max-height="300" v-bind:src="npc.image"></v-img>
                        <v-card-title>{{ npc.name }}</v-card-title>
                        <v-card-subtitle>{{ npc.description }}</v-card-subtitle>
                        <v-card-text>Last Known Location: {{ npc.location }}</v-card-text>
                    </v-card>

                </v-col>
            </v-row>
        </v-main>
    </v-app>
</template>

<script setup>
import { ref, computed } from 'vue'
import Airtable from 'airtable';

const npcs = ref([])
const nameSearch = ref('')
const locationSearch = ref('')
const searchResults = computed(() => updateSearchResults())


Airtable.configure({
    endpointUrl: "https://api.airtable.com",
    apiKey: 'keyZ5ixWh8VGnbyu6'
});
const base = Airtable.base('appduBHdfZ4KDNSqi');

base('Table 1')
    .select({
    }).eachPage(function page(records, fetchNextPage) {
        records.forEach(function (record) {
            const image = record.get('Image')
            const npc = {
                "name": record.get('Name'),
                "location": record.get('Location'),
                "description": record.get('Description'),
                "image": image[0].url,
                "current": record.get('CurrentContact'),
                "id": record.get('id')

            }
            npcs.value.push(npc)

        })
        fetchNextPage()
    })

function updateSearchResults() {
    if (!nameSearch.value) {
        console.log("No Input")
        return npcs.value
    }


    return npcs.value.filter((npc =>
        npc.name.toLowerCase().includes(nameSearch.value.toLowerCase())),
    )
}






</script>