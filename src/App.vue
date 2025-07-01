<template>
  <v-app>
    <v-main>
      <v-container fluid class="pa-4">
        <!-- Header -->
        <v-row justify="center" class="mb-4">
          <v-col cols="12" class="text-center">
            <h1 class="text-h4 font-weight-bold primary--text">
              <v-icon class="mr-2" size="large">mdi-cards-playing</v-icon>
              Shwoop Score Tracker
            </h1>
          </v-col>
        </v-row>

        <!-- Leader Section -->
        <v-row v-if="players.length > 0" justify="center" class="mb-4">
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
        <v-row justify="center" class="mb-4">
          <v-col cols="12">
            <v-btn
              @click="addPlayer"
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
            <div class="players-container">
              <div
                v-for="(player, index) in players"
                :key="player.id"
                class="player-row"
              >
                <!-- Player Icon -->
                <div class="player-icon">
                  <v-avatar color="primary" size="40">
                    <v-icon color="white">mdi-account</v-icon>
                  </v-avatar>
                </div>

                <!-- Player Name -->
                <div class="player-name">
                  <v-text-field
                    v-model="player.name"
                    variant="outlined"
                    density="compact"
                    hide-details
                    min-width="250"
                    class="player-name-input"
                    @blur="saveToStorage"
                  />
                </div>

                <!-- Spacer -->
                <div class="spacer"></div>

                <!-- Score Input with Plus Button -->
                <div class="score-input-group">
                  <v-text-field
                    v-model.number="scoreInputs[player.id]"
                    @keyup.enter="addScore(player.id)"
                    type="number"
                    min="1"
                    variant="outlined"
                    density="compact"
                    label="Add points"
                    hide-details
                    class="score-input"
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

                <!-- Total Score -->
                <div class="total-score">
                  <div class="score-label-value">
                    <span class="text-caption text-medium-emphasis">Total</span>
                    <span class="text-h6 font-weight-bold primary--text">{{ player.score }}</span>
                  </div>
                </div>

                <!-- Action Buttons -->
                <div class="action-buttons">
                  <v-btn-group>
                  <v-btn
                    color="warning"
                    size="small"
                    icon="mdi-refresh"
                    class="action-btn px-6"
                    @click="clearPlayerScore(player.id)"
                  />
                  <v-btn
                    color="error"
                    size="small"
                    icon="mdi-delete"
                    class="action-btn px-6"
                    @click="removePlayer(index)"
                  />
                </v-btn-group>
                </div>
              </div>
            </div>
          </v-col>
        </v-row>

        <!-- Control Buttons -->
        <v-row v-if="players.length > 0" justify="center" class="mt-6">
          <v-col cols="12">
            <v-row dense>
              <v-col cols="sm-12 md-6">
                <v-btn
                  @click="showClearScoresDialog = true"
                  color="warning"
                  block
                  prepend-icon="mdi-refresh"
                >
                  Clear All Scores
                </v-btn>
              </v-col>
              <v-col cols="sm-12 md-6">
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
const showClearScoresDialog = ref(false);
const showClearAllDialog = ref(false);
const showClearPlayerDialog = ref(false);
const selectedPlayerId = ref(null);
const selectedPlayerName = ref('');

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

// Methods
const addPlayer = () => {
  const newPlayer = {
    id: Date.now(),
    name: `Player ${players.value.length + 1}`,
    score: 0
  };
  players.value.push(newPlayer);
  scoreInputs.value[newPlayer.id] = '';
  saveToStorage();
};

const removePlayer = (index) => {
  const playerId = players.value[index].id;
  players.value.splice(index, 1);
  delete scoreInputs.value[playerId];
  saveToStorage();
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
    // Initialize score inputs for loaded players
    players.value.forEach(player => {
      scoreInputs.value[player.id] = '';
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

.players-container {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.player-row {
  display: flex;
  align-items: center;
  background: white;
  border-radius: 12px;
  border: 1px solid #e0e0e0;
  padding: 12px 16px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  transition: transform 0.2s ease-in-out, box-shadow 0.2s ease-in-out;
  gap: 12px;
}

.player-row:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0,0,0,0.15);
}

.player-icon {
  flex-shrink: 0;
}

.player-name {
  flex-shrink: 0;
  width: 140px;
}

.player-name-input {
  max-width: 140px;
}

.spacer {
  flex-grow: 1;
}

.score-input-group {
  display: flex;
  align-items: center;
  flex-shrink: 0;
}

.score-input {
  width: 100px;
}

.score-input :deep(.v-field) {
  border-top-right-radius: 0 !important;
  border-bottom-right-radius: 0 !important;
}

.plus-btn {
  flex-shrink: 0;
  border-top-left-radius: 0 !important;
  border-bottom-left-radius: 0 !important;
  border-top-right-radius: 6px !important;
  border-bottom-right-radius: 6px !important;
  margin-left: -1px;
}

.total-score {
  flex-shrink: 0;
  min-width: 60px;
  text-align: center;
}

.score-label-value {
  display: flex;
  flex-direction: column;
  align-items: center;
  line-height: 1.2;
}

.action-buttons {
  display: flex;
  gap: 4px;
  flex-shrink: 0;
}

.action-btn {
  flex-shrink: 0;
}

/* Mobile optimizations */
@media (max-width: 600px) {
  .v-container {
    padding: 8px !important;
  }
  
  .player-row {
    padding: 8px 12px;
    gap: 8px;
  }
  
  .player-name {
    width: 100px;
  }
  
  .player-name-input {
    max-width: 100px;
  }
  
  .score-input {
    width: 80px;
  }
  
  .total-score {
    min-width: 50px;
  }
  
  .score-label-value {
    font-size: 0.9em;
  }
  
  .action-buttons {
    gap: 2px;
  }
}

/* Very small screens - stack score label/value */
@media (max-width: 480px) {
  .score-label-value {
    display: flex;
    flex-direction: column;
  }
}
</style>