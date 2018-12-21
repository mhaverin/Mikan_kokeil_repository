<template>
  <div id="app">
    <h1>TEHTÄVÄLISTA</h1>

    <input v-model="uusiTehtava" placeholder="Uusi tehtävä" />
    <input v-model="uusiAlaTehtava" placeholder="Uusi alatehtävä" />
    <input v-model="uusiPrioriteetti" placeholder="Anna Priority" />
    <link
      rel="stylesheet"
      href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css"
    />

    <button v-on:click="lisaaTehtava" class="btn btn-primary">
      Lisää tehtävä
    </button>

    <template v-for="(tehtava, indeksi1) in tehtavaLista">
      <div>
        <div v-on:click="valitseTehtava(indeksi1);" class="eka">
          <template v-if="tehtava.done">
            lisaaTehdytListaan (tehtava,indeksi1);
          </template>
          <template v-else>
            <strong>{{ indeksi1 + 1 }}: Tehtävä: {{ tehtava.teksti }}</strong
            >, Prio: {{ tehtava.prioriteetti }}
          </template>
        </div>
        <div>
          <button v-on:click="lisaaAlaTehtava(indeksi1);" class="btn btn-info">
            Lisää alatehtävä
          </button>

          <button
            v-on:click="poistaTehtava(tehtavaLista, indeksi1);"
            class="btn btn-danger"
          >
            Poista
          </button>
          <button v-on:click="muutaPrioriteetti(indeksi1);" class="btn">
            Muuta Prioriteetti
          </button>
        </div>
      </div>

      <template v-for="(alaTehtava, indeksi2) in tehtava.alaTehtavaLista">
        <div>
          <div v-on:click="valitseAlaTehtava(indeksi1, indeksi2);" class="eka">
            <template v-if="alaTehtava.done">
              {{ indeksi1 + 1 }}.{{ indeksi2 + 1 }}: Alatehtävä:
              {{ alaTehtava.teksti }}
              <strong style="color:green;">TEHTY!</strong>
            </template>
            <template v-else>
              {{ indeksi1 + 1 }}.{{ indeksi2 + 1 }}: Alatehtävä:
              {{ alaTehtava.teksti }}
            </template>
          </div>
          <div>
            <button
              v-on:click="poistaAlaTehtava(indeksi1, indeksi2);"
              class="btn btn-danger"
            >
              Poista
            </button>
          </div>
        </div>
      </template>
    </template>
    <br /><br /><br />
    <div style="width: 600px;color: grey; background: rgb(0,70,0);">
      <strong>Tehdyt tehtävät:</strong> <br />
      <template v-for="(tehtava, indeksi1) in tehdytTehtavatLista">
        <button v-on:click="poistaTehtava(tehdytTehtavatLista, indeksi1);">
          Poista
        </button>
        <button v-on:click="palautaTehtava(indeksi1);">Palauta</button>
        {{ indeksi1 + 1 }}: Tehtävä: {{ tehtava.teksti }} <br />
        <template v-for="(alaTehtava, indeksi2) in tehtava.alaTehtavaLista">
          {{ indeksi1 + 1 }}.{{ indeksi2 + 1 }}: Alatehtävä:{{
            alaTehtava.teksti
          }}
          <button v-on:click="poistaTehtyAlaTehtava(indeksi1, indeksi2);">
            Poista
          </button>

          <br />
        </template>
      </template>
    </div>
  </div>
</template>

<script>
import { components } from 'aws-amplify-vue'

export default {
  name: "App",
  components: {},
  data() {
    return {
      tehtavaLista: [],
      alaTehtavaLista: [],
      tehdytTehtavatLista: [],
      alaTehtava: "",
      tehtava: "",
      uusiTehtava: "",
      uusiAlaTehtava: "",
      prioriteetti: "",
      uusiPrioriteetti: ""
    };
  },

  mounted: function() {
    console.log("mounted kutsuttu");
    this.lataa();
    this.lataaTehdyt();
  },
  methods: {
    lisaaTehtava: function() {
      // luodaan tehtavaobjekti, joka näyttää: { teksti: ..., done: ... }
      var tehtava = {
        teksti: this.uusiTehtava,
        done: false,
        prioriteetti: "3",
        alaTehtavaLista: []
      };
      this.tehtavaLista.push(tehtava);
      this.uusiTehtava = "";
      this.tallenna(this.tehtavaLista);
    },

    lisaaAlaTehtava: function(index1) {
      // luodaan tehtavaobjekti, joka näyttää: { teksti: ..., done: ... }
      var alaTehtava = {
        teksti: this.uusiAlaTehtava,
        done: false
      };
      this.tehtavaLista[index1].alaTehtavaLista.push(alaTehtava);
      this.uusiAlaTehtava = "";
      this.tallenna(this.tehtavaLista);
    },

    lataa: function() {
      var vanhaData = localStorage.getItem("tekemattomat");

      // jos old_stateen ei ole koskaan vielä talletettu mitään,
      // se on "null" --> ei aseteta sitä minnekään
      if (vanhaData) {
        this.tehtavaLista = JSON.parse(vanhaData);
      }
    },

    lataaTehdyt: function() {
      var vanhaData = localStorage.getItem("tehdyt");

      // jos old_stateen ei ole koskaan vielä talletettu mitään,
      // se on "null" --> ei aseteta sitä minnekään
      if (vanhaData) {
        this.tehdytTehtavatLista = JSON.parse(vanhaData);
      }
    },

    tallenna: function(tieto) {
      console.log("tallenna tekemättömät");
      localStorage.setItem("tekemattomat", JSON.stringify(tieto));
    },

    tallennaTehdyt: function(tieto) {
      console.log("tallenna tehdyt");
      localStorage.setItem("tehdyt", JSON.stringify(tieto));
    },

    muutaPrioriteetti: function(index) {
      this.tehtavaLista[index].prioriteetti = this.uusiPrioriteetti;
      this.tallenna(this.tehtavaLista);
    },

    poistaTehtava: function(lista, index) {
      lista.splice(index, 1);
      this.tallenna(this.tehtavaLista);
      this.tallennaTehdyt(this.tehdytTehtavatLista);
    },

    palautaTehtava: function(index) {
      this.tehdytTehtavatLista[index].done = false;
      this.tehtavaLista.push(this.tehdytTehtavatLista[index]);
      this.tehdytTehtavatLista.splice(index, 1);
      this.tallenna(this.tehtavaLista);
      this.tallennaTehdyt(this.tehdytTehtavatLista);
    },

    poistaTehtyAlaTehtava: function(index1, index2) {
      //console.log(index1);
      this.tehdytTehtavatLista[index1].alaTehtavaLista.splice(index2, 1);
      this.tallenna(this.tehtavaLista);
      this.tallennaTehdyt(this.tehdytTehtavatLista);
    },

    lisaaTehdytListaan: function(index) {
      this.tehdytTehtavatLista.push(this.tehtavaLista[index]);
      this.tehtavaLista.splice(index, 1);
      this.tallenna(this.tehtavaLista);
      this.tallennaTehdyt(this.tehdytTehtavatLista);
    },

    poistaAlaTehtava: function(index1, index2) {
      //console.log(index1);
      this.tehtavaLista[index1].alaTehtavaLista.splice(index2, 1);
      this.tallenna(this.tehtavaLista);
      this.tallennaTehdyt(this.tehdytTehtavatLista);
    },

    valitseTehtava: function(index) {
      this.tehtavaLista[index].done = !this.tehtavaLista[index].done;
      console.log(index);
      this.lisaaTehdytListaan(index);
      this.tallenna(this.tehtavaLista);
      //jos haluaa poistaa valitut suoraan// this.poistaTehtava(index);
    },

    valitseAlaTehtava: function(index1, index2) {
      //console.log("ylä", this.tehtavaLista[index1]);
      //console.log("ala", this.tehtavaLista[index1].alaTehtavaLista[index2]);
      this.tehtavaLista[index1].alaTehtavaLista[index2].done = !this
        .tehtavaLista[index1].alaTehtavaLista[index2].done;
      this.tallenna(this.tehtavaLista);
    }
  },

  computed: {}
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  padding: 100px;
}

.eka {
  display: inline-block;
  width: 300px;
  float: left;
}
.toka {
  display: inline-block;
  float: left;
}
</style>
