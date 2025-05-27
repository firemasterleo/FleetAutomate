<template>
    <div class="tab-navigation">
      <div 
        v-for="tab in tabs" 
        :key="tab.id" 
        :class="['tab', { 'active': activeTab === tab.id }]"
        @click="$emit('tab-changed', tab.id)"
      >
        {{ tab.label }}
      </div>
    </div>
  </template>
  
  <script setup>
  defineProps({
    tabs: {
      type: Array,
      required: true
    },
    activeTab: {
      type: String,
      required: true
    }
  });
  
  defineEmits(['tab-changed']);
  </script>
  
  <style lang="scss" scoped>
  .tab-navigation {
    display: flex;
    border-bottom: none;
    margin-bottom: 0;
    
    .tab {
      padding: 12px 24px;
      cursor: pointer;
      background-color: #f5f7fa;
      border: 1px solid #ddd;
      border-bottom: none;
      border-radius: 8px 8px 0 0;
      margin-right: 4px;
      transition: all 0.2s ease;
      
      &:hover {
        background-color: #e9ecef;
      }
      
      &.active {
        background-color: #fff;
        border-bottom: 1px solid #fff;
        position: relative;
        bottom: -1px;
        font-weight: bold;
      }
    }
  }
  </style>