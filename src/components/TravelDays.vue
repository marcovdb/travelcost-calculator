<script setup>
import { ref, watch } from "vue";

const emit = defineEmits(["update:TravelDays"]);

const days = [
  "zondag",
  "maandag",
  "dinsdag",
  "woensdag",
  "donderdag",
  "vrijdag",
  "zaterdag",
];
const selected = ref([]);

watch(selected, () => {
  emit("update:TravelDays", selected.value);
});
</script>

<template>
  <fieldset>
    <legend>Kantoordagen</legend>
    <div class="form-check" v-for="(day, index) in days" :key="day">
      <input
        class="form-check-input"
        type="checkbox"
        :id="day"
        :value="index"
        v-model="selected"
        :disabled="index === 0 || index === days.length - 1"
      />
      <label class="form-check-label" :for="day">{{ day }}</label>
    </div>
  </fieldset>
</template>
