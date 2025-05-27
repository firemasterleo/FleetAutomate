<template>
  <div class="vehicle-maintenance">
    <TabNavigation 
      :tabs="tabs" 
      :activeTab="activeTab" 
      @tab-changed="handleTabChange" 
    />
    
    <!-- Tab Content -->
    <div class="tab-content">
      <!-- Log Service Tab -->
      <LogServiceTab 
        v-if="activeTab === 'log-service'" 
        :vehicles="vehicles" 
        :selectedVehicleId="selectedVehicleId"
        @service-logged="handleServiceLogged" 
      />
      
      <!-- Service Logs Tab -->
      <ServiceLogsTab 
        v-else-if="activeTab === 'service-logs'" 
        :vehicles="vehicles" 
        :maintenanceLogs="maintenanceLogs" 
      />
      
      <!-- Settings Tab -->
      <SettingsTab 
        v-else-if="activeTab === 'settings'" 
        :vehicles="vehicles" 
        @vehicle-added="addVehicle" 
        @vehicle-updated="updateVehicle"
        @vehicle-archived="archiveVehicle"
        @vehicle-restored="restoreVehicle"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import TabNavigation from '~/components/TabNavigation.vue';
import LogServiceTab from '~/components/LogServiceTab.vue';
import ServiceLogsTab from '~/components/ServiceLogsTab.vue';
import SettingsTab from '~/components/SettingsTab.vue';

// Tab configuration
const tabs = [
  { id: 'log-service', label: '🛠️ Log Service' },
  { id: 'service-logs', label: '📄 Service Logs' },
  { id: 'settings', label: '⚙️ Settings' }
];

const activeTab = ref('log-service');
const selectedVehicleId = ref(null);

// Vehicle data
const vehicles = ref([
  { 
    id: 1, 
    plateNumber: 'ABC-123K', 
    model: 'DAF XF', 
    oilChangeThreshold: 60, 
    lastOilChange: '2024-03-12',
    status: 'active'
  },
  { 
    id: 2, 
    plateNumber: 'XYZ-456L', 
    model: 'MAN TGX', 
    oilChangeThreshold: 90, 
    lastOilChange: '2024-02-20',
    status: 'archived'
  }
]);

// Maintenance logs
const maintenanceLogs = ref([
  { 
    id: 1, 
    vehicleId: 1, 
    type: 'oil-change', 
    date: '2024-03-12', 
    notes: 'Changed oil & filter' 
  },
  { 
    id: 2, 
    vehicleId: 1, 
    type: 'oil-change', 
    date: '2024-01-10', 
    notes: 'Routine check' 
  },
  { 
    id: 3, 
    vehicleId: 1, 
    type: 'brake-inspection', 
    date: '2024-02-28', 
    notes: 'Front pads inspected' 
  }
]);

// Methods
const handleTabChange = (tabId) => {
  activeTab.value = tabId;
  
  // Reset selectedVehicleId when changing tabs (except when navigating from a direction)
  if (tabId !== 'log-service') {
    selectedVehicleId.value = null;
  }
};

const handleNavigation = (navInfo) => {
  activeTab.value = navInfo.tab;
  if (navInfo.vehicleId) {
    selectedVehicleId.value = navInfo.vehicleId;
  }
};

const handleServiceLogged = (serviceData) => {
  const newLog = {
    id: maintenanceLogs.value.length + 1,
    ...serviceData
  };
  
  maintenanceLogs.value.push(newLog);
  
  // Update last service date if it's an oil change
  if (serviceData.type === 'oil-change') {
    const vehicle = vehicles.value.find(v => v.id === serviceData.vehicleId);
    if (vehicle) {
      vehicle.lastOilChange = serviceData.date;
    }
  }
};

const addVehicle = (vehicleData) => {
  const newVehicle = {
    id: vehicles.value.length + 1,
    ...vehicleData,
    status: 'active'
  };
  vehicles.value.push(newVehicle);
};

const updateVehicle = (updatedVehicle) => {
  const index = vehicles.value.findIndex(v => v.id === updatedVehicle.id);
  if (index !== -1) {
    vehicles.value[index] = { ...vehicles.value[index], ...updatedVehicle };
  }
};

const archiveVehicle = (vehicleId) => {
  const vehicle = vehicles.value.find(v => v.id === vehicleId);
  if (vehicle) {
    vehicle.status = 'archived';
  }
};

const restoreVehicle = (vehicleId) => {
  const vehicle = vehicles.value.find(v => v.id === vehicleId);
  if (vehicle) {
    vehicle.status = 'active';
  }
};
</script>

<style lang="scss" scoped>
.vehicle-maintenance {
  max-width: 1100px;
  margin: 0 auto;
  // padding: 1.5rem;
  font-family: 'Inter', 'Segoe UI', sans-serif;
  color: #334155;
  background-color: #f8fafc;
  border-radius: 12px;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
  
  .tab-content {
    background-color: #fff;
    border-radius: 0 0 12px 12px;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.08);
    padding: 1.5rem;
    min-height: 450px;
    transition: all 0.3s ease;
  }
}

/* Global form styles */
.form-group {
  margin-bottom: 1.25rem;
  
  label {
    display: block;
    font-size: 0.875rem;
    font-weight: 500;
    margin-bottom: 0.35rem;
    color: #475569;
  }
  
  input[type="text"],
  input[type="number"],
  input[type="date"],
  select,
  textarea {
    width: 100%;
    padding: 0.65rem 0.75rem;
    border: 1px solid #cbd5e1;
    border-radius: 6px;
    font-size: 0.95rem;
    background-color: #fff;
    transition: border 0.2s ease, box-shadow 0.2s ease;
    
    &:focus {
      border-color: #3b82f6;
      box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.2);
      outline: none;
    }
  }
  
  textarea {
    min-height: 90px;
    resize: vertical;
  }
}

/* Button styles */
.btn {
  padding: 0.6rem 1.25rem;
  border-radius: 6px;
  font-size: 0.95rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
  border: none;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  
  &-primary {
    background-color: #2563eb;
    color: white;
    
    &:hover {
      background-color: #1d4ed8;
    }
  }
  
  &-secondary {
    background-color: #e2e8f0;
    color: #334155;
    
    &:hover {
      background-color: #cbd5e1;
    }
  }
  
  &-success {
    background-color: #10b981;
    color: white;
    
    &:hover {
      background-color: #059669;
    }
  }
  
  &-danger {
    background-color: #ef4444;
    color: white;
    
    &:hover {
      background-color: #dc2626;
    }
  }
  
  svg {
    width: 18px;
    height: 18px;
  }
}

/* Custom radio buttons */
.radio-group {
  display: flex;
  gap: 1rem;
  margin-top: 0.5rem;
  
  .radio-option {
    position: relative;
    
    input[type="radio"] {
      position: absolute;
      opacity: 0;
      width: 0;
      height: 0;
      
      &:checked + .radio-label {
        background-color: #e0f2fe;
        border-color: #3b82f6;
        color: #1e40af;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
        
        &::before {
          transform: scale(1);
        }
      }
      
      &:focus + .radio-label {
        box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.3);
      }
    }
    
    .radio-label {
      display: flex;
      align-items: center;
      gap: 0.5rem;
      padding: 0.75rem 1.25rem;
      border: 1px solid #cbd5e1;
      border-radius: 8px;
      font-size: 0.95rem;
      cursor: pointer;
      transition: all 0.2s ease;
      background-color: #f8fafc;
      
      &:hover {
        border-color: #94a3b8;
        background-color: #f1f5f9;
      }
      
      &::before {
        content: '';
        display: inline-block;
        width: 18px;
        height: 18px;
        border-radius: 50%;
        background-color: #3b82f6;
        transform: scale(0);
        transition: transform 0.15s ease;
      }
      
      .icon {
        font-size: 1.15rem;
        margin-right: 0.25rem;
      }
    }
  }
}

/* Table styles */
table {
  width: 100%;
  border-collapse: separate;
  border-spacing: 0;
  font-size: 0.95rem;
  
  th, td {
    padding: 0.75rem 1rem;
    text-align: left;
    border-bottom: 1px solid #e2e8f0;
  }
  
  th {
    background-color: #f8fafc;
    font-weight: 600;
    color: #475569;
    font-size: 0.875rem;
    text-transform: uppercase;
    letter-spacing: 0.05em;
  }
  
  tr:last-child td {
    border-bottom: none;
  }
  
  tr:hover td {
    background-color: #f1f5f9;
  }
  
  .status-badge {
    display: inline-block;
    padding: 0.25rem 0.5rem;
    border-radius: 4px;
    font-size: 0.75rem;
    font-weight: 500;
    text-transform: uppercase;
    
    &.active {
      background-color: #ecfdf5;
      color: #047857;
    }
    
    &.archived {
      background-color: #f1f5f9;
      color: #64748b;
    }
  }
}

/* Card styles */
.card {
  background-color: #fff;
  border-radius: 8px;
  padding: 1.25rem;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  margin-bottom: 1.5rem;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  
  &:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  }
  
  .card-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 1rem;
    
    h3 {
      font-size: 1.1rem;
      font-weight: 600;
      color: #1e293b;
      margin: 0;
    }
  }
  
  .card-body {
    font-size: 0.95rem;
  }
}

/* Alert component */
.alert {
  padding: 0.75rem 1rem;
  border-radius: 6px;
  margin-bottom: 1rem;
  display: flex;
  align-items: center;
  
  &-info {
    background-color: #e0f2fe;
    color: #0369a1;
    border-left: 4px solid #0ea5e9;
  }
  
  &-success {
    background-color: #ecfdf5;
    color: #047857;
    border-left: 4px solid #10b981;
  }
  
  &-warning {
    background-color: #fffbeb;
    color: #b45309;
    border-left: 4px solid #f59e0b;
  }
  
  &-error {
    background-color: #fef2f2;
    color: #b91c1c;
    border-left: 4px solid #ef4444;
  }
  
  .icon {
    margin-right: 0.5rem;
    font-size: 1.25rem;
  }
}

/* Empty state */
.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 3rem 1rem;
  text-align: center;
  
  .icon {
    font-size: 3rem;
    color: #cbd5e1;
    margin-bottom: 1rem;
  }
  
  h3 {
    font-size: 1.2rem;
    color: #475569;
    margin-bottom: 0.5rem;
  }
  
  p {
    color: #64748b;
    max-width: 400px;
    margin: 0 auto 1.5rem;
  }
}
</style>