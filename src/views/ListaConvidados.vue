<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar color="primary">
        <ion-buttons slot="start">
          <ion-back-button></ion-back-button>
        </ion-buttons>
        <ion-title>Convidados</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="false" class="ion-padding">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">AppCheckIn</ion-title>
        </ion-toolbar>
      </ion-header>

      <ion-searchbar :debounce="500" @ionInput="search($event)"></ion-searchbar>

      <ion-list :inset="true">
        <ion-item :button="false" v-for="guest in guests">
            <ion-icon :color=" ( ( guest.check === undefined || guest.check === null ) ? 0: guest.check ) ? 'success' : 'warning'" 
                      :icon=" ( ( guest.check === undefined || guest.check === null ) ? 0: guest.check) ? checkmarkCircleOutline : closeCircleOutline"
                      slot="start"></ion-icon>
            <ion-label>
              <strong>{{ guest.name }}</strong><br />
              <ion-text>{{ guest.secname }}</ion-text><br />
            </ion-label>
            <ion-note slot="end">{{ guest.hour }}</ion-note>
        </ion-item>
      </ion-list>
    </ion-content>
  </ion-page>
</template>

<script lang="ts">
  import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonBackButton, IonButtons,
           IonItem,
           IonList,
           IonLabel,
           IonIcon,
           IonText,
           IonSearchbar,
           IonNote } from '@ionic/vue';
  import { checkmarkCircleOutline,
           closeCircleOutline
         } from 'ionicons/icons';
  import { defineComponent, ref } from 'vue';

  export default defineComponent({
    components: { IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonBackButton, IonButtons,
                  IonItem,
                  IonList,
                  IonSearchbar, IonLabel,
                  IonIcon,
                  IonText, 
                  IonNote },
    setup() {
      let data = null;
      data = JSON.parse( localStorage.getItem('guests') || '{}' );
      const guests = ref(data);

      //console.log( JSON.stringify( data ) );

      return { data, guests,
               checkmarkCircleOutline, closeCircleOutline };
    },
    methods: {
      search(event) {
        const d = null;
        const query = event.target.value.toLowerCase();
        this.guests = this.data.filter((d) => d.name.toLowerCase().indexOf(query) > -1);
      },
    },
  });
</script>

<style scoped>

ion-content {
  /*--background: #f4f5f8;*/
}

</style>
