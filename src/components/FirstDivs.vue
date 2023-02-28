<script setup lang="ts">
defineProps<{
    msg: string;
}>();
</script>

<script lang="ts">
import mapboxgl, { type LngLatLike } from "mapbox-gl";
export default {
    data() {
        return {
            accessToken:
                "pk.eyJ1IjoiYnJ1bm9hY21kcyIsImEiOiJjbGU3NmI4bnAwMmczM3duYnhjdzFiYzg3In0.U1QV92f0JnBZ1oJQvgJXDQ",
            latLong: "",
            age: 30,
            firstname: "Jean",
            year: 2022,
            map: "" as any,
            remember: false,
            buttonVisible: false,
            family: [
                {
                    firstname: "Jojo",
                    lastname: "Bernard",
                    age: "25",
                    phoneNumber: "0606060606",
                    address: "25 Rue du Maréchal Leclerc 74000 Annecy",
                    sexe: "Male",
                },
                {
                    firstname: "Marie",
                    lastname: "Blachère",
                    age: "29",
                    phoneNumber: "0606060606",
                    address: "4 Rue du Levant 74000 Annecy",
                    sexe: "Female",
                },
                {
                    firstname: "Jean",
                    lastname: "Bernard",
                    age: "20",
                    phoneNumber: "0606060606",
                    address: "17 rue Jean Jaures 74000 Annecy",
                    sexe: "Male",
                },
                {
                    firstname: "Paul",
                    lastname: "Pogba",
                    age: "17",
                    phoneNumber: "0606060606",
                    address: "31 Avenue de Loverchy 74000 Annecy",
                    sexe: "Star",
                },
            ],
        };
    },

    methods: {
        changeName(remember: boolean) {
            this.remember = !this.remember;
            if (remember) return (this.firstname = "Jean");
            else {
                return (this.firstname = "Patrick");
            }
        },
        increaseAgeAndyear() {
            this.age++;
            this.year++;
        },
        addFamilyMember(submitEvent: any) {
            this.family.push({
                firstname: submitEvent.target.elements.firstname.value,
                lastname: submitEvent.target.elements.lastname.value,
                age: submitEvent.target.elements.age.value,
                address: submitEvent.target.elements.addresse.value,
                phoneNumber: submitEvent.target.elements.phoneNumber.value,
                sexe: submitEvent.target.elements.sexe.value,
            });
            mapboxgl.accessToken = this.accessToken;
            const mapboxClient = mapboxSdk({ accessToken: mapboxgl.accessToken });
            mapboxClient.geocoding
                .forwardGeocode({
                    query: submitEvent.target.elements.addresse.value,
                    autocomplete: false,
                    limit: 1
                })
                .send()
                .then((response) => {
                    if (
                        !response ||
                        !response.body ||
                        !response.body.features ||
                        !response.body.features.length
                    ) {
                        console.error('Invalid response:');
                        console.error(response);
                        return;
                    }
                    const feature = response.body.features[0];
                    new mapboxgl.Marker({ color: 'red', rotation: 50 })
                    .setLngLat(feature.center)
                    .setPopup(new mapboxgl.Popup({ offset: 25 }) // add popups
                                .setHTML(
                                    `<h3 style="color: #333333;">${submitEvent.target.elements.firstname.value}</h3><p style="color: #333333;">${submitEvent.target.elements.age.value} ans</p>`
                                )
                            )
                    .addTo(this.map);
                });
        },
        deleteUser(indexOfFamilyMember: number) {
            this.family.splice(indexOfFamilyMember, 1);
        },
        toggleVisibleButton() {
            this.buttonVisible = !this.buttonVisible;
        },

        loopThroughMemberAndAddMarkers() {
            Object.keys(this.family).forEach((key) => {
                let member = this.family[key];
                const mapboxClient = mapboxSdk({ accessToken: mapboxgl.accessToken });
                mapboxClient.geocoding
                    .forwardGeocode({
                        query: member.address,
                        autocomplete: false,
                        limit: 1
                    })
                    .send()
                    .then((response) => {
                        if (
                            !response ||
                            !response.body ||
                            !response.body.features ||
                            !response.body.features.length
                        ) {
                            console.error('Invalid response:');
                            console.error(response);
                            return;
                        }
                        const feature = response.body.features[0];
                        new mapboxgl.Marker({ color: 'Yellow', rotation: 25 })
                            .setLngLat(feature.center)
                            .setPopup(new mapboxgl.Popup({ offset: 25 }) // add popups
                                .setHTML(
                                    `<h3 style="color: #333333;">${member.firstname}</h3><p style="color: #333333;">${member.age} ans</p>`
                                )
                            )
                            .addTo(this.map);
                    });
            });
        },
    },
    mounted() {
        setInterval(this.increaseAgeAndyear, 3000);
        this.orderedUsers, (mapboxgl.accessToken = this.accessToken);
        mapboxgl.accessToken = this.accessToken
        this.map = new mapboxgl.Map({
            container: "mapContainer",
            style: "mapbox://styles/mapbox/streets-v11",
            center: [6.0934836, 45.8899344],
            zoom: 12,
        });
        let marker = new mapboxgl.Marker()
            .setLngLat([6.0934836, 45.8899344]) // Set the marker's longitude and latitude
            .addTo(this.map); // Add the marker to the map

        /*this.map.on('drag',  () => {
            let centro = this.map.getCenter();
            marker.setLngLat(centro);
        });
        this.map.on('zoom',  () => {
            let centro = this.map.getCenter();
            marker.setLngLat(centro);
        });*/
        this.loopThroughMemberAndAddMarkers();
    },
    computed: {
        orderedUsers: function () {
            return this.family.sort(function (a, b) {
                let ageA = parseInt(a.age);
                let ageB = parseInt(b.age);
                return ageA - ageB;
            });
        },
    },
};
</script>

