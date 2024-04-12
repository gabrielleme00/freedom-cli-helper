<script setup>
import { ref, computed } from 'vue'
import stores from './stores'

const version = APP_VERSION
const hmgTutorialLink = 'https://magazine.atlassian.net/wiki/spaces/SCF/pages/4015063363/FREEDOM+CLI+-+USANDO+AMBIENTE+DE+HMG?version=619203d9-da5f-44d9-8ce4-bd3dd64f2269'

const getLocalStorage = (key, defaultValue) => {
  const value = localStorage.getItem(key)
  return value !== null ? value : defaultValue
}

let path = ref(getLocalStorage('path', '/home/magalu/dev/luizalabs'))
let storeId = ref(getLocalStorage('storeId', stores[11].id))
let project = ref(getLocalStorage('project', 'pdp'))
let deployAllModules = ref(getLocalStorage('deployAllModules', true))
let pullStrategy = ref(getLocalStorage('pullStrategy', 'USE_LOCAL'))
let remoteWatch = ref(getLocalStorage('remoteWatch', false))

const cliStartCommand = computed(() => {
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

const cliWatchCommand = computed(() => {
  const pathTrim = (path.value || '').trim().replace(/\/+$/, '')
  const projTrim = (project.value || '').trim()
  const dist = stores.find(store => store.id == storeId.value).dist
  const ev = storeId.value === 'L_ESTANTEVIRTUAL'
  const remote = remoteWatch.value === 'true'
  return `
    ${remote ? 'watch-with-remote-deploy' : 'watch'}
    -p ${pathTrim}/ff-webstore-${ev ? 'ev-' : ''}${projTrim}-view/dist/${dist}
    -s ${storeId.value}
    -m webstore-${projTrim}
  `
})

const startViewCommand = computed(() => {
  const dist = stores.find(store => store.id == storeId.value).dist
  return `npm run dev --store=${dist}`
})

const startBffCommand = computed(() => {
  const remote = remoteWatch.value === 'true'
  return remote ? '' : 'npm run start:dev'
})

const copyText = id => {
  const text = {
    cliStartCommand: cliStartCommand,
    cliWatchCommand: cliWatchCommand,
    startViewCommand: startViewCommand,
    startBffCommand: startBffCommand,
  }[id].value.trim().replace(/[\r\n]/gm, '').replace(/\s+/g, ' ')
  navigator.clipboard.writeText(text)
}

const saveItem = e => {
  const { id, value } = e.target
  localStorage.setItem(id, value)
}
</script>

<template>
  <header>Freedom CLI Helper v{{ version }}</header>

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

    <div class="inputs">
      <div class="input-group">
        <h3>Projects Path</h3>
        <input id="path" v-model="path" @keyup="saveItem" placeholder="Ex: /home/username/dev">
      </div>
      <div class="input-group">
        <h3>Project</h3>
        <input id="project" v-model="project" @keyup="saveItem" placeholder="Ex: pdp">
      </div>
      <div class="input-group">
        <h3>Watch With Remote Deploy</h3>
        <select id="remoteWatch" v-model="remoteWatch" @change="saveItem">
          <option value="true">True</option>
          <option value="false">False</option>
        </select>
      </div>
    </div>

    <div class="results">
      <div class="result-group">
        <h3>Start</h3>
        <div class="code-wrapper">
          <button @click="() => copyText('cliStartCommand')">&#128203;</button>
          <code>{{ cliStartCommand }}</code>
        </div>
      </div>
      <div class="result-group">
        <h3>Watch</h3>
        <div class="code-wrapper">
          <button @click="() => copyText('cliWatchCommand')">&#128203;</button>
          <code>{{ cliWatchCommand }}</code>
        </div>
      </div>
      <div class="result-group">
        <h3>View</h3>
        <div class="code-wrapper">
          <button @click="() => copyText('startViewCommand')">&#128203;</button>
          <code>{{ startViewCommand }}</code>
        </div>
      </div>
      <div v-if="startBffCommand" class="result-group">
        <h3>BFF</h3>
        <div class="code-wrapper">
          <button @click="() => copyText('startBffCommand')">&#128203;</button>
          <code>{{ startBffCommand }}</code>
        </div>
      </div>
      <p v-else>
        Copie a versão gerada no CLI e cole na URL de HMG do projeto como
        <b>query param</b> (?version=VERSION_NUMBER).
        Veja o tutorial <a :href="hmgTutorialLink">aqui</a>.
      </p>
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

.result-group > .code-wrapper {
  display: flex;
  align-items: center;
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
