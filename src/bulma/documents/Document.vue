<template>
    <div class="document-wrapper level has-margin-top-medium">
        <div class="level-left">
            <div class="level-item">
                <span class="icon is-small has-margin-small">
                    <fa icon="file"/>
                </span>
                <span>
                    {{ shortName(file.name) }}
                </span>
            </div>
        </div>
        <div class="level-right">
            <transition enter-active-class="animated fadeIn"
                leave-active-class="animated fadeOut">
                <div class="level-item has-text-grey">
                    <button v-if="canAccess('core.files.link')"
                        class="button is-naked"
                        @click="link">
                        <span class="icon">
                            <fa icon="link"/>
                        </span>
                    </button>
                    <button class="button is-naked"
                        @click="show">
                        <span class="icon">
                            <fa icon="eye"/>
                        </span>
                    </button>
                    <a class="button is-naked"
                        :href="downloadLink">
                        <span class="icon">
                            <fa icon="cloud-download-alt"/>
                        </span>
                    </a>
                    <confirmation @confirm="$emit('delete')">
                        <button class="button is-naked">
                            <span class="icon">
                                <fa icon="trash-alt"/>
                            </span>
                        </button>
                    </confirmation>
                    <v-popover
                        trigger="hover"
                        placement="top">
                        <span class="icon">
                            <fa icon="info-circle"/>
                        </span>
                        <template v-slot:popover>
                            <div class="info">
                                <p>
                                    <span class="icon is-small">
                                        <fa icon="user"/>
                                    </span>
                                    {{ file.owner.name }}
                                </p>
                                <p>
                                    <span class="icon is-small">
                                        <fa icon="calendar-alt"/>
                                    </span>
                                    {{ timeFromNow(file.createdAt) }}
                                </p>
                                <p>
                                    <span class="icon is-small">
                                        <fa icon="database"/>
                                    </span>
                                    {{ $options.filters.numberFormat(file.size) }} Kb
                                </p>
                            </div>
                        </template>
                    </v-popover>
                </div>
            </transition>
            <url :show="temporaryLink !== ''"
                :link="temporaryLink"
                @close="temporaryLink = ''"/>
        </div>
    </div>
</template>

<script>
import { VPopover } from 'v-tooltip';
import { library } from '@fortawesome/fontawesome-svg-core';
import {
    faFile, faEye, faCloudDownloadAlt, faTrashAlt, faLink,
    faInfoCircle, faUser, faCalendarAlt, faDatabase,
} from '@fortawesome/free-solid-svg-icons';
import Confirmation from '@enso-ui/confirmation/bulma';
import formatDistance from '@core-modules/plugins/date-fns/formatDistance';
import Url from '@core-pages/files/components/Url.vue';

library.add([
    faFile, faEye, faCloudDownloadAlt, faTrashAlt, faLink,
    faInfoCircle, faUser, faCalendarAlt, faDatabase,
]);

export default {
    name: 'Documents',

    components: { VPopover, Confirmation, Url },

    inject: ['canAccess', 'errorHandler'],

    props: {
        file: {
            type: Object,
            required: true,
        },
    },

    data: () => ({
        temporaryLink: '',
    }),

    computed: {
        downloadLink() {
            return route('core.files.download', this.file.id);
        },
        openLink() {
            return route('core.files.show', this.file.id);
        },
    },

    methods: {
        link() {
            axios.get(route('core.files.link', this.file.id))
                .then(({ data }) => (this.temporaryLink = data.link))
                .catch(this.errorHandler);
        },
        show() {
            window.open(this.openLink, '_blank').focus();
        },
        timeFromNow(date) {
            return formatDistance(date);
        },
        shortName(name) {
            return name.length > 20
                ? `${name.substring(0, 20)}...`
                : name;
        },
    },
};
</script>

<style lang="scss">
    .document-wrapper {
        padding: .2rem;
        width: 100%;
        -webkit-box-shadow: 0px 0px 3px 1px rgba(133,133,133,1);
        box-shadow: 0px 0px 3px 1px rgba(133,133,133,1);
        border-radius: 5px;

        &:hover {
            border-left: 4px solid #3273dc;
        }

        &:not(:last-child) {
            margin-bottom: 0.75rem;
        }
    }
</style>