<template>
    <div id="presentation ">
        <p>Bonjour je m'appelle {{ firstname }}</p>
    </div>
    <br />
    <div id="age">
        <p>En {{ year }}, Jean aura {{ age }} ans</p>
    </div>
    <br />
    <div id="family" v-for="(member, index) in family" :key="member.firstname">
        <p>
            {{ member.firstname }}, {{ member.lastname }}, {{ member.age }},
            {{ member.address }}, {{ member.phoneNumber }}, {{ member.sexe }}
        </p>
        <button @click="deleteUser(index)">Supprime !</button>
    </div>
    <button v-if="buttonVisible" id="crazy-button" @click="changeName(remember)">
        Change le nom !
    </button>
    <button @click="toggleVisibleButton" id="toggleCrazy">
        Change la visibilité du crazy-button
    </button>

    <form @submit.prevent="addFamilyMember">
        <label for="firstname">Prénom:</label><br />
        <input type="text" id="firstname" name="firstname" placeholder="John" /><br />
        <label for="lastname">Nom:</label><br />
        <input type="text" id="lastname" name="lastname" placeholder="Doe" /><br />
        <label for="age">Age:</label><br />
        <input type="text" id="age" name="age" placeholder="28" min="10" /><br />
        <label for="sexe">Sexe:</label><br />
        <input type="text" id="sexe" name="sexe" placeholder="Male" /><br />
        <mapbox-address-autofill
            access-token="pk.eyJ1IjoiYnJ1bm9hY21kcyIsImEiOiJjbGU3NmI4bnAwMmczM3duYnhjdzFiYzg3In0.U1QV92f0JnBZ1oJQvgJXDQ">
            <label for="addresse">Addresse:</label><br />
            <input type="text" id="addresse" name="addresse" placeholder="25 Rue"
                autocomplete="shipping street-address" /><br />
        </mapbox-address-autofill>
        <label for="phoneNumber">Numéro:</label><br />
        <input type="tel" id="phoneNumber" name="phoneNumber" placeholder="0606060606" /><br />
        <button id="add-family-member" type="submit">Ajoute un membre</button>
    </form>
<div id="mapContainer" class="basemap"></div>
</template>

<style>
#app {
    background-color: black;
    color: white;
    max-width: 4000px;
    max-height: 2000px;
}

#toggleCrazy {
    width: 100px;
    height: 100px;
}

#crazy-button {
    width: 100px;
    height: 100px;
}

#mapContainer {
    width: 600px;
    height: 600px;
}
</style>
