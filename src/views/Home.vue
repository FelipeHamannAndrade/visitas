<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title class="centralizado">Visitas Comerciais</ion-title>
      </ion-toolbar>
    </ion-header>

      <ion-grid>
        <h1 class="centralizado">Visitas</h1>
        <ion-row>
          <ion-col>
            <ion-label>Estabelecimento: </ion-label>
            <ion-input type="text" id="estabelecimento" placeholder="Digite o nome do estabelecimento" v-model="estabelecimento"></ion-input>
            <ion-label>Contato: </ion-label>
            <ion-input type="text" id="contato" placeholder="Digite o nome do contato... " v-model="contato"></ion-input>
          </ion-col>
        </ion-row>
      </ion-grid>

      <ion-content>
      <div class='container'>
          <h4>Coordenadas</h4>
          <h10>{{  latitude  }}, {{  longitude  }}</h10>
          <ion-button expand="block" @click="gravar()">Gravar Visita</ion-button>
          <ion-button expand="block" @click="limparLista()">Apagar Lista</ion-button>
          <h3>{{  cidade  }}</h3>
      </div>

      <ion-grid>
        <ion-row>
          <ion-col>
            <ul v-for="(cidade, index) in list" v-bind:key="index">
              <li>{{  cidade  }}</li>
              
            </ul>
          </ion-col>
        </ion-row>
      </ion-grid>

    </ion-content>
  </ion-page>
</template>

<script>
import { IonContent, IonInput, IonLabel, IonHeader, IonPage, IonTitle, IonToolbar, IonButton, IonGrid, IonRow, IonCol } from '@ionic/vue';
import { defineComponent } from 'vue';
import { Geolocation } from '@capacitor/geolocation';
import { Http } from '@capacitor-community/http';
import { Storage } from '@ionic/storage'

export default defineComponent({
  name: 'Home',
  components: {
    IonContent,
    IonHeader,
    IonPage,
    IonTitle,
    IonToolbar,
    IonButton,
    IonGrid, 
    IonRow,
    IonCol,
    IonInput,
    IonLabel
  },
  data() {
    return {
      localStorage: new Storage(),
      latitude: 0,
      longitude: 0,
      cidade: '',
      estabelecimento: '',
      contato: '',
      mensagem: '',
      list: []
    }
  },
  ionViewWillEnter(){
    this.printCurrentPosition();

  },
  created: async function () {
    await this.localStorage.create();

  },
  mounted: async function() {
    const list = await this.localStorage.get('list');
      if (JSON.parse(list) == null) {
        this.list = [];
      }
      else
        this.list = JSON.parse(list);
  },   

  methods: {
    printCurrentPosition: async function() {
      const coordinates = await Geolocation.getCurrentPosition();
      console.log('Current position: ', coordinates);


        this.latitude = coordinates.coords.latitude;
        this.longitude = coordinates.coords.longitude; 
    },
    gravar: async function(){
        const mensagem = await 'Cliente: ' + this.estabelecimento + ', Contato: ' + this.contato;
        const ACCESS_KEY = 'c1088209a10b5a9fd7e856834d06c278';
        //let url = 'http://api.positionstack.com/v1/reverso?access_key=' + ACCESS_KEY + '&query=' + this.latitude + ',' + this.longitude;


        const options = {
          url: `http://api.positionstack.com/v1/reverse?access_key=${ACCESS_KEY}&query=${this.latitude},${this.longitude}&limit=1`
        };

        const response = await Http.get(options);
        console.log('Resposta recebida: ', response);

        //this.cidade =`${mensagem}, Local: ${response.data.data[0].locality},${response.data.data[0].county},${response.data.data[0].region_code}` 
        this.cidade = (mensagem) + ', Endereço: ' + response.data.data[0].label + ', ' + response.data.data[0].locality + ', ' + response.data.data[0].county + ', ' + response.data.data[0].region_code;
               
          this.list.push(this.cidade);
          await this.localStorage.set('list', JSON.stringify(this.list));
          console.log(this.list);
           this.contato = '';
           this.estabelecimento = '';
    },
    limparLista: async function() {
        this.list = [];
        this.contato = '';
        this.estabelecimento = '';

    },
  }
  
});
</script>

<style scoped>
.container {
  text-align: center;
}
ion-input {
  border: 1px solid lightgray;
  border-radius: 5px;
}
.centralizado {
  text-align: center;
}

</style>