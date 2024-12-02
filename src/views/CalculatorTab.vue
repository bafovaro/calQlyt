<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Calculator</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true" @click="collapseHistory">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Calculator</ion-title>
        </ion-toolbar>
      </ion-header>

      <div class="calculator" @click.stop>
        <div class="display" :class="{ expanded: showHistory }" @click="toggleHistory">{{ display }}</div>
        <div class="buttons">
          <ion-button @click="clear" expand="full" color="danger">C</ion-button>
          <div></div>
          <div></div>
          <div></div>
          <ion-button @click="append('7')" expand="full">7</ion-button>
          <ion-button @click="append('8')" expand="full">8</ion-button>
          <ion-button @click="append('9')" expand="full">9</ion-button>
          <ion-button @click="append('/')" expand="full">/</ion-button>
          <ion-button @click="append('4')" expand="full">4</ion-button>
          <ion-button @click="append('5')" expand="full">5</ion-button>
          <ion-button @click="append('6')" expand="full">6</ion-button>
          <ion-button @click="append('*')" expand="full">*</ion-button>
          <ion-button @click="append('1')" expand="full">1</ion-button>
          <ion-button @click="append('2')" expand="full">2</ion-button>
          <ion-button @click="append('3')" expand="full">3</ion-button>
          <ion-button @click="append('-')" expand="full">-</ion-button>
          <ion-button @click="backspace" expand="full" class="backspace"><<</ion-button>
          <ion-button @click="append('<')" expand="full" class="backspace">&lt;</ion-button>
          <ion-button @click="append('0')" expand="full">0</ion-button>
          <ion-button @click="append('+')" expand="full">+</ion-button>
          <div></div>
          <div></div>
          <ion-button @click="append('.')" expand="full">.</ion-button>
          <ion-button @click="calculate" expand="full" color="success">=</ion-button>
        </div>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonButton } from '@ionic/vue';

const display = ref('');
const history = ref<string[]>([]);
const showHistory = ref(false);

const append = (char: string) => {
  display.value += char;
};

const clear = () => {
  display.value = '';
};

const backspace = () => {
  display.value = display.value.slice(0, -1);
};

const calculate = () => {
  try {
    const result = eval(display.value).toString();
    history.value.push(`${display.value} = ${result}`);
    display.value = result;
  } catch (e) {
    display.value = 'Error';
  }
};

const toggleHistory = () => {
  showHistory.value = !showHistory.value;
  if (showHistory.value) {
    display.value = history.value.join('\n');
  } else {
    display.value = history.value.length > 0 ? history.value[history.value.length - 1].split(' = ')[0] : '';
  }
};

const collapseHistory = () => {
  if (showHistory.value) {
    showHistory.value = false;
    display.value = history.value.length > 0 ? history.value[history.value.length - 1].split(' = ')[0] : '';
  }
};
</script>

<style scoped>
.calculator {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
}

.display {
  width: 100%;
  height: 50px;
  background: var(--ion-color-light);
  color: var(--ion-color-dark);
  text-align: right;
  padding: 10px;
  font-size: 24px;
  margin-bottom: 10px;
  border: 1px solid var(--ion-color-medium);
  border-radius: 5px;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}

.display.expanded {
  height: 50vh;
  overflow-y: auto;
  white-space: pre-wrap;
}

.buttons {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 10px;
  width: 100%;
}

ion-button {
  height: 60px;
  font-size: 24px;
}

.backspace {
  --background: var(--ion-color-warning);
  --background-activated: var(--ion-color-warning-shade);
  --background-focused: var(--ion-color-warning-tint);
}
</style>
