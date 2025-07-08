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
        <v-textarea
          :model-value="equationDisplay"
          label="Cards Added"
          variant="outlined"
          readonly
          class="equation-field"
          hide-details
          auto-grow
          rows="1"
          no-resize
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
        <div
          @click="addCard('A', 1, $event)"
          class="card-btn ace-btn"
        >
          <div class="card-content">
            <div class="card-value">A</div>
            <div class="card-suit">♠</div>
          </div>
        </div>

        <!-- Number Cards 2-9 -->
        <div
          v-for="num in [2, 3, 4, 5, 6, 7, 8, 9]"
          :key="num"
          @click="addCard(num.toString(), num, $event)"
          class="card-btn number-btn"
        >
          <div class="card-content">
            <div class="card-value">{{ num }}</div>
            <div class="card-suit">♦</div>
          </div>
        </div>

        <!-- Face Card -->
        <div
          @click="addCard('K', 10, $event)"
          class="card-btn face-btn"
        >
          <div class="card-content">
            <div class="card-value">K</div>
            <div class="card-suit">♥</div>
          </div>
        </div>

        <!-- Wild Card -->
        <div
          @click="addCard('W', 50, $event)"
          class="card-btn wild-btn"
        >
          <div class="card-content">
            <div class="card-value">W</div>
            <div class="card-suit">★</div>
          </div>
        </div>
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
const addCard = (display, value, event) => {
  createRipple(event);
  cards.value.push({
    display,
    value,
    numericValue: value
  });
};

const createRipple = (event) => {
  const button = event.currentTarget;
  const rect = button.getBoundingClientRect();
  const size = Math.max(rect.width, rect.height);
  const x = event.clientX - rect.left - size / 2;
  const y = event.clientY - rect.top - size / 2;
  
  const ripple = document.createElement('span');
  ripple.className = 'ripple';
  ripple.style.width = ripple.style.height = size + 'px';
  ripple.style.left = x + 'px';
  ripple.style.top = y + 'px';
  
  button.appendChild(ripple);
  
  // Remove ripple after animation
  setTimeout(() => {
    if (ripple.parentNode) {
      ripple.parentNode.removeChild(ripple);
    }
  }, 600);
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
  height: 90px;
  width: 100%;
  border: 2px solid #ddd;
  border-radius: 8px;
  background: white;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  transition: all 0.2s ease;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  user-select: none;
  position: relative;
  overflow: hidden;
}

.card-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0,0,0,0.15);
}

.card-btn:active {
  transform: translateY(0px);
}
.card-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100%;
  color: #333;
  position: relative;
  z-index: 1;
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

/* Ripple Effect */
.ripple {
  position: absolute;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.6);
  transform: scale(0);
  animation: ripple-animation 0.6s linear;
  pointer-events: none;
  z-index: 0;
}

@keyframes ripple-animation {
  to {
    transform: scale(4);
    opacity: 0;
  }
}
/* Card-specific styling */
.ace-btn .card-content {
  color: #000;
}

.ace-btn .card-suit {
  color: #000;
}

.ace-btn .ripple {
  background: rgba(0, 0, 0, 0.2);
}
.number-btn .card-content {
  color: #d32f2f;
}

.number-btn .card-suit {
  color: #d32f2f;
}

.number-btn .ripple {
  background: rgba(211, 47, 47, 0.3);
}
.face-btn .card-content {
  color: #d32f2f;
}

.face-btn .card-suit {
  color: #d32f2f;
}

.face-btn .ripple {
  background: rgba(211, 47, 47, 0.3);
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

.wild-btn .ripple {
  background: rgba(255, 255, 255, 0.4);
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
    height: 75px;
  }
  
  .card-value {
    font-size: 16px;
  }
  
  .card-suit {
    font-size: 14px;
  }
}
</style>