<script setup>
import { ref, computed } from "vue";
import { subMonths, format, eachDayOfInterval } from "date-fns";
import * as Papa from "papaparse";
import DatePicker from "./components/DatePicker.vue";
import TravelDays from "./components/TravelDays.vue";

const distance = ref(0);
const end = ref(format(new Date(), "yyyy-MM-dd"));
const reimbursement = ref(0.19);
const start = ref(format(subMonths(new Date(), 1), "yyyy-MM-dd"));
const travelDates = ref([]);
const travelDays = ref([]);

const range = computed(() => {
  const range = eachDayOfInterval({
    start: new Date(start.value),
    end: new Date(end.value),
  });
  return range;
});

const tripDistance = computed(() => distance.value * 2);

const setTravelDays = (payload) => {
  travelDays.value = payload;
};

const setTravelDates = (payload) => {
  travelDates.value.length = 0;
  travelDates.value.push(...payload);
};

const round = (input) => Math.round(input * 100) / 100;

const exportToCsv = () => {
  const csv = Papa.unparse(
    travelDates.value.reduce((prevVal, curVal) => {
      prevVal.push({
        date: format(curVal, "PPPP"),
        distance: tripDistance.value,
        amount: round(tripDistance.value * reimbursement.value),
      });
      return prevVal;
    }, []),
    { delimiter: ";" }
  );

  const blob = new Blob([csv]);
  const a = document.createElement("a");
  a.href = URL.createObjectURL(blob, { type: "text/csv" });
  a.download = "export.csv";
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
};
</script>

<template>
  <div class="container py-4 px-3 mx-auto">
    <h1>Travel cost calculator</h1>
    <form autocomplete="off">
      <div class="row">
        <div class="col-md-4">
          <div class="mb-3">
            <label for="startdate" class="form-label">Start date</label>
            <input
              id="startdate"
              class="form-control"
              type="date"
              v-model="start"
            />
          </div>
          <div class="mb-3">
            <label for="enddate" class="form-label">End date</label>
            <input
              id="enddate"
              class="form-control"
              type="date"
              v-model="end"
            />
          </div>
        </div>
        <div class="col-md-4">
          <TravelDays @update:TravelDays="setTravelDays" />
        </div>
        <div class="col-md-4">
          <div class="mb-3">
            <label class="form-label" for="reimbursement"
              >Reimbursement per km</label
            >
            <div class="input-group">
              <span class="input-group-text">€</span>
              <input
                id="reimbursement"
                class="form-control"
                type="number"
                v-model="reimbursement"
              />
            </div>
          </div>
          <div class="input-group mb-3">
            <label class="form-label" for="distance">Distance (one way)</label>
            <div class="input-group">
              <input
                id="distance"
                class="form-control"
                type="number"
                v-model="distance"
              />
              <span class="input-group-text">km</span>
            </div>
          </div>
        </div>
      </div>
      <div class="row">
        <div class="col-md-4 mt-3">
          <DatePicker
            :range="range"
            :travel-days="travelDays"
            @update:travelDates="setTravelDates"
          />
        </div>
        <div class="col-md-8">
          <table class="table">
            <thead>
              <tr>
                <th>Date</th>
                <th class="text-end">Distance driven (km)</th>
                <th class="text-end">Amount to be reimbursed (€)</th>
              </tr>
            </thead>
            <tfoot class="table-group-divider">
              <tr>
                <td>&nbsp;</td>
                <td class="text-end">
                  {{ round(tripDistance * travelDates.length) }}
                </td>
                <td class="text-end">
                  {{ round(tripDistance * reimbursement * travelDates.length) }}
                </td>
              </tr>
            </tfoot>
            <tbody class="table-group-divider">
              <tr v-for="date in travelDates" :key="date.toString()">
                <td>{{ format(date, "PPPP") }}</td>
                <td class="text-end">{{ tripDistance }}</td>
                <td class="text-end">
                  {{ round(tripDistance * reimbursement) }}
                </td>
              </tr>
            </tbody>
          </table>
        </div>
        <div class="col-12 text-end">
          <button class="btn btn-primary" type="button" @click="exportToCsv">
            Export to CSV
          </button>
        </div>
      </div>
    </form>
  </div>
</template>

<style>
@import "bootstrap/dist/css/bootstrap.css";
</style>
