<template>
  <div class="log-service-tab">
    <h2>Log New Service</h2>
    
    <form @submit.prevent="submitForm" class="service-form">
      <!-- Vehicle Selection -->
<div class="form-group">
  <label for="vehicle-select">Select Vehicle</label>
  <select 
    id="vehicle-select" 
    v-model="formData.vehicleNumber"
    required
    class="form-control"
  >
    <option value="" disabled>-- Select a vehicle --</option>
    <option 
      v-for="vehicle in activeVehicles" 
      :key="vehicle.vehicle_number" 
      :value="vehicle.vehicle_number"
    >
      {{ vehicle.vehicle_number }} ({{ vehicle.vehicle_name || 'Unnamed' }})
    </option>
  </select>
</div>
      
      <!-- Service Types -->
      <div class="form-group service-types">
        <label class="group-label">Select Service Type(s)</label>
        
        <div class="service-options">
          <div class="service-type" :class="{ active: formData.serviceTypes.oilChange }">
            <div class="service-header">
              <div class="checkbox-wrapper">
                <input 
                  type="checkbox" 
                  id="oil-change" 
                  v-model="formData.serviceTypes.oilChange"
                  @change="toggleNotes('oilChange')"
                >
                <label for="oil-change" class="checkbox-label">
                  <span class="service-icon">🔧</span>
                  Oil Change
                </label>
              </div>
            </div>
            
            <div v-if="formData.serviceTypes.oilChange" class="notes-input">
              <textarea 
                id="oil-change-notes"
                v-model="formData.notes.oilChange"
                placeholder="Enter details about the oil change..."
                rows="2"
              ></textarea>
            </div>
          </div>
          
          <div class="service-type" :class="{ active: formData.serviceTypes.brakeInspection }">
            <div class="service-header">
              <div class="checkbox-wrapper">
                <input 
                  type="checkbox" 
                  id="brake-inspection" 
                  v-model="formData.serviceTypes.brakeInspection"
                  @change="toggleNotes('brakeInspection')"
                >
                <label for="brake-inspection" class="checkbox-label">
                  <span class="service-icon">🛑</span>
                  Brake Inspection
                </label>
              </div>
            </div>
            
            <div v-if="formData.serviceTypes.brakeInspection" class="notes-input">
              <textarea 
                id="brake-inspection-notes"
                v-model="formData.notes.brakeInspection"
                placeholder="Enter details about the brake inspection..."
                rows="2"
              ></textarea>
            </div>
          </div>
          
          <div class="service-type" :class="{ active: formData.serviceTypes.tireReplacement }">
            <div class="service-header">
              <div class="checkbox-wrapper">
                <input 
                  type="checkbox" 
                  id="tire-replacement" 
                  v-model="formData.serviceTypes.tireReplacement"
                  @change="toggleNotes('tireReplacement')"
                >
                <label for="tire-replacement" class="checkbox-label">
                  <span class="service-icon">🛞</span>
                  Tire Replacement
                </label>
              </div>
            </div>
            
            <div v-if="formData.serviceTypes.tireReplacement" class="notes-input">
              <textarea 
                id="tire-replacement-notes"
                v-model="formData.notes.tireReplacement"
                placeholder="Enter details about the tire replacement..."
                rows="2"
              ></textarea>
            </div>
          </div>
          
          <div class="service-type" :class="{ active: formData.serviceTypes.batteryReplacement }">
            <div class="service-header">
              <div class="checkbox-wrapper">
                <input 
                  type="checkbox" 
                  id="battery-replacement" 
                  v-model="formData.serviceTypes.batteryReplacement"
                  @change="toggleNotes('batteryReplacement')"
                >
                <label for="battery-replacement" class="checkbox-label">
                  <span class="service-icon">🔋</span>
                  Battery Replacement
                </label>
              </div>
            </div>
            
            <div v-if="formData.serviceTypes.batteryReplacement" class="notes-input">
              <textarea 
                id="battery-replacement-notes"
                v-model="formData.notes.batteryReplacement"
                placeholder="Enter details about the battery replacement..."
                rows="2"
              ></textarea>
            </div>
          </div>
          
          <div class="service-type" :class="{ active: formData.serviceTypes.generalService }">
            <div class="service-header">
              <div class="checkbox-wrapper">
                <input 
                  type="checkbox" 
                  id="general-service" 
                  v-model="formData.serviceTypes.generalService"
                  @change="toggleNotes('generalService')"
                >
                <label for="general-service" class="checkbox-label">
                  <span class="service-icon">🔨</span>
                  General Service
                </label>
              </div>
            </div>
            
            <div v-if="formData.serviceTypes.generalService" class="notes-input">
              <textarea 
                id="general-service-notes"
                v-model="formData.notes.generalService"
                placeholder="Enter details about the general service..."
                rows="2"
              ></textarea>
            </div>
          </div>
        </div>
      </div>
      
      <!-- Service Date -->
      <div class="form-group">
        <label for="service-date">Date of Service</label>
        <input 
          type="date" 
          id="service-date" 
          v-model="formData.date"
          required
          class="form-control"
          :max="today"
        >
      </div>
      
      <!-- Submit Button -->
      <div class="form-actions">
        <button 
          type="submit" 
          class="btn btn-primary"
          :disabled="!isFormValid"
        >
          <span class="btn-icon">✓</span>
          Submit Service Log
        </button>
      </div>
    </form>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { createClient } from '@supabase/supabase-js'

// Supabase setup
const SUPABASE_URL = 'https://wmhpulllzyarvfatyfuw.supabase.co'
const SUPABASE_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6IndtaHB1bGxsenlhcnZmYXR5ZnV3Iiwicm9sZSI6InNlcnZpY2Vfcm9sZSIsImlhdCI6MTc0MjcyODU5NiwiZXhwIjoyMDU4MzA0NTk2fQ.PY8wgMW8hx-hRAsnKr1wTXh3PqiMavav0ljB3SvnMps'
const supabase = createClient(SUPABASE_URL, SUPABASE_KEY)

// Reactive data
const vehicles = ref([])
const today = new Date().toISOString().split('T')[0]

const formData = ref({
  vehicleNumber: '',
  serviceTypes: {
    oilChange: false,
    brakeInspection: false,
    tireReplacement: false,
    batteryReplacement: false,
    generalService: false
  },
  notes: {
    oilChange: '',
    brakeInspection: '',
    tireReplacement: '',
    batteryReplacement: '',
    generalService: ''
  },
  date: today
})

// Fetch only active vehicles
onMounted(async () => {
  const { data, error } = await supabase
    .from('ddhaul_vehicles')
    .select('vehicle_number, vehicle_name, status')
    .eq('status', 'active')

  if (error) {
    console.error('Error fetching vehicles:', error)
  } else {
    vehicles.value = data
  }
})

const activeVehicles = computed(() => vehicles.value)

const isFormValid = computed(() => {
  const hasVehicle = !!formData.value.vehicleNumber
  const hasAtLeastOneService = Object.values(formData.value.serviceTypes).some(Boolean)
  const hasDate = !!formData.value.date
  return hasVehicle && hasAtLeastOneService && hasDate
})

const toggleNotes = (serviceType) => {
  if (!formData.value.serviceTypes[serviceType]) {
    formData.value.notes[serviceType] = ''
  }
}

const submitForm = async () => {
  if (!isFormValid.value) {
    alert('Please complete all required fields.')
    return
  }

  const serviceMap = {
    oilChange: 'oil-change',
    brakeInspection: 'brake-inspection',
    tireReplacement: 'tire-replacement',
    batteryReplacement: 'battery-replacement',
    generalService: 'general-service'
  }

  const entries = Object.keys(formData.value.serviceTypes)
    .filter(key => formData.value.serviceTypes[key])
    .map(key => ({
      vehicle_number: formData.value.vehicleNumber,
      service_type: serviceMap[key],
      date: formData.value.date,
      notes: formData.value.notes[key] || null
    }))

  const { error } = await supabase.from('ddhaul_service_logs').insert(entries)

  if (error) {
    console.error('Submission error:', error)
    alert('Failed to log service(s). Please try again.')
    return
  }

  alert('Service(s) logged successfully!')

  // Reset form
  formData.value = {
    vehicleNumber: '',
    serviceTypes: {
      oilChange: false,
      brakeInspection: false,
      tireReplacement: false,
      batteryReplacement: false,
      generalService: false
    },
    notes: {
      oilChange: '',
      brakeInspection: '',
      tireReplacement: '',
      batteryReplacement: '',
      generalService: ''
    },
    date: today
  }
}
</script>

<style lang="scss" scoped>
.log-service-tab {
  h2 {
    font-size: 1.6rem;
    font-weight: 600;
    margin-bottom: 1.5rem;
    color: #1e293b;
    padding-bottom: 0.75rem;
    border-bottom: 1px solid #e2e8f0;
  }
  
  .service-form {
    max-width: 700px;
    margin: 0 auto;
    
    .form-group {
      margin-bottom: 1.5rem;
      
      label {
        display: block;
        margin-bottom: 0.4rem;
        font-size: 0.9rem;
        font-weight: 500;
        color: #475569;
      }
      
      .form-control {
        width: 100%;
        padding: 0.65rem 0.75rem;
        border: 1px solid #cbd5e1;
        border-radius: 6px;
        font-size: 0.95rem;
        transition: all 0.2s ease;
        
        &:focus {
          outline: none;
          border-color: #3b82f6;
          box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.2);
        }
      }
      
      select.form-control {
        appearance: none;
        background-image: url("data:image/svg+xml;charset=utf-8,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 20 20'%3E%3Cpath stroke='%236B7280' stroke-linecap='round' stroke-linejoin='round' stroke-width='1.5' d='m6 8 4 4 4-4'/%3E%3C/svg%3E");
        background-position: right 0.75rem center;
        background-repeat: no-repeat;
        background-size: 1em;
        padding-right: 2.5rem;
      }
      
      &.service-types {
        background-color: #f8fafc;
        padding: 1.25rem;
        border-radius: 10px;
        border: 1px solid #e2e8f0;
        
        .group-label {
          font-size: 0.95rem;
          margin-bottom: 1rem;
          color: #1e293b;
          font-weight: 600;
        }
        
        .service-options {
          display: grid;
          grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
          gap: 1rem;
        }
        
        .service-type {
          background-color: white;
          border-radius: 8px;
          border: 1px solid #e2e8f0;
          transition: all 0.2s ease;
          overflow: hidden;
          
          &.active {
            border-color: #3b82f6;
            box-shadow: 0 2px 8px rgba(59, 130, 246, 0.15);
            
            .service-header {
              background-color: #ebf5ff;
            }
          }
          
          .service-header {
            padding: 0.8rem 1rem;
            background-color: #f9fafb;
            transition: background-color 0.2s ease;
          }
          
          .checkbox-wrapper {
            display: flex;
            align-items: center;
            
            input[type="checkbox"] {
              position: absolute;
              opacity: 0;
              width: 0;
              height: 0;
              
              &:checked + .checkbox-label::before {
                background-color: #3b82f6;
                border-color: #3b82f6;
                background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 20 20' fill='%23fff'%3E%3Cpath fill-rule='evenodd' d='M16.707 5.293a1 1 0 0 1 0 1.414l-8 8a1 1 0 0 1-1.414 0l-4-4a1 1 0 0 1 1.414-1.414L8 12.586l7.293-7.293a1 1 0 0 1 1.414 0z' clip-rule='evenodd'/%3E%3C/svg%3E");
                background-size: 70%;
                background-position: center;
                background-repeat: no-repeat;
              }
              
              &:focus + .checkbox-label::before {
                box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.3);
              }
            }
            
            .checkbox-label {
              display: flex;
              align-items: center;
              cursor: pointer;
              font-weight: 500;
              font-size: 0.95rem;
              
              &::before {
                content: '';
                width: 20px;
                height: 20px;
                border: 2px solid #cbd5e1;
                border-radius: 4px;
                margin-right: 0.75rem;
                transition: all 0.2s ease;
                flex-shrink: 0;
              }
              
              .service-icon {
                margin-right: 0.5rem;
                font-size: 1.1rem;
              }
            }
          }
          
          .notes-input {
            padding: 0.75rem;
            background-color: #fff;
            
            textarea {
              width: 100%;
              padding: 0.6rem;
              border: 1px solid #e2e8f0;
              border-radius: 6px;
              resize: vertical;
              font-size: 1rem;
              transition: all 0.2s ease;
              
              &:focus {
                outline: none;
                border-color: #3b82f6;
                box-shadow: 0 0 0 2px rgba(59, 130, 246, 0.1);
              }
              
              &::placeholder {
                color: #94a3b8;
                font-style: italic;
                font-size: 0.85rem;
              }
            }
          }
        }
      }
    }
    
    .form-actions {
      margin-top: 2rem;
      display: flex;
      justify-content: center;
      
      .btn {
        display: inline-flex;
        align-items: center;
        justify-content: center;
        gap: 0.5rem;
        padding: 0.75rem 1.5rem;
        font-size: 0.95rem;
        font-weight: 500;
        border-radius: 8px;
        transition: all 0.2s ease;
        border: none;
        cursor: pointer;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        
        &-primary {
          background-color: #2563eb;
          color: white;
          
          &:hover:not(:disabled) {
            background-color: #1d4ed8;
            transform: translateY(-1px);
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
          }
          
          &:active:not(:disabled) {
            transform: translateY(0);
          }
        }
        
        &:disabled {
          background-color: #cbd5e1;
          color: #94a3b8;
          cursor: not-allowed;
          box-shadow: none;
        }
        
        .btn-icon {
          font-size: 0.9rem;
        }
      }
    }
  }
}
</style>