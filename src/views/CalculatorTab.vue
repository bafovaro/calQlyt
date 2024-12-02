<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title class="centered-title">calQlyt</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title class="centered-title">calQlyt</ion-title>
        </ion-toolbar>
      </ion-header>

      <div class="calculator" @click.self="collapseHistory">
        <div class="display" :class="{ expanded: showHistory }" @click.stop="toggleHistory">{{ display }}</div>
        <div class="buttons">
          <ion-button @click.stop="clearAll" expand="full" color="danger">C</ion-button>
          <div></div>
          <div></div>
          <div></div>
          <ion-button @click.stop="append('7')" expand="full" :disabled="showHistory">7</ion-button>
          <ion-button @click.stop="append('8')" expand="full" :disabled="showHistory">8</ion-button>
          <ion-button @click.stop="append('9')" expand="full" :disabled="showHistory">9</ion-button>
          <ion-button @click.stop="append('/')" expand="full" :disabled="showHistory">/</ion-button>
          <ion-button @click.stop="append('4')" expand="full" :disabled="showHistory">4</ion-button>
          <ion-button @click.stop="append('5')" expand="full" :disabled="showHistory">5</ion-button>
          <ion-button @click.stop="append('6')" expand="full" :disabled="showHistory">6</ion-button>
          <ion-button @click.stop="append('*')" expand="full" :disabled="showHistory">*</ion-button>
          <ion-button @click.stop="append('1')" expand="full" :disabled="showHistory">1</ion-button>
          <ion-button @click.stop="append('2')" expand="full" :disabled="showHistory">2</ion-button>
          <ion-button @click.stop="append('3')" expand="full" :disabled="showHistory">3</ion-button>
          <ion-button @click.stop="append('-')" expand="full" :disabled="showHistory">-</ion-button>
          <ion-button @click.stop="stepBack" expand="full" class="backspace"><<</ion-button>
          <ion-button @click.stop="backspace" expand="full" class="backspace" :disabled="showHistory">&lt;</ion-button>
          <ion-button @click.stop="append('0')" expand="full" :disabled="showHistory">0</ion-button>
          <ion-button @click.stop="append('+')" expand="full" :disabled="showHistory">+</ion-button>
          <div></div>
          <div></div>
          <ion-button @click.stop="append('.')" expand="full" :disabled="showHistory">.</ion-button>
          <ion-button @click.stop="calculateExpression" expand="full" color="success" :disabled="showHistory">=</ion-button>
        </div>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup>
import { ref } from 'vue';
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonButton } from '@ionic/vue';

const showHistory = ref(false);
const display = ref('');
const history = ref([]);
const currentInput = ref('');
const lastDisplay = ref('');
const lastSteppedBack = ref(false);

const append = (char) => {
  display.value += char;
  currentInput.value += char;
};

const clearAll = () => {
  display.value = '';
  history.value = [];
  currentInput.value = '';
  lastDisplay.value = '';
  lastSteppedBack.value = false;
};

const backspace = () => {
  if (display.value === 'Error' || display.value === '') {
    return;
  }
  display.value = display.value.slice(0, -1);
  currentInput.value = currentInput.value.slice(0, -1);
};

const collapseHistory = () => {
  console.log('Collapsing history...');
  console.log('Before collapse - showHistory:', showHistory.value, 'display:', display.value, 'lastDisplay:', lastDisplay.value, 'history:', history.value);
  if (showHistory.value) {
    showHistory.value = false;
    if (history.value.length > 0) {
      const lastEntry = history.value[history.value.length - 1];
      const [equation, answer] = lastEntry.split(' = ');
      display.value = answer.trim(); // Show the answer of the latest history equation when collapsing
    } else {
      display.value = '';
    }
    lastDisplay.value = display.value; // Update lastDisplay to the latest value
  }
  console.log('After collapse - showHistory:', showHistory.value, 'display:', display.value, 'lastDisplay:', lastDisplay.value, 'history:', history.value);
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

const stepBack = () => {
  console.log('Stepping back...');
  console.log('Before step back - showHistory:', showHistory.value, 'display:', display.value, 'lastDisplay:', lastDisplay.value, 'history:', history.value);
  if (currentInput.value) {
    currentInput.value = '';
    display.value = '';
  } else if (history.value.length > 0) {
    if (showHistory.value) {
      // Remove the last entry from the history
      history.value.pop();
      display.value = history.value.join('\n');
      if (history.value.length > 0) {
        const lastEntry = history.value[history.value.length - 1];
        const [equation, answer] = lastEntry.split(' = ');
        lastDisplay.value = answer.trim(); // Update lastDisplay to the answer of the latest history entry
      } else {
        lastDisplay.value = '';
      }
    } else {
      const lastHistoryEntry = history.value.pop();
      if (lastHistoryEntry) {
        const [equation, answer] = lastHistoryEntry.split(' = ');
        display.value = equation.trim(); // Show the equation when stepping back
        lastDisplay.value = answer.trim(); // Update lastDisplay to the answer of the latest history entry
      } else {
        display.value = '';
        lastDisplay.value = '';
      }
    }
    lastSteppedBack.value = true;
  }
  console.log('After step back - showHistory:', showHistory.value, 'display:', display.value, 'lastDisplay:', lastDisplay.value, 'history:', history.value);
};

const calculateExpression = () => {
  if (lastSteppedBack.value && history.value.length > 0) {
    // Update the latest entry in the history
    const lastHistoryEntry = history.value[history.value.length - 1];
    const equation = display.value; // Use the current display value as the equation
    const answer = eval(equation).toString();
    history.value[history.value.length - 1] = `${equation} = ${answer}`;
    display.value = answer;
  } else {
    calculate(display.value);
  }
  currentInput.value = ''; // Clear current input after calculation
  lastSteppedBack.value = false;
};

const calculate = (equation) => {
  if (typeof equation !== 'string') {
    console.error('Invalid equation:', equation);
    return;
  }

  console.log('Calculating...');
  console.log('Before calculate - display:', display.value, 'history:', history.value);
  try {
    const answer = eval(equation).toString();
    history.value.push(`${equation} = ${answer}`);
    display.value = answer;
    currentInput.value = '';
    if (showHistory.value) {
      display.value = history.value.join('\n');
    }
    lastDisplay.value = display.value; // Update lastDisplay to the latest value after calculation
  } catch (e) {
    display.value = 'Error';
  }
  console.log('After calculate - display:', display.value, 'history:', history.value);
};
</script>

<style scoped>
.centered-title {
  text-align: center;
  width: 100%;
}

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