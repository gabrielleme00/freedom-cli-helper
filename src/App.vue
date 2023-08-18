<script setup>
import { ref, computed } from 'vue'

const stores = [
  { name: 'Allianz', id: 'L_ALLIANZPARQUE', dist: 'allianzparque' },
  { name: 'Clube Netshoes', id: 'L_CLUBENETSHOES', dist: 'clubenetshoes' },
  { name: 'ComSchool', id: 'L_COMSCHOOL', dist: 'comschool' },
  { name: 'Estante Virtual', id: 'L_ESTANTEVIRTUAL', dist: 'estantevirtual' },
  { name: 'Fluxo', id: 'L_FLUXO', dist: 'fluxo' },
  { name: 'Freelace', id: 'L_FREELACE', dist: 'freelace' },
  { name: 'Gap', id: 'L_GAP', dist: 'gapbrasil' },
  { name: 'Kappa', id: 'L_KAPPA', dist: 'kappa' },
  { name: 'Loja da Chape', id: 'L_CHAPECOENSE', dist: 'lojadachape' },
  { name: 'Loja do Inter', id: 'L_INTERNACIONAL', dist: 'lojadointer' },
  { name: 'NBA', id: 'L_NBA', dist: 'zattini' },
  { name: 'Netshoes', id: 'L_NETSHOES', dist: 'netshoesbr' },
  { name: 'NFL', id: 'L_NFL', dist: 'nflshop' },
  { name: 'Rainha', id: 'L_RAINHA', dist: 'rainha' },
  { name: 'Santos', id: 'L_SANTOS', dist: 'santosstore' },
  { name: 'SP Mania', id: 'L_SAOPAULO', dist: 'saopaulomania' },
  { name: 'Shoestock', id: 'L_SHOESTOCK', dist: 'shoestock' },
  { name: 'Shop Cruzeiro', id: 'L_CRUZEIRO', dist: 'shopcruzeiro' },
  { name: 'Shop Timão', id: 'L_CORINTHIANS', dist: 'shoptimao' },
  { name: 'Shop Vasco', id: 'L_VASCO', dist: 'shopvasco' },
  { name: 'Topper', id: 'L_TOPPER', dist: 'topper' },
  { name: 'WSL', id: 'L_WSL', dist: 'wslstore' },
  { name: 'Zattini', id: 'L_ZATTINI', dist: 'zattini' },
]

let path = ref(localStorage.getItem('path') || '/home/magalu/dev/luizalabs')
let storeId = ref(localStorage.getItem('storeId') || stores[11].id)
let project = ref(localStorage.getItem('project') || 'pdp')
let deployAllModules = ref(localStorage.getItem('deployAllModules') || true)
let pullStrategy = ref(localStorage.getItem('pullStrategy') || 'USE_LOCAL')

const startCode = computed(() => {
  const dam = deployAllModules.value === 'true' ? '-dam ' : ''
  const store = storeId.value || 'L_NETSHOES'
  const ps = pullStrategy.value
  return `
    start
    ${dam}
    -s ${store}
    -ps ${ps}
  `
})

const watchCode = computed(() => {
  const pathTrim = (path.value || '').trim().replace(/\/+$/, '')
  const projTrim = (project.value || '').trim()
  const dist = stores.find(store => store.id == storeId.value).dist
  return `
    watch
    -p ${pathTrim}/ff-webstore-${projTrim}-view/dist/${dist}
    -s ${storeId.value}
    -m webstore-${projTrim}
  `
})

const copyText = id => {
  let text = id == 'start' ? startCode.value : watchCode.value
  text = text.trim().replace(/[\r\n]/gm, '').replace(/\s+/g, ' ')
  navigator.clipboard.writeText(text)
}

const saveItem = e => {
  const { id, value } = e.target
  localStorage.setItem(id, value)
  console.log(id, value)
}
</script>

<template>
  <header>
    Freedom CLI Helper
  </header>

  <main>
    <div class="inputs">
      <div class="input-group">
        <h3>Store</h3>
        <select id="storeId" v-model="storeId" @change="saveItem">
          <option v-for="store in stores" :value="store.id">
            {{ store.name }}
          </option>
        </select>
      </div>
      <div class="input-group">
        <h3>Projects Path</h3>
        <input id="path" v-model="path" @keyup="saveItem">
      </div>
    </div>

    <div class="inputs">
      <div class="input-group">
        <h3>Project</h3>
        <input id="project" v-model="project" @keyup="saveItem">
      </div>
      <div class="input-group">
        <h3>Deploy All Modules</h3>
        <select id="deployAllModules" v-model="deployAllModules" @change="saveItem">
          <option value="true">True</option>
          <option value="false">False</option>
        </select>
      </div>
      <div class="input-group">
        <h3>Pull Strategy</h3>
        <select id="pullStrategy" v-model="pullStrategy" @change="saveItem">
          <option value="USE_LOCAL">USE_LOCAL</option>
          <option value="FORCE_PULL">FORCE_PULL</option>
        </select>
      </div>
    </div>

    <div class="results">
      <div class="result-group">
        <h3>Start</h3>
        <button @click="() => copyText('start')">&#128203;</button>
        <code>{{ startCode }}</code>
      </div>
      <div class="result-group">
        <h3>Watch</h3>
        <button @click="() => copyText('watch')">&#128203;</button>
        <code> {{ watchCode }}</code>
      </div>
    </div>

  </main>
</template>

<style scoped>
header {
  text-align: center;
  font-size: 2rem;
}

.inputs {
  display: flex;
  justify-content: space-between;
  column-gap: 20px;
  margin: var(--section-gap) 0;
}

@media screen and (max-width: 768px) {
  .inputs {
    flex-direction: column;
  }
}

.input-group {
  flex-grow: 1;
  background-color: var(--color-background-soft);
  padding: 20px;
  border-radius: 4px;
}

.results {
  display: flex;
  flex-direction: column;
  row-gap: 20px;
  background-color: var(--color-background-mute);
  padding: 20px;
  border-radius: 4px;
}

.result-group h3 {
  margin-bottom: 10px;
}

.result-group button {
  margin-right: 20px;
  font-size: 24px;
}

code {
  display: inline-block;
  background-color: var(--color-background);
  border-left: 4px solid var(--nord7);
  padding: 10px;
  margin: 10px 0;
}

code::before {
  content: '> ';
}
</style>
