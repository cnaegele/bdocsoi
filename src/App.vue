<template>
  <div v-if="messageerreur !== ''">{{ messageerreur }}</div>
  <form ref="formRef" method="POST" :action="formaction" target="_top">
    <input ref="applicationRef" type="hidden" name="application" value="GOELAND" />
    <input ref="documentRef" type="hidden" name="document" :value="'print' + siddoc" />
    <input ref="formatRef" type="hidden" name="format" :value="sformatdoc" />
    <input ref="fluxXML64Ref" type="hidden" name="fluxXML64" :value="dataxml64enc" />
  </form>
</template>

<script setup lang="ts">
import { ref, onMounted, nextTick } from 'vue'
import axios from 'axios'
import type { AxiosResponse } from 'axios'

const messageerreur = ref<string>('')
const formaction = ref<string>('https://print-vdl.lausanne.ch/wsprint-v1.6/print/post')
//L'authentification via loginsso pose problème car le POST est transformé en GET lors du processus d'authentification.
//Problème connu au SOI.
//Proposition SOI (01.06.2026 Laurent Dormont)
//Faire que l'application envoie tout d'abord une requête HTTP GET sur https://print-vdl.lausanne.ch/wsprint-v1.6/jsp/print.jsp 
//afin de passer le processus d'authentification SSO, puis envoyer la requête HTTP POST avec les donnes du formulaire
const urldummysession = ref<string>('https://print-vdl.lausanne.ch/wsprint-v1.6/jsp/print.jsp')
const urlParams = new URLSearchParams(window.location.search)
let idaffaire: number | null = null
let idenveloppe: number | null = null
let iddocument: number | null = null
let contexte: string | null = null
let environnement: string = 'prod'
let controle: string = ''
const prmsidaffaire = urlParams.get('idaffaire')
if (prmsidaffaire !== null && prmsidaffaire !== '') {
  idaffaire = stringToPositiveInteger(prmsidaffaire)
}
const prmscontexte = urlParams.get('contexte')
if (prmscontexte !== null && prmscontexte !== '') {
  contexte = prmscontexte
}
const prmsenvironnement = urlParams.get('environnement')
if (prmsenvironnement !== null && prmsenvironnement !== '') {
  environnement = prmsenvironnement
}
const prmscontrole = urlParams.get('controle')
if (prmscontrole !== null && prmscontrole !== '') {
  controle = prmscontrole
}
switch (environnement) {
  case 'test':
    formaction.value = 'https://print-vdl-test.lausanne.ch/wsprint-v1.6/print/post'
    urldummysession.value = 'https://print-vdl-test.lausanne.ch/wsprint-v1.6/jsp/print.jsp'
    break
  case 'vali':
    formaction.value = 'https://print-vdl-vali.lausanne.ch/wsprint-v1.6/print/post'
    urldummysession.value = 'https://print-vdl-vali.lausanne.ch/wsprint-v1.6/jsp/print.jsp'
    break
}
const prmsidocument = urlParams.get('iddocument')
const prmsidenveloppe = urlParams.get('idenveloppe')


let bprmsok = true
let siddoccontexte: string = ''
let sformatdoccontexte: string = ''
let pagecontexte: string = ''
let params = new URLSearchParams([['idaffaire', (idaffaire ?? 0).toString()]])

switch (contexte) {
  case 'afft11lettredemandeprealable':
    siddoccontexte = 'OPCDemandePrealable'
    sformatdoccontexte = '7'
    pagecontexte = '/goeland/bdocsoi/axios/afft11lettredemandeprealable.php'
    break;
  case 'afft76permishabiterutiliser':
    siddoccontexte = 'PermisHabiterUtiliser'
    sformatdoccontexte = '7'
    pagecontexte = '/goeland/bdocsoi/axios/afft76permishabiterutiliser.php'
    break;
  case 'afft218correctionenquete':
    siddoccontexte = 'OPCCorrectionAvantEnquete'
    sformatdoccontexte = '7'
    pagecontexte = '/goeland/bdocsoi/axios/afft218lettrecorrectionenquete.php'
    break;
  case 'afft218informationenquete':
    siddoccontexte = 'OPCInformationEnquete'
    sformatdoccontexte = '7'
    pagecontexte = '/goeland/bdocsoi/axios/afft218lettreinformationenquete.php'
    break;
  case 'afft218avisenquete':
    siddoccontexte = 'OPCAvisEnquete'
    sformatdoccontexte = '7'
    pagecontexte = '/goeland/bdocsoi/axios/afft218avisenquete.php'
    break;
  case 'afft218accusereceptionoppint':
    siddoccontexte = 'OPCAccuseReceptionOppInt'
    sformatdoccontexte = '7'
    pagecontexte = '/goeland/bdocsoi/axios/afft218accusereceptionoppint.php'
    if (prmsidocument !== null && prmsidocument !== '') {
      iddocument = stringToPositiveInteger(prmsidocument)
    }
    params = new URLSearchParams([['idaffaire', (idaffaire ?? 0).toString()], ['iddocument', (iddocument ?? 0).toString()]])
    break; 
  case 'afft218abandonprojet':
    siddoccontexte = 'OPCAbandonProjet'
    sformatdoccontexte = '7'
    pagecontexte = '/goeland/bdocsoi/axios/afft218lettreabandonprojet.php'
    if (prmsidenveloppe !== null && prmsidenveloppe !== '') {
      idenveloppe = stringToPositiveInteger(prmsidenveloppe)
    }
    params = new URLSearchParams([['idaffaire', (idaffaire ?? 0).toString()], ['idenveloppe', (idenveloppe ?? 0).toString()]])
    break;
  case 'afft218permisconstruire':
    siddoccontexte = 'PermisDeConstruire'
    sformatdoccontexte = '7'
    pagecontexte = '/goeland/bdocsoi/axios/afft218permisconstruire.php'
    break;
  case 'afft327lettredispenseautorisation':
    siddoccontexte = 'OPCDispenseAutorisation'
    sformatdoccontexte = '7'
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
console.log(`environnement : ${environnement}`)
console.log(`formaction : ${formaction.value}`)
console.log(`params : ${params}`)
console.log(`bprmsok : ${bprmsok}`)
console.log(`messageerreur : ${messageerreur.value}`)

const siddoc = ref<string>('')
const sformatdoc = ref<string>('')
const dataxml64enc = ref<string>('')
siddoc.value = siddoccontexte
sformatdoc.value = sformatdoccontexte
const formRef = ref<HTMLFormElement | null>(null)
const applicationRef = ref<HTMLFormElement | null>(null)
const documentRef = ref<HTMLFormElement | null>(null)
const formatRef = ref<HTMLFormElement | null>(null)
const fluxXML64Ref = ref<HTMLFormElement | null>(null)

/**
 * Établit la session F5 SSO via window.open.
 * Ouvre une mini-fenêtre vers le service, F5 intercepte et fait le SSO,
 * le cookie est posé. On attend 1 secondes, puis on ferme la fenêtre.
 */
function etablirSessionSso(): Promise<void> {
  return new Promise((resolve) => {
    const wssosession = window.open(
      urldummysession.value,
      'ssosession',
      'width=1,height=1,left=-9999,top=-9999'
    )

    if (!wssosession) {
      console.warn('popup SSO bloqué par le navigateur, on continue sans')
      resolve()
      return
    }

    // On laisse 2.5 secondes pour que F5 fasse le SSO complet,
    // puis on ferme la fenêtre et on continue
    setTimeout(() => {
      try { wssosession.close() } catch { /* ignore */ }
      console.log('session F5 SSO établie via popup (délai 1s)')
      resolve()
    }, 2500)
  })
}
onMounted(async () => {
  if (bprmsok) {
    // Lancer l'établissement SSO en parallèle de la récupération des données
    const ssoPromise = etablirSessionSso()

    let server: string = ''
    if (import.meta.env.DEV) {
      server = 'https://mygolux.lausanne.ch'
    }
    const page: string = pagecontexte
    const urlaffdl: string = `${server}${page}`
    console.log(params)
    const response: AxiosResponse = await axios.get(urlaffdl, { params })
    console.log(response.data)
    dataxml64enc.value = response.data
    await nextTick()
    console.log("input applicationRef contient", applicationRef.value)
    console.log("input documentRef contient", documentRef.value)
    console.log("input formatRef contient", formatRef.value)
    console.log("input fluxXML64 contient", fluxXML64Ref.value)
    if (dataxml64enc.value.indexOf('ERREUR:') === 0) {
      messageerreur.value += dataxml64enc.value
    } else {
      if (controle !== 'nopostdebug') {
        // Attendre que la session SSO soit établie avant de submit
        await ssoPromise
        console.log('SSO prêt, soumission du formulaire')
        formRef.value?.submit()
      }
    }
  }
})

function stringToPositiveInteger(str: string): number | null {
  if (!str || str.trim() === '') {
    return null;
  }
  const num = Number(str);
  if (isNaN(num) || !Number.isInteger(num) || num < 0) {
    return null;
  }
  return num;
}
</script>

<style scoped></style>