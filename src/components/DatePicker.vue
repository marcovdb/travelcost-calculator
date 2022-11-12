<script setup>
import { onMounted, ref, watch } from "vue";
import flatpickr from "flatpickr";
import { getDay } from "date-fns";

const props = defineProps({ range: Array, travelDays: Array });
const emit = defineEmits(["update:travelDates"]);

const options = {
  mode: "multiple",
  inline: true,
  defaultDate: [],
  onChange(selectedDates) {
    console.log(selectedDates);
    emit("update:travelDates", selectedDates);
  },
};
const picker = ref();
let flatpickrInstance = null;

const setDays = () => {
  const opts = { ...options };
  opts.defaultDate.length = 0;
  props.range.forEach((date) => {
    const day = getDay(date);
    if (props.travelDays.includes(day)) {
      opts.defaultDate.push(date);
    }
  });
  console.log(opts.defaultDate);
  emit("update:travelDates", opts.defaultDate);
  if (flatpickrInstance) {
    flatpickrInstance.destroy();
  }
  flatpickrInstance = flatpickr(picker.value, opts);
};

onMounted(setDays);

watch(() => props.range, setDays);
watch(() => props.travelDays, setDays);
</script>

<template>
  <div class="mb-3">
    <span ref="picker"></span>
  </div>
</template>

<style>
@import url("flatpickr/dist/flatpickr.min.css");
</style>
