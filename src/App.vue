<template>
  <div class="bg-gray-900 h-screen flex items-center justify-center font-sans">
    <div ref="cardRef" class="card" :class="cardClasses" @click="flipCard">
      <div v-if="!pokemon" class="flex items-center justify-center h-full">
        <div class="animate-spin rounded-full h-16 w-16 border-t-2 border-b-2 border-white"></div>
      </div>
      
      <div v-if="pokemon" class="card-inner" :class="{ 'is-flipped': isFlipped }">
        <div class="card-face card-front">
          <div class="pokemon-image-container">
            <img :src="pokemon.image" :alt="pokemon.name" class="pokemon-image" />
            <h1 class="pokemon-name">{{ pokemon.name }}</h1>
            <div class="flex justify-center gap-2 mt-1">
              <span v-for="type in pokemon.types" :key="type" class="type-tag" :class="getTypeColor(type)">
                {{ type }}
              </span>
            </div>
          </div>
          
          <div class="info-section">
            <h2 class="section-title">Abilities</h2>
            <div class="flex flex-wrap justify-center gap-2 mb-4">
              <span v-for="ability in pokemon.abilities" :key="ability" class="ability-tag">
                {{ ability }}
              </span>
            </div>
            
            <div class="flex justify-between mt-3">
              <div class="text-center">
                <span class="text-xs text-white/80">Weight</span>
                <p class="text-sm">{{ pokemon.weight / 10 }} kg</p>
              </div>
              <div class="text-center">
                <span class="text-xs text-white/80">Height</span>
                <p class="text-sm">{{ pokemon.height / 10 }} m</p>
              </div>
              <div class="text-center">
                <span class="text-xs text-white/80">ID</span>
                <p class="text-sm">#{{ pokemon.id }}</p>
              </div>
            </div>
          </div>
          
          <div class="card-flip-hint">
            Click to see stats
          </div>
        </div>
        
        <div class="card-face card-back">
          <h1 class="pokemon-name mb-3">{{ pokemon.name }} Stats</h1>
          
          <div class="info-section p-4">
            <div class="space-y-4">
              <div v-for="stat in pokemon.stats" :key="stat.name" class="stat-container">
                <div class="stat-header">
                  <span class="stat-name">{{ stat.name }}</span>
                  <span class="stat-value">{{ stat.value }}</span>
                </div>
                <div class="stat-bar-bg">
                  <div class="h-2.5 rounded-full" :class="getStatColor(stat.name)" :style="`width: ${(stat.value / 255) * 100}%`"></div>
                </div>
              </div>
            </div>
            
            <div class="mt-6">
              <h2 class="section-title">Characteristic</h2>
              <p class="text-center text-sm">
                {{ getCharacteristic(pokemon.stats) }}
              </p>
            </div>
          </div>
          
          <div class="card-flip-hint">
            Click to see front
          </div>
        </div>
      </div>
      
      <button v-if="pokemon" @click.stop="fetchPokemon" class="new-pokemon-button">
        🔄 New Pokémon
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import gsap from 'gsap'

const cardRef = ref(null)
const pokemon = ref(null)
const isFlipped = ref(false)

const getTypeColor = (type) => {
  const typeColors = {
    normal: 'type-normal',
    fire: 'type-fire',
    water: 'type-water',
    electric: 'type-electric',
    grass: 'type-grass',
    ice: 'type-ice',
    fighting: 'type-fighting',
    poison: 'type-poison',
    ground: 'type-ground',
    flying: 'type-flying',
    psychic: 'type-psychic',
    bug: 'type-bug',
    rock: 'type-rock',
    ghost: 'type-ghost',
    dragon: 'type-dragon',
    dark: 'type-dark',
    steel: 'type-steel',
    fairy: 'type-fairy'
  }
  return typeColors[type] || 'type-normal'
}

const getStatColor = (statName) => {
  const statColors = {
    'hp': 'stat-hp',
    'attack': 'stat-attack',
    'defense': 'stat-defense',
    'sp. attack': 'stat-special-attack',
    'sp. defense': 'stat-special-defense',
    'speed': 'stat-speed'
  }
  return statColors[statName] || 'stat-default'
}

const getCharacteristic = (stats) => {
  const highestStat = stats.reduce((prev, current) => 
    (prev.value > current.value) ? prev : current
  )
  
  const characteristics = {
    'hp': 'Loves to eat',
    'attack': 'Proud of its power',
    'defense': 'Sturdy body',
    'sp. attack': 'Highly curious',
    'sp. defense': 'Strong willed',
    'speed': 'Likes to run'
  }
  
  return characteristics[highestStat.name] || 'Mischievous'
}

const cardClasses = computed(() => {
  if (!pokemon.value || !pokemon.value.types || pokemon.value.types.length === 0) {
    return 'card-default'
  }
  
  const primaryType = pokemon.value.types[0]
  const secondaryType = pokemon.value.types.length > 1 ? pokemon.value.types[1] : primaryType
  
  return [`card-${primaryType}`, `card-secondary-${secondaryType}`]
})

const flipCard = (event) => {
  if (event.target.tagName === 'BUTTON' || event.target.closest('button')) {
    return
  }
  
  isFlipped.value = !isFlipped.value
}

const fetchPokemon = async () => {
  pokemon.value = null
  isFlipped.value = false
  
  const id = Math.floor(Math.random() * 150) + 1
  try {
    const res = await fetch(`https://pokeapi.co/api/v2/pokemon/${id}`)
    const data = await res.json()
    
    pokemon.value = {
      id: data.id,
      name: data.name.charAt(0).toUpperCase() + data.name.slice(1),
      image: data.sprites.other['official-artwork'].front_default,
      types: data.types.map(t => t.type.name),
      stats: [
        { name: 'hp', value: data.stats[0].base_stat },
        { name: 'attack', value: data.stats[1].base_stat },
        { name: 'defense', value: data.stats[2].base_stat },
        { name: 'sp. attack', value: data.stats[3].base_stat },
        { name: 'sp. defense', value: data.stats[4].base_stat },
        { name: 'speed', value: data.stats[5].base_stat }
      ],
      abilities: data.abilities.map(a => a.ability.name),
      height: data.height,
      weight: data.weight
    }
    
    resetAnimation()
  } catch (error) {
    console.error('Error fetching Pokemon:', error)
  }
}

const resetAnimation = () => {
  gsap.killTweensOf(cardRef.value)
  
  gsap.set(cardRef.value, {
    y: 0,
    scale: 1,
    opacity: 1,
    boxShadow: "0 0 20px rgba(255, 255, 255, 0.4)"
  })
  
  gsap.to(cardRef.value, {
    y: -20,
    duration: 2,
    ease: "sine.inOut",
    repeat: -1,
    yoyo: true
  })

  gsap.to(cardRef.value, {
    scale: 1.05,
    duration: 3,
    ease: "power1.inOut",
    repeat: -1,
    yoyo: true,
    delay: 0.5
  })

  gsap.to(cardRef.value, {
    boxShadow: "0 0 80px rgba(255, 255, 255, 0.8)",
    duration: 2.5,
    repeat: -1,
    yoyo: true,
    ease: "power2.inOut"
  })

  gsap.from(cardRef.value, {
    scale: 0,
    opacity: 0,
    duration: 2,
    ease: "back.out(1.7)"
  })
}

onMounted(() => {
  fetchPokemon()
})
</script>
