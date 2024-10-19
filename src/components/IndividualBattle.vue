<template>
  <div v-if="!pokemonDebilitado">
    <audio ref="audioPlayer" autoplay loop>
      <source src="@/assets/Battle-Pokemon.mp3" type="audio/mpeg">
      Tu navegador no soporta la reproducción de audio.
    </audio>
    <div class="combate-individual-box">
      <div class="pk-team1 animate__animated animate__flash">
        <h2>{{ toUpperCase(this.detallesC.t1_pkActivo.nombre) }}</h2>
        <img :src="this.detallesC.t1_pkActivo.sprite" alt="Imagen de Pokemon" height="300"/>
        <div class="hp">
          <h3>HP = {{ this.pk1_hp }}/{{ this.pk1_hp_total }} pts</h3>
          <div class="hp-border">
            <div class="hp-bar"
                :class="hpBarColor(pk1_hp, pk1_hp_total)"
                :style="{ width: calculateHpBarWidth(pk1_hp, pk1_hp_total) + '%' }">
            </div>
          </div>
          
        </div>
      </div>
      <div class="pk-team2 animate__animated animate__flash">
        <h2>{{ toUpperCase(this.detallesC.t2_pkActivo.nombre) }}</h2>
        <img :src="this.detallesC.t2_pkActivo.sprite" alt="Imagen de Pokemon" height="300"/>
        <div class="hp">
          <h3>HP = {{ this.pk2_hp }}/{{ this.pk2_hp_total }} pts</h3>
          <div class="hp-border">
            <div class="hp-bar"
                  :class="hpBarColor(pk2_hp, pk2_hp_total)"
                  :style="{ width: calculateHpBarWidth(pk2_hp, pk2_hp_total) + '%' }">
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="texto-box">
      <span>Round {{ this.ronda }}!</span>
      <div v-if="!this.permitirCambioAtacante" class="bloque-text">
        <h1>{{toUpperCase(this.nombreAtacante)}} comienza utilizando {{this.nombreAtaque}} y realiza {{ this.dañoRealizado }} de daño ! {{ this.textoCritico }}</h1>
        <button :disabled="isButtonDisabled" @click="permitirCambio" :class="{ 'desaparecer': isInvisible }" class="next">Continuar</button>
      </div>
      <div v-else class="bloque-text">
        <h1>{{toUpperCase(this.nombreAtacante)}} responde utilizando {{this.nombreAtaque}} y realiza {{ this.dañoRealizado }} de daño ! {{ this.textoCritico }}</h1>
        <button :disabled="isButtonDisabled" @click="permitirAvance" :class="{ 'desaparecer': isInvisible }" class="next">Siguiente turno</button>
      </div>
    </div>
  </div>
  <div v-else class="vista-pokemon-debilitado">
    <h1 v-if="this.pk1_hp === 0"> {{ this.detallesC.t1_pkActivo.nombre }} se ha debilitado!</h1>
    <h1 v-else> {{ this.detallesC.t2_pkActivo.nombre }} se ha debilitado!</h1>
  </div>
</template>
  
  <script>
  import axios from 'axios';

  export default {
    props: {
      detallesC: {
        type: Object,
        required: true
      }
    },
    name: 'IndividualBattle',
    data() {
      return {
        ronda: 1,
        nombreAtacante: "",
        nombreDefensor: "",
        nombreAtaque: "",
        claseAtaque: "",
        atacanteActual: 0,
        defensorActual: 0,
        dañoRealizado: 0,
        textoCritico: "",
        pokemonDebilitado: false,
        pk_debilitadoData: {},
        permitirCambioAtacante: false,
        permitirCambioTurno: false,
        pk1_hp_total: 0,
        pk2_hp_total: 0,
        pk1_hp: -1.0,
        pk2_hp: -1.0,
        isButtonDisabled: false,
        isInvisible: false
      };
    },
    
    methods: {

      calculateHpBarWidth(currentHp, totalHp) {
        return Math.max(0, (currentHp / totalHp) * 100);
      },

      // Devuelve la clase CSS según el porcentaje de vida para cambiar el color
      hpBarColor(currentHp, totalHp) {
        const hpPercentage = (currentHp / totalHp) * 100;

        if (hpPercentage <= 25) {
          return 'low-hp';  // Menos de 25% de HP
        } else if (hpPercentage <= 50) {
          return 'mid-hp';  // Entre 25% y 50% de HP
        } else {
          return 'high-hp'; // Más de 50% de HP
        }
      },

      desactivarBoton() {

      },

      enviarAPokemonBattle() {
        this.$emit('enviar', this.pk_debilitadoData);
      },

      toUpperCase(text) {
        return text.toUpperCase();
      },

      avanzarTurno() {
        this.ronda +=1;
        this.nombreAtacante = "";
        this.nombreAtaque = "";
        this.claseAtaque = "";
        this.atacanteActual = 0;
        this.defensorActual = 0;
        this.dañoRealizado = 0;
        this.permitirCambioAtacante = false;
        this.permitirCambioTurno = false;
      },

      permitirCambio() {
        this.isButtonDisabled = true;
        console.log("Botón clickeado");
        this.permitirCambioAtacante = true;
        this.resolveCambioAtacante();
        setTimeout(() => {
          this.isButtonDisabled = false;
        }, 800);
      },

      permitirAvance() {
        this.isButtonDisabled = true;
        console.log("Botón clickeado");
        this.permitirCambioTurno = true;
        this.resolveCambioTurno();
        setTimeout(() => {
          this.isButtonDisabled = false;
        }, 800);
      },

      cambiarAtacante() {
        if (this.atacanteActual === 1) {
          this.atacanteActual = 2;
          this.nombreAtacante = this.detallesC.t2_pkActivo.nombre;
        } else {
          this.atacanteActual = 1;
          this.nombreAtacante = this.detallesC.t1_pkActivo.nombre; 
        }
      },
      
      esperarCambioAtacante() {
        return new Promise(resolve => {
          this.resolveCambioAtacante = resolve;
        });
      },

      esperarCambioTurno() {
        return new Promise(resolve => {
          this.resolveCambioTurno = resolve;
        });
      },

      async obtenerMovimiento(url) {
        try {
          const response = await axios.get(`${process.env.VUE_APP_API_URL}/pokemon/move/`, {
            params: {
              url: url
            }
          });
          console.log("Datos del movimiento:", response.data);
          return response.data;
        } catch (error) {
          console.error("Error al obtener el movimiento:", error);
        }
      },

      encontrarAtacante() {
        const min = 1;
        const max = 2;
        const numero = Math.floor(Math.random() * (max - min + 1)) + min;
        console.log(numero);
        if (numero === 1) {
          this.atacanteActual = 1
          this.nombreAtacante = this.detallesC.t1_pkActivo.nombre;
          this.defensorActual = 2
          this.nombreDefensor = this.detallesC.t2_pkActivo.nombre;
          return "Ataca primero el Pokemon del equipo 1"
        } else {
          this.atacanteActual = 2
          this.nombreAtacante = this.detallesC.t2_pkActivo.nombre;
          this.defensorActual = 1
          this.nombreDefensor = this.detallesC.t1_pkActivo.nombre;
          return "Ataca primero el Pokemon del equipo 2"
        }
      },

      /*
      async seleccionarAtaque() {
        let atacante;

        if (this.atacanteActual === 1) {
          atacante = this.detallesC.t1_pkActivo;
        } else {
          atacante = this.detallesC.t2_pkActivo;
        }

        console.log("ATACANTE: ", atacante);

        const min = 0;
        const max = atacante.ataques.length - 1;
        const iRandom = Math.floor(Math.random() * (max - min + 1)) + min;

        const ataque = atacante.ataques[iRandom];
        console.log("El ataque obtenido aleatoriamente es: ", ataque);

        const detallesAtaque = await this.obtenerMovimiento(ataque.url);
        this.nombreAtaque = await detallesAtaque.nombre;
        this.claseAtaque = await detallesAtaque.clase;
      },
      */

      async seleccionarAtaque() {
        let atacante;

        if (this.atacanteActual === 1) {
            atacante = this.detallesC.t1_pkActivo;
        } else {
            atacante = this.detallesC.t2_pkActivo;
        }

        console.log("ATACANTE: ", atacante);

        // Verificar que el atacante y sus ataques existen
        if (!atacante || !atacante.ataques || atacante.ataques.length === 0) {
            console.error("El atacante no tiene ataques definidos o está indefinido");
            return; // Sal de la función si no hay ataques disponibles
        }

        const min = 0;
        const max = atacante.ataques.length - 1;
        const iRandom = Math.floor(Math.random() * (max - min + 1)) + min;

        const ataque = atacante.ataques[iRandom];
        console.log("El ataque obtenido aleatoriamente es: ", ataque);

        try {
            const detallesAtaque = await this.obtenerMovimiento(ataque.url);

            if (!detallesAtaque || !detallesAtaque.nombre || !detallesAtaque.clase) {
                throw new Error("Detalles del ataque incompletos");
            }
            this.nombreAtaque = detallesAtaque.nombre;
            this.claseAtaque = detallesAtaque.clase;

            console.log("Nombre del ataque: ", this.nombreAtaque);
            console.log("Clase del ataque: ", this.claseAtaque);
        } catch (error) {
            console.error("Error al obtener detalles del ataque: ", error);
        }
      },

      calcularFactorTipo() {
        let pk_tipos = [];
        let atacante = null;

        if (this.atacanteActual === 1) {
          atacante = this.detallesC.t1_pkActivo;
        } else {
          atacante = this.detallesC.t2_pkActivo;
        }

        atacante.tipos.forEach((tipo) => {
          pk_tipos.push(tipo.type.name);
        });

        let factor = 1;

        if (pk_tipos.includes("fire") && !pk_tipos.includes("water")) {
          factor *= 1.5;
        } else if (pk_tipos.includes("fire") && pk_tipos.includes("water")) {
          factor *= 1.5 * 0.5;
        } else if (!pk_tipos.includes("fire") && pk_tipos.includes("water")) {
          factor *= 0.5;
        } else {
          factor *= 1;
        }

        console.log("Los tipos del atacante son: ", pk_tipos);
        console.log("Por lo tanto su factor es: ", factor);

        return factor;
      },
      
      setearHpRestante(damage) {

        console.log("EL DAÑO RECIBIDO POR LA FUNCIÓN SETEAR HP ES: " , damage);

        if (this.atacanteActual === 1) {
          this.pk2_hp = parseFloat((this.pk2_hp - damage).toFixed(2));
          if (this.pk2_hp <= 0) {
            this.pk2_hp = 0;
            this.isButtonDisabled = true;
            this.isInvisible = !this.isInvisible;
            setTimeout(() => {
              this.pokemonDebilitado = true;
              this.pk_debilitadoData = this.detallesC.t2_pkActivo;
              console.log("SE HA ACTUALIZADO LA DATA DEL POKEMON DEBILITADO Y ES: ", this.pk_debilitadoData);
              this.enviarAPokemonBattle();
            }, 3000);
          }
        } else {
          this.pk1_hp = parseFloat((this.pk1_hp - damage).toFixed(2));
          if (this.pk1_hp <= 0) {
            this.pk1_hp = 0;
            this.isButtonDisabled = true;
            this.isInvisible = !this.isInvisible;
            setTimeout(() => {
              this.pokemonDebilitado = true;
              this.pk_debilitadoData = this.detallesC.t1_pkActivo;
              console.log("SE HA ACTUALIZADO LA DATA DEL POKEMON DEBILITADO Y ES: ", this.pk_debilitadoData);
              this.enviarAPokemonBattle();
            }, 3000);
          }
        }
      },

      esGolpeCrítico() {
        const min = 1;
        const max = 8;
        const numeroAleatorio = Math.floor(Math.random() * (max - min + 1)) + min;
        console.log(numeroAleatorio);
        if (numeroAleatorio == 7) {
          this.textoCritico = "Ha sido un golpe crítico !";
          return true;
          
        } else {
          this.textoCritico = "";
          return false;
        }
      },

      calcularDañoRealizado() {
        let atacante = {}
        let defensor = {}

        if (this.atacanteActual === 1) {
          atacante = this.detallesC.t1_pkActivo;
          defensor = this.detallesC.t2_pkActivo;
        } else {
          atacante = this.detallesC.t2_pkActivo;
          defensor = this.detallesC.t1_pkActivo;
        }
        let attackerStat = 0;
        let defenderStat = 0;

        console.log("ATACANTE: ", atacante, "DEFENSOR: ", defensor);

        // Se obtiene el stat de ataque del atacante
        if (this.claseAtaque === "special") {
          atacante.estadisticas.forEach((stat => {
            if (stat.stat.name === "special-attack") {
              attackerStat = stat.base_stat;
            }
          }))
        } else {
          atacante.estadisticas.forEach((stat => {
            if (stat.stat.name === "attack") {
              attackerStat = stat.base_stat;
            }
          }))
        }
        // Se obtiene el stat de defensa del defensor
        if (this.claseAtaque === "special") {
          defensor.estadisticas.forEach((stat => {
            if (stat.stat.name === "special-defense") {
              defenderStat = stat.base_stat;
            }
          }))
        } else {
          defensor.estadisticas.forEach((stat => {
            if (stat.stat.name === "defense") {
              defenderStat = stat.base_stat;
            }
          }))
        }

        console.log("Los stats de ataque y defensa de atacante y defensor respectivamente son: ", attackerStat, " y ", defenderStat);

        const daño_parcial = parseFloat((attackerStat / defenderStat).toFixed(2));

        const multiplicador = this.calcularFactorTipo(atacante);
        const critico = this.esGolpeCrítico();

        if (critico) {
          const dañoFinal = parseFloat((10 * daño_parcial * multiplicador * 1.33).toFixed(2))
          this.dañoRealizado = dañoFinal;
          console.log("EL DAÑO A REALIZAR ES: " , dañoFinal)
          this.setearHpRestante(dañoFinal);
          
        } else {
          const dañoFinal = parseFloat((10 * daño_parcial * multiplicador).toFixed(2))
          this.dañoRealizado = dañoFinal;
          console.log("EL DAÑO A REALIZAR ES: " , dañoFinal)
          this.setearHpRestante(dañoFinal);
          
        }
      }
    },
    
    async mounted() {
      this.$refs.audioPlayer.volume = 0.01; // Volumen inicial bajo (10%)
      console.log('Se han obtenido los pokemons que batallan: ', this.detallesC);

      this.detallesC.t1_pkActivo.estadisticas.forEach((stat => {
        if (stat.stat.name === "hp") {
          this.pk1_hp = stat.base_stat;
          this.pk1_hp_total = stat.base_stat;
        }
      }))

      this.detallesC.t2_pkActivo.estadisticas.forEach((stat => {
        if (stat.stat.name === "hp") {
          this.pk2_hp = stat.base_stat;
          this.pk2_hp_total = stat.base_stat;
        }
      }))

      //while (this.pk1_hp !== 0 && this.pk2_hp !== 0 && !this.pokemonDebilitado) {
      while (!this.pokemonDebilitado) {
        this.encontrarAtacante();
        await this.seleccionarAtaque();
        this.calcularDañoRealizado();

        await this.esperarCambioAtacante(); 
        
        this.cambiarAtacante();
        await this.seleccionarAtaque();
        this.calcularDañoRealizado();

        await this.esperarCambioTurno(); 
        
        this.avanzarTurno();
      }
    }
  };
  </script>
  
  <style scoped>

  .desaparecer {
    display: none;
  }

  h1 {
    font-size: 1.4em;
  }

  .vista-pokemon-debilitado {
    height: 100vh;
    width: 100vw;
    /*padding: 4em 12em;*/
    background-image: url('../assets/img/fondo-celeste2.png');
    background-size: cover;
    background-position: center; 
    background-repeat: no-repeat; 
    box-sizing: border-box;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0 6em;
  }

  .combate-individual-box {
    height: 80vh;
    width: 100vw;
    /*padding: 4em 12em;*/
    background-image: url('../assets/img/fondo-vs.png');
    background-size: cover;
    background-position: center; 
    background-size: 80% 100%;
    background-position: center;
    background-repeat: no-repeat; 
    box-sizing: border-box;
    display: flex;
    justify-content: space-around;
    align-items: center;
    padding: 0 2em;
  }

  .texto-box {
    display: flex;
    flex-direction: column;
    justify-content: space-evenly;
    align-items: center;
    color: black;
    font-size: 2em;
    margin: 0 auto;
    padding: 0 1em;
    height: 20vh;
    width: 100vw;
    background: linear-gradient(to right, yellow, #00CAF6);
    background: linear-gradient(to right, orange, red);
    box-sizing: border-box;
    border: 6px solid black;
    border-radius: 30px;
    .next {
      padding: 0.8em;
      background-color: #00CAF6;
      border-radius: 10px;
      font-size: 0.8em;
    }
  }

  .bloque-text {
    display: flex;
    justify-content: space-between;
    align-items: center;
    width: 100%;
    h1 {
      font-size: 1em;
    }

    button {
      margin: 0;
    }
  }

  .pk-team1, pk-team2 {
    transition: transform 2s ease;
  }

  .hp {
    width: 100%; /* El contenedor de la barra de vida ocupa todo el ancho */
  }

  .hp-border {
    border: 5px solid rgb(0, 53, 0);
  }

  .hp-bar {
    height: 20px;
    transition: width 0.5s ease, background-color 0.5s ease;
    background-color: green; /* Color por defecto (más del 50%) */
  }

  .low-hp {
    background-color: red; /* Menos del 25% */
  }

  .mid-hp {
    background-color: yellow; /* Entre 25% y 50% */
  }

  .high-hp {
    background-color: green; /* Más del 50% */
  }
  </style>