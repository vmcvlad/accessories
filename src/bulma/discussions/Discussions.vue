<template>
    <div class="discussions-wrapper">
        <inputor class="raises-on-hover animated fadeIn"
            :message="discussion"
            placeholder="Share your idea..."
            title
            type="discussion"
            @store="store()"
            @update="update()"
            @cancel="
                inputor = false;
                discussion = discussion.id ? discussion : null
            "
            v-if="inputor"/>
        <discussion class="animated fadeIn"
            :discussion="discussion"
            @edit="inputor = true"
            @back="discussion = null; get()"
            @delete="destroy()"
            v-else-if="discussion"/>
        <div v-else>
            <div class="controls has-margin-bottom-large">
                <button class="button"
                    @click="discussion = factory(); inputor = true;">
                    <span v-if="!isMobile">
                        {{ i18n('New Topic') }}
                    </span>
                    <span class="icon is-small">
                        <fa icon="plus"/>
                    </span>
                </button>
                <button class="button has-margin-left-small"
                    @click="fetch()">
                    <span v-if="!isMobile">
                        {{ i18n('Reload') }}
                    </span>
                    <span class="icon is-small">
                        <fa icon="sync"/>
                    </span>
                </button>
                <p class="control has-icons-left has-margin-left-small">
                    <input type="search"
                        class="input is-rounded"
                        v-model="query">
                    <span class="icon is-small is-left">
                        <fa icon="search"/>
                    </span>
                </p>
            </div>
            <div class="box has-background-light raises-on-hover"
                v-for="(topic, index) in filteredDiscussions"
                :key="index">
                <discussion-preview class="is-clickable"
                    :discussion="topic"
                    :last="index === discussions.length - 1"
                    @click.native="discussion = topic"/>
            </div>
        </div>
    </div>
</template>

<script>
import { mapState } from 'vuex';
import { library } from '@fortawesome/fontawesome-svg-core';
import { faPlus, faSearch, faSync } from '@fortawesome/free-solid-svg-icons';
import Discussion from './Discussion.vue';
import DiscussionPreview from './DiscussionPreview.vue';
import Inputor from './Inputor.vue';

library.add(faPlus, faSearch, faSync);

export default {
    name: 'Discussions',

    components: { Discussion, DiscussionPreview, Inputor },

    inject: ['errorHandler', 'i18n'],

    props: {
        id: {
            type: Number,
            required: true,
        },
        type: {
            type: String,
            required: true,
        },
    },

    data: () => ({
        query: '',
        inputor: false,
        discussion: null,
        discussions: [],
        loading: false,
    }),

    computed: {
        ...mapState('layout', ['isMobile']),
        filteredDiscussions() {
            const query = this.query.toLowerCase();

            return query
                ? this.discussions.filter(({ title, body }) => title.toLowerCase().indexOf(query) >= 0
                    || body.toLowerCase().indexOf(query) >= 0)
                : this.discussions;
        },
        count() {
            return this.filteredDiscussions.length;
        },
    },

    created() {
        this.fetch();
    },

    methods: {
        fetch() {
            this.loading = true;

            axios.get(route('core.discussions.index'), {
                params: {
                    discussable_id: this.id,
                    discussable_type: this.type,
                },
            }).then(({ data }) => {
                this.discussions = data;
                this.$emit('update');
                this.loading = false;
            }).catch(this.errorHandler);
        },
        store() {
            axios.post(route('core.discussions.store'), this.discussion)
                .then(({ data }) => {
                    this.discussion = data;
                    this.discussions.unshift(this.discussion);
                    this.$emit('update');
                    this.inputor = false;
                }).catch(this.errorHandler);
        },
        update() {
            axios.patch(
                route('core.discussions.update', this.discussion.id),
                this.discussion,
            )
                .then(() => (this.inputor = false))
                .catch(this.errorHandler);
        },
        destroy() {
            axios.delete(route('core.discussions.destroy', this.discussion.id))
                .then(() => {
                    const index = this.discussions.findIndex(({ id }) => id === this.discussion.id);
                    this.discussions.splice(index, 1);
                    this.$emit('update');
                    this.discussion = null;
                    this.fetch();
                }).catch(this.errorHandler);
        },
        factory() {
            return {
                id: null,
                discussable_id: this.id,
                discussable_type: this.type,
                title: null,
                body: null,
            };
        },
    },
};
</script>

<style lang="scss">
    .discussions-wrapper {
        .header {
            border-bottom: 2px solid black;
            margin-bottom: 2rem;
            padding-top: 0.5em;
            padding-bottom: 0.5em;
        }

        .controls {
            display: flex;
            justify-content: center;
        }
    }
</style>
