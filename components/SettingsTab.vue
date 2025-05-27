<template>
  <div class="settings-tab">
    <h2>Settings</h2>
    
    <!-- Settings Navigation -->
    <div class="settings-nav">
      <div 
        :class="['settings-nav-item', { active: activeSettingsSection === 'vehicles' }]"
        @click="activeSettingsSection = 'vehicles'"
      >
        🚚 Vehicle Management
      </div>
      <div 
        :class="['settings-nav-item', { active: activeSettingsSection === 'thresholds' }]"
        @click="activeSettingsSection = 'thresholds'"
      >
        🔧 Maintenance Thresholds
      </div>
    </div>
    
    <!-- Vehicle Management Section -->
    <div v-if="activeSettingsSection === 'vehicles'" class="settings-section">
      <h3>Vehicle Management</h3>
      
      <!-- Vehicle Filtering -->
      <div class="filter-controls">
        <div class="filter-label">Filter:</div>
        <div 
          :class="['filter-option', { active: activeFilter === 'all' }]"
          @click="activeFilter = 'all'"
        >
          All
        </div>
        <div 
          :class="['filter-option', { active: activeFilter === 'active' }]"
          @click="activeFilter = 'active'"
        >
          Active
        </div>
        <div 
          :class="['filter-option', { active: activeFilter === 'decommissioned' }]"
          @click="activeFilter = 'decommissioned'"
        >
          Decommissioned
        </div>
      </div>
      
      <!-- Vehicle List -->
      <div class="vehicle-list">
        <div class="table-container">
          <table>
            <thead>
              <tr>
                <th>Vehicle Number</th>
                <th>Name</th>
                <th>Status</th>
                <th>Actions</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="vehicle in filteredVehicles" :key="vehicle.vehicle_number">
                <td>{{ vehicle.vehicle_number }}</td>
                <td>{{ vehicle.vehicle_name || '-' }}</td>
                <td>
                  <span :class="['status-badge', vehicle.status]">
                    {{ vehicle.status === 'active' ? 'Active' : 'Decommissioned' }}
                  </span>
                </td>
                <td class="actions">
                  <button 
                    @click="editVehicle(vehicle)" 
                    class="btn-edit"
                    v-if="vehicle.status === 'active'"
                  >
                    Edit
                  </button>
                  <button 
                    v-if="vehicle.status === 'active'"
                    @click="decommissionVehicle(vehicle.vehicle_number)"
                    class="btn-archive"
                  >
                    Decommission
                  </button>
                  <button 
                    v-else
                    @click="restoreVehicle(vehicle.vehicle_number)"
                    class="btn-restore"
                  >
                    Restore
                  </button>
                </td>
              </tr>
              <tr v-if="filteredVehicles.length === 0">
                <td colspan="4" class="no-vehicles">No vehicles found matching the current filter.</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
      
      <!-- Add/Edit Vehicle Form -->
      <div class="add-vehicle">
        <h4>{{ isEditing ? 'Edit Vehicle' : 'Add New Vehicle' }}</h4>
        <form @submit.prevent="saveVehicle" class="vehicle-form">
          <div class="form-group">
            <label for="vehicle-number">Vehicle Number</label>
            <input 
              type="text" 
              id="vehicle-number" 
              v-model="vehicleForm.vehicle_number"
              required
              class="form-control"
              placeholder="ABC-123K"
              :disabled="isEditing"
            >
          </div>
          
          <div class="form-group">
            <label for="vehicle-name">Vehicle Name (optional)</label>
            <input 
              type="text" 
              id="vehicle-name" 
              v-model="vehicleForm.vehicle_name"
              class="form-control"
              placeholder="DAF XF"
            >
          </div>
          
          <div class="form-group">
            <label for="oil-change-threshold">Oil Change Threshold (Days)</label>
            <input 
              type="number" 
              id="oil-change-threshold" 
              v-model="vehicleForm.oilChangeThreshold"
              required
              min="1"
              max="365"
              class="form-control"
            >
          </div>
          
          <div class="form-group">
            <label for="last-oil-change">Last Oil Change Date</label>
            <input 
              type="date" 
              id="last-oil-change" 
              v-model="vehicleForm.lastOilChange"
              required
              class="form-control"
              :max="today"
            >
          </div>
          
          <div class="form-actions">
            <button type="submit" class="btn-save">Save</button>
            <button 
              type="button" 
              class="btn-cancel" 
              v-if="isEditing"
              @click="cancelEdit"
            >
              Cancel
            </button>
          </div>
        </form>
      </div>
    </div>
    
    <!-- Thresholds Section -->
    <div v-else-if="activeSettingsSection === 'thresholds'" class="settings-section">
      <h3>Oil/Filter Change Thresholds</h3>
      
      <div class="thresholds-summary">
        <div class="table-container">
          <table>
            <thead>
              <tr>
                <th>Vehicle</th>
                <th>Threshold (days)</th>
                <th>Threshold (km)</th>
                <th>Last Update</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="threshold in thresholds" :key="threshold.id">
                <td>{{ threshold.vehicle_number }} {{ getVehicleName(threshold.vehicle_number) }}</td>
                <td>
                  <input 
                    type="number"
                    v-model="threshold.threshold_days"
                    min="1"
                    max="365"
                    class="threshold-input"
                  >
                </td>
                <td>
                  <input 
                    type="number"
                    v-model="threshold.threshold_km"
                    min="0"
                    class="threshold-input"
                  >
                </td>
                <td>{{ formatDate(threshold.last_service_date) || 'Not set' }}</td>
              </tr>
              <tr v-if="thresholds.length === 0">
                <td colspan="4" class="no-vehicles">No active vehicles found.</td>
              </tr>
            </tbody>
          </table>
        </div>
        
        <div class="threshold-actions">
          <button @click="saveThresholds" class="btn-save">Save Changes</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue';
import { createClient } from '@supabase/supabase-js';

// Supabase setup
const SUPABASE_URL = 'https://wmhpulllzyarvfatyfuw.supabase.co';
const SUPABASE_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6IndtaHB1bGxsenlhcnZmYXR5ZnV3Iiwicm9sZSI6InNlcnZpY2Vfcm9sZSIsImlhdCI6MTc0MjcyODU5NiwiZXhwIjoyMDU4MzA0NTk2fQ.PY8wgMW8hx-hRAsnKr1wTXh3PqiMavav0ljB3SvnMps';
const supabase = createClient(SUPABASE_URL, SUPABASE_KEY);

// Settings navigation
const activeSettingsSection = ref('vehicles');
const activeFilter = ref('all');

// Data
const vehicles = ref([]);
const thresholds = ref([]);
const isLoading = ref(true);
const errorMessage = ref('');

// Vehicle form
const vehicleForm = ref({
  vehicle_number: '',
  vehicle_name: '',
  oilChangeThreshold: 60,
  lastOilChange: new Date().toISOString().split('T')[0],
  status: 'active'
});

// Editing state
const isEditing = ref(false);

// Today's date
const today = new Date().toISOString().split('T')[0];

// Filtered vehicles
const filteredVehicles = computed(() => {
  if (activeFilter.value === 'all') {
    return vehicles.value;
  }
  return vehicles.value.filter(v => v.status === activeFilter.value);
});

// Initialize data
onMounted(async () => {
  await fetchVehicles();
  await fetchThresholds();
  isLoading.value = false;
});

// Fetch vehicles from Supabase
const fetchVehicles = async () => {
  try {
    const { data, error } = await supabase
      .from('ddhaul_vehicles')
      .select('*');
    
    if (error) throw error;
    vehicles.value = data || [];
  } catch (error) {
    console.error('Error fetching vehicles:', error);
    errorMessage.value = 'Failed to load vehicles.';
  }
};

// Fetch thresholds from Supabase
const fetchThresholds = async () => {
  try {
    const { data, error } = await supabase
      .from('ddhaul_service_thresholds')
      .select('*')
      .eq('service_type', 'oil-change');
    
    if (error) throw error;
    thresholds.value = data || [];
  } catch (error) {
    console.error('Error fetching thresholds:', error);
    errorMessage.value = 'Failed to load thresholds.';
  }
};

// Format date
const formatDate = (dateString) => {
  if (!dateString) return null;
  
  const date = new Date(dateString);
  return date.toLocaleDateString('en-US', { 
    year: 'numeric', 
    month: 'short', 
    day: 'numeric' 
  });
};

// Get vehicle name for display
const getVehicleName = (vehicleNumber) => {
  const vehicle = vehicles.value.find(v => v.vehicle_number === vehicleNumber);
  return vehicle && vehicle.vehicle_name ? `(${vehicle.vehicle_name})` : '';
};

// Edit vehicle
const editVehicle = (vehicle) => {
  // Get the threshold for this vehicle if it exists
  const vehicleThreshold = thresholds.value.find(t => 
    t.vehicle_number === vehicle.vehicle_number && t.service_type === 'oil-change'
  );
  
  vehicleForm.value = {
    vehicle_number: vehicle.vehicle_number,
    vehicle_name: vehicle.vehicle_name || '',
    oilChangeThreshold: vehicleThreshold ? vehicleThreshold.threshold_days : 60,
    lastOilChange: vehicleThreshold && vehicleThreshold.last_service_date 
      ? new Date(vehicleThreshold.last_service_date).toISOString().split('T')[0]
      : new Date().toISOString().split('T')[0],
    status: vehicle.status
  };
  
  isEditing.value = true;
};

// Cancel edit
const cancelEdit = () => {
  vehicleForm.value = {
    vehicle_number: '',
    vehicle_name: '',
    oilChangeThreshold: 60,
    lastOilChange: new Date().toISOString().split('T')[0],
    status: 'active'
  };
  
  isEditing.value = false;
};

// Save vehicle
const saveVehicle = async () => {
  try {
    if (isEditing.value) {
      // Update vehicle
      const { error: vehicleError } = await supabase
        .from('ddhaul_vehicles')
        .update({
          vehicle_name: vehicleForm.value.vehicle_name,
          status: vehicleForm.value.status
        })
        .eq('vehicle_number', vehicleForm.value.vehicle_number);
      
      if (vehicleError) throw vehicleError;
      
      // Update or insert threshold
      const thresholdData = {
        vehicle_number: vehicleForm.value.vehicle_number,
        service_type: 'oil-change',
        threshold_days: vehicleForm.value.oilChangeThreshold,
        last_service_date: vehicleForm.value.lastOilChange
      };
      
      const existingThreshold = thresholds.value.find(t => 
        t.vehicle_number === vehicleForm.value.vehicle_number && t.service_type === 'oil-change'
      );
      
      if (existingThreshold) {
        const { error: thresholdError } = await supabase
          .from('ddhaul_service_thresholds')
          .update(thresholdData)
          .eq('id', existingThreshold.id);
        
        if (thresholdError) throw thresholdError;
      } else {
        const { error: thresholdError } = await supabase
          .from('ddhaul_service_thresholds')
          .insert(thresholdData);
        
        if (thresholdError) throw thresholdError;
      }
    } else {
      // Add new vehicle
      const { error: vehicleError } = await supabase
        .from('ddhaul_vehicles')
        .insert({
          vehicle_number: vehicleForm.value.vehicle_number,
          vehicle_name: vehicleForm.value.vehicle_name,
          status: 'active'
        });
      
      if (vehicleError) throw vehicleError;
      
      // Add threshold
      const { error: thresholdError } = await supabase
        .from('ddhaul_service_thresholds')
        .insert({
          vehicle_number: vehicleForm.value.vehicle_number,
          service_type: 'oil-change',
          threshold_days: vehicleForm.value.oilChangeThreshold,
          last_service_date: vehicleForm.value.lastOilChange
        });
      
      if (thresholdError) throw thresholdError;
    }
    
    // Refresh data
    await fetchVehicles();
    await fetchThresholds();
    
    // Reset form
    cancelEdit();
  } catch (error) {
    console.error('Error saving vehicle:', error);
    alert('Failed to save vehicle: ' + error.message);
  }
};

// Decommission vehicle
const decommissionVehicle = async (vehicleNumber) => {
  try {
    const { error } = await supabase
      .from('ddhaul_vehicles')
      .update({ status: 'decommissioned' })
      .eq('vehicle_number', vehicleNumber);
    
    if (error) throw error;
    
    // Refresh vehicles
    await fetchVehicles();
  } catch (error) {
    console.error('Error decommissioning vehicle:', error);
    alert('Failed to decommission vehicle: ' + error.message);
  }
};

// Restore vehicle
const restoreVehicle = async (vehicleNumber) => {
  try {
    const { error } = await supabase
      .from('ddhaul_vehicles')
      .update({ status: 'active' })
      .eq('vehicle_number', vehicleNumber);
    
    if (error) throw error;
    
    // Refresh vehicles
    await fetchVehicles();
  } catch (error) {
    console.error('Error restoring vehicle:', error);
    alert('Failed to restore vehicle: ' + error.message);
  }
};

// Save thresholds
const saveThresholds = async () => {
  try {
    for (const threshold of thresholds.value) {
      const { error } = await supabase
        .from('ddhaul_service_thresholds')
        .update({
          threshold_days: threshold.threshold_days,
          threshold_km: threshold.threshold_km || null
        })
        .eq('id', threshold.id);
      
      if (error) throw error;
    }
    
    alert('Thresholds saved successfully!');
    await fetchThresholds();
  } catch (error) {
    console.error('Error saving thresholds:', error);
    alert('Failed to save thresholds: ' + error.message);
  }
};
</script>

<style lang="scss" scoped>
.settings-tab {
  max-width: 100%;
  
  h2 {
    margin-bottom: 20px;
    color: #2c3e50;
  }
  
  .settings-nav {
    display: flex;
    margin-bottom: 20px;
    border-bottom: 1px solid #ddd;
    
    .settings-nav-item {
      padding: 12px 24px;
      cursor: pointer;
      margin-right: 10px;
      margin-bottom: 5px; /* Add spacing for wrapped items */
      background-color: #f5f7fa;
      border: 1px solid #ddd;
      border-bottom: none;
      border-radius: 4px 4px 0 0;
      
      &:hover {
        background-color: #e9ecef;
      }
      
      &.active {
        background-color: #fff;
        position: relative;
        bottom: -1px;
        font-weight: bold;
      }
    }
  }
  
  .filter-controls {
    display: flex;
    align-items: center;
    margin-bottom: 15px;
    flex-wrap: wrap; /* Allow wrapping on small screens */
    
    .filter-label {
      margin-right: 10px;
      font-weight: bold;
      margin-bottom: 5px; /* Add spacing for wrapped items */
    }
    
    .filter-option {
      padding: 6px 12px;
      margin-right: 5px;
      margin-bottom: 5px; /* Add spacing for wrapped items */
      cursor: pointer;
      border-radius: 4px;
      background-color: #f5f7fa;
      
      &:hover {
        background-color: #e9ecef;
      }
      
      &.active {
        background-color: #4caf50;
        color: white;
      }
    }
  }
  
  .settings-section {
    padding: 20px;
    background-color: #fff;
    border-radius: 0 0 4px 4px;
    
    h3 {
      margin-top: 0;
      margin-bottom: 20px;
      color: #2c3e50;
      border-bottom: 1px solid #eee;
      padding-bottom: 10px;
    }

    .vehicle-list, .thresholds-summary {
      margin-bottom: 30px;
      
      /* Add table container for horizontal scrolling */
      .table-container {
        width: 100%;
        overflow-x: auto;
        -webkit-overflow-scrolling: touch; /* Smooth scrolling on iOS */
        margin-bottom: 15px;
      }
      
      table {
        width: 100%;
        min-width: 600px; /* Ensure a minimum width for the table */
        border-collapse: collapse;
        
        th, td {
          padding: 12px 15px;
          text-align: left;
          border-bottom: 1px solid #ddd;
          white-space: nowrap; /* Prevent text wrapping in cells */
        }
        
        th {
          background-color: #f8f9fa;
          font-weight: bold;
          position: sticky;
          top: 0;
          z-index: 1;
        }
        
        .status-badge {
          display: inline-block;
          padding: 4px 8px;
          border-radius: 4px;
          
          &.active {
            background-color: #e8f5e9;
            color: #2e7d32;
          }
          
          &.decommissioned {
            background-color: #f5f5f5;
            color: #757575;
          }
        }
        
        .actions {
          button {
            margin-right: 5px;
            margin-bottom: 5px; /* Add spacing when buttons wrap */
            border: none;
            padding: 4px 8px;
            border-radius: 4px;
            cursor: pointer;
            
            &.btn-edit {
              background-color: #2196f3;
              color: white;
              
              &:hover {
                background-color: #0d8aee;
              }
            }
            
            &.btn-archive {
              background-color: #ff9800;
              color: white;
              
              &:hover {
                background-color: #f57c00;
              }
            }
            
            &.btn-restore {
              background-color: #9e9e9e;
              color: white;
              
              &:hover {
                background-color: #757575;
              }
            }
          }
        }
        
        .no-vehicles {
          text-align: center;
          padding: 20px;
          font-style: italic;
          color: #666;
        }
      }
    }
    
    .add-vehicle {
      background-color: #f8f9fa;
      padding: 20px;
      border-radius: 8px;
      
      h4 {
        margin-top: 0;
        margin-bottom: 20px;
      }
      
      .vehicle-form {
        .form-group {
          margin-bottom: 15px;
          
          label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
          }
          
          .form-control {
            width: 100%;
            max-width: 100%; /* Ensure inputs don't overflow */
            padding: 8px;
            border: 1px solid #ddd;
            border-radius: 4px;
            box-sizing: border-box; /* Include padding in width calculation */
            
            &:focus {
              outline: none;
              border-color: #4caf50;
              box-shadow: 0 0 0 2px rgba(76, 175, 80, 0.2);
            }
            
            &:disabled {
              background-color: #f5f5f5;
              cursor: not-allowed;
            }
          }
        }
        
        .form-actions {
          margin-top: 20px;
          display: flex;
          flex-wrap: wrap; /* Allow wrapping on small screens */
          
          button {
            padding: 8px 16px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            margin-right: 10px;
            margin-bottom: 10px; /* Add spacing when buttons wrap */
            
            &.btn-save {
              background-color: #4caf50;
              color: white;
              
              &:hover {
                background-color: #388e3c;
              }
            }
            
            &.btn-cancel {
              background-color: #f44336;
              color: white;
              
              &:hover {
                background-color: #d32f2f;
              }
            }
          }
        }
      }
    }
    
    .thresholds-summary {
      .threshold-input {
        width: 80px;
        padding: 6px;
        border: 1px solid #ddd;
        border-radius: 4px;
        text-align: center;
        
        &:focus {
          outline: none;
          border-color: #4caf50;
        }
      }
      
      .threshold-actions {
        text-align: right;
        
        .btn-save {
          background-color: #4caf50;
          color: white;
          border: none;
          padding: 8px 16px;
          border-radius: 4px;
          cursor: pointer;
          
          &:hover {
            background-color: #388e3c;
          }
        }
      }
    }
  }
  
  /* Responsive adjustments */
  @media (max-width: 768px) {
    .settings-section {
      padding: 15px 10px;
    }
    
    .add-vehicle {
      padding: 15px !important;
    }
  }
}
</style>