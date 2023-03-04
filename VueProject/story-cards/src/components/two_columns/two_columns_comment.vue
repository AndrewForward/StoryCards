<style>
.two_columns_comment {
    background-color: rgb(255, 255, 255);
    margin-left: v-bind(indent * 10 + 'px');
}
</style>

<script>
export default {
    data() {
        return {
            text_type: {
                NORMAL: 'NORMAL',
                LINK: 'LINK',
                ITALIC: 'ITALIC',
            },

            comment_data: {},
            comment_posted_at: {
                total_milliseconds: 0,
                minutes: 0,
                hours: 0,
                days: 0,
                date: 0,
            },
            comment_text_data: [],
        }
    },

    props: {
        comment_id: Number,
        indent: Number,
    },

    methods: {
        print_comment_data: function() {
            console.log({data: this.comment_data, text_data: this.comment_text_data});
        },

        insert_to_string: function(string, insert_string, at) {
            return string.substring(0, at) + insert_string + string.substring(at);
        },

        remove_from_string: function(string, from, to) {
            return string.substring(0, from) + string.substring(to);
        },

        insert_separators: function(string, separator_string, separate_tag, add_separator_after) {
            var finding_at = 0;
            while (finding_at != -1) {
                finding_at = string.indexOf(separate_tag, finding_at);
                if (finding_at != -1) {
                    finding_at += separate_tag.length;
                    if (add_separator_after) {
                        string = this.insert_to_string(string, separator_string, finding_at);
                    }
                    else {
                        string = this.insert_to_string(string, separator_string, finding_at - separate_tag.length);
                        finding_at += separator_string.length;
                    }
                }
            }
            return string;
        },

        get_attribute: function(string, attribute) {
            var attribute_start = string.indexOf(attribute + '="') + attribute.length + 2;
            var attribute_end = string.indexOf('"', attribute_start);
            return string.substring(attribute_start, attribute_end);
        },

        delete_attribute: function(string, attribute) {
            var attribute_start = string.indexOf(attribute + '="') - 1;
            var attribute_end = string.indexOf('"', attribute_start + attribute.length + 3) + 1;
            return this.remove_from_string(string, attribute_start, attribute_end);
        },

        delete_tag: function(string, tag_name) {
            var result = string.replace('<' + tag_name + '>', '');
            result = result.replace('</' + tag_name + '>', '');
            return result;
        },

        convert_text: function(text) {
            var converted_text = text;

            var text_area = document.createElement('textarea');
            text_area.innerHTML = converted_text;
            converted_text = text_area.value;

            var comment_paragraph_list = converted_text.split('<p>');
            var result = [];

            for (var i = 0; i < comment_paragraph_list.length; i++) {
                var paragraph = comment_paragraph_list[i];

                var separator_string = '<separate>';
                paragraph = this.insert_separators(paragraph, separator_string, '<a', false);
                paragraph = this.insert_separators(paragraph, separator_string, '</a>', true);

                var splitted_paragraph = paragraph.split(separator_string);
                for (var j = 0; j < splitted_paragraph.length; j++) {
                    if (splitted_paragraph[j] == '') {
                        splitted_paragraph.splice(j, 1);
                    } 
                }

                var splitted_paragraph_result = [];
                for (var j = 0; j < splitted_paragraph.length; j++) {
                    var splitted = splitted_paragraph[j];
                    var splitted_text_type = -1;
                    var splitted_text = '';
                    var splitted_href = '';
                    var splitted_rel = '';
                    if (splitted.indexOf('<a') != -1) {
                        splitted_text_type = this.text_type.LINK;

                        splitted_href = this.get_attribute(splitted, 'href');
                        splitted_rel = this.get_attribute(splitted, 'rel');

                        splitted_text = splitted;
                        splitted_text = this.delete_attribute(splitted_text, 'href');
                        splitted_text = this.delete_attribute(splitted_text, 'rel');
                        splitted_text = this.delete_tag(splitted_text, 'a');
                    }
                    else if (splitted.indexOf('<i') != -1) {
                        splitted_text_type = this.text_type.ITALIC;

                        splitted_text = splitted;
                        splitted_text = this.delete_tag(splitted_text, 'i');
                    }
                    else {
                        splitted_text_type = this.text_type.NORMAL;

                        splitted_text = splitted;
                    }

                    splitted_paragraph_result.push({
                        text_type: splitted_text_type,
                        text: splitted_text,
                        href: splitted_href,
                        rel: splitted_rel,
                    });
                }

                result.push(splitted_paragraph_result);
            }

            return result;
        }
    },

    created: function() {
        fetch('https://hacker-news.firebaseio.com/v0/item/' + this.comment_id + '.json')
        .then(res => res.json())
        .then(out => {
            this.comment_data = out;

            this.comment_posted_at.date = new Date(this.comment_data.time);
            this.comment_posted_at.total_milliseconds = Date.now() / 1000 - this.comment_data.time;
            this.comment_posted_at.minutes = Math.round(this.comment_posted_at.total_milliseconds / 60 % 60);
            this.comment_posted_at.hours = Math.round(this.comment_posted_at.total_milliseconds / 60 / 60 % 24);
            this.comment_posted_at.days = Math.round(this.comment_posted_at.total_milliseconds / 60 / 60 / 24);

            this.comment_text_data = this.convert_text(this.comment_data.text);
        })
        .catch(error => { throw error });
    },
}
</script>

<template>
    <div class="two_columns_comment" @click="print_comment_data()">
        <p class="two_columns_comment_paragraph" v-for="paragraph in comment_text_data">
            <template v-for="text in paragraph">
                <template v-if="text.text_type == text_type.NORMAL">{{ text.text }}</template>
                <a v-else-if="text.text_type == text_type.LINK" :href="text.href" :rel="text.rel">{{ text.text }}</a>
                <i v-else-if="text.text_type == text_type.ITALIC">{{ text.text }}</i>
            </template>
        </p>
    </div>
</template>