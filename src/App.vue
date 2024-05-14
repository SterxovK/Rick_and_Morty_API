<template>
    <h2 style="text-align: center; color: #D6D6E7; margin: 0 0 2rem 0">Фильтр</h2>
    <div class="filter__form">
        <div class="group">
            <input required="" type="text" class="input" v-model="name">
            <span class="highlight"></span>
            <span class="bar"></span>
            <label>Имя</label>
        </div>
        <div class="group">
            <input required="" type="text" class="input" v-model="status">
            <span class="highlight"></span>
            <span class="bar"></span>
            <label>Статус</label>
        </div>
        <button @click='setFilter(name, status)' role="button" class="button-name">Применить</button>
    </div>

    <div class="card__container">
        <article v-if="characters" v-for="character in characters" :key="character.id" class="card">
            <div class="card__img-box"><img :src="`${character.image}`" alt="" class="card__img"></div>
            <div class="card__text">
                <div class="section">
                    <h2>{{ character.name }}</h2>
                    <span class="status">
                        <span class="status__icon"
                              :class="{ 'status__icon_red': character.status == 'Dead', 'status__icon_gray': character.status == 'unknown' }"></span>
                        {{ character.status }} - {{ character.species }}</span>
                </div>
                <div class="section"><span class="text-gray">Last known location:</span> {{ character.location.name }}
                </div>
                <div class="section"><span
                    class="text-gray">First seen in:</span>{{ getAllFirstEpisodeName(character.id) }}
                </div>
            </div>
        </article>
    </div>
    <div class="pagination">
        <button v-if="page > 1" @click="goPage('back')" role="button" class="button-name">Назад</button>
        <button @click="goPage('next')" role="button" class="button-name">Вперед</button>
    </div>

    <VuePreloader
        background-color="#091a28"
        color="#ffffff"
        transition-type="fade-up"
        :loading-speed="25"
        :transition-speed="1400"
    ></VuePreloader>
</template>
<script setup>
import {ref, onBeforeMount} from 'vue'
import {VuePreloader} from 'vue-preloader';
import '../node_modules/vue-preloader/dist/style.css'

const characters = ref(null)
const episodes = ref(null)
const page = ref(1)
let name = ref(null)
let status = ref(null)

const getAllFirstEpisodeName = (id) => {
    const idFirstEpisode = characters.value.find(item => item.id == id)?.episode[0].match(/\d+/)[0]
    if (episodes.value) {
        const currentEpisode = episodes.value.find(item => item.id == idFirstEpisode)
        return currentEpisode?.name
    }
    return 'не определено'
}

async function setFilter(name, status) {
    const response = ref(null)
    try {
        if (!status) {
            response.value = await fetch(`https://rickandmortyapi.com/api/character/?name=${name}`);
        } else if (!name) {
            response.value = await fetch(`https://rickandmortyapi.com/api/character/?status=${status}`);
        } else {
            response.value = await fetch(`https://rickandmortyapi.com/api/character/?name=${name}&&status=${status}`);
        }
        const res = await response.value.json();
        characters.value = res.results
    } catch (error) {
        console.error(error);
    }
}

async function getCharacters() {
    try {
        const response = await fetch(`https://rickandmortyapi.com/api/character/?page=${page.value}`);
        const res = await response.json();
        characters.value = res.results

    } catch (error) {
        console.error(error);
    }
}

async function getAllEpisode() {
    try {
        const response = await fetch(`https://rickandmortyapi.com/api/episode/${[...Array(52)].map((_, i) => i)}`)
        episodes.value = await response.json();
    } catch (error) {
        console.error(error);
    }
}

const goPage = (slag) => {
    switch (slag) {
        case 'next':
            page.value++
            getCharacters(page);
            break
        case 'back':
            page.value--
            getCharacters(page);
            break
    }

}

onBeforeMount(() => {
    getCharacters(page)
    getAllEpisode()
})


</script>
<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Raleway:ital,wght@0,100..900;1,100..900&family=Roboto:ital,wght@0,100;0,300;0,400;0,500;0,700;0,900;1,100;1,300;1,400;1,500;1,700;1,900&display=swap');

.card__container {
    display: flex;
    -webkit-box-pack: center;
    justify-content: center;
    -webkit-box-align: center;
    align-items: center;
    padding: 2rem 0;
    min-height: calc(-60px + 50vh);
    flex-wrap: wrap;
    max-width: 1920px;
}

.card {
    width: 600px;
    height: 220px;
    display: flex;
    overflow: hidden;
    background: rgb(60, 62, 68);
    border-radius: 0.5rem;
    margin: 0.75rem;
    box-shadow: rgba(0, 0, 0, 0.1) 0px 4px 6px -1px, rgba(0, 0, 0, 0.06) 0px 2px 4px -1px;
}

.card__img-box {
    flex: 2 1 0%;
    width: 100%;
}

.card__img {
    width: 100%;
    height: 100%;
    margin: 0px;
    opacity: 1;
    transition: opacity 0.5s ease 0s;
    object-position: center center;
    object-fit: cover;
}

