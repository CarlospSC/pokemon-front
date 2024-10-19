<template>
  <div class="contenedor-completo" v-if="filteredTeam1Pokemons.length > 0 && filteredTeam2Pokemons.length > 0 && this.tiempoCarga==3 && !this.cargarCombateIndividual">
    <h1>Simulador de batalla Pokemon</h1>
    <div class="bloque-players">
      <div class="animated__animate">
        <h1>Equipo 1</h1>
        <div class="bloque-player animate__backInRight" v-for="pokemon in filteredTeam1Pokemons" :key="pokemon.id">
          <h2>{{ toUpperCase(pokemon.nombre) }}</h2>
          <img :src="pokemon.sprite" alt="Imagen de Pokémon" height="200"/>
        </div>
      </div>
      <div @click="iniciarCombate" class="iniciar animate__animated animate__heartBeat">
        <img :src="require('@/assets/img/pokeball-start.png')" alt="Pokeball iniciar batalla" height="280" />
      </div>
      <div class="animated__animate">
        <h1>Equipo 2</h1>
        <div class="bloque-player animate__backInLeft" v-for="pokemon in filteredTeam2Pokemons" :key="pokemon.id">
          <h2>{{ toUpperCase(pokemon.nombre) }}</h2>
          <img :src="pokemon.sprite" alt="Imagen de Pokemon" height="200"/>
        </div>
      </div>
    </div>
  </div>
  <div v-if="this.cargarInicio" class="pantalla-carga">
    <h1 class="titulo-carga">Cargando Pokemon aleatorios...</h1>
  </div>
  <div v-if="error">
    <p>{{ error }}</p>
  </div>

  <div v-if="this.cargarCombateIndividual">
    <IndividualBattle @enviar="recibirDebilitadoData" :detallesC="detalles"/>
  </div>

  <div v-if="this.existeEquipoGanador" class="pantalla-winners">
    <div v-if="this.winnerTeam === 'Equipo 1'" class="contenedor-winners1">
      <!--
      <img :src="require('@/assets/img/winner-equipo-1.png')" alt="Imagen Equipo 1 ganador" height="935" class="animate__animated animate__heartBeat"/>
      -->

      <div class="ganadores">
        <div class="fila1-resultados">
          <img :src="require('@/assets/img/winner-equipo-1.png')" alt="Imagen Equipo 1 ganador" height="600" class="animate__animated animate__heartBeat"/>
          <button class="boton-reinicio" @click="reloadPage">Nuevo combate</button>
        </div>
        <div class="fila2-resultados">
          <div class="bloque-winners-contenedor">
            <div class="bloque-winners" v-for="pokemon in this.team1Inicial" :key="pokemon.id">
              <h2>{{ toUpperCase(pokemon.nombre) }}</h2>
              <img :src="pokemon.sprite" alt="Imagen de Pokémon" height="200"/>
            </div>
          </div>
          <img :src="require('@/assets/img/pikachu-firma2.png')" alt="Firma y pikachu" height="330" class="pikachu-firma"/>
        </div>
      </div>
    </div>
    <div v-else class="contenedor-winners2">
      <div class="ganadores">
        <div class="fila1-resultados">
          <img :src="require('@/assets/img/winner-equipo-2.png')" alt="Imagen Equipo 2 ganador" height="600" class="animate__animated animate__heartBeat"/>
          <button class="boton-reinicio" @click="reloadPage">Nuevo combate</button>
        </div>
        <div class="fila2-resultados">
          <div class="bloque-winners-contenedor">
            <div class="bloque-winners" v-for="pokemon in this.team2Inicial" :key="pokemon.id">
              <h2>{{ toUpperCase(pokemon.nombre) }}</h2>
              <img :src="pokemon.sprite" alt="Imagen de Pokémon" height="200"/>
            </div>
          </div>
          <img :src="require('@/assets/img/pikachu-firma2.png')" alt="Firma y pikachu" height="330" class="pikachu-firma"/>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import IndividualBattle from './IndividualBattle.vue';

