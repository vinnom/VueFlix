<script setup>
import { onBeforeUnmount, onMounted, ref } from "vue";

const KEY_VUEFLIX = "vueflix";
const RATE = {
    none: 0,
    like: 1,
    dislike: 2,
};

const isAddingMovie = ref(false);
const switchAddMovieForm = () => {
    isAddingMovie.value = !isAddingMovie.value;
    return isAddingMovie.value;
};

const finishForm = () => {
    clearMovieField();
    switchAddMovieForm();
};

const movieField = ref({
    name: "",
    url: "",
    releaseDate: "",
    genre: "",
    rate: RATE.none,
});

const movies = ref([]);

const addMovie = () => {
    movies.value.push({
        name: movieField.value.name,
        url: movieField.value.url,
        releaseDate: movieField.value.releaseDate,
        genre: movieField.value.genre,
        rate: RATE.none,
    });

    finishForm();
};

const removeMovie = (movieIndex) => {
    const warning = `Você tem certeza que gostaria de remover "${movies.value[movieIndex].name}" de sua VueFlix?`;
    if (confirm(warning)) {
        movies.value.splice(movieIndex, 1);
    }
};

const rateMovie = (movieIndex, rate) => {
    movies.value[movieIndex].rate = rate;
};

const markRated = (movieIndex, rate) => {
    return {
        ativo: movies.value[movieIndex].rate === rate,
    };
};

const filter = ref(RATE.none);
const filterMovieRate = (rate) => {
    filter.value = rate;
};

const markActive = (selected) => {
    return {
        ativo: filter.value === selected,
    };
};

const clearMovieField = () => {
    movieField.value.name = "";
    movieField.value.url = "";
    movieField.value.releaseDate = "";
    movieField.value.genre = "";
};

onMounted(() => {
    movies.value = JSON.parse(localStorage.getItem(KEY_VUEFLIX)) || [];
});

onBeforeUnmount(() => {
    localStorage.setItem(KEY_VUEFLIX, JSON.stringify(movies.value));
});
</script>

<template>
    <div class="vueflix">
        <div class="acoes-usuario">
            <div class="filtros">
                <div class="titulo">Filtrar</div>
                <div class="opcoes-filtros">
                    <button @click="filterMovieRate(RATE.none)" :class="markActive(RATE.none)" class="botao">
                        Todos
                    </button>
                    <button @click="filterMovieRate(RATE.like)" :class="markActive(RATE.like)" class="botao">
                        Gostei
                    </button>
                    <button @click="filterMovieRate(RATE.dislike)" :class="markActive(RATE.dislike)" class="botao">
                        Não Gostei
                    </button>
                </div>
            </div>

            <div class="novo-filme">
                <div v-if="isAddingMovie" class="adicionar-filme">
                    <input v-model="movieField.name" type="text" autocomplete="off" placeholder="Nome do Filme" />
                    <input v-model="movieField.url" type="text" autocomplete="off" placeholder="URL da Imagem" />
                    <input v-model="movieField.releaseDate" type="text" autocomplete="off" placeholder="Lançamento" />
                    <input v-model="movieField.genre" type="text" autocomplete="off" placeholder="Gênero" />
                    <div class="acoes">
                        <button @click="addMovie" class="botao ativo">Salvar</button>
                        <button @click="finishForm" class="botao danger ativo">Cancelar</button>
                    </div>
                </div>
                <button v-else @click="switchAddMovieForm" class="botao ativo">Adicionar Filme</button>
            </div>
        </div>

        <div class="filmes">
            <div v-for="(movie, index) in movies" class="filme">
                <div v-if="movie.rate === filter || filter === RATE.none">
                    <div class="capa-container">
                        <div class="acoes-filme">
                            <button
                                @click="rateMovie(index, RATE.like)"
                                :class="markRated(index, RATE.like)"
                                class="botao">
                                Gostei
                            </button>
                            <button
                                @click="rateMovie(index, RATE.dislike)"
                                :class="markRated(index, RATE.dislike)"
                                class="botao danger">
                                Não Gostei
                            </button>
                            <button @click="removeMovie(index)" class="botao danger">Excluir</button>
                        </div>
                        <img class="capa" :src="movie.url" alt="" />
                    </div>
                    <div class="nome">{{ movie.name }}</div>
                    <div class="info">{{ movie.releaseDate }} - {{ movie.genre }}</div>
                </div>
            </div>
        </div>
    </div>
</template>

<style lang="scss" scoped>
.vueflix {
    padding: 16px;

    .acoes-usuario {
        display: flex;
        justify-content: space-between;
        align-items: end;
        margin: 30px 0;

        .adicionar-filme {
            display: flex;
        }
    }

    .filmes {
        display: flex;
        gap: 20px;
        flex-wrap: wrap;
        min-height: 350px;

        .filme {
            .capa-container {
                width: 200px;
                height: auto;
                position: relative;

                .capa {
                    width: 100%;
                    height: 100%;
                }

                .acoes-filme {
                    position: absolute;
                    bottom: 0;
                    padding-bottom: 12px;
                    background-image: linear-gradient(to bottom, rgba(255, 0, 0, 0), rgb(0, 0, 0));
                    display: none;
                    flex-direction: column;
                    width: 100%;
                    height: 100%;
                    justify-content: flex-end;
                }
            }

            .nome {
                font-weight: bold;
            }

            .info {
                font-size: 12px;
            }

            &:hover {
                .acoes-filme {
                    display: flex;
                }
            }
        }
    }
}
</style>
