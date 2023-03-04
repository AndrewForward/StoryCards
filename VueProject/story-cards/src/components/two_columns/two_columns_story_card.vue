<style>
.two_columns_story_card {
    background-color: bisque;
    padding: 5px;
    margin-bottom: 5px;
}
.two_columns_story_card:hover {
    background-color: rgb(255, 242, 227);
}
</style>

<script>
export default {
    data() {
        return {
            card_data: {},
            story_posted_at: {
                total_milliseconds: 0,
                minutes: 0,
                hours: 0,
                days: 0,
                date: 0,
            },
        }
    },

    props: {
        card_id: Number,
    },

    emits: ['load_comments'],

    created: function() {
        fetch('https://hacker-news.firebaseio.com/v0/item/' + this.card_id + '.json')
        .then(res => res.json())
        .then(out => {
            this.card_data = out;

            this.story_posted_at.date = new Date(this.card_data.time);
            this.story_posted_at.total_milliseconds = Date.now() / 1000 - this.card_data.time;
            this.story_posted_at.minutes = Math.round(this.story_posted_at.total_milliseconds / 60 % 60);
            this.story_posted_at.hours = Math.round(this.story_posted_at.total_milliseconds / 60 / 60 % 24);
            this.story_posted_at.days = Math.round(this.story_posted_at.total_milliseconds / 60 / 60 / 24);
        })
        .catch(error => { throw error });
    },
}
</script>

<template>
    <div class="two_columns_story_card" @click="$emit('load_comments', card_data)">
        <h3 class="two_columns_story_card_title">{{ card_data.title }}</h3>
        <p class="two_columns_story_card_url">{{ card_data.url }}</p>
        <p class="two_columns_story_card_score">{{ card_data.score }} points</p>
        <p class="two_columns_story_card_by">by {{ card_data.by }}</p>
        <p class="two_columns_story_card_time">days: {{ story_posted_at.days }}, hours: {{ story_posted_at.hours }}, minutes: {{ story_posted_at.minutes }}</p>
    </div>
</template>