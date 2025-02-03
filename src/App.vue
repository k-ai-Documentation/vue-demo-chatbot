<template lang="pug">
.content
    .logo 
        img(:src="kaiLogo")
    p.text-bold-20.text-white chatbot
    Chat
</template>

<script setup lang="ts">
import Chat from './components/Chat.vue';
import 'kai-assets/styles/main.scss';
import kaiLogo from "kai-assets/logo.svg"
</script>

<style lang="scss" scoped>
.content {
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 100%;
    height: 100%;

    .logo {
        width: 133px;
        height: fit-content;
        margin-top: 100px;
        margin-bottom: 20px;
        filter: var(--svg-filter-white-color);
    }
}
</style>
