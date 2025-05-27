<!-- Template Section -->
<template>
  <div class="service-history">
    <h2>Service History</h2>
    
    <!-- Vehicle Selection -->
    <div class="vehicle-filter">
      <label for="vehicle-select">Vehicle:</label>
      <select 
        id="vehicle-select" 
        v-model="selectedVehicleNumber"
        class="vehicle-select"
      >
        <option 
          v-for="vehicle in activeVehicles" 
          :key="vehicle.vehicle_number" 
          :value="vehicle.vehicle_number"
        >
          {{ vehicle.vehicle_number }} - {{ vehicle.vehicle_name }}
        </option>
      </select>
    </div>
    
    <!-- Service Status Summary -->
    <div class="service-status-summary" v-if="currentVehicle">
      <h3>Service Status</h3>
      <div class="status-cards">
        <div class="status-card" :class="getOilChangeStatusClass()">
          <div class="service-icon">🔧</div>
          <div class="service-info">
            <div class="service-name">Oil Change</div>
            <div class="service-status">{{ getOilChangeStatusMessage() }}</div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- Service History Table -->
    <div class="history-table-container">
      <table class="history-table">
        <thead>
          <tr>
            <th>Service Type</th>
            <th>Date</th>
            <th>Mileage</th>
            <th>Notes</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <!-- Oil Change Section -->
          <tr class="service-section-header">
            <td colspan="5">Oil Change</td>
          </tr>
          <tr 
            v-for="log in oilChangeLogs.slice(0, oilChangeDisplayCount)" 
            :key="log.id"
            :class="getOilChangeRowClass(log)"
          >
            <td class="service-type oil-change">
              <span class="service-icon">🔧</span>
              Oil Change
              <span class="time-info">{{ getTimeDisplay(log.date) }}</span>
            </td>
            <td>{{ formatDate(log.date) }}</td>
            <td>{{ log.mileage || 'N/A' }}</td>
            <td class="notes-cell">{{ log.notes || 'No details provided' }}</td>
            <td class="actions-cell">
              <button class="action-btn edit-btn" @click="editLog(log)">Edit</button>
              <button class="action-btn delete-btn" @click="deleteLog(log)">Delete</button>
            </td>
          </tr>
          <tr v-if="oilChangeLogs.length === 0">
            <td colspan="5" class="no-logs">No oil change history found</td>
          </tr>
          <tr v-if="oilChangeLogs.length > oilChangeDisplayCount" class="load-more-row">
            <td colspan="5">
              <button @click="loadMoreOilChanges" class="load-more-btn">
                Load more oil changes
              </button>
            </td>
          </tr>
          
          <!-- Tire Replacement Section -->
          <tr class="service-section-header">
            <td colspan="5">Tire Replacement</td>
          </tr>
          <tr 
            v-for="log in tireReplacementLogs.slice(0, otherServiceDisplayCount)" 
            :key="log.id"
          >
            <td class="service-type tire-replacement">
              <span class="service-icon">🛞</span>
              Tire Replacement
              <span class="time-info">{{ getTimeDisplay(log.date) }}</span>
            </td>
            <td>{{ formatDate(log.date) }}</td>
            <td>{{ log.mileage || 'N/A' }}</td>
            <td class="notes-cell">{{ log.notes || 'No details provided' }}</td>
            <td class="actions-cell">
              <button class="action-btn edit-btn" @click="editLog(log)">Edit</button>
              <button class="action-btn delete-btn" @click="deleteLog(log)">Delete</button>
            </td>
          </tr>
          <tr v-if="tireReplacementLogs.length === 0">
            <td colspan="5" class="no-logs">No tire replacement history found</td>
          </tr>
          <tr v-if="tireReplacementLogs.length > otherServiceDisplayCount" class="load-more-row">
            <td colspan="5">
              <button @click="loadMoreTireReplacements" class="load-more-btn">
                Load more tire replacements
              </button>
            </td>
          </tr>
          
          <!-- Brake Inspection Section -->
          <tr class="service-section-header">
            <td colspan="5">Brake Inspection</td>
          </tr>
          <tr 
            v-for="log in brakeInspectionLogs.slice(0, otherServiceDisplayCount)" 
            :key="log.id"
          >
            <td class="service-type brake-inspection">
              <span class="service-icon">🛑</span>
              Brake Inspection
              <span class="time-info">{{ getTimeDisplay(log.date) }}</span>
            </td>
            <td>{{ formatDate(log.date) }}</td>
            <td>{{ log.mileage || 'N/A' }}</td>
            <td class="notes-cell">{{ log.notes || 'No details provided' }}</td>
            <td class="actions-cell">
              <button class="action-btn edit-btn" @click="editLog(log)">Edit</button>
              <button class="action-btn delete-btn" @click="deleteLog(log)">Delete</button>
            </td>
          </tr>
          <tr v-if="brakeInspectionLogs.length === 0">
            <td colspan="5" class="no-logs">No brake inspection history found</td>
          </tr>
          <tr v-if="brakeInspectionLogs.length > otherServiceDisplayCount" class="load-more-row">
            <td colspan="5">
              <button @click="loadMoreBrakeInspections" class="load-more-btn">
                Load more brake inspections
              </button>
            </td>
          </tr>
          
          <!-- Battery Replacement Section -->
          <tr class="service-section-header">
            <td colspan="5">Battery Replacement</td>
          </tr>
          <tr 
            v-for="log in batteryReplacementLogs.slice(0, otherServiceDisplayCount)" 
            :key="log.id"
          >
            <td class="service-type battery-replacement">
              <span class="service-icon">🔋</span>
              Battery Replacement
              <span class="time-info">{{ getTimeDisplay(log.date) }}</span>
            </td>
            <td>{{ formatDate(log.date) }}</td>
            <td>{{ log.mileage || 'N/A' }}</td>
            <td class="notes-cell">{{ log.notes || 'No details provided' }}</td>
            <td class="actions-cell">
              <button class="action-btn edit-btn" @click="editLog(log)">Edit</button>
              <button class="action-btn delete-btn" @click="deleteLog(log)">Delete</button>
            </td>
          </tr>
          <tr v-if="batteryReplacementLogs.length === 0">
            <td colspan="5" class="no-logs">No battery replacement history found</td>
          </tr>
          <tr v-if="batteryReplacementLogs.length > otherServiceDisplayCount" class="load-more-row">
            <td colspan="5">
              <button @click="loadMoreBatteryReplacements" class="load-more-btn">
                Load more battery replacements
              </button>
            </td>
          </tr>
          
          <!-- General Service Section -->
          <tr class="service-section-header">
            <td colspan="5">General Service</td>
          </tr>
          <tr 
            v-for="log in generalServiceLogs.slice(0, otherServiceDisplayCount)" 
            :key="log.id"
          >
            <td class="service-type general-service">
              <span class="service-icon">🔨</span>
              General Service
              <span class="time-info">{{ getTimeDisplay(log.date) }}</span>
            </td>
            <td>{{ formatDate(log.date) }}</td>
            <td>{{ log.mileage || 'N/A' }}</td>
            <td class="notes-cell">{{ log.notes || 'No details provided' }}</td>
            <td class="actions-cell">
              <button class="action-btn edit-btn" @click="editLog(log)">Edit</button>
              <button class="action-btn delete-btn" @click="deleteLog(log)">Delete</button>
            </td>
          </tr>
          <tr v-if="generalServiceLogs.length === 0">
            <td colspan="5" class="no-logs">No general service history found</td>
          </tr>
          <tr v-if="generalServiceLogs.length > otherServiceDisplayCount" class="load-more-row">
            <td colspan="5">
              <button @click="loadMoreGeneralServices" class="load-more-btn">
                Load more general services
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    
    <!-- Fleet Alerts Section -->
    <div class="fleet-alerts" v-if="vehiclesNeedingOilChange.length > 0">
      <div class="alerts-header">
        <h3>Oil Change Alerts</h3>
        <button class="see-all-btn" @click="showAllAlerts = true">See All</button>
      </div>
      <div class="alert-cards">
        <div 
          v-for="vehicle in vehiclesNeedingOilChange" 
          :key="vehicle.vehicle_number"
          class="alert-card"
          :class="getOilChangeAlertClass(vehicle)"
          @click="switchToVehicle(vehicle.vehicle_number)"
        >
          <div class="vehicle-info">
            <div class="plate-number">{{ vehicle.vehicle_number }}</div>
            <div class="model">{{ vehicle.vehicle_name }}</div>
          </div>
          <div class="days-info">
            <div class="status">{{ getFleetVehicleStatus(vehicle) }}</div>
            <div class="threshold-info">Service due: {{ getServiceDueInfo(vehicle) }}</div>
          </div>
        </div>
      </div>
    </div>

    <!-- All Alerts Popup -->
    <div v-if="showAllAlerts" class="modal-overlay" @click.self="showAllAlerts = false">
      <div class="modal-content">
        <div class="modal-header">
          <h3>All Oil Change Alerts</h3>
          <button class="close-btn" @click="showAllAlerts = false">×</button>
        </div>
        <div class="modal-body">
          <table class="alerts-table">
            <thead>
              <tr>
                <th>Vehicle</th>
                <th>Name</th>
                <th>Status</th>
                <th>Days Since Last</th>
                <th>Threshold</th>
              </tr>
            </thead>
            <tbody>
              <tr 
                v-for="vehicle in allOilChangeAlerts" 
                :key="vehicle.vehicle_number"
                :class="getOilChangeAlertClass(vehicle)"
                @click="switchToVehicle(vehicle.vehicle_number)"
              >
                <td>{{ vehicle.vehicle_number }}</td>
                <td>{{ vehicle.vehicle_name }}</td>
                <td>{{ getFleetVehicleStatus(vehicle) }}</td>
                <td>{{ vehicle.days_since_last_change }}</td>
                <td>{{ vehicle.threshold_days }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<!-- Script Section -->
<script setup>
import { ref, computed, watch, onMounted } from 'vue';
import { createClient } from '@supabase/supabase-js';

// Supabase setup
const SUPABASE_URL = 'https://wmhpulllzyarvfatyfuw.supabase.co';
const SUPABASE_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6IndtaHB1bGxsenlhcnZmYXR5ZnV3Iiwicm9sZSI6InNlcnZpY2Vfcm9sZSIsImlhdCI6MTc0MjcyODU5NiwiZXhwIjoyMDU4MzA0NTk2fQ.PY8wgMW8hx-hRAsnKr1wTXh3PqiMavav0ljB3SvnMps';
const supabase = createClient(SUPABASE_URL, SUPABASE_KEY);

// State
const vehicles = ref([]);
const serviceLogs = ref([]);
const oilChangeDueData = ref([]);
const isLoading = ref(true);
const error = ref(null);
const showAllAlerts = ref(false);

// Default to first vehicle
const selectedVehicleNumber = ref('');

// Display count variables
const oilChangeDisplayCount = ref(1);
const otherServiceDisplayCount = ref(3);

// Fetch vehicle data
const fetchVehicles = async () => {
  try {
    const { data, error: fetchError } = await supabase
      .from('ddhaul_vehicles')
      .select('*')
      .eq('status', 'active');
    
    if (fetchError) throw fetchError;
    vehicles.value = data || [];
    
    if (vehicles.value.length > 0 && !selectedVehicleNumber.value) {
      selectedVehicleNumber.value = vehicles.value[0].vehicle_number;
    }
  } catch (err) {
    console.error('Error fetching vehicles:', err);
    error.value = 'Failed to load vehicles';
  }
};

// Fetch maintenance logs
const fetchServiceLogs = async () => {
  try {
    const { data, error: fetchError } = await supabase
      .from('ddhaul_service_logs')
      .select('*');
    
    if (fetchError) throw fetchError;
    serviceLogs.value = data || [];
  } catch (err) {
    console.error('Error fetching service logs:', err);
    error.value = 'Failed to load service logs';
  }
};

// Fetch oil change due data
const fetchOilChangeDueData = async () => {
  try {
    const { data, error: fetchError } = await supabase
      .from('ddhaul_oil_change_due_view')
      .select('*');
    
    if (fetchError) throw fetchError;
    oilChangeDueData.value = data || [];
  } catch (err) {
    console.error('Error fetching oil change due data:', err);
    error.value = 'Failed to load oil change due data';
  } finally {
    isLoading.value = false;
  }
};

// Only active vehicles
const activeVehicles = computed(() => {
  return vehicles.value.filter(v => v.status === 'active');
});

// Current vehicle
const currentVehicle = computed(() => {
  return vehicles.value.find(v => v.vehicle_number === selectedVehicleNumber.value) || null;
});

// Current vehicle oil change status
const currentVehicleOilChangeStatus = computed(() => {
  const vehicleData = oilChangeDueData.value.find(v => v.vehicle_number === selectedVehicleNumber.value);
  if (!vehicleData) return { days: 0, threshold: 90, status: 'unknown' };
  
  const daysSinceService = vehicleData.days_since_last_change;
  const threshold = vehicleData.threshold_days;
  
  let status = 'good';
  if (daysSinceService >= threshold) {
    status = 'overdue';
  } else if (daysSinceService >= threshold - 10) {
    status = 'due-soon';
  }
  
  return {
    days: daysSinceService,
    threshold: threshold,
    status: status,
    daysRemaining: threshold - daysSinceService
  };
});

// Filter logs by vehicle and service type
const vehicleLogs = computed(() => {
  if (!selectedVehicleNumber.value) return [];
  return serviceLogs.value.filter(log => log.vehicle_number === selectedVehicleNumber.value);
});

// Sorted logs by service type
const oilChangeLogs = computed(() => {
  return vehicleLogs.value
    .filter(log => log.service_type === 'oil-change')
    .sort((a, b) => new Date(b.date) - new Date(a.date));
});

const brakeInspectionLogs = computed(() => {
  return vehicleLogs.value
    .filter(log => log.service_type === 'brake-inspection')
    .sort((a, b) => new Date(b.date) - new Date(a.date));
});

const tireReplacementLogs = computed(() => {
  return vehicleLogs.value
    .filter(log => log.service_type === 'tire-replacement')
    .sort((a, b) => new Date(b.date) - new Date(a.date));
});

const batteryReplacementLogs = computed(() => {
  return vehicleLogs.value
    .filter(log => log.service_type === 'battery-replacement')
    .sort((a, b) => new Date(b.date) - new Date(a.date));
});

const generalServiceLogs = computed(() => {
  return vehicleLogs.value
    .filter(log => log.service_type === 'general-service')
    .sort((a, b) => new Date(b.date) - new Date(a.date));
});

// All oil change alerts (including those not yet due soon)
const allOilChangeAlerts = computed(() => {
  return oilChangeDueData.value
    .filter(vehicle => vehicle.days_since_last_change !== 9999)
    .sort((a, b) => b.days_since_last_change - a.days_since_last_change);
});

// Vehicles needing oil change (directly from view)
const vehiclesNeedingOilChange = computed(() => {
  return oilChangeDueData.value
    .filter(vehicle => {
      // Only include vehicles with actual oil change history (not the 9999 placeholder)
      return vehicle.days_since_last_change !== 9999 && 
             vehicle.days_since_last_change >= vehicle.threshold_days - 10;
    })
    .sort((a, b) => b.days_since_last_change - a.days_since_last_change)
    .slice(0, 6); // Show max 6 in the cards view
});

// Load more functions
const loadMoreOilChanges = () => {
  oilChangeDisplayCount.value += 2;
};

const loadMoreBrakeInspections = () => {
  otherServiceDisplayCount.value += 2;
};

const loadMoreTireReplacements = () => {
  otherServiceDisplayCount.value += 2;
};

const loadMoreBatteryReplacements = () => {
  otherServiceDisplayCount.value += 2;
};

const loadMoreGeneralServices = () => {
  otherServiceDisplayCount.value += 2;
};

// Format date nicely
const formatDate = (dateString) => {
  const date = new Date(dateString);
  return date.toLocaleDateString('en-US', { 
    year: 'numeric', 
    month: 'short', 
    day: 'numeric' 
  });
};

// Calculate days since a given date
const getDaysSince = (dateString) => {
  const date = new Date(dateString);
  const today = new Date();
  const diffTime = Math.abs(today - date);
  const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24));
  return diffDays;
};

// Get human readable time display (e.g. "3 months ago" instead of "90 days ago")
const getTimeDisplay = (dateString) => {
  const days = getDaysSince(dateString);
  
  if (days === 0) return 'Today';
  if (days === 1) return 'Yesterday';
  if (days < 7) return `${days} days ago`;
  if (days < 14) return '1 week ago';
  if (days < 30) return `${Math.floor(days / 7)} weeks ago`;
  if (days < 60) return '1 month ago';
  if (days < 365) return `${Math.floor(days / 30)} months ago`;
  if (days < 730) return '1 year ago';
  return `${Math.floor(days / 365)} years ago`;
};

// Get oil change status message for the current vehicle
const getOilChangeStatusMessage = () => {
  const status = currentVehicleOilChangeStatus.value;
  
  if (status.status === 'unknown') return 'No data available';
  if (status.status === 'overdue') return `OVERDUE - ${status.days} days since last service`;
  if (status.status === 'due-soon') return `DUE SOON - Service due in ${status.daysRemaining} days`;
  return `Good - ${status.days} days since last service`;
};

// Get CSS class for oil change status
const getOilChangeStatusClass = () => {
  const status = currentVehicleOilChangeStatus.value.status;
  if (status === 'overdue') return 'status-overdue';
  if (status === 'due-soon') return 'status-due-soon';
  return 'status-good';
};

// Get CSS class for oil change row
const getOilChangeRowClass = (log) => {
  const daysSince = getDaysSince(log.date);
  const threshold = currentVehicleOilChangeStatus.value.threshold;
  
  if (daysSince >= threshold) return 'alert-red';
  if (daysSince >= threshold - 10) return 'alert-yellow';
  return '';
};

// Get CSS class for fleet oil change alert
const getOilChangeAlertClass = (vehicle) => {
  if (vehicle.days_since_last_change >= vehicle.threshold_days) return 'alert-red';
  if (vehicle.days_since_last_change >= vehicle.threshold_days - 10) return 'alert-yellow';
  return '';
};

// Get fleet vehicle status message
const getFleetVehicleStatus = (vehicle) => {
  if (vehicle.days_since_last_change >= vehicle.threshold_days) {
    return 'OVERDUE';
  }
  if (vehicle.days_since_last_change >= vehicle.threshold_days - 10) {
    return 'DUE SOON';
  }
  return 'OK';
};

// Get service due info
const getServiceDueInfo = (vehicle) => {
  const daysRemaining = vehicle.threshold_days - vehicle.days_since_last_change;
  
  if (daysRemaining <= 0) {
    return `${Math.abs(daysRemaining)} days overdue`;
  }
  return `in ${daysRemaining} days`;
};

// Switch to a specific vehicle
const switchToVehicle = (vehicleNumber) => {
  selectedVehicleNumber.value = vehicleNumber;
  showAllAlerts.value = false;
};

// Edit log function
const editLog = (log) => {
  console.log('Editing log:', log);
  // Implement your edit functionality here
  alert(`Edit functionality for log ${log.id} would go here`);
};

// Delete log function
const deleteLog = async (log) => {
  if (confirm('Are you sure you want to delete this service record?')) {
    try {
      const { error } = await supabase
        .from('ddhaul_service_logs')
        .delete()
        .eq('id', log.id);
      
      if (error) throw error;
      
      // Refresh the data
      await fetchServiceLogs();
    } catch (err) {
      console.error('Error deleting log:', err);
      alert('Failed to delete service record');
    }
  }
};

// Reset display counts when vehicle changes
watch(selectedVehicleNumber, () => {
  oilChangeDisplayCount.value = 1;
  otherServiceDisplayCount.value = 3;
});

// Load data on component mount
onMounted(async () => {
  await Promise.all([
    fetchVehicles(),
    fetchServiceLogs(),
    fetchOilChangeDueData()
  ]);
});
</script>

<!-- Style Section -->
<style lang="scss" scoped>
.service-history {
  background-color: #ffffff;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
  
  h2 {
    font-size: 1.25rem;
    font-weight: 600;
    color: #1e293b;
    margin-bottom: 0.75rem;
    padding-bottom: 0.5rem;
    border-bottom: 1px solid #e2e8f0;
  }
  
  h3 {
    font-size: 1rem;
    font-weight: 600;
    color: #1e293b;
    margin-bottom: 0.5rem;
  }
  
  .vehicle-filter {
    display: flex;
    align-items: center;
    margin-bottom: 0.75rem;
    
    label {
      font-weight: 500;
      margin-right: 0.5rem;
      color: #475569;
      font-size: 0.85rem;
    }
    
    .vehicle-select {
      flex: 1;
      max-width: 400px;
      padding: 0.4rem 0.5rem;
      border: 1px solid #cbd5e1;
      border-radius: 4px;
      font-size: 0.85rem;
      background-color: #f8fafc;
      appearance: none;
      background-image: url("data:image/svg+xml;charset=utf-8,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 20 20'%3E%3Cpath stroke='%236B7280' stroke-linecap='round' stroke-linejoin='round' stroke-width='1.5' d='m6 8 4 4 4-4'/%3E%3C/svg%3E");
      background-position: right 0.5rem center;
      background-repeat: no-repeat;
      background-size: 1em;
      padding-right: 2rem;
      
      &:focus {
        outline: none;
        border-color: #3b82f6;
        box-shadow: 0 0 0 2px rgba(59, 130, 246, 0.1);
      }
    }
  }
  
  // Service Status Summary
  .service-status-summary {
    margin-bottom: 1rem;
    
    .status-cards {
      display: flex;
      gap: 0.5rem;
      flex-wrap: wrap;
      
      .status-card {
        display: flex;
        align-items: center;
        padding: 0.5rem 0.75rem;
        border-radius: 6px;
        background-color: #f1f5f9;
        min-width: 200px;
        
        &.status-good {
          background-color: #dcfce7;
          border-left: 3px solid #22c55e;
        }
        
        &.status-due-soon {
          background-color: #fef9c3;
          border-left: 3px solid #eab308;
        }
        
        &.status-overdue {
          background-color: #fee2e2;
          border-left: 3px solid #ef4444;
        }
        
        .service-icon {
          font-size: 1.2rem;
          margin-right: 0.5rem;
        }
        
        .service-info {
          .service-name {
            font-weight: 600;
            font-size: 0.85rem;
          }
          
          .service-status {
            font-size: 0.75rem;
          }
        }
      }
    }
  }
  
  .history-table-container {
    overflow-x: auto;
    margin-bottom: 1rem;
    
    .history-table {
      width: 100%;
      border-collapse: separate;
      border-spacing: 0;
      white-space: nowrap;
      font-size: 0.8rem;
      
      th, td {
        padding: 0.35rem 0.5rem;
        text-align: left;
      }
      
      th {
        background-color: #f1f5f9;
        color: #475569;
        font-weight: 600;
        font-size: 0.8rem;
        position: sticky;
        top: 0;
        z-index: 1;
        border-bottom: 1px solid #e2e8f0;
      }
      
      td {
        border-bottom: 1px solid #e2e8f0;
      }
      
      .notes-cell {
        min-width: 200px;
        max-width: 400px;
        overflow-x: hidden;
        text-overflow: ellipsis;
      }

      .actions-cell {
        white-space: nowrap;
        
        .action-btn {
          padding: 0.2rem 0.4rem;
          margin-right: 0.3rem;
          font-size: 0.7rem;
          border-radius: 3px;
          cursor: pointer;
          transition: all 0.2s ease;
          
          &.edit-btn {
            background-color: #e0f2fe;
            color: #0369a1;
            border: 1px solid #bae6fd;
            
            &:hover {
              background-color: #bae6fd;
            }
          }
          
          &.delete-btn {
            background-color: #fee2e2;
            color: #b91c1c;
            border: 1px solid #fecaca;
            
            &:hover {
              background-color: #fecaca;
            }
          }
        }
      }
      
      .service-section-header {
        background-color: #f8fafc;
        font-weight: 500;
        color: #334155;
        
        td {
          padding: 0.25rem 0.5rem;
          font-size: 10px;
        }
      }
      
      .service-type {
        font-weight: 500;
        display: flex;
        align-items: center;
        white-space: nowrap;
        
        .service-icon {
          margin-right: 0.3rem;
          font-size: 0.9rem;
        }
        
        .time-info {
          margin-left: 0.5rem;
          font-size: 0.7rem;
          color: #64748b;
          font-weight: normal;
        }
        
        &.oil-change {
          color: #0369a1;
        }
        
        &.brake-inspection {
          color: #be123c;
        }
        
        &.tire-replacement {
          color: #15803d;
        }
        
        &.battery-replacement {
          color: #b45309;
        }
        
        &.general-service {
          color: #6d28d9;
        }
      }
      
      .no-logs {
        text-align: center;
        padding: 0.5rem;
        color: #94a3b8;
        font-style: italic;
        font-size: 8px;
      }
      
      .load-more-row {
        background-color: #f9fafb;
        
        td {
          text-align: center;
          padding: 0.25rem;
        }
        
        .load-more-btn {
          background-color: #f1f5f9;
          border: 1px solid #cbd5e1;
          border-radius: 4px;
          padding: 0.2rem 0.5rem;
          font-size: 0.7rem;
          color: #475569;
          cursor: pointer;
          transition: all 0.2s ease;
          
          &:hover {
            background-color: #e2e8f0;
          }
        }
      }
      
      tr.alert-yellow {
        background-color: #fef9c3;
        
        td {
          border-bottom-color: #fde047;
        }
      }
      
      tr.alert-red {
        background-color: #fee2e2;
        
        td {
          border-bottom-color: #fecaca;
        }
      }
    }
  }
  
  .fleet-alerts {
    .alerts-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 0.5rem;
      
      .see-all-btn {
        background-color: #e0f2fe;
        color: #0369a1;
        border: 1px solid #bae6fd;
        border-radius: 4px;
        padding: 0.2rem 0.5rem;
        font-size: 0.7rem;
        cursor: pointer;
        
        &:hover {
          background-color: #bae6fd;
        }
      }
    }
    
    .alert-cards {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
      gap: 0.5rem;
      
      .alert-card {
        border-radius: 6px;
        padding: 0.5rem;
        display: flex;
        justify-content: space-between;
        font-size: 0.8rem;
        cursor: pointer;
        transition: transform 0.2s ease;
        
        &:hover {
          transform: translateY(-2px);
        }
        
        &.alert-yellow {
          background-color: #fef9c3;
          border-left: 3px solid #eab308;
        }
        
        &.alert-red {
          background-color: #fee2e2;
          border-left: 3px solid #ef4444;
        }
        
        .vehicle-info {
          .plate-number {
            font-weight: 600;
            font-size: 0.9rem;
            color: #1e293b;
          }
          
          .model {
            color: #64748b;
            font-size: 0.75rem;
          }
        }
        
        .days-info {
          text-align: right;
          
          .status {
            font-weight: 600;
            font-size: 0.9rem;
          }
          
          .threshold-info {
            font-size: 0.7rem;
            color: #64748b;
          }
        }
      }
    }
  }

  /* Modal styles */
  .modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 1000;
  }

  .modal-content {
    background-color: white;
    border-radius: 8px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
    width: 80%;
    max-width: 800px;
    max-height: 80vh;
    overflow: auto;
  }

  .modal-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1rem;
    border-bottom: 1px solid #e2e8f0;
    
    h3 {
      margin: 0;
    }
    
    .close-btn {
      background: none;
      border: none;
      font-size: 1.5rem;
      cursor: pointer;
      color: #64748b;
      
      &:hover {
        color: #334155;
      }
    }
  }

  .modal-body {
    padding: 1rem;
  }

  .alerts-table {
    width: 100%;
    border-collapse: collapse;
    font-size: 0.8rem;
    
    th, td {
      padding: 0.5rem;
      text-align: left;
      border-bottom: 1px solid #e2e8f0;
    }
    
    th {
      background-color: #f1f5f9;
      font-weight: 600;
    }
    
    tr {
      cursor: pointer;
      
      &:hover {
        background-color: #f8fafc;
      }
      
      &.alert-yellow {
        background-color: #fef9c3;
      }
      
      &.alert-red {
        background-color: #fee2e2;
      }
    }
  }
}
</style>