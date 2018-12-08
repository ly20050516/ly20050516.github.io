<template>
    <div id="app">
        <background style="position: absolute;top: 0;z-index: -1" class="background"></background>
        <router-view/>
    </div>
</template>

<script>
    import { mapGetters } from 'vuex'
    import Background from './views/layout/components/canvas-background.vue'

    export default {
        name: 'App',
      components: {
        Background
      },
        computed: {
            ...mapGetters([
                'githubUsername',
                'htmlTitle'
            ])
        },
        created() {
            this.$store.dispatch("Init")
            this.$store.dispatch("GetInfo")
            this.$setTitle(this.$route.meta.title)
            let windowSize = this.$util.getWindowSize()
            let pathArr = this.$route.path.split("/")
            console.log(pathArr);
            if (pathArr[1] == "user" && windowSize.height > windowSize.width * 1.2) {
                this.$router.push("/mobile/user/blog")
            }
            if (pathArr[1] == "mobile" && windowSize.height <= windowSize.width * 1.2) {
                this.$router.push("/")
            }
        }
    }
</script>
