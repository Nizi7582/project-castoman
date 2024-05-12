<template>
  <div class="">
    <div class="z-50 relative w-screen h-screen">
      <div
        class="p-7 flex justify-center items-center fixed left-0 top-0 w-full h-full bg-gray-900 bg-opacity-50 z-50 duration-300"
      >
        <div class="bg-white flex rounded-lg w-1/2 relative">
          <div class="flex flex-col items-start">
            <div class="p-7 flex justify-between items-center w-full">
              <div class="text-gray-900 font-bold text-lg">
                <span v-if="!dejaExistant">Ajout d'un nouveau produit</span>
                <span v-if="dejaExistant">Mise à jour d'un ancien produit</span>

              </div>
              
              <svg
                @click="ClosePopup()"
                class="ml-auto fill-current text-gray-700 w-5 h-5 cursor-pointer"
                xmlns="http://www.w3.org/2000/svg"
                viewBox="0 0 18 18"
              >
                <path
                  d="M14.53 4.53l-1.06-1.06L9 7.94 4.53 3.47 3.47 4.53 7.94 9l-4.47 4.47 1.06 1.06L9 10.06l4.47 4.47 1.06-1.06L10.06 9z"
                />
              </svg>
            </div>

            <div class="grid grid-cols-3 gap-y-6">
              <div class="px-7 overflow-x-hidden">
                <div class="space-y-2">
                  <label
                    class="text-sm font-medium leading-none peer-disabled:cursor-not-allowed peer-disabled:opacity-70 "
                    for="nom"
                    >Nom du produit</label
                  >
                  <input 
                    class="flex h-10 w-full rounded-md border px-3 py-2 text-sm file:bg-transparent disabled:cursor-not-allowed disabled:opacity-50 capitalize"
                    id="nom"
                    type="text"
                    placeholder="Cahier"
                    name="nom"
                    v-model="nom"
                    v-if="!dejaExistant"
                  />
                  <select v-if="dejaExistant" v-model="nom" class="flex h-10 w-full rounded-md border px-3 py-2 text-sm file:bg-transparent disabled:cursor-not-allowed disabled:opacity-50 capitalize">
                    <option v-for="produit in inventaire" :key="produit.nom" :value="produit.nom">{{ produit.nom }}</option>
                  </select>
                </div>
              </div>
              <div class="px-7 overflow-x-hidden">
                <div class="space-y-2">
                  <label
                    class="text-sm font-medium leading-none peer-disabled:cursor-not-allowed peer-disabled:opacity-70"
                    for="quantite"
                    >Quantité</label
                  >
                  <input
                    class="flex h-10 w-full rounded-md border px-3 py-2 text-sm file:bg-transparent disabled:cursor-not-allowed disabled:opacity-50"
                    id="quantite"
                    type="number"
                    placeholder="10"
                    name="quantite"
                    v-model="quantite"
                  />
                </div>
              </div>
              <div v-if="!dejaExistant" class="px-7 overflow-x-hidden">
                <div class="space-y-2">
                  <label
                    class="text-sm font-medium leading-none peer-disabled:cursor-not-allowed peer-disabled:opacity-70"
                    for="prix"
                    >Prix en $ (USD)</label
                  >
                  <input
                    class="flex h-10 w-full rounded-md border px-3 py-2 text-sm file:bg-transparent disabled:cursor-not-allowed disabled:opacity-50"
                    id="prix"
                    type="number"
                    placeholder="77"
                    name="prix"
                    v-model="prix"
                  />
                </div>
              </div>
            </div>

            <div class="p-7 flex justify-between items-center w-full underline">
              <button type="button" @click="dejaExistant = !dejaExistant">
                <span v-if="!dejaExistant">Produit déja existant ?</span>
                <span v-if="dejaExistant">Nouveau produit ?</span>

              </button>
              <div>
                <button
                  type="button"
                  @click="Validate()"
                  class="bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 rounded mr-3"
                >
                  Ajouter
                </button>
                <button
                  type="button"
                  @click="ClosePopup()"
                  class="bg-transparent hover:bg-gray-500 text-red-700 font-semibold hover:text-white py-2 px-4 border border-red-500 hover:border-transparent rounded"
                >
                  Fermer
                </button>
              </div>
              
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      open: true,
      nom: "",
      quantite: "",
      prix: "",
      dejaExistant: false
    };
  },
  props: {
    inventaire: {
      type: Array, 
      required: true
    }
  },
  methods: {
    ClosePopup() {
      this.open = false;
      this.$emit("close", true);
    },
    Validate() {
      if (!this.dejaExistant) {
        const nouveauProduit = {
          nom: this.nom,
          quantite: this.quantite,
          prix: this.prix,
        };
        this.$emit("addProduct", nouveauProduit);
      }
      if (this.dejaExistant) {

        const majProduit = {
          nom: this.nom,
          quantite: this.quantite,
        };
        console.log(majProduit)

        this.$emit("modifyProduct", majProduit);
      }  
      this.ClosePopup();
    },
  },
};
</script>
