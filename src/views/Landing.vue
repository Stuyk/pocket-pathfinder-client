<template>
    <div class="home">
        <!-- <Connection /> -->
        <p>{{ session }}</p>
    </div>
</template>

<script>
    // @ is an alias to /src
    import Connection from '@/components/Connection.vue';

    export default {
        name: 'Landing',
        props: ['session'],
        components: {
            Connection
        }
    };
</script>