.card__text {
    flex: 3 1 0%;
    position: relative;
    padding: 0.75rem;
    color: rgb(255, 255, 255);
    display: flex;
    flex-direction: column;
}

.card__text .section {
    flex: 1 1 0;
    display: flex;
    flex-direction: column;
    -webkit-box-pack: center;
    justify-content: center;
}

.card__text h2 {
    font-size: 1.5rem;
    margin: 0;
    padding: 0;
}

.card__text .status {
    display: flex;
    -webkit-box-align: center;
    align-items: center;
    text-transform: capitalize;
}

.card__text .status__icon {
    height: 0.5rem;
    width: 0.5rem;
    margin-right: 0.375rem;
    background: rgb(85, 204, 68);
    border-radius: 50%;
}

.card__text .status__icon_red {
    height: 0.5rem;
    width: 0.5rem;
    margin-right: 0.375rem;
    background: rgb(252, 2, 2);
    border-radius: 50%;
}

.card__text .status__icon_gray {
    height: 0.5rem;
    width: 0.5rem;
    margin-right: 0.375rem;
    background: rgb(158, 158, 158);
    border-radius: 50%;
}

.card__text .text-gray {
    color: rgb(158, 158, 158);
}

.card__text .section:first-child {
    -webkit-box-pack: start;
    justify-content: flex-start;
}

.card__text .section:last-child {
    -webkit-box-pack: end;
    justify-content: flex-end;
}

.pagination {
    display: flex;
    justify-content: space-around;
}

.button-name {
    align-items: center;
    appearance: none;
    background-color: #FCFCFD;
    border-radius: 4px;
    border-width: 0;
    box-shadow: rgba(45, 35, 66, 0.2) 0 2px 4px, rgba(45, 35, 66, 0.15) 0 7px 13px -3px, #D6D6E7 0 -3px 0 inset;
    box-sizing: border-box;
    color: #36395A;
    cursor: pointer;
    display: inline-flex;
    font-family: "JetBrains Mono", monospace;
    height: 48px;
    justify-content: center;
    line-height: 1;
    list-style: none;
    overflow: hidden;
    padding-left: 16px;
    padding-right: 16px;
    position: relative;
    text-align: left;
    text-decoration: none;
    transition: box-shadow .15s, transform .15s;
    user-select: none;
    -webkit-user-select: none;
    touch-action: manipulation;
    white-space: nowrap;
    will-change: box-shadow, transform;
    font-size: 18px;
}

.button-name:focus {
    box-shadow: #D6D6E7 0 0 0 1.5px inset, rgba(45, 35, 66, 0.4) 0 2px 4px, rgba(45, 35, 66, 0.3) 0 7px 13px -3px, #D6D6E7 0 -3px 0 inset;
}

.button-name:hover {
    box-shadow: rgba(45, 35, 66, 0.3) 0 4px 8px, rgba(45, 35, 66, 0.2) 0 7px 13px -3px, #D6D6E7 0 -3px 0 inset;
    transform: translateY(-2px);
}

.button-name:active {
    box-shadow: #D6D6E7 0 3px 7px inset;
    transform: translateY(2px);
}

.group {
    position: relative;
}

.input {
    font-size: 16px;
    padding: 10px 10px 10px 5px;
    display: block;
    width: 200px;
    border: none;
    border-bottom: 1px solid #515151;
    background: transparent;
    color: #D6D6E7;
}

.input:focus {
    outline: none;
}

label {
    color: #999;
    font-size: 18px;
    font-weight: normal;
    position: absolute;
    pointer-events: none;
    left: 5px;
    top: 10px;
    transition: 0.2s ease all;
    -moz-transition: 0.2s ease all;
    -webkit-transition: 0.2s ease all;
}

.input:focus ~ label, .input:valid ~ label {
    top: -20px;
    font-size: 14px;
    color: #5264AE;
}

.bar {
    position: relative;
    display: block;
    width: 200px;
}

.bar:before, .bar:after {
    content: '';
    height: 2px;
    width: 0;
    bottom: 1px;
    position: absolute;
    background: #5264AE;
    transition: 0.2s ease all;
    -moz-transition: 0.2s ease all;
    -webkit-transition: 0.2s ease all;
}

.bar:before {
    left: 50%;
}

.bar:after {
    right: 50%;
}

.input:focus ~ .bar:before, .input:focus ~ .bar:after {
    width: 50%;
}

.highlight {
    position: absolute;
    height: 60%;
    width: 100px;
    top: 25%;
    left: 0;
    pointer-events: none;
    opacity: 0.5;
}

.input:focus ~ .highlight {
    animation: inputHighlighter 0.3s ease;
}

@keyframes inputHighlighter {
    from {
        background: #5264AE;
    }

    to {
        width: 0;
        background: transparent;
    }
}

.filter__form {
    display: flex;
    justify-content: space-around
}


@media (max-width: 40.625em) {
    .card {
        flex-direction: column;
        height: initial;
        width: 100%;
    }

    .card__img {
        height: 300px;
    }

    .card__text {
        pointer-events: none;
    }
    .filter__form {
        flex-direction: column;
        align-items: center;
        height: 13rem;
    }
}
</style>
