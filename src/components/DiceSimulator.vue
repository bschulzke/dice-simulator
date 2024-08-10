<template>
  <div class="simulator">
    <div class="flex">
      <div class="label">number of dice</div>
      <input type="number" min="0" v-model="numDice"/>
    </div>
    <div class="flex">
      <div class="label">skill level</div>
      <input type="number" min="0" max="4" v-model="skillLevel"/>
    </div>
    <div class="flex">
      <div class="label">rerolls</div>
      <input type="number" min="0" :max="numDice" v-model="numRerolls"/>
    </div>
    <div class="flex">
      <div class="label">require success on all</div>
      <input v-model="requireSuccessOnAll" type="checkbox"/>
    </div>
    <div class="flex">
      <div class="label">simulate <span class="variable">n</span> rolls</div>
      <input v-model="simulateNRolls" type="checkbox"/>
    </div>
    <div :class="{strike: !simulateNRolls}" class="flex">
      <div class="label">simulations (<span class="variable">n</span>):</div>
      <input :disabled="!simulateNRolls" type="number" v-model="simulations">
    </div>
    <hr/>
    <div class="flex">
      <button @click="roll">Roll</button>
    </div>
    <hr/>
    <div>{{ result }}</div>
    <div v-if="originalRoll">{{ reroll ? 'Original Roll:' : '' }} {{ originalRoll }}</div>
    <div v-if="reroll">Reroll: {{ reroll }}</div>
  </div>
</template>

<script setup>
  import { ref } from 'vue';

  const skillSuccess = {
    0: [6],
    1: [5,6],
    2: [4,5,6],
    3: [3,4,5,6],
    4: [2,3,4,5,6]
  }

  const result = ref('');
  const numDice = ref(6);
  const skillLevel = ref(0);
  const originalRoll = ref(null);
  const reroll = ref(null);
  const numRerolls = ref(0);
  const requireSuccessOnAll = ref(false);
  const simulateNRolls = ref(false);
  const simulations = ref(10000);
  const minSuccess = ref(6);

  function roll() {
    if (!simulateNRolls.value) {
      let success = rollOnce();
      if (success) {
        result.value = 'SUCCESS'
      } else {
        result.value = 'FAILURE'
      }
    } else {
      let successRatio = rollNTimes();
      originalRoll.value = null;
      reroll.value = null;
      result.value = 'Success ratio: ' + successRatio;
    }
  }

  function rollNTimes() {
    let successes = 0;
    for (let i = 0; i < simulations.value; i++) {
      if (rollOnce()) {
        successes++;
      }
    }
    console.log("successes: " + successes + ", n: " + simulations.value);
    return successes / simulations.value;
  }

  function rollOnce() {
    let success = false;
    const results = [];
    for (let i = 0; i < numDice.value; i++) {
      results.push(Math.floor(Math.random() * 6) + 1);
    }

    success = isSuccess(results);
    originalRoll.value = [...results];
    let numRerolled = 0;
    for (let i = 0; i < results.length; i++) {
      let result = results[i];
      if (result === 1 && numRerolled < numRerolls.value) {
        results[i] = Math.floor(Math.random() * 6) + 1;
        numRerolled++;
      }
    }
    if (numRerolled > 0) {
      reroll.value = [...results];
      success = isSuccess(results);
    } else {
      reroll.value = null;
    }
    return success;
  }

  function isSuccess(results) {
    if (succeeds(results, skillLevel.value)) {
      return true;
    } else {
      return false;
    }
  }

  function succeeds(roll, skillLevel) {
    if (requireSuccessOnAll.value) {
      return succeedsOnAll(roll, skillLevel);
    } else {
      return succeedsOnOne(roll, skillLevel);
    }
  }

  function succeedsOnOne(roll, skillLevel) {
    for (let i = 0; i < skillSuccess[skillLevel].length; i++) {
      const successValue = skillSuccess[skillLevel][i];
      if (roll.includes(successValue)) {
        return true;
      }
    }
    return false;
  }

  function succeedsOnAll(roll, skillLevel) {
    for (const value of roll) {
      if (!skillSuccess[skillLevel].includes(value)) {
        return false;
      }
    }
    return true;
  }

</script>

<style scoped>
  .flex {
    display: flex; 
    justify-content: left;
    align-items: center;
    margin: 0.5rem 0;
  }
  .label {
    margin-right: 1rem;
    font-size: 0.75rem;
    width: 15rem;
    text-align: left;
    text-transform: uppercase;
    letter-spacing: 0.1rem;
  }
  input[type="number"] {
    width: 5rem;
  }
  .variable {
    font-style: italic;
  }
  .strike {
    text-decoration: line-through;
  }
</style>
