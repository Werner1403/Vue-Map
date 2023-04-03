<template>
    <div id="app" class="is-fullheight">
        <div class="columns is-fullheight">
            <aside class="column is-narrow">
                <section class="section">
                    <h1 class="title">Aktionen</h1>
                    <nav>
                        <ul>
                            <li>
                                <label>Zufälliger Marker</label>
                                <br>
                                <a class="button is-success" @click="NewRandomMarker">Neuer zufälliger Marker</a>
                            </li>
                            <br />
                            <label>Marker nach Ortsnamen hinzufügen</label>
                            <br>
                            <li>
                                <input class="input" type="text" v-model="locationName" placeholder="Ortsname eingeben" />
                                <br>
                                <button class="button is-info" @click="addMarkerByLocationName">Ortsmarker hinzufügen</button>
                            </li>
                        </ul>
                    </nav>
                    <br>
                    <h2 class="subtitle">Marker Liste</h2>
                    <ul>
                        <li v-for="(marker, index) in markers" :key="index">
                            <a class="button" @click="zoomToMarker(marker)">
                                <strong>{{ marker.name }}: </strong>{{ marker.lat.toFixed(4) }}, {{ marker.lng.toFixed(4) }}&#9;&#9;&#9;
                            </a>
                            <button class="button is-danger" @click.prevent="deleteMarker(index)">Löschen</button>
                        </li>
                    </ul>
                    <br>
                    <br>
                    <div>
                        <label>Eigenen Marker hinzufügen</label>
                        <input placeholder="Marker-Namen eingeben" class="input" type="text" id="customMarkerName" v-model="customMarkerName" />
                        <br>
                        <button class="button" v-bind:class="{'is-info': !addingCustomMarker, 'is-danger': addingCustomMarker}" v-bind="{'disabled': addingCustomMarker}" @click="enableCustomMarker">Eigenen Marker hinzufügen</button>
                    </div>
                    <br>
                    <br>
                    <div class="notification is-info is-light has-text-centered info-label">
                        <label>
                            Um einen Marker mit selbst gewählten Namen hinzuzufügen, geben Sie den Name in das Textfeld ein, klicken sie auf "Marker hinzufügen" und klicken Sie auf die gewünschte Position. Der button "Aktuelle Location" zeigt Ihnen auf der Karte Ihren aktuellen Aufenthaltsort an. Mit "Neuer zufälliger Marker" wird ein zufälliger Marker auf der Karte gesetzt.
                        </label>
                    </div>
                </section>
            </aside>
            <main class="column">
                <section class="section">
                    <GoogleMap
                        ref="googleMap"
                        api-key="AIzaSyDGFI1HVHzO2rStOO5msC-TAQ3h_g0kHrE"
                        :center="this.markers[0]"
                        :zoom="5"
                        class="google-map"
                        @click="addMarkerOnClick"
                        style="width: 100vw; height: 100vh"
                    >
                        <Marker :options="{ position: this.curLoc }" />
                        <InfoWindow :options="{ position: this.curLoc, content: 'Dein aktueller Standort!' }"/>
                        <CustomControl position="TOP_CENTER">
                            <br>
                            <button class="button is-success custom-btn" @click="getCurLoc">Aktuelle Location</button>
                        </CustomControl>
        
                        <Marker v-for="(marker, index) in markers" :key="'marker' + index" :options="{ position: marker }"/>
                        <Circle
                            v-for="(marker, index) in markers"
                            :key="'circle' + index"
                            :options="{
                                center: marker,
                                radius: 50000,
                                clickable: true,
                                fillColor: 'transparent',
                                strokeColor: 'transparent',
                            }"
                            @mouseover.native="showInfoWindow(index)"
                            @mouseout.native="hideInfoWindow(index)"
                        />
                        <InfoWindow v-if="currentInfoWindow" :options="{ content: currentInfoWindow.content, position: currentInfoWindow.position }"></InfoWindow>
                    </GoogleMap>
                </section>
            </main>
        </div>
    </div>
</template>
  
<script>
    import { defineComponent } from "vue";
    import { GoogleMap, Marker, InfoWindow, CustomControl, Circle } from "vue3-google-map";
  
    export default defineComponent({
        components: { GoogleMap, Marker, InfoWindow, CustomControl, Circle },
        mounted() {
            this.NewRandomMarker();
        },
  
        data() {
            return {
                curLoc: null,
                markers: [],
                currentInfoWindow: null,
                customMarkerName: "",
                addingCustomMarker: false,
                locationName: "",
            };
        },
        methods: {
            getCurLoc() {
                navigator.geolocation.getCurrentPosition(geolocation => {
                let loc = {
                    lat: +geolocation.coords.latitude,
                    lng: +geolocation.coords.longitude,
                    name: "Aktueller Standort",
                    showInfo: false,
                };
                this.curLoc = loc;
                this.markers.unshift(loc)
                });
            },
            NewRandomMarker() {
                const newCoordinates = {
                    lat: Math.random() * 180 - 90,
                    lng: Math.random() * 360 - 180,
                    name: "Zufälliger Marker " + (this.markers.length + 1),
                    showInfo: false,
                };
                this.markers.push(newCoordinates);
                console.log(this.markers);
            },
            zoomToMarker(marker) {
                this.$refs.googleMap.map.setCenter(marker);
                this.$refs.googleMap.map.setZoom(5);
            },
            showInfoWindow(index) {
                const marker = this.markers[index];
                const name = marker.name
                const coordinates = `Lat: ${marker.lat.toFixed(4)}, Lng: ${marker.lng.toFixed(4)}`;
                this.currentInfoWindow = {
                    position: marker,
                    content: 
                    `
                    <div>
                        <p>${name}</p>
                        <p>${coordinates}</p>
                    </div>
                    `,
                };
            },
            hideInfoWindow() {
                this.currentInfoWindow = null;
            },
            enableCustomMarker() {
                this.addingCustomMarker = true;
            },
            addMarkerOnClick(event) {
                if (this.addingCustomMarker) {
                const newMarker = {
                    lat: event.latLng.lat(),
                    lng: event.latLng.lng(),
                    name: this.customMarkerName || `Marker ${this.markers.length + 1}`,
                    showInfo: false,
                };
                this.markers.push(newMarker);
                this.addingCustomMarker = false;
                this.customMarkerName = "";
                }
            },
            deleteMarker(index) {
                this.markers.splice(index, 1);
            },
            addMarkerByLocationName() {
                if (!this.locationName) return;

                const geocoder = new window.google.maps.Geocoder();

                geocoder.geocode({ address: this.locationName }, (results, status) => {
                if (status === window.google.maps.GeocoderStatus.OK) {
                    const location = results[0].geometry.location;
                    const newMarker = {
                    lat: location.lat(),
                    lng: location.lng(),
                    name: this.locationName,
                    showInfo: false,
                    };
                    this.markers.push(newMarker);
                    this.locationName = "";
                } else {
                    alert("Ort konnte nicht gefunden werden: " + status);
                }
                });
            },
        },
    });
</script>
  
<style>
    html, body, #app {
        margin: 0;
        padding: 0;
        width: 100%;
        height: 100%;
        overflow: hidden;
    }
    
    .google-map {
        width: 100%;
        height: 100%;
        margin: 0;
        padding: 0;
    }

    .info-label {
    width: 100%;
    max-width: 250px;
    word-wrap: break-word;
    }
</style>
  