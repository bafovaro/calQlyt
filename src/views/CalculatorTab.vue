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
          <ion-button @click="clearAll" expand="full" color="danger">C</ion-button>
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
          <ion-button @click="stepBack" expand="full" class="backspace"><<</ion-button>
          <ion-button @click="backspace" expand="full" class="backspace">&lt;</ion-button>
          <ion-button @click="append('0')" expand="full">0</ion-button>
          <ion-button @click="append('+')" expand="full">+</ion-button>
          <div></div>
          <div></div>
          <ion-button @click="append('.')" expand="full">.</ion-button>
          <ion-button @click="calculateExpression" expand="full" color="success">=</ion-button>
        </div>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup>
import { ref } from 'vue';

const display = ref('');
const history = ref([]);
const currentInput = ref('');
const viewingHistory = ref(false);
const showHistory = ref(false);
const lastDisplay = ref('');

const append = (char) => {
  if (display.value === 'Error') {
    display.value = '';
  }
  display.value += char;
  currentInput.value += char;
};

const clearAll = () => {
  display.value = '';
  history.value = [];
  currentInput.value = '';
  viewingHistory.value = false;
  lastDisplay.value = '';
};

const backspace = () => {
  if (display.value === 'Error' || display.value === '') {
    return;
  }
  display.value = display.value.slice(0, -1);
  currentInput.value = currentInput.value.slice(0, -1);
};

const stepBack = () => {
  console.log('Stepping back...');
  console.log('Before step back - showHistory:', showHistory.value, 'display:', display.value, 'lastDisplay:', lastDisplay.value, 'history:', history.value);
  if (currentInput.value) {
    currentInput.value = '';
    display.value = '';
  } else if (history.value.length > 0) {
    history.value.pop();
    if (showHistory.value) {
      display.value = history.value.join('\n');
    } else {
      display.value = history.value.length > 0 ? history.value[history.value.length - 1].split(' = ')[0].trim() : '';
    }
    lastDisplay.value = display.value; // Update lastDisplay to the latest value after stepping back
  }
  console.log('After step back - showHistory:', showHistory.value, 'display:', display.value, 'lastDisplay:', lastDisplay.value, 'history:', history.value);
};

const calculate = (expression) => {
  if (typeof expression !== 'string') {
    console.error('Invalid expression:', expression);
    return;
  }

  console.log('Calculating...');
  console.log('Before calculate - display:', display.value, 'history:', history.value);
  try {
    const result = eval(expression).toString();
    history.value.push(`${expression} = ${result}`);
    display.value = result;
    currentInput.value = '';
    viewingHistory.value = false;
    if (showHistory.value) {
      display.value = history.value.join('\n');
    }
    lastDisplay.value = display.value; // Update lastDisplay to the latest value after calculation
  } catch (e) {
    display.value = 'Error';
  }
  console.log('After calculate - display:', display.value, 'history:', history.value);
};

const calculateExpression = () => {
  calculate(display.value);
};

const toggleHistory = () => {
  console.log('Toggling history...');
  console.log('Before toggle - showHistory:', showHistory.value, 'display:', display.value, 'lastDisplay:', lastDisplay.value);
  if (showHistory.value) {
    display.value = lastDisplay.value;
  } else {
    lastDisplay.value = display.value;
    display.value = history.value.join('\n');
  }
  showHistory.value = !showHistory.value;
  console.log('After toggle - showHistory:', showHistory.value, 'display:', display.value, 'lastDisplay:', lastDisplay.value);
};

const collapseHistory = () => {
  console.log('Collapsing history...');
  console.log('Before collapse - showHistory:', showHistory.value, 'display:', display.value, 'lastDisplay:', lastDisplay.value, 'history:', history.value);
  if (showHistory.value) {
    showHistory.value = false;
    if (history.value.length > 0) {
      const lastEntry = history.value[history.value.length - 1];
      display.value = lastEntry.split(' = ')[1].trim(); // Show the result when collapsing
    } else {
      display.value = '';
    }
    lastDisplay.value = display.value; // Update lastDisplay to the latest value
  }
  console.log('After collapse - showHistory:', showHistory.value, 'display:', display.value, 'lastDisplay:', lastDisplay.value, 'history:', history.value);
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