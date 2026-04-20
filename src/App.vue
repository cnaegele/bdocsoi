<template>
  <div v-if="messageerreur !== ''">{{ messageerreur }}</div>
   <form ref="formRef" method="POST" action="https://print-vdl-test.lausanne.ch/wsprint-v1.6/print/post"
    target="_top" >
    <input ref="applicationRef" type="hidden" name="application" value="GOELAND" />
    <input ref="documentRef" type="hidden" name="document" :value="'print' + siddoc" />
    <input ref="formatRef" type="hidden" name="format" value="7" />
    <input ref="fluxXML64Ref" type="hidden" name="fluxXML64" :value="dataxml64enc" />
  </form>
</template>

<script setup lang="ts">
import { ref, onMounted, nextTick } from 'vue'
import axios from 'axios'
import type { AxiosResponse } from 'axios'

const messageerreur = ref<string>('')
const urlParams = new URLSearchParams(window.location.search)
let idaffaire: number | null = null
let contexte: string | null = null
const prmsidaffaire = urlParams.get('idaffaire')
if (prmsidaffaire !== null && prmsidaffaire !== '') {
  idaffaire = stringToPositiveInteger(prmsidaffaire)
}
const prmscontexte = urlParams.get('contexte')
if (prmscontexte !== null && prmscontexte !== '') {
  contexte = prmscontexte
}
let bprmsok = true
let siddoccontexte: string = ''
let pagecontexte: string = ''
switch (contexte) {
  case 'afft327lettredispenseautorisation':
    siddoccontexte = 'OPCDispenseAutorisation'
    pagecontexte = '/goeland/bdocsoi/axios/afft327lettredispenseautorisation.php'
    break;
  default:
    messageerreur.value += 'parametre "contexte" manquant ou invalide. '
    bprmsok = false
}
if (idaffaire === null) {
  messageerreur.value += 'parametre "idaffaire" manquant ou invalide. '
  bprmsok = false
}
console.log(`idaffaire : ${idaffaire}`)
console.log(`contexte : ${contexte}`)
console.log(`bprmsok : ${bprmsok}`)
console.log(`messageerreur : ${messageerreur.value}`)

const siddoc = ref<string>('')
const dataxml64enc = ref<string>('')
siddoc.value = siddoccontexte
const formRef = ref<HTMLFormElement | null>(null)
const applicationRef = ref<HTMLFormElement | null>(null)    
const documentRef = ref<HTMLFormElement | null>(null)    
const formatRef = ref<HTMLFormElement | null>(null)    
const fluxXML64Ref = ref<HTMLFormElement | null>(null)    

onMounted(async () => {
  if (bprmsok) {
    let server: string = ''
    if (import.meta.env.DEV) {
      server = 'https://mygolux.lausanne.ch'
    }
    const page: string = pagecontexte
    const urlaffdl: string = `${server}${page}`
    const params = new URLSearchParams([['idaffaire', (idaffaire ?? 0).toString()]])
    const response: AxiosResponse = await axios.get(urlaffdl, { params })
    console.log(response.data)
    dataxml64enc.value = response.data
    await nextTick()
    console.log("input applicationRef contient", applicationRef.value)
    console.log("input documentRef contient", documentRef.value)
    console.log("input formatRef contient", formatRef.value)
    console.log("input fluxXML64 contient", fluxXML64Ref.value)
    formRef.value?.submit()
  }
})

function stringToPositiveInteger(str: string): number | null {
  // Vérifie que c'est une string non vide
  if (!str || str.trim() === '') {
    return null;
  }

  // Convertit en number
  const num = Number(str);

  // Vérifie que c'est un nombre valide, un entier et positif
  if (isNaN(num) || !Number.isInteger(num) || num < 0) {
    return null;
  }

  return num;
}
</script>

<style scoped></style>
