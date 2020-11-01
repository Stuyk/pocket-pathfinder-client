<template>
    <div class="home">
        <img alt="Vue logo" src="../assets/logo.svg" width="400" />
        <Connection />
    </div>
</template>

<script>
// @ is an alias to /src
import Connection from '@/components/Connection.vue';

export default {
    name: 'Landing',
    components: {
        Connection
    }
};
</script>
