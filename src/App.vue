<template>
    <div id="app">
        <div id="presort-form" v-if="!sorting">
        <ul id="presets">
            <li v-on:click="usePreset()">Beatles songs</li>
        </ul>
        <textarea v-model="itemList"></textarea>
        <br>
        <input type="radio" id="random" value="random" v-model="sortMethod">
        <label for="random">Random</label>
        <br>
        <input type="radio" id="quick" value="quick" v-model="sortMethod">
        <label for="quick">Quick</label>
        <br>
        <button v-on:click="goToSort()">Sort</button>
        </div>
        <SortForm v-if="sorting"
              v-bind:items="itemList.split('\n')"
        />
    </div>
</template>

<script>
import Beatles from './assets/presets/beatles2.json';
import SortForm from './components/SortForm.vue';

export default {
    name: 'app',
    components: {
        SortForm,
    },
    data() {
        return {
            Beatles,
            itemList: '',
            sortMethod: '',
            sorting: false,
        }
    },
    methods: {
        usePreset: function() {
            this.itemList = this.Beatles.data;
        },
        goToSort: function() {
            this.sorting = true;
        },
    },
}
</script>

<style>
</style>
