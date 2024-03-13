<template>
    <ion-page>
        <ion-header :translucent="true">
            <ion-toolbar color="primary">
            <ion-buttons slot="start">
                <ion-back-button></ion-back-button>
            </ion-buttons>
            <ion-title>Sobre</ion-title>
            </ion-toolbar>
        </ion-header>

        <ion-content :fullscreen="false" class="ion-padding">
            <ion-header collapse="condense">
                <ion-toolbar>
                    <ion-title size="large">AppCheckIn</ion-title>
                </ion-toolbar>
            </ion-header>

            <div id="container">

                <ion-card color="light">
                    <ion-card-header>
                    <ion-card-title>{{  appversion.name }}</ion-card-title>
                    <ion-card-subtitle>Versão: {{ appversion.version }}</ion-card-subtitle>
                    </ion-card-header>

                    <ion-card-content>
                    {{ appversion.detalhes }}
                    </ion-card-content>
                </ion-card>
                
            </div>
        </ion-content>
  </ion-page>
</template>

<script lang="ts">
  import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonButtons, IonBackButton,
           IonCard, IonCardContent, IonCardHeader, IonCardSubtitle, IonCardTitle } from '@ionic/vue';
  import { defineComponent, ref } from 'vue';

  export default defineComponent({
    components: { IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonButtons, IonBackButton,
                  IonCard, IonCardContent, IonCardHeader, IonCardSubtitle, IonCardTitle },
    setup() {
      let dbversion = null;
      dbversion = JSON.parse( localStorage.getItem("appversion") || '{}' );
      const appversion = ref(dbversion);

      //console.log( JSON.stringify( appversion ) );

      return { appversion };
    },
    //methods: {
    //},
  });
</script>

<style scoped>

ion-content {
  /*--background: #f4f5f8; */
}

#container {
  position: fixed;
  left: 0;
  right: 0;
  top: 8%;
}

#container strong {
  font-size: 20px;
  line-height: 26px;
}

#container a {
  text-decoration: none;
}

</style>