export default {
  data() {
    return {
      detalles: {},
      cargarInicio: true,
      cargarCombateIndividual: false,
      tiempoCarga: 0,
      team1Inicial: [],
      team2Inicial: [],
      team1Pokemons: [],
      team2Pokemons: [],
      t1_cantidadVivos: 3,
      t2_cantidadVivos: 3,
      error: null,
      pk_debilitadoBattle: {},
      existeEquipoGanador: false,
      winnerTeam: ""
    };
  },
  components: {
    IndividualBattle
  },
  computed: {
    filteredTeam1Pokemons() {
      return this.team1Pokemons.filter(pokemon => pokemon);
    },
    filteredTeam2Pokemons() {
      return this.team2Pokemons.filter(pokemon => pokemon);
    }
  },
  methods: {

    reloadPage() {
      window.location.reload();
    },

    recibirDebilitadoData(data) {
      this.pk_debilitadoBattle = data;
      console.log("SE HA RECIBIDO LA DATA DEL POKEMON DEBILITADO EN EL COMPONENTE PADRE, ESTE ES: " , this.pk_debilitadoBattle);

      this.team1Pokemons = this.team1Pokemons.filter(obj => obj.id !== this.pk_debilitadoBattle.id);
      this.team2Pokemons = this.team2Pokemons.filter(obj => obj.id !== this.pk_debilitadoBattle.id);
      

      this.cargarCombateIndividual = false;

      if (this.team1Pokemons.length === 0) {
        this.winnerTeam = "Equipo 2"
        this.existeEquipoGanador = true;
      }

      if (this.team2Pokemons.length === 0) {
        this.winnerTeam = "Equipo 1"
        this.existeEquipoGanador = true;
      }
    },


    async fetchPokemons() {
      try {
        const response = await axios.get(`${process.env.VUE_APP_API_URL}`);
        console.log('Respuesta de la API:', response.data);
        this.team1Pokemons = response.data.player1_pokemons || [];
        this.team2Pokemons = response.data.player2_pokemons || [];
        this.team1Inicial = this.team1Pokemons;
        this.team2Inicial = this.team2Pokemons;
        console.log('Pokémons Jugador 1:', this.team1Pokemons);
        console.log('Pokémons Jugador 2:', this.team2Pokemons);
      } catch (error) {
        this.error = 'Error al obtener la data de los Pokemon';
        console.error('Error al obtener los Pokémon:', error);
      }
    },

    iniciarCombate() {

      this.cargarCombateIndividual = true;

      let datosCombate = {
        t1_pksVivos: [],
        t1_pksDebilitados: [],
        t1_pkActivo: {},
        t2_pksVivos: [],
        t2_pksDebilitados: [],
        t2_pkActivo: {}
      }

      this.team1Pokemons.forEach((pokemon, index)=> {
        if (index == 0) {
          datosCombate.t1_pkActivo = pokemon
        }
        datosCombate.t1_pksVivos.push(pokemon);
      })

      this.team2Pokemons.forEach((pokemon, index)=> {
        if (index ==0) {
          datosCombate.t2_pkActivo = pokemon
        }
        datosCombate.t2_pksVivos.push(pokemon);
      })

      this.detalles = datosCombate;

      console.log("El primer Pokemon activo de cada equipo será: " , datosCombate.t1_pkActivo, " y ", datosCombate.t2_pkActivo)

      while (this.t1_cantidadVivos > 1 && this.t2_cantidadVivos > 1) {
        console.log("Los pokemones vivos de cada uno son: ", datosCombate.t1_pksVivos, " y ", datosCombate.t2_pksVivos);
        this.t1_cantidadVivos -=1;
      }
    },

    iniciarContadorSegundos() {
      setTimeout(() => {
        this.tiempoCarga = 3;
        this.cargarInicio= false;
        console.log("Ahora la variable tendrá el valor de 3: ", this.tiempoCarga);
      }, 2000);
      
    },

    toUpperCase(text) {
      return text.toUpperCase();
    }
  },
  mounted() {
    
    this.fetchPokemons();
    this.iniciarContadorSegundos();
  }
};
</script>

<style scoped>
  h1 {
    padding: 1.2em;
  }

  h2 {
    margin: 0 1em;

  }
  
  .pantalla-carga {
    height: 100vh;
    width: 100vw;
    background-image: url('../assets/img/inicio-nuevo.png');
    background-size: 90% auto;
    background-position: center 10%;
    background-repeat: no-repeat;
    box-sizing: border-box;
    padding: 1em;
  }
  
  .titulo-carga {
    margin-right: 10em;
  }
  
  .contenedor-completo {
    display: flex;
    flex-direction: column;
    justify-content: center;
    height: 100vh;
    width: 100vw;
    padding: 4em 12em;
    padding: 0;
    background-image: url('../assets/img/fondo-celeste2.png');
    background-size: cover;
    background-position: center; 
    background-repeat: no-repeat;
  }
  


  .bloque-players, .bloque-player {
    display: flex;
  }

  .bloque-players {
    justify-content: space-evenly;
    align-items: center;
  }

  .bloque-player {
    align-items: center;
    animation-duration: 2s;
    padding: 1em 0;
  }

  .bloque-versus {
    margin: 0 2em;
  }

  .iniciar {
    padding: .5em;
    width: 20em;
    margin: 1em;
    text-align: center;
    transition: transform 0.8s ease;
  }

  .iniciar:hover {
    cursor: pointer;
    transform: scale(0.98);
  }

  .iniciar:active {
    transform: scale(0.96);
    opacity: 0.9;
  }

  .pantalla-winners {
    height: 99.84vh;
    width: 99.5vw;
    background-image: url('../assets/img/fondo-celeste2.png');
    background-size: cover;
    background-position: center; 
    background-repeat: no-repeat;
  }

  .ganadores {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }

  .fila1-resultados {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 7.5em;
  }

  .fila2-resultados {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 2.5em;
    .bloque-winners-contenedor {
      display: flex;
      margin-left: 25em;
    }
  }

  .boton-reinicio {
    padding: 0.8em;
    background: linear-gradient(to right, orange, red);
    border-radius: 10px;
    font-size: 1.5em;
    margin-left: 4.5em;
  }

  .boton-reinicio:hover {
    cursor: pointer;
    background: linear-gradient(to left, orange, red);
  }

  .pikachu-firma {
    margin-left: 25em;
  }

  /*
  .contenedor-winners1 {
    background-image: url('../assets/img/winner-equipo-1.png');
  }

  .contenedor-winners2 {
    background-image: url('../assets/img/winner-equipo-2.png');
  }

  .contenedor-winners1, .contenedor-winners2 {
    height: 100vh;
    width: 100vw;
    padding: 4em 12em;
    padding: 0;
    background-size: cover;
    background-position: center; 
    background-repeat: no-repeat;

    .bloque-winners {
      display: flex;
    }
  }
  */



</style>
