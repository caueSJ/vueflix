<script setup>
import { computed, onMounted, ref, watch } from 'vue';

const showAddMovie = ref(false);
const hasError = ref(false);

// Add new movie inputs
const movieName = ref('');
const moviePosterUrl = ref('');
const movieRelease = ref('');
const movieGenre = ref('');

// Initial movie list
const movies = ref([]);

const save = () => {
  if (!validate()) {
    hasError.value = true;
    return;
  }
  
  movies.value.push({
    name: movieName.value,
    release: movieRelease.value,
    genre: movieGenre.value,
    posterUrl: moviePosterUrl.value,
  });

  clear();
}

const clear = () => {
  movieName.value = '';
  moviePosterUrl.value = '';
  movieRelease.value = '';
  movieGenre.value = '';

  showAddMovie.value = hasError.value = false;
}

const validate = () => {
  return movieName.value && movieRelease.value && movieGenre.value && moviePosterUrl.value;
}

const like = (index, status) => {
  if (movies.value[index].hasOwnProperty('like') &&
      movies.value[index].like === status) {
    delete movies.value[index].like;
  } else {
    movies.value[index].like = status;
  }
}

const removeMovie = (index) => {
  const alertMessage = `Você tem certeza que deseja excluir o filme ${movies.value[index].name}?`;
  if (confirm(alertMessage)) {
    movies.value.splice(index, 1);
  }
};

const selectedFilter = ref('all');
const moviesToShow = computed(() => {
  switch (selectedFilter.value) {
    case 'like':
      return movies.value.filter((item) => item.like == true);
    case 'dislike':
      return movies.value.filter((item) => item.like == false);
    default: // Filter "All/Todos"
      return movies.value;
  }
});

watch(movies, (newInfo, oldInfo) => {
  localStorage.setItem('vue-flix', JSON.stringify(newInfo));
}, {deep: true} );

onMounted(() => {
  const dataLocalStorage = localStorage.getItem('vue-flix');
  console.log(dataLocalStorage);
  if (dataLocalStorage) {
    movies.value = JSON.parse(dataLocalStorage);
  }
});

</script>
<template>
  <div class="vueflix">
    <div class="acoes-usuario">
      <div class="filtros">
        <div class="titulo">Filtrar</div>
        <div class="opcoes-filtros">
          <button
            class="botao"
            @click="selectedFilter = 'all'"
            :class="{ ativo: selectedFilter == 'all' }">Todos</button>
          
          <button
            class="botao"
            @click="selectedFilter = 'like'"
            :class="{ ativo: selectedFilter == 'like' }">Gostei</button>
          
          <button
            class="botao"
            @click="selectedFilter = 'dislike'"
            :class="{ ativo: selectedFilter == 'dislike' }">Não Gostei</button>
        </div>
      </div>

      <!-- ADD NEW MOVIE -->
      <div class="novo-filme">
        <div class="adicionar-filme" v-show="showAddMovie">
          <span v-if="hasError" class="error-message">Preencha todos os campos</span>
          <input v-model.trim="movieName" type="text" autocomplete="off" placeholder="Nome do Filme" />
          <input v-model.trim="moviePosterUrl" type="text" autocomplete="off" placeholder="URL do Poster" />
          <input v-model.trim="movieRelease" type="text" autocomplete="off" placeholder="Data de Lançamento" />
          <input v-model.trim="movieGenre" type="text" autocomplete="off" placeholder="Gênero" />
          <div class="acoes">
            <button class="botao ativo" @click="save">Salvar</button>
            <button class="botao danger ativo" @click="clear();">Cancelar</button>
          </div>
        </div>
        <button class="botao ativo" @click="showAddMovie = true" v-show="!showAddMovie">
          Adicionar Filme
        </button>
      </div>
    </div>

    <!-- MOVIES LIST -->
    <div class="filmes">
      <div class="filme" v-for="(movie, index) in moviesToShow">
        <div class="capa-container">
          <div class="acoes-filme">
            <button 
              class="botao"
              @click="like(index, true)"
              :class="{ativo: movie.like === true}">Gostei</button>

            <button 
              class="botao danger"
              @click="like(index, false)"
              :class="{ativo: movie.like === false}">Não Gostei</button>

            <button
              class="botao danger"
              @click="removeMovie(index)">Excluir</button>
          </div>
          <img
            class="capa"
            :src="movie.posterUrl"
            :alt="movie.name + ' poster movie'"
          />
        </div>
        <div class="nome">{{ movie.name }}</div>
        <div class="info">{{ movie.release }} - {{ movie.genre }}</div>
      </div>
      <p v-if="moviesToShow.length == 0" :style="{flex:1, textAlign:'center', marginTop:'16px'}">Não há filmes para exibir.</p>
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

    .error-message {
      align-self: center;
      color: var(--vt-c-red);
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
