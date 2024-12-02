<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Calculator</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Calculator</ion-title>
        </ion-toolbar>
      </ion-header>

      <div class="calculator" @click="collapseHistory">
        <div class="display" :class="{ expanded: showHistory }" @click.stop="toggleHistory">{{ display }}</div>
        <div class="buttons">
          <ion-button @click.stop="clearAll" expand="full" color="danger">C</ion-button>
          <div></div>
          <div></div>
          <div></div>
          <ion-button @click.stop="append('7')" expand="full">7</ion-button>
          <ion-button @click.stop="append('8')" expand="full">8</ion-button>
          <ion-button @click.stop="append('9')" expand="full">9</ion-button>
          <ion-button @click.stop="append('/')" expand="full">/</ion-button>
          <ion-button @click.stop="append('4')" expand="full">4</ion-button>
          <ion-button @click.stop="append('5')" expand="full">5</ion-button>
          <ion-button @click.stop="append('6')" expand="full">6</ion-button>
          <ion-button @click.stop="append('*')" expand="full">*</ion-button>
          <ion-button @click.stop="append('1')" expand="full">1</ion-button>
          <ion-button @click.stop="append('2')" expand="full">2</ion-button>
          <ion-button @click.stop="append('3')" expand="full">3</ion-button>
          <ion-button @click.stop="append('-')" expand="full">-</ion-button>
          <ion-button @click.stop="stepBack" expand="full" class="backspace"><<</ion-button>
          <ion-button @click.stop="backspace" expand="full" class="backspace">&lt;</ion-button>
          <ion-button @click.stop="append('0')" expand="full">0</ion-button>
          <ion-button @click.stop="append('+')" expand="full">+</ion-button>
          <div></div>
          <div></div>
          <ion-button @click.stop="append('.')" expand="full">.</ion-button>
          <ion-button @click.stop="calculateExpression" expand="full" color="success">=</ion-button>
        </div>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup>
import { ref } from 'vue';
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonButton } from '@ionic/vue';

const display = ref('');
const history = ref([]);
const currentInput = ref('');
const viewingHistory = ref(false);
const showHistory = ref(false);
const lastDisplay = ref('');
const isSteppingBack = ref(false);

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
  isSteppingBack.value = true;

  if (currentInput.value) {
    currentInput.value = '';
    display.value = '';
  } else if (history.value.length > 0) {
    if (showHistory.value) {
      // Remove the last entry from the history
      history.value.pop();
      display.value = history.value.join('\n');
    } else {
      if (display.value === history.value[history.value.length - 1]) {
        // Remove the latest history item and move to the next
        history.value.pop();
      }
      if (history.value.length > 0) {
        display.value = history.value[history.value.length - 1];
      } else {
        display.value = '';
      }
      // Remove the '=' and everything after it in the display
      const equalIndex = display.value.indexOf('=');
      if (equalIndex !== -1) {
        display.value = display.value.substring(0, equalIndex).trim();
      }
      lastDisplay.value = display.value; // Update lastDisplay to the latest value after stepping back
    }
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
    if (isSteppingBack.value && history.value.length > 0) {
      // Update the latest history item
      history.value[history.value.length - 1] = `${expression} = ${result}`;
    } else {
      // Add a new history item
      history.value.push(`${expression} = ${result}`);
    }
    display.value = result;
    currentInput.value = '';
    viewingHistory.value = false;
    isSteppingBack.value = false; // Reset the flag after calculation
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
  currentInput.value = ''; // Clear current input after calculation
};

const toggleHistory = () => {
  console.log('Toggling history...');
  console.log('Before toggle - showHistory:', showHistory.value, 'display:', display.value, 'lastDisplay:', lastDisplay.value);
  if (showHistory.value) {
    collapseHistory(); // Collapse history if it is currently expanded
  } else {
    lastDisplay.value = display.value;
    display.value = history.value.join('\n');
    showHistory.value = true;
  }
  console.log('After toggle - showHistory:', showHistory.value, 'display:', display.value, 'lastDisplay:', lastDisplay.value);
};

const collapseHistory = () => {
  console.log('Collapsing history...');
  console.log('Before collapse - showHistory:', showHistory.value, 'display:', display.value, 'lastDisplay:', lastDisplay.value, 'history:', history.value);
  if (showHistory.value) {
    showHistory.value = false;
    if (history.value.length > 0) {
      if (isSteppingBack.value) {
        // Maintain the calculation when collapsing after stepping back
        display.value = history.value[history.value.length - 1].split(' = ')[0].trim();
      } else {
        // Show the result when collapsing after calculation
        display.value = history.value[history.value.length - 1].split(' = ')[1].trim();
      }
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