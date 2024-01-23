<template>
    <v-app>
        <v-app-bar>
            <v-app-bar-title>Mona Lisa</v-app-bar-title>
            <v-spacer></v-spacer>
            <v-switch class="mt-6" label="Current Contacts Only" color="primary" v-model="currentlyKnown"></v-switch>
            <v-spacer></v-spacer>
            <v-combobox class="mt-6" density="compact" variant="solo" label="By Location" v-model="locationSelect"
                :items="locations" clearable></v-combobox>

            <v-spacer></v-spacer>
            <v-text-field label="By name" density="compact" variant="solo" append-inner-icon="mdi-magnify" single-line
                hide-details v-model="nameSearch" clear-icon="mdi-close-circle" clearable>
            </v-text-field>

        </v-app-bar>
        <v-main>

            <v-row>
                <v-col>
                    <v-card variant="outlined" class="my-4 pa-2 mx-auto" max-width="400" v-for="npc in searchResults">
                        <v-img max-height="300" v-bind:src="npc.image"><v-overlay v-if="npc.deceased" v-model="overlay"
                                contained absolute class="align-center justify-center"><span
                                    class="text-h5 text-red bg-black font-weight-bold">DECEASED</span></v-overlay></v-img>
                        <v-card-title>{{ npc.name }}<span v-if="npc.pronouns" class="text-body-2"> ({{
                            npc.pronouns
                        }})</span></v-card-title>
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
const npcList = ref([])
const locations = ref([])
const nameSearch = ref('')
const searchResults = computed(() => updateSearchResults())
const locationSelect = ref('')
const currentlyKnown = ref()
const overlay = true

Airtable.configure({
    endpointUrl: "https://api.airtable.com",
    apiKey: 'patCs1veReEJPZ4vR.07fe2b3ceae4ec040302c98f6541d5bbe4f5dd9d15bd8847a651bc8dfda33d9f'
});
const base = Airtable.base('appduBHdfZ4KDNSqi');

base('Table 1')
    .select({
    }).eachPage(function page(records, next) {
        records.forEach(function (record) {

            try {

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
                    npc.image = "/not_found.png"
                } else {
                    npc.image = record.get('Image')[0].url
                }

                if (record.get('Faction')) {
                    npc.faction = record.get('Faction')
                }

                if (record.get('Deceased')) {
                    npc.deceased = true
                }

                if (record.get('Pronouns')) {
                    npc.pronouns = record.get('Pronouns')
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

    //if there are no name or location searches, return everything, but sorted
    if (!nameSearch.value && !locationSelect.value) {
        console.log("No filters!")
        npcList.value = npcs.value.sort(compare)
    }

    //if there are no location searches, return all name search matches, but sorted
    else if (!locationSelect.value) {
        npcList.value = npcs.value.filter(npc =>
            npc.name.toLowerCase().includes(nameSearch.value.toLowerCase())
        ).sort(compare)
    }

    //if there are no name searches, return all location search matches, but sorted
    else if (!nameSearch.value) {
        npcList.value = npcs.value.filter(npc =>
            npc.location.includes(locationSelect.value)
        ).sort(compare)
    }

    //assuming both searches exist, return everything that matches both searches, but sorted
    else {
        npcList.value = npcs.value.filter(npc =>
            npc.location.includes(locationSelect.value) && npc.name.toLowerCase().includes(nameSearch.value.toLowerCase())
        )
    }

    //take the list from above, and filter by the known switch if relevant.
    if (currentlyKnown.value) {
        console.log("known only")
        return npcList.value.filter(npc =>
            npc.current == true)
    }
    else {
        return npcList.value
    }

}

//little sorting script to alphabetize
function compare(a, b) {
    if (a.name < b.name) { return -1 }
    if (a.name > b.name) { return 1 }
    return 0
}


console.log("npcs: ", npcs.value)


</script>