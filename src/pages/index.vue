<template>
  <div class="h-screen flex justify-center w-full bg-orange-100">
    <div class="w-full max-w-5xl mx-auto relative">
      <div class="grid grid-cols-3">
        <div>
          <img
            src="../assets/img/castoman-high-resolution-logo-transparent.png"
            class="h-20 m-4"
          />
        </div>
        <div class="text-center text-gray-800 text-4xl py-8 font-extrabold uppercase">
          <h1>Inventaire</h1>
        </div>
        <div class="flex justify-center items-center">
          <div
            @click="popup = true"
            class="mx-auto border-2 border-gray-800 rounded-lg bg-red-600 hover:bg-red-700 transition hover:scale-105 flex justify-center items-center gap-x-4 cursor-pointer"
          >
            <img
              src="https://www.nicepng.com/png/full/251-2519428_0-add-icon-white-png.png"
              class="w-5 h-5 ml-4"
            />
            <div class="text-gray-100 font-bold border-l-2 border-gray-800 px-6 py-3">
              Ajouter un produit
            </div>
          </div>
        </div>
      </div>

      <div class="w-full flex justify-center p-8">
        <div class="grid grid-cols-3 gap-8 w-full">
          <div
            v-for="produit in inventaire"
            :key="produit.nom"
            class="pl-4 py-3 pr-12 border-2 border-gray-800 relative bg-gray-100"
          >
            <div class="flex flex-col justify-center">
              <div class="text-sm font-bold uppercase">
                {{ produit.nom }}
              </div>
              <div class="text-red-700 text-2xl font-bold">{{ produit.prix }} $</div>
            </div>

            <div
              class="rounded-full w-10 h-10 text-center bg-white border-2 border-gray-800 text-gray-800 absolute -right-2 -bottom-3"
            >
              <div
                class="flex justify-center items-center w-full h-full text-xs font-bold"
              >
                x{{ produit.quantite }}
              </div>
            </div>
          </div>
        </div>
      </div>

      <div class="absolute bottom-0 w-full">
        <div class="flex justify-center pb-3">
          <div class="bg-white roudned-xl border-2 border-gray-800 px-6 py-2">
              Monnaies: 
              <select v-model="monnaieSouhaite">
                <option v-for="monnaie in monnaies" :key="monnaie.currency" :value="monnaie.currency">{{ monnaie.currency }}</option>
              </select>
          </div>
        </div>
        <div
          class="grid grid-cols-3 py-4 px-20 bg-red-700 text-xl text-gray-100 border-t-2 border-gray-800 border-x-2 rounded-t-full"
        >
          <div class="font-semibold cursor-pointer bg-gray-700 p-2 self-center text-center w-60 rounded-xl text-lg border-2 border-white hover:bg-gray-600" @click="exportToCSV()">Exportez en CSV</div>

          <button
            type="button"
            @click="convertCurrency()"
            class="hover:scale-110 transition cursor-pointer border-2 border-gray-800 rounded-xl bg-gray-100 text-sm font-bold uppercase py-3 text-center text-gray-800"
          >
            Convertion en {{ monnaieSouhaite }}
          </button>
          <div class="font-semibold text-right self-center">
            Prix total :
            <span class="font-bold">
              <span v-if="prixTotalConvertit === 0">{{ prixTotalInitial }}</span> 
              <span v-else>{{ prixTotalConvertit }}</span>
              {{ monnaieActuelle }}</span>
          </div>
        </div>
      </div>
    </div>
    <Popup
      v-if="popup"
      @close="popup = false"
      @addProduct="addProduct"
      @modifyProduct="modifyProduct"
      :inventaire="inventaire"
      class="absolute"
    />
  </div>
</template>

<script>
import inventoryData from "../model/products.json";

export default {
  data() {
    return {
      inventaire: inventoryData,
      prixTotalInitial: 0,
      prixTotalConvertit: 0,
      popup: false,
      monnaies: [],
      monnaieActuelle: 'USD',
      monnaieSouhaite: 'USD',
      isUpdate: false
    };
  },
  async mounted() {
    await this.calculateTotalPrice();
    await this.getCurrency()
  },
  methods: {
    addProduct(nouveauProduit) {
      this.inventaire.push(nouveauProduit);
      this.calculateTotalPrice();
      this.convertCurrency()
    },
    modifyProduct(majProduit) {
      console.log('modification')
      const index = this.inventaire.findIndex(
        (produit) => produit.nom === majProduit.nom
      );
      if (index !== -1) {
        this.inventaire[index].quantite = majProduit.quantite;
        this.calculateTotalPrice();
        this.convertCurrency();
      }
    },
    calculateTotalPrice() {
      this.prixTotalInitial = 0;
      for (let i = 0; i < this.inventaire.length; i++) {
        this.prixTotalInitial =
          this.inventaire[i].prix * this.inventaire[i].quantite + this.prixTotalInitial;
      }
    },
    async getCurrency() {
      try {
        const response = await useFetch(
          `https://v6.exchangerate-api.com/v6/4a24708336c2f16228120e03/latest/USD`
        );
        const conversionRate = response.data._value.conversion_rates;

        if (conversionRate) {
          this.monnaies = Object.entries(conversionRate).map(([currency, value]) => ({ currency, value }));
        } else {
          console.error("Echec d'accès à la liste des monnaies");
        }
      } catch (error) {
        console.error("Requête indisponible:", error);
      }
    },
    async convertCurrency() {
      try {
        const response = await useFetch(
          `https://v6.exchangerate-api.com/v6/4a24708336c2f16228120e03/latest/USD`
        );

        const conversionRate = response.data._value.conversion_rates[this.monnaieSouhaite];
        console.log('Monnaie Actuelle', this.monnaieActuelle)

        console.log('Monnaie Souhaité', this.monnaieSouhaite)

        console.log('Prix convertit', conversionRate)

        if (conversionRate) {
          this.prixTotalConvertit = this.prixTotalInitial;
      
          this.prixTotalConvertit *= conversionRate;
          this.prixTotalConvertit = Math.round(this.prixTotalConvertit);
          
          this.monnaieActuelle = this.monnaieSouhaite
        
        } else {
          console.error("Conversion rate not available for the target currency.");
        }
      } catch (error) {
        console.error("Error fetching conversion rates:", error);
      }
    },
    exportToCSV() {
      const csvContent = this.formatInventoryToCSV();
      this.downloadCSV(csvContent, 'inventory.csv');
    },
    formatInventoryToCSV() {
      let csvContent = "Nom du produit,Quantité,Prix (USD)\n";
      this.inventaire.forEach((produit) => {
        csvContent += `"${produit.nom}",${produit.quantite},${produit.prix}\n`;
      });
      return csvContent;
    },
    downloadCSV(csvContent, filename) {
      const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
      if (navigator.msSaveBlob) {
        // For IE 10 and above
        navigator.msSaveBlob(blob, filename);
      } else {
        const link = document.createElement('a');
        if (link.download !== undefined) {
          // Browsers that support HTML5 download attribute
          const url = URL.createObjectURL(blob);
          link.setAttribute('href', url);
          link.setAttribute('download', filename);
          link.style.visibility = 'hidden';
          document.body.appendChild(link);
          link.click();
          document.body.removeChild(link);
        }
      }
    }
  },
};
</script>
