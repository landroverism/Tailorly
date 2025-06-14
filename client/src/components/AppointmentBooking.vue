<template>
  <div>
    <!-- Appointment Booking Button -->
    <el-button 
      type="primary" 
      @click="openBookingDialog"
      class="booking-button"
    >
      <el-icon class="mr-2"><Calendar /></el-icon>
      Book Appointment
    </el-button>
    
    <!-- Appointment Booking Dialog -->
    <el-dialog
      v-model="dialogVisible"
      title="Book Appointment"
      class="appointment-dialog modern-dialog"
      width="90%"
      :fullscreen="false"
      top="5vh"
      append-to-body
      destroy-on-close
    >
      <el-form :model="appointmentForm" label-position="top" class="appointment-form">
        <!-- Service Selection -->
        <el-form-item label="Service Type" class="mb-4">
          <el-select 
            v-model="appointmentForm.service"             
            placeholder="Select a service"
            class="mobile-full-width custom-select light-select"
            size="large"
            popper-class="appointment-select-dropdown light-dropdown"
            :style="{ backgroundColor: '#FFFFFF', color: '#2E3A3F', border: '1px solid #D4AF37' }"
          >
            <el-option label="Measurement Taking" value="measurement" />
            <el-option label="Fitting Session" value="fitting" />
            <el-option label="Design Consultation" value="consultation" />
            <el-option label="Fabric Selection" value="fabric" />
          </el-select>
        </el-form-item>
        
        <!-- Date and Time Selection -->
        <div class="form-row">
          <el-form-item label="Date" class="mb-5">
            <el-date-picker
              v-model="appointmentForm.date"
              type="date"
              placeholder="Select date"
              format="YYYY/MM/DD"
              class="mobile-full-width"
              :disabled-date="disablePastDates"
              size="large"
              style="height: 48px;"
              popper-class="date-picker-dropdown"
            />
          </el-form-item>
          
          <el-form-item label="Time" class="mb-5">
            <el-select
              v-model="appointmentForm.time"
              placeholder="Select available time"
              class="mobile-full-width custom-select light-select"
              :disabled="!availableSlots.length"
              size="large"
              style="height: 48px; background-color: #FFFFFF !important; color: #2E3A3F !important; border: 1px solid #D4AF37 !important;"
              popper-class="time-select-dropdown light-dropdown"
            >
              <el-option
                v-for="(slot, index) in availableSlots"
                :key="index"
                :label="`${slot.time} - ${slot.endTime}`"
                :value="new Date(`2000-01-01T${slot.time}`)"
              />
            </el-select>
            <p v-if="appointmentForm.date && appointmentForm.service && !availableSlots.length" class="text-sm text-red-500 mt-2 px-2">
              No available slots for this date and service
            </p>
          </el-form-item>
        </div>
        
        <!-- Client Information -->
        <el-form-item label="Your Name" class="mb-5">
          <el-input 
            v-model="appointmentForm.name" 
            placeholder="Enter your full name"
            size="large"
            class="mobile-input"
            style="height: 48px;"
          />
        </el-form-item>
        
        <el-form-item label="Phone Number" class="mb-5">
          <el-input 
            v-model="appointmentForm.phone" 
            placeholder="Enter your phone number"
            size="large"
            class="mobile-input"
            style="height: 48px;"
          />
        </el-form-item>
        
        <!-- Additional Notes -->
        <el-form-item label="Additional Notes" class="mb-5">
          <el-input 
            v-model="appointmentForm.notes" 
            type="textarea" 
            :rows="3"
            placeholder="Any special requirements or preferences?"
            class="mobile-input"
            resize="none"
          />
        </el-form-item>
      </el-form>
      
      <template #footer>
        <div class="dialog-footer">
          <el-button 
            @click="dialogVisible = false" 
            class="cancel-btn"
            size="large"
            plain
          >
            Cancel
          </el-button>
          <el-button 
            type="primary" 
            @click="submitAppointment" 
            :loading="submitting"
            class="submit-btn"
            size="large"
          >
            Book Appointment
          </el-button>
        </div>
      </template>
    </el-dialog>
    
    <!-- Confirmation Dialog -->
    <el-dialog
      v-model="confirmationVisible"
      title="Appointment Confirmed"
      class="appointment-dialog confirmation-dialog modern-dialog"
      :width="windowWidth <= 768 ? '95%' : '400px'"
      top="10vh"
      :show-close="false"
    >
      <div class="text-center">
        <div class="success-icon-container mb-6">
          <el-icon class="success-icon"><CircleCheckFilled /></el-icon>
        </div>
        <h3 class="text-xl font-bold mb-3">Appointment Confirmed!</h3>
        <p class="text-gray-600 mb-5">We've sent the details to your phone.</p>
        
        <div class="appointment-details bg-gray-50 p-5 rounded-lg text-left mb-5 shadow-sm">
          <div class="flex items-center mb-3">
            <el-icon class="mr-3 text-primary"><Calendar /></el-icon>
            <p><strong>Service:</strong> {{ getServiceName(appointmentForm.service) }}</p>
          </div>
          <div class="flex items-center mb-3">
            <el-icon class="mr-3 text-primary"><Calendar /></el-icon>
            <p><strong>Date:</strong> {{ formatDate(appointmentForm.date) }}</p>
          </div>
          <div class="flex items-center">
            <el-icon class="mr-3 text-primary"><Clock /></el-icon>
            <p><strong>Time:</strong> {{ formatTime(appointmentForm.time) }}</p>
          </div>
        </div>
      </div>
      
      <template #footer>
        <div class="dialog-footer flex justify-center">
          <el-button type="primary" @click="confirmationVisible = false">
            Done
          </el-button>
        </div>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted, onUnmounted, watch } from 'vue';
import { Calendar, CircleCheckFilled, Clock } from '@element-plus/icons-vue';
import { ElMessage } from 'element-plus';
import appointmentService from '../services/appointment.service';
import { useUserStore } from '../stores/user';

// Get user store
const userStore = useUserStore();

// Track window width for responsive layout
const windowWidth = ref(window.innerWidth);

// Available time slots
const availableSlots = ref([]);

// Update window width on resize
const handleResize = () => {
  windowWidth.value = window.innerWidth;
};

// Add and remove event listeners
onMounted(() => {
  window.addEventListener('resize', handleResize);
});

onUnmounted(() => {
  window.removeEventListener('resize', handleResize);
});

// Dialog visibility states
const dialogVisible = ref(false);
const confirmationVisible = ref(false);
const submitting = ref(false);

// Appointment form data
const appointmentForm = reactive({
  service: '',
  date: null,
  time: null,
  name: '',
  phone: '',
  notes: ''
});

// Open booking dialog
const openBookingDialog = () => {
  dialogVisible.value = true;
};

// Disable past dates in date picker
const disablePastDates = (date) => {
  return date < new Date(new Date().setHours(0, 0, 0, 0));
};

// Fetch available time slots when date or service changes
const fetchAvailableSlots = async () => {
  if (!appointmentForm.date || !appointmentForm.service) {
    availableSlots.value = [];
    return;
  }
  
  try {
    const formattedDate = formatDateForApi(appointmentForm.date);
    const response = await appointmentService.getAvailableSlots(formattedDate, appointmentForm.service);
    availableSlots.value = response.data || [];
  } catch (error) {
    console.error('Error fetching available slots:', error);
    ElMessage.error('Failed to load available time slots');
    availableSlots.value = [];
  }
};

// Watch for changes to date or service
watch([() => appointmentForm.date, () => appointmentForm.service], () => {
  if (appointmentForm.date && appointmentForm.service) {
    fetchAvailableSlots();
  }
});

// Submit appointment
const submitAppointment = async () => {
  // Validate form
  if (!appointmentForm.service || !appointmentForm.date || !appointmentForm.time || !appointmentForm.name || !appointmentForm.phone) {
    // Show validation error
    ElMessage.error('Please fill in all required fields');
    return;
  }
  
  submitting.value = true;
  
  try {
    // Format date and time for API
    const formattedDate = formatDateForApi(appointmentForm.date);
    const formattedTime = formatTimeForApi(appointmentForm.time);
    
    // Create appointment data
    const appointmentData = {
      service_type: appointmentForm.service,
      date: formattedDate,
      time: formattedTime,
      client_name: appointmentForm.name,
      client_phone: appointmentForm.phone,
      notes: appointmentForm.notes || ''
    };
    
    // Call API to create appointment
    await appointmentService.createAppointment(appointmentData);
    
    // Close booking dialog
    dialogVisible.value = false;
    
    // Show confirmation
    confirmationVisible.value = true;
    
    // Reset form
    Object.keys(appointmentForm).forEach(key => {
      if (key !== 'name' && key !== 'phone') {
        appointmentForm[key] = key === 'service' ? '' : null;
      }
    });
  } catch (error) {
    console.error('Error booking appointment:', error);
    ElMessage.error('Failed to book appointment. Please try again.');
  } finally {
    submitting.value = false;
  }
};

// Format date for display
const formatDate = (date) => {
  if (!date) return '';
  
  const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' };
  return new Date(date).toLocaleDateString(undefined, options);
};

// Format time for display
const formatTime = (time) => {
  if (!time) return '';
  
  return new Date(time).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
};

// Format date for API (YYYY-MM-DD)
const formatDateForApi = (date) => {
  if (!date) return '';
  
  const d = new Date(date);
  const year = d.getFullYear();
  const month = String(d.getMonth() + 1).padStart(2, '0');
  const day = String(d.getDate()).padStart(2, '0');
  
  return `${year}-${month}-${day}`;
};

// Format time for API (HH:MM)
const formatTimeForApi = (time) => {
  if (!time) return '';
  
  const d = new Date(time);
  const hours = String(d.getHours()).padStart(2, '0');
  const minutes = String(d.getMinutes()).padStart(2, '0');
  
  return `${hours}:${minutes}`;
};

// Get service name from value
const getServiceName = (value) => {
  const services = {
    measurement: 'Measurement Taking',
    fitting: 'Fitting Session',
    consultation: 'Design Consultation',
    fabric: 'Fabric Selection'
  };
  
  return services[value] || value;
};
</script>

<style scoped>
/* Enhanced styles for appointment booking */
.booking-button {
  width: 100%;
  height: 48px;
  font-size: 16px;
  border-radius: 8px;
  background-color: #D4AF37 !important;
  border-color: #D4AF37 !important;
  color: #FFEFD5 !important;
  font-weight: 600 !important;
  box-shadow: 0 4px 12px rgba(212, 175, 55, 0.3) !important;
  transition: all 0.3s ease !important;
}

.booking-button:hover {
  background-color: #C9A227 !important;
  border-color: #C9A227 !important;
  transform: translateY(-1px) !important;
  box-shadow: 0 6px 16px rgba(212, 175, 55, 0.4) !important;
}

/* Dialog improvements for mobile */
.modern-dialog :deep(.el-dialog) {
  border-radius: 16px;
  overflow: hidden;
  margin: 0 auto !important;
  width: 92% !important;
  max-width: 450px;
  box-shadow: 0 8px 30px rgba(212, 175, 55, 0.2);
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  border: 1px solid #D4AF37;
}

.modern-dialog :deep(.el-dialog__header) {
  padding: 20px 20px 16px;
  margin-right: 0;
  text-align: center;
  border-bottom: 1px solid #D4AF37;
  background-color: #FFF8E7;
}

.modern-dialog :deep(.el-dialog__headerbtn) {
  position: absolute;
  top: 16px;
  right: 16px;
  width: 30px;
  height: 30px;
  font-size: 20px;
}

.modern-dialog :deep(.el-dialog__title) {
  font-size: 1.3rem;
  font-weight: 600;
  color: #D4AF37;
}

.modern-dialog :deep(.el-dialog__body) {
  padding: 20px;
  max-height: 60vh;
  overflow-y: auto;
  background-color: #FFEFD5;
}

.modern-dialog :deep(.el-dialog__footer) {
  padding: 16px 20px;
  border-top: 1px solid #D4AF37;
  background-color: #FFF8E7;
}

/* Form improvements */
.appointment-form :deep(.el-form-item__label) {
  font-weight: 500;
  color: #2E3A3F;
  padding-bottom: 8px;
  font-size: 16px;
}

.appointment-form :deep(.el-input__wrapper),
.appointment-form :deep(.el-textarea__inner) {
  border-radius: 8px;
  padding: 12px;
  box-shadow: 0 2px 6px rgba(212, 175, 55, 0.15);
  background-color: #FFFFFF !important;
  border: 1px solid rgba(212, 175, 55, 0.3) !important;
}

/* Form input enhancements */
.appointment-form :deep(.el-form-item__label) {
  font-size: 16px;
  font-weight: 500;
  padding-bottom: 8px;
  color: #2E3A3F;
}

.appointment-form :deep(.el-input__wrapper),
.appointment-form :deep(.el-textarea__inner) {
  box-shadow: 0 2px 6px rgba(212, 175, 55, 0.15) !important;
  border-radius: 10px !important;
  padding: 4px 12px !important;
  background-color: #FFFFFF !important;
  border: 1px solid rgba(212, 175, 55, 0.3) !important;
}

.appointment-form :deep(.el-input__wrapper.is-focus),
.appointment-form :deep(.el-textarea__inner:focus) {
  box-shadow: 0 0 0 2px #D4AF37 inset !important;
  border-color: #D4AF37 !important;
}

/* Mobile specific styles */
.mobile-full-width {
  width: 100% !important;
}

.mobile-input {
  width: 100% !important;
}

/* Footer buttons */
.dialog-footer {
  display: flex;
  justify-content: space-between;
  width: 100%;
  gap: 16px;
}

.cancel-btn,
.submit-btn {
  flex: 1;
  height: 50px;
  font-size: 16px;
  border-radius: 10px;
  padding: 0 24px;
  font-weight: 500;
  transition: all 0.3s ease;
}

.cancel-btn {
  background-color: transparent !important;
  border: 1px solid #D4AF37 !important;
  color: #2E3A3F !important;
}

.cancel-btn:hover {
  background-color: rgba(212, 175, 55, 0.1) !important;
  color: #2E3A3F !important;
}

.submit-btn {
  font-weight: 600;
  background-color: #D4AF37 !important;
  border-color: #D4AF37 !important;
  color: #FFEFD5 !important;
}

.submit-btn:hover {
  background-color: #C9A227 !important;
  border-color: #C9A227 !important;
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(212, 175, 55, 0.3);
}

/* Dropdown styling */
:deep(.el-popper.is-light) {
  border: 1px solid #D4AF37 !important;
  background-color: #FFF8E7 !important;
}

/* Fix for the dark dropdown trigger background */
:deep(.el-input__wrapper) {
  background-color: #FFFFFF !important;
  border: 1px solid rgba(212, 175, 55, 0.3) !important;
}

/* Fix for dropdown items */
:deep(.el-select-dropdown__item) {
  background-color: #FFF8E7 !important;
  color: #2E3A3F !important;
}

:deep(.el-select-dropdown__item.hover),
:deep(.el-select-dropdown__item:hover) {
  background-color: rgba(212, 175, 55, 0.15) !important;
}

:deep(.el-select-dropdown__item.selected) {
  background-color: rgba(212, 175, 55, 0.25) !important;
  color: #D4AF37 !important;
  font-weight: 600 !important;
}

:deep(.el-select-dropdown__item) {
  height: 40px !important;
  line-height: 40px !important;
  padding: 0 16px !important;
  color: #2E3A3F !important;
}

:deep(.el-select-dropdown__item.selected),
:deep(.el-select-dropdown__item.hover) {
  background-color: rgba(212, 175, 55, 0.1) !important;
}

:deep(.el-select-dropdown__item.selected) {
  color: #D4AF37 !important;
  font-weight: 600 !important;
}

.success-icon-container {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 70px;
  height: 70px;
  margin: 0 auto;
  background-color: #ecfdf5;
  border-radius: 50%;
}

.success-icon {
  font-size: 40px;
  color: #10b981;
}

.appointment-details {
  border-left: 3px solid #10b981;
}

/* Mobile optimizations */
@media (max-width: 640px) {
  .modern-dialog :deep(.el-dialog__body) {
    padding: 16px;
  }
  
  .appointment-form :deep(.el-form-item) {
    margin-bottom: 16px;
  }
  
  .mobile-full-width {
    width: 100% !important;
  }
  
  .form-row {
    gap: 12px !important;
  }
  
  .appointment-details {
    padding: 16px;
  }
}

@media (min-width: 640px) {
  .booking-button {
    width: auto;
  }
}
/* Direct targeting of the dark dropdown backgrounds */
:deep(.el-input__wrapper),
:deep(.el-select .el-input__wrapper),
:deep(.el-select .el-input .el-input__wrapper),
:deep(.el-select-dropdown__item),
:deep(.el-select-dropdown__list),
:deep(.el-scrollbar__view),
:deep(.el-scrollbar),
:deep(.el-select-dropdown),
:deep(.el-popper),
:deep(.el-select__popper) {
  background-color: #FFFFFF !important;
  color: #2E3A3F !important;
  box-shadow: none !important;
  border-color: #D4AF37 !important;
}

/* Direct override to prevent dark backgrounds */
.light-select :deep(*),
.light-dropdown :deep(*),
.light-dropdown :deep(.el-scrollbar),
.light-dropdown :deep(.el-select-dropdown__wrap),
.light-select :deep(.el-input__wrapper),
.light-select :deep(.el-input) {
  background-color: #FFFFFF !important;
  color: #2E3A3F !important;
}

/* Target Element Plus internal classes more aggressively */
::v-deep(.el-select-dropdown),
::v-deep(.el-scrollbar__view),
::v-deep(.el-select-dropdown__list),
::v-deep(.el-select-dropdown__item),
::v-deep(.el-popper),
.el-select-dropdown,
.el-scrollbar__view,
.el-select-dropdown__list,
.el-select-dropdown__item,
.el-popper {
  background-color: #FFFFFF !important;
  color: #2E3A3F !important;
  border-color: #D4AF37 !important;
}

/* Force override for Element Plus dark theme on selectors */
:deep(.el-select .el-input__wrapper),
:deep(.el-input .el-input__wrapper) {
  background-color: #FFFFFF !important;
  box-shadow: 0 0 0 1px #D4AF37 inset !important;
}

:deep(.el-select:hover .el-input__wrapper) {
  box-shadow: 0 0 0 1px #C9A227 inset !important;
}

:deep(.el-select .is-focus .el-input__wrapper) {
  box-shadow: 0 0 0 2px #D4AF37 inset !important;
}

/* Fix for dropdown arrows */
:deep(.el-input__suffix),
:deep(.el-select .el-input__suffix) {
  color: #D4AF37 !important;
  background-color: transparent !important;
}

:deep(.el-select .el-input__suffix-inner .el-icon) {
  color: #D4AF37 !important;
  background-color: transparent !important;
}

/* Fix text colors */
:deep(.el-input__inner),
:deep(.el-select .el-input__inner) {
  color: #2E3A3F !important;
}

/* Fix dropdown menus */
:deep(.el-popper),
:deep(.el-select-dropdown),
:deep(.el-select__popper),
:deep(.el-select-dropdown__wrap),
:deep(.el-scrollbar__wrap),
:deep(.el-scrollbar__view) {
  background-color: #FFFFFF !important;
  border: 1px solid #D4AF37 !important;
  box-shadow: 0 6px 16px rgba(212, 175, 55, 0.2) !important;
}

/* Fix dropdown items */
:deep(.el-select-dropdown__item) {
  color: #2E3A3F !important;
  background-color: #FFFFFF !important;
}

:deep(.el-select-dropdown__item.hover),
:deep(.el-select-dropdown__item:hover) {
  background-color: rgba(212, 175, 55, 0.15) !important;
}

:deep(.el-select-dropdown__item.selected) {
  background-color: rgba(212, 175, 55, 0.25) !important;
  color: #D4AF37 !important;
  font-weight: 600 !important;
}
/* Force override for Element Plus dark theme on selectors - important! */
.el-select .el-input .el-input__wrapper,
.el-input .el-input__wrapper {
  background-color: #FFFFFF !important;
  box-shadow: 0 0 0 1px #D4AF37 inset !important;
}

.el-select-dropdown,
.el-popper.is-pure,
.el-popper.is-light,
.el-select__popper {
  background-color: #FFFFFF !important;
  border: 1px solid #D4AF37 !important;
}

/* Global style override for Element Plus selectors */
:root {
  --el-bg-color: #FFFFFF !important;
  --el-fill-color-blank: #FFFFFF !important;
  --el-bg-color-overlay: #FFFFFF !important;
  --el-text-color-primary: #2E3A3F !important;
  --el-border-color: #D4AF37 !important;
  --el-border-color-hover: #C9A227 !important;
}
/* Element Plus global theme override */
:root {
  --el-color-white: #FFFFFF !important;
  --el-bg-color: #FFFFFF !important;
  --el-bg-color-overlay: #FFFFFF !important;
  --el-fill-color-blank: #FFFFFF !important;
  --el-text-color-primary: #2E3A3F !important;
  --el-border-color: #D4AF37 !important;
  --el-border-color-hover: #C9A227 !important;
  --el-color-primary: #D4AF37 !important;
  --el-color-primary-light-3: rgba(212, 175, 55, 0.8) !important;
  --el-color-primary-light-5: rgba(212, 175, 55, 0.6) !important;
  --el-color-primary-light-7: rgba(212, 175, 55, 0.4) !important;
  --el-color-primary-light-9: rgba(212, 175, 55, 0.1) !important;
  --el-color-black: #2E3A3F !important;
}

/* Brutal force override for dark elements */
#app .el-select-dropdown,
#app .el-popper,
#app .el-scrollbar,
#app .el-scrollbar__view,
#app .el-select-dropdown__list,
#app .el-select-dropdown__item,
body .el-select-dropdown,
body .el-popper,
body .el-scrollbar,
body .el-scrollbar__view,
body .el-select-dropdown__list,
body .el-select-dropdown__item {
  background-color: #FFFFFF !important;
  color: #2E3A3F !important;
  border-color: #D4AF37 !important;
}

/* Remove dark backgrounds from all Element Plus components */
div[class*="el-"] {
  background-color: #FFFFFF !important;
}
</style>
