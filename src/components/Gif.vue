<template>
    <v-flex xs12 md4 class="center" mb-2>
        <video :src="gif.images.looping.mp4" type="video/mp4" autoplay loop @click="copyLink(gif.bitly_url)"></video>
    </v-flex>
</template>

<script>
export default {
    props: ['gif'],

    methods: {
        copyLink: function(link){
            const textarea = document.createElement("textarea");
            textarea.value = link;
            textarea.setAttribute("readonly", "");
            textarea.style.cssText = "position:fixed;pointer-events:none;z-index:-9999;opacity:0;";

            document.body.appendChild(textarea);

            textarea.select();

            let success = false;

            try {
                success = document.execCommand('copy');
            } catch (err) {
                //
            }

            document.body.removeChild(textarea);

            this.$emit('copied-to-clipboard');

            return success;
        }
    }
}
</script>

<style scoped>
    video {
        max-width: 100%;
        cursor: pointer;
    }

    .hidden{
        display: none;
    }

    .center{
        display: flex;
        flex-direction: column;
        align-items: center;
    }
</style>