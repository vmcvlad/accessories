<template>
    <div class="discussion-wrapper">
        <div class="box has-background-light raises-on-hover">
            <article class="media">
                <figure class="media-left">
                    <p class="image is-48x48">
                        <img class="is-rounded"
                            :src="avatar">
                    </p>
                </figure>
                <div class="media-content">
                    <div class="content">
                        <h4 class="title is-4 is-marginless">
                            {{ discussion.title }}
                        </h4>
                        <span class="has-text-info is-bold">
                            {{ discussion.owner.name }}
                        </span>
                        &bull;
                        <small class="has-text-muted">
                            {{ timeFromNow(discussion.updatedAt || discussion.createdAt) }}
                        </small>
                        <span v-if="edited">
                            &bull;
                            <small class="has-text-muted">
                                {{ i18n('edited') }}
                            </small>
                        </span>
                    </div>
                </div>
                <div class="media-right">
                    <button class="button"
                        @click="$emit('back')">
                        <span>
                            {{ i18n('Back') }}
                        </span>
                        <span class="icon is-small">
                            <fa icon="arrow-left"/>
                        </span>
                    </button>
                </div>
            </article>
            <div class="is-flex is-pulled-right"
                v-if="discussion.isEditable">
                <button class="button is-naked is-small has-margin-right-small"
                    @click="$emit('edit')">
                    <span class="icon is-small has-text-grey">
                        <fa icon="pencil-alt"/>
                    </span>
                </button>
                <confirmation placement="bottom-end"
                    @confirm="$emit('delete')"
                    @show="confirmation = true"
                    v-if="!discussion.replies.length">
                    <button class="button is-naked is-small"
                        @click="confirmation = true">
                        <span class="icon is-small has-text-grey">
                            <fa icon="trash-alt"/>
                        </span>
                    </button>
                </confirmation>
            </div>
            <div class="discussion-body has-padding-medium"
                v-html="discussion.body"/>
                <reactions class="has-margin-top-large"
                    :reactable="discussion"
                    type="discussion"/>
        </div>
        <h5 class="title is-5 is-flex">
            <span class="tag is-info is-rounded has-text-weight-bold">
                {{ discussion.replies.length }}
            </span>
            <span class="has-margin-left-medium">
                {{ i18n('Replies') }}
            </span>
        </h5>
        <reply v-for="(reply, index) in discussion.replies"
            :key="index"
            :reply="reply"
            @delete="destroy(reply, index)"
            @update="update(reply, index)"/>
        <reply :reply="reply"
            @store="store()"
            @cancel="reply = null;"
            v-if="reply"/>
        <button class="button"
            @click="reply = replyFactory()"
            v-else>
            <span>
                {{ i18n('Reply') }}
            </span>
            <span class="icon is-small">
                <fa icon="plus"/>
            </span>
        </button>
    </div>
</template>

<script>
import { mapState } from 'vuex';
import { library } from '@fortawesome/fontawesome-svg-core';
import { faArrowLeft, faTrashAlt, faPencilAlt } from '@fortawesome/free-solid-svg-icons';
import Confirmation from '@enso-ui/confirmation/bulma';
import formatDistance from '@core-modules/plugins/date-fns/formatDistance';
import Reply from './Reply.vue';
import Reactions from './Reactions.vue';

library.add(faArrowLeft, faTrashAlt, faPencilAlt);

export default {
    name: 'Discussion',

    components: { Reply, Reactions, Confirmation },

    inject: ['i18n'],

    props: {
        discussion: {
            type: Object,
            required: true,
        },
    },

    data: () => ({
        confirmation: false,
        reply: null,
    }),

    computed: {
        ...mapState(['user']),
        avatar() {
            return route(
                'core.avatars.show',
                this.discussion.owner.avatarId,
            );
        },
        edited() {
            return this.discussion.createdAt !== this.discussion.updatedAt;
        },
    },

    methods: {
        store() {
            axios.post(route('core.discussions.storeReply'), this.reply)
                .then(({ data }) => {
                    this.discussion.replies.push(data);
                    this.reply = null;
                })
                .catch(error => this.handleErorr(error));
        },
        update(reply, index) {
            axios.patch(route('core.discussions.updateReply', reply.id), reply)
                .then(({ data }) => this.discussion.replies.splice(index, 1, data))
                .catch(error => this.handleErorr(error));
        },
        destroy(reply, index) {
            axios.delete(route('core.discussions.destroyReply', reply.id))
                .then(() => this.discussion.replies.splice(index, 1))
                .catch(error => this.handleErorr(error));
        },
        timeFromNow(date) {
            return formatDistance(date);
        },
        replyFactory() {
            return {
                discussion_id: this.discussion.id,
                body: null,
                owner: {
                    name: this.user.name,
                    avatarId: this.user.avatar.id,
                },
            };
        },
    },
};
</script>

<style lang="scss">
    .discussion-wrapper .discussion-body {
        h1 {
            font-size: 2em;
        }

        h2 {
            font-size: 1.5em;
        }

        .ql-align-center {
            text-align: center;
        }

        ol, ul {
            padding-left: 3em;
        }

        img {
            width: 1.4rem;
            height: 1.4rem;
            margin-bottom: -0.3rem;
            border-radius: 290486px;
        }
    }
</style>
