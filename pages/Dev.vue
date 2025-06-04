<template>
    <div class="sectioncontainer">
      <div class="section">
        <!-- Header -->
        <div class="section1">
          <p>FleetAutomate</p>
          <h2>Vehicle Fleet Management System</h2>
          <div class="subtitle">Complete fleet management and compliance monitoring</div>
          
          <!-- Tab Navigation -->
          <div class="tab-navigation">
            <button 
              :class="['tab-button', { active: activeTab === 'management' }]"
              @click="activeTab = 'management'"
            >
              Fleet Management
            </button>
            <button 
              :class="['tab-button', { active: activeTab === 'compliance' }]"
              @click="activeTab = 'compliance'"
            >
              Compliance Dashboard
            </button>
          </div>
        </div>
  
        <!-- Fleet Management Tab -->
        <div v-show="activeTab === 'management'" class="section2">
          <div class="management-header">
            <h3 class="subheading">Fleet Management</h3>
            <button @click="showAddVehicleForm" class="add-vehicle-btn">
              + Add Vehicle
            </button>
          </div>
  
          <div v-if="loading" class="loader-container">
            <div class="loader"></div>
          </div>
  
          <div v-else class="table-container">
            <table class="management-table">
              <thead>
                <tr>
                  <th>Vehicle #</th>
                  <th>Vehicle Name</th>
                  <th>Driver</th>
                  <th>License Expiry</th>
                  <th>Insurance Expiry</th>
                  <th>Road Worthiness</th>
                  <th>Hackney Permit</th>
                  <th>State Carriage</th>
                  <th>Actions</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="vehicle in allVehicles" :key="vehicle.id">
                  <td>
                    <input 
                      v-if="editingVehicle === vehicle.id"
                      v-model="editForm.vehicle_number"
                      class="edit-input"
                    />
                    <span v-else>{{ vehicle.vehicle_number }}</span>
                  </td>
                  <td>
                    <input 
                      v-if="editingVehicle === vehicle.id"
                      v-model="editForm.vehicle_name"
                      class="edit-input"
                    />
                    <span v-else>{{ vehicle.vehicle_name }}</span>
                  </td>
                  <td>
                    <input 
                      v-if="editingVehicle === vehicle.id"
                      v-model="editForm.driver_name"
                      class="edit-input"
                    />
                    <span v-else>{{ vehicle.driver_name || 'N/A' }}</span>
                  </td>
                  <td>
                    <input 
                      v-if="editingVehicle === vehicle.id"
                      v-model="editForm.vehicle_license_expiry"
                      type="date"
                      class="edit-input"
                    />
                    <span v-else :class="getDateClass(vehicle.vehicle_license_expiry)">
                      {{ formatDate(vehicle.vehicle_license_expiry) }}
                    </span>
                  </td>
                  <td>
                    <input 
                      v-if="editingVehicle === vehicle.id"
                      v-model="editForm.insurance_expiry"
                      type="date"
                      class="edit-input"
                    />
                    <span v-else :class="getDateClass(vehicle.insurance_expiry)">
                      {{ formatDate(vehicle.insurance_expiry) }}
                    </span>
                  </td>
                  <td>
                    <input 
                      v-if="editingVehicle === vehicle.id"
                      v-model="editForm.road_worthiness_expiry"
                      type="date"
                      class="edit-input"
                    />
                    <span v-else :class="getDateClass(vehicle.road_worthiness_expiry)">
                      {{ formatDate(vehicle.road_worthiness_expiry) }}
                    </span>
                  </td>
                  <td>
                    <input 
                      v-if="editingVehicle === vehicle.id"
                      v-model="editForm.hackney_permit_expiry"
                      type="date"
                      class="edit-input"
                    />
                    <span v-else :class="getDateClass(vehicle.hackney_permit_expiry)">
                      {{ formatDate(vehicle.hackney_permit_expiry) }}
                    </span>
                  </td>
                  <td>
                    <input 
                      v-if="editingVehicle === vehicle.id"
                      v-model="editForm.state_carriage_expiry"
                      type="date"
                      class="edit-input"
                    />
                    <span v-else :class="getDateClass(vehicle.state_carriage_expiry)">
                      {{ formatDate(vehicle.state_carriage_expiry) }}
                    </span>
                  </td>
                  <td class="actions-cell">
                    <div v-if="editingVehicle === vehicle.id" class="action-buttons">
                      <button @click="saveVehicle" class="save-btn">Save</button>
                      <button @click="cancelEdit" class="cancel-btn">Cancel</button>
                    </div>
                    <div v-else class="action-buttons">
                      <button @click="editVehicle(vehicle)" class="edit-btn">Edit</button>
                      <button @click="deleteVehicle(vehicle.id)" class="delete-btn">Delete</button>
                    </div>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
  
          <!-- Add Vehicle Modal -->
          <div v-if="showingAddForm" class="modal-overlay" @click="hideAddVehicleForm">
            <div class="modal-content" @click.stop>
              <h3>Add New Vehicle</h3>
              <form @submit.prevent="addVehicle" class="add-vehicle-form">
                <div class="form-grid">
                  <div class="form-group">
                    <label>Vehicle Number</label>
                    <input v-model="addForm.vehicle_number" required />
                  </div>
                  <div class="form-group">
                    <label>Vehicle Name</label>
                    <input v-model="addForm.vehicle_name" required />
                  </div>
                  <div class="form-group">
                    <label>Driver Name</label>
                    <input v-model="addForm.driver_name" />
                  </div>
                  <div class="form-group">
                    <label>License Expiry</label>
                    <input v-model="addForm.vehicle_license_expiry" type="date" />
                  </div>
                  <div class="form-group">
                    <label>Insurance Expiry</label>
                    <input v-model="addForm.insurance_expiry" type="date" />
                  </div>
                  <div class="form-group">
                    <label>Road Worthiness Expiry</label>
                    <input v-model="addForm.road_worthiness_expiry" type="date" />
                  </div>
                </div>
                <div class="form-actions">
                  <button type="submit" class="save-btn">Add Vehicle</button>
                  <button type="button" @click="hideAddVehicleForm" class="cancel-btn">Cancel</button>
                </div>
              </form>
            </div>
          </div>
        </div>
  
        <!-- Compliance Dashboard Tab -->
        <div v-show="activeTab === 'compliance'" class="section2">
          <!-- Documents expiring in next 30 days -->
          <div class="compliance-section">
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
  
          <!-- Already expired documents -->
          <div class="compliance-section">
            <h3 class="subheading">Expired Documents</h3>
            <div v-if="loading" class="loader-container">
              <div class="loader"></div>
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
    </div>
  </template>
  
  <script setup>
  import { ref, computed, onMounted, reactive } from 'vue';
  import { createClient } from '@supabase/supabase-js';
  
  const SUPABASE_URL = 'https://wmhpulllzyarvfatyfuw.supabase.co';
  const SUPABASE_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6IndtaHB1bGxsenlhcnZmYXR5ZnV3Iiwicm9sZSI6InNlcnZpY2Vfcm9sZSIsImlhdCI6MTc0MjcyODU5NiwiZXhwIjoyMDU4MzA0NTk2fQ.PY8wgMW8hx-hRAsnKr1wTXh3PqiMavav0ljB3SvnMps';
  
  const supabase = createClient(SUPABASE_URL, SUPABASE_KEY);
  
  // State
  const activeTab = ref('management');
  const allVehicles = ref([]);
  const complianceData = ref([]);
  const loading = ref(true);
  const error = ref(null);
  const editingVehicle = ref(null);
  const showingAddForm = ref(false);
  
  // Forms
  const editForm = reactive({});
  const addForm = reactive({
    vehicle_number: '',
    vehicle_name: '',
    driver_name: '',
    vehicle_license_expiry: null,
    insurance_expiry: null,
    road_worthiness_expiry: null,
    hackney_permit_expiry: null,
    state_carriage_expiry: null,
    gra_permit_expiry: null,
    heavy_duty_permit_expiry: null,
    proof_of_ownership_expiry: null,
    tipper_permit_expiry: null
  });
  
  // Fetch all vehicles for management
  const fetchAllVehicles = async () => {
    loading.value = true;
    try {
      const { data, error: fetchError } = await supabase
        .from('fleetautomate_vehicles_docs')
        .select('*')
        .order('vehicle_number');
  
      if (fetchError) throw fetchError;
      allVehicles.value = data;
    } catch (err) {
      console.error('Error fetching vehicles:', err);
      error.value = err.message;
    } finally {
      loading.value = false;
    }
  };
  
  // Fetch compliance data from view
  const fetchComplianceData = async () => {
    try {
      const { data, error: fetchError } = await supabase
        .from('fleetautomate_compliance_view')
        .select('*')
        .order('vehicle_number');
  
      if (fetchError) throw fetchError;
      complianceData.value = data;
    } catch (err) {
      console.error('Error fetching compliance data:', err);
      error.value = err.message;
    }
  };
  
  // Vehicle Management Functions
  const showAddVehicleForm = () => {
    showingAddForm.value = true;
    resetAddForm();
  };
  
  const hideAddVehicleForm = () => {
    showingAddForm.value = false;
    resetAddForm();
  };
  
  const resetAddForm = () => {
    Object.keys(addForm).forEach(key => {
      addForm[key] = key.includes('expiry') ? null : '';
    });
  };
  
  const addVehicle = async () => {
    try {
      const { data, error: insertError } = await supabase
        .from('fleetautomate_vehicles_docs')
        .insert([addForm])
        .select();
  
      if (insertError) throw insertError;
      
      await fetchAllVehicles();
      await fetchComplianceData();
      hideAddVehicleForm();
      alert('Vehicle added successfully!');
    } catch (err) {
      console.error('Error adding vehicle:', err);
      alert('Error adding vehicle: ' + err.message);
    }
  };
  
  const editVehicle = (vehicle) => {
    editingVehicle.value = vehicle.id;
    Object.assign(editForm, vehicle);
  };
  
  const cancelEdit = () => {
    editingVehicle.value = null;
    Object.keys(editForm).forEach(key => delete editForm[key]);
  };
  
  const saveVehicle = async () => {
    try {
      const { error: updateError } = await supabase
        .from('fleetautomate_vehicles_docs')
        .update(editForm)
        .eq('id', editingVehicle.value);
  
      if (updateError) throw updateError;
      
      await fetchAllVehicles();
      await fetchComplianceData();
      cancelEdit();
      alert('Vehicle updated successfully!');
    } catch (err) {
      console.error('Error updating vehicle:', err);
      alert('Error updating vehicle: ' + err.message);
    }
  };
  
  const deleteVehicle = async (vehicleId) => {
    if (!confirm('Are you sure you want to delete this vehicle?')) return;
    
    try {
      const { error: deleteError } = await supabase
        .from('fleetautomate_vehicles_docs')
        .delete()
        .eq('id', vehicleId);
  
      if (deleteError) throw deleteError;
      
      await fetchAllVehicles();
      await fetchComplianceData();
      alert('Vehicle deleted successfully!');
    } catch (err) {
      console.error('Error deleting vehicle:', err);
      alert('Error deleting vehicle: ' + err.message);
    }
  };
  
  // Utility Functions
  const formatDate = (dateString) => {
    if (!dateString) return 'N/A';
    return new Date(dateString).toLocaleDateString();
  };
  
  const getDateClass = (dateString) => {
    if (!dateString) return '';
    const today = new Date();
    const expiry = new Date(dateString);
    const daysLeft = Math.ceil((expiry - today) / (1000 * 60 * 60 * 24));
    
    if (daysLeft <= 0) return 'expired-date';
    if (daysLeft <= 10) return 'critical-date';
    if (daysLeft <= 30) return 'warning-date';
    return '';
  };
  
  const getDocStatusClass = (daysLeft) => {
    if (daysLeft <= 0) return 'expired';
    if (daysLeft <= 10) return 'critical';
    return 'warning';
  };
  
  // Computed properties for compliance dashboard
  const expiringSoonVehicles = computed(() => {
    return complianceData.value
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
    return complianceData.value
      .filter(vehicle => 
        vehicle.soon_to_expire_docs?.some(doc => doc.days_left <= 0)
      );
  });
  
  // Initialize data
  onMounted(async () => {
    await fetchAllVehicles();
    await fetchComplianceData();
  });
  </script>
  
  <style lang="scss" scoped>
  @use "@/assets/sass/variables" as *;
  
  .sectioncontainer {
    background-color: rgb(70, 96, 181);
    width: 100%;
    min-height: 100vh;
    display: flex;
    color: $text-dark;
    
    .section {
      background-color: $bg-white;
      width: 90rem;
      max-width: 95vw;
      height: 100%;
      // margin-inline: auto;
      padding: 1rem;
  
      .section1 {
        color: $text-dark;
        margin-bottom: 2rem;
  
        p {
          font-size: 12px;
          padding-bottom: 1rem;
          padding-top: 1rem;
          color: #666;
        }
  
        h2 {
          font-size: 24px;
          margin-bottom: 0.5rem;
        }
  
        .subtitle {
          font-size: 14px;
          font-weight: 400;
          color: #666;
          margin-bottom: 2rem;
        }
  
        .tab-navigation {
          display: flex;
          gap: 1rem;
          border-bottom: 2px solid #e5e5e5;
          
          .tab-button {
            padding: 0.75rem 1.5rem;
            border: none;
            background: none;
            cursor: pointer;
            font-size: 14px;
            font-weight: 500;
            color: #666;
            border-bottom: 2px solid transparent;
            transition: all 0.3s ease;
  
            &:hover {
              color: rgb(70, 96, 181);
            }
  
            &.active {
              color: rgb(70, 96, 181);
              border-bottom-color: rgb(70, 96, 181);
            }
          }
        }
      }
  
      .section2 {
        .management-header {
          display: flex;
          justify-content: space-between;
          align-items: center;
          margin-bottom: 1rem;
  
          .subheading {
            font-size: 16px;
            font-weight: bold;
            margin: 0;
          }
  
          .add-vehicle-btn {
            padding: 0.5rem 1rem;
            background-color: rgb(173, 206, 88);
            color: white;
            border: none;
            border-radius: 0.5rem;
            cursor: pointer;
            font-weight: 500;
  
            &:hover {
              background-color: rgb(150, 180, 70);
            }
          }
        }
  
        .table-container {
          max-height: 60vh;
          overflow: auto;
          border-radius: 0.5rem;
          background-color: rgb(249, 249, 249);
          border: 1px solid #e5e5e5;
        }
  
        .management-table {
          width: 100%;
          border-collapse: collapse;
          font-size: 12px;
  
          th, td {
            padding: 0.75rem 0.5rem;
            text-align: left;
            border-bottom: 1px solid #ddd;
          }
  
          th {
            background-color: rgb(225, 230, 236);
            font-weight: 600;
            position: sticky;
            top: 0;
            z-index: 10;
          }
  
          .edit-input {
            width: 100%;
            padding: 0.25rem;
            border: 1px solid #ccc;
            border-radius: 3px;
            font-size: 12px;
          }
  
          .actions-cell {
            width: 120px;
          }
  
          .action-buttons {
            display: flex;
            gap: 0.5rem;
  
            button {
              padding: 0.25rem 0.5rem;
              border: none;
              border-radius: 3px;
              cursor: pointer;
              font-size: 11px;
              font-weight: 500;
            }
  
            .edit-btn {
              background-color: #007bff;
              color: white;
            }
  
            .delete-btn {
              background-color: #dc3545;
              color: white;
            }
  
            .save-btn {
              background-color: #28a745;
              color: white;
            }
  
            .cancel-btn {
              background-color: #6c757d;
              color: white;
            }
          }
  
          .expired-date {
            color: #dc3545;
            font-weight: 600;
          }
  
          .critical-date {
            color: #fd7e14;
            font-weight: 600;
          }
  
          .warning-date {
            color: #ffc107;
            font-weight: 600;
          }
        }
  
        .compliance-section {
          margin-bottom: 3rem;
  
          .subheading {
            font-size: 16px;
            font-weight: bold;
            margin-bottom: 1rem;
            padding: 1rem;
            position: sticky;
            top: 0;
            background-color: $bg-white;
            z-index: 10;
          }
  
          .vehicles-table {
            width: 100%;
            border-collapse: collapse;
            max-height: 20rem;
            overflow: auto;
            border-radius: 0.5rem;
            background-color: rgb(249, 249, 249);
  
            th, td {
              padding: 0.75rem;
              text-align: left;
              border-bottom: 1px solid #ddd;
              font-size: 12px;
            }
  
            th {
              background-color: rgb(225, 230, 236);
              position: sticky;
              top: 0;
              z-index: 5;
            }
  
            .doc-status-list {
              display: flex;
              flex-direction: row;
              gap: 0.5rem;
              flex-wrap: wrap;
            }
  
            .doc-status {
              display: inline-flex;
              gap: 0.25rem;
              padding: 0.25rem 0.5rem;
              border-radius: 5px;
              font-size: 11px;
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
        }
      }
  
      // Modal styles
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
        background: white;
        padding: 2rem;
        border-radius: 0.5rem;
        width: 90%;
        max-width: 600px;
        max-height: 80vh;
        overflow-y: auto;
  
        h3 {
          margin-bottom: 1.5rem;
          font-size: 18px;
        }
  
        .add-vehicle-form {
          .form-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 1rem;
            margin-bottom: 2rem;
          }
  
          .form-group {
            display: flex;
            flex-direction: column;
  
            label {
              font-size: 12px;
              font-weight: 600;
              margin-bottom: 0.25rem;
              color: #333;
            }
  
            input {
              padding: 0.5rem;
              border: 1px solid #ccc;
              border-radius: 0.25rem;
              font-size: 14px;
  
              &:focus {
                outline: none;
                border-color: rgb(70, 96, 181);
              }
            }
          }
  
          .form-actions {
            display: flex;
            gap: 1rem;
            justify-content: flex-end;
  
            button {
              padding: 0.75rem 1.5rem;
              border: none;
              border-radius: 0.25rem;
              cursor: pointer;
              font-weight: 500;
  
              &.save-btn {
                background-color: #28a745;
                color: white;
              }
  
              &.cancel-btn {
                background-color: #6c757d;
                color: white;
              }
            }
          }
        }
      }
    }
  }
  
  .loader-container {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 10rem;
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
    border: 4px solid rgb(168, 152, 152);
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
  
  @media (max-width: 800px) {
    .sectioncontainer .section {
      width: 100vw;
      padding-inline: 0.5rem;
  
      .section2 {
        .table-container {
          overflow-x: auto;
        }
  
        .management-header {
          flex-direction: column;
          gap: 1rem;
          align-items: stretch;
        }
      }
    }
  }
  
  </style>