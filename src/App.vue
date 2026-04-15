<template>
  <iframe name="printFrame" style="width: 100%; height: 100vh; border: none;" />
  <form ref="formRef" method="POST" action="https://print-vdl-test.lausanne.ch/wsprint-v1.6/print/post"
    target="printFrame" style="display: none;">
    <input type="hidden" name="application" value="GOELAND" />
    <input type="hidden" name="document" :value="'print' + siddoc" />
    <input type="hidden" name="format" value="7" />
    <input type="hidden" name="fluxXML64" :value="dataxml64enc" />
  </form>
</template>

<script setup lang="ts">
import { ref, onMounted, nextTick } from 'vue'
import axios from 'axios'
import type { AxiosResponse, AxiosError } from 'axios'

const urlParams = new URLSearchParams(window.location.search)
let idaffaire: number | null = null
const prmsidaffaire = urlParams.get('idaffaire')
if (prmsidaffaire !== null && prmsidaffaire !== '') {
  idaffaire = stringToPositiveInteger(prmsidaffaire)
}
const siddoc = ref<string>('OPCDispenseAutorisation')
const dataxml64enc = ref<string>('')
const formRef = ref<HTMLFormElement | null>(null)
onMounted(async () => {
  if (idaffaire !== null) {
    console.log(`idaffaire : ${idaffaire}`)

    let server: string = ''
    if (import.meta.env.DEV) {
      server = 'https://mygolux.lausanne.ch'
    }
    const page: string = '/goeland/bdocsoi/axios/afft327lettredispenseautorisation.php'
    const urlaffdl: string = `${server}${page}`
    const params = new URLSearchParams([['idaffaire', idaffaire.toString()]])
    const response: AxiosResponse = await axios.get(urlaffdl, { params })
    console.log(response.data)
    dataxml64enc.value = response.data
    await nextTick()
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
