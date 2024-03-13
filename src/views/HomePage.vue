<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar color="primary">
        <ion-title>AppChckIn</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true" class="ion-padding">
      
      <ion-fab slot="fixed" vertical="top" horizontal="end" :edge="true">
        <ion-fab-button>
          <ion-icon :icon="chevronDownCircle"></ion-icon>
        </ion-fab-button>
        <ion-fab-list side="bottom">
          
          <ion-fab-button id="btnListaPresenca" @click="()=>router.push('/lista-convidados')">
            <ion-icon :icon="fileTrayFullOutline"></ion-icon>
          </ion-fab-button>

          <ion-fab-button id="btnConfig">
            <ion-icon :icon="construct"></ion-icon>
          </ion-fab-button>

          <ion-fab-button id="btnSobre" @click="()=>router.push('/sobre')">
            <ion-icon :icon="informationCircleOutline"></ion-icon>
          </ion-fab-button>

        </ion-fab-list>
      </ion-fab>

      <div id="container">
        <ion-input label="Código de Barras" 
                   label-placement="floating" 
                   color="primary"
                   maxlength="10"
                   :clear-input="true"
                   :counter="true"></ion-input>
        <ion-button expand="block">Confirmar Entrada</ion-button>
      </div>
      <div id="status">
        <p>Status: <ion-label id="lblStatus" 
                              color="medium">{{ isSupported ? "Plugin On": "Plugin Off" }}</ion-label>
           &nbsp;/&nbsp;
           <ion-label color="medium">{{ barcodes }}</ion-label>&nbsp;-&nbsp;
           <ion-label color="medium">{{ rQRCode }}</ion-label>
        </p>
      </div>

      <ion-fab slot="fixed" vertical="bottom" horizontal="end">
        <ion-fab-button @click="scanSingleBarcode" :disabled="!isSupported">
          <ion-icon :icon="barcodeOutline"></ion-icon>
        </ion-fab-button>
      </ion-fab>
    </ion-content>
  </ion-page>
</template>

<script lang="ts">
  import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar,
           IonFab, IonFabButton, IonFabList, IonIcon,
           IonInput, IonButton, IonLabel, alertController } from '@ionic/vue';
  import {
    chevronDownCircle,
    chevronForwardCircle,
    chevronUpCircle,
    
    construct,
    fileTrayFullOutline,
    informationCircleOutline,
    barcodeOutline
  } from 'ionicons/icons';
  
  import { defineComponent, ref } from 'vue';
  import { BarcodeScanner,
           BarcodeFormat,
           /* LensFacing, */ } from "@capacitor-mlkit/barcode-scanning";

   /*
    * Instrucao usada para permitir usar as rotas
    * nos eventos onClicks dos botoes
    */
  import { useRouter } from 'vue-router'; 

  export default defineComponent({
    components: { IonContent, IonHeader, IonPage, IonTitle, IonToolbar,
                  IonFab, IonFabButton, IonFabList, IonIcon,
                  IonInput, IonButton, IonLabel, alertController, },    
    data() {
      return {
        //hasSupported: false,
        barcodes: []
      };
    },
    setup() {
      const appversion = {
                          'name':'AppChckIn', 
                          'detalhes':'App responsavel por auxiliar na validação e verificação de convidados.', 
                          'version':'2023.12.05-0940'
                        };
      const data = [ 
                      {'name':'Teste', 'secname':'01 Segundo Nome', 'check':1, 'hour':'19:34'},
                      {'name':'Convidado', 'secname':'02 Segundo Nome', 'check':0, 'hour':''}
                    ];

      const router = useRouter();
      const rQRCode = ref('');
      
      localStorage.removeItem("guests");
      localStorage.removeItem("appversion");

      localStorage.setItem("appversion", JSON.stringify(appversion));
      localStorage.setItem("guests", JSON.stringify(data) );

      const showMessage = async (caption, textMessage) => {
        const alert = await alertController.create({
          header: caption,
        //subHeader: 'A Sub Header Is Optional',
          message: textMessage,
          buttons: ['Ok'],
        });

        await alert.present();
      };

      /**
       * Metodos relacionados ao Plugin de Leitura 
       * de Codigo de Barras 
       **/
      const startScan = async () => {
        // The camera is visible behind the WebView, so that you can customize the UI in the WebView.
        // However, this means that you have to hide all elements that should not be visible.
        // You can find an example in our demo repository.
        // In this case we set a class `barcode-scanner-active`, which then contains certain CSS rules for our app.
        document.querySelector('body')?.classList.add('barcode-scanner-active');

        // Add the `barcodeScanned` listener
        const listener = await BarcodeScanner.addListener(
          'barcodeScanned',
          async result => {
            rQRCode.value = result.barcode
            console.log(result.barcode);
          },
        );

        // Start the barcode scanner
        await BarcodeScanner.startScan();
      };

      const stopScan = async () => {
        // Make all elements in the WebView visible again
        document.querySelector('body')?.classList.remove('barcode-scanner-active');

        // Remove all listeners
        await BarcodeScanner.removeAllListeners();

        // Stop the barcode scanner
        await BarcodeScanner.stopScan();
      };

      const scanSingleBarcode = async () => {
        return new Promise(async resolve => {
          document.querySelector('body')?.classList.add('barcode-scanner-active');

          const listener = await BarcodeScanner.addListener(
            'barcodeScanned',
            async result => {
              await listener.remove();
              document
                .querySelector('body')
                ?.classList.remove('barcode-scanner-active');
              await BarcodeScanner.stopScan();
              resolve(result.barcode);
            },
          );

          const bcode = await BarcodeScanner.startScan();
          if (bcode.hasContent){
            this.showMessage('Leitura', 'QR Code lido com Sucesso. QRCode: ' + bcode.content);
            this.barcodes.push(...bcode.content);
          }
        });
      };

      const scan = async () => {
        const { barcodes } = await BarcodeScanner.scan({
          formats: [BarcodeFormat.QrCode, 
                    BarcodeFormat.Ean8, 
                    BarcodeFormat.Ean13],
        });
        return barcodes;
      };

      const isSupported = async () => {
        const { supported } = await BarcodeScanner.isSupported();
        this.hasSupported = supported;
        //return supported;
      };

      const checkPermissions = async () => {
        const { camera } = await BarcodeScanner.checkPermissions();
        return camera;
      };

      const requestPermissions = async () => {
        const { camera } = await BarcodeScanner.requestPermissions();
        return camera;
      };
 
      /*******************************************/

      return { router,

               isSupported, scan, scanSingleBarcode, rQRCode,

               chevronDownCircle, chevronForwardCircle, chevronUpCircle,
               construct, fileTrayFullOutline, informationCircleOutline, barcodeOutline 
      };
    },
    //methods: {
    //}
  });
</script>

<style scoped>

ion-content {
  /*--background: #f4f5f8;
  --background: transparent;
  */
}

#container {
  position: absolute;
  left: 10%;
  right: 10%;
  top: 50%;
  transform: translateY(-50%);
}

#container strong {
  font-size: 20px;
  line-height: 26px;
}

#container p {
  font-size: 16px;
  line-height: 22px;
  
  color: #8c8c8c;
  
  margin: 0;
}

#container a {
  text-decoration: none;
}

#status {
  position: absolute;
  left: 10%;
  right: 10%;
  top: 70%;
  transform: translateY(-50%);
}

#status strong {
  font-size: 20px;
  line-height: 26px;
}

#status p {
  font-size: 16px;
  line-height: 22px;
  
  color: #8c8c8c;
  
  margin: 0;
}

ion-input {
  text-align: right;
}

</style>
