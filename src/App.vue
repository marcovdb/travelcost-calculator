<script setup>
import { ref, computed } from "vue";
import { subMonths, format, eachDayOfInterval } from "date-fns";
import { nl } from "date-fns/locale";
import * as Papa from "papaparse";
import DatePicker from "./components/DatePicker.vue";
import TravelDays from "./components/TravelDays.vue";

const allowance = ref(0.19);
const distance = ref(0);
const end = ref(format(new Date(), "yyyy-MM-dd", { locale: nl }));
const start = ref(
  format(subMonths(new Date(), 1), "yyyy-MM-dd", { locale: nl })
);
const travelDates = ref([]);
const travelDays = ref([]);

const range = computed(() => {
  if (!start.value || !end.value) {
    return [];
  }
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
  travelDates.value = payload.map((date) =>
    format(date, "PPPP", { locale: nl })
  );
};

const round = (input) => (Math.round(input * 100) / 100).toLocaleString();

const exportToCsv = () => {
  const tripCount = travelDates.value.length;
  const csv = Papa.unparse(
    [
      ...travelDates.value.reduce((prevVal, date) => {
        prevVal.push({
          date,
          distance: round(tripDistance.value),
          amount: round(tripDistance.value * allowance.value),
        });
        return prevVal;
      }, []),
      ...[
        {
          date: "TOTAAL",
          distance: `=SUM(B2:B${tripCount + 1})`,
          amount: `=SUM(C2:C${tripCount + 1})`,
        },
      ],
    ],
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
    <h1>Reiskostendeclaratiegenerator</h1>
    <p class="lead">Geen zin om handmatig een Excel te maken? Ik ook niet!</p>
    <form autocomplete="off">
      <div class="row">
        <div class="col-lg-4">
          <div class="mb-3">
            <label for="startdate" class="form-label">Startdatum</label>
            <input
              id="startdate"
              class="form-control"
              type="date"
              v-model="start"
            />
          </div>
          <div class="mb-3">
            <label for="enddate" class="form-label">Einddatum</label>
            <input
              id="enddate"
              class="form-control"
              type="date"
              v-model="end"
            />
          </div>
          <div class="mb-3 pt-3">
            <DatePicker
              :range="range"
              :travel-days="travelDays"
              @update:travelDates="setTravelDates"
            />
          </div>
        </div>
        <div class="col-lg-8">
          <div class="row">
            <div class="col-md-5">
              <TravelDays @update:TravelDays="setTravelDays" />
            </div>
            <div class="col-md-7">
              <div class="input-group mb-3">
                <label class="form-label" for="distance"
                  >Afstand huis - kantoor</label
                >
                <div class="input-group">
                  <input
                    id="distance"
                    class="form-control"
                    type="number"
                    v-model="distance"
                  />
                  <span class="input-group-text">km</span>
                </div>
                <span class="form-text"
                  >Enkele reis dus! We verdubbelen het later.</span
                >
              </div>
              <div class="mb-3">
                <label class="form-label" for="allowance"
                  >Vergoeding per km</label
                >
                <div class="input-group">
                  <span class="input-group-text">€</span>
                  <input
                    id="allowance"
                    class="form-control"
                    type="number"
                    v-model="allowance"
                  />
                </div>
              </div>
            </div>
          </div>
          <div class="col-12">
            <hr />
            <table class="table table-sm">
              <thead>
                <tr>
                  <th>Datum</th>
                  <th class="text-end">Gereden afstand (km)</th>
                  <th class="text-end">Vergoeding (€)</th>
                </tr>
              </thead>
              <tfoot class="table-group-divider">
                <tr>
                  <td>{{ travelDates.length }} dagen</td>
                  <td class="text-end">
                    {{ round(tripDistance * travelDates.length) }}
                  </td>
                  <td class="text-end">
                    {{ round(tripDistance * allowance * travelDates.length) }}
                  </td>
                </tr>
              </tfoot>
              <tbody class="table-group-divider">
                <tr v-for="date in travelDates" :key="date.toString()">
                  <td>{{ date }}</td>
                  <td class="text-end">{{ round(tripDistance) }}</td>
                  <td class="text-end">
                    {{ round(tripDistance * allowance) }}
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
          <div class="col-12 text-end">
            <button class="btn btn-primary" type="button" @click="exportToCsv">
              Exporteer naar CSV
            </button>
          </div>
        </div>
      </div>
    </form>
  </div>
</template>

<style>
@import "bootstrap/dist/css/bootstrap.css";
</style>
