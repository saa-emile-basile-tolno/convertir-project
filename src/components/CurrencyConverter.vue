<script setup>
import { ref, watch } from 'vue'
import { devises, taux } from '../data/taux.js'

// Montant saisi : une chaîne pour faciliter la saisie au clavier
const montant = ref('0')
// Devises sélectionnées dans les deux listes
const deviseSource = ref('USD')
const deviseCible = ref('EUR')
// Résultat de la conversion (vide tant qu'on n'a pas converti)
const resultat = ref('')
// Affiche le message d'erreur quand le montant vaut 0
const erreur = ref(false)

// Les 12 touches du clavier numérique (ordre d'affichage de la grille 4 x 3)
const touches = ['7', '8', '9', '4', '5', '6', '1', '2', '3', 'C', '0', '.']

// Gère l'appui sur une touche du clavier de la page
function appuyer(touche) {
  // Le message d'erreur disparaît à la saisie suivante
  erreur.value = false

  // Touche « C » : on remet tout à zéro
  if (touche === 'C') {
    montant.value = '0'
    resultat.value = ''
    return
  }

  // Le montant est limité à 10 caractères
  if (montant.value.length >= 10) return

  // Un seul point décimal autorisé : le second est ignoré
  if (touche === '.' && montant.value.includes('.')) return

  // Montant initial « 0 » : le premier chiffre tapé le remplace
  // (mais si on tape « . », on garde le 0 pour obtenir « 0. »)
  if (montant.value === '0' && touche !== '.') {
    montant.value = touche
  } else {
    montant.value += touche
  }
}

// Convertit le montant en passant par l'euro (devise pivot)
function convertir() {
  const valeur = parseFloat(montant.value)

  // Règle de gestion : montant nul → message d'erreur
  if (!valeur) {
    erreur.value = true
    resultat.value = ''
    return
  }
  erreur.value = false

  // Étape 1 : montant source → euros
  const montantEnEuro = valeur / taux[deviseSource.value]
  // Étape 2 : euros → devise cible
  const montantConverti = montantEnEuro * taux[deviseCible.value]

  // Arrondi à 2 décimales
  resultat.value = montantConverti.toFixed(2)
}

// Échange les deux devises, puis relance la conversion si un résultat est affiché
function inverser() {
  const temporaire = deviseSource.value
  deviseSource.value = deviseCible.value
  deviseCible.value = temporaire

  if (resultat.value !== '') {
    convertir()
  }
}

// Bonus 1 : conversion automatique quand le montant ou une devise change
watch([montant, deviseSource, deviseCible], () => {
  if (parseFloat(montant.value)) {
    convertir()
  } else {
    // Montant nul : on vide le résultat sans afficher d'erreur
    resultat.value = ''
    erreur.value = false
  }
})
</script>

<template>
  <div class="carte">
    <!-- Zone du haut : affichage -->
    <div class="affichage">
      <h1>Currency Converter</h1>

      <!-- Devise source + montant saisi -->
      <div class="ligne">
        <select v-model="deviseSource">
          <option v-for="d in devises" :key="d" :value="d">{{ d }}</option>
        </select>
        <div class="valeur">{{ montant }}</div>
      </div>

      <!-- Bouton pour inverser les devises -->
      <button class="inverser" @click="inverser">⇅</button>

      <!-- Devise cible + résultat -->
      <div class="ligne">
        <select v-model="deviseCible">
          <option v-for="d in devises" :key="d" :value="d">{{ d }}</option>
        </select>
        <div class="valeur">{{ resultat }}</div>
      </div>

      <button class="convertir" @click="convertir">CONVERTIR</button>

      <!-- Message d'erreur conditionnel -->
      <p v-if="erreur" class="erreur">Veuillez saisir un montant</p>
    </div>

    <!-- Zone du bas : clavier numérique -->
    <div class="clavier">
      <button
        v-for="t in touches"
        :key="t"
        :class="{ 'touche-effacer': t === 'C' }"
        @click="appuyer(t)"
      >
        {{ t }}
      </button>
    </div>
  </div>
</template>

<style scoped>
/* Carte centrée, 360 px maximum */
.carte {
  max-width: 360px;
  margin: 0 auto;
  font-family: Arial, Helvetica, sans-serif;
  border-radius: 24px;
  overflow: hidden;
  background: #f7f9fa;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.12);
}

/* Zone du haut : fond bleu turquoise uni, en Flexbox */
.affichage {
  background: #2BB8E8;
  color: white;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
  padding: 20px;
}

h1 {
  margin: 0 0 8px;
  font-size: 20px;
  text-align: center;
  color: white;
}

/* Une ligne = liste déroulante à gauche, valeur à droite */
.ligne {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  width: 100%;
  gap: 12px;
}

select {
  padding: 6px 8px;
  border: 1px solid #d0d0d0;
  border-radius: 6px;
  background: white;
  color: #333;
  font-size: 14px;
}

/* Montant et résultat : alignés à droite, 48 px, soulignés */
.valeur {
  flex: 1;
  min-height: 58px;
  font-size: 48px;
  text-align: right;
  border-bottom: 2px solid white;
  overflow: hidden;
}

.inverser {
  background: none;
  border: none;
  color: white;
  font-size: 24px;
  cursor: pointer;
}

.convertir {
  background: white;
  color: #1BA6C9;
  border: none;
  border-radius: 24px;
  padding: 12px 40px;
  font-size: 14px;
  font-weight: bold;
  cursor: pointer;
}

.erreur {
  margin: 0;
  padding: 4px 12px;
  border-radius: 12px;
  background: #F0727A;
  color: white;
  font-size: 14px;
}

/* Zone du bas : clavier en CSS Grid (3 colonnes, 4 lignes) */
.clavier {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  padding: 16px;
}

/* Touches sans bordure ni fond, chiffres en bleu turquoise, 32 px */
.clavier button {
  background: none;
  border: none;
  padding: 14px;
  font-size: 32px;
  color: #1BA6C9;
  cursor: pointer;
}

/* Touche « C » en rouge clair (classe liée avec :class) */
.clavier .touche-effacer {
  color: #F0727A;
}
</style>