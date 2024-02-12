<template>
  <div class="container">
    <div v-if="isLoading">Loading...</div>
    <table v-else class="table table-bordered">
      <thead>
        <tr>
          <th v-for="header in displayedHeaders" :key="header" @click="sortTable(header)">{{ header }}</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(user, index) in sortedUsers" :key="user.email">
          <td scope="row">{{ index + 1 }}</td>
          <td>{{user.name.title}} {{ user.name.first }} {{ user.name.last }}</td>
          <td>{{ formatDate(user.dob.date) }}</td>
          <td>{{ user.email }}</td>
          <td v-html="formatLocation(user.location)"></td>
          <td>{{ user.phone }}</td>
          <td><img :src="user.picture.large" alt=""></td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue';

const users = ref([]);
const isLoading = ref(true);
const API_URL = 'https://randomuser.me/api/?results=50';

const fetchData = async () => {
  try {
    const response = await fetch(API_URL);
    const data = await response.json();
    users.value = data.results;
    isLoading.value = false; // Set isLoading to false once data is fetched
  } catch (error) {
    console.error('Error fetching users:', error);
  }
};

onMounted(() => {
  // fetchData method call here on page load
  fetchData()
});

// table header 
const desiredHeaders = ['slno.', 'Name', 'DOB', 'Email', 'Location', 'Phone', 'Picture'];

// header make reactive
const displayedHeaders = computed(() => {
  return desiredHeaders;
});

const formatLocation = (location) => {
  // location in single level object 
  return `${location.street.number},${location.street.name} <br>${location.city},${location.state}<br>${location.postcode},${location.country}`;
};

const formatDate = (date) => {
  // date format for dd-mm-yyyy
  const formattedDate = new Date(date);
  const day = formattedDate.getDate();
  const month = formattedDate.getMonth() + 1;
  const year = formattedDate.getFullYear();
  return `${day < 10 ? '0' + day : day}-${month < 10 ? '0' + month : month}-${year}`;
};

const sortedUsers = computed(() => {
  return users.value.slice().sort((a, b) => {
    let comparison = 0;
    // start sort functinality by header Name to column
    if (sortBy.value === 'Name') {
      comparison = a.name.first.localeCompare(b.name.first);
    } else if (sortBy.value === 'DOB') {
      comparison = new Date(a.dob.date) - new Date(b.dob.date);
    } else if (sortBy.value === 'Email') {
      comparison = a.email.localeCompare(b.email);
    } else if (sortBy.value === 'Location') {
      const locationA = `${a.location.street.number} ${a.location.street.name} ${a.location.city} ${a.location.state} ${a.location.postcode} ${a.location.country}`;
      const locationB = `${b.location.street.number} ${b.location.street.name} ${b.location.city} ${b.location.state} ${b.location.postcode} ${b.location.country}`;
      comparison = locationA.localeCompare(locationB);
    }else if (sortBy.value === 'Phone') {
      comparison = a.phone.localeCompare(b.phone);
    }
   
    return comparison;
  });
});
const sortBy = ref(null);

const sortTable = (header) => {
  // If the header is already being sorted by, reverse the order
  if (sortBy.value === header) {
    sortBy.value = null;
  } else {
    sortBy.value = header;
  }
};
</script>

<style>
.table-bordered td, .table-bordered th {
  font-size: 14px;
  cursor: pointer;
}
</style>
