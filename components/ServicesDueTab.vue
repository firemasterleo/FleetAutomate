
<template>
    <div class="services-due-tab">
      <h2>Services Due</h2>
      
      <div class="services-table">
        <table>
          <thead>
            <tr>
              <th>Vehicle</th>
              <th>Last Oil Change</th>
              <th>Days Since</th>
              <th>Threshold</th>
              <th>Status</th>
              <th>Action</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="vehicle in activeVehiclesNeedingService" :key="vehicle.id">
              <td>{{ vehicle.plateNumber }} ({{ vehicle.model }})</td>
              <td>{{ formatDate(vehicle.lastOilChange) }}</td>
              <td>{{ vehicle.daysSinceOilChange }} days</td>
              <td>{{ vehicle.oilChangeThreshold }} days</td>
              <td>
                <span :class="['status-badge', vehicle.serviceStatus]">
                  {{ vehicle.serviceStatus === 'overdue' ? '❗ Overdue' : '✅ OK' }}
                </span>
              </td>
              <td>
                <button class="btn-log-service" @click="navigateToLogService(vehicle.id)">
                  Log Service
                </button>
              </td>
            </tr>
            <tr v-if="activeVehiclesNeedingService.length === 0">
              <td colspan="6" class="no-services">No services due at this time!</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </template>
  
  <script setup>
  import { computed } from 'vue';
  
  const props = defineProps({
    vehicles: {
      type: Array,
      required: true
    }
  });
  
  // Compute the number of days since last oil change for each vehicle
  const vehiclesWithStatus = computed(() => {
    const today = new Date();
    
    return props.vehicles.map(vehicle => {
      const lastOilChange = new Date(vehicle.lastOilChange);
      const daysDifference = Math.floor((today - lastOilChange) / (1000 * 60 * 60 * 24));
      
      return {
        ...vehicle,
        daysSinceOilChange: daysDifference,
        serviceStatus: daysDifference > vehicle.oilChangeThreshold ? 'overdue' : 'ok'
      };
    });
  });
  
  // Filter to only show active vehicles needing service, sorted by status (overdue first)
  const activeVehiclesNeedingService = computed(() => {
    return vehiclesWithStatus.value
      .filter(v => v.status === 'active')
      .sort((a, b) => {
        // Sort overdue first
        if (a.serviceStatus === 'overdue' && b.serviceStatus !== 'overdue') return -1;
        if (a.serviceStatus !== 'overdue' && b.serviceStatus === 'overdue') return 1;
        
        // Then sort by days since oil change (highest first)
        return b.daysSinceOilChange - a.daysSinceOilChange;
      });
  });
  
  // Format date to be more readable
  const formatDate = (dateString) => {
    const date = new Date(dateString);
    return date.toLocaleDateString('en-US', { 
      year: 'numeric', 
      month: 'short', 
      day: 'numeric' 
    });
  };
  
  // Function to navigate to log service tab with the vehicle pre-selected
  const navigateToLogService = (vehicleId) => {
    // This would be implemented with your app's routing or state management
    // For now, we'll just emit an event that can be handled by the parent
    emit('navigate', { tab: 'log-service', vehicleId });
  };
  
  const emit = defineEmits(['navigate']);
  </script>
  
  <style lang="scss" scoped>
  .services-due-tab {
    h2 {
      margin-bottom: 20px;
      color: #2c3e50;
    }
    
    .services-table {
      width: 100%;
      overflow-x: auto;
      
      table {
        width: 100%;
        border-collapse: collapse;
        
        th, td {
          padding: 12px 15px;
          text-align: left;
          border-bottom: 1px solid #ddd;
        }
        
        th {
          background-color: #f8f9fa;
          font-weight: bold;
        }
        
        tr:hover {
          background-color: #f5f5f5;
        }
        
        .status-badge {
          display: inline-block;
          padding: 4px 8px;
          border-radius: 4px;
          font-weight: bold;
          
          &.overdue {
            background-color: #ffecec;
            color: #e63946;
          }
          
          &.ok {
            background-color: #e8f5e9;
            color: #2e7d32;
          }
        }
        
        .btn-log-service {
          background-color: #4caf50;
          color: white;
          border: none;
          padding: 6px 12px;
          border-radius: 4px;
          cursor: pointer;
          transition: background-color 0.2s;
          
          &:hover {
            background-color: #388e3c;
          }
        }
        
        .no-services {
          text-align: center;
          font-style: italic;
          color: #666;
        }
      }
    }
  }
  </style>