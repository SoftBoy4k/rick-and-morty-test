<template>
    <div>
        <div class="filter">
          <input v-model="filterName" type="text" placeholder="Фильтр по имени">
          <select v-model="filterStatus">
            <option value="">Фильтр по статусу</option>
            <option value="Alive">Жив</option>
            <option value="Dead">Мёртв</option>
            <option value="unknown">Неизвестно</option>
          </select>
          <button @click="applyFiltersAndResetPage">Применить</button>
        </div>
        <div v-if="isLoading">Загрузка...</div>
        <div v-else-if="!characters || (characters && characters.length === 0)">Нет данных</div>
        <div class="wrapper" v-else>
          <div class="card__wrapper" v-for="character in characters" :key="character.id">
            <div class="card">
                <div class="card__image"><img :src="character.image" alt="Character Image"></div>
                <div class="card__text">
                    <div class="section">
                        <p class="title" href="">{{ character.name }}</p>
                        <span class="status">
                            <span class="status__icon" :class="character.status === 'Alive' ? 'green' : character.status === 'Dead' ? 'red' : 'grey'"></span>
                            {{ character.status }} - {{ character.species }}
                        </span>
                    </div>
                    <div class="section">
                        <span class="text-grey">Last known location:</span>
                        <p>{{ character.location.name }}</p>
                    </div>
                    <div class="section">
                        <span class="text-grey">First seen in:</span>
                        <p>{{ 
                          getEpisode(character.episode[0])
                        }}</p>
                    </div>
                </div>
            </div>
          </div>
        </div>
        <div>
          <button @click="prevPage" :disabled="page <= 1 || isLoading">Назад</button>
          <span>{{ page }}</span>
          <button @click="nextPage" :disabled="page >= totalPages || isLoading">Вперед</button>
        </div>
    </div>
  </template>

<script>
    import { ref, onMounted } from 'vue';
    
    export default {
        name: 'RickAndMorty',
        setup() {
        const characters = ref([]);
        const page = ref(1);
        const totalPages = ref(1);
        const filterName = ref('');
        const filterStatus = ref('');
        const isLoading = ref(false);
        const allEpisode = ref(['']);
    
        const fetchCharacters = async () => {
            try {
            isLoading.value = true;
            let response;
            if (filterName.value || filterStatus.value) {
                response = await fetch(`https://rickandmortyapi.com/api/character/?name=${filterName.value}&status=${filterStatus.value}&page=${page.value}`);
            } else {
                response = await fetch(`https://rickandmortyapi.com/api/character/?page=${page.value}`);
            }
            const data = await response.json();
            characters.value = data.results;
            totalPages.value = data.info.pages;
    
            } catch (error) {
            console.error('Ошибка при загрузке данных:', error);
            } finally {
            isLoading.value = false;
            }
        };
    
        const fetchEpisode = async () => {
            try {
                isLoading.value = true;
                for(let i = 1; i <= 3; i++) {
                    const response = await fetch(`https://rickandmortyapi.com/api/episode/?page=${i}`);
                    const data = await response.json();
                    allEpisode.value.push(...data.results);
                }
            } catch (error) {
                console.error('Ошибка при загрузке данных:', error);
            } finally {
                isLoading.value = false;
            }
        }
    
        const getEpisode = (episodeURL) => {
            const currentEpisode = allEpisode.value.filter(el => el.url === episodeURL)[0]
            return currentEpisode ? currentEpisode.name : 'unknown';
        } 
    
        const nextPage = () => {
            if (page.value < totalPages.value && !isLoading.value) {
            page.value++;
            fetchCharacters();
            }
        };
    
        const prevPage = () => {
            if (page.value > 1 && !isLoading.value) {
            page.value--;
            fetchCharacters();
            }
        };
    
        const resetFilters = () => {
            filterName.value = '';
            filterStatus.value = '';
        };
    
        const applyFiltersAndResetPage = () => {
            resetPage();
            fetchCharacters();
        };
    
        const resetPage = () => {
            page.value = 1;
        };
    
        onMounted(() => {
            fetchEpisode();
            fetchCharacters();
        });
    
        return {
            characters,
            page,
            totalPages,
            filterName,
            filterStatus,
            nextPage,
            prevPage,
            applyFiltersAndResetPage,
            resetFilters,
            isLoading,
            getEpisode
        };
        }
    };
    </script>
  
<style>
* {
    box-sizing: inherit;
    margin: 0;
    padding: 0;
}

.filter {
    margin: 15px;
}

.wrapper {
    display: flex;
    justify-content: space-between;
    flex-wrap: wrap;
    padding: 2% 5%
}

.card__wrapper {
    width: 600px;
    margin-bottom: 2%;
}

.card {
    width: 600px;
    height: 220px;
    display: flex;
    overflow: hidden;
    background: rgb(60, 62, 68);
    border-radius: 0.5rem;
    box-shadow: rgba(0, 0, 0, 0.1) 0px 4px 6px -1px, rgba(0, 0, 0, 0.06) 0px 2px 4px -1px;
}

.card__image {
    flex: 2 1 0%;
    width: 100%;
}

.card__image > img {
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
    font-size: 16px;
}

.section {
    flex: 1 1 0%;
    display: flex;
    flex-direction: column;
    justify-content: space-around;
}

.title {
    font-size: 25px;
}

.status {
    display: flex;
    align-items: center;
    text-transform: capitalize;
    font-size: 17px;
}

.status__icon {
    height: 0.5rem;
    width: 0.5rem;
    margin-right: 0.375rem;
    border-radius: 50%;
}

.text-grey {
    color: rgb(158, 158, 158);
}

.green {
  background-color: green;
}

.red {
  background-color: red;
}

.grey {
  background-color: grey;
}

</style>
  