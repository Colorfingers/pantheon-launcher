<!--
//  Events.vue
//
//  Created by Kalila L. on 10 Jun 2020.
//  Copyright 2020 Vircadia contributors.
//
//  Distributed under the Apache License, Version 2.0.
//  See the accompanying file LICENSE or http://www.apache.org/licenses/LICENSE-2.0.html
-->
<template>
    <v-app id="">
        <v-main class="mainContentContainer" id="eventsContainer">
            <v-container
                id=""
                class="fill-container"
                fluid
            >
                <v-data-iterator
                    :items="events"
                    hide-default-footer
                >
					<template v-slot:header>
						<v-toolbar
                            color="primary"
                            dark
                            flat
                            class="mb-2"
						>
						<v-toolbar-title>Featured Events</v-toolbar-title>
						</v-toolbar>
					</template>

                    <template v-slot:default="props">
                        <v-row>
                            <v-col
                                v-for="item in props.items"
                                :key="item.name"
                                cols="12"
                                sm="6"
                                md="4"
                                lg="3"
                            >

                                <v-card class="event-cards" width="230">
                                    <v-img
                                        :src="item.img"
                                        height="200px"
                                    ></v-img>

                                    <v-card-title>
                                        {{ item.name }}
                                    </v-card-title>
                                    <v-card-subtitle>
                                        {{ getDayAndDate(item.date.fullUTC) }} <br />
                                        {{ getTimeAndTimezone(item.date.fullUTC) }}
                                    </v-card-subtitle>
                                    
                                    <v-divider></v-divider>
                                    
                                    <v-card-actions>
                                        <v-btn color="primary" text v-on:click="sendEvent('open-url', item.links.help)">
                                            Help
                                        </v-btn>
                                        
                                        <v-spacer></v-spacer>

                                        <v-btn color="purple" v-on:click="sendEvent('open-interface-url', item.links.join)">
                                            Join
                                        </v-btn>

                                    </v-card-actions>

                                </v-card>

                            </v-col>
                        </v-row>
                    </template>

				</v-data-iterator>
            </v-container>
        </v-main>
    </v-app>
</template>

<script>
var vue_this;
import { EventBus } from '../plugins/event-bus.js';

require('electron').ipcRenderer.on('events-list', (event, message) => {
    if (message.eventsList.length > 0) {
        vue_this.events = message.eventsList;
    } else {
        vue_this.sendEvent("no-events-found");
    }
})

export default {
    name: 'Events',
    methods: {
        sendEvent: function(command, data) {
            EventBus.$emit(command, data);
        },
        getDayAndDate(dateAndTime) {
            // We are assuming that the given date and time is in UTC.
            var baseUTC = this.moment.utc(dateAndTime);
            var converted = baseUTC.tz(this.userTimezone.name());
            var convertedLocale = this.moment(converted).format('dddd, MMMM Do'); // https://momentjs.com/docs/#/displaying/format/
            return convertedLocale;
        },
        getTimeAndTimezone(dateAndTime) {
            // We are assuming that the given date and time is in UTC.
            var baseUTC = this.moment.utc(dateAndTime);
            var converted = baseUTC.tz(this.userTimezone.name());
            var convertedLocale = this.moment(converted).format('h:mmA zz'); // https://momentjs.com/docs/#/displaying/format/
            return convertedLocale;
        }
    },
    computed: {
        
    },
    mounted() {
    },
    created: function () { 
        vue_this = this;
        const { ipcRenderer } = require('electron');
        
        ipcRenderer.send('check-for-events');
        
        this.userTimezone = this.$jstz.determine();
    },
    data: () => ({
        userTimezone: null,
        events: [
        ]
    }),
};
</script>
