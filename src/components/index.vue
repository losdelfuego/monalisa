<template>
    <v-app>
        <v-app-bar>
            <v-app-bar-title>Mona Lisa</v-app-bar-title>
            <v-spacer></v-spacer>
            <v-combobox class="mt-6" density="compact" variant="solo" label="View by Location" v-model="locationSelect"
                :items="locations" clearable></v-combobox>

            <v-spacer></v-spacer>
            <v-text-field label="Search by NPC name" density="compact" variant="solo" append-inner-icon="mdi-magnify"
                single-line hide-details v-model="nameSearch" clear-icon="mdi-close-circle" clearable>
            </v-text-field>

        </v-app-bar>
        <v-main>

            <v-row>
                <v-col>
                    <v-card variant="outlined" class="my-4 pa-2 mx-auto" max-width="400" v-for="npc in searchResults">
                        <v-img max-height="300" v-bind:src="npc.image"></v-img>
                        <v-card-title>{{ npc.name }}</v-card-title>
                        <v-card-subtitle>{{ npc.description }}</v-card-subtitle>
                        <v-card-text>Last Known Location: {{ npc.location.join(", ") }}</v-card-text>
                        <v-card-text v-if="npc.faction">Affiliations: {{ npc.faction }}</v-card-text>
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
const locations = ref([])
const nameSearch = ref('')
const searchResults = computed(() => updateSearchResults())
const locationSelect = ref('')

Airtable.configure({
    endpointUrl: "https://api.airtable.com",
    apiKey: 'keyZ5ixWh8VGnbyu6'
});
const base = Airtable.base('appduBHdfZ4KDNSqi');

base('Table 1')
    .select({
    }).eachPage(function page(records, next) {
        records.forEach(function (record) {

            try {

                // const image = record.get('Image')
                const npc = {
                    "name": record.get('Name'),
                    "location": location,
                    "description": record.get('Description'),
                    "current": record.get('CurrentContact'),
                    "id": record.get('id')

                }
                if (!record.get('Location')) {
                    npc.location = ["Unknown"]
                } else {
                    npc.location = record.get('Location')
                }

                if (!record.get('Image')) {
                    npc.image = "src/assets/not_found.jpeg"

                } else {
                    npc.image = record.get('Image')[0].url
                }

                if (record.get('Faction')) {
                    npc.faction = record.get('Faction')
                }

                npcs.value.push(npc)

                npc.location.forEach((location) => {
                    if (!locations.value.includes(location)) {
                        locations.value.push(location)
                    }
                })


                locations.value.sort()

            } catch (err) {

                console.error(err);

            }

        })

        try {

            next()


        } catch { return }



    }, function (err) {
        if (err) {
            console.error(err);
            rej(err.statusCode);
            return;
        }
        acpt(npcs.value);
    })

function updateSearchResults() {

    if (!nameSearch.value && !locationSelect.value) {
        console.log("No filters!")
        return npcs.value.sort(compare)
    }

    if (!locationSelect.value) {
        return npcs.value.filter(npc =>
            npc.name.toLowerCase().includes(nameSearch.value.toLowerCase())
        ).sort(compare)
    }

    if (!nameSearch.value) {
        return npcs.value.filter(npc =>
            npc.location.includes(locationSelect.value)
        ).sort(compare)
    }

    return npcs.value.filter(npc =>
        npc.location === locationSelect.value && npc.name.toLowerCase().includes(nameSearch.value.toLowerCase())
    )
}


function compare(a, b) {
    if (a.name < b.name) { return -1 }
    if (a.name > b.name) { return 1 }
    return 0
}


console.log("npcs: ", npcs.value)


</script>