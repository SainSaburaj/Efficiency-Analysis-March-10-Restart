<template>
  <div class="text-center w-full max-w-7xl mx-auto">
    <div class="text-2xl md:text-3xl lg:text-4xl font-bold mb-6 sm:mb-10 lg:mb-16 text-gray-800">
      Efficiency Analysis
    </div>
    <div class="flex justify-center gap-6 flex-wrap">
      <!-- Production Efficiency Analysis Tile -->
      <router-link
        :to="{ name: 'ProductionEfficiencyAnalysis' }"
        class="relative flex flex-col items-center justify-center bg-white rounded-2xl shadow-lg hover:shadow-2xl transform hover:-translate-y-2 transition-all duration-300 h-40 w-64"
      >
        <div class="bg-gray-200 rounded-full flex items-center justify-center w-16 h-16 mb-4 transition-colors duration-300 hover:bg-gray-300">
          <i class="fas fa-chart-line text-3xl text-gray-600"></i>
        </div>
        <div class="text-center">
          <h3 class="text-base font-bold text-gray-800">Production Efficiency Analysis</h3>
          <p class="mt-2 text-xs text-gray-600">Production analysis without repairs</p>
        </div>
      </router-link>
      <!-- Overall Efficiency Analysis Tile -->
      <router-link
      :to="{ name: 'OverallEfficiencyAnalysis' }"
      class="relative flex flex-col items-center justify-center bg-white rounded-2xl shadow-lg hover:shadow-2xl transform hover:-translate-y-2 transition-all duration-300 h-40 w-64"
      >
      <div class="bg-gray-200 rounded-full flex items-center justify-center w-16 h-16 mb-4 transition-colors duration-300 hover:bg-gray-300">
        <i class="fas fa-industry text-3xl text-gray-600"></i>
      </div>
        <div class="text-center">
          <h3 class="text-base font-bold text-gray-800">Overall Efficiency Analysis</h3>
          <p class="mt-2 text-xs text-gray-600">Comprehensive production analysis</p>
        </div>
      </router-link>

      <!-- Repair Efficiency Analysis Tile -->
      <router-link
        :to="{ name: 'RepairEfficiencyAnalysis' }"
        class="relative flex flex-col items-center justify-center bg-white rounded-2xl shadow-lg hover:shadow-2xl transform hover:-translate-y-2 transition-all duration-300 h-40 w-64"
      >
        <div class="bg-gray-200 rounded-full flex items-center justify-center w-16 h-16 mb-4 transition-colors duration-300 hover:bg-gray-300">
          <i class="fas fa-tools text-3xl text-gray-600"></i>
        </div>
        <div class="text-center">
          <h3 class="text-base font-bold text-gray-800">Repair Efficiency Analysis</h3>
          <p class="mt-2 text-xs text-gray-600">Repair production analysis</p>
        </div>
      </router-link>

    </div>
  </div>
</template>

<script>
export default {
  name: 'EfficiencyAnalysisHome'
};
</script>

<style scoped>
.translate-x-full {
  transform: translateX(100%);
}
</style>
