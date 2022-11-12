<script setup>
import { onMounted, ref, watch } from "vue";
import flatpickr from "flatpickr";
import { Dutch } from "flatpickr/dist/l10n/nl";
import { getDay } from "date-fns";

const props = defineProps({ range: Array, travelDays: Array });
const emit = defineEmits(["update:travelDates"]);

const options = {
  mode: "multiple",
  inline: true,
  defaultDate: [],
  locale: Dutch,
  onChange(selectedDates) {
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
  emit("update:travelDates", opts.defaultDate);
  if (flatpickrInstance) {
    flatpickrInstance.destroy();
  }
  flatpickrInstance = flatpickr(picker.value, opts);
};

watch(() => props.range, setDays);
watch(() => props.travelDays, setDays);

onMounted(setDays);
</script>

<template>
  <span ref="picker"></span>
  <div class="mt-3">
    <span class="form-text"
      >Tip! Je kunt extra dagen selecteren of dagen waarop je niet ging,
      deselecteren.</span
    >
  </div>
</template>

<style>
@import url("flatpickr/dist/flatpickr.min.css");
</style>
