<template>
  <v-app>
    <!-- Top App Bar -->
    <v-app-bar color="primary" dark elevation="4">
      <v-toolbar-title class="d-flex align-center justify-center w-100">
        <v-icon class="mr-2" size="large">mdi-cards-playing</v-icon>
        Shwoop Score Tracker
      </v-toolbar-title>
    </v-app-bar>

    <v-main>
      <v-container fluid class="pa-4">
        <!-- Leader Section -->
        <v-row v-if="players.length > 0" justify="center">
          <v-col cols="12">
            <v-card class="leader-card" elevation="4">
              <v-card-text class="text-center py-4">
                <v-icon class="crown-icon mb-2" size="40" color="amber">mdi-crown</v-icon>
                <div class="text-h6 font-weight-bold">
                  {{ leaderText }}
                </div>
                <div class="text-h4 font-weight-bold primary--text">
                  {{ leaderScore }} points
                </div>
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>

        <!-- Add Player Button -->
        <v-row justify="center">
          <v-col cols="12">
            <v-btn
              @click="showAddPlayerDialog = true"
              color="success"
              size="large"
              block
              prepend-icon="mdi-account-plus"
            >
              Add Player
            </v-btn>
          </v-col>
        </v-row>

        <!-- Players List -->
        <v-row v-if="players.length > 0">
          <v-col cols="12">
            <v-list class="players-list">
              <div
                v-for="(player, index) in players"
                :key="player.id"
                class="swipe-item-container"
              >
                <!-- Background Actions -->
                <div class="swipe-actions">
                  <!-- Left action (Clear Score) -->
                  <div class="swipe-action swipe-action-left align-baseline">
                    <div class="d-flex flex-column justify-center align-center ml-7">
                      <v-icon color="white" size="24">mdi-refresh</v-icon>
                      <span class="action-text">Clear</span>
                    </div>
                  </div>
                  <!-- Right action (Delete) -->
                  <div class="swipe-action swipe-action-right align-end">
                    <div class="d-flex flex-column justify-center align-center mr-7">
                      <v-icon color="white" size="24">mdi-delete</v-icon>
                      <span class="action-text">Delete</span>
                    </div>
                  </div>
                </div>

                <!-- Main List Item -->
                <v-list-item
                  class="swipe-item"
                  :class="{ 'swiping': swipeStates[player.id]?.swiping }"
                  :style="{ transform: `translateX(${swipeStates[player.id]?.translateX || 0}px)` }"
                  @touchstart="handleTouchStart($event, player.id)"
                  @touchmove="handleTouchMove($event, player.id)"
                  @touchend="handleTouchEnd($event, player.id, index)"
                  @mousedown="handleMouseDown($event, player.id)"
                  @mousemove="handleMouseMove($event, player.id)"
                  @mouseup="handleMouseEnd($event, player.id, index)"
                  @mouseleave="handleMouseEnd($event, player.id, index)"
                >
                  <template v-slot:prepend>
                    <v-avatar color="primary" size="40">
                      <v-icon color="white">mdi-account</v-icon>
                    </v-avatar>
                  </template>

                  <v-list-item-title>{{ player.name }}</v-list-item-title>
                  <v-list-item-subtitle>{{ player.score }} points</v-list-item-subtitle>

                  <template v-slot:append>
                    <div class="actions-section">
                      <!-- Score Input Group -->
                      <div class="score-input-group mr-2">
                        <v-text-field
                          v-model.number="scoreInputs[player.id]"
                          @keyup.enter="addScore(player.id)"
                          min="1"
                          variant="outlined"
                          density="compact"
                          label="Add points"
                          hide-details
                          class="score-input"
                          :inputmode="'numeric'"
                          :pattern="'[0-9]*'"
                          type="text"
                        />
                        <v-btn
                          @click="addScore(player.id)"
                          :disabled="!isValidScore(scoreInputs[player.id])"
                          color="primary"
                          size="small"
                          icon="mdi-plus"
                          class="plus-btn"
                        />
                      </div>
                    </div>
                  </template>
                </v-list-item>
              </div>
            </v-list>
          </v-col>
        </v-row>

        <!-- Control Buttons -->
        <v-row v-if="players.length > 0" justify="center" class="mt-2">
          <v-col cols="12">
            <v-row dense>
              <v-col cols="12" sm="6">
                <v-btn
                  @click="showClearScoresDialog = true"
                  color="warning"
                  block
                  prepend-icon="mdi-refresh"
                >
                  Clear All Scores
                </v-btn>
              </v-col>
              <v-col cols="12" sm="6">
                <v-btn
                  @click="showClearAllDialog = true"
                  color="error"
                  block
                  prepend-icon="mdi-delete-sweep"
                >
                  Clear All Players
                </v-btn>
              </v-col>
            </v-row>
          </v-col>
        </v-row>

        <!-- Empty State -->
        <v-row v-if="players.length === 0" justify="center" class="mt-8">
          <v-col cols="12" class="text-center">
            <v-icon size="80" color="grey-lighten-1" class="mb-4">mdi-account-group</v-icon>
            <div class="text-h6 text-medium-emphasis">No players yet</div>
            <div class="text-body-2 text-medium-emphasis">Add your first player to get started!</div>
          </v-col>
        </v-row>
      </v-container>
    </v-main>

    <!-- Add Player Dialog -->
    <v-dialog v-model="showAddPlayerDialog" max-width="400">
      <v-card>
        <v-card-title class="text-h6">Add New Player</v-card-title>
        <v-card-text>
          <v-text-field
            v-model="newPlayerName"
            label="Player Name"
            variant="outlined"
            autofocus
            @keyup.enter="confirmAddPlayer"
            :error-messages="playerNameError"
          />
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn @click="cancelAddPlayer" text>Cancel</v-btn>
          <v-btn 
            @click="confirmAddPlayer" 
            color="success"
            :disabled="!isValidPlayerName"
          >
            Add Player
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Clear All Scores Dialog -->
    <v-dialog v-model="showClearScoresDialog" max-width="400">
      <v-card>
        <v-card-title class="text-h6">Clear All Scores?</v-card-title>
        <v-card-text>
          This will reset all player scores to 0. Player names will be kept.
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn @click="showClearScoresDialog = false" text>Cancel</v-btn>
          <v-btn @click="clearAllScores" color="warning">Clear Scores</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Clear All Players Dialog -->
    <v-dialog v-model="showClearAllDialog" max-width="400">
      <v-card>
        <v-card-title class="text-h6">Clear All Players?</v-card-title>
        <v-card-text>
          This will remove all players and their scores. This cannot be undone.
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn @click="showClearAllDialog = false" text>Cancel</v-btn>
          <v-btn @click="clearAll" color="error">Clear All</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Delete Player Dialog -->
    <v-dialog v-model="showDeletePlayerDialog" max-width="400">
      <v-card>
        <v-card-title class="text-h6">Delete {{ selectedPlayerName }}?</v-card-title>
        <v-card-text>
          This will permanently remove {{ selectedPlayerName }} and their score. This cannot be undone.
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn @click="showDeletePlayerDialog = false" text>Cancel</v-btn>
          <v-btn @click="confirmDeletePlayer" color="error">Delete Player</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Clear Player Score Dialog -->
    <v-dialog v-model="showClearPlayerDialog" max-width="400">
      <v-card>
        <v-card-title class="text-h6">Clear {{ selectedPlayerName }}'s Score?</v-card-title>
        <v-card-text>
          This will reset {{ selectedPlayerName }}'s score to 0.
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn @click="showClearPlayerDialog = false" text>Cancel</v-btn>
          <v-btn @click="confirmClearPlayerScore" color="warning">Clear Score</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-app>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue';

// Reactive data
const players = ref([]);
const scoreInputs = ref({});
const swipeStates = ref({});
const showClearScoresDialog = ref(false);
const showClearAllDialog = ref(false);
const showClearPlayerDialog = ref(false);
const showDeletePlayerDialog = ref(false);
const showAddPlayerDialog = ref(false);
const newPlayerName = ref('');
const selectedPlayerId = ref(null);
const selectedPlayerIndex = ref(null);
const selectedPlayerName = ref('');

// Swipe constants
const SWIPE_THRESHOLD = 80;
const MAX_SWIPE_DISTANCE = 120;

// Computed properties
const leaderInfo = computed(() => {
  if (players.value.length === 0) return { leaders: [], score: 0 };
  
  const maxScore = Math.max(...players.value.map(p => p.score));
  const leaders = players.value.filter(p => p.score === maxScore);
  
  return { leaders, score: maxScore };
});

const leaderText = computed(() => {
  const { leaders } = leaderInfo.value;
  if (leaders.length === 0) return '';
  if (leaders.length === 1) return leaders[0].name;
  return `${leaders.map(p => p.name).join(' & ')} (Tie)`;
});

const leaderScore = computed(() => leaderInfo.value.score);

const isValidPlayerName = computed(() => {
  return newPlayerName.value.trim().length > 0;
});

const playerNameError = computed(() => {
  if (!newPlayerName.value.trim() && newPlayerName.value.length > 0) {
    return ['Player name cannot be empty'];
  }
  if (players.value.some(p => p.name.toLowerCase() === newPlayerName.value.trim().toLowerCase())) {
    return ['Player name already exists'];
  }
  return [];
});

// Swipe handling methods
const initSwipeState = (playerId) => {
  if (!swipeStates.value[playerId]) {
    swipeStates.value[playerId] = {
      startX: 0,
      currentX: 0,
      translateX: 0,
      swiping: false,
      isDragging: false
    };
  }
};

const handleTouchStart = (event, playerId) => {
  initSwipeState(playerId);
  const touch = event.touches[0];
  swipeStates.value[playerId].startX = touch.clientX;
  swipeStates.value[playerId].isDragging = true;
  swipeStates.value[playerId].swiping = true;
};

const handleTouchMove = (event, playerId) => {
  if (!swipeStates.value[playerId]?.isDragging) return;
  
  event.preventDefault();
  const touch = event.touches[0];
  const deltaX = touch.clientX - swipeStates.value[playerId].startX;
  
  // Limit swipe distance
  const clampedDelta = Math.max(-MAX_SWIPE_DISTANCE, Math.min(MAX_SWIPE_DISTANCE, deltaX));
  swipeStates.value[playerId].translateX = clampedDelta;
  swipeStates.value[playerId].currentX = touch.clientX;
};

const handleTouchEnd = (event, playerId, playerIndex) => {
  if (!swipeStates.value[playerId]?.isDragging) return;
  
  const deltaX = swipeStates.value[playerId].translateX;
  
  // Check if swipe threshold was met
  if (Math.abs(deltaX) > SWIPE_THRESHOLD) {
    if (deltaX > 0) {
      // Swiped right - clear score
      clearPlayerScore(playerId);
    } else {
      // Swiped left - delete player (with confirmation)
      showDeleteConfirmation(playerId, playerIndex);
    }
  }
  
  // Reset swipe state
  resetSwipeState(playerId);
};

// Mouse events for desktop support
const handleMouseDown = (event, playerId) => {
  initSwipeState(playerId);
  swipeStates.value[playerId].startX = event.clientX;
  swipeStates.value[playerId].isDragging = true;
  swipeStates.value[playerId].swiping = true;
};

const handleMouseMove = (event, playerId) => {
  if (!swipeStates.value[playerId]?.isDragging) return;
  
  const deltaX = event.clientX - swipeStates.value[playerId].startX;
  const clampedDelta = Math.max(-MAX_SWIPE_DISTANCE, Math.min(MAX_SWIPE_DISTANCE, deltaX));
  swipeStates.value[playerId].translateX = clampedDelta;
};

const handleMouseEnd = (event, playerId, playerIndex) => {
  if (!swipeStates.value[playerId]?.isDragging) return;
  
  const deltaX = swipeStates.value[playerId].translateX;
  
  if (Math.abs(deltaX) > SWIPE_THRESHOLD) {
    if (deltaX > 0) {
      clearPlayerScore(playerId);
    } else {
      showDeleteConfirmation(playerId, playerIndex);
    }
  }
  
  resetSwipeState(playerId);
};

const resetSwipeState = (playerId) => {
  if (swipeStates.value[playerId]) {
    swipeStates.value[playerId].translateX = 0;
    swipeStates.value[playerId].swiping = false;
    swipeStates.value[playerId].isDragging = false;
  }
};

// Player management methods
const confirmAddPlayer = () => {
  if (!isValidPlayerName.value || playerNameError.value.length > 0) return;
  
  const newPlayer = {
    id: Date.now(),
    name: newPlayerName.value.trim(),
    score: 0
  };
  players.value.push(newPlayer);
  scoreInputs.value[newPlayer.id] = '';
  initSwipeState(newPlayer.id);
  
  // Reset dialog
  newPlayerName.value = '';
  showAddPlayerDialog.value = false;
  saveToStorage();
};

const cancelAddPlayer = () => {
  newPlayerName.value = '';
  showAddPlayerDialog.value = false;
};

const showDeleteConfirmation = (playerId, playerIndex) => {
  const player = players.value.find(p => p.id === playerId);
  if (player) {
    selectedPlayerId.value = playerId;
    selectedPlayerIndex.value = playerIndex;
    selectedPlayerName.value = player.name;
    showDeletePlayerDialog.value = true;
  }
};

const confirmDeletePlayer = () => {
  if (selectedPlayerIndex.value !== null) {
    const playerId = players.value[selectedPlayerIndex.value].id;
    players.value.splice(selectedPlayerIndex.value, 1);
    delete scoreInputs.value[playerId];
    delete swipeStates.value[playerId];
    saveToStorage();
  }
  
  // Reset dialog state
  showDeletePlayerDialog.value = false;
  selectedPlayerId.value = null;
  selectedPlayerIndex.value = null;
  selectedPlayerName.value = '';
};

const isValidScore = (score) => {
  return score && Number.isInteger(Number(score)) && Number(score) > 0;
};

const addScore = (playerId) => {
  const score = scoreInputs.value[playerId];
  if (!isValidScore(score)) return;
  
  const player = players.value.find(p => p.id === playerId);
  if (player) {
    player.score += Number(score);
    scoreInputs.value[playerId] = '';
    saveToStorage();
  }
};

const clearPlayerScore = (playerId) => {
  const player = players.value.find(p => p.id === playerId);
  if (player) {
    selectedPlayerId.value = playerId;
    selectedPlayerName.value = player.name;
    showClearPlayerDialog.value = true;
  }
};

const confirmClearPlayerScore = () => {
  const player = players.value.find(p => p.id === selectedPlayerId.value);
  if (player) {
    player.score = 0;
    saveToStorage();
  }
  showClearPlayerDialog.value = false;
  selectedPlayerId.value = null;
  selectedPlayerName.value = '';
};

const clearAllScores = () => {
  players.value.forEach(player => {
    player.score = 0;
  });
  showClearScoresDialog.value = false;
  saveToStorage();
};

const clearAll = () => {
  players.value = [];
  scoreInputs.value = {};
  swipeStates.value = {};
  showClearAllDialog.value = false;
  saveToStorage();
};

const saveToStorage = () => {
  localStorage.setItem('shwoop-players', JSON.stringify(players.value));
};

const loadFromStorage = () => {
  const saved = localStorage.getItem('shwoop-players');
  if (saved) {
    players.value = JSON.parse(saved);
    // Initialize score inputs and swipe states for loaded players
    players.value.forEach(player => {
      scoreInputs.value[player.id] = '';
      initSwipeState(player.id);
    });
  }
};

// Lifecycle
onMounted(() => {
  loadFromStorage();
});

// Watch for changes to save automatically
watch(players, saveToStorage, { deep: true });
</script>

<style scoped>
.leader-card {
  background: linear-gradient(135deg, #fff3e0 0%, #ffe0b2 100%);
  border: 2px solid #ffb74d;
}

.crown-icon {
  animation: bounce 2s infinite;
}

@keyframes bounce {
  0%, 20%, 50%, 80%, 100% {
    transform: translateY(0);
  }
  40% {
    transform: translateY(-10px);
  }
  60% {
    transform: translateY(-5px);
  }
}

.players-list {
  padding: 0;
}

.swipe-item-container {
  position: relative;
  margin-bottom: 8px;
  overflow: hidden;
  border-radius: 8px;
  background: white;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.swipe-actions {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  z-index: 1;
}

.swipe-action {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: white;
  font-weight: 500;
  gap: 4px;
}

.swipe-action-left {
  background: linear-gradient(135deg, #ff9800, #f57c00);
}

.swipe-action-right {
  background: linear-gradient(135deg, #f44336, #d32f2f);
}

.action-text {
  font-size: 12px;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.swipe-item {
  position: relative;
  z-index: 2;
  background: white;
  transition: transform 0.2s ease-out;
  user-select: none;
  cursor: grab;
}

.swipe-item.swiping {
  transition: none;
  cursor: grabbing;
}

.swipe-item:hover {
  box-shadow: 0 4px 8px rgba(0,0,0,0.12);
}

.actions-section {
  display: flex;
  align-items: center;
  flex-shrink: 0;
}

.score-input-group {
  display: flex;
  align-items: center;
}

.score-input {
  width: 100px;
}

.score-input :deep(.v-field) {
  border-top-right-radius: 0 !important;
  border-bottom-right-radius: 0 !important;
}

.plus-btn {
  border-top-left-radius: 0 !important;
  border-bottom-left-radius: 0 !important;
  border-top-right-radius: 6px !important;
  border-bottom-right-radius: 6px !important;
  margin-left: -1px;
}

/* Mobile optimizations */
@media (max-width: 600px) {
  .v-container {
    padding: 8px !important;
  }
  
  .score-input {
    width: 80px;
  }
  
  .actions-section {
    gap: 4px;
  }
}

/* Very small screens - stack actions */
@media (max-width: 480px) {
  .actions-section {
    flex-direction: column;
    gap: 8px;
  }
  
  .score-input-group {
    order: 1;
  }
}
</style>