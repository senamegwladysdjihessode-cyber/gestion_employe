<template>
  <div class="page">
    <h1>👥 Gestion des Employés</h1>

    <!-- Statistiques -->
    <div class="stats">
      <div class="stat-card">
        <h4>Total employés</h4>
        <p>{{ nombreEmployes }}</p>
      </div>
      <div class="stat-card">
        <h4>Masse salariale</h4>
        <p>{{ salaireTotal }} FCFA</p>
      </div>
      <div class="stat-card">
        <h4>Salaire moyen</h4>
        <p>{{ salaireMoyen }} FCFA</p>
      </div>
    </div>

    <div class="conteneur">
      <!-- Formulaire -->
      <div class="formulaire">
        <h3>{{ modeModification ? 'Modifier l\'employé' : 'Ajouter un employé' }}</h3>
        <label>Nom</label>
        <input v-model="employe.nom" placeholder="Nom" />
        <label>Prénom</label>
        <input v-model="employe.prenom" placeholder="Prénom" />
        <label>Poste</label>
        <input v-model="employe.poste" placeholder="Poste occupé" />
        <label>Département</label>
        <select v-model="employe.departement">
          <option value="">Choisir un département</option>
          <option value="Informatique">Informatique</option>
          <option value="Ressources Humaines">Ressources Humaines</option>
          <option value="Finance">Finance</option>
          <option value="Marketing">Marketing</option>
          <option value="Production">Production</option>
        </select>
        <label>Salaire (FCFA)</label>
        <input v-model="employe.salaire" type="number" placeholder="Salaire mensuel" />
        <button class="btn-ajouter" @click="modeModification ? sauvegarder() : ajouter()">
          {{ modeModification ? '💾 Sauvegarder' : '➕ Ajouter' }}
        </button>
        <button v-if="modeModification" class="btn-annuler" @click="annuler">Annuler</button>
      </div>

      <!-- Tableau -->
      <div class="tableau-container">
        <div class="filtres">
          <input v-model="recherche" placeholder="🔍 Rechercher un employé..." />
          <select v-model="filtreDepartement">
            <option value="">Tous les départements</option>
            <option value="Informatique">Informatique</option>
            <option value="Ressources Humaines">Ressources Humaines</option>
            <option value="Finance">Finance</option>
            <option value="Marketing">Marketing</option>
            <option value="Production">Production</option>
          </select>
        </div>

        <table>
          <thead>
            <tr>
              <th>Nom</th>
              <th>Prénom</th>
              <th>Poste</th>
              <th>Département</th>
              <th>Salaire</th>
              <th>Actions</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(e, index) in employesFiltres" :key="index">
              <td v-html="surligner(e.nom)"></td>
              <td>{{ e.prenom }}</td>
              <td>{{ e.poste }}</td>
              <td><span class="badge-dept">{{ e.departement }}</span></td>
              <td>{{ e.salaire }} FCFA</td>
              <td class="actions">
                <button class="btn-modifier" @click="modifier(index)">✏️</button>
                <button class="btn-supprimer" @click="supprimer(index)">❌</button>
              </td>
            </tr>
            <tr v-if="employesFiltres.length === 0">
              <td colspan="6" style="text-align:center; color:#888">Aucun employé trouvé</td>
            </tr>
          </tbody>
        </table>

        <!-- Répartition par département -->
        <div class="repartition">
          <h4>Répartition par département</h4>
          <div v-for="dept in departements" :key="dept" class="ligne-dept">
            <span>{{ dept }}</span>
            <span>{{ compterParDepartement(dept) }} employé(s)</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed } from 'vue'
import Swal from 'sweetalert2'

const employe = reactive({
  nom: "",
  prenom: "",
  poste: "",
  departement: "",
  salaire: 0
})

const employes = ref([])
const recherche = ref("")
const filtreDepartement = ref("")
const modeModification = ref(false)
const indexModification = ref(null)

const departements = ['Informatique', 'Ressources Humaines', 'Finance', 'Marketing', 'Production']

const nombreEmployes = computed(() => employes.value.length)

const salaireTotal = computed(() => {
  return employes.value.reduce((acc, e) => acc + Number(e.salaire), 0)
})

const salaireMoyen = computed(() => {
  if (employes.value.length === 0) return 0
  return Math.round(salaireTotal.value / employes.value.length)
})

const employesFiltres = computed(() => {
  return employes.value.filter(e => {
    const matchRecherche = e.nom.toLowerCase().includes(recherche.value.toLowerCase())
    const matchDept = filtreDepartement.value === "" || e.departement === filtreDepartement.value
    return matchRecherche && matchDept
  })
})

function compterParDepartement(dept) {
  return employes.value.filter(e => e.departement === dept).length
}

function surligner(nom) {
  if (!recherche.value) return nom
  const regex = new RegExp(`(${recherche.value})`, 'gi')
  return nom.replace(regex, '<mark>$1</mark>')
}

function ajouter() {
  if (employe.nom === "" || employe.prenom === "" || employe.poste === "" || employe.departement === "" || employe.salaire <= 0) {
    Swal.fire({ icon: 'error', title: 'Erreur !', text: 'Veuillez remplir tous les champs correctement !' })
    return
  }
  employes.value.push({
    nom: employe.nom,
    prenom: employe.prenom,
    poste: employe.poste,
    departement: employe.departement,
    salaire: Number(employe.salaire)
  })
  Swal.fire({ icon: 'success', title: 'Succès !', text: 'Employé ajouté !', timer: 2000, showConfirmButton: false })
  annuler()
}

function modifier(index) {
  const e = employes.value[index]
  employe.nom = e.nom
  employe.prenom = e.prenom
  employe.poste = e.poste
  employe.departement = e.departement
  employe.salaire = e.salaire
  indexModification.value = index
  modeModification.value = true
}

function sauvegarder() {
  if (employe.nom === "" || employe.prenom === "" || employe.poste === "" || employe.departement === "" || employe.salaire <= 0) {
    Swal.fire({ icon: 'error', title: 'Erreur !', text: 'Veuillez remplir tous les champs correctement !' })
    return
  }
  employes.value[indexModification.value] = {
    nom: employe.nom,
    prenom: employe.prenom,
    poste: employe.poste,
    departement: employe.departement,
    salaire: Number(employe.salaire)
  }
  Swal.fire({ icon: 'success', title: 'Modifié !', text: 'Employé modifié avec succès !', timer: 2000, showConfirmButton: false })
  annuler()
}

function annuler() {
  employe.nom = ""
  employe.prenom = ""
  employe.poste = ""
  employe.departement = ""
  employe.salaire = 0
  modeModification.value = false
  indexModification.value = null
}

function supprimer(index) {
  Swal.fire({
    title: 'Êtes-vous sûr ?',
    text: 'Cette action est irréversible !',
    icon: 'warning',
    showCancelButton: true,
    confirmButtonColor: '#e07b6a',
    cancelButtonColor: '#8a9bb0',
    confirmButtonText: 'Oui, supprimer !',
    cancelButtonText: 'Annuler'
  }).then((result) => {
    if (result.isConfirmed) {
      employes.value.splice(index, 1)
      Swal.fire({ icon: 'success', title: 'Supprimé !', text: 'Employé supprimé !', timer: 2000, showConfirmButton: false })
    }
  })
}
</script>

<style scoped>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

.page {
  background: #f5f0e8;
  min-height: 100vh;
  padding-bottom: 30px;
}

h1 {
  text-align: center;
  color: #5a3e2b;
  padding: 25px;
  font-size: 28px;
  background: #e8dcc8;
  border-bottom: 2px solid #c9b99a;
}

.stats {
  display: flex;
  gap: 15px;
  max-width: 1400px;
  margin: 20px auto;
  padding: 0 20px;
}

.stat-card {
  flex: 1;
  background: #fff8ef;
  padding: 20px;
  border-radius: 12px;
  border: 1px solid #d9c8b0;
  text-align: center;
  box-shadow: 0 4px 12px rgba(0,0,0,0.08);
}

.stat-card h4 {
  color: #7a5c3e;
  margin-bottom: 10px;
  font-size: 14px;
}

.stat-card p {
  color: #5a3e2b;
  font-size: 24px;
  font-weight: bold;
}

.conteneur {
  display: flex;
  gap: 20px;
  max-width: 1400px;
  margin: 20px auto;
  align-items: flex-start;
  padding: 0 20px;
}

.formulaire {
  width: auto;
  min-width: 280px;
  max-width: 350px;
  background: #fff8ef;
  padding: 25px;
  border-radius: 16px;
  border: 1px solid #d9c8b0;
  box-shadow: 0 4px 12px rgba(0,0,0,0.08);
}

h3 {
  color: #5a3e2b;
  margin-bottom: 20px;
  font-size: 18px;
}

label {
  color: #7a5c3e;
  display: block;
  margin-bottom: 5px;
  font-size: 14px;
  font-weight: bold;
}

input, select {
  width: 100%;
  padding: 10px;
  margin-bottom: 15px;
  border-radius: 8px;
  border: 1px solid #d9c8b0;
  background: #fdf6ec;
  color: #5a3e2b;
  font-size: 14px;
}

input::placeholder {
  color: #b0a090;
}

.btn-ajouter {
  width: 100%;
  padding: 12px;
  background: #8a6a4a;
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
  transition: transform 0.1s;
  margin-bottom: 10px;
}

.btn-ajouter:hover {
  transform: scale(1.02);
  background: #7a5a3a;
}

.btn-annuler {
  width: 100%;
  padding: 10px;
  background: #c9b99a;
  color: #5a3e2b;
  border: none;
  border-radius: 8px;
  font-size: 14px;
  cursor: pointer;
  transition: transform 0.1s;
}

.btn-annuler:hover {
  transform: scale(1.02);
}

.tableau-container {
  flex: 1;
  background: #fff8ef;
  padding: 20px;
  border-radius: 16px;
  border: 1px solid #d9c8b0;
  box-shadow: 0 4px 12px rgba(0,0,0,0.08);
}

.filtres {
  display: flex;
  gap: 10px;
  margin-bottom: 15px;
}

.filtres input {
  flex: 1;
  margin-bottom: 0;
}

.filtres select {
  margin-bottom: 0;
  width: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: #e8dcc8;
}

th {
  padding: 12px;
  color: #5a3e2b;
  text-align: left;
  font-size: 14px;
  font-weight: bold;
}

td {
  padding: 12px;
  color: #5a3e2b;
  border-bottom: 1px solid #e8dcc8;
  font-size: 14px;
  word-wrap: break-word;
}

tr:hover {
  background: #f5ede0;
}

.badge-dept {
  background: #e8dcc8;
  color: #5a3e2b;
  padding: 4px 10px;
  border-radius: 20px;
  font-size: 12px;
  font-weight: bold;
}

.actions {
  display: flex;
  gap: 5px;
  align-items: center;
}

.btn-modifier {
  background: #c9a96e;
  border: none;
  padding: 6px 10px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  transition: transform 0.1s;
}

.btn-modifier:hover {
  transform: scale(1.1);
}

.btn-supprimer {
  background: #e07b6a;
  border: none;
  padding: 6px 10px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  transition: transform 0.1s;
}

.btn-supprimer:hover {
  transform: scale(1.1);
}

.repartition {
  margin-top: 25px;
  padding-top: 20px;
  border-top: 1px solid #e8dcc8;
}

.repartition h4 {
  color: #5a3e2b;
  margin-bottom: 15px;
}

.ligne-dept {
  display: flex;
  justify-content: space-between;
  padding: 8px 0;
  color: #7a5c3e;
  font-size: 14px;
  border-bottom: 1px solid #f0e8d8;
}

mark {
  background: #f0c070;
  color: #5a3e2b;
  border-radius: 4px;
  padding: 0 3px;
}
</style>