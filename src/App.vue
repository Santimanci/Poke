<template>
    <div class="cards" :class="claseFondo" :style="estiloFondo">
      <div class="search-container">
        <input 
          v-model="searchInput" 
          @keypress.enter="buscarPokemon"
          placeholder="Escribe nombre o número..."
          class="search-input"
        >
        <button @click="buscarPokemon" class="search-button">Buscar</button>
      </div>
      
      <div class="card-header">
        <h2>{{ "N°" + pokeid }}</h2>
        <p class="pokemon-name">{{ pokemonname }}</p>
      </div>
      
      <div class="image-container">
        <img :src="img1" alt="Imagen del Pokemon">
      </div>
      
      <div class="card-body">
        <div class="info-basica">
          <p><strong>Altura:</strong> {{ altura }} m</p>
          <p><strong>Peso:</strong> {{ peso }} kg</p>
        </div>

        <h3>Tipo del Pokemon:</h3>
        <div class="tipos">
          <span class="tipo" :class="espe">{{ espe }}</span>
          <span v-if="espes" class="tipo" :class="espes">{{ espes }}</span>
        </div>

        <h3>Debilidades:</h3>
        <div class="tipos">
          <span v-for="debilidad in debilidades" :key="debilidad" class="tipo" :class="debilidad">
            {{ debilidad }}
          </span>
        </div>

        <h4>Estadísticas:</h4>
        <div class="estadisticas-barras">
          <div v-for="stat in estadisticas" :key="stat.name" class="stat-bar">
            <span class="stat-name">{{ stat.name }}</span>
            <div class="bar-container">
              <div 
                class="bar-fill" 
                :style="{ width: calcularPorcentaje(stat.base) + '%' }"
              ></div>
              <span class="stat-value">{{ stat.base }}</span>
            </div>
          </div>
        </div>
      </div>

      <div class="button">
        <button @click="obtenerPokemonAleatorio">Obtener Pokémon Aleatorio</button>
      </div>
    </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import axios from 'axios'

let pokemonname = ref('')
let pokeid = ref('')
let img1 = ref('')
let espe = ref('')
let espes = ref('')
let estadisticas = ref([])
let searchInput = ref('')
let altura = ref('')
let peso = ref('')
let debilidades = ref([])

const tipoDebilidades = {
  normal: ['fighting'],
  fire: ['water', 'ground', 'rock'],
  water: ['electric', 'grass'],
  electric: ['ground'],
  grass: ['fire', 'ice', 'poison', 'flying', 'bug'],
  ice: ['fire', 'fighting', 'rock', 'steel'],
  fighting: ['flying', 'psychic', 'fairy'],
  poison: ['ground', 'psychic'],
  ground: ['water', 'grass', 'ice'],
  flying: ['electric', 'ice', 'rock'],
  psychic: ['bug', 'ghost', 'dark'],
  bug: ['fire', 'flying', 'rock'],
  rock: ['water', 'grass', 'fighting', 'ground', 'steel'],
  ghost: ['ghost', 'dark'],
  dragon: ['ice', 'dragon', 'fairy'],
  dark: ['fighting', 'bug', 'fairy'],
  steel: ['fire', 'fighting', 'ground'],
  fairy: ['poison', 'steel']
}

const claseFondo = computed(() => {
  if (!espe.value) return 'fondo-default'
  
  const tipo1 = espe.value;
  const tipo2 = espes.value;
  
  if (tipo2) {
    return 'fondo-degradado'
  } else {
    return `fondo-${tipo1}`
  }
})

const estiloFondo = computed(() => {
  if (!espe.value) return {}
  
  const tipo1 = espe.value;
  const tipo2 = espes.value;
  
  if (tipo2) {
    return {
      background: `linear-gradient(135deg, ${coloresTipos[tipo1]} 0%, ${coloresTipos[tipo2]} 100%)`
    }
  }
  
  return {}
})

// Mapeo de colores para los fondos
const coloresTipos = {
  normal: '#A8A878',
  fire: '#F08030',
  water: '#6890F0',
  electric: '#F8D030',
  grass: '#78C850',
  ice: '#98D8D8',
  fighting: '#C03028',
  poison: '#A040A0',
  ground: '#E0C068',
  flying: '#A890F0',
  psychic: '#F85888',
  bug: '#A8B820',
  rock: '#B8A038',
  ghost: '#705898',
  dragon: '#7038F8',
  dark: '#705848',
  steel: '#B8B8D0',
  fairy: '#EE99AC'
}

function calcularPorcentaje(baseStat) {
  const maxStat = 255;
  return (baseStat / maxStat) * 100;
}

function calcularDebilidades(tipos) {
  const debilidadesSet = new Set()
  
  tipos.forEach(tipo => {
    if (tipoDebilidades[tipo]) {
      tipoDebilidades[tipo].forEach(debilidad => {
        debilidadesSet.add(debilidad)
      })
    }
  })
  
  // Eliminar tipos que son inmunes
  tipos.forEach(tipo => {
    if (tipo === 'flying') {
      debilidadesSet.delete('ground')
    }
    if (tipo === 'ghost') {
      debilidadesSet.delete('fighting')
      debilidadesSet.delete('normal')
    }
  })
  
  return Array.from(debilidadesSet)
}

// TUS FUNCIONES EXISTENTES
async function buscarPokemon() {
  if (!searchInput.value.trim()) return;
  
  try {
    const identifier = searchInput.value.toLowerCase().trim();
    const response = await axios.get(`https://pokeapi.co/api/v2/pokemon/${identifier}`)
    console.log(`Datos del Pokemon buscado:`, response.data)
    
    procesarDatosPokemon(response.data)
    searchInput.value = ''
    
  } catch(error) {
    console.log(`Error al buscar el Pokemon:`, error)
    alert('Pokémon no encontrado. Intenta con otro nombre o número.')
  }
}

async function obtenerPokemonAleatorio(){
  try{
    const idAleatorio = generarIdAleatorio()
    const response = await axios.get(`https://pokeapi.co/api/v2/pokemon/${idAleatorio}`)
    console.log(`Datos del Pokemon:`, response.data)
    
    procesarDatosPokemon(response.data)

  } catch(error){
    console.log(`Error al obtener el Pokemon:`, error)
  }
}

function procesarDatosPokemon(pokemonData) {
  pokemonname.value = pokemonData.name
  pokeid.value = pokemonData.id
  img1.value = pokemonData.sprites.front_default
  espe.value = pokemonData.types[0].type.name
  espes.value = pokemonData.types[1]?.type.name || ''
  
  altura.value = (pokemonData.height / 10).toFixed(1)
  peso.value = (pokemonData.weight / 10).toFixed(1)
  
  const tipos = pokemonData.types.map(t => t.type.name)
  debilidades.value = calcularDebilidades(tipos)
  
  estadisticas.value = pokemonData.stats.map(stat => ({
    name: abreviarStatName(stat.stat.name),
    base: stat.base_stat
  }))
}

function abreviarStatName(statName) {
  const statsMap = {
    'hp': 'HP',
    'attack': 'ATK',
    'defense': 'DEF',
    'special-attack': 'SP. ATK',
    'special-defense': 'SP. DEF',
    'speed': 'SPD'
  }
  return statsMap[statName] || statName
}

function generarIdAleatorio(){
  return Math.floor(Math.random() * 898) + 1;   
}

obtenerPokemonAleatorio()
</script>

<style>
* {
  box-sizing: border-box;
}

body {
  color: rgb(0, 0, 0);
  background-color: rgb(0, 0, 0);
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  margin: 0;
  padding: 20px;
  font-family: 'Arial', sans-serif;
}

.cards {
  border: 3px solid #3a3a3a;
  border-radius: 15px;
  width: 380px; 
  min-height: 580px;
  text-align: center;
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.2);
  padding: 12px; 
  position: relative;
  overflow: hidden;
  transition: all 0.3s ease;
}

/* FONDOS POR TIPO - UN SOLO TIPO */
.fondo-normal { background: linear-gradient(135deg, #A8A878 0%, #d8d8b8 100%); }
.fondo-fire { background: linear-gradient(135deg, #F08030 0%, #f8b070 100%); }
.fondo-water { background: linear-gradient(135deg, #6890F0 0%, #a8c0f8 100%); }
.fondo-electric { background: linear-gradient(135deg, #F8D030 0%, #f8e070 100%); }
.fondo-grass { background: linear-gradient(135deg, #78C850 0%, #a8e080 100%); }
.fondo-ice { background: linear-gradient(135deg, #98D8D8 0%, #c8f0f0 100%); }
.fondo-fighting { background: linear-gradient(135deg, #C03028 0%, #e07068 100%); }
.fondo-poison { background: linear-gradient(135deg, #A040A0 0%, #d080d0 100%); }
.fondo-ground { background: linear-gradient(135deg, #E0C068 0%, #f0d8a0 100%); }
.fondo-flying { background: linear-gradient(135deg, #A890F0 0%, #d0c0f8 100%); }
.fondo-psychic { background: linear-gradient(135deg, #F85888 0%, #f8a0c0 100%); }
.fondo-bug { background: linear-gradient(135deg, #A8B820 0%, #d8e070 100%); }
.fondo-rock { background: linear-gradient(135deg, #B8A038 0%, #e0c878 100%); }
.fondo-ghost { background: linear-gradient(135deg, #705898 0%, #a090c8 100%); }
.fondo-dragon { background: linear-gradient(135deg, #7038F8 0%, #a870f8 100%); }
.fondo-dark { background: linear-gradient(135deg, #705848 0%, #a09078 100%); }
.fondo-steel { background: linear-gradient(135deg, #B8B8D0 0%, #e0e0f0 100%); }
.fondo-fairy { background: linear-gradient(135deg, #EE99AC 0%, #f8c0d0 100%); }

.cards::before {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 8px; 
  background: linear-gradient(90deg, #ff0000 0%, #ff0000 50%, #ffffff 50%, #3a3a3a 100%);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px; 
  padding-bottom: 8px; 
  border-bottom: 2px solid rgba(255, 255, 255, 0.3);
}

.card-header h2 {
  font-family: 'Arial', sans-serif;
  font-size: 20px; 
  margin: 0;
  color: #000000;
  font-weight: bold;
}

.pokemon-name {
  font-family: 'Arial', sans-serif;
  font-size: 22px; 
  margin: 0;
  text-transform: capitalize;
  font-weight: bold;
  color: #000000;
}

.image-container {
  background-color: rgba(255, 255, 255, 0.3);
  border-radius: 8px;
  padding: 12px;
  margin: 12px 0; 
  border: 2px solid rgba(255, 255, 255, 0.5);
  box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(5px);
}

img {
  width: 150px; 
  height: 150px; 
  image-rendering: pixelated;
}

.card-body {
  margin: 12px 0; 
}

.card-body h3, .card-body h4 {
  font-size: 14px; 
  margin: 12px 0 8px 0;
  color: #000000;
  text-align: left;
}

.info-basica {
  display: flex;
  justify-content: space-around;
  margin: 12px 0; 
  padding: 8px; 
  background-color: rgba(255, 255, 255, 0.3);
  border-radius: 6px; 
 border: 1px solid rgba(255, 255, 255, 0.5);
  backdrop-filter: blur(5px);
}

.info-basica p {
  margin: 0;
  font-size: 12px; 
  color: #000000;
  font-weight: bold;
}

.tipos {
  display: flex;
  justify-content: flex-start;
  gap: 8px;
  margin-top: 4px; 
  flex-wrap: wrap;
}

.tipo {
  padding: 4px 12px;
  border-radius: 15px; 
  color: white;
  font-weight: bold;
  text-transform: capitalize;
  font-size: 12px; 
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

/* Colores de tipos */
.tipo.normal { background-color: #A8A878; }
.tipo.fire { background-color: #F08030; }
.tipo.water { background-color: #6890F0; }
.tipo.electric { background-color: #F8D030; }
.tipo.grass { background-color: #78C850; }
.tipo.ice { background-color: #98D8D8; }
.tipo.fighting { background-color: #C03028; }
.tipo.poison { background-color: #A040A0; }
.tipo.ground { background-color: #E0C068; }
.tipo.flying { background-color: #A890F0; }
.tipo.psychic { background-color: #F85888; }
.tipo.bug { background-color: #A8B820; }
.tipo.rock { background-color: #B8A038; }
.tipo.ghost { background-color: #705898; }
.tipo.dragon { background-color: #7038F8; }
.tipo.dark { background-color: #705848; }
.tipo.steel { background-color: #B8B8D0; }
.tipo.fairy { background-color: #EE99AC; }


.estadisticas-barras {
  margin-top: 8px; 
}

.stat-bar {
  display: flex;
  align-items: center;
  margin-bottom: 6px; 
  gap: 8px; 
}

.stat-name {
  font-size: 10px;
  font-weight: bold;
  color: #000000;
  width: 70px; 
  text-align: left;
  text-transform: uppercase;
}

.bar-container {
  flex: 1;
  background-color: rgba(255, 255, 255, 0.3);
  border-radius: 8px; 
  height: 16px; 
  position: relative;
  overflow: hidden;
  border: 1px solid rgba(0, 0, 0, 0.2);
}

.bar-fill {
  height: 100%;
  background: linear-gradient(90deg, #4CAF50, #8BC34A);
  border-radius: 8px; 
  transition: width 0.5s ease-in-out;
  box-shadow: inset 0 0 5px rgba(255, 255, 255, 0.5);
}

.stat-value {
  position: absolute;
  right: 6px; 
  top: 50%;
  transform: translateY(-50%);
  font-size: 9px; 
  font-weight: bold;
  color: #000000;
  text-shadow: 0 0 2px rgba(255, 255, 255, 0.8);
}

.button {
  margin-top: 15px; 
}

button {
  padding: 8px 16px; 
  font-size: 14px; 
  background-color: rgba(43, 45, 43, 0.8);
  color: rgb(255, 255, 255);
  border: none;
  border-radius: 6px; 
  cursor: pointer;
  font-weight: bold;
  transition: all 0.3s ease;
  width: 100%;
  backdrop-filter: blur(5px);
}

button:hover {
  background-color: rgba(248, 23, 23, 0.9);
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.search-container {
  display: flex;
  gap: 8px; 
  margin-bottom: 15px;
  justify-content: center;
}

.search-input {
  padding: 8px;
  border: 2px solid #3a3a3a;
  border-radius: 6px; 
   font-size: 14px; 
  width: 180px; 
  background-color: rgba(255, 255, 255, 0.9);
}

.search-button {
  padding: 8px 16px; 
  background-color: rgba(43, 45, 43, 0.9);
  color: white;
  border: none;
  border-radius: 6px; 
  cursor: pointer;
  font-weight: bold;
  transition: all 0.3s ease;
  font-size: 14px; 
}

.search-button:hover {
  background-color: rgba(248, 23, 23, 0.9);
  transform: translateY(-2px);
}
</style>