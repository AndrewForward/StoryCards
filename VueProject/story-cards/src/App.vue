<style>
* {
    font-family: sans-serif;
}

#two_columns_contents {
    display: flex;
}

#two_columns_main {

}

#two_columns_side {
    flex: 1 0 400px;
    background-color: azure;
}
</style>

<script>
import two_columns_story_card from './components/two_columns/two_columns_story_card.vue'
import two_columns_comment from './components/two_columns/two_columns_comment.vue'

export default {
    data() {
        return {
            card_id_list: [],
            card_list: [],
            comment_list: [],
        }
    },

    components: {
        two_columns_story_card,
        two_columns_comment,
    },

    methods: {
        load_comments: function(card_data) {
            this.comment_list = [];
            for (var i = 0; i < card_data.kids.length; i++) {
                this.comment_list.push({id: card_data.kids[i]});
            }
        }
    },

    created: function () {
        fetch('https://hacker-news.firebaseio.com/v0/topstories.json')
        .then(res => res.json())
        .then(out => {
            this.card_id_list = out;
            for (var i = 0; i < 50; i++) {
                this.card_list.push({ id: this.card_id_list[i] });
            }
        })
        .catch(error => { throw error });
    },
}
</script>

<template>
    <header>
        <h2>StoryCards</h2>
    </header>

    <div id="two_columns_contents" >
        <div id="two_columns_main">
            <two_columns_story_card v-for="card in card_list" 
                                    :card_id="card.id" 
                                    @load_comments="(card_data) => load_comments(card_data)"></two_columns_story_card>
        </div>
    
        <div id="two_columns_side">
            <two_columns_comment v-for="comment in comment_list" :comment_id="comment.id"></two_columns_comment>
        </div>
    </div>

    <footer>
    </footer>
</template>