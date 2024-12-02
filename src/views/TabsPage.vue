<template>
  <ion-page>
    <ion-tabs>
      <ion-router-outlet></ion-router-outlet>
      <ion-tab-bar slot="bottom">
        <ion-tab-button tab="CalculatorTab" href="/tabs/CalculatorTab">
          <ion-icon aria-hidden="true" :icon="calculator" />
        </ion-tab-button>

        <!-- Conditionally render the HistoryTab button -->
        <ion-tab-button v-if="showHistoryTab" tab="HistoryTab" href="/tabs/HistoryTab">
          <ion-icon aria-hidden="true" :icon="time" />
        </ion-tab-button>
      </ion-tab-bar>
    </ion-tabs>
  </ion-page>
</template>

<script setup lang="ts">
import { IonTabBar, IonTabButton, IonTabs, IonLabel, IonIcon, IonPage, IonRouterOutlet } from '@ionic/vue';
import { calculator, time } from 'ionicons/icons';
import { ref } from 'vue';

// Define a reactive variable to control the visibility of the HistoryTab
const showHistoryTab = ref(false);
</script>
