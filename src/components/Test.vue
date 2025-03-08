<template>
 <h2>{{ testFunction }}</h2>
</template>

<script>
 export default {
    data () {
        return {
            reactiveVariable: '',
        }
    },
    computed: {
        testFunction() {
         return 'Hello world!'
        },
    }
 }
</script>

<style>

</style>