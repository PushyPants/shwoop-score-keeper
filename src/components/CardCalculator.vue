<template>
  <v-card class="card-calculator" elevation="4">
    <v-card-title class="d-flex justify-space-between align-center pb-2">
      <div class="d-flex align-center">
        <v-icon class="mr-2" color="primary">mdi-calculator</v-icon>
        Card Calculator
      </div>
      <v-btn
        @click="$emit('close')"
        icon="mdi-close"
        variant="text"
        size="small"
      />
    </v-card-title>
    
    <v-card-text>
      <!-- Display Fields -->
      <div class="calculator-display mb-4">
        <!-- Equation Display -->
        <v-text-field
          :model-value="equationDisplay"
          label="Cards Added"
          variant="outlined"
          readonly
          class="equation-field"
          hide-details
        />
        
        <!-- Total Display -->
        <v-text-field
          :model-value="total"
          label="Total"
          variant="outlined"
          readonly
          class="total-field mt-2"
          hide-details
        />
      </div>

      <!-- Card Buttons Grid -->
      <div class="card-buttons-grid mb-4">
        <!-- Ace -->
        <v-btn
          @click="addCard('A', 1)"
          class="card-btn ace-btn"
          size="large"
        >
          <div class="card-content">
            <div class="card-value">A</div>
            <div class="card-suit">♠</div>
          </div>
        </v-btn>

        <!-- Number Cards 2-9 -->
        <v-btn
          v-for="num in [2, 3, 4, 5, 6, 7, 8, 9]"
          :key="num"
          @click="addCard(num.toString(), num)"
          class="card-btn number-btn"
          size="large"
        >
          <div class="card-content">
            <div class="card-value">{{ num }}</div>
            <div class="card-suit">♦</div>
          </div>
        </v-btn>

        <!-- Face Card -->
        <v-btn
          @click="addCard('K', 10)"
          class="card-btn face-btn"
          size="large"
        >
          <div class="card-content">
            <div class="card-value">K</div>
            <div class="card-suit">♥</div>
          </div>
        </v-btn>

        <!-- Wild Card -->
        <v-btn
          @click="addCard('W', 50)"
          class="card-btn wild-btn"
          size="large"
        >
          <div class="card-content">
            <div class="card-value">W</div>
            <div class="card-suit">★</div>
          </div>
        </v-btn>
      </div>

      <!-- Action Buttons -->
      <div class="action-buttons">
        <v-btn
          @click="backspace"
          :disabled="cards.length === 0"
          color="warning"
          size="large"
          prepend-icon="mdi-backspace"
          class="flex-grow-1 mr-2"
        >
          Backspace
        </v-btn>
        
        <v-btn
          @click="done"
          :disabled="cards.length === 0"
          color="success"
          size="large"
          prepend-icon="mdi-check"
          class="flex-grow-1"
        >
          Done
        </v-btn>
      </div>
    </v-card-text>
  </v-card>
</template>

<script setup>
import { ref, computed } from 'vue';

// Props
const props = defineProps({
  userId: {
    type: [String, Number],
    required: true
  }
});

// Emits
const emit = defineEmits(['result', 'close']);

// Reactive data
const cards = ref([]);

// Computed properties
const equationDisplay = computed(() => {
  if (cards.value.length === 0) return '';
  return cards.value.map((card, index) => {
    return index === 0 ? card.value.toString() : `+ ${card.value}`;
  }).join(' ');
});

const total = computed(() => {
  return cards.value.reduce((sum, card) => sum + card.numericValue, 0);
});

// Methods
const addCard = (display, value) => {
  cards.value.push({
    display,
    value,
    numericValue: value
  });
};

const backspace = () => {
  if (cards.value.length > 0) {
    cards.value.pop();
  }
};

const done = () => {
  if (cards.value.length > 0) {
    // Emit result with userId and total
    emit('result', { [props.userId]: total.value });
    
    // Clear the calculator
    cards.value = [];
    
    // Emit close
    emit('close');
  }
};
</script>

<style scoped>
.card-calculator {
  max-width: 400px;
  margin: 0 auto;
}

.calculator-display {
  background: #f5f5f5;
  padding: 16px;
  border-radius: 8px;
}

.equation-field :deep(.v-field__input) {
  font-family: 'Courier New', monospace;
  font-size: 14px;
}

.total-field :deep(.v-field__input) {
  font-weight: bold;
  font-size: 18px;
  color: #1976d2;
}

.card-buttons-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 8px;
}

.card-btn {
  aspect-ratio: 2.5/8;
  border: 2px solid #ddd;
  border-radius: 8px;
  background: white;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  transition: all 0.2s ease;
}

.card-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0,0,0,0.15);
}

.card-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100%;
  color: #333;
}

.card-value {
  font-size: 18px;
  font-weight: bold;
  line-height: 1;
}

.card-suit {
  font-size: 16px;
  margin-top: 2px;
}

/* Card-specific styling */
.ace-btn .card-content {
  color: #000;
}

.ace-btn .card-suit {
  color: #000;
}

.number-btn .card-content {
  color: #d32f2f;
}

.number-btn .card-suit {
  color: #d32f2f;
}

.face-btn .card-content {
  color: #d32f2f;
}

.face-btn .card-suit {
  color: #d32f2f;
}

.wild-btn {
  background: linear-gradient(135deg, #9c27b0, #673ab7) !important;
  border-color: #9c27b0 !important;
}

.wild-btn .card-content {
  color: white;
}

.wild-btn .card-suit {
  color: #ffd700;
}

.action-buttons {
  display: flex;
  gap: 8px;
}

/* Mobile responsiveness */
@media (max-width: 480px) {
  .card-buttons-grid {
    grid-template-columns: repeat(3, 1fr);
  }
  
  .card-btn {
    aspect-ratio: 2.5/7;
  }
  
  .card-value {
    font-size: 16px;
  }
  
  .card-suit {
    font-size: 14px;
  }
}
</style>