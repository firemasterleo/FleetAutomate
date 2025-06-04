<template>
  <div class="sectioncontainer">
    <div class="section">
      <div class="section1">
        <p>Admin</p>

          <h2>Vehicle Compliance Dashboard</h2>
          <div class="update">
            <p class="lastupdated">Last Updated: <span>{{ formattedLastSyncedAt }}</span></p>
            <button @click="triggerFullSync">Update</button>
          </div>

      </div>

      <!-- Section: Documents expiring in next 30 days -->
      <div class="section2">
        <h3 class="subheading">Documents Expiring Within 30 Days</h3>
        <div v-if="loading" class="loader-container">
          <div class="loader"></div>
        </div>
        <table v-else class="vehicles-table">
          <thead>
            <tr>
              <th>Vehicle Number</th>
              <th>Vehicle Name</th>
              <th>Expiring Documents</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="vehicle in expiringSoonVehicles" :key="vehicle.vehicle_number">
              <td>{{ vehicle.vehicle_number }}</td>
              <td>{{ vehicle.vehicle_name }}</td>
              <td>
                <div class="doc-status-list">
                  <div 
                    v-for="doc in vehicle.soon_to_expire_docs.filter(d => d.days_left > 0 && d.days_left <= 30)" 
                    :key="`${vehicle.vehicle_number}-${doc.name}`"
                    :class="['doc-status', getDocStatusClass(doc.days_left)]"
                  >
                    <span class="doc-name">{{ doc.name }}</span>
                    <span class="doc-expiry">({{ `${doc.days_left} days left` }})</span>
                  </div>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- Section: Already expired documents -->
      <div class="section2">
        <h3 class="subheading">Expired Documents</h3>
        <div v-if="loading" class="loader-container">
          <div class="loader --1"></div>
        </div>
        <table v-else class="vehicles-table">
          <thead>
            <tr>
              <th>Vehicle Number</th>
              <th>Vehicle Name</th>
              <th>Expired Documents</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="vehicle in expiredVehicles" :key="vehicle.vehicle_number">
              <td>{{ vehicle.vehicle_number }}</td>
              <td>{{ vehicle.vehicle_name }}</td>
              <td>
                <div class="doc-status-list">
                  <div 
                    v-for="doc in vehicle.soon_to_expire_docs.filter(d => d.days_left <= 0)" 
                    :key="`${vehicle.vehicle_number}-${doc.name}`"
                    class="doc-status expired"
                  >
                    <span class="doc-name">{{ doc.name }}</span>
                    <span class="doc-expiry">(EXPIRED)</span>
                  </div>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import { createClient } from '@supabase/supabase-js';


const SUPABASE_URL = 'https://wmhpulllzyarvfatyfuw.supabase.co';
const SUPABASE_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6IndtaHB1bGxsenlhcnZmYXR5ZnV3Iiwicm9sZSI6InNlcnZpY2Vfcm9sZSIsImlhdCI6MTc0MjcyODU5NiwiZXhwIjoyMDU4MzA0NTk2fQ.PY8wgMW8hx-hRAsnKr1wTXh3PqiMavav0ljB3SvnMps';

const supabase = createClient(SUPABASE_URL, SUPABASE_KEY);

const lastSyncedAt = ref('Loading...');
const formattedLastSyncedAt = ref('Loading...');


const vehicles = ref([]);
const loading = ref(true);
const error = ref(null);


// Function to format the date with suffix (e.g., 1st, 2nd, 3rd, 4th)
const formatDateWithSuffix = (date) => {
  const day = date.getDate();
  const daySuffix = ['th', 'st', 'nd', 'rd'][
    (day % 10 === 1 && day !== 11) ? 1 :
    (day % 10 === 2 && day !== 12) ? 2 :
    (day % 10 === 3 && day !== 13) ? 3 : 0
  ];
  const month = date.toLocaleString('default', { month: 'long' });
  const year = date.getFullYear();
  return `${day}${daySuffix} ${month} ${year}`;
};


// Fetch the last sync time from Supabase
const getLastSyncTime = async () => {
  try {
    const { data, error } = await supabase
      .from('demo_sync_log')  // Your table name
      .select('last_synced_at')
      .order('last_synced_at', { ascending: false })
      .limit(1);  // Get the most recent sync record

    if (error) {
      console.error('Error fetching sync time:', error.message);
      lastSyncedAt.value = 'Error fetching sync time';
    } else {
      const lastSyncTime = data && data.length > 0 ? data[0].last_synced_at : null;
      if (lastSyncTime) {
        const date = new Date(lastSyncTime);
        formattedLastSyncedAt.value = formatDateWithSuffix(date);
      } else {
        formattedLastSyncedAt.value = 'Never synced';
      }
    }
  } catch (err) {
    console.error('❌ Error fetching sync time:', err);
    formattedLastSyncedAt.value = 'Failed to load';
  }
};

const triggerFullSync = async () => {
  try {
    // Show loading state or disable the button while sync is in progress
    const button = document.querySelector('button'); // Assuming button is the trigger element
    button.disabled = true;
    button.textContent = 'Syncing...';

    const response = await fetch('https://automation-backend-9n51.onrender.com/api/full-sync-demo', {
      method: 'GET', // You can change this to 'POST' if your API uses POST for this route
    });

    if (!response.ok) {
      throw new Error('Sync failed with status: ' + response.status);
    }

    const data = await response.json();
    if (data.success) {
      alert(data.message);  // Notify user of success
      getLastSyncTime();  // Refresh last synced time
      await fetchVehicleData(); // 🔁 This updates the table with fresh data

    } else {
      alert('Sync failed. Please try again.');
    }
  } catch (err) {
    console.error('❌ Error during sync:', err);
    alert('Error occurred while syncing. Please try again.');
  } finally {
    // Re-enable the button and restore its text after the process is complete
    const button = document.querySelector('button');
    button.disabled = false;
    button.textContent = 'Update';  // Reset to initial text
  }
};


const fetchVehicleData = async () => {
  loading.value = true;
  error.value = null;

  try {
    const { data, error: fetchError } = await supabase
      .from('demo_vehicle_docs_review')
      .select('*')
      .order('review_date', { ascending: false });

    if (fetchError) throw fetchError;
    vehicles.value = data;
  } catch (err) {
    console.error('Error fetching vehicle data:', err);
    error.value = err.message || 'Failed to load vehicle data';
  } finally {
    loading.value = false;
  }
};


const expiringSoonVehicles = computed(() => {
  return vehicles.value
  .filter(vehicle =>
  vehicle.soon_to_expire_docs?.some(doc => doc.days_left > 0 && doc.days_left <= 30)
)
.sort((a, b) => {
  const aMin = Math.min(...a.soon_to_expire_docs.map(doc => doc.days_left));
  const bMin = Math.min(...b.soon_to_expire_docs.map(doc => doc.days_left));
  return aMin - bMin;
});
});

const expiredVehicles = computed(() => {
  return vehicles.value
  .filter(vehicle =>
  vehicle.soon_to_expire_docs?.some(doc => doc.days_left <= 0)
);
});

const getDocStatusClass = (daysLeft) => {
  if (daysLeft <= 0) return 'expired';
  if (daysLeft <= 10) return 'critical';
  return 'warning';
};
// Fetch the last synced time and vehicle data when the component is mounted
onMounted(() => {
  getLastSyncTime();   // Get last sync time
  fetchVehicleData();  // Fetch vehicle data
});


</script>

<style lang="scss" scoped>
@use "@/assets/sass/variables" as *;

.sectioncontainer {
  background-color: rgb(70, 96, 181);
  width: 100%;
  height: fit-content;
  display: flex;
  color: $text-dark;
  // border: solid red;
  
  .section {
    background-color: $bg-white;
    width: 80rem;
    height: 100%;
    margin-inline: auto;
    padding: 1rem;
    // overflow-y: auto;

    .section1 {
      color: $text-dark;

      p {
        font-size: 12px;
        padding-bottom: 1.5rem;
        padding-top: 1rem;
      }

      h2 {
        font-size: 20px;
      }
      .update {
        display: flex;
        gap: 5rem;
        button {
          width: 5rem;
          height: 1.5rem;
          border-radius: 0.5rem;
          border: none;
          background-color: rgb(173, 206, 88);
          color: $text-muted;
        }
      }

      .subtitle {
        font-size: 13px;
        font-weight: 400;
        color: #666;
        margin-top: -1rem;
        margin-bottom: 1rem;
      }

      .lastupdated {
        font-size: 12px;
        padding-bottom: 1.5rem;
        padding-top: 0.5rem;
      }
    }

    .section2 {
      max-height: 20rem;
      overflow: auto;
      margin-bottom: 6rem;
      border-radius: 0.5rem;
      background-color: rgb(249, 249, 249);

      .subheading {
        font-size: 14px;
        font-weight: bold;
        margin-bottom: 1rem;
        padding-inline: 1rem;
        position: sticky;
        top: 0;
        left: 0;
        background-color: $bg-white;
      }

      .vehicles-table {
        width: max-content;
        min-width: 100%;
        border-collapse: collapse;
    
        th, td {
          padding: 0.5rem;
          text-align: left;
          border-bottom: 1px solid #ddd;
          white-space: nowrap;
          font-size: 12px;
        }
        
        th {
          height: fit-content;
          padding: 0rem;
          padding-inline: 0.5rem;
          background-color: rgb(225, 230, 236);
          border-radius: 0.5rem;
        }

        .doc-status-list {
          display: flex;
          flex-direction: row;
          gap: 0.5rem;
          flex-wrap: nowrap;
          overflow-x: auto;
          padding: 0.25rem 0;
        }

        .doc-status {
          display: inline-flex;
          gap: 0.25rem;
          padding: 0.25rem 0.5rem;
          border-radius: 5px;
          font-size: 12px;
          background-color: #eee;
          white-space: nowrap;

          &.expired {
            background-color: #fdd;
            color: #a00;
          }

          &.critical {
            background-color: #ffe9c0;
            color: #c47a00;
          }

          &.warning {
            background-color: #ffffcc;
            color: #999900;
          }
        }
      }
      
      overflow-x: auto;
      overflow-y: auto;

      // Force scrollbar visibility
      scrollbar-width: auto;
      scrollbar-color: #ccc #f0f0f0;

      &::-webkit-scrollbar {
        height: 8px;
        width: 8px;
      }

      &::-webkit-scrollbar-track {
        background: #f0f0f0;
      }

      &::-webkit-scrollbar-thumb {
        background-color: #ccc;
        border-radius: 4px;
      }
    }
  }
}

@media (max-width: 800px) {
  .sectioncontainer .section {
    width: 100vw;
    height: 130vh;
    padding-inline: 1rem;
  }
}

.loader-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 17rem;
  // border: solid;
}

.loader {
  display: grid;
  place-items: center;
  position: relative;
  width: 50px;
  height: 50px;
}

.loader::before,
.loader::after {
  content: '';
  box-sizing: border-box;
  position: absolute;
  border-radius: 50%;
}

.loader::before {
  width: 20px;
  height: 20px;
  border: 4px solid rgb(168, 152, 152); /* dark text color */
  border-top-color: transparent;
  animation: loader-spin 1s linear infinite;
}

.loader::after {
  width: 30px;
  height: 30px;
  border: 2px solid transparent;
  border-top-color: $text-dark;
  animation: loader-spin 0.6s linear reverse infinite;
}

@keyframes loader-spin {
  100% {
    transform: rotate(360deg);
  }
}
</style>