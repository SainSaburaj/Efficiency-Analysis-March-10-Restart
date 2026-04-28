<template>
    <!-- No Data Popup Modal -->
    <div v-if="showNoDataPopup"
        class="fixed inset-0 flex justify-center items-center bg-black bg-opacity-50 backdrop-blur-sm z-50 transition-opacity duration-300">
        <div class="bg-white p-8 rounded-xl shadow-2xl transform scale-95 transition-transform duration-300 ease-out">
            <div class="text-center">
                <div class="w-16 h-16 mx-auto mb-4 flex justify-center items-center bg-red-100 rounded-full">
                    <svg class="w-10 h-10 text-red-500" fill="none" stroke="currentColor" stroke-width="2"
                        viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                        <path stroke-linecap="round" stroke-linejoin="round"
                            d="M18.364 18.364A9 9 0 115.636 5.636a9 9 0 0112.728 12.728z"></path>
                        <path stroke-linecap="round" stroke-linejoin="round" d="M15 9l-6 6m0-6l6 6"></path>
                    </svg>
                </div>
                <h2 class="text-2xl font-bold text-gray-800">No Data Available</h2>
                <p class="text-gray-500 mt-2">Sorry, we couldn't find any data matching your criteria.</p>
                <button @click="closeNoDataPopup"
                    class="mt-6 px-6 py-3 bg-gray-500 text-white rounded-lg shadow-md hover:bg-gray-600 transition duration-200 transform hover:scale-105">
                    Close
                </button>
            </div>
        </div>
    </div>

    <div v-if="isInitialLoading || loading" class="flex justify-center items-center h-screen">
        <div class="animate-spin rounded-full h-10 w-10 border-t-4 border-gray-500"></div>
    </div>

    <div v-else class="flex h-screen overflow-hidden">
        <!-- Sidebar -->
        <aside class="w-64 bg-gradient-to-b text-white flex flex-col shadow-lg h-screen sticky top-0 flex-shrink-0 overflow-y-auto scrollbar-hidden">
            <!-- Sidebar Header -->
            <div class="p-5 text-lg font-bold flex items-center space-x-2 text-gray-800">
                <i class="fas fa-map-marker-alt text-gray-800"></i>
                <span>LOCATION</span>
            </div>

            <nav class="flex-1 overflow-auto scrollbar-hidden">
                <div v-for="(location, index) in locations" :key="index">
                    <button @click="toggleLocationView(location.internalid.value)" class="flex justify-between items-center py-3 px-5 w-full text-sm font-semibold transition-all 
                         hover:bg-gray-100 hover:text-white hover:scale-105 focus:outline-none"
                        :class="{ 'bg-gray-730 text-white': expandedLocation === location.internalid.value }">


                        <div class="flex items-center space-x-2 text-gray-800">
                            <i class="fas fa-building text-gray-800 "></i>
                            <span>{{ formatName(location.name.value) }}</span>
                        </div>
                        <i
                            :class="expandedLocation === location.internalid.value ? 'fas fa-chevron-down' : 'fas fa-chevron-right'"></i>
                    </button>

                    <ul v-if="expandedLocation === location.internalid.value"
                        class="ml-5 text-gray-500 transition-all text-[12px]">
                        <li v-for="(dept, dIndex) in location.departments" :key="dIndex">
                            <div @click="toggleDepartmentView(dept.name)" class="flex justify-between items-center py-2 px-4 hover:bg-gray-650 hover:text-white hover:scale-105 
                                 cursor-pointer transition-all rounded-md text-[12px]"
                                :class="{ 'bg-gray-400 text-white': expandedDepartment === dept.name }">

                                <div class="flex items-center space-x-2 text-gray-800">
                                    <i class="fa fa-archive"></i>
                                    <span>{{ formatName(dept.name) }}</span>
                                </div>
                                <i
                                    :class="expandedDepartment === dept.name ? 'fas fa-chevron-down' : 'fas fa-chevron-right'"></i>
                            </div>

                            <ul v-if="expandedDepartment === dept.name"
                                class="ml-6 text-gray-400 transition-all text-[12px] text-gray-800">
                                <li v-for="(emp, eIndex) in dept.employees" :key="eIndex" class="py-1 px-5 hover:bg-gray-800 hover:text-white hover:scale-105 cursor-pointer 
                                      flex items-center space-x-2 transition-all text-[12px] rounded-md">
                                    <i class="fa fa-user"></i>
                                    <span>{{ formatName(emp.name) }}</span>
                                </li>

                            </ul>
                        </li>
                    </ul>

                </div>
            </nav>

        </aside>

        <!-- Main Content -->
        <main class="flex-1 p-4 overflow-auto scrollbar-hidden min-h-screen">
            <!-- Header Section -->
            <div class="flex justify-between items-center mb-6">
                <h1 class="text-2xl font-bold">{{ dashboardTitle }}</h1>

                <div class="flex items-center space-x-4">
                    <!-- Legend (Separate Line) -->
                    <div class="flex items-center space-x-6 text-gray-600 text-sm mb-2">
                        <div class="flex items-center space-x-2">
                            <i class="fas fa-arrow-up text-green-500"></i>
                            <span>Production</span>
                        </div>
                        <div class="flex items-center space-x-2">
                            <i class="fas fa-arrow-down text-red-500"></i>
                            <span>Loss</span>
                        </div>
                    </div>


                    <!-- Standardized Date Filter -->
                    <div>
                        <VueDatePicker v-model="selectedDateRange" range :enable-time-picker="false"
                            :format="'MM/dd/yyyy'" :auto-apply="true" placeholder="Select date range"
                            menu-class-name="custom-date-picker"
                            class="custom-datepicker flex-1 text-gray-700 cursor-pointer" />
                    </div>

                    <!---- Download Button-->
                    <button @click="downloadData">
                        <div class="w-6 h-6 bg-black rounded-full flex items-center justify-center ml-2">
                            <i class="fa fa-download text-white text-sm"></i>
                        </div>
                    </button>
                </div>
            </div>


            <!-- Loading Spinner -->
            <div v-if="loading" class="flex justify-center items-center">
                <div class="animate-spin rounded-full h-10 w-10 border-t-4 border-gray-500"></div>
            </div>
            <div class="main-card-grid grid grid-cols-5 gap-2">

                <!-- Locations Section -->
                <div v-if="!loading && !showDepartments && !showEmployees" v-for="(location, index) in locations"
                    :key="index" @click="toggleLocationView(location.internalid.value)"
                    class="flex shadow-lg rounded-lg hover:shadow-sm hover:-translate-y-1 transition-all cursor-pointer p-1 items-center w-full bg-gray-200">

                    <!-- Left Icon Section -->
                    <div class="flex items-center justify-center w-10 h-full bg-gray-200">
                        <i class="fas fa-map-marker-alt text-gray-600 text-sm"></i>
                    </div>

                    <!-- Right Content Section -->
                    <div class="flex-1 pl-2 pr-2 py-2 bg-white rounded-r-lg border-2 border-white shadow-md">
                        <p class="text-xs font-semibold text-gray-700">{{ location.name.value }}</p>

                        <!-- Bag Count -->
                        <div class="bg-green-50 p-1 rounded mb-2 text-center">
                            <p class="text-[10px] font-semibold text-gray-700">No. of Bags: <span class="text-blue-600">{{ getLocationTotalBagCount(location) || 0 }}</span></p>
                        </div>

                        <div class="text-[11px] font-semibold w-full text-center bg-gray-100 rounded p-1 mb-1">Issued & Loss Quantity</div>

                        <div class="grid grid-cols-2 gap-y-0.5">
                            <!-- Gold Section -->
                            <div class="flex flex-col text-left">
                                <div class="flex items-center space-x-0.5">
                                    <i class="fas fa-ring text-gray-500 text-[9px]"></i>
                                    <span class="font-semibold text-gray-600 text-[11px]">Gold</span>
                                </div>
                                <span class="text-gray-700 text-[11px]">grams</span>
                                <div class="flex items-center space-x-0.5">
                                    <i class="fas fa-arrow-up text-green-500 text-[9px]"></i>
                                    <p class="text-green-500 text-[10px]">{{ (getLocationTotalIssuedQtyGold(location) || 0).toFixed(2) }}</p>
                                </div>
                                <div class="flex items-center space-x-0.5">
                                    <i class="fas fa-arrow-down text-red-500 text-[9px]"></i>
                                    <p class="text-red-500 text-[10px]">{{ (getLocationTotalLossQtyGold(location) || 0).toFixed(2) }}</p>
                                </div>
                            </div>

                            <!-- Diamond Section -->
                            <div class="flex flex-col">  
                                <div class="flex items-center space-x-0.5">
                                    <i class="fas fa-gem text-gray-500 text-[9px]"></i>
                                    <span class="font-semibold text-gray-600 text-[11px]">Diamond</span>
                                </div>
                                <div class="grid grid-cols-2 gap-x-1">
                                    <div class="flex flex-col items-center">
                                        <span class="text-gray-700 text-[11px]">carat</span>
                                        <div class="flex items-center space-x-0.5">
                                            <i class="fas fa-arrow-up text-green-500 text-[9px]"></i>
                                            <p class="text-green-500 text-[10px]">{{ (getLocationTotalIssuedQtyDiamond(location) || 0).toFixed(2) }}</p>
                                        </div>
                                        <div class="flex items-center space-x-0.5">
                                            <i class="fas fa-arrow-down text-red-500 text-[9px]"></i>
                                            <p class="text-red-500 text-[10px]">{{ (getLocationTotalLossQtyDiamond(location) || 0).toFixed(2) }}</p>
                                        </div>
                                    </div>
                                    <div class="flex flex-col items-right">
                                        <span class="text-gray-700 text-[11px]">pieces</span>
                                        <div class="flex items-center space-x-0.5">
                                            <i class="fas fa-arrow-up text-green-500 text-[9px]"></i>
                                            <p class="text-green-500 text-[10px]">{{ (getLocationTotalIssuedPiecesDiamond(location) || 0).toFixed(2) }}</p>
                                        </div>
                                        <div class="flex items-center space-x-0.5">
                                            <i class="fas fa-arrow-down text-red-500 text-[9px]"></i>
                                            <p class="text-red-500 text-[10px]">{{ (getLocationTotalLossPiecesDiamond(location) || 0).toFixed(2) }}</p>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="text-[11px] font-semibold w-full text-center bg-gray-100 rounded p-1 mb-1 mt-1">Actual Production and Loss</div>
                        <!-- Actual Production and Loss Section -->
                        <div class="grid grid-cols-2 gap-y-1">
                            <!-- Actual Gold Section -->
                            <div class="flex flex-col text-left">
                                <div class="flex items-center space-x-1">
                                    <i class="fas fa-ring text-gray-500 text-[9px]"></i>
                                    <span class="text-[11px] font-semibold text-gray-600">Gold</span>
                                </div>
                                <div class="flex items-center space-x-1">
                                    <i class="fas fa-arrow-up text-green-500 text-[9px]"></i>
                                    <p class="text-green-500 text-[10px]">{{ (getLocationTotalActualProductionGold(location) || 0).toFixed(2) }}</p>
                                </div>
                            </div>

                            <!-- Actual Diamond Section -->
                            <div class="flex flex-col">
                                <div class="flex items-center space-x-1">
                                    <i class="fas fa-gem text-gray-500 text-[9px]"></i>
                                    <span class="text-[11px] font-semibold text-gray-600">Diamond</span>
                                </div>
                                <div class="flex items-center space-x-1">
                                    <i class="fas fa-arrow-up text-green-500 text-[9px]"></i>
                                    <p class="text-green-500 text-[10px]">{{ (getLocationTotalActualProductionDiamond(location) || 0).toFixed(2) }}</p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Departments Section -->
                <div v-if="!loading && showDepartments && !showEmployees" v-for="(dept, index) in selectedDepartments"
                    :key="index" @click="toggleDepartmentView(dept.name)"
                    class="flex shadow-lg rounded-lg hover:shadow-sm hover:-translate-y-1 transition-all cursor-pointer p-1 items-center w-full bg-gray-200">

                    <!-- Left Icon Section -->
                    <div class="flex items-center justify-center w-10 h-full bg-gray-200">
                        <i class="fa fa-archive text-gray-600 text-sm"></i>
                    </div>

                    <!-- Right Content Section -->
                    <div class="flex-1 pl-2 pr-2 py-2 bg-white rounded-r-lg border-2 border-white shadow-md">
                        <p class="text-sm font-semibold text-gray-700 mb-2">{{ dept.name }}</p>

                        <!-- Bag Count -->
                        <div class="bg-green-50 p-1 rounded mb-2 text-center">
                            <p class="text-[10px] font-semibold text-gray-700">No. of Bags: <span class="text-blue-600">{{ dept.bag_count || 0 }}</span></p>
                        </div>

                        <div class="text-[11px] font-semibold w-full text-center bg-gray-100 rounded p-1 mb-1">Issued & Loss Quantity</div>

                        <div class="grid grid-cols-2 mt-1 gap-y-1">
                            <!-- Gold Section -->
                            <div class="flex flex-col text-left">
                                <div class="flex items-center space-x-1">
                                    <i class="fas fa-ring text-gray-500 text-[9px]"></i>
                                    <span class="text-[11px] font-semibold text-gray-600">Gold</span>
                                </div>
                                <span class="text-[11px] text-gray-700 mt-1">grams</span>
                                <div class="flex items-center space-x-1">
                                    <i class="fas fa-arrow-up text-green-500 text-[9px]"></i>
                                    <p class="text-green-500 text-[10px]">{{ (getDepartmentTotalIssuedQtyGold(dept) || 0).toFixed(2) }}</p>
                                </div>
                                <div class="flex items-center space-x-1">
                                    <i class="fas fa-arrow-down text-red-500 text-[9px]"></i>
                                    <p class="text-red-500 text-[10px]">{{ (getDepartmentWaxTreeLossGold(dept) !== null ? getDepartmentWaxTreeLossGold(dept) : getDepartmentTotalLossQtyGold(dept) || 0).toFixed(2) }}</p>
                                </div>
                            </div>

                            <!-- Diamond Section -->
                            <div class="flex flex-col">
                                <div class="flex items-center space-x-1">
                                    <i class="fas fa-gem text-gray-500 text-[9px]"></i>
                                    <span class="text-[11px] font-semibold text-gray-600">Diamond</span>
                                </div>
                                <div class="grid grid-cols-2 gap-x-2 mt-1">
                                    <!-- Diamond Carat Section -->
                                    <div class="flex flex-col items-center">
                                        <span class="text-[11px] text-gray-700">carat</span>
                                        <div class="flex items-center space-x-1">
                                            <i class="fas fa-arrow-up text-green-500 text-[9px]"></i>
                                            <p class="text-green-500 text-[10px]">{{ (getDepartmentTotalIssuedQtyDiamond(dept) || 0).toFixed(2) }}</p>
                                        </div>
                                        <div class="flex items-center space-x-1">
                                            <i class="fas fa-arrow-down text-red-500 text-[9px]"></i>
                                            <p class="text-red-500 text-[10px]">{{ (getDepartmentTotalLossQtyDiamond(dept) || 0).toFixed(2) }}</p>
                                        </div>
                                    </div>

                                    <!-- Diamond Pieces Section -->
                                    <div class="flex flex-col items-right">
                                        <span class="text-[11px] text-gray-700">pieces</span>
                                        <div class="flex items-center space-x-1">
                                            <i class="fas fa-arrow-up text-green-500 text-[9px]"></i>
                                            <p class="text-green-500 text-[10px]">{{ (getDepartmentTotalIssuedPiecesDiamond(dept) || 0).toFixed(2) }}</p>
                                        </div>
                                        <div class="flex items-center space-x-1">
                                            <i class="fas fa-arrow-down text-red-500 text-[9px]"></i>
                                            <p class="text-red-500 text-[10px]">{{ (getDepartmentTotalLossPiecesDiamond(dept) || 0).toFixed(2) }}</p>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>

                        <div class="text-[11px] font-semibold w-full text-center bg-gray-100 rounded p-1 mb-1 mt-1">Actual Production and Loss</div>

                        <!-- Actual Production and Loss Section -->
                        <div class="grid grid-cols-2 gap-y-1">
                            <!-- Actual Gold Section -->
                            <div class="flex flex-col text-left">
                                <div class="flex items-center space-x-1">
                                    <i class="fas fa-ring text-gray-500 text-[9px]"></i>
                                    <span class="text-[11px] font-semibold text-gray-600">Gold</span>
                                </div>
                                <div class="flex items-center space-x-1">
                                    <i class="fas fa-arrow-up text-green-500 text-[9px]"></i>
                                    <p class="text-green-500 text-[10px]">{{ (getDepartmentTotalActualProductionGold(dept) || 0).toFixed(2) }}</p>
                                </div>
                            </div>

                            <!-- Actual Diamond Section -->
                            <div class="flex flex-col">
                                <div class="flex items-center space-x-1">
                                    <i class="fas fa-gem text-gray-500 text-[9px]"></i>
                                    <span class="text-[11px] font-semibold text-gray-600">Diamond</span>
                                </div>
                                <div class="flex items-center space-x-1">
                                    <i class="fas fa-arrow-up text-green-500 text-[9px]"></i>
                                    <p class="text-green-500 text-[10px]">{{ (getDepartmentTotalActualProductionDiamond(dept) || 0).toFixed(2) }}</p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Employees Section -->
                <div v-if="!loading && showEmployees" v-for="(emp, index) in selectedEmployees" :key="index"
                    class="flex shadow-lg rounded-lg hover:shadow-sm hover:-translate-y-1 transition-all cursor-pointer p-1 items-center w-full bg-gray-200">

                    <!-- Left Icon Section -->
                    <div class="flex items-center justify-center w-10 h-full bg-gray-200">
                        <i class="fas fa-user text-gray-600 text-sm"></i>
                    </div>

                    <!-- Right Content Section -->
                    <div class="flex-1 pl-2 pr-2 py-2 bg-white rounded-r-lg border-2 border-white shadow-md">
                        <p class="text-sm font-semibold text-gray-700">{{ emp.name }}</p>

                        <!-- Bag Count -->
                        <div class="bg-green-50 p-1 rounded mb-2 text-center">
                            <p class="text-[10px] font-semibold text-gray-700">No. of Bags: <span class="text-blue-600">{{ emp.bag_count || 0 }}</span></p>
                        </div>

                        <div class="text-[11px] font-semibold w-full text-center bg-gray-100 rounded p-1 mb-1">Issued & Loss Quantity</div>

                        <div class="grid grid-cols-2 mt-1 gap-y-1">
                            <!-- Gold Section -->
                            <div class="flex flex-col text-left">
                                <div class="flex items-center space-x-1">
                                    <i class="fas fa-ring text-gray-500 text-[9px]"></i>
                                    <span class="text-[11px] font-semibold text-gray-600">Gold</span>
                                </div>
                                <span class="text-[11px] text-gray-700 mt-1">grams</span>
                                <div class="flex items-center space-x-1">
                                    <i class="fas fa-arrow-up text-green-500 text-[9px]"></i>
                                    <p class="text-green-500 text-[10px]">{{ (getEmployeeTotalIssuedQtyGold(emp) || 0).toFixed(2) }}</p>
                                </div>
                                <div class="flex items-center space-x-1">
                                    <i class="fas fa-arrow-down text-red-500 text-[9px]"></i>
                                    <p class="text-red-500 text-[10px]">{{ (getEmployeeTotalLossQtyGold(emp) || 0).toFixed(2) }}</p>
                                </div>
                            </div>

                            <!-- Diamond Section -->
                            <div class="flex flex-col">
                                <div class="flex items-center space-x-1">
                                    <i class="fas fa-gem text-gray-500 text-[9px]"></i>
                                    <span class="text-[11px] font-semibold text-gray-600">Diamond</span>
                                </div>
                                <div class="grid grid-cols-2 gap-x-2 mt-1">
                                    <!-- Diamond Carat Section -->
                                    <div class="flex flex-col items-center">
                                        <span class="text-[11px] text-gray-700">carat</span>
                                        <div class="flex items-center space-x-1">
                                            <i class="fas fa-arrow-up text-green-500 text-[9px]"></i>
                                            <p class="text-green-500 text-[10px]">{{ (getEmployeeTotalIssuedQtyDiamond(emp) || 0).toFixed(2) }}</p>
                                        </div>
                                        <div class="flex items-center space-x-1">
                                            <i class="fas fa-arrow-down text-red-500 text-[9px]"></i>
                                            <p class="text-red-500 text-[10px]">{{ (getEmployeeTotalLossQtyDiamond(emp) || 0).toFixed(2) }}</p>
                                        </div>
                                    </div>

                                    <!-- Diamond Pieces Section -->
                                    <div class="flex flex-col items-right">
                                        <span class="text-[11px] text-gray-700">pieces</span>
                                        <div class="flex items-center space-x-1">
                                            <i class="fas fa-arrow-up text-green-500 text-[9px]"></i>
                                            <p class="text-green-500 text-[10px]">{{ (getEmployeeTotalIssuedPiecesDiamond(emp) || 0).toFixed(2) }}</p>
                                        </div>
                                        <div class="flex items-center space-x-1">
                                            <i class="fas fa-arrow-down text-red-500 text-[9px]"></i>
                                            <p class="text-red-500 text-[10px]">{{ (getEmployeeTotalLossPiecesDiamond(emp) || 0).toFixed(2) }}</p>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>

                        <div class="text-[11px] font-semibold w-full text-center bg-gray-100 rounded p-1 mb-1 mt-1">Actual Production and Loss</div>

                        <!-- Actual Production Section -->
                        <div class="grid grid-cols-2 gap-y-1">
                            <!-- Actual Gold Section -->
                            <div class="flex flex-col text-left">
                                <div class="flex items-center space-x-1">
                                    <i class="fas fa-ring text-gray-500 text-[9px]"></i>
                                    <span class="text-[11px] font-semibold text-gray-600">Gold</span>
                                </div>
                                <div class="flex items-center space-x-1">
                                    <i class="fas fa-arrow-up text-green-500 text-[9px]"></i>
                                    <p class="text-green-500 text-[10px]">{{ (getEmployeeTotalActualProductionGold(emp) || 0).toFixed(2) }}</p>
                                </div>
                            </div>

                            <!-- Actual Diamond Section -->
                            <div class="flex flex-col">
                                <div class="flex items-center space-x-1">
                                    <i class="fas fa-gem text-gray-500 text-[9px]"></i>
                                    <span class="text-[11px] font-semibold text-gray-600">Diamond</span>
                                </div>
                                <div class="flex items-center space-x-1">
                                    <i class="fas fa-arrow-up text-green-500 text-[9px]"></i>
                                    <p class="text-green-500 text-[10px]">{{ (getEmployeeTotalActualProductionDiamond(emp) || 0).toFixed(2) }}</p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>


            <!-- Table Section -->
            <div v-if="showTable" class="bg-white p-3 rounded-lg shadow mt-4">
                <h2 class="text-md font-bold mb-1 text-gray-700">
                    {{ showEmployeesTable ? 'Employee Details' : 'Department Details' }}
                </h2>
                <div class="table-container overflow-x-auto overflow-y-auto max-h-[45rem] scrollbar-custom">
                    <table class="w-full border-collapse border border-gray-200 text-[10px]"> <!-- Reduced text size -->
                        <thead class="bg-gray-100 text-gray-600 uppercase text-[11px]" style="position: sticky; top: 0; z-index: 10;">
                            <tr>
                                <th class="px-3 py-2 text-left font-semibold">#</th>
                                <th class="px-3 py-2 text-left font-semibold">
                                    {{ showEmployeesTable ? 'Employees' : 'Department' }}
                                </th>
                                <!-- <th class="px-3 py-2 text-left font-semibold">Dates</th> -->
                                <th class="px-3 py-2 text-left font-semibold">No. of Bags</th>
                                <th class="px-3 py-2 text-left font-semibold">Item category</th>
                                <th class="px-3 py-2 text-left font-semibold">Sub Category</th>
                                <th class="px-3 py-2 text-left font-semibold">Bag Count</th>
                                <th class="px-3 py-2 text-left font-semibold">Style Number</th>
                                <th class="px-3 py-2 text-left font-semibold">Bag Name</th>

                                <th class="px-3 py-2 text-left font-semibold">Issued Net Weight</th>
                                <th class="px-3 py-2 text-left font-semibold">Received Quantity</th>
                                <th class="px-3 py-2 text-left font-semibold">Gross Loss</th>
                                <th class="px-3 py-2 text-left font-semibold">Gross Loss %</th>

                                <th class="px-3 py-2 text-left font-semibold">Pure Weight</th>
                                <th class="px-3 py-2 text-left font-semibold">Pure Loss</th>

                                <th class="px-3 py-2 text-left font-semibold">Net Loss</th>
                                <th class="px-3 py-2 text-left font-semibold">Net Loss %</th>

                                <th class="px-3 py-2 text-left font-semibold">TM Production Diamond</th>
                                <!-- <th class="px-3 py-2 text-left font-semibold">Issued Qty Diamond</th> -->
                                <th class="px-3 py-2 text-left font-semibold">Actual Production Diamond</th>
                                <th class="px-3 py-2 text-left font-semibold">Loss Qty Diamond</th>
                                <!-- <th class="px-3 py-2 text-left font-semibold">Scrap Qty Diamond</th>
                                <!-- <th class="px-3 py-2 text-left font-semibold">Balance Qty Diamond</th> -->
                                <th class="px-3 py-2 text-left font-semibold">Diamond Loss %</th>


                                <th class="px-3 py-2 text-left font-semibold">Gold Recovery Weight (gm)</th>
                                <th class="px-3 py-2 text-left font-semibold">Gold Recovery Percentage</th>
                                
                                <!-- <th class="px-3 py-2 text-left font-semibold">Net Loss Gold</th>
                                <th class="px-3 py-2 text-left font-semibold">Diamond Recovery Weight (ct)</th>
                                <th class="px-3 py-2 text-left font-semibold">Net Loss Diamond</th> -->
                            </tr>
                        </thead>

                        <tbody class="text-gray-700">
                            <!-- Department Details with Categories -->
                            <template v-if="!showEmployeesTable" v-for="(dept, deptIndex) in selectedDepartmentData" :key="'dept-' + deptIndex">
                                <!-- If department has categories, show them with rowspan -->
                                <template v-if="dept.unique_categories_array && dept.unique_categories_array.length > 0">
                                    <!-- Compute total bag rows for this dept (sum of bags per category) -->
                                    <template v-for="(category, catIndex) in dept.unique_categories_array" :key="'dept-' + deptIndex + '-cat-' + catIndex">
                                        <!-- bags for this category -->
                                        <template v-for="(bagName, bagIndex) in (dept.category_bag_names_map?.[category] || [''])" :key="'dept-' + deptIndex + '-cat-' + catIndex + '-bag-' + bagIndex">
                                            <tr class="border-b group hover:bg-gray-50 transition-all duration-200 text-[11px]"
                                                :class="{ 'border-b': catIndex === dept.unique_categories_array.length - 1 && bagIndex === (dept.category_bag_names_map?.[category]?.length || 1) - 1 }">
                                                <!-- SL No - only on very first row of dept -->
                                                <td v-if="catIndex === 0 && bagIndex === 0"
                                                    :rowspan="dept.unique_categories_array.reduce((s, c) => s + (dept.category_bag_names_map?.[c]?.length || 1), 0)"
                                                    class="px-3 py-2 group-hover:!bg-white border-r">{{ deptIndex + 1 }}</td>

                                                <!-- Department Name - only on very first row of dept -->
                                                <td v-if="catIndex === 0 && bagIndex === 0"
                                                    :rowspan="dept.unique_categories_array.reduce((s, c) => s + (dept.category_bag_names_map?.[c]?.length || 1), 0)"
                                                    class="px-3 py-2 group-hover:!bg-white border-r">
                                                    <a v-if="getDeptNsUrl(dept)" 
                                                       :href="getDeptNsUrl(dept)" 
                                                       target="_blank"
                                                       class="text-blue-600 hover:underline">{{ dept.name }}</a>
                                                    <span v-else>{{ dept.name }}</span>
                                                </td>

                                                <!-- No. of Bags - only on very first row of dept -->
                                                <td v-if="catIndex === 0 && bagIndex === 0"
                                                    :rowspan="dept.unique_categories_array.reduce((s, c) => s + (dept.category_bag_names_map?.[c]?.length || 1), 0)"
                                                    class="px-3 py-2 font-semibold text-center bg-blue-50 border-r">{{ dept.bag_count || 0 }}</td>

                                                <!-- Item Category - rowspan = number of bags in this category -->
                                                <td v-if="bagIndex === 0"
                                                    :rowspan="dept.category_bag_names_map?.[category]?.length || 1"
                                                    class="px-3 py-2 group-hover:shadow-md border-r">
                                                    <a :href="getCategoryNsUrl(dept, category, bagName)"
                                                       target="_blank"
                                                       class="text-blue-600 hover:underline">{{ category || 'N/A' }}</a>
                                                </td>

                                                <!-- Sub Category - per bag -->
                                                <td class="px-3 py-2 group-hover:shadow-md border-r">
                                                    <a v-if="getSubCategoryNsUrl(dept, category, bagName)"
                                                       :href="getSubCategoryNsUrl(dept, category, bagName)"
                                                       target="_blank"
                                                       class="text-blue-600 hover:underline">{{ dept.category_bag_sub_category_map?.[category]?.[bagName] || '-' }}</a>
                                                    <span v-else>{{ dept.category_bag_sub_category_map?.[category]?.[bagName] || '-' }}</span>
                                                </td>

                                                <!-- Bag Count (per category) - rowspan = bags in category -->
                                                <td v-if="bagIndex === 0"
                                                    :rowspan="dept.category_bag_names_map?.[category]?.length || 1"
                                                    class="px-3 py-2 group-hover:shadow-md text-center bg-blue-50">{{ dept.category_bag_count_map?.[category] || 0 }}</td>

                                                <!-- Style Number (Print Design) - per bag, no rowspan -->
                                                <td class="px-3 py-2 group-hover:shadow-md">
                                                    <a v-if="getStyleNsUrlPerBag(dept, category, bagName)" 
                                                       :href="getStyleNsUrlPerBag(dept, category, bagName)"
                                                       target="_blank"
                                                       class="text-blue-600 hover:underline">{{ dept.category_bag_print_design_map?.[category]?.[bagName] || '-' }}</a>
                                                    <span v-else>{{ dept.category_bag_print_design_map?.[category]?.[bagName] || '-' }}</span>
                                                </td>

                                                <!-- Bag Name -->
                                                <td class="px-3 py-2 group-hover:shadow-md">
                                                    <a v-if="getBagNsUrl(dept, category, bagName)"
                                                       :href="getBagNsUrl(dept, category, bagName)"
                                                       target="_blank"
                                                       class="text-blue-600 hover:underline">{{ bagName }}</a>
                                                    <span v-else>{{ bagName || '-' }}</span>
                                                </td>

                                                <!-- Issued Net Weight (without loss) = starting + issued - (scrap + balance) -->
                                                <td class="px-3 py-2 group-hover:shadow-md">{{ roundToTwo(getBagStartingQtyGoldRaw(dept, category, bagName) + getBagIssuedQtyGoldRaw(dept, category, bagName) - (getBagScrapQtyGoldRaw(dept, category, bagName) + getBagBalanceQtyGoldRaw(dept, category, bagName))) }}</td>

                                                <!-- Received Quantity = starting + issued - (loss + scrap + balance) -->
                                                <td class="px-3 py-2 group-hover:shadow-md">{{ roundToTwo(getBagStartingQtyGoldRaw(dept, category, bagName) + getBagIssuedQtyGoldRaw(dept, category, bagName) - (getBagLossQtyGoldRaw(dept, category, bagName) + getBagScrapQtyGoldRaw(dept, category, bagName) + getBagBalanceQtyGoldRaw(dept, category, bagName))) }}</td>

                                                <!-- Gross Loss -->
                                                <td class="px-3 py-2 text-red-500 group-hover:shadow-md">{{ roundToTwo(getBagLossQtyGoldRaw(dept, category, bagName)) }}</td>

                                                <!-- Gross Loss % -->
                                                <td class="px-3 py-2 text-red-500 group-hover:shadow-md">{{ roundToTwo(calculateGoldLossPercentage(getBagStartingQtyGoldRaw(dept, category, bagName), getBagIssuedQtyGoldRaw(dept, category, bagName), getBagLossQtyGoldRaw(dept, category, bagName), getBagScrapQtyGoldRaw(dept, category, bagName), getBagBalanceQtyGoldRaw(dept, category, bagName))) }}%</td>

                                                <!-- Pure Weight -->
                                                <td class="px-3 py-2 group-hover:shadow-md">{{ getBagPureWeight(dept, category, bagName) }}</td>

                                                <!-- Pure Loss -->
                                                <td class="px-3 py-2 text-red-500 group-hover:shadow-md">{{ getBagPureLoss(dept, category, bagName) }}</td>

                                                <!-- Net Loss -->
                                                <td class="px-3 py-2 text-red-500 group-hover:shadow-md">{{ (() => { const recv = getBagStartingQtyGoldRaw(dept, category, bagName) + getBagIssuedQtyGoldRaw(dept, category, bagName) - (getBagLossQtyGoldRaw(dept, category, bagName) + getBagScrapQtyGoldRaw(dept, category, bagName) + getBagBalanceQtyGoldRaw(dept, category, bagName)); return recv > 0 ? roundToTwo(getBagLossQtyGoldRaw(dept, category, bagName) / recv) : '0.00'; })() }}</td>
                                                
                                                <!-- Net Loss % -->
                                                <td class="px-3 py-2 text-red-500 group-hover:shadow-md">{{ (() => { const recv = getBagStartingQtyGoldRaw(dept, category, bagName) + getBagIssuedQtyGoldRaw(dept, category, bagName) - (getBagLossQtyGoldRaw(dept, category, bagName) + getBagScrapQtyGoldRaw(dept, category, bagName) + getBagBalanceQtyGoldRaw(dept, category, bagName)); return recv > 0 ? roundToTwo((getBagLossQtyGoldRaw(dept, category, bagName) / recv) * 100) + '%' : '0.00%'; })() }}</td>

                                                <!-- TM Production Diamond -->
                                                <td class="px-3 py-2 group-hover:shadow-md">{{ roundToTwo(getBagStartingQtyDiamondRaw(dept, category, bagName)) }}</td>

                                                <!-- Actual Production Diamond -->
                                                <td class="px-3 py-2 group-hover:shadow-md">{{ roundToTwo(getBagStartingQtyDiamondRaw(dept, category, bagName) + getBagIssuedQtyDiamondRaw(dept, category, bagName) - getBagLossQtyDiamondRaw(dept, category, bagName) - getBagScrapQtyDiamondRaw(dept, category, bagName) - getBagBalanceQtyDiamondRaw(dept, category, bagName)) }}</td>

                                                <!-- Loss Qty Diamond -->
                                                <td class="px-3 py-2 text-red-500 group-hover:shadow-md">{{ roundToTwo(getBagLossQtyDiamondRaw(dept, category, bagName)) }}</td>

                                                <!-- Diamond Loss % -->
                                                <td class="px-3 py-2 text-red-500 group-hover:shadow-md">{{ roundToTwo(calculateDiamondLossPercentage(getBagStartingQtyDiamondRaw(dept, category, bagName), getBagIssuedQtyDiamondRaw(dept, category, bagName), getBagLossQtyDiamondRaw(dept, category, bagName), getBagScrapQtyDiamondRaw(dept, category, bagName), getBagBalanceQtyDiamondRaw(dept, category, bagName))) }}%</td>

                                                <!-- Gold Recovery Weight — per bag: bag pure loss × global avg recovery % -->
                                                <td class="px-3 py-2 group-hover:shadow-md">
                                                    {{ (getBagLossQtyGoldRaw(dept, category, bagName) * getBagPurityFactor(dept, category, bagName)) > 0 && globalAvgRecoveryPercentage > 0
                                                        ? roundToTwo(getBagLossQtyGoldRaw(dept, category, bagName) * getBagPurityFactor(dept, category, bagName) * (globalAvgRecoveryPercentage / 100))
                                                        : '0.00' }}
                                                </td>
                                                
                                                <!-- Gold Recovery Percentage — global avg -->
                                                <td class="px-3 py-2 group-hover:shadow-md">
                                                    {{ (getBagLossQtyGoldRaw(dept, category, bagName) * getBagPurityFactor(dept, category, bagName)) > 0 && globalAvgRecoveryPercentage > 0
                                                        ? roundToTwo(globalAvgRecoveryPercentage) + '%'
                                                        : '0%' }}
                                                </td>

                                                <!-- Net Loss Gold -->
                                                <!-- <td class="px-3 py-2 text-red-500 group-hover:shadow-md">-</td> -->

                                                <!-- Diamond Recovery Weight -->
                                                <!-- <td class="px-3 py-2 group-hover:shadow-md">-</td> -->

                                                <!-- Net Loss Diamond -->
                                                <!-- <td class="px-3 py-2 group-hover:shadow-md">-</td> -->
                                            </tr>
                                        </template>
                                    </template>
                                </template>

                                <!-- Department Total Row -->
                                <tr class="bg-blue-50 font-bold border-b-8 border-b-gray-200 text-[11px] ">
                                    <td class="px-3 py-2"></td>
                                    <td class="px-3 py-2 text-center" colspan="4">{{ dept.name }} — Total</td>
                                    <td class="px-3 py-2 text-center bg-blue-100">{{ dept.bag_count || 0 }}</td>
                                    <td class="px-3 py-2" colspan="2"></td>
                                    <td class="px-3 py-2">{{ getDeptTotals(dept).issuedNetWt }}</td>
                                    <td class="px-3 py-2">{{ getDeptTotals(dept).recvG }}</td>
                                    <td class="px-3 py-2 text-red-500">{{ getDeptTotals(dept).lG }}</td>
                                    <td class="px-3 py-2 text-red-500">{{ getDeptTotals(dept).grossLossPct }}</td>
                                    <td class="px-3 py-2">{{ getDeptTotals(dept).pw }}</td>
                                    <td class="px-3 py-2 text-red-500">{{ getDeptTotals(dept).pl }}</td>
                                    <td class="px-3 py-2 text-red-500">{{ getDeptTotals(dept).netLoss }}</td>
                                    <td class="px-3 py-2 text-red-500">{{ getDeptTotals(dept).netLossPct }}</td>
                                    <td class="px-3 py-2">{{ getDeptTotals(dept).sD }}</td>
                                    <td class="px-3 py-2">{{ getDeptTotals(dept).aD }}</td>
                                    <td class="px-3 py-2 text-red-500">{{ getDeptTotals(dept).lD }}</td>
                                    <td class="px-3 py-2 text-red-500">{{ getDeptTotals(dept).diamondLossPct }}</td>
                                    <td class="px-3 py-2">{{ getDeptTotals(dept).recoveryWt }}</td>
                                    <td class="px-3 py-2">{{ getDeptTotals(dept).recoveryPct }}</td>
                                </tr>
                            </template>

                            <!-- Total Row for Departments -->
                            <tr v-if="!showEmployeesTable" class="bg-gray-100 font-bold border-t text-[11px]">
                                <td class="px-3 py-2" colspan="2" style="text-align: center;">Total</td>

                                <!-- Total No. of Bags -->
                                <td class="px-3 py-2 font-semibold text-center bg-blue-50">{{ totalDeptBagCount }}</td>

                                <!-- Item Category (empty) -->
                                <td class="px-3 py-2 font-semibold"></td>
                                
                                <!-- Item Sub Category (empty) -->
                                <td class="px-3 py-2 font-semibold"></td>

                                <!-- Style Number (empty) -->
                                <td class="px-3 py-2 font-semibold"></td>

                                <!-- Bag Count (empty) -->
                                <td class="px-3 py-2 font-semibold"></td>

                                <!-- Bag Name (empty) -->
                                <td class="px-3 py-2 font-semibold"></td>

                                <!-- Issued Net Weight -->
                                <td class="px-3 py-2">{{ totalDeptIssuedNetWeightGold }}</td>

                                <!-- Received Quantity -->
                                <td class="px-3 py-2">{{ totalDeptReceivedQtyGold }}</td>

                                <!-- Gross Loss -->
                                <td class="px-3 py-2 text-red-500">{{ totalDeptLossQtyGold }}</td>
                                
                                <!-- Gross Loss % -->
                                <td class="px-3 py-2 text-red-500">{{ parseFloat(totalDeptReceivedQtyGold) > 0 ? roundToTwo((parseFloat(totalDeptLossQtyGold) / parseFloat(totalDeptReceivedQtyGold)) * 100) + '%' : '0.00%' }}</td>
                                
                                <!-- Pure Weight -->
                                <td class="px-3 py-2">{{ totalDeptPureWeight }}</td>

                                <!-- Pure Loss -->
                                <td class="px-3 py-2 text-red-500">{{ totalDeptPureLoss }}</td>
                                
                                <!-- Net Loss -->
                                <td class="px-3 py-2 text-red-500">{{ parseFloat(totalDeptReceivedQtyGold) > 0 ? roundToTwo(parseFloat(totalDeptLossQtyGold) / parseFloat(totalDeptReceivedQtyGold)) : '0.00' }}</td>
                                
                                <!-- Net Loss % -->
                                <td class="px-3 py-2 text-red-500">{{ parseFloat(totalDeptReceivedQtyGold) > 0 ? roundToTwo((parseFloat(totalDeptLossQtyGold) / parseFloat(totalDeptReceivedQtyGold)) * 100) + '%' : '0.00%' }}</td>
                                
                                <!-- Starting Qty Diamond -->
                                <td class="px-3 py-2">{{ totalDeptStartingQuantityDiamond }}</td>
                                
                                <!-- Issued Qty Diamond -->
                                <!-- <td class="px-3 py-2">{{ totalDeptIssuedQtyDiamond }}</td> -->
                                
                                <!-- Actual Production Diamond -->
                                <td class="px-3 py-2">{{ totalDeptActualProductionDiamond }}</td>
                                
                                <!-- Loss Diamond -->
                                <td class="px-3 py-2 text-red-500">{{ totalDeptLossQtyDiamond }}</td>
                                
                                <!-- Diamond Loss % -->
                                <td class="px-3 py-2 text-red-500">{{ parseFloat(totalDeptActualProductionDiamond) > 0 ? roundToTwo((parseFloat(totalDeptLossQtyDiamond) / parseFloat(totalDeptActualProductionDiamond)) * 100) + '%' : '0.00%' }}</td>
                                

                                <!-- Gold Recovery Weight -->
                                <td class="px-3 py-2">{{ totalDeptGoldRecoveryWeight }}</td>
                                
                                <!-- Gold Recovery Percentage -->
                                <td class="px-3 py-2">{{ globalAvgRecoveryPercentage > 0 ? roundToTwo(globalAvgRecoveryPercentage) + '%' : '' }}</td>

                                <!-- Net Loss Gold -->
                                <!-- <td class="px-3 py-2 text-red-500"></td> -->

                                <!-- Diamond Recovery Weight -->
                                <!-- <td class="px-3 py-2"></td> -->

                                <!-- Net Loss Diamond -->
                                <!-- <td class="px-3 py-2 text-red-500"></td> -->
                            </tr>
                        </tbody>

                        <!-- Employee Details -->
                        <tbody class="text-gray-700">
                            <template v-if="showEmployeesTable" v-for="(emp, empIndex) in selectedEmployees" :key="'emp-' + empIndex">
                                <!-- Employee row with categories expanded to individual bags -->
                                <template v-if="emp.unique_categories_array && emp.unique_categories_array.length > 0">
                                    <template v-for="(category, catIndex) in emp.unique_categories_array" :key="'emp-' + empIndex + '-cat-' + catIndex">
                                        <template v-for="(bagName, bagIndex) in (emp.category_bag_names_map?.[category] || [''])" :key="'emp-' + empIndex + '-cat-' + catIndex + '-bag-' + bagIndex">
                                            <tr class="border-b group hover:bg-gray-50 transition-all duration-200 text-[11px]"
                                                :class="{ 'border-b': catIndex === emp.unique_categories_array.length - 1 && bagIndex === (emp.category_bag_names_map?.[category]?.length || 1) - 1 }">
                                                <!-- SL No - only on very first row of emp -->
                                                <td v-if="catIndex === 0 && bagIndex === 0"
                                                    :rowspan="emp.unique_categories_array.reduce((s, c) => s + (emp.category_bag_names_map?.[c]?.length || 1), 0)"
                                                    class="px-3 py-2 group-hover:!bg-white">{{ empIndex + 1 }}</td>

                                                <!-- Employee Name - only on very first row -->
                                                <td v-if="catIndex === 0 && bagIndex === 0"
                                                    :rowspan="emp.unique_categories_array.reduce((s, c) => s + (emp.category_bag_names_map?.[c]?.length || 1), 0)"
                                                    class="px-3 py-2 group-hover:!bg-white">
                                                    <a v-if="getEmpNsUrl(emp)" 
                                                       :href="getEmpNsUrl(emp)"
                                                       target="_blank"
                                                       class="text-blue-600 hover:underline">{{ emp.name }}</a>
                                                    <span v-else>{{ emp.name }}</span>
                                                </td>

                                                <!-- No. of Bags - only on very first row -->
                                                <td v-if="catIndex === 0 && bagIndex === 0"
                                                    :rowspan="emp.unique_categories_array.reduce((s, c) => s + (emp.category_bag_names_map?.[c]?.length || 1), 0)"
                                                    class="px-3 py-2 font-semibold text-center bg-blue-50">{{ emp.bag_count || 0 }}</td>

                                                <!-- Item Category - rowspan = bags in this category -->
                                                <td v-if="bagIndex === 0"
                                                    :rowspan="emp.category_bag_names_map?.[category]?.length || 1"
                                                    class="px-3 py-2 group-hover:shadow-md border-r">
                                                    <a :href="getCategoryNsUrl(emp, category, bagName)"
                                                       target="_blank"
                                                       class="text-blue-600 hover:underline">{{ category || 'N/A' }}</a>
                                                </td>

                                                <!-- Sub Category - per bag -->
                                                <td class="px-3 py-2 group-hover:shadow-md border-r">
                                                    <a v-if="getSubCategoryNsUrl(emp, category, bagName)" 
                                                       :href="getSubCategoryNsUrl(emp, category, bagName)"
                                                       target="_blank"
                                                       class="text-blue-600 hover:underline">{{ emp.category_bag_sub_category_map?.[category]?.[bagName] || '-' }}</a>
                                                    <span v-else>{{ emp.category_bag_sub_category_map?.[category]?.[bagName] || '-' }}</span>
                                                </td>

                                                <!-- Bag Count (per category) - rowspan = bags in category -->
                                                <td v-if="bagIndex === 0"
                                                    :rowspan="emp.category_bag_names_map?.[category]?.length || 1"
                                                    class="px-3 py-2 group-hover:shadow-md text-center bg-blue-50">{{ emp.category_bag_count_map?.[category] || 0 }}</td>

                                                <!-- Style Number (Print Design) - per bag, no rowspan -->
                                                <td class="px-3 py-2 group-hover:shadow-md">
                                                    <a v-if="getStyleNsUrlPerBag(emp, category, bagName)" 
                                                       :href="getStyleNsUrlPerBag(emp, category, bagName)"
                                                       target="_blank"
                                                       class="text-blue-600 hover:underline">{{ emp.category_bag_print_design_map?.[category]?.[bagName] || '-' }}</a>
                                                    <span v-else>{{ emp.category_bag_print_design_map?.[category]?.[bagName] || '-' }}</span>
                                                </td>

                                                <!-- Bag Name -->
                                                <td class="px-3 py-2 group-hover:shadow-md">
                                                    <a v-if="getEmpBagNsUrl(emp, category, bagName)"
                                                       :href="getEmpBagNsUrl(emp, category, bagName)"
                                                       target="_blank"
                                                       class="text-blue-600 hover:underline">{{ bagName }}</a>
                                                    <span v-else>{{ bagName || '-' }}</span>
                                                </td>

                                                <!-- Issued Net Weight (without loss) = starting + issued - (scrap + balance) -->
                                                <td class="px-3 py-2 group-hover:shadow-md">{{ roundToTwo(getEmpBagStartingQtyGoldRaw(emp, category, bagName) + getEmpBagIssuedQtyGoldRaw(emp, category, bagName) - (getEmpBagScrapQtyGoldRaw(emp, category, bagName) + getEmpBagBalanceQtyGoldRaw(emp, category, bagName))) }}</td>

                                                <!-- Received Quantity -->
                                                <td class="px-3 py-2 group-hover:shadow-md">{{ roundToTwo(getEmpBagStartingQtyGoldRaw(emp, category, bagName) + getEmpBagIssuedQtyGoldRaw(emp, category, bagName) - (getEmpBagLossQtyGoldRaw(emp, category, bagName) + getEmpBagScrapQtyGoldRaw(emp, category, bagName) + getEmpBagBalanceQtyGoldRaw(emp, category, bagName))) }}</td>

                                                <!-- Gross Loss -->
                                                <td class="px-3 py-2 text-red-500 group-hover:shadow-md">{{ roundToTwo(getEmpBagLossQtyGoldRaw(emp, category, bagName)) }}</td>

                                                <!-- Gross Loss % -->
                                                <td class="px-3 py-2 text-red-500 group-hover:shadow-md">{{ roundToTwo(calculateGoldLossPercentage(getEmpBagStartingQtyGoldRaw(emp, category, bagName), getEmpBagIssuedQtyGoldRaw(emp, category, bagName), getEmpBagLossQtyGoldRaw(emp, category, bagName), getEmpBagScrapQtyGoldRaw(emp, category, bagName), getEmpBagBalanceQtyGoldRaw(emp, category, bagName))) }}%</td>

                                                <!-- Pure Weight -->
                                                <td class="px-3 py-2 group-hover:shadow-md">{{ getEmpBagPureWeight(emp, category, bagName) }}</td>

                                                <!-- Pure Loss -->
                                                <td class="px-3 py-2 text-red-500 group-hover:shadow-md">{{ getEmpBagPureLoss(emp, category, bagName) }}</td>
                                                
                                                <!-- Net Loss -->
                                                <td class="px-3 py-2 text-red-500 group-hover:shadow-md">{{ (() => { const recv = getEmpBagStartingQtyGoldRaw(emp, category, bagName) + getEmpBagIssuedQtyGoldRaw(emp, category, bagName) - (getEmpBagLossQtyGoldRaw(emp, category, bagName) + getEmpBagScrapQtyGoldRaw(emp, category, bagName) + getEmpBagBalanceQtyGoldRaw(emp, category, bagName)); return recv > 0 ? roundToTwo(getEmpBagLossQtyGoldRaw(emp, category, bagName) / recv) : '0.00'; })() }}</td>
                                                
                                                <!-- Net Loss % -->
                                                <td class="px-3 py-2 text-red-500 group-hover:shadow-md">{{ (() => { const recv = getEmpBagStartingQtyGoldRaw(emp, category, bagName) + getEmpBagIssuedQtyGoldRaw(emp, category, bagName) - (getEmpBagLossQtyGoldRaw(emp, category, bagName) + getEmpBagScrapQtyGoldRaw(emp, category, bagName) + getEmpBagBalanceQtyGoldRaw(emp, category, bagName)); return recv > 0 ? roundToTwo((getEmpBagLossQtyGoldRaw(emp, category, bagName) / recv) * 100) + '%' : '0.00%'; })() }}</td>

                                                <!-- TM Production Diamond -->
                                                <td class="px-3 py-2 group-hover:shadow-md">{{ roundToTwo(getEmpBagStartingQtyDiamondRaw(emp, category, bagName)) }}</td>

                                                <!-- Actual Production Diamond -->
                                                <td class="px-3 py-2 group-hover:shadow-md">{{ roundToTwo(getEmpBagStartingQtyDiamondRaw(emp, category, bagName) + getEmpBagIssuedQtyDiamondRaw(emp, category, bagName) - getEmpBagLossQtyDiamondRaw(emp, category, bagName) - getEmpBagScrapQtyDiamondRaw(emp, category, bagName) - getEmpBagBalanceQtyDiamondRaw(emp, category, bagName)) }}</td>

                                                <!-- Loss Qty Diamond -->
                                                <td class="px-3 py-2 text-red-500 group-hover:shadow-md">{{ roundToTwo(getEmpBagLossQtyDiamondRaw(emp, category, bagName)) }}</td>

                                                <!-- Diamond Loss % -->
                                                <td class="px-3 py-2 text-red-500 group-hover:shadow-md">{{ roundToTwo(calculateDiamondLossPercentage(getEmpBagStartingQtyDiamondRaw(emp, category, bagName), getEmpBagIssuedQtyDiamondRaw(emp, category, bagName), getEmpBagLossQtyDiamondRaw(emp, category, bagName), getEmpBagScrapQtyDiamondRaw(emp, category, bagName), getEmpBagBalanceQtyDiamondRaw(emp, category, bagName))) }}%</td>


                                                <!-- Gold Recovery Weight -->
                                                <td class="px-3 py-2 group-hover:shadow-md">
                                                    {{ (getEmpBagLossQtyGoldRaw(emp, category, bagName) * getEmpBagPurityFactor(emp, category, bagName)) > 0 && globalAvgRecoveryPercentage > 0
                                                        ? roundToTwo(getEmpBagLossQtyGoldRaw(emp, category, bagName) * getEmpBagPurityFactor(emp, category, bagName) * (globalAvgRecoveryPercentage / 100))
                                                        : '0.00' }}
                                                </td>
                                                
                                                <!-- Gold Recovery Percentage -->
                                                <td class="px-3 py-2 group-hover:shadow-md">
                                                    {{ (getEmpBagLossQtyGoldRaw(emp, category, bagName) * getEmpBagPurityFactor(emp, category, bagName)) > 0 && globalAvgRecoveryPercentage > 0
                                                        ? roundToTwo(globalAvgRecoveryPercentage) + '%'
                                                        : '0%' }}
                                                </td>

                                                <!-- Net Loss Gold -->
                                                <!-- <td class="px-3 py-2 text-red-500 group-hover:shadow-md">-</td> -->

                                                <!-- Diamond Recovery Weight -->
                                                <!-- <td class="px-3 py-2 group-hover:shadow-md">-</td> -->

                                                <!-- Net Loss Diamond -->
                                                <!-- <td class="px-3 py-2 group-hover:shadow-md">-</td> -->
                                            </tr>
                                        </template>
                                    </template>
                                </template>

                                <!-- Employee Total Row -->
                                <tr class="bg-blue-50 font-bold border-b-8 border-b-gray-200 border-blue-200 text-[11px]">
                                    <td class="px-3 py-2"></td>
                                    <td class="px-3 py-2 text-center" colspan="4">{{ emp.name }} — Total</td>
                                    <td class="px-3 py-2 text-center bg-blue-100">{{ emp.bag_count || 0 }}</td>
                                    <td class="px-3 py-2" colspan="2"></td>
                                    <td class="px-3 py-2">{{ getEmpTotals(emp).issuedNetWt }}</td>
                                    <td class="px-3 py-2">{{ getEmpTotals(emp).recvG }}</td>
                                    <td class="px-3 py-2 text-red-500">{{ getEmpTotals(emp).lG }}</td>
                                    <td class="px-3 py-2 text-red-500">{{ getEmpTotals(emp).grossLossPct }}</td>
                                    <td class="px-3 py-2">{{ getEmpTotals(emp).pw }}</td>
                                    <td class="px-3 py-2 text-red-500">{{ getEmpTotals(emp).pl }}</td>
                                    <td class="px-3 py-2 text-red-500">{{ getEmpTotals(emp).netLoss }}</td>
                                    <td class="px-3 py-2 text-red-500">{{ getEmpTotals(emp).netLossPct }}</td>
                                    <td class="px-3 py-2">{{ getEmpTotals(emp).sD }}</td>
                                    <td class="px-3 py-2">{{ getEmpTotals(emp).aD }}</td>
                                    <td class="px-3 py-2 text-red-500">{{ getEmpTotals(emp).lD }}</td>
                                    <td class="px-3 py-2 text-red-500">{{ getEmpTotals(emp).diamondLossPct }}</td>
                                    <td class="px-3 py-2">{{ getEmpTotals(emp).recoveryWt }}</td>
                                    <td class="px-3 py-2">{{ getEmpTotals(emp).recoveryPct }}</td>
                                </tr>
                            </template>

                            <!-- No data found row when selectedEmployees is empty -->
                            <tr v-if="showEmployeesTable && selectedEmployees.length === 0" class="border-b group hover:bg-gray-50 transition-all duration-200 text-[11px]">
                                <td colspan="16" style="text-align: center;" class="px-3 py-2 text-gray-900 italic">No data found for this Employee</td>
                            </tr>

                            <!-- Total Row for Employees -->
                            <tr v-if="showEmployeesTable" class="bg-gray-100 font-bold border-t text-[11px]">
                                <td class="px-3 py-2" colspan="2" style="text-align: center;">Total</td>

                                <!-- Total No. of Bags -->
                                <td class="px-3 py-2 font-semibold text-center bg-blue-50">{{ totalEmpBagCount }}</td>
                                
                                <!-- Item Category (empty) -->
                                <td class="px-3 py-2"></td>
                                
                                <!-- Item Sub Category (empty) -->
                                <td class="px-3 py-2"></td>

                                <!-- Style Number (empty) -->
                                <td class="px-3 py-2"></td>

                                <!-- Bag Count (empty) -->
                                <td class="px-3 py-2"></td>

                                <!-- Bag Name (empty) -->
                                <td class="px-3 py-2"></td>

                                <!-- Issued Net Weight -->
                                <td class="px-3 py-2">{{ totalEmpIssuedNetWeightGold }}</td>

                                <!-- Received Quantity -->
                                <td class="px-3 py-2">{{ totalEmpReceivedQtyGold }}</td>
                                
                                <!-- Gross Loss -->
                                <td class="px-3 py-2 text-red-500">{{ totalEmpLossQuantityGold }}</td>
                                
                                <!-- Gross Loss % -->
                                <td class="px-3 py-2 text-red-500">{{ parseFloat(totalEmpReceivedQtyGold) > 0 ? roundToTwo((parseFloat(totalEmpLossQuantityGold) / parseFloat(totalEmpReceivedQtyGold)) * 100) + '%' : '0.00%' }}</td>
                                
                                <!-- Pure Weight -->
                                <td class="px-3 py-2">{{ totalEmpPureWeight }}</td>

                                <!-- Pure Loss -->
                                <td class="px-3 py-2 text-red-500">{{ totalEmpPureLoss }}</td>
                                
                                <!-- Net Loss -->
                                <td class="px-3 py-2 text-red-500">{{ parseFloat(totalEmpReceivedQtyGold) > 0 ? roundToTwo(parseFloat(totalEmpLossQuantityGold) / parseFloat(totalEmpReceivedQtyGold)) : '0.00' }}</td>

                                <!-- Net Loss % -->
                                <td class="px-3 py-2 text-red-500">{{ parseFloat(totalEmpReceivedQtyGold) > 0 ? roundToTwo((parseFloat(totalEmpLossQuantityGold) / parseFloat(totalEmpReceivedQtyGold)) * 100) + '%' : '0.00%' }}</td>

                                <!-- Starting Qty Diamond -->
                                <td class="px-3 py-2">{{ totalEmpStartingQuantityDiamond }}</td>

                                <!-- Issued Qty Diamond -->
                                <!-- <td class="px-3 py-2">{{ totalEmpIssuedQuantityDiamond }}</td> -->
                                
                                <!-- Actual Production Diamond -->
                                <td class="px-3 py-2">{{ totalEmpActualProductionDiamondCalculated }}</td>

                                <!-- Loss Qty Diamond -->
                                <td class="px-3 py-2 text-red-500">{{ totalEmpLossQuantityDiamond }}</td>

                                <!-- Diamond Loss % -->
                                <td class="px-3 py-2 text-red-500">{{ parseFloat(totalEmpActualProductionDiamondCalculated) > 0 ? roundToTwo((parseFloat(totalEmpLossQuantityDiamond) / parseFloat(totalEmpActualProductionDiamondCalculated)) * 100) + '%' : '0.00%' }}</td>

                                <!-- Gold Recovery Weight -->
                                <td class="px-3 py-2">{{ totalEmpGoldRecoveryWeight }}</td>
                                
                                <!-- Gold Recovery Percentage -->
                                <td class="px-3 py-2">{{ globalAvgRecoveryPercentage > 0 ? roundToTwo(globalAvgRecoveryPercentage) + '%' : '' }}</td>

                                <!-- Net Loss Gold -->
                                <!-- <td class="px-3 py-2 text-red-500"></td> -->
                                
                                <!-- Diamond Recovery Weight -->
                                <!-- <td class="px-3 py-2"></td> -->

                                <!-- Net Loss Diamond -->
                                <!-- <td class="px-3 py-2 text-red-500"></td> -->
                            </tr>
                        </tbody>

                    </table>
                </div>
            </div>
            <!-- Chart & Production Section -->
            <div class="grid grid-cols-1 xs:grid-cols-1 sm:grid-cols-1 md:grid-cols-1 lg:grid-cols-1 xl:grid-cols-2 gap-4 mt-4">
                <!-- Doughnut: Gold vs Diamond production split -->
                <div class="p-4 rounded-lg shadow w-full h-80 flex flex-col bg-transparent/2">
                    <h2 class="text-sm font-bold mb-2 text-center text-gray-700">Production</h2>
                    <div class="relative flex-1">
                        <canvas id="productionChart"></canvas>
                    </div>
                </div>
                <!-- Bar/Line: Production & Loss per entity -->
                <div class="p-4 rounded-lg shadow w-full h-80 flex flex-col bg-transparent/2">
                    <h2 class="text-sm font-bold mb-2 text-center text-gray-700">{{ dashboardTitle }} — Gold & Diamond</h2>
                    <div class="relative flex-1">
                        <canvas id="cryptoChart"></canvas>
                    </div>
                </div>
            </div>
        </main>
    </div>
</template>

<script>
import { onMounted, ref, watch, computed } from 'vue';
import Chart from 'chart.js/auto';
import { useAllEfficiencyAnalysisApi } from "@/composables/reports-api/efficiency-analysis-api/FetchEfficiencyAnalysisApi";
import VueDatePicker from '@vuepic/vue-datepicker';
import '@vuepic/vue-datepicker/dist/main.css';
import ExcelJS from 'exceljs';
import { saveAs } from 'file-saver';
import { ENV_VAR } from "@/shared/constants";

export default {
    components: {
        VueDatePicker,
    },
    props: {
        type: {
            type: String,
            default: 'overall', // 'overall' or 'repair'
        }
    },
    setup(props) {

        const expandedLocation = ref(null);
        const expandedDepartment = ref(null);
        const showDepartments = ref(false);
        const showEmployees = ref(false);
        const selectedDateRange = ref(null); // Holds selected date range
        const baseTitle = props.type === 'repair' ? "Repair Efficiency Analysis" : props.type === 'production' ? "Production Efficiency Analysis" : "Overall Efficiency Analysis";
        const dashboardTitle = ref(baseTitle);
        const selectedDepartments = ref([]);
        const selectedEmployees = ref([]);
        const showTable = ref(false);
        const selectedDepartmentData = ref([]);
        const showEmployeesTable = ref(false);
        const showNoDataMessage = ref(false); // Show message if no data is available after retries

        let cryptoChart = null;
        let productionChart = null;
        const {
            loading,
            loadingComponents,
            error,
            fetchlistLocations,
            listLocationsData,
            listEfficiencyData,
            fetchListEfficiencyAnalysis,
            fetchInventoryAdjustments,
            listInventoryAdjustments,
            fetchRecoveryDataByDept,
        } = useAllEfficiencyAnalysisApi();
        
        const to = ref(false); // ✅ Controls the visibility of the popup
        const locations = ref([]);
        const isInitialLoading = ref(true);
        const showNoDataPopup = ref(false); // ✅ Define showNoDataPopup as a reactive variable
        const globalAvgRecoveryPercentage = ref(0);

        const CASTING = Number(ENV_VAR.CONSTANTS.DEPARTMENT_CASTING);
        const TREE_CUTTING_CLEANING = Number(ENV_VAR.CONSTANTS.DEPARTMENT_TREE_CUTTING_CLEANING);
        const GRINDING = Number(ENV_VAR.CONSTANTS.DEPARTMENT_GRINDING);

        // Function to get the start of the current month and today’s date
        // const getDefaultDateRange = () => {
        //     const today = new Date();
        //     const firstDayOfMonth = new Date(today.getFullYear(), today.getMonth(), 1);
        //     return [firstDayOfMonth, today]; // Returns the range [start of month, today]
        // };
        const getDefaultDateRange = () => {
            const today = new Date();
            const yesterday = new Date(today);
            yesterday.setDate(today.getDate() - 1);
            // return [yesterday, yesterday]; // Returns range: [yesterday, yesterday]
            return [today, today]; // Returns range: [today, today]
        };
        selectedDateRange.value = getDefaultDateRange(); // Initialize with today's date range
        const getDefaultDateRangeLast = () => {
            const today = new Date();

            // Subtract 1 year from today's date
            const lastYear = today.getFullYear() - 1;

            // Get the first day of the month, one year ago
            const firstDayOfLastYearMonth = new Date(lastYear, today.getMonth(), 1);

            // If today is the first day of the month, return the last day of the previous month from last year
            if (today.getDate() === 1) {
                const lastDayPrevMonth = new Date(lastYear, today.getMonth(), 0);
                return [lastDayPrevMonth, today]; // Returns range: [last day of previous month, today]
            }

            return [firstDayOfLastYearMonth, today]; // Normal case
        };


        // Compute Department Table Totals (sum all rows in table)
        const totalDeptStartingQuantityGold = computed(() => {
            let sum = 0;
            selectedDepartmentData.value.forEach(dept => {
                if (dept.category_qty_map) {
                    Object.keys(dept.category_qty_map).forEach(key => {
                        if (key.startsWith(`${dept.id}_`)) {
                            const catData = dept.category_qty_map[key];
                            sum += parseFloat(catData.starting_qty_gold || 0);
                        }
                    });
                }
            });
            return roundToTwo(sum);
        });

        const totalDeptStartingQuantityDiamond = computed(() => {
            let sum = 0;
            selectedDepartmentData.value.forEach(dept => {
                if (dept.category_qty_map) {
                    Object.keys(dept.category_qty_map).forEach(key => {
                        if (key.startsWith(`${dept.id}_`)) {
                            const catData = dept.category_qty_map[key];
                            sum += parseFloat(catData.starting_qty_diamond || 0);
                        }
                    });
                }
            });
            return roundToTwo(sum);
        });

        const totalDeptActualProductionGold = computed(() => {
            let sum = 0;
            selectedDepartmentData.value.forEach(dept => {
                if (dept.category_qty_map) {
                    Object.keys(dept.category_qty_map).forEach(key => {
                        if (key.startsWith(`${dept.id}_`)) {
                            const catData = dept.category_qty_map[key];
                            const actualProduction = (parseFloat(catData.starting_qty_gold || 0) + parseFloat(catData.issued_qty_gold || 0) - parseFloat(catData.loss_qty_gold || 0) - parseFloat(catData.scrap_qty_gold || 0) - parseFloat(catData.balance_qty_gold || 0));
                            sum += actualProduction;
                        }
                    });
                }
            });
            return roundToTwo(sum);
        });

        const totalDeptGrossLossGold = computed(() => {
            let sum = 0;
            selectedDepartmentData.value.forEach(dept => {
                sum += parseFloat(dept.loss || 0);
            });
            return roundToTwo(sum);
        });

        const totalDeptActualProductionDiamond = computed(() => {
            let sum = 0;
            selectedDepartmentData.value.forEach(dept => {
                if (dept.category_qty_map) {
                    Object.keys(dept.category_qty_map).forEach(key => {
                        if (key.startsWith(`${dept.id}_`)) {
                            const catData = dept.category_qty_map[key];
                            const actualProduction = (parseFloat(catData.starting_qty_diamond || 0) + parseFloat(catData.issued_qty_diamond || 0) - parseFloat(catData.loss_qty_diamond || 0) - parseFloat(catData.scrap_qty_diamond || 0) - parseFloat(catData.balance_qty_diamond || 0));
                            sum += actualProduction;
                        }
                    });
                }
            });
            return roundToTwo(sum);
        });

        const totalDeptGrossLossDiamond = computed(() => {
            let sum = 0;
            selectedDepartmentData.value.forEach(dept => {
                sum += parseFloat(dept.loss_diamond || 0);
            });
            return roundToTwo(sum);
        });

        const totalDeptGoldRecoveryWeight = computed(() => {
            if (!globalAvgRecoveryPercentage.value) return '0.00';
            let sum = 0;
            selectedDepartmentData.value.forEach(dept => {
                (dept.unique_categories_array || []).forEach(cat => {
                    (dept.category_bag_names_map?.[cat] || []).forEach(bagName => {
                        const pureLoss = getBagLossQtyGoldRaw(dept, cat, bagName) * getBagPurityFactor(dept, cat, bagName);
                        if (pureLoss > 0) {
                            sum += pureLoss * (globalAvgRecoveryPercentage.value / 100);
                        }
                    });
                });
            });
            return roundToTwo(sum);
        });

        const totalDeptNetLossGold = computed(() => {
            let sum = 0;
            selectedDepartmentData.value.forEach(dept => {
                sum += parseFloat(dept.netLoss || 0);
            });
            return roundToTwo(sum);
        });

        const totalDeptTmProductionGold = computed(() => {
            let sum = 0;
            selectedDepartmentData.value.forEach(dept => {
                sum += parseFloat(dept.production || 0);
            });
            return roundToTwo(sum);
        });

        const totalDeptTmProductionDiamond = computed(() => {
            let sum = 0;
            selectedDepartmentData.value.forEach(dept => {
                sum += parseFloat(dept.production_diamond || 0);
            });
            return roundToTwo(sum);
        });

        const totalDeptBagCount = computed(() => {
            // Count unique bags across all departments
            const uniqueBags = new Set();
            selectedDepartmentData.value.forEach(dept => {
                if (dept.unique_bags_array && Array.isArray(dept.unique_bags_array)) {
                    dept.unique_bags_array.forEach(bag => uniqueBags.add(bag));
                }
            });
            // If no unique_bags_array, fallback to summing bag_count
            if (uniqueBags.size === 0) {
                let sum = 0;
                selectedDepartmentData.value.forEach(dept => {
                    sum += parseInt(dept.bag_count || 0);
                });
                return sum;
            }
            return uniqueBags.size;
        });

        const totalDeptIssuedQtyGold = computed(() => {
            let sum = 0;
            selectedDepartmentData.value.forEach(dept => {
                if (dept.category_qty_map) {
                    Object.keys(dept.category_qty_map).forEach(key => {
                        if (key.startsWith(`${dept.id}_`)) {
                            sum += parseFloat(dept.category_qty_map[key].issued_qty_gold || 0);
                        }
                    });
                }
            });
            return roundToTwo(sum);
        });

        const totalDeptLossQtyGold = computed(() => {
            let sum = 0;
            selectedDepartmentData.value.forEach(dept => {
                if (dept.category_qty_map) {
                    Object.keys(dept.category_qty_map).forEach(key => {
                        if (key.startsWith(`${dept.id}_`)) {
                            const catData = dept.category_qty_map[key];
                            sum += parseFloat(catData.loss_qty_gold || 0);
                        }
                    });
                }
            });
            return roundToTwo(sum);
        });

        const totalDeptScrapQtyGold = computed(() => {
            let sum = 0;
            selectedDepartmentData.value.forEach(dept => {
                if (dept.category_qty_map) {
                    Object.keys(dept.category_qty_map).forEach(key => {
                        if (key.startsWith(`${dept.id}_`)) {
                            const catData = dept.category_qty_map[key];
                            sum += parseFloat(catData.scrap_qty_gold || 0);
                        }
                    });
                }
            });
            return roundToTwo(sum);
        });

        // Starting Net Weight total = starting + issued - (loss + scrap)
        const totalDeptNetStartingGold = computed(() => {
            const s = parseFloat(totalDeptStartingQuantityGold.value || 0);
            const i = parseFloat(totalDeptIssuedQtyGold.value || 0);
            const l = parseFloat(totalDeptLossQtyGold.value || 0);
            const sc = parseFloat(totalDeptScrapQtyGold.value || 0);
            return roundToTwo(s + i - (l + sc));
        });

        // Received Quantity total = starting + issued - (loss + scrap + balance)
        const totalDeptReceivedQtyGold = computed(() => {
            let sum = 0;
            selectedDepartmentData.value.forEach(dept => {
                if (dept.category_qty_map) {
                    Object.keys(dept.category_qty_map).forEach(key => {
                        if (key.startsWith(`${dept.id}_`)) {
                            const d = dept.category_qty_map[key];
                            sum += parseFloat(d.starting_qty_gold || 0)
                                 + parseFloat(d.issued_qty_gold || 0)
                                 - parseFloat(d.loss_qty_gold || 0)
                                 - parseFloat(d.scrap_qty_gold || 0)
                                 - parseFloat(d.balance_qty_gold || 0);
                        }
                    });
                }
            });
            return roundToTwo(sum);
        });

        // Issued Net Weight total = starting + issued - (scrap + balance)  [loss NOT subtracted]
        const totalDeptIssuedNetWeightGold = computed(() => {
            let sum = 0;
            selectedDepartmentData.value.forEach(dept => {
                if (dept.category_qty_map) {
                    Object.keys(dept.category_qty_map).forEach(key => {
                        if (key.startsWith(`${dept.id}_`)) {
                            const d = dept.category_qty_map[key];
                            sum += parseFloat(d.starting_qty_gold || 0)
                                 + parseFloat(d.issued_qty_gold || 0)
                                 - parseFloat(d.scrap_qty_gold || 0)
                                 - parseFloat(d.balance_qty_gold || 0);
                        }
                    });
                }
            });
            return roundToTwo(sum);
        });

        const totalDeptIssuedQtyDiamond = computed(() => {
            let sum = 0;
            selectedDepartmentData.value.forEach(dept => {
                sum += parseFloat(dept.issued_quantity_diamond || 0);
            });
            return roundToTwo(sum);
        });

        const totalDeptLossQtyDiamond = computed(() => {
            let sum = 0;
            selectedDepartmentData.value.forEach(dept => {
                if (dept.category_qty_map) {
                    Object.keys(dept.category_qty_map).forEach(key => {
                        if (key.startsWith(`${dept.id}_`)) {
                            const catData = dept.category_qty_map[key];
                            sum += parseFloat(catData.loss_qty_diamond || 0);
                        }
                    });
                }
            });
            return roundToTwo(sum);
        });

        const totalEmpTmProductionGold = computed(() => {
            let sum = 0;
            selectedEmployees.value.forEach(emp => {
                sum += parseFloat(emp.tmProduction || 0);
            });
            return roundToTwo(sum);
        });

        const totalEmpTmProductionDiamond = computed(() => {
            let sum = 0;
            selectedEmployees.value.forEach(emp => {
                sum += parseFloat(emp.tmProductionDiamond || 0);
            });
            return roundToTwo(sum);
        });

        const totalEmpBagCount = computed(() => {
            // Count unique bags across all employees
            const uniqueBags = new Set();
            selectedEmployees.value.forEach(emp => {
                if (emp.unique_bags_array && Array.isArray(emp.unique_bags_array)) {
                    emp.unique_bags_array.forEach(bag => uniqueBags.add(bag));
                }
            });
            // If no unique_bags_array, fallback to summing bag_count
            if (uniqueBags.size === 0) {
                let sum = 0;
                selectedEmployees.value.forEach(emp => {
                    sum += parseInt(emp.bag_count || 0);
                });
                return sum;
            }
            return uniqueBags.size;
        });

        // Compute Employee Table Totals (sum all rows in table)
        const totalEmpActualProductionGold = computed(() => {
            let sum = 0;
            selectedEmployees.value.forEach(emp => {
                sum += parseFloat(emp.actualProductionGold || 0);
            });
            return roundToTwo(sum);
        });

        const totalEmpGrossLossGold = computed(() => {
            let sum = 0;
            selectedEmployees.value.forEach(emp => {
                sum += parseFloat(emp.grossLoss || 0);
            });
            return roundToTwo(sum);
        });

        const totalEmpActualProductionDiamond = computed(() => {
            let sum = 0;
            selectedEmployees.value.forEach(emp => {
                sum += parseFloat(emp.actualProductionDiamond || 0);
            });
            return roundToTwo(sum);
        });

        const totalEmpGrossLossDiamond = computed(() => {
            let sum = 0;
            selectedEmployees.value.forEach(emp => {
                sum += parseFloat(emp.grossLossDiamond || 0);
            });
            return roundToTwo(sum);
        });

        const totalEmpGoldRecoveryWeight = computed(() => {
            if (!globalAvgRecoveryPercentage.value) return '0.00';
            let sum = 0;
            selectedEmployees.value.forEach(emp => {
                (emp.unique_categories_array || []).forEach(cat => {
                    (emp.category_bag_names_map?.[cat] || []).forEach(bagName => {
                        const pureLoss = getEmpBagLossQtyGoldRaw(emp, cat, bagName) * getEmpBagPurityFactor(emp, cat, bagName);
                        if (pureLoss > 0) {
                            sum += pureLoss * (globalAvgRecoveryPercentage.value / 100);
                        }
                    });
                });
            });
            return roundToTwo(sum);
        });

        const totalEmpNetLossGold = computed(() => {
            let sum = 0;
            selectedEmployees.value.forEach(emp => {
                sum += parseFloat(emp.netLoss || 0);
            });
            return roundToTwo(sum);
        });

        const totalEmpStartingQuantityGold = computed(() => {
            let sum = 0;
            selectedEmployees.value.forEach(emp => {
                (emp.categories || []).forEach(cat => { sum += parseFloat(cat.starting_qty_gold || 0); });
            });
            return roundToTwo(sum);
        });

        const totalEmpIssuedQuantityGold = computed(() => {
            let sum = 0;
            selectedEmployees.value.forEach(emp => {
                (emp.categories || []).forEach(cat => { sum += parseFloat(cat.issued_qty_gold || 0); });
            });
            return roundToTwo(sum);
        });

        const totalEmpLossQuantityGold = computed(() => {
            let sum = 0;
            selectedEmployees.value.forEach(emp => {
                (emp.categories || []).forEach(cat => { sum += parseFloat(cat.loss_qty_gold || 0); });
            });
            return roundToTwo(sum);
        });

        const totalEmpScrapQtyGold = computed(() => {
            let sum = 0;
            selectedEmployees.value.forEach(emp => {
                (emp.categories || []).forEach(cat => { sum += parseFloat(cat.scrap_qty_gold || 0); });
            });
            return roundToTwo(sum);
        });

        // Starting Net Weight total = starting + issued - (loss + scrap)
        const totalEmpNetStartingGold = computed(() => {
            const s = parseFloat(totalEmpStartingQuantityGold.value || 0);
            const i = parseFloat(totalEmpIssuedQuantityGold.value || 0);
            const l = parseFloat(totalEmpLossQuantityGold.value || 0);
            const sc = parseFloat(totalEmpScrapQtyGold.value || 0);
            return roundToTwo(s + i - (l + sc));
        });

        // Received Quantity total = starting + issued - (loss + scrap + balance)
        const totalEmpReceivedQtyGold = computed(() => {
            let sum = 0;
            selectedEmployees.value.forEach(emp => {
                (emp.categories || []).forEach(cat => {
                    sum += parseFloat(cat.starting_qty_gold || 0)
                         + parseFloat(cat.issued_qty_gold || 0)
                         - parseFloat(cat.loss_qty_gold || 0)
                         - parseFloat(cat.scrap_qty_gold || 0)
                         - parseFloat(cat.balance_qty_gold || 0);
                });
            });
            return roundToTwo(sum);
        });

        // Issued Net Weight total = starting + issued - (scrap + balance)  [loss NOT subtracted]
        const totalEmpIssuedNetWeightGold = computed(() => {
            let sum = 0;
            selectedEmployees.value.forEach(emp => {
                (emp.categories || []).forEach(cat => {
                    sum += parseFloat(cat.starting_qty_gold || 0)
                         + parseFloat(cat.issued_qty_gold || 0)
                         - parseFloat(cat.scrap_qty_gold || 0)
                         - parseFloat(cat.balance_qty_gold || 0);
                });
            });
            return roundToTwo(sum);
        });

        const totalEmpStartingQuantityDiamond = computed(() => {
            let sum = 0;
            selectedEmployees.value.forEach(emp => {
                (emp.categories || []).forEach(cat => { sum += parseFloat(cat.starting_qty_diamond || 0); });
            });
            return roundToTwo(sum);
        });

        const totalEmpIssuedQuantityDiamond = computed(() => {
            let sum = 0;
            selectedEmployees.value.forEach(emp => {
                (emp.categories || []).forEach(cat => { sum += parseFloat(cat.issued_qty_diamond || 0); });
            });
            return roundToTwo(sum);
        });

        const totalEmpLossQuantityDiamond = computed(() => {
            let sum = 0;
            selectedEmployees.value.forEach(emp => {
                (emp.categories || []).forEach(cat => { sum += parseFloat(cat.loss_qty_diamond || 0); });
            });
            return roundToTwo(sum);
        });

        const totalEmpActualProductionGoldCalculated = computed(() => {
            let sum = 0;
            selectedEmployees.value.forEach(emp => {
                (emp.categories || []).forEach(cat => {
                    sum += parseFloat(cat.starting_qty_gold || 0) + parseFloat(cat.issued_qty_gold || 0)
                         - parseFloat(cat.loss_qty_gold || 0) - parseFloat(cat.scrap_qty_gold || 0)
                         - parseFloat(cat.balance_qty_gold || 0);
                });
            });
            return roundToTwo(sum);
        });

        const totalEmpActualProductionDiamondCalculated = computed(() => {
            let sum = 0;
            selectedEmployees.value.forEach(emp => {
                (emp.categories || []).forEach(cat => {
                    sum += parseFloat(cat.starting_qty_diamond || 0) + parseFloat(cat.issued_qty_diamond || 0)
                         - parseFloat(cat.loss_qty_diamond || 0) - parseFloat(cat.scrap_qty_diamond || 0)
                         - parseFloat(cat.balance_qty_diamond || 0);
                });
            });
            return roundToTwo(sum);
        });

        const totalDeptPureWeight = computed(() => {
            let sum = 0;
            selectedDepartmentData.value.forEach(dept => {
                (dept.unique_categories_array || []).forEach(cat => {
                    (dept.category_bag_names_map?.[cat] || []).forEach(bagName => {
                        const bKey = `${dept.id}_${cat}_${bagName}`;
                        const d = (dept.category_bag_qty_map || {})[bKey] || {};
                        const purity = parseFloat(d.metal_purity_percent || 0) / 100;
                        const aG = parseFloat(d.starting_qty_gold || 0) + parseFloat(d.issued_qty_gold || 0) - parseFloat(d.loss_qty_gold || 0) - parseFloat(d.scrap_qty_gold || 0) - parseFloat(d.balance_qty_gold || 0);
                        sum += aG * purity;
                    });
                });
            });
            return roundToTwo(sum);
        });

        const totalDeptPureLoss = computed(() => {
            let sum = 0;
            selectedDepartmentData.value.forEach(dept => {
                (dept.unique_categories_array || []).forEach(cat => {
                    (dept.category_bag_names_map?.[cat] || []).forEach(bagName => {
                        const bKey = `${dept.id}_${cat}_${bagName}`;
                        const d = (dept.category_bag_qty_map || {})[bKey] || {};
                        const purity = parseFloat(d.metal_purity_percent || 0) / 100;
                        sum += parseFloat(d.loss_qty_gold || 0) * purity;
                    });
                });
            });
            return roundToTwo(sum);
        });

        const totalEmpPureWeight = computed(() => {
            let sum = 0;
            selectedEmployees.value.forEach(emp => {
                (emp.categories || []).forEach(cat => {
                    const purity = parseFloat(cat.metal_purity_percent || 0) / 100;
                    const aG = parseFloat(cat.starting_qty_gold || 0) + parseFloat(cat.issued_qty_gold || 0) - parseFloat(cat.loss_qty_gold || 0) - parseFloat(cat.scrap_qty_gold || 0) - parseFloat(cat.balance_qty_gold || 0);
                    sum += aG * purity;
                });
            });
            return roundToTwo(sum);
        });

        const totalEmpPureLoss = computed(() => {
            let sum = 0;
            selectedEmployees.value.forEach(emp => {
                (emp.categories || []).forEach(cat => {
                    const purity = parseFloat(cat.metal_purity_percent || 0) / 100;
                    sum += parseFloat(cat.loss_qty_gold || 0) * purity;
                });
            });
            return roundToTwo(sum);
        });

        // Function to format the name of locations, departments, and employees
        const formatName = (name) => {
            if (!name) return "";
            return name
                .toLowerCase()
                .replace(/\b\w/g, (char) => char.toUpperCase()); // Capitalizes the first letter of each word
        };

        // Helper function to calculate gold loss percentage using loss qty and actual production
        const calculateGoldLossPercentage = (startingGold, issuedGold, lossGold, scrapGold, balanceGold) => {
            const starting = startingGold || 0;
            const issued = issuedGold || 0;
            const loss = lossGold || 0;
            const scrap = scrapGold || 0;
            const balance = balanceGold || 0;
            
            // Calculate actual production: starting + issued - loss - scrap - balance
            const actualProduction = starting + issued - loss - scrap - balance;
            
            // If actual production is 0 or negative, return 0
            if (actualProduction <= 0) return 0;
            
            // Loss percentage = (loss / actual production) * 100
            return (loss / actualProduction) * 100;
        };

        // Helper function to calculate diamond loss percentage using loss qty and actual production
        const calculateDiamondLossPercentage = (startingDiamond, issuedDiamond, lossDiamond, scrapDiamond, balanceDiamond) => {
            const starting = startingDiamond || 0;
            const issued = issuedDiamond || 0;
            const loss = lossDiamond || 0;
            const scrap = scrapDiamond || 0;
            const balance = balanceDiamond || 0;
            
            // Calculate actual production: starting + issued - loss - scrap - balance
            const actualProduction = starting + issued - loss - scrap - balance;
            
            // If actual production is 0 or negative, return 0
            if (actualProduction <= 0) return 0;
            
            // Loss percentage = (loss / actual production) * 100
            return (loss / actualProduction) * 100;
        };

        // Set default date range on mount
        selectedDateRange.value = getDefaultDateRange();

        // fetchLocations function
        const fetchLocations = async () => {
            try {
                loading.value = true;
                isInitialLoading.value = true;
                await fetchlistLocations();
                if (listLocationsData.value) {
                    locations.value = listLocationsData.value.map(location => ({
                        internalid: { value: location.internalid.value },
                        name: { value: location.name.value },
                        production: "0", // Set default values
                        loss: "0",
                        departments: [] // Departments will be fetched dynamically
                    }));
                }
            } catch (err) {
                console.error("Error fetching locations:", err);
            } finally {
                loading.value = false;
                isInitialLoading.value = false;
            }
        };

        const fetchInventoryAdjustmentsDetails = async (startDate, endDate) => {
            try {
                loading.value = true;
                // Pass the dates to the API call if it expects them
                await fetchInventoryAdjustments(startDate, endDate);
            } catch (err) {
                console.error("Error fetching inventory Adjustment:", err);
            } finally {
                loading.value = false;
            }
        };
        // // download data function
        // const downloadData = () => {
        //     if (!listEfficiencyData.value || Object.keys(listEfficiencyData.value).length === 0) {
        //         alert("No data available for download.");
        //         return;
        //     }

        //     // ✅ Format the selected date range correctly
        //     const formatDate = (date) => {
        //         const offset = date.getTimezoneOffset();
        //         date = new Date(date.getTime() - (offset * 60 * 1000));
        //         return date.toISOString().split('T')[0];
        //     };

        //     const startDate = selectedDateRange.value[0] ? formatDate(selectedDateRange.value[0]) : "N/A";
        //     const endDate = selectedDateRange.value[1] ? formatDate(selectedDateRange.value[1]) : "N/A";

        //     // ✅ Retrieve Inventory Adjustments (Finding Recovery Weight Like `fetchEfficiencyAnalysisData`)
        //     let inventoryAdjustments = listInventoryAdjustments.value || [];
        //     console.log("Inventory Adjustments:", inventoryAdjustments);

        //     let departmentQuantities = {};
        //     let employeeQuantities = {}; // ✅ Store Department-Employee Key

        //     // **Process inventory adjustments to aggregate quantities by department & employee**
        //     inventoryAdjustments.forEach(item => {
        //         const departmentId = item.custbody_jj_recovery_department;
        //         const employeeId = item.custbody_jj_recovered_employee;
        //         const key = `${departmentId}_${employeeId}`; // ✅ Department-Specific Employee Key
        //         const quantity = parseFloat(item.quantity || 0);

        //         // Aggregate by department
        //         if (!departmentQuantities[departmentId]) {
        //             departmentQuantities[departmentId] = 0;
        //         }
        //         departmentQuantities[departmentId] += quantity;

        //         // Aggregate by department-employee key
        //         if (!employeeQuantities[key]) {
        //             employeeQuantities[key] = 0;
        //         }
        //         employeeQuantities[key] += quantity;
        //     });

        //     console.log("Aggregated Department Quantities:", departmentQuantities);
        //     console.log("Aggregated Employee Quantities:", employeeQuantities);

        //     // ✅ CSV Header
        //     let csvContent = `Efficiency Report\nDate Range: ${startDate} to ${endDate}\n\n`;
        //     csvContent += "Location, Department, Employee, TM Production Gold (g), Gross Loss Gold (g), TM Production Diamond (carat), Gross Loss Diamond (carat), TM Production Diamond (pieces), Gross Loss Diamond (pieces), Gold Recovery Weight (gm), Net Loss Gold, Diamond Recovery Weight (ct), Net Loss Diamond\n";

        //     Object.entries(listEfficiencyData.value).forEach(([locationId, location]) => {
        //         Object.entries(location.departments || {}).forEach(([deptId, department]) => {
        //             Object.entries(department.employees || {}).forEach(([empId, employee]) => {

        //                 // ✅ Find `tmGrossLossWeight` (Gold Recovery Weight) Using Department-Specific Key
        //                 const key = `${deptId}_${empId}`;
        //                 const goldRecoveryWeight = employeeQuantities[key] || 0;
        //                 const netLossGold = (parseFloat(employee.grossLoss || 0) - goldRecoveryWeight).toFixed(2);

        //                 csvContent += [
        //                     `"${location.location_name}"`,
        //                     `"${department.department_name}"`,
        //                     `"${employee.name}"`,
        //                     roundToTwo(employee.tmProduction || 0),
        //                     roundToTwo(employee.grossLoss || 0),
        //                     roundToTwo(employee.tmProductionDiamond || 0),
        //                     roundToTwo(employee.grossLossDiamond || 0),
        //                     roundToTwo(employee.tmProductionDiamondPieces || 0),
        //                     roundToTwo(employee.grossLossDiamondPieces || 0),
        //                     roundToTwo(goldRecoveryWeight),  // ✅ Department-Specific Gold Recovery Weight
        //                     roundToTwo(netLossGold),
        //                     roundToTwo(0),
        //                     roundToTwo(0)
        //                 ].join(",") + "\n";
        //             });

        //             // **🔸 Add Department Total Row**
        //             const totalGoldRecoveryWeight = departmentQuantities[deptId] || 0;
        //             const totalNetLossGold = (parseFloat(department.loss || 0) - totalGoldRecoveryWeight).toFixed(2);

        //             csvContent += [
        //                 `""`, // Separator Line
        //                 `"${department.department_name} (TOTAL)"`,
        //                 `""`, // Empty Employee column
        //                 roundToTwo(department.production || 0),
        //                 roundToTwo(department.loss || 0),
        //                 roundToTwo(department.production_diamond || 0),
        //                 roundToTwo(department.loss_diamond || 0),
        //                 roundToTwo(department.production_diamond_pieces || 0),
        //                 roundToTwo(department.loss_diamond_pieces || 0),
        //                 roundToTwo(totalGoldRecoveryWeight),
        //                 roundToTwo(totalNetLossGold),
        //                 roundToTwo(department.totalWeightDiamond || 0),
        //                 roundToTwo(department.netLossDiamond || 0)
        //             ].join(",") + "\n";
        //         });
        //     });

        //     // ✅ Generate and Download CSV File
        //     const timestamp = new Date().toISOString().replace(/[-T:]/g, "_").split(".")[0];
        //     const filename = `Efficiency_Report_${timestamp}.csv`;

        //     const blob = new Blob([csvContent], { type: "text/csv;charset=utf-8;" });
        //     const url = URL.createObjectURL(blob);
        //     const link = document.createElement("a");
        //     link.setAttribute("href", url);
        //     link.setAttribute("download", filename);
        //     document.body.appendChild(link);
        //     link.click();
        //     document.body.removeChild(link);
        // };

        // download data function
        const downloadData = async () => {
            if (!locations.value || locations.value.length === 0) {
                alert("No data available for download.");
                return;
            }

            const formatDate = (date) => {
                const offset = date.getTimezoneOffset();
                date = new Date(date.getTime() - (offset * 60 * 1000));
                return date.toISOString().split('T')[0];
            };

            const startDate = selectedDateRange.value[0] ? formatDate(selectedDateRange.value[0]) : "N/A";
            const endDate = selectedDateRange.value[1] ? formatDate(selectedDateRange.value[1]) : "N/A";

            const workbook = new ExcelJS.Workbook();
            const ws = workbook.addWorksheet(baseTitle);

            // colour palette matching UI
            const C = {
                headerBg:   '1F2937', headerFg: 'FFFFFF',
                blueBg:     'DBEAFE', blueFg:   '1D4ED8',
                redFg:      'EF4444',
                subtotalBg: 'EFF6FF',   // very light blue — whole subtotal row
                grandBg:    'F0FDF4',   // very light green — whole grand total row
                rowBg:      'FFFFFF',   // white for all data rows
                borderColor:'D1D5DB',
            };
            const headerFont   = { bold: true, color: { argb: 'FF' + C.headerFg }, size: 10 };
            const boldFont     = { bold: true, size: 10 };
            const normalFont   = { size: 10 };
            const redFont      = { size: 10, color: { argb: 'FF' + C.redFg } };
            const boldRedFont  = { bold: true, size: 10, color: { argb: 'FF' + C.redFg } };
            const blueBoldFont = { bold: true, size: 10, color: { argb: 'FF' + C.blueFg } };
            const thinBorder   = {
                top:   { style:'thin', color:{ argb:'FF'+C.borderColor } },
                left:  { style:'thin', color:{ argb:'FF'+C.borderColor } },
                bottom:{ style:'thin', color:{ argb:'FF'+C.borderColor } },
                right: { style:'thin', color:{ argb:'FF'+C.borderColor } },
            };
            const addThickTopBorder = (row, colCount) => {
                for (let c = 1; c <= colCount; c++) {
                    const cell = row.getCell(c);
                    cell.border = { ...cell.border, bottom: { style: 'medium', color: { argb: 'FF374151' } } };
                }
            };
            const fillSolid = (hex) => ({ type:'pattern', pattern:'solid', fgColor:{ argb:'FF'+hex } });
            const styleRow  = (row, bg, font, colCount) => {
                for (let c = 1; c <= colCount; c++) {
                    const cell = row.getCell(c);
                    cell.fill      = fillSolid(bg);
                    cell.font      = { ...font };
                    cell.border    = thinBorder;
                    cell.alignment = { vertical:'middle', wrapText:true };
                }
            };
            // Center-align numeric columns (from startCol to colCount)
            const centerNums = (row, startCol, colCount) => {
                for (let c = startCol; c <= colCount; c++) {
                    row.getCell(c).alignment = { vertical:'middle', horizontal:'center', wrapText:true };
                }
            };
            const centerCols = (row, cols) => cols.forEach(c => {
                row.getCell(c).alignment = { vertical:'middle', horizontal:'center', wrapText:true };
            });
            const redCols     = (row, cols) => cols.forEach(c => { row.getCell(c).font = { ...redFont }; });
            const redColsBold = (row, cols) => cols.forEach(c => { row.getCell(c).font = { ...boldRedFont }; });

            // Title
            ws.addRow([baseTitle]);
            ws.getRow(ws.rowCount).font = { bold:true, size:13 };
            ws.addRow([`Date Range: ${startDate} to ${endDate}`]);
            ws.getRow(ws.rowCount).font = { size:10, italic:true };
            ws.addRow([]);

            if (showEmployeesTable.value) {
                // ── EMPLOYEE TABLE ──────────────────────────────────────
                // Col layout (24 cols):
                // 1:Location, 2:Department, 3:Employee, 4:No.of Bags,
                // 5:Item Category, 6:Sub Category, 7:Bag Count, 8:Style Number, 9:Bag Name,
                // 10:Issued Net Weight, 11:Received Quantity,
                // 12:Gross Loss, 13:Gross Loss%, 14:Pure Weight, 15:Pure Loss, 16:Net Loss, 17:Net Loss%,
                // 18:TM Production Diamond, 19:Actual Production Diamond, 20:Loss Qty Diamond, 21:Diamond Loss%,
                // 22:Gold Recovery Weight(gm), 23:Net Loss Gold, 24:Diamond Recovery Weight(ct), 25:Net Loss Diamond
                const EMP_HEADERS = [
                    'Location','Department','Employee','No. of Bags',
                    'Item Category','Sub Category','Bag Count','Style Number','Bag Name',
                    'Issued Net Weight','Received Quantity',
                    'Gross Loss','Gross Loss %',
                    'Pure Weight','Pure Loss','Net Loss','Net Loss %',
                    'TM Production Diamond','Actual Production Diamond','Loss Qty Diamond','Diamond Loss %',
                    'Gold Recovery Weight (gm)','Net Loss Gold','Diamond Recovery Weight (ct)','Net Loss Diamond'
                ];
                const ECOL = EMP_HEADERS.length; // 24
                // loss cols: Gross Loss=12, Gross Loss%=13, Pure Loss=15, Net Loss=16, Net Loss%=17, Diamond Loss%=21
                const EMP_LOSS_COLS = [12,13,15,16,17,21];
                const ehRow = ws.addRow(EMP_HEADERS);
                styleRow(ehRow, C.headerBg, headerFont, ECOL);
                ehRow.height = 30;
                EMP_HEADERS.forEach((h, i) => { ws.getColumn(i+1).width = i===0?14:i<=5?16:18; });

                let gt = { sG:0,iG:0,lG:0,scG:0,bG:0, sD:0,iD:0,lD:0,scD:0,bD:0, pw:0, pl:0 };

                // merge col c from startRow to endRow inclusive; no-op for single row
                const mergeEmpCol = (col, startRow, endRow) => {
                    if (endRow > startRow) ws.mergeCells(startRow, col, endRow, col);
                };

                locations.value.forEach(loc => {
                    const locName = loc.name?.value || "";
                    let locFirstRow = null;

                    (loc.departments || []).forEach(dept => {
                        const deptName = dept.name || "";
                        let deptFirstRow = null;
                        let deptSub = { sG:0,iG:0,lG:0,scG:0,bG:0, sD:0,iD:0,lD:0,scD:0,bD:0, pw:0, pl:0, bags:new Set() };

                        (dept.employees || []).forEach(emp => {
                            const empName  = emp.name || "";
                            const bagCount = emp.bag_count || 0;
                            const cats     = emp.unique_categories_array || [];
                            let sub = { sG:0,iG:0,lG:0,scG:0,bG:0, sD:0,iD:0,lD:0,scD:0,bD:0, pw:0, pl:0 };
                            let empFirstRow = null;

                            cats.forEach((cat) => {
                                const bags = (emp.category_bag_names_map||{})[cat] || [''];
                                const printDesign = (emp.category_print_design_map||{})[cat] || '-';
                                const catBagCount = (emp.category_bag_count_map||{})[cat] || 0;
                                let catFirstRow = null;

                                bags.forEach((bagName) => {
                                    const bagEntry = (emp.categories||[]).find(c => c.category_name === cat && c.bag_name === bagName) || {};
                                    const subCat = (emp.category_bag_sub_category_map||{})[cat]?.[bagName] || '-';
                                    const sG=parseFloat(bagEntry.starting_qty_gold||0), iG=parseFloat(bagEntry.issued_qty_gold||0), lG=parseFloat(bagEntry.loss_qty_gold||0), scG=parseFloat(bagEntry.scrap_qty_gold||0), bG=parseFloat(bagEntry.balance_qty_gold||0);
                                    const sD=parseFloat(bagEntry.starting_qty_diamond||0), iD=parseFloat(bagEntry.issued_qty_diamond||0), lD=parseFloat(bagEntry.loss_qty_diamond||0), scD=parseFloat(bagEntry.scrap_qty_diamond||0), bD=parseFloat(bagEntry.balance_qty_diamond||0);
                                    const recvG = sG+iG-lG-scG-bG;
                                    const issuedNetWt = sG+iG-scG-bG;
                                    const aD=sD+iD-lD-scD-bD;
                                    const gLP = recvG!==0 ? (lG/recvG)*100 : 0;
                                    const dLP = aD!==0 ? (lD/aD)*100 : 0;
                                    const netLossG = recvG!==0 ? lG/recvG : 0;
                                    const netLossPctG = recvG!==0 ? (lG/recvG)*100 : 0;
                                    const purity=(parseFloat(bagEntry.metal_purity_percent||0))/100;
                                    const pureWeight=+roundToTwo(recvG*purity), pureLoss=+roundToTwo(lG*purity);
                                    sub.sG+=sG;sub.iG+=iG;sub.lG+=lG;sub.scG+=scG;sub.bG+=bG;
                                    sub.sD+=sD;sub.iD+=iD;sub.lD+=lD;sub.scD+=scD;sub.bD+=bD;
                                    sub.pw+=recvG*purity; sub.pl+=lG*purity;
                                    deptSub.sG+=sG;deptSub.iG+=iG;deptSub.lG+=lG;deptSub.scG+=scG;deptSub.bG+=bG;
                                    deptSub.sD+=sD;deptSub.iD+=iD;deptSub.lD+=lD;deptSub.scD+=scD;deptSub.bD+=bD;
                                    deptSub.pw+=recvG*purity; deptSub.pl+=lG*purity;
                                    if (bagName) deptSub.bags.add(bagName);
                                    gt.sG+=sG;gt.iG+=iG;gt.lG+=lG;gt.scG+=scG;gt.bG+=bG;
                                    gt.sD+=sD;gt.iD+=iD;gt.lD+=lD;gt.scD+=scD;gt.bD+=bD;
                                    gt.pw+=recvG*purity; gt.pl+=lG*purity;

                                    const empPureLoss = lG * purity;
                                    const empRecoveryWeight = empPureLoss > 0 && globalAvgRecoveryPercentage.value > 0
                                        ? +roundToTwo(empPureLoss * (globalAvgRecoveryPercentage.value / 100)) : 0;
                                    const empRecoveryPct = empPureLoss > 0 && globalAvgRecoveryPercentage.value > 0
                                        ? roundToTwo(globalAvgRecoveryPercentage.value) + '%' : '0%';

                                    // Write full values every row — merging handles visual grouping for filter compatibility
                                    const dr = ws.addRow([
                                        locName, deptName, empName,
                                        bagCount, cat, subCat, catBagCount, printDesign, bagName||'-',
                                        +roundToTwo(issuedNetWt), +roundToTwo(recvG),
                                        +roundToTwo(lG), roundToTwo(gLP)+'%',
                                        pureWeight, pureLoss, +roundToTwo(netLossG), roundToTwo(netLossPctG)+'%',
                                        +roundToTwo(sD), +roundToTwo(aD), +roundToTwo(lD), roundToTwo(dLP)+'%',
                                        empRecoveryWeight, empRecoveryPct, '-', '-'
                                    ]);
                                    styleRow(dr, C.rowBg, normalFont, ECOL);
                                    centerNums(dr, 10, ECOL);
                                    centerCols(dr, [4, 7, 8]);
                                    redCols(dr, EMP_LOSS_COLS);

                                    const rn = dr.number;
                                    if (locFirstRow === null) locFirstRow = rn;
                                    if (deptFirstRow === null) deptFirstRow = rn;
                                    if (empFirstRow === null) empFirstRow = rn;
                                    if (catFirstRow === null) catFirstRow = rn;
                                });

                                // merge Item Category (col5), Bag Count (col7) across all bags in this category
                                mergeEmpCol(5, catFirstRow, ws.rowCount);
                                mergeEmpCol(7, catFirstRow, ws.rowCount);
                            });

                            // subtotal row — col3 (Employee name) is empty so it merges into the emp cell above
                            if (cats.length > 0) {
                                const subRecvG = sub.sG+sub.iG-sub.lG-sub.scG-sub.bG;
                                const subIssuedNetWt = sub.sG+sub.iG-sub.scG-sub.bG;
                                const aD=sub.sD+sub.iD-sub.lD-sub.scD-sub.bD;
                                const subNetLossG = subRecvG!==0 ? sub.lG/subRecvG : 0;
                                const subGrossLossPct = subRecvG!==0 ? roundToTwo((sub.lG/subRecvG)*100)+'%' : '0.00%';
                                const subNetLossPct = subRecvG!==0 ? roundToTwo((sub.lG/subRecvG)*100)+'%' : '0.00%';
                                const subDiamondLossPct = aD!==0 ? roundToTwo((sub.lD/aD)*100)+'%' : '0.00%';
                                const subRecoveryWeight = sub.pl > 0 && globalAvgRecoveryPercentage.value > 0 ? +roundToTwo(sub.pl * (globalAvgRecoveryPercentage.value / 100)) : 0;
                                const subRecoveryPct = globalAvgRecoveryPercentage.value > 0 ? roundToTwo(globalAvgRecoveryPercentage.value)+'%' : '';
                                const sr = ws.addRow([
                                    locName, deptName, '',  // col3 blank — will be merged into emp cell
                                    bagCount,'','','','','',
                                    +roundToTwo(subIssuedNetWt), +roundToTwo(subRecvG),
                                    +roundToTwo(sub.lG), subGrossLossPct,
                                    +roundToTwo(sub.pw), +roundToTwo(sub.pl), +roundToTwo(subNetLossG), subNetLossPct,
                                    +roundToTwo(sub.sD), +roundToTwo(aD), +roundToTwo(sub.lD), subDiamondLossPct,
                                    subRecoveryWeight, subRecoveryPct, '-', '-'
                                ]);
                                styleRow(sr, C.subtotalBg, boldFont, ECOL);
                                centerNums(sr, 10, ECOL);
                                centerCols(sr, [4, 7, 8]);
                                addThickTopBorder(sr, ECOL);
                                redColsBold(sr, [12,13,15,16,17,21]);

                                // merge Employee (col3) and No. of Bags (col4) across all emp rows + subtotal
                                // (must happen before adding the emp total row so the merge doesn't swallow it)
                                mergeEmpCol(3, empFirstRow, ws.rowCount);
                                mergeEmpCol(4, empFirstRow, ws.rowCount);

                                // Employee total row — added AFTER merge so col3 is not swallowed
                                const etr = ws.addRow([
                                    locName, deptName, `${empName} — Total`,
                                    bagCount,'','','','','',
                                    +roundToTwo(subIssuedNetWt), +roundToTwo(subRecvG),
                                    +roundToTwo(sub.lG), subGrossLossPct,
                                    +roundToTwo(sub.pw), +roundToTwo(sub.pl), +roundToTwo(subNetLossG), subNetLossPct,
                                    +roundToTwo(sub.sD), +roundToTwo(aD), +roundToTwo(sub.lD), subDiamondLossPct,
                                    subRecoveryWeight, subRecoveryPct, '-', '-'
                                ]);
                                styleRow(etr, C.subtotalBg, boldFont, ECOL);
                                centerNums(etr, 10, ECOL);
                                centerCols(etr, [4, 7, 8]);
                                redColsBold(etr, [12,13,15,16,17,21]);
                            } else {
                                // no categories — still merge emp cols
                                mergeEmpCol(3, empFirstRow, ws.rowCount);
                                mergeEmpCol(4, empFirstRow, ws.rowCount);
                            }
                        });

                        // ── DEPARTMENT TOTAL ROW ──────────────────────────────
                        if (deptFirstRow !== null) {
                            const dRecvG = deptSub.sG+deptSub.iG-deptSub.lG-deptSub.scG-deptSub.bG;
                            const dIssuedNetWt = deptSub.sG+deptSub.iG-deptSub.scG-deptSub.bG;
                            const dAD = deptSub.sD+deptSub.iD-deptSub.lD-deptSub.scD-deptSub.bD;
                            const dNetLossG = dRecvG!==0 ? deptSub.lG/dRecvG : 0;
                            const dGrossLossPct = dRecvG!==0 ? roundToTwo((deptSub.lG/dRecvG)*100)+'%' : '0.00%';
                            const dNetLossPct = dRecvG!==0 ? roundToTwo((deptSub.lG/dRecvG)*100)+'%' : '0.00%';
                            const dDiamondLossPct = dAD!==0 ? roundToTwo((deptSub.lD/dAD)*100)+'%' : '0.00%';
                            const dRecoveryWeight = deptSub.pl > 0 && globalAvgRecoveryPercentage.value > 0 ? +roundToTwo(deptSub.pl * (globalAvgRecoveryPercentage.value / 100)) : 0;
                            const dRecoveryPct = globalAvgRecoveryPercentage.value > 0 ? roundToTwo(globalAvgRecoveryPercentage.value)+'%' : '';
                            const deptBagCount = deptSub.bags.size;
                            const dtr = ws.addRow([
                                locName, `${deptName} — DEPT TOTAL`, '', deptBagCount,'','','','','',
                                +roundToTwo(dIssuedNetWt), +roundToTwo(dRecvG),
                                +roundToTwo(deptSub.lG), dGrossLossPct,
                                +roundToTwo(deptSub.pw), +roundToTwo(deptSub.pl), +roundToTwo(dNetLossG), dNetLossPct,
                                +roundToTwo(deptSub.sD), +roundToTwo(dAD), +roundToTwo(deptSub.lD), dDiamondLossPct,
                                dRecoveryWeight, dRecoveryPct, '-', '-'
                            ]);
                            styleRow(dtr, C.grandBg, boldFont, ECOL);
                            centerNums(dtr, 10, ECOL);
                            centerCols(dtr, [4, 7, 8]);
                            addThickTopBorder(dtr, ECOL);
                            redColsBold(dtr, [12,13,15,16,17,21]);
                        }

                        // merge Department (col2) across all dept rows (employees + subtotals + dept total)
                        if (deptFirstRow !== null) mergeEmpCol(2, deptFirstRow, ws.rowCount);
                    });

                    // merge Location (col1) across all loc rows
                    if (locFirstRow !== null) mergeEmpCol(1, locFirstRow, ws.rowCount);
                });

                const gtRecvG = gt.sG+gt.iG-gt.lG-gt.scG-gt.bG;
                const gtIssuedNetWt = gt.sG+gt.iG-gt.scG-gt.bG;
                const aD=gt.sD+gt.iD-gt.lD-gt.scD-gt.bD;
                const gtNetLossG = gtRecvG!==0 ? gt.lG/gtRecvG : 0;
                const gtGrossLossPct = gtRecvG!==0 ? roundToTwo((gt.lG/gtRecvG)*100)+'%' : '0.00%';
                const gtNetLossPct = gtRecvG!==0 ? roundToTwo((gt.lG/gtRecvG)*100)+'%' : '0.00%';
                const gtDiamondLossPct = aD!==0 ? roundToTwo((gt.lD/aD)*100)+'%' : '0.00%';
                const allEmpBags = new Set();
                locations.value.forEach(loc => (loc.departments||[]).forEach(dept => (dept.employees||[]).forEach(emp => (emp.unique_bags_array||[]).forEach(b => allEmpBags.add(b)))));
                const grandEmpBagCount = allEmpBags.size || 0;
                const grandEmpRecoveryWeight = gt.pl > 0 && globalAvgRecoveryPercentage.value > 0 ? +roundToTwo(gt.pl * (globalAvgRecoveryPercentage.value / 100)) : 0;
                const grandEmpRecoveryPct = globalAvgRecoveryPercentage.value > 0 ? roundToTwo(globalAvgRecoveryPercentage.value)+'%' : '';
                const egr = ws.addRow([
                    '','','GRAND TOTAL',grandEmpBagCount,'','','','','',
                    +roundToTwo(gtIssuedNetWt), +roundToTwo(gtRecvG),
                    +roundToTwo(gt.lG), gtGrossLossPct,
                    +roundToTwo(gt.pw), +roundToTwo(gt.pl), +roundToTwo(gtNetLossG), gtNetLossPct,
                    +roundToTwo(gt.sD), +roundToTwo(aD), +roundToTwo(gt.lD), gtDiamondLossPct,
                    grandEmpRecoveryWeight, grandEmpRecoveryPct, '-', '-'
                ]);
                styleRow(egr, C.grandBg, boldFont, ECOL);
                centerNums(egr, 10, ECOL);
                centerCols(egr, [4, 7, 8]);
                redColsBold(egr, [12,13,15,16,17,21]);

            } else {
                // ── DEPARTMENT TABLE ────────────────────────────────────
                // Col layout (23 cols):
                // 1:Location, 2:Department, 3:No.of Bags,
                // 4:Item Category, 5:Sub Category, 6:Bag Count, 7:Style Number, 8:Bag Name,
                // 9:Issued Net Weight, 10:Received Quantity,
                // 11:Gross Loss, 12:Gross Loss%, 13:Pure Weight, 14:Pure Loss, 15:Net Loss, 16:Net Loss%,
                // 17:TM Production Diamond, 18:Actual Production Diamond, 19:Loss Qty Diamond, 20:Diamond Loss%,
                // 21:Gold Recovery Weight(gm), 22:Net Loss Gold, 23:Diamond Recovery Weight(ct), 24:Net Loss Diamond
                const DEPT_HEADERS = [
                    'Location','Department','No. of Bags',
                    'Item Category','Sub Category','Bag Count','Style Number','Bag Name',
                    'Issued Net Weight','Received Quantity',
                    'Gross Loss','Gross Loss %',
                    'Pure Weight','Pure Loss','Net Loss','Net Loss %',
                    'TM Production Diamond','Actual Production Diamond','Loss Qty Diamond','Diamond Loss %',
                    'Gold Recovery Weight (gm)','Net Loss Gold','Diamond Recovery Weight (ct)','Net Loss Diamond'
                ];
                const DCOL = DEPT_HEADERS.length; // 23
                // loss cols: Gross Loss=11, Gross Loss%=12, Pure Loss=14, Net Loss=15, Net Loss%=16, Diamond Loss%=20
                const DEPT_LOSS_COLS = [11,12,14,15,16,20];
                const dhRow = ws.addRow(DEPT_HEADERS);
                styleRow(dhRow, C.headerBg, headerFont, DCOL);
                dhRow.height = 30;
                DEPT_HEADERS.forEach((h, i) => { ws.getColumn(i+1).width = i===0?14:i<=4?16:18; });

                let gt = { sG:0,iG:0,lG:0,scG:0,bG:0, sD:0,iD:0,lD:0,scD:0,bD:0, pw:0, pl:0 };

                const mergeDeptCol = (col, startRow, endRow) => {
                    if (endRow > startRow) ws.mergeCells(startRow, col, endRow, col);
                };

                locations.value.forEach(loc => {
                    const locName = loc.name?.value || "";
                    let locFirstRow = null;

                    (loc.departments || []).forEach(dept => {
                        const deptName = dept.name || "";
                        const bagCount = dept.bag_count || 0;
                        const cats     = dept.unique_categories_array || [];
                        let sub = { sG:0,iG:0,lG:0,scG:0,bG:0, sD:0,iD:0,lD:0,scD:0,bD:0, pw:0, pl:0 };
                        let deptFirstRow = null;

                        cats.forEach((cat) => {
                            const bags = (dept.category_bag_names_map||{})[cat] || [''];
                            const printDesign = (dept.category_print_design_map||{})[cat] || '-';
                            const catBagCount = (dept.category_bag_count_map||{})[cat] || 0;
                            let catFirstRow = null;

                            bags.forEach((bagName) => {
                                const bqKey = `${dept.id}_${cat}_${bagName}`;
                                const d = (dept.category_bag_qty_map||{})[bqKey] || {};
                                const subCat = (dept.category_bag_sub_category_map||{})[cat]?.[bagName] || '-';
                                const sG=parseFloat(d.starting_qty_gold||0), iG=parseFloat(d.issued_qty_gold||0), lG=parseFloat(d.loss_qty_gold||0), scG=parseFloat(d.scrap_qty_gold||0), bG=parseFloat(d.balance_qty_gold||0);
                                const sD=parseFloat(d.starting_qty_diamond||0), iD=parseFloat(d.issued_qty_diamond||0), lD=parseFloat(d.loss_qty_diamond||0), scD=parseFloat(d.scrap_qty_diamond||0), bD=parseFloat(d.balance_qty_diamond||0);
                                const recvG = sG+iG-lG-scG-bG; // Received Quantity
                                const issuedNetWt = sG+iG-scG-bG; // Issued Net Weight (without loss)
                                const aD=sD+iD-lD-scD-bD;
                                const gLP = recvG!==0 ? (lG/recvG)*100 : 0;
                                const dLP = aD!==0 ? (lD/aD)*100 : 0;
                                const netLossG = recvG!==0 ? lG/recvG : 0;
                                const netLossPctG = recvG!==0 ? (lG/recvG)*100 : 0;
                                const purity=(parseFloat(d.metal_purity_percent||0))/100;
                                const pureWeight=+roundToTwo(recvG*purity), pureLoss=+roundToTwo(lG*purity);
                                sub.sG+=sG;sub.iG+=iG;sub.lG+=lG;sub.scG+=scG;sub.bG+=bG;
                                sub.sD+=sD;sub.iD+=iD;sub.lD+=lD;sub.scD+=scD;sub.bD+=bD;
                                sub.pw+=recvG*purity; sub.pl+=lG*purity;
                                gt.sG+=sG;gt.iG+=iG;gt.lG+=lG;gt.scG+=scG;gt.bG+=bG;
                                gt.sD+=sD;gt.iD+=iD;gt.lD+=lD;gt.scD+=scD;gt.bD+=bD;
                                gt.pw+=recvG*purity; gt.pl+=lG*purity;

                                const deptPureLoss = lG * purity;
                                const deptRecoveryWeight = deptPureLoss > 0 && globalAvgRecoveryPercentage.value > 0
                                    ? +roundToTwo(deptPureLoss * (globalAvgRecoveryPercentage.value / 100)) : 0;
                                const deptRecoveryPct = deptPureLoss > 0 && globalAvgRecoveryPercentage.value > 0
                                    ? roundToTwo(globalAvgRecoveryPercentage.value) + '%' : '0%';

                                // Write full values every row — merging handles visual grouping for filter compatibility
                                const dr = ws.addRow([
                                    locName, deptName,
                                    bagCount, cat, subCat, catBagCount, printDesign, bagName||'-',
                                    +roundToTwo(issuedNetWt), +roundToTwo(recvG),
                                    +roundToTwo(lG), roundToTwo(gLP)+'%',
                                    pureWeight, pureLoss, +roundToTwo(netLossG), roundToTwo(netLossPctG)+'%',
                                    +roundToTwo(sD), +roundToTwo(aD), +roundToTwo(lD), roundToTwo(dLP)+'%',
                                    deptRecoveryWeight, deptRecoveryPct, '-', '-'
                                ]);
                                styleRow(dr, C.rowBg, normalFont, DCOL);
                                centerNums(dr, 9, DCOL);
                                centerCols(dr, [3, 6, 7]);
                                redCols(dr, DEPT_LOSS_COLS);

                                const rn = dr.number;
                                if (locFirstRow === null) locFirstRow = rn;
                                if (deptFirstRow === null) deptFirstRow = rn;
                                if (catFirstRow === null) catFirstRow = rn;
                            });

                            // merge Item Category (col4), Bag Count (col6) across all bags in this category
                            mergeDeptCol(4, catFirstRow, ws.rowCount);
                            mergeDeptCol(6, catFirstRow, ws.rowCount);
                        });

                        // subtotal row — col2 (Dept name) is empty so it merges into the dept cell above
                        if (cats.length > 0) {
                            const subRecvG = sub.sG+sub.iG-sub.lG-sub.scG-sub.bG;
                            const subIssuedNetWt = sub.sG+sub.iG-sub.scG-sub.bG;
                            const aD=sub.sD+sub.iD-sub.lD-sub.scD-sub.bD;
                            const subNetLossG = subRecvG!==0 ? sub.lG/subRecvG : 0;
                            const subGrossLossPct = subRecvG!==0 ? roundToTwo((sub.lG/subRecvG)*100)+'%' : '0.00%';
                            const subNetLossPct = subRecvG!==0 ? roundToTwo((sub.lG/subRecvG)*100)+'%' : '0.00%';
                            const subDiamondLossPct = aD!==0 ? roundToTwo((sub.lD/aD)*100)+'%' : '0.00%';
                            const subDeptRecoveryWeight = sub.pl > 0 && globalAvgRecoveryPercentage.value > 0 ? +roundToTwo(sub.pl * (globalAvgRecoveryPercentage.value / 100)) : 0;
                            const subRecoveryPct = globalAvgRecoveryPercentage.value > 0 ? roundToTwo(globalAvgRecoveryPercentage.value)+'%' : '';
                            const sr = ws.addRow([
                                locName, '',  // col2 blank — will be merged into dept cell
                                bagCount,'','','','','',
                                +roundToTwo(subIssuedNetWt), +roundToTwo(subRecvG),
                                +roundToTwo(sub.lG), subGrossLossPct,
                                +roundToTwo(sub.pw), +roundToTwo(sub.pl), +roundToTwo(subNetLossG), subNetLossPct,
                                +roundToTwo(sub.sD), +roundToTwo(aD), +roundToTwo(sub.lD), subDiamondLossPct,
                                subDeptRecoveryWeight, subRecoveryPct, '-', '-'
                            ]);
                            styleRow(sr, C.subtotalBg, boldFont, DCOL);
                            centerNums(sr, 9, DCOL);
                            centerCols(sr, [3, 6, 7]);
                            addThickTopBorder(sr, DCOL);
                            redColsBold(sr, [11,12,14,15,16,20]);
                        }

                        // merge Department (col2) and No. of Bags (col3) across all dept rows + subtotal
                        if (deptFirstRow !== null) {
                            mergeDeptCol(2, deptFirstRow, ws.rowCount);
                            mergeDeptCol(3, deptFirstRow, ws.rowCount);
                        }
                    });

                    // merge Location (col1) across all loc rows
                    if (locFirstRow !== null) mergeDeptCol(1, locFirstRow, ws.rowCount);
                });

                const gtRecvG = gt.sG+gt.iG-gt.lG-gt.scG-gt.bG;
                const gtIssuedNetWt = gt.sG+gt.iG-gt.scG-gt.bG;
                const aD=gt.sD+gt.iD-gt.lD-gt.scD-gt.bD;
                const gtNetLossG = gtRecvG!==0 ? gt.lG/gtRecvG : 0;
                const gtGrossLossPct = gtRecvG!==0 ? roundToTwo((gt.lG/gtRecvG)*100)+'%' : '0.00%';
                const gtNetLossPct = gtRecvG!==0 ? roundToTwo((gt.lG/gtRecvG)*100)+'%' : '0.00%';
                const gtDiamondLossPct = aD!==0 ? roundToTwo((gt.lD/aD)*100)+'%' : '0.00%';
                const allDeptBags = new Set();
                locations.value.forEach(loc => (loc.departments||[]).forEach(dept => (dept.unique_bags_array||[]).forEach(b => allDeptBags.add(b))));
                const grandDeptBagCount = allDeptBags.size || locations.value.reduce((s,loc)=>(loc.departments||[]).reduce((s2,dept)=>s2+parseInt(dept.bag_count||0),s),0);
                const grandDeptRecoveryWeight = gt.pl > 0 && globalAvgRecoveryPercentage.value > 0 ? +roundToTwo(gt.pl * (globalAvgRecoveryPercentage.value / 100)) : 0;
                const grandDeptRecoveryPct = globalAvgRecoveryPercentage.value > 0 ? roundToTwo(globalAvgRecoveryPercentage.value)+'%' : '';
                const dgr = ws.addRow([
                    '','GRAND TOTAL',grandDeptBagCount,'','','','','',
                    +roundToTwo(gtIssuedNetWt), +roundToTwo(gtRecvG),
                    +roundToTwo(gt.lG), gtGrossLossPct,
                    +roundToTwo(gt.pw), +roundToTwo(gt.pl), +roundToTwo(gtNetLossG), gtNetLossPct,
                    +roundToTwo(gt.sD), +roundToTwo(aD), +roundToTwo(gt.lD), gtDiamondLossPct,
                    grandDeptRecoveryWeight, grandDeptRecoveryPct, '-', '-'
                ]);
                styleRow(dgr, C.grandBg, boldFont, DCOL);
                centerNums(dgr, 9, DCOL);
                centerCols(dgr, [3, 6, 7]);
                redColsBold(dgr, [11,12,14,15,16,20]);
            }

            // Download as .xlsx
            const timestamp = new Date().toISOString().replace(/[-T:]/g, "_").split(".")[0];
            const buffer = await workbook.xlsx.writeBuffer();
            saveAs(new Blob([buffer], { type:'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' }), `Efficiency_Report_${timestamp}.xlsx`);
        };



        // **Helper Function to Round Values to 2 Decimal Places**
        const roundToTwo = (value) => {
            return value ? parseFloat(value).toFixed(2) : "0.00";
        };

        // Helper function to get total actual production gold for a location
        const getLocationTotalActualProductionGold = (location) => {
            if (!location.departments || location.departments.length === 0) return 0;
            let sum = 0;
            location.departments.forEach(dept => {
                sum += getDepartmentTotalActualProductionGold(dept);
            });
            return sum;
        };

        // Helper function to get total actual production diamond for a location
        const getLocationTotalActualProductionDiamond = (location) => {
            if (!location.departments || location.departments.length === 0) return 0;
            let sum = 0;
            location.departments.forEach(dept => {
                sum += getDepartmentTotalActualProductionDiamond(dept);
            });
            return sum;
        };

        // Helper function to get total unique bags for a location
        const getLocationTotalBagCount = (location) => {
            if (!location.departments || location.departments.length === 0) return 0;
            const uniqueBags = new Set();
            location.departments.forEach(dept => {
                if (dept.unique_bags_array && Array.isArray(dept.unique_bags_array)) {
                    dept.unique_bags_array.forEach(bag => uniqueBags.add(bag));
                }
            });
            // If no unique_bags_array, fallback to summing bag_count
            if (uniqueBags.size === 0) {
                let sum = 0;
                location.departments.forEach(dept => {
                    sum += parseInt(dept.bag_count || 0);
                });
                return sum;
            }
            return uniqueBags.size;
        };

        // Helper function to get total issued quantity gold for a location
        const getLocationTotalIssuedQtyGold = (location) => {
            if (!location.departments || location.departments.length === 0) return 0;
            let sum = 0;
            location.departments.forEach(dept => {
                sum += getDepartmentTotalIssuedQtyGold(dept);
            });
            return sum;
        };

        // Helper function to get total issued quantity diamond for a location
        const getLocationTotalIssuedQtyDiamond = (location) => {
            if (!location.departments || location.departments.length === 0) return 0;
            let sum = 0;
            location.departments.forEach(dept => {
                sum += getDepartmentTotalIssuedQtyDiamond(dept);
            });
            return sum;
        };

        // Helper function to get total loss quantity gold for a location
        const getLocationTotalLossQtyGold = (location) => {
            if (!location.departments || location.departments.length === 0) return 0;
            let sum = 0;
            location.departments.forEach(dept => {
                sum += getDepartmentTotalLossQtyGold(dept);
            });
            return sum;
        };

        // Helper function to get total loss quantity diamond for a location
        const getLocationTotalLossQtyDiamond = (location) => {
            if (!location.departments || location.departments.length === 0) return 0;
            let sum = 0;
            location.departments.forEach(dept => {
                sum += getDepartmentTotalLossQtyDiamond(dept);
            });
            return sum;
        };

        // Helper function to get total issued pieces diamond for a location
        const getLocationTotalIssuedPiecesDiamond = (location) => {
            if (!location.departments || location.departments.length === 0) return 0;
            let sum = 0;
            location.departments.forEach(dept => {
                sum += getDepartmentTotalIssuedPiecesDiamond(dept);
            });
            return sum;
        };

        // Helper function to get total loss pieces diamond for a location
        const getLocationTotalLossPiecesDiamond = (location) => {
            if (!location.departments || location.departments.length === 0) return 0;
            let sum = 0;
            location.departments.forEach(dept => {
                sum += getDepartmentTotalLossPiecesDiamond(dept);
            });
            return sum;
        };

        // Helper function to get total issued quantity gold for a department
        // newIssuedQty = starting + issued - scrap - balance (loss is shown separately in the card)
        const getDepartmentTotalIssuedQtyGold = (dept) => {
            if (!dept.category_qty_map) return 0;
            let sum = 0;
            Object.keys(dept.category_qty_map).forEach(key => {
                // Only sum if the key starts with this department's ID
                if (key.startsWith(`${dept.id}_`)) {
                    const catData = dept.category_qty_map[key];
                    sum += parseFloat(catData.starting_qty_gold || 0)
                         + parseFloat(catData.issued_qty_gold || 0)
                         - parseFloat(catData.scrap_qty_gold || 0)
                         - parseFloat(catData.balance_qty_gold || 0);
                }
            });
            return sum;
        };

        // Helper function to get total issued quantity diamond for a department
        const getDepartmentTotalIssuedQtyDiamond = (dept) => {
            if (!dept.category_qty_map) return 0;
            let sum = 0;
            Object.keys(dept.category_qty_map).forEach(key => {
                // Only sum if the key starts with this department's ID
                if (key.startsWith(`${dept.id}_`)) {
                    const catData = dept.category_qty_map[key];
                    sum += parseFloat(catData.starting_qty_diamond || 0)
                         + parseFloat(catData.issued_qty_diamond || 0)
                         - parseFloat(catData.scrap_qty_diamond || 0)
                         - parseFloat(catData.balance_qty_diamond || 0);
                }
            });
            return sum;
        };

        // Helper function to get total issued quantity gold for an employee
        const getEmployeeTotalIssuedQtyGold = (emp) => {
            if (emp.categories && emp.categories.length > 0) {
                let sum = 0;
                emp.categories.forEach(cat => {
                    sum += parseFloat(cat.starting_qty_gold || 0)
                         + parseFloat(cat.issued_qty_gold || 0)
                         - parseFloat(cat.scrap_qty_gold || 0)
                         - parseFloat(cat.balance_qty_gold || 0);
                });
                return sum;
            }
            return 0;
        };

        // Helper function to get total issued quantity diamond for an employee
        const getEmployeeTotalIssuedQtyDiamond = (emp) => {
            if (emp.categories && emp.categories.length > 0) {
                let sum = 0;
                emp.categories.forEach(cat => {
                    sum += parseFloat(cat.starting_qty_diamond || 0)
                         + parseFloat(cat.issued_qty_diamond || 0)
                         - parseFloat(cat.scrap_qty_diamond || 0)
                         - parseFloat(cat.balance_qty_diamond || 0);
                });
                return sum;
            }
            return 0;
        };
        
        // Helper function to get total loss quantity gold for a department
        const getDepartmentTotalLossQtyGold = (dept) => {
            if (!dept.category_qty_map) return 0;
            let sum = 0;
            Object.keys(dept.category_qty_map).forEach(key => {
                // Only sum if the key starts with this department's ID
                if (key.startsWith(`${dept.id}_`)) {
                    const catData = dept.category_qty_map[key];
                    sum += parseFloat(catData.loss_qty_gold || 0);
                }
            });
            return sum;
        };

        // Helper function to get total loss quantity diamond for a department
        const getDepartmentTotalLossQtyDiamond = (dept) => {
            if (!dept.category_qty_map) return 0;
            let sum = 0;
            Object.keys(dept.category_qty_map).forEach(key => {
                // Only sum if the key starts with this department's ID
                if (key.startsWith(`${dept.id}_`)) {
                    const catData = dept.category_qty_map[key];
                    sum += parseFloat(catData.loss_qty_diamond || 0);
                }
            });
            return sum;
        };

        // Helper function to get total loss quantity gold for an employee
        const getEmployeeTotalLossQtyGold = (emp) => {
            if (!emp.categories || emp.categories.length === 0) return 0;
            let sum = 0;
            emp.categories.forEach(cat => {
                sum += parseFloat(cat.loss_qty_gold || 0);
            });
            return sum;
        };
        
        // Helper function to get total loss quantity diamond for an employee
        const getEmployeeTotalLossQtyDiamond = (emp) => {
            if (emp.categories && emp.categories.length > 0) {
                let sum = 0;
                emp.categories.forEach(cat => {
                    sum += parseFloat(cat.loss_qty_diamond || 0);
                });
                return sum;
            }
            return parseFloat(emp.loss_quantity_diamond || 0);
        };

        // Helper function to get total actual production gold for a department
        const getDepartmentTotalActualProductionGold = (dept) => {
            // For special departments, return Wax Tree actual production gold directly
            if ([CASTING, TREE_CUTTING_CLEANING, GRINDING].includes(parseInt(dept.id))) {
                return parseFloat(dept.wax_tree_actual_production_gold || 0);
            }
            if (!dept.category_qty_map) return 0;
            let sum = 0;
            Object.keys(dept.category_qty_map).forEach(key => {
                if (key.startsWith(`${dept.id}_`)) {
                    const catData = dept.category_qty_map[key];
                    const actualProd = (
                        parseFloat(catData.starting_qty_gold || 0) +
                        parseFloat(catData.issued_qty_gold || 0) -
                        parseFloat(catData.loss_qty_gold || 0) -
                        parseFloat(catData.scrap_qty_gold || 0) -
                        parseFloat(catData.balance_qty_gold || 0)
                    );
                    sum += actualProd;
                }
            });
            return sum;
        };

        // Helper: wax tree loss gold for casting/tree cutting/grinding (card display)
        const getDepartmentWaxTreeLossGold = (dept) => {
            if (![CASTING, TREE_CUTTING_CLEANING, GRINDING].includes(parseInt(dept.id))) return null;
            return parseFloat(dept.wax_tree_loss_gold || 0);
        };

        // Helper function to get total actual production diamond for a department
        const getDepartmentTotalActualProductionDiamond = (dept) => {
            if (!dept.category_qty_map) return 0;
            let sum = 0;
            Object.keys(dept.category_qty_map).forEach(key => {
                if (key.startsWith(`${dept.id}_`)) {
                    const catData = dept.category_qty_map[key];
                    const actualProd = (
                        parseFloat(catData.starting_qty_diamond || 0) +
                        parseFloat(catData.issued_qty_diamond || 0) -
                        parseFloat(catData.loss_qty_diamond || 0) -
                        parseFloat(catData.scrap_qty_diamond || 0) -
                        parseFloat(catData.balance_qty_diamond || 0)
                    );
                    sum += actualProd;
                }
            });
            return sum;
        };

        // Helper function to get total actual production gold for an employee
        const getEmployeeTotalActualProductionGold = (emp) => {
            if (!emp.categories || emp.categories.length === 0) return 0;
            let sum = 0;
            emp.categories.forEach(cat => {
                const actualProd = (
                    parseFloat(cat.starting_qty_gold || 0) +
                    parseFloat(cat.issued_qty_gold || 0) -
                    parseFloat(cat.loss_qty_gold || 0) -
                    parseFloat(cat.scrap_qty_gold || 0) -
                    parseFloat(cat.balance_qty_gold || 0)
                );
                sum += actualProd;
            });
            return sum;
        };

        // Helper function to get total actual production diamond for an employee
        const getEmployeeTotalActualProductionDiamond = (emp) => {
            if (!emp.categories || emp.categories.length === 0) return 0;
            let sum = 0;
            emp.categories.forEach(cat => {
                const actualProd = (
                    parseFloat(cat.starting_qty_diamond || 0) +
                    parseFloat(cat.issued_qty_diamond || 0) -
                    parseFloat(cat.loss_qty_diamond || 0) -
                    parseFloat(cat.scrap_qty_diamond || 0) -
                    parseFloat(cat.balance_qty_diamond || 0)
                );
                sum += actualProd;
            });
            return sum;
        };


        // Helper function to get total issued pieces diamond for a department
        const getDepartmentTotalIssuedPiecesDiamond = (dept) => {
            if (!dept.category_qty_map) return 0;
            let sum = 0;
            Object.keys(dept.category_qty_map).forEach(key => {
                if (key.startsWith(`${dept.id}_`)) {
                    const catData = dept.category_qty_map[key];
                    sum += parseFloat(catData.issued_pieces_diamond || 0);
                }
            });
            return sum;
        };

        // Helper function to get total loss pieces diamond for a department
        const getDepartmentTotalLossPiecesDiamond = (dept) => {
            if (!dept.category_qty_map) return 0;
            let sum = 0;
            Object.keys(dept.category_qty_map).forEach(key => {
                if (key.startsWith(`${dept.id}_`)) {
                    const catData = dept.category_qty_map[key];
                    sum += parseFloat(catData.loss_pieces_diamond || 0);
                }
            });
            return sum;
        };

        // Helper function to get total issued pieces diamond for an employee
        const getEmployeeTotalIssuedPiecesDiamond = (emp) => {
            if (!emp.categories || emp.categories.length === 0) return 0;
            let sum = 0;
            emp.categories.forEach(cat => {
                sum += parseFloat(cat.issued_pieces_diamond || 0);
            });
            return sum;
        };

        // Helper function to get total loss pieces diamond for an employee
        const getEmployeeTotalLossPiecesDiamond = (emp) => {
            if (!emp.categories || emp.categories.length === 0) return 0;
            let sum = 0;
            emp.categories.forEach(cat => {
                sum += parseFloat(cat.loss_pieces_diamond || 0);
            });
            return sum;
        };

        

        // Helper function to get raw numeric category-level starting quantity for Gold (for calculations)
        const getCategoryStartingQtyGoldRaw = (dept, category) => {
            if (!dept.category_qty_map) return 0;
            const key = `${dept.id}_${category}`;
            const data = dept.category_qty_map[key];
            return data ? parseFloat(data.starting_qty_gold || 0) : 0;
        };

        // Helper function to get raw numeric category-level starting quantity for Diamond (for calculations)
        const getCategoryStartingQtyDiamondRaw = (dept, category) => {
            if (!dept.category_qty_map) return 0;
            const key = `${dept.id}_${category}`;
            const data = dept.category_qty_map[key];
            return data ? parseFloat(data.starting_qty_diamond || 0) : 0;
        };

        // Helper function to get raw numeric category-level issued quantity for Gold (for calculations)
        const getCategoryIssuedQtyGoldRaw = (dept, category) => {
            if (!dept.category_qty_map) return 0;
            const key = `${dept.id}_${category}`;
            const data = dept.category_qty_map[key];
            return data ? parseFloat(data.issued_qty_gold || 0) : 0;
        };

        // Helper function to get raw numeric category-level issued quantity for Diamond (for calculations)
        const getCategoryIssuedQtyDiamondRaw = (dept, category) => {
            if (!dept.category_qty_map) return 0;
            const key = `${dept.id}_${category}`;
            const data = dept.category_qty_map[key];
            return data ? parseFloat(data.issued_qty_diamond || 0) : 0;
        };

        // Helper function to get raw numeric category-level loss quantity for Gold (for calculations)
        const getCategoryLossQtyGoldRaw = (dept, category) => {
            if (!dept.category_qty_map) return 0;
            const key = `${dept.id}_${category}`;
            const data = dept.category_qty_map[key];
            return data ? parseFloat(data.loss_qty_gold || 0) : 0;
        };

        // Helper function to get raw numeric category-level loss quantity for Diamond (for calculations)
        const getCategoryLossQtyDiamondRaw = (dept, category) => {
            if (!dept.category_qty_map) return 0;
            const key = `${dept.id}_${category}`;
            const data = dept.category_qty_map[key];
            return data ? parseFloat(data.loss_qty_diamond || 0) : 0;
        };

        // Helper function to get raw numeric category-level scrap quantity for Gold (for calculations)
        const getCategoryScrapQtyGoldRaw = (dept, category) => {
            if (!dept.category_qty_map) return 0;
            const key = `${dept.id}_${category}`;
            const data = dept.category_qty_map[key];
            return data ? parseFloat(data.scrap_qty_gold || 0) : 0;
        };

        // Helper function to get raw numeric category-level scrap quantity for Diamond (for calculations)
        const getCategoryScrapQtyDiamondRaw = (dept, category) => {
            if (!dept.category_qty_map) return 0;
            const key = `${dept.id}_${category}`;
            const data = dept.category_qty_map[key];
            return data ? parseFloat(data.scrap_qty_diamond || 0) : 0;
        };

        // Helper function to get raw numeric category-level balance quantity for Gold (for calculations)
        const getCategoryBalanceQtyGoldRaw = (dept, category) => {
            if (!dept.category_qty_map) return 0;
            const key = `${dept.id}_${category}`;
            const data = dept.category_qty_map[key];
            return data ? parseFloat(data.balance_qty_gold || 0) : 0;
        };

        // Helper function to get raw numeric category-level balance quantity for Diamond (for calculations)
        const getCategoryBalanceQtyDiamondRaw = (dept, category) => {
            if (!dept.category_qty_map) return 0;
            const key = `${dept.id}_${category}`;
            const data = dept.category_qty_map[key];
            return data ? parseFloat(data.balance_qty_diamond || 0) : 0;
        };

        // ── Per-bag helpers (dept) ──────────────────────────────────────────────
        const getBagQtyData = (dept, category, bagName) => {
            if (!dept.category_bag_qty_map) return null;
            const key = `${dept.id}_${category}_${bagName}`;
            return dept.category_bag_qty_map[key] || null;
        };
        const getBagStartingQtyGoldRaw    = (dept, cat, bag) => parseFloat(getBagQtyData(dept, cat, bag)?.starting_qty_gold    || 0);
        const getBagIssuedQtyGoldRaw      = (dept, cat, bag) => parseFloat(getBagQtyData(dept, cat, bag)?.issued_qty_gold      || 0);
        const getBagLossQtyGoldRaw        = (dept, cat, bag) => parseFloat(getBagQtyData(dept, cat, bag)?.loss_qty_gold        || 0);
        const getBagScrapQtyGoldRaw       = (dept, cat, bag) => parseFloat(getBagQtyData(dept, cat, bag)?.scrap_qty_gold       || 0);
        const getBagBalanceQtyGoldRaw     = (dept, cat, bag) => parseFloat(getBagQtyData(dept, cat, bag)?.balance_qty_gold     || 0);
        const getBagStartingQtyDiamondRaw = (dept, cat, bag) => parseFloat(getBagQtyData(dept, cat, bag)?.starting_qty_diamond || 0);
        const getBagIssuedQtyDiamondRaw   = (dept, cat, bag) => parseFloat(getBagQtyData(dept, cat, bag)?.issued_qty_diamond   || 0);
        const getBagLossQtyDiamondRaw     = (dept, cat, bag) => parseFloat(getBagQtyData(dept, cat, bag)?.loss_qty_diamond     || 0);
        const getBagScrapQtyDiamondRaw    = (dept, cat, bag) => parseFloat(getBagQtyData(dept, cat, bag)?.scrap_qty_diamond    || 0);
        const getBagBalanceQtyDiamondRaw  = (dept, cat, bag) => parseFloat(getBagQtyData(dept, cat, bag)?.balance_qty_diamond  || 0);

        // ── Per-bag helpers (employee) ─────────────────────────────────────────
        // Employee categories array has per-bag entries: { category_name, bag_name, ...qty fields }
        const getEmpBagQtyData = (emp, category, bagName) => {
            if (!emp.categories) return null;
            return emp.categories.find(c => c.category_name === category && c.bag_name === bagName) || null;
        };
        const getEmpBagStartingQtyGoldRaw    = (emp, cat, bag) => parseFloat(getEmpBagQtyData(emp, cat, bag)?.starting_qty_gold    || 0);
        const getEmpBagIssuedQtyGoldRaw      = (emp, cat, bag) => parseFloat(getEmpBagQtyData(emp, cat, bag)?.issued_qty_gold      || 0);
        const getEmpBagLossQtyGoldRaw        = (emp, cat, bag) => parseFloat(getEmpBagQtyData(emp, cat, bag)?.loss_qty_gold        || 0);
        const getEmpBagScrapQtyGoldRaw       = (emp, cat, bag) => parseFloat(getEmpBagQtyData(emp, cat, bag)?.scrap_qty_gold       || 0);
        const getEmpBagBalanceQtyGoldRaw     = (emp, cat, bag) => parseFloat(getEmpBagQtyData(emp, cat, bag)?.balance_qty_gold     || 0);
        const getEmpBagStartingQtyDiamondRaw = (emp, cat, bag) => parseFloat(getEmpBagQtyData(emp, cat, bag)?.starting_qty_diamond || 0);
        const getEmpBagIssuedQtyDiamondRaw   = (emp, cat, bag) => parseFloat(getEmpBagQtyData(emp, cat, bag)?.issued_qty_diamond   || 0);
        const getEmpBagLossQtyDiamondRaw     = (emp, cat, bag) => parseFloat(getEmpBagQtyData(emp, cat, bag)?.loss_qty_diamond     || 0);
        const getEmpBagScrapQtyDiamondRaw    = (emp, cat, bag) => parseFloat(getEmpBagQtyData(emp, cat, bag)?.scrap_qty_diamond    || 0);
        const getEmpBagBalanceQtyDiamondRaw  = (emp, cat, bag) => parseFloat(getEmpBagQtyData(emp, cat, bag)?.balance_qty_diamond  || 0);
        // ──────────────────────────────────────────────────────────────────────

        // Pure Weight = Actual Production Gold × (metal_purity_percent / 100)
        // Pure Loss   = Loss Qty Gold          × (metal_purity_percent / 100)
        const getBagPurityFactor = (dept, cat, bag) => parseFloat(getBagQtyData(dept, cat, bag)?.metal_purity_percent || 0) / 100;
        const getBagPureWeight = (dept, cat, bag) => {
            const purity = getBagPurityFactor(dept, cat, bag);
            const actualProdGold = getBagStartingQtyGoldRaw(dept, cat, bag) + getBagIssuedQtyGoldRaw(dept, cat, bag) - getBagLossQtyGoldRaw(dept, cat, bag) - getBagScrapQtyGoldRaw(dept, cat, bag) - getBagBalanceQtyGoldRaw(dept, cat, bag);
            return roundToTwo(actualProdGold * purity);
        };
        const getBagPureLoss = (dept, cat, bag) => {
            const purity = getBagPurityFactor(dept, cat, bag);
            return roundToTwo(getBagLossQtyGoldRaw(dept, cat, bag) * purity);
        };

        const getEmpBagPurityFactor = (emp, cat, bag) => parseFloat(getEmpBagQtyData(emp, cat, bag)?.metal_purity_percent || 0) / 100;
        const getEmpBagPureWeight = (emp, cat, bag) => {
            const purity = getEmpBagPurityFactor(emp, cat, bag);
            const actualProdGold = getEmpBagStartingQtyGoldRaw(emp, cat, bag) + getEmpBagIssuedQtyGoldRaw(emp, cat, bag) - getEmpBagLossQtyGoldRaw(emp, cat, bag) - getEmpBagScrapQtyGoldRaw(emp, cat, bag) - getEmpBagBalanceQtyGoldRaw(emp, cat, bag);
            return roundToTwo(actualProdGold * purity);
        };
        const getEmpBagPureLoss = (emp, cat, bag) => {
            const purity = getEmpBagPurityFactor(emp, cat, bag);
            return roundToTwo(getEmpBagLossQtyGoldRaw(emp, cat, bag) * purity);
        };

        // Compute all totals for a single department (for the per-dept total row in UI table)
        const getDeptTotals = (dept) => {
            let sG=0,iG=0,lG=0,scG=0,bG=0,sD=0,iD=0,lD=0,scD=0,bD=0,pw=0,pl=0;
            // Iterate per-bag using dept.id prefix so we only sum this dept's bags
            (dept.unique_categories_array || []).forEach(cat => {
                (dept.category_bag_names_map?.[cat] || []).forEach(bagName => {
                    const key = `${dept.id}_${cat}_${bagName}`;
                    const d = (dept.category_bag_qty_map || {})[key] || {};
                    const s=parseFloat(d.starting_qty_gold||0), i=parseFloat(d.issued_qty_gold||0),
                          l=parseFloat(d.loss_qty_gold||0), sc=parseFloat(d.scrap_qty_gold||0), b=parseFloat(d.balance_qty_gold||0);
                    const purity=(parseFloat(d.metal_purity_percent||0))/100;
                    const recv=s+i-l-sc-b;
                    sG+=s;iG+=i;lG+=l;scG+=sc;bG+=b;
                    sD+=parseFloat(d.starting_qty_diamond||0);
                    iD+=parseFloat(d.issued_qty_diamond||0);
                    lD+=parseFloat(d.loss_qty_diamond||0);
                    scD+=parseFloat(d.scrap_qty_diamond||0);
                    bD+=parseFloat(d.balance_qty_diamond||0);
                    pw+=recv*purity; pl+=l*purity;
                });
            });
            const recvG=sG+iG-lG-scG-bG;
            const issuedNetWt=sG+iG-scG-bG;
            const aD=sD+iD-lD-scD-bD;
            return {
                issuedNetWt: roundToTwo(issuedNetWt),
                recvG: roundToTwo(recvG),
                lG: roundToTwo(lG),
                grossLossPct: recvG>0 ? roundToTwo((lG/recvG)*100)+'%' : '0.00%',
                pw: roundToTwo(pw),
                pl: roundToTwo(pl),
                netLoss: recvG>0 ? roundToTwo(lG/recvG) : '0.00',
                netLossPct: recvG>0 ? roundToTwo((lG/recvG)*100)+'%' : '0.00%',
                sD: roundToTwo(sD),
                aD: roundToTwo(aD),
                lD: roundToTwo(lD),
                diamondLossPct: aD>0 ? roundToTwo((lD/aD)*100)+'%' : '0.00%',
                recoveryWt: pl>0 && globalAvgRecoveryPercentage.value>0 ? roundToTwo(pl*(globalAvgRecoveryPercentage.value/100)) : '0.00',
                recoveryPct: globalAvgRecoveryPercentage.value>0 ? roundToTwo(globalAvgRecoveryPercentage.value)+'%' : '0%'
            };
        };

        // Compute all totals for a single employee (for the per-employee total row)
        const getEmpTotals = (emp) => {
            let sG=0,iG=0,lG=0,scG=0,bG=0,sD=0,iD=0,lD=0,scD=0,bD=0,pw=0,pl=0;
            (emp.categories || []).forEach(cat => {
                const s=parseFloat(cat.starting_qty_gold||0), i=parseFloat(cat.issued_qty_gold||0),
                      l=parseFloat(cat.loss_qty_gold||0), sc=parseFloat(cat.scrap_qty_gold||0), b=parseFloat(cat.balance_qty_gold||0);
                const purity=(parseFloat(cat.metal_purity_percent||0))/100;
                const recv=s+i-l-sc-b;
                sG+=s;iG+=i;lG+=l;scG+=sc;bG+=b;
                sD+=parseFloat(cat.starting_qty_diamond||0);
                iD+=parseFloat(cat.issued_qty_diamond||0);
                lD+=parseFloat(cat.loss_qty_diamond||0);
                scD+=parseFloat(cat.scrap_qty_diamond||0);
                bD+=parseFloat(cat.balance_qty_diamond||0);
                pw+=recv*purity; pl+=l*purity;
            });
            const recvG=sG+iG-lG-scG-bG;
            const issuedNetWt=sG+iG-scG-bG;
            const aD=sD+iD-lD-scD-bD;
            return {
                issuedNetWt: roundToTwo(issuedNetWt),
                recvG: roundToTwo(recvG),
                lG: roundToTwo(lG),
                grossLossPct: recvG>0 ? roundToTwo((lG/recvG)*100)+'%' : '0.00%',
                pw: roundToTwo(pw),
                pl: roundToTwo(pl),
                netLoss: recvG>0 ? roundToTwo(lG/recvG) : '0.00',
                netLossPct: recvG>0 ? roundToTwo((lG/recvG)*100)+'%' : '0.00%',
                sD: roundToTwo(sD),
                aD: roundToTwo(aD),
                lD: roundToTwo(lD),
                diamondLossPct: aD>0 ? roundToTwo((lD/aD)*100)+'%' : '0.00%',
                recoveryWt: pl>0 && globalAvgRecoveryPercentage.value>0 ? roundToTwo(pl*(globalAvgRecoveryPercentage.value/100)) : '0.00',
                recoveryPct: globalAvgRecoveryPercentage.value>0 ? roundToTwo(globalAvgRecoveryPercentage.value)+'%' : '0%'
            };
        };

        // Build NetSuite record links — always use app.netsuite.com for record navigation.
        // extforms.netsuite.com (used in dev) is for unauthenticated external forms only;
        // record links must go to app.netsuite.com where the user's session cookie is valid.
        const _appendRaw = ENV_VAR.NS_API.API.BAG_MANAGEMENT_APP_ENDPOINT.APPEND || '';
        const _compidMatch = _appendRaw.match(/compid=([^&]+)/);
        const _compidValue = _compidMatch ? _compidMatch[1] : '';
        // Derive app.netsuite.com base from compid (strip _SB1 suffix for sandbox → use -sb1 subdomain)
        const _isSandbox = _compidValue.includes('_SB');
        const _sbNum = _isSandbox ? _compidValue.split('_SB')[1] : '';
        const _accountId = _compidValue.split('_SB')[0];
        const _appBase = _isSandbox
            ? `https://${_accountId}-sb${_sbNum}.app.netsuite.com`
            : `https://${_accountId}.app.netsuite.com`;
        const _compid = _compidValue ? `&compid=${_compidValue}` : '';

        const getBagNsUrl = (dept, category, bagName) => {
            const bagId = dept.category_bag_ids_map?.[category]?.[bagName];
            if (!bagId) return null;
            return `${_appBase}/app/common/custom/custrecordentry.nl?rectype=1026&id=${bagId}${_compid}`;
        };
        const getEmpBagNsUrl = (emp, category, bagName) => {
            const bagId = emp.category_bag_ids_map?.[category]?.[bagName];
            if (!bagId) return null;
            return `${_appBase}/app/common/custom/custrecordentry.nl?rectype=1026&id=${bagId}${_compid}`;
        };

        // Department link (customrecord_jj_manufacturing_dept)
        const getDeptNsUrl = (dept) => {
            if (!dept?.id) return null;
            return `${_appBase}/app/common/custom/custrecordentry.nl?rectype=1011&id=${dept.id}${_compid}`;
        };

        // Employee link (standard employee record)
        const getEmpNsUrl = (emp) => {
            if (!emp?.id) return null;
            return `${_appBase}/app/common/entity/employee.nl?id=${emp.id}${_compid}`;
        };

        // Style Number (assembly item) link — per bag
        const getStyleNsUrlPerBag = (entity, category, bagName) => {
            const itemId = entity?.category_bag_print_design_id_map?.[category]?.[bagName];
            if (!itemId) return null;
            return `${_appBase}/app/common/item/item.nl?id=${itemId}${_compid}`;
        };

        // Category link (CUSTOMRECORD_JJ_CATEGORY) — per bag
        // Uses category_bag_category_id_map if available (requires backend deployment),
        // otherwise falls back to a NS search URL by category name
        const getCategoryNsUrl = (entity, category, bagName) => {
            const catId = entity?.category_bag_category_id_map?.[category]?.[bagName];
            if (catId) {
                return `${_appBase}/app/common/custom/custrecordentry.nl?rectype=1007&id=${catId}`;
            }
            // Fallback: search for the category record by name
            if (category) {
                return `${_appBase}/app/common/custom/custrecordlist.nl?rectype=customrecord_jj_category&searchtype=CUSTOMRECORD_JJ_CATEGORY&query=${encodeURIComponent(category)}${_compid}`;
            }
            return null;
        };

        // Sub Category link — per bag
        // Uses category_bag_sub_category_id_map if available (requires backend deployment),
        // otherwise falls back to a NS search URL by sub-category name
        const getSubCategoryNsUrl = (entity, category, bagName) => {
            const subCatId = entity?.category_bag_sub_category_id_map?.[category]?.[bagName];
            if (subCatId) {
                return `${_appBase}/app/common/custom/custrecordentry.nl?rectype=1020&id=${subCatId}`;
            }
            // Fallback: search for the sub-category record by name
            const subCatName = entity?.category_bag_sub_category_map?.[category]?.[bagName];
            if (subCatName && subCatName !== '-') {
                return `${_appBase}/app/common/custom/custrecordlist.nl?rectype=customrecord_jj_sub_category&searchtype=CUSTOMRECORD_JJ_SUB_CATEGORY&query=${encodeURIComponent(subCatName)}${_compid}`;
            }
            return null;
        };
        // Helper function to get category-level starting quantity for Gold
        const getCategoryStartingQtyGold = (dept, category) => {
            if (!dept.category_qty_map) return '-';
            const key = `${dept.id}_${category}`;
            const data = dept.category_qty_map[key];
            return data ? roundToTwo(data.starting_qty_gold) : '-';
        };

        // Helper function to get category-level starting quantity for Diamond
        const getCategoryStartingQtyDiamond = (dept, category) => {
            if (!dept.category_qty_map) return '-';
            const key = `${dept.id}_${category}`;
            const data = dept.category_qty_map[key];
            return data ? roundToTwo(data.starting_qty_diamond) : '-';
        };

        // Generic wrapper for starting quantity (defaults to Gold)
        const getCategoryStartingQty = (dept, category) => {
            return getCategoryStartingQtyGold(dept, category);
        };

        // Helper function to get category-level issued quantity for Gold
        const getCategoryIssuedQtyGold = (dept, category) => {
            if (!dept.category_qty_map) return '-';
            const key = `${dept.id}_${category}`;
            const data = dept.category_qty_map[key];
            return data ? roundToTwo(data.issued_qty_gold) : '-';
        };

        // Helper function to get category-level issued quantity for Diamond
        const getCategoryIssuedQtyDiamond = (dept, category) => {
            if (!dept.category_qty_map) return '-';
            const key = `${dept.id}_${category}`;
            const data = dept.category_qty_map[key];
            return data ? roundToTwo(data.issued_qty_diamond) : '-';
        };

        // Generic wrapper for issued quantity (defaults to Gold)
        const getCategoryIssuedQty = (dept, category) => {
            return getCategoryIssuedQtyGold(dept, category);
        };

        // Helper function to get category-level loss quantity for Gold
        const getCategoryLossQtyGold = (dept, category) => {
            if (!dept.category_qty_map) return '-';
            const key = `${dept.id}_${category}`;
            const data = dept.category_qty_map[key];
            return data ? roundToTwo(data.loss_qty_gold) : '-';
        };

        // Helper function to get category-level loss quantity for Diamond
        const getCategoryLossQtyDiamond = (dept, category) => {
            if (!dept.category_qty_map) return '-';
            const key = `${dept.id}_${category}`;
            const data = dept.category_qty_map[key];
            return data ? roundToTwo(data.loss_qty_diamond) : '-';
        };

        // Generic wrapper for loss quantity (defaults to Gold)
        const getCategoryLossQty = (dept, category) => {
            return getCategoryLossQtyGold(dept, category);
        };

        // Helper function to get category-level scrap quantity for Gold
        const getCategoryScrapQtyGold = (dept, category) => {
            if (!dept.category_qty_map) return '-';
            const key = `${dept.id}_${category}`;
            const data = dept.category_qty_map[key];
            return data ? roundToTwo(data.scrap_qty_gold) : '-';
        };

        // Helper function to get category-level scrap quantity for Diamond
        const getCategoryScrapQtyDiamond = (dept, category) => {
            if (!dept.category_qty_map) return '-';
            const key = `${dept.id}_${category}`;
            const data = dept.category_qty_map[key];
            return data ? roundToTwo(data.scrap_qty_diamond) : '-';
        };

        // Generic wrapper for scrap quantity (defaults to Gold)
        const getCategoryScrapQty = (dept, category) => {
            return getCategoryScrapQtyGold(dept, category);
        };

        // Helper function to get category-level balance quantity for Gold
        const getCategoryBalanceQtyGold = (dept, category) => {
            if (!dept.category_qty_map) return '-';
            const key = `${dept.id}_${category}`;
            const data = dept.category_qty_map[key];
            return data ? roundToTwo(data.balance_qty_gold) : '-';
        };

        // Helper function to get category-level balance quantity for Diamond
        const getCategoryBalanceQtyDiamond = (dept, category) => {
            if (!dept.category_qty_map) return '-';
            const key = `${dept.id}_${category}`;
            const data = dept.category_qty_map[key];
            return data ? roundToTwo(data.balance_qty_diamond) : '-';
        };

        // Generic wrapper for balance quantity (defaults to Gold)
        const getCategoryBalanceQty = (dept, category) => {
            return getCategoryBalanceQtyGold(dept, category);
        };

        // ===== EMPLOYEE CATEGORY-LEVEL GETTER FUNCTIONS =====

        // Helper function to get raw numeric employee category-level starting quantity for Gold (for calculations)
        const getEmployeeCategoryStartingQtyGoldRaw = (emp, category) => {
            if (!emp.category_qty_map) return 0;
            const key = `${emp.id}_${category}`;
            const data = emp.category_qty_map[key];
            return data ? parseFloat(data.starting_qty_gold || 0) : 0;
        };

        // Helper function to get raw numeric employee category-level starting quantity for Diamond (for calculations)
        const getEmployeeCategoryStartingQtyDiamondRaw = (emp, category) => {
            if (!emp.category_qty_map) return 0;
            const key = `${emp.id}_${category}`;
            const data = emp.category_qty_map[key];
            return data ? parseFloat(data.starting_qty_diamond || 0) : 0;
        };

        // Helper function to get raw numeric employee category-level issued quantity for Gold (for calculations)
        const getEmployeeCategoryIssuedQtyGoldRaw = (emp, category) => {
            if (!emp.category_qty_map) return 0;
            const key = `${emp.id}_${category}`;
            const data = emp.category_qty_map[key];
            return data ? parseFloat(data.issued_qty_gold || 0) : 0;
        };

        // Helper function to get raw numeric employee category-level issued quantity for Diamond (for calculations)
        const getEmployeeCategoryIssuedQtyDiamondRaw = (emp, category) => {
            if (!emp.category_qty_map) return 0;
            const key = `${emp.id}_${category}`;
            const data = emp.category_qty_map[key];
            return data ? parseFloat(data.issued_qty_diamond || 0) : 0;
        };

        // Helper function to get raw numeric employee category-level loss quantity for Gold (for calculations)
        const getEmployeeCategoryLossQtyGoldRaw = (emp, category) => {
            if (!emp.category_qty_map) return 0;
            const key = `${emp.id}_${category}`;
            const data = emp.category_qty_map[key];
            return data ? parseFloat(data.loss_qty_gold || 0) : 0;
        };

        // Helper function to get raw numeric employee category-level loss quantity for Diamond (for calculations)
        const getEmployeeCategoryLossQtyDiamondRaw = (emp, category) => {
            if (!emp.category_qty_map) return 0;
            const key = `${emp.id}_${category}`;
            const data = emp.category_qty_map[key];
            return data ? parseFloat(data.loss_qty_diamond || 0) : 0;
        };

        // Helper function to get raw numeric employee category-level scrap quantity for Gold (for calculations)
        const getEmployeeCategoryScrapQtyGoldRaw = (emp, category) => {
            if (!emp.category_qty_map) return 0;
            const key = `${emp.id}_${category}`;
            const data = emp.category_qty_map[key];
            return data ? parseFloat(data.scrap_qty_gold || 0) : 0;
        };

        // Helper function to get raw numeric employee category-level scrap quantity for Diamond (for calculations)
        const getEmployeeCategoryScrapQtyDiamondRaw = (emp, category) => {
            if (!emp.category_qty_map) return 0;
            const key = `${emp.id}_${category}`;
            const data = emp.category_qty_map[key];
            return data ? parseFloat(data.scrap_qty_diamond || 0) : 0;
        };

        // Helper function to get raw numeric employee category-level balance quantity for Gold (for calculations)
        const getEmployeeCategoryBalanceQtyGoldRaw = (emp, category) => {
            if (!emp.category_qty_map) return 0;
            const key = `${emp.id}_${category}`;
            const data = emp.category_qty_map[key];
            return data ? parseFloat(data.balance_qty_gold || 0) : 0;
        };

        // Helper function to get raw numeric employee category-level balance quantity for Diamond (for calculations)
        const getEmployeeCategoryBalanceQtyDiamondRaw = (emp, category) => {
            if (!emp.category_qty_map) return 0;
            const key = `${emp.id}_${category}`;
            const data = emp.category_qty_map[key];
            return data ? parseFloat(data.balance_qty_diamond || 0) : 0;
        };

        // Helper function to get employee category-level starting quantity for Gold
        const getEmployeeCategoryStartingQtyGold = (emp, category) => {
            if (!emp.category_qty_map) return '-';
            const key = `${emp.id}_${category}`;
            const data = emp.category_qty_map[key];
            return data ? roundToTwo(data.starting_qty_gold) : '-';
        };

        // Helper function to get employee category-level starting quantity for Diamond
        const getEmployeeCategoryStartingQtyDiamond = (emp, category) => {
            if (!emp.category_qty_map) return '-';
            const key = `${emp.id}_${category}`;
            const data = emp.category_qty_map[key];
            return data ? roundToTwo(data.starting_qty_diamond) : '-';
        };

        // Generic wrapper for employee category starting quantity (defaults to Gold)
        const getEmployeeCategoryStartingQty = (emp, category) => {
            return getEmployeeCategoryStartingQtyGold(emp, category);
        };

        // Helper function to get employee category-level issued quantity for Gold
        const getEmployeeCategoryIssuedQtyGold = (emp, category) => {
            if (!emp.category_qty_map) return '-';
            const key = `${emp.id}_${category}`;
            const data = emp.category_qty_map[key];
            return data ? roundToTwo(data.issued_qty_gold) : '-';
        };

        // Helper function to get employee category-level issued quantity for Diamond
        const getEmployeeCategoryIssuedQtyDiamond = (emp, category) => {
            if (!emp.category_qty_map) return '-';
            const key = `${emp.id}_${category}`;
            const data = emp.category_qty_map[key];
            return data ? roundToTwo(data.issued_qty_diamond) : '-';
        };

        // Generic wrapper for employee category issued quantity (defaults to Gold)
        const getEmployeeCategoryIssuedQty = (emp, category) => {
            return getEmployeeCategoryIssuedQtyGold(emp, category);
        };

        // Helper function to get employee category-level loss quantity for Gold
        const getEmployeeCategoryLossQtyGold = (emp, category) => {
            if (!emp.category_qty_map) return '-';
            const key = `${emp.id}_${category}`;
            const data = emp.category_qty_map[key];
            return data ? roundToTwo(data.loss_qty_gold) : '-';
        };

        // Helper function to get employee category-level loss quantity for Diamond
        const getEmployeeCategoryLossQtyDiamond = (emp, category) => {
            if (!emp.category_qty_map) return '-';
            const key = `${emp.id}_${category}`;
            const data = emp.category_qty_map[key];
            return data ? roundToTwo(data.loss_qty_diamond) : '-';
        };

        // Generic wrapper for employee category loss quantity (defaults to Gold)
        const getEmployeeCategoryLossQty = (emp, category) => {
            return getEmployeeCategoryLossQtyGold(emp, category);
        };

        // Helper function to get employee category-level scrap quantity for Gold
        const getEmployeeCategoryScrapQtyGold = (emp, category) => {
            if (!emp.category_qty_map) return '-';
            const key = `${emp.id}_${category}`;
            const data = emp.category_qty_map[key];
            return data ? roundToTwo(data.scrap_qty_gold) : '-';
        };

        // Helper function to get employee category-level scrap quantity for Diamond
        const getEmployeeCategoryScrapQtyDiamond = (emp, category) => {
            if (!emp.category_qty_map) return '-';
            const key = `${emp.id}_${category}`;
            const data = emp.category_qty_map[key];
            return data ? roundToTwo(data.scrap_qty_diamond) : '-';
        };

        // Generic wrapper for employee category scrap quantity (defaults to Gold)
        const getEmployeeCategoryScrapQty = (emp, category) => {
            return getEmployeeCategoryScrapQtyGold(emp, category);
        };

        // Helper function to get employee category-level balance quantity for Gold
        const getEmployeeCategoryBalanceQtyGold = (emp, category) => {
            if (!emp.category_qty_map) return '-';
            const key = `${emp.id}_${category}`;
            const data = emp.category_qty_map[key];
            return data ? roundToTwo(data.balance_qty_gold) : '-';
        };

        // Helper function to get employee category-level balance quantity for Diamond
        const getEmployeeCategoryBalanceQtyDiamond = (emp, category) => {
            if (!emp.category_qty_map) return '-';
            const key = `${emp.id}_${category}`;
            const data = emp.category_qty_map[key];
            return data ? roundToTwo(data.balance_qty_diamond) : '-';
        };

        // Generic wrapper for employee category balance quantity (defaults to Gold)
        const getEmployeeCategoryBalanceQty = (emp, category) => {
            return getEmployeeCategoryBalanceQtyGold(emp, category);
        };

        // Helper function to sum total starting quantity (gold + diamond) for an employee across all categories
        const getEmployeeTotalStartingQtyAllCategories = (emp) => {
            if (!emp.category_qty_map || !emp.unique_categories_array) return 0;
            let sum = 0;
            emp.unique_categories_array.forEach(category => {
                const key = `${emp.id}_${category}`;
                const data = emp.category_qty_map[key];
                if (data) {
                    sum += parseFloat(data.starting_qty_gold || 0) + parseFloat(data.starting_qty_diamond || 0);
                }
            });
            return sum;
        };

        // Helper function to sum total issued quantity (gold + diamond) for an employee across all categories
        const getEmployeeTotalIssuedQtyAllCategories = (emp) => {
            if (!emp.category_qty_map || !emp.unique_categories_array) return 0;
            let sum = 0;
            emp.unique_categories_array.forEach(category => {
                const key = `${emp.id}_${category}`;
                const data = emp.category_qty_map[key];
                if (data) {
                    sum += parseFloat(data.issued_qty_gold || 0) + parseFloat(data.issued_qty_diamond || 0);
                }
            });
            return sum;
        };

        // Helper function to sum total loss quantity (gold + diamond) for an employee across all categories
        const getEmployeeTotalLossQtyAllCategories = (emp) => {
            if (!emp.category_qty_map || !emp.unique_categories_array) return 0;
            let sum = 0;
            emp.unique_categories_array.forEach(category => {
                const key = `${emp.id}_${category}`;
                const data = emp.category_qty_map[key];
                if (data) {
                    sum += parseFloat(data.loss_qty_gold || 0) + parseFloat(data.loss_qty_diamond || 0);
                }
            });
            return sum;
        };

        // Function to fetch efficiency analysis data for a specific location
        // const fetchEfficiencyAnalysisData = async (locationId = null) => {
        //     try {
        //         loading.value = true;
        //         showNoDataPopup.value = false; // Reset popup visibility

        //         // Convert date range to properly formatted strings (fixing time zone issue)
        //         if (!selectedDateRange.value || selectedDateRange.value.length < 2) return;

        //         const formatDate = (date) => {
        //             const offset = date.getTimezoneOffset();
        //             date = new Date(date.getTime() - (offset * 60 * 1000)); // Adjust for timezone offset
        //             return date.toISOString().split('T')[0]; // Format as YYYY-MM-DD
        //         };

        //         const formattedStartDate = formatDate(selectedDateRange.value[0]);
        //         const formattedEndDate = formatDate(selectedDateRange.value[1]);

        //         console.log(`Fetching data for ${locationId ? `Location: ${locationId}` : 'All Locations'}, Start Date: ${formattedStartDate}, End Date: ${formattedEndDate}`);

        //         // Call API with the selected date range
        //         await fetchListEfficiencyAnalysis(locationId, formattedStartDate, formattedEndDate);

        //         if (listEfficiencyData.value && Object.keys(listEfficiencyData.value).length > 0) {
        //             console.log("Efficiency Data:", listEfficiencyData.value);

        //             if (!locationId) {
        //                 // If no location is selected, update all locations
        //                 locations.value = Object.entries(listEfficiencyData.value).map(([locId, locData]) => ({
        //                     internalid: { value: locId },
        //                     name: { value: locData.location_name },
        //                     tmproduction_gold: locData.tmproduction_gold || 0,
        //                     loss_gold: locData.loss_gold || 0,
        //                     tmproduction_diamond: locData.tmproduction_diamond || 0,
        //                     loss_diamond: locData.loss_diamond || 0,
        //                     tmproduction_diamond_pieces: locData.tmproduction_diamond_pieces || 0,
        //                     loss_diamond_pieces: locData.loss_diamond_pieces || 0,
        //                     departments: Object.entries(locData.departments || {}).map(([deptId, dept]) => ({
        //                         name: dept.department_name,
        //                         production: dept.production,
        //                         loss: dept.loss,
        //                         date: dept.date,
        //                         production_diamond: dept.production_diamond,
        //                         loss_diamond: dept.loss_diamond,
        //                         loss_diamond_pieces: dept.loss_diamond_pieces,
        //                         production_diamond_pieces: dept.production_diamond_pieces,

        //                         employees: Object.entries(dept.employees || {}).map(([empId, emp]) => ({
        //                             id: empId,
        //                             name: emp.name !== "null null" ? emp.name : "Unknown Employee",
        //                             tmProduction: emp.tmProduction || 0,
        //                             tmProductionDiamond: emp.tmProductionDiamond || 0,
        //                             grossLossDiamond: emp.grossLossDiamond || 0,
        //                             grossLoss: emp.grossLoss || 0,
        //                             loss: emp.loss || 0,
        //                             netLoss: emp.netLoss || 0,
        //                             recovery: emp.recovery || 0,
        //                             date: emp.date || "N/A"
        //                         }))
        //                     }))
        //                 }));
        //             } else {
        //                 // Update specific location
        //                 const efficiencyData = listEfficiencyData.value[locationId];
        //                 const updatedLocation = locations.value.find(loc => loc.internalid.value === locationId);

        //                 if (updatedLocation && efficiencyData) {
        //                     updatedLocation.tmproduction_gold = efficiencyData.tmproduction_gold || 0;
        //                     updatedLocation.loss_gold = efficiencyData.loss_gold || 0;
        //                     updatedLocation.tmproduction_diamond = efficiencyData.tmproduction_diamond || 0;
        //                     updatedLocation.loss_diamond = efficiencyData.loss_diamond || 0;
        //                     updatedLocation.tmproduction_diamond_pieces = efficiencyData.tmproduction_diamond_pieces || 0;
        //                     updatedLocation.loss_diamond_pieces = efficiencyData.loss_diamond_pieces || 0;
        //                     updatedLocation.departments = Object.entries(efficiencyData.departments || {}).map(([deptId, dept]) => ({
        //                         name: dept.department_name,
        //                         production: dept.production,
        //                         loss: dept.loss,
        //                         date: dept.date,
        //                         production_diamond: dept.production_diamond,
        //                         loss_diamond: dept.loss_diamond,
        //                         loss_diamond_pieces: dept.loss_diamond_pieces,
        //                         production_diamond_pieces: dept.production_diamond_pieces,
        //                         employees: Object.entries(dept.employees || {}).map(([empId, emp]) => ({
        //                             id: empId,
        //                             name: emp.name !== "null null" ? emp.name : "Unknown Employee",
        //                             tmProduction: emp.tmProduction || 0,
        //                             grossLoss: emp.grossLoss || 0,
        //                             tmProductionDiamond: emp.tmProductionDiamond || 0,
        //                             grossLossDiamond: emp.grossLossDiamond || 0,
        //                             tmProductionDiamondPieces: emp.tmProductionDiamondPieces || 0,
        //                             grossLossDiamondPieces: emp.grossLossDiamondPieces || 0,
        //                             loss: emp.loss || 0,
        //                             netLoss: emp.netLoss || 0,
        //                             recovery: emp.recovery || 0,
        //                             date: emp.date || "N/A"
        //                         }))
        //                     }));
        //                 }
        //             }
        //         } else {
        //             // Show Popup if No Data Found
        //             showNoDataPopup.value = true;
        //             setTimeout(() => {
        //                 showNoDataPopup.value = false; // Auto-hide popup after 3 sec
        //                 selectedDateRange.value = getDefaultDateRange(); // ✅ Reset date filter value
        //             }, 3000);
        //         }
        //     } catch (err) {
        //         console.error("Error fetching efficiency analysis data:", err);
        //         showNoDataPopup.value = true; // Show popup in case of an error
        //         setTimeout(() => {
        //             showNoDataPopup.value = false;
        //             selectedDateRange.value = getDefaultDateRange(); // ✅ Reset date filter value
        //         }, 3000);
        //     } finally {
        //         loading.value = false;
        //     }
        // };
        //latest
        // const fetchEfficiencyAnalysisData = async (locationId = null) => {
        //     try {
        //         loading.value = true;
        //         showNoDataPopup.value = false;

        //         if (!selectedDateRange.value || selectedDateRange.value.length < 2) return;

        //         const formatDate = (date) => {
        //             const offset = date.getTimezoneOffset();
        //             date = new Date(date.getTime() - (offset * 60 * 1000));
        //             return date.toISOString().split('T')[0];
        //         };

        //         const formattedStartDate = formatDate(selectedDateRange.value[0]);
        //         const formattedEndDate = formatDate(selectedDateRange.value[1]);

        //         console.log(`Fetching data for ${locationId ? `Location: ${locationId}` : 'All Locations'}, Start Date: ${formattedStartDate}, End Date: ${formattedEndDate}`);

        //         // Fetch Efficiency Analysis Data
        //         await fetchListEfficiencyAnalysis(locationId, formattedStartDate, formattedEndDate);

        //         // Fetch Inventory Adjustments
        //         await fetchInventoryAdjustmentsDetails(formattedStartDate, formattedEndDate);

        //         if (listEfficiencyData.value && Object.keys(listEfficiencyData.value).length > 0) {
        //             console.log("Efficiency Data:", listEfficiencyData.value);

        //             let inventoryAdjustments = listInventoryAdjustments.value || [];
        //             console.log("Inventory Adjustments:", inventoryAdjustments);

        //             let departmentQuantities = {};
        //             let employeeQuantities = {}; // ✅ Stores department-wise employee data

        //             // **Process inventory adjustments to aggregate quantities by department & employee**
        //             inventoryAdjustments.forEach(item => {
        //                 const departmentId = item.custbody_jj_recovery_department;
        //                 const employeeId = item.custbody_jj_recovered_employee;
        //                 const key = `${departmentId}_${employeeId}`; // ✅ Unique Key for Department-Employee

        //                 const quantity = parseFloat(item.quantity || 0);

        //                 // Aggregate by department
        //                 if (!departmentQuantities[departmentId]) {
        //                     departmentQuantities[departmentId] = 0;
        //                 }
        //                 departmentQuantities[departmentId] += quantity;

        //                 // Aggregate by department-wise employee
        //                 if (!employeeQuantities[key]) {
        //                     employeeQuantities[key] = 0;
        //                 }
        //                 employeeQuantities[key] += quantity;
        //             });

        //             console.log("Aggregated Department Quantities:", departmentQuantities);
        //             console.log("Aggregated Employee Quantities:", employeeQuantities);

        //             if (!locationId) {
        //                 // ✅ Updating all locations
        //                 locations.value = Object.entries(listEfficiencyData.value).map(([locId, locData]) => ({
        //                     internalid: { value: locId },
        //                     name: { value: locData.location_name },
        //                     tmproduction_gold: locData.tmproduction_gold || 0,
        //                     loss_gold: locData.loss_gold || 0,
        //                     tmproduction_diamond: locData.tmproduction_diamond || 0,
        //                     loss_diamond: locData.loss_diamond || 0,
        //                     tmproduction_diamond_pieces: locData.tmproduction_diamond_pieces || 0,
        //                     loss_diamond_pieces: locData.loss_diamond_pieces || 0,
        //                     departments: Object.entries(locData.departments || {}).map(([deptId, dept]) => {
        //                         const goldRecoveryWeightDept = departmentQuantities[deptId] || 0;
        //                         const grossLossGoldDept = parseFloat(dept.loss || 0);
        //                         const netLossGoldDept = grossLossGoldDept - goldRecoveryWeightDept;

        //                         return {
        //                             name: dept.department_name,
        //                             production: dept.production,
        //                             loss: roundToTwo(grossLossGoldDept),
        //                             netLoss: roundToTwo(netLossGoldDept),
        //                             date: dept.date,
        //                             production_diamond: dept.production_diamond,
        //                             loss_diamond: dept.loss_diamond,
        //                             loss_diamond_pieces: dept.loss_diamond_pieces,
        //                             production_diamond_pieces: dept.production_diamond_pieces,
        //                             totalWeightGold: roundToTwo(goldRecoveryWeightDept),
        //                             employees: Object.entries(dept.employees || {}).map(([empId, emp]) => {
        //                                 const key = `${deptId}_${empId}`;
        //                                 const goldRecoveryWeight = employeeQuantities[key] || 0;
        //                                 const grossLossGold = parseFloat(emp.grossLoss || 0);
        //                                 const netLossGold = grossLossGold - goldRecoveryWeight;

        //                                 return {
        //                                     id: empId,
        //                                     name: emp.name !== "null null" ? emp.name : "Unknown Employee",
        //                                     tmProduction: emp.tmProduction || 0,
        //                                     tmProductionDiamond: emp.tmProductionDiamond || 0,
        //                                     grossLossDiamond: emp.grossLossDiamond || 0,
        //                                     grossLoss: roundToTwo(grossLossGold),
        //                                     loss: emp.loss || 0,
        //                                     netLoss: roundToTwo(netLossGold),
        //                                     recovery: emp.recovery || 0,
        //                                     date: emp.date || "N/A",
        //                                     tmGrossLossWeight: roundToTwo(goldRecoveryWeight)
        //                                 };
        //                             })
        //                         };
        //                     })
        //                 }));
        //             } else {
        //                 // ✅ Updating a specific location
        //                 const efficiencyData = listEfficiencyData.value[locationId];
        //                 const updatedLocation = locations.value.find(loc => loc.internalid.value === locationId);

        //                 if (updatedLocation && efficiencyData) {
        //                     updatedLocation.tmproduction_gold = efficiencyData.tmproduction_gold || 0;
        //                     updatedLocation.loss_gold = efficiencyData.loss_gold || 0;
        //                     updatedLocation.tmproduction_diamond = efficiencyData.tmproduction_diamond || 0;
        //                     updatedLocation.loss_diamond = efficiencyData.loss_diamond || 0;
        //                     updatedLocation.tmproduction_diamond_pieces = efficiencyData.tmproduction_diamond_pieces || 0;
        //                     updatedLocation.loss_diamond_pieces = efficiencyData.loss_diamond_pieces || 0;
        //                     updatedLocation.departments = Object.entries(efficiencyData.departments || {}).map(([deptId, dept]) => {
        //                         const goldRecoveryWeightDept = departmentQuantities[deptId] || 0;
        //                         const grossLossGoldDept = parseFloat(dept.loss || 0);
        //                         const netLossGoldDept = grossLossGoldDept - goldRecoveryWeightDept;

        //                         return {
        //                             name: dept.department_name,
        //                             production: dept.production,
        //                             loss: roundToTwo(grossLossGoldDept),
        //                             netLoss: roundToTwo(netLossGoldDept),
        //                             production_diamond: dept.production_diamond,
        //                             loss_diamond: dept.loss_diamond,
        //                             loss_diamond_pieces: dept.loss_diamond_pieces,
        //                             production_diamond_pieces: dept.production_diamond_pieces,
        //                             totalWeightGold: roundToTwo(goldRecoveryWeightDept),
        //                             employees: Object.entries(dept.employees || {}).map(([empId, emp]) => {
        //                                 const key = `${deptId}_${empId}`;
        //                                 const goldRecoveryWeight = employeeQuantities[key] || 0;
        //                                 const grossLossGold = parseFloat(emp.grossLoss || 0);
        //                                 const netLossGold = grossLossGold - goldRecoveryWeight;

        //                                 return {
        //                                     id: empId,
        //                                     name: emp.name !== "null null" ? emp.name : "Unknown Employee",
        //                                     tmProduction: emp.tmProduction || 0,
        //                                     tmProductionDiamond: emp.tmProductionDiamond || 0,
        //                                     grossLossDiamond: emp.grossLossDiamond || 0,
        //                                     grossLoss: roundToTwo(grossLossGold),
        //                                     tmProductionDiamondPieces: emp.tmProductionDiamondPieces || 0,
        //                                     grossLossDiamondPieces: emp.grossLossDiamondPieces || 0,
        //                                     loss: emp.loss || 0,
        //                                     netLoss: roundToTwo(netLossGold),
        //                                     recovery: emp.recovery || 0,
        //                                     date: emp.date || "N/A",
        //                                     tmGrossLossWeight: roundToTwo(goldRecoveryWeight)
        //                                 };
        //                             })
        //                         };
        //                     });
        //                 }
        //             }
        //         } else {
        //             showNoDataPopup.value = true;
        //             setTimeout(() => {
        //                 showNoDataPopup.value = false;
        //                 selectedDateRange.value = getDefaultDateRange();
        //             }, 3000);
        //         }
        //     } catch (err) {
        //         console.error("Error fetching efficiency analysis data:", err);
        //         showNoDataPopup.value = true;
        //         setTimeout(() => {
        //             showNoDataPopup.value = false;
        //             selectedDateRange.value = getDefaultDateRange();
        //         }, 3000);
        //     } finally {
        //         loading.value = false;
        //     }
        // };
        const maxRetries = 1; // Prevent infinite loop
        let retryCount = 0;
        const fetchEfficiencyAnalysisData = async (locationId = null) => {
            try {
                loading.value = true;
                showNoDataPopup.value = false;

                if (!selectedDateRange.value || selectedDateRange.value.length < 2) return;

                const formatDate = (date) => {
                    const offset = date.getTimezoneOffset();
                    date = new Date(date.getTime() - (offset * 60 * 1000));
                    return date.toISOString().split('T')[0];
                };

                const formattedStartDate = formatDate(selectedDateRange.value[0]);
                const formattedEndDate = formatDate(selectedDateRange.value[1]);

                // Fetch Efficiency Analysis Data (Overall - All Operations)
                const isRepairOnly = props.type === 'repair' ? true : props.type === 'production' ? false : null;
                await fetchListEfficiencyAnalysis(locationId, formattedStartDate, formattedEndDate, isRepairOnly);
                
                // ✅ COMPREHENSIVE CONSOLE LOGS FOR FETCHED DATA
                // console.log("\n" + "=".repeat(60));
                // console.log("🔵 EFFICIENCY ANALYSIS DATA FETCH - FRONTEND CONSOLE");
                // console.log("=".repeat(60));
                // console.log("📅 Date Range:", formattedStartDate, "to", formattedEndDate);
                // console.log("📍 Location ID:", locationId || "All Locations");
                // console.log("=".repeat(60));
                
                // Raw API Response
                // console.log("\n📊 RAW API RESPONSE (listEfficiencyData):");
                // console.log(JSON.stringify(listEfficiencyData.value, null, 2));
                
                if (listEfficiencyData.value && Object.keys(listEfficiencyData.value).length > 0) {

                    if (!locationId) {
                        // ✅ Updating all locations with departments, employees, and bag counts
                        locations.value = Object.entries(listEfficiencyData.value).map(([locId, locData]) => ({
                            internalid: { value: locId },
                            name: { value: locData.location_name },
                            departments: Object.entries(locData.departments || {}).map(([deptId, dept]) => {
                                return {
                                    id: deptId,
                                    name: dept.department_name,
                                    bag_count: dept.bag_count || 0,
                                    unique_bags_array: dept.unique_bags_array || [],
                                    category_count: dept.category_count || 0,
                                    unique_categories_array: dept.unique_categories_array || [],
                                    starting_qty: dept.starting_qty || 0,
                                    loss_qty: dept.loss_qty || 0,
                                    category_qty_map: dept.category_qty_map || {},
                                    category_bag_qty_map: dept.category_bag_qty_map || {},
                                    category_print_design_map: dept.category_print_design_map || {},
                                    category_bag_count_map: dept.category_bag_count_map || {},
                                    category_bag_names_map: dept.category_bag_names_map || {},
                                    category_bag_ids_map: dept.category_bag_ids_map || {},
                                    category_bag_print_design_map: dept.category_bag_print_design_map || {},
                                    category_bag_print_design_id_map: dept.category_bag_print_design_id_map || {},
                                    category_bag_sub_category_map: dept.category_bag_sub_category_map || {},
                                    category_bag_category_id_map: dept.category_bag_category_id_map || {},
                                    category_bag_sub_category_id_map: dept.category_bag_sub_category_id_map || {},
                                    category_print_design_id_map: dept.category_print_design_id_map || {},
                                    wax_tree_actual_production_gold: dept.wax_tree_actual_production_gold ?? null,
                                    wax_tree_loss_gold: dept.wax_tree_loss_gold ?? null,
                                    employees: (dept.employees_array || []).map(emp => {
                                        // Build category_qty_map from categories array
                                        const categoryQtyMap = {};
                                        if (emp.categories && Array.isArray(emp.categories)) {
                                            emp.categories.forEach(cat => {
                                                const key = `${emp.employee_id}_${cat.category_name}`;
                                                categoryQtyMap[key] = {
                                                    starting_qty_gold: cat.starting_qty_gold || 0,
                                                    starting_qty_diamond: cat.starting_qty_diamond || 0,
                                                    issued_qty_gold: cat.issued_qty_gold || 0,
                                                    issued_qty_diamond: cat.issued_qty_diamond || 0,
                                                    loss_qty_gold: cat.loss_qty_gold || 0,
                                                    loss_qty_diamond: cat.loss_qty_diamond || 0,
                                                    scrap_qty_gold: cat.scrap_qty_gold || 0,
                                                    scrap_qty_diamond: cat.scrap_qty_diamond || 0,
                                                    balance_qty_gold: cat.balance_qty_gold || 0,
                                                    balance_qty_diamond: cat.balance_qty_diamond || 0
                                                };
                                            });
                                        }
                                        
                                        const empObj = {
                                            id: emp.employee_id,
                                            name: emp.name,
                                            bag_count: emp.bag_count || 0,
                                            unique_bags_array: emp.unique_bags_array || [],
                                            category_count: emp.category_count || 0,
                                            unique_categories_array: emp.unique_categories_array || [],
                                            starting_qty: emp.starting_qty || 0,
                                            loss_qty: emp.loss_qty || 0,
                                            categories: emp.categories || [],
                                            category_qty_map: categoryQtyMap,
                                            category_print_design_map: emp.category_print_design_map || {},
                                            category_print_design_id_map: emp.category_print_design_id_map || {},
                                            category_bag_count_map: emp.category_bag_count_map || {},
                                            category_bag_names_map: emp.category_bag_names_map || {},
                                            category_bag_ids_map: emp.category_bag_ids_map || {},
                                            category_bag_print_design_map: emp.category_bag_print_design_map || {},
                                            category_bag_print_design_id_map: emp.category_bag_print_design_id_map || {},
                                            category_bag_sub_category_map: emp.category_bag_sub_category_map || {},
                                            category_bag_category_id_map: emp.category_bag_category_id_map || {},
                                            category_bag_sub_category_id_map: emp.category_bag_sub_category_id_map || {}
                                        };
                                        return empObj;
                                    })
                                };
                            })
                        }));

                        // ✅ DETAILED CONSOLE LOGS - STRUCTURED DATA
                        // console.log("\n📍 PROCESSED LOCATIONS DATA:");
                        // console.log("─".repeat(60));
                        locations.value.forEach((loc, locIdx) => {
                            // console.log(`\n[Location ${locIdx + 1}] ${loc.name.value}`);
                            // console.log(`Located ID: ${loc.internalid.value}`);
                            
                            loc.departments.forEach((dept, deptIdx) => {
                                // console.log(`\n  [Department ${deptIdx + 1}] ${dept.name} (ID: ${dept.id})`);
                                // console.log(`    Total Bags: ${dept.bag_count}`);
                                // console.log(`    Unique Bags: [${dept.unique_bags_array.join(', ')}]`);
                                // console.log(`    Total Categories: ${dept.category_count}`);
                                // console.log(`    Unique Categories: [${dept.unique_categories_array.join(', ')}]`);
                                // console.log(`    Department Starting Qty: ${dept.starting_qty}`);
                                // console.log(`    Department Loss Qty: ${dept.loss_qty}`);
                                // console.log(`    Category Qty Map:`, dept.category_qty_map);
                                // console.log(`    Category Print Design Map:`, dept.category_print_design_map);
                                // console.log(`    Total Employees: ${dept.employees.length}`);
                                
                                dept.employees.forEach((emp, empIdx) => {
                                    // console.log(`\n    [Employee ${empIdx + 1}] ${emp.name} (ID: ${emp.id})`);
                                    // console.log(`      Bag Count: ${emp.bag_count}`);
                                    // console.log(`      Unique Bags: [${emp.unique_bags_array.join(', ')}]`);
                                    // console.log(`      Category Count: ${emp.category_count}`);
                                    // console.log(`      Unique Categories: [${emp.unique_categories_array.join(', ')}]`);
                                    // console.log(`      Starting Qty: ${emp.starting_qty}`);
                                    // console.log(`      Loss Qty: ${emp.loss_qty}`);
                                    // console.log(`      Categories Detail:`, emp.categories);
                                    // console.log(`      Category Qty Map:`, emp.category_qty_map);
                                });
                            });
                        });
                        // console.log("\n" + "─".repeat(60));
                        
                        // ✅ SUMMARY STATISTICS
                        // console.log("\n📈 FETCH SUMMARY:");
                        // console.log("─".repeat(60));
                        let totalDepts = 0, totalEmps = 0, totalBags = 0, totalCategories = 0;
                        locations.value.forEach(loc => {
                            loc.departments.forEach(dept => {
                                totalDepts++;
                                totalBags += dept.bag_count;
                                totalCategories += dept.category_count;
                                totalEmps += dept.employees.length;
                            });
                        });
                        // console.log(`✅ Total Locations: ${locations.value.length}`);
                        // console.log(`✅ Total Departments: ${totalDepts}`);
                        // console.log(`✅ Total Employees: ${totalEmps}`);
                        // console.log(`✅ Total Bags: ${totalBags}`);
                        // console.log(`✅ Total Unique Categories: ${totalCategories}`);
                        // console.log("─".repeat(60));
                        // console.log("🟢 DATA FETCH COMPLETE - Ready for UI display");
                        // console.log("=".repeat(60) + "\n");
                    } else {
                        // ✅ locationId provided — update only that specific location's departments
                        const locData = listEfficiencyData.value[locationId];
                        if (locData) {
                            const targetLoc = locations.value.find(loc => loc.internalid.value == locationId);
                            if (targetLoc) {
                                targetLoc.departments = Object.entries(locData.departments || {}).map(([deptId, dept]) => {
                                    const categoryQtyMapByEmp = {};
                                    const employees = (dept.employees_array || []).map(emp => {
                                        const empCategoryQtyMap = {};
                                        if (emp.categories && Array.isArray(emp.categories)) {
                                            emp.categories.forEach(cat => {
                                                const key = `${emp.employee_id}_${cat.category_name}`;
                                                empCategoryQtyMap[key] = {
                                                    starting_qty_gold: cat.starting_qty_gold || 0,
                                                    starting_qty_diamond: cat.starting_qty_diamond || 0,
                                                    issued_qty_gold: cat.issued_qty_gold || 0,
                                                    issued_qty_diamond: cat.issued_qty_diamond || 0,
                                                    loss_qty_gold: cat.loss_qty_gold || 0,
                                                    loss_qty_diamond: cat.loss_qty_diamond || 0,
                                                    scrap_qty_gold: cat.scrap_qty_gold || 0,
                                                    scrap_qty_diamond: cat.scrap_qty_diamond || 0,
                                                    balance_qty_gold: cat.balance_qty_gold || 0,
                                                    balance_qty_diamond: cat.balance_qty_diamond || 0
                                                };
                                            });
                                        }
                                        return {
                                            id: emp.employee_id,
                                            name: emp.name,
                                            bag_count: emp.bag_count || 0,
                                            unique_bags_array: emp.unique_bags_array || [],
                                            category_count: emp.category_count || 0,
                                            unique_categories_array: emp.unique_categories_array || [],
                                            starting_qty: emp.starting_qty || 0,
                                            loss_qty: emp.loss_qty || 0,
                                            categories: emp.categories || [],
                                            category_qty_map: empCategoryQtyMap,
                                            category_print_design_map: emp.category_print_design_map || {},
                                            category_bag_count_map: emp.category_bag_count_map || {},
                                            category_bag_names_map: emp.category_bag_names_map || {},
                                            category_bag_ids_map: emp.category_bag_ids_map || {},
                                            category_bag_print_design_map: emp.category_bag_print_design_map || {},
                                            category_bag_print_design_id_map: emp.category_bag_print_design_id_map || {},
                                            category_bag_sub_category_map: emp.category_bag_sub_category_map || {},
                                            category_bag_category_id_map: emp.category_bag_category_id_map || {},
                                            category_bag_sub_category_id_map: emp.category_bag_sub_category_id_map || {},
                                            category_print_design_id_map: emp.category_print_design_id_map || {}
                                        };
                                    });
                                    return {
                                        id: deptId,
                                        name: dept.department_name,
                                        bag_count: dept.bag_count || 0,
                                        unique_bags_array: dept.unique_bags_array || [],
                                        category_count: dept.category_count || 0,
                                        unique_categories_array: dept.unique_categories_array || [],
                                        starting_qty: dept.starting_qty || 0,
                                        loss_qty: dept.loss_qty || 0,
                                        category_qty_map: dept.category_qty_map || {},
                                        category_bag_qty_map: dept.category_bag_qty_map || {},
                                        category_print_design_map: dept.category_print_design_map || {},
                                        category_bag_count_map: dept.category_bag_count_map || {},
                                        category_bag_names_map: dept.category_bag_names_map || {},
                                        category_bag_ids_map: dept.category_bag_ids_map || {},
                                        category_bag_print_design_map: dept.category_bag_print_design_map || {},
                                        category_bag_print_design_id_map: dept.category_bag_print_design_id_map || {},
                                        category_bag_sub_category_map: dept.category_bag_sub_category_map || {},
                                        category_bag_category_id_map: dept.category_bag_category_id_map || {},
                                        category_bag_sub_category_id_map: dept.category_bag_sub_category_id_map || {},
                                        category_print_design_id_map: dept.category_print_design_id_map || {},
                                        wax_tree_actual_production_gold: dept.wax_tree_actual_production_gold ?? null,
                                        wax_tree_loss_gold: dept.wax_tree_loss_gold ?? null,
                                        employees
                                    };
                                });
                            }
                        }
                    }

                    // Fetch global avg recovery % once for the selected date range
                    // console.log('[Recovery] Fetching with dates:', formattedStartDate, formattedEndDate);
                    const recoveryData = await fetchRecoveryDataByDept(formattedStartDate, formattedEndDate);
                    globalAvgRecoveryPercentage.value = parseFloat(recoveryData.avgRecoveryPercentage || 0);
                    // console.log('[Recovery] Global avg recovery %:', globalAvgRecoveryPercentage.value);

                    isInitialLoading.value = false;
                } else {
                    console.warn("❌ NO DATA FOUND for the selected date range");
                    showNoDataPopup.value = true;
                    isInitialLoading.value = false;
                }

                loading.value = false;
            } catch (error) {
                console.error('Error fetching efficiency analysis data:', error);
                loading.value = false;
                isInitialLoading.value = false;
                showNoDataPopup.value = true;
            }
        };

        // Function to close no data popup
        const closeNoDataPopup = () => {
            console.log("Closing No Data Popup");
            showNoDataPopup.value = false;
            // Reset departments so cards show with all 0 values, keeping date picker usable
            locations.value = locations.value.map(loc => ({ ...loc, departments: [] }));
        };

        const formatDate = (date) => {
            const offset = date.getTimezoneOffset();
            date = new Date(date.getTime() - (offset * 60 * 1000)); // Adjust for timezone offset
            return date.toISOString().split('T')[0]; // Format as YYYY-MM-DD
        };

        // Watch for changes in the selected date range and re-fetch data
        watch(selectedDateRange, async (newRange) => {
            if (newRange && newRange.length === 2) {
                console.log("Date Range Changed:", newRange);
                const formattedStartDate = formatDate(newRange[0]);
                const formattedEndDate = formatDate(newRange[1]);

                console.log(`Date Range Changed: ${formattedStartDate} - ${formattedEndDate}`);

                if (expandedLocation.value) {
                    await fetchEfficiencyAnalysisData(expandedLocation.value);

                    const selectedLocation = locations.value.find(loc => loc.internalid.value === expandedLocation.value);

                    if (showDepartments.value) {
                        selectedDepartments.value = selectedLocation?.departments || [];
                        selectedDepartmentData.value = [...selectedDepartments.value];
                    }
                    if (showEmployees.value) {
                        selectedEmployees.value = selectedDepartments.value
                            .find(dept => dept.name === expandedDepartment.value)?.employees || [];
                    }
                } else {
                    await fetchEfficiencyAnalysisData();
                }

                updateCharts(); // Ensure charts are updated with new data
            }
        });



        // Function to toggle location view and fetch department data
        const toggleLocationView = async (locationId) => {
            if (expandedLocation.value == locationId) {
                expandedLocation.value = null;
                expandedDepartment.value = null;
                showDepartments.value = false;
                showEmployees.value = false;
                showTable.value = false;
                dashboardTitle.value = baseTitle;
                updateCharts();
            } else {
                expandedLocation.value = locationId;
                expandedDepartment.value = null;
                showDepartments.value = true;
                showEmployees.value = false;
                showTable.value = true;
                showEmployeesTable.value = false;
                dashboardTitle.value = "Department Details";

                await fetchEfficiencyAnalysisData(locationId);

                const selectedLocation = locations.value.find(loc => loc.internalid.value === locationId);
                selectedDepartments.value = [...(selectedLocation?.departments || [])]; // Force reactivity
                
                // Set department data with calculated bag counts
                selectedDepartmentData.value = [...selectedDepartments.value];

                // Log departments with bag counts and unique bag names
                // let deptLogMsg = "=== DEPARTMENTS WITH BAG COUNTS ===\n";
                // selectedDepartments.value.forEach(dept => {
                //     const bagNames = dept.unique_bags_array || [];
                //     const categoryNames = dept.unique_categories_array || [];
                //     deptLogMsg += `Dept: ${dept.name} | Bag Count: ${dept.bag_count || 0} | Category Count: ${dept.category_count || 0} | Bags: [${bagNames.join(', ')}] | Categories: [${categoryNames.join(', ')}]\n`;
                // });
                // deptLogMsg += "===================================";
                // console.log(deptLogMsg);
                
                updateCharts();
            }
        };


        // Function to toggle department view and fetch employee data
        const toggleDepartmentView = (deptName) => {
            if (expandedDepartment.value === deptName) {
                expandedDepartment.value = null;
                showEmployees.value = false;
                showEmployeesTable.value = false;
                dashboardTitle.value = "Department Details";

            } else {
                expandedDepartment.value = deptName;
                showEmployees.value = true;
                showEmployeesTable.value = true;
                dashboardTitle.value = "Employee Details";

                // Get employees from the already-populated selectedDepartments
                const selectedDept = selectedDepartments.value.find(dept => dept.name === deptName);
                
                // console.log(`\n🔵 TOGGLE DEPARTMENT VIEW - ${deptName}`);
                // console.log(`Selected Department:`, selectedDept);
                // console.log(`Employees Array:`, selectedDept?.employees);
                
                // Add department ID to each employee
                selectedEmployees.value = (selectedDept?.employees || []).map(emp => {
                    return {
                        ...emp,
                        departmentId: selectedDept.id
                    };
                });
                
                // console.log(`✅ selectedEmployees populated with ${selectedEmployees.value.length} employees`);
                // console.log(`📋 Employee Details:`, selectedEmployees.value);
                // console.log(`Department: ${deptName} (ID: ${selectedDept?.id}) - Employees:`, selectedEmployees.value);
            }
            updateCharts();
        };


        // const updateCharts = () => {
        //     setTimeout(() => {
        //         let labels, datasets;
        //         let productionLabels = [];
        //         let productionValues = [];

        //         if (!showDepartments.value && !showEmployees.value) {
        //             // ✅ Line Chart for Overall Location-based Data
        //             labels = locations.value.map(loc => loc.name.value);
        //             let goldProductionData = locations.value.map(loc => parseInt(loc.tmproduction_gold || 0));
        //             let goldLossData = locations.value.map(loc => parseInt(loc.loss_gold || 0));
        //             let diamondProductionData = locations.value.map(loc => parseInt(loc.tmproduction_diamond || 0));
        //             let diamondLossData = locations.value.map(loc => parseInt(loc.loss_diamond || 0));
        //             let diamondProductionPiecesData = locations.value.map(loc => parseInt(loc.tmproduction_diamond_pieces || 0));
        //             let diamondLossPiecesData = locations.value.map(loc => parseInt(loc.loss_diamond_pieces || 0));

        //             datasets = [
        //                 {
        //                     label: "Gold Production (g)",
        //                     data: goldProductionData,
        //                     borderColor: "#B59E5F",
        //                     backgroundColor: "rgba(181, 158, 95, 0.2)",
        //                     fill: true,
        //                     tension: 0.4,
        //                 },
        //                 {
        //                     label: "Gold Loss (g)",
        //                     data: goldLossData,
        //                     borderColor: "#D9A066",
        //                     backgroundColor: "rgba(217, 160, 102, 0.2)",
        //                     fill: true,
        //                     tension: 0.4,
        //                 },
        //                 {
        //                     label: "Diamond Production (cts)",
        //                     data: diamondProductionData,
        //                     borderColor: "#7092BE",
        //                     backgroundColor: "rgba(112, 146, 190, 0.2)",
        //                     fill: true,
        //                     tension: 0.4,
        //                 },
        //                 {
        //                     label: "Diamond Loss (cts)",
        //                     data: diamondLossData,
        //                     borderColor: "#A0B9D9",
        //                     backgroundColor: "rgba(160, 185, 217, 0.2)",
        //                     fill: true,
        //                     tension: 0.4,
        //                 },
        //                 {
        //                     label: "Diamond Production (pieces)",
        //                     data: diamondProductionPiecesData,
        //                     borderColor: "#5B7C99",
        //                     backgroundColor: "rgba(91, 124, 153, 0.2)",
        //                     fill: true,
        //                     tension: 0.4,
        //                 },
        //                 {
        //                     label: "Diamond Loss (pieces)",
        //                     data: diamondLossPiecesData,
        //                     borderColor: "#7F9FB9",
        //                     backgroundColor: "rgba(127, 159, 185, 0.2)",
        //                     fill: true,
        //                     tension: 0.4,
        //                 }
        //             ];

        //             // ✅ Doughnut Chart for Overall Production
        //             productionLabels = ["Gold Production", "Diamond Production (cts)", "Diamond Production (pieces)"];
        //             productionValues = [
        //                 locations.value.reduce((sum, loc) => sum + parseInt(loc.tmproduction_gold || 0), 0),
        //                 locations.value.reduce((sum, loc) => sum + parseInt(loc.tmproduction_diamond || 0), 0),
        //                 locations.value.reduce((sum, loc) => sum + parseInt(loc.tmproduction_diamond_pieces || 0), 0)
        //             ];

        //         } else if (showDepartments.value && !showEmployees.value) {
        //             // ✅ Department-specific Line Chart
        //             labels = selectedDepartments.value.map(dept => dept.name);
        //             let productionData = selectedDepartments.value.map(dept => parseInt(dept.production || 0));
        //             let lossData = selectedDepartments.value.map(dept => parseInt(dept.loss || 0));

        //             datasets = [
        //                 {
        //                     label: "Production",
        //                     data: productionData,
        //                     borderColor: "#10B981",
        //                     backgroundColor: "rgba(16,185,129,0.2)",
        //                     fill: true,
        //                     tension: 0.4,
        //                 },
        //                 {
        //                     label: "Loss",
        //                     data: lossData,
        //                     borderColor: "#DC2626",
        //                     backgroundColor: "rgba(220,38,38,0.2)",
        //                     fill: true,
        //                     tension: 0.4,
        //                 }
        //             ];

        //             // ✅ Doughnut Chart Uses Department Data
        //             productionLabels = selectedDepartments.value.map(dept => dept.name);
        //             productionValues = selectedDepartments.value.map(dept => parseInt(dept.production || 0));
        //         } else {
        //             // ✅ Employee-specific Line Chart
        //             labels = selectedEmployees.value.map(emp => emp.name);
        //             let productionData = selectedEmployees.value.map(emp => parseFloat(emp.tmProduction || 0));
        //             let lossData = selectedEmployees.value.map(emp => parseFloat(emp.loss || 0));

        //             datasets = [
        //                 {
        //                     label: "Production",
        //                     data: productionData,
        //                     borderColor: "#10B981",
        //                     backgroundColor: "rgba(16,185,129,0.2)",
        //                     fill: true,
        //                     tension: 0.4,
        //                 },
        //                 {
        //                     label: "Loss",
        //                     data: lossData,
        //                     borderColor: "#DC2626",
        //                     backgroundColor: "rgba(220,38,38,0.2)",
        //                     fill: true,
        //                     tension: 0.4,
        //                 }
        //             ];

        //             // ✅ Doughnut Chart Uses Employee Data
        //             productionLabels = selectedEmployees.value.map(emp => emp.name);
        //             productionValues = selectedEmployees.value.map(emp => parseInt(emp.tmProduction || 0));
        //         }

        //         // ✅ Line Chart for `cryptoChart`
        //         const ctx1 = document.getElementById('cryptoChart')?.getContext('2d');
        //         if (ctx1) {
        //             if (cryptoChart) cryptoChart.destroy();
        //             cryptoChart = new Chart(ctx1, {
        //                 type: "line",
        //                 data: {
        //                     labels: labels,
        //                     datasets: datasets
        //                 },
        //                 options: {
        //                     responsive: true,
        //                     maintainAspectRatio: false,
        //                     plugins: {
        //                         legend: { display: true, position: 'top' },
        //                         tooltip: { enabled: true }
        //                     },
        //                     scales: {
        //                         y: {
        //                             beginAtZero: true,
        //                             grid: { color: "#D1D5DB" }
        //                         },
        //                         x: {
        //                             grid: { color: "#D1D5DB" }
        //                         }
        //                     }
        //                 }
        //             });
        //         }

        //         // ✅ Doughnut Chart for `productionChart`
        //         const ctx2 = document.getElementById('productionChart')?.getContext('2d');
        //         if (ctx2) {
        //             if (productionChart) productionChart.destroy();
        //             productionChart = new Chart(ctx2, {
        //                 type: "doughnut",
        //                 data: {
        //                     labels: productionLabels,
        //                     datasets: [{
        //                         data: productionValues,
        //                         backgroundColor: ["#3B82F6", "#F59E0B", "#8B5CF6", "#14B8A6", "#EF4444"], // New Colors

        //                         hoverOffset: 10
        //                     }]
        //                 },
        //                 options: {
        //                     responsive: true,
        //                     maintainAspectRatio: false,
        //                     plugins: {
        //                         legend: { display: true, position: 'right' },
        //                         tooltip: { enabled: true }
        //                     }
        //                 }
        //             });
        //         }
        //     }, 500);
        // };

        const updateCharts = () => {
            setTimeout(() => {
                // ── helpers to sum category_qty_map for a dept or employee ──────────
                const sumCatMap = (entity, field) => {
                    if (!entity.category_qty_map || !entity.unique_categories_array) return 0;
                    return entity.unique_categories_array.reduce((s, cat) => {
                        const d = entity.category_qty_map[`${entity.id}_${cat}`] || {};
                        return s + parseFloat(d[field] || 0);
                    }, 0);
                };
                const actualProdGold    = (e) => sumCatMap(e, 'starting_qty_gold')    + sumCatMap(e, 'issued_qty_gold')    - sumCatMap(e, 'loss_qty_gold')    - sumCatMap(e, 'scrap_qty_gold')    - sumCatMap(e, 'balance_qty_gold');
                const actualProdDiamond = (e) => sumCatMap(e, 'starting_qty_diamond') + sumCatMap(e, 'issued_qty_diamond') - sumCatMap(e, 'loss_qty_diamond') - sumCatMap(e, 'scrap_qty_diamond') - sumCatMap(e, 'balance_qty_diamond');
                const lossGold          = (e) => sumCatMap(e, 'loss_qty_gold');
                const lossDiamond       = (e) => sumCatMap(e, 'loss_qty_diamond');

                let labels = [];
                let barDatasets = [];
                let doughnutLabels = [];
                let doughnutValues = [];

                if (!showDepartments.value && !showEmployees.value) {
                    // Location level — one bar per location, aggregated across all its departments
                    labels = locations.value.map(l => l.name.value);

                    const prodGoldData    = locations.value.map(loc =>
                        parseFloat(roundToTwo(loc.departments.reduce((s, d) => s + actualProdGold(d), 0))));
                    const lossGoldData    = locations.value.map(loc =>
                        parseFloat(roundToTwo(loc.departments.reduce((s, d) => s + lossGold(d), 0))));
                    const prodDiamondData = locations.value.map(loc =>
                        parseFloat(roundToTwo(loc.departments.reduce((s, d) => s + actualProdDiamond(d), 0))));
                    const lossDiamondData = locations.value.map(loc =>
                        parseFloat(roundToTwo(loc.departments.reduce((s, d) => s + lossDiamond(d), 0))));

                    barDatasets = [
                        { label: 'Actual Production Gold (g)',    data: prodGoldData,    backgroundColor: '#e5c000', borderColor: '#e5c000', borderWidth: 1 },
                        { label: 'Loss Gold (g)',                  data: lossGoldData,    backgroundColor: '#ff4034', borderColor: '#ff4034', borderWidth: 1 },
                        { label: 'Actual Production Diamond (ct)', data: prodDiamondData, backgroundColor: '#4993f7', borderColor: '#4993f7', borderWidth: 1 },
                        { label: 'Loss Diamond (ct)',              data: lossDiamondData, backgroundColor: 'rgba(220,38,38,0.5)', borderColor: '#DC2626', borderWidth: 1 },
                    ];

                    const totalProdGold    = prodGoldData.reduce((s, v) => s + v, 0);
                    const totalProdDiamond = prodDiamondData.reduce((s, v) => s + v, 0);
                    const totalLossGold    = lossGoldData.reduce((s, v) => s + v, 0);
                    const totalLossDiamond = lossDiamondData.reduce((s, v) => s + v, 0);
                    doughnutLabels = ['Production Gold', 'Production Diamond', 'Loss Gold', 'Loss Diamond'];
                    doughnutValues = [totalProdGold, totalProdDiamond, totalLossGold, totalLossDiamond];
                } else if (showDepartments.value && !showEmployees.value) {
                    // Department level — mirror the department table
                    const depts = selectedDepartmentData.value;
                    labels = depts.map(d => d.name);

                    const prodGoldData    = depts.map(d => parseFloat(roundToTwo(actualProdGold(d))));
                    const lossGoldData    = depts.map(d => parseFloat(roundToTwo(lossGold(d))));
                    const prodDiamondData = depts.map(d => parseFloat(roundToTwo(actualProdDiamond(d))));
                    const lossDiamondData = depts.map(d => parseFloat(roundToTwo(lossDiamond(d))));

                    barDatasets = [
                        { label: 'Actual Production Gold (g)',    data: prodGoldData,    backgroundColor: '#e5c000',  borderColor: '#e5c000', borderWidth: 1 },
                        { label: 'Loss Gold (g)',                  data: lossGoldData,    backgroundColor: '#ff4034',   borderColor: '#ff4034', borderWidth: 1 },
                        { label: 'Actual Production Diamond (ct)', data: prodDiamondData, backgroundColor: '#4993f7', borderColor: '#4993f7', borderWidth: 1 },
                        { label: 'Loss Diamond (ct)',              data: lossDiamondData, backgroundColor: 'rgba(220,38,38,0.5)',   borderColor: '#DC2626', borderWidth: 1 },
                    ];

                    const totalProdGold    = prodGoldData.reduce((s, v) => s + v, 0);
                    const totalProdDiamond = prodDiamondData.reduce((s, v) => s + v, 0);
                    const totalLossGold    = lossGoldData.reduce((s, v) => s + v, 0);
                    const totalLossDiamond = lossDiamondData.reduce((s, v) => s + v, 0);
                    doughnutLabels = ['Production Gold', 'Production Diamond', 'Loss Gold', 'Loss Diamond'];
                    doughnutValues = [totalProdGold, totalProdDiamond, totalLossGold, totalLossDiamond];
                } else {
                    // Employee level — mirror the employee table
                    const emps = selectedEmployees.value;
                    labels = emps.map(e => e.name);

                    const prodGoldData    = emps.map(e => parseFloat(roundToTwo(actualProdGold(e))));
                    const lossGoldData    = emps.map(e => parseFloat(roundToTwo(lossGold(e))));
                    const prodDiamondData = emps.map(e => parseFloat(roundToTwo(actualProdDiamond(e))));
                    const lossDiamondData = emps.map(e => parseFloat(roundToTwo(lossDiamond(e))));

                    barDatasets = [
                        { label: 'Actual Production Gold (g)',    data: prodGoldData,    backgroundColor: '#e5c000',  borderColor: '#e5c000', borderWidth: 1 },
                        { label: 'Loss Gold (g)',                  data: lossGoldData,    backgroundColor: '#ff4034',   borderColor: '#ff4034', borderWidth: 1 },
                        { label: 'Actual Production Diamond (ct)', data: prodDiamondData, backgroundColor: '#4993f7', borderColor: '#4993f7', borderWidth: 1 },
                        { label: 'Loss Diamond (ct)',              data: lossDiamondData, backgroundColor: 'rgba(220,38,38,0.5)',   borderColor: '#DC2626', borderWidth: 1 },
                    ];

                    const totalProdGold    = prodGoldData.reduce((s, v) => s + v, 0);
                    const totalProdDiamond = prodDiamondData.reduce((s, v) => s + v, 0);
                    const totalLossGold    = lossGoldData.reduce((s, v) => s + v, 0);
                    const totalLossDiamond = lossDiamondData.reduce((s, v) => s + v, 0);
                    doughnutLabels = ['Production Gold', 'Production Diamond', 'Loss Gold', 'Loss Diamond'];
                    doughnutValues = [totalProdGold, totalProdDiamond, totalLossGold, totalLossDiamond];
                }

                // ── Bar chart (cryptoChart) ───────────────────────────────────────
                const ctx1 = document.getElementById('cryptoChart')?.getContext('2d');
                if (ctx1) {
                    if (cryptoChart) cryptoChart.destroy();
                    cryptoChart = new Chart(ctx1, {
                        type: 'bar',
                        data: { labels, datasets: barDatasets },
                        options: {
                            responsive: true,
                            maintainAspectRatio: false,
                            plugins: {
                                legend: { display: true, position: 'top', labels: { font: { size: 10 } } },
                                tooltip: { enabled: true }
                            },
                            scales: {
                                y: { beginAtZero: true, grid: { color: '#E5E7EB' }, ticks: { font: { size: 10 } } },
                                x: { grid: { display: false }, ticks: { font: { size: 10 }, maxRotation: 30 } }
                            }
                        }
                    });
                }

                // ── Doughnut chart (productionChart) ─────────────────────────────
                const ctx2 = document.getElementById('productionChart')?.getContext('2d');
                if (ctx2) {
                    if (productionChart) productionChart.destroy();
                    productionChart = new Chart(ctx2, {
                        type: 'doughnut',
                        data: {
                            labels: doughnutLabels,
                            datasets: [{
                                data: doughnutValues,
                                backgroundColor: ['#e5c000', '#4993f7', '#ff4034', '#DC2626'],
                                hoverOffset: 8
                            }]
                        },
                        options: {
                            responsive: true,
                            maintainAspectRatio: false,
                            plugins: {
                                legend: { display: true, position: 'bottom', labels: { font: { size: 10 }, boxWidth: 12 } },
                                tooltip: {
                                    callbacks: {
                                        label: (ctx) => ` ${ctx.label}: ${ctx.parsed.toFixed(2)}`
                                    }
                                }
                            }
                        }
                    });
                }
            }, 300);
        };



        onMounted(async () => {
            await fetchLocations();  // Ensure locations are loaded
            await fetchEfficiencyAnalysisData();  // Ensure efficiency data is loaded
            setTimeout(() => {
                updateCharts();  // Delay execution to allow DOM to be ready
            }, 500);
        });



        return {
            expandedLocation,
            expandedDepartment,
            showDepartments,
            showEmployees,
            loading,
            dashboardTitle,
            locations,
            selectedDepartments,
            selectedEmployees,
            toggleLocationView,
            toggleDepartmentView,
            showTable,
            selectedDepartmentData,
            showEmployeesTable,
            selectedDateRange,
            isInitialLoading,
            showNoDataPopup,
            selectedDateRange,
            closeNoDataPopup,
            downloadData,
            formatName,
            roundToTwo,
            getLocationTotalBagCount,
            getLocationTotalActualProductionGold,
            getLocationTotalActualProductionDiamond,
            getLocationTotalIssuedQtyGold,
            getLocationTotalIssuedQtyDiamond,
            getLocationTotalLossQtyGold,
            getLocationTotalLossQtyDiamond,
            getLocationTotalIssuedPiecesDiamond,
            getLocationTotalLossPiecesDiamond,
            getDepartmentTotalIssuedQtyGold,
            getDepartmentTotalIssuedQtyDiamond,
            getDepartmentTotalLossQtyGold,
            getDepartmentTotalLossQtyDiamond,
            getEmployeeTotalIssuedQtyGold,
            getEmployeeTotalIssuedQtyDiamond,
            getEmployeeTotalLossQtyGold,
            getEmployeeTotalLossQtyDiamond,
            getEmployeeTotalActualProductionGold,
            getEmployeeTotalActualProductionDiamond,
            getEmployeeTotalStartingQtyAllCategories,
            getEmployeeTotalIssuedQtyAllCategories,
            getEmployeeTotalLossQtyAllCategories,
            getDepartmentTotalActualProductionGold,
            getDepartmentWaxTreeLossGold,
            getDepartmentTotalActualProductionDiamond,
            getDepartmentTotalIssuedPiecesDiamond,
            getDepartmentTotalLossPiecesDiamond,
            getEmployeeTotalIssuedPiecesDiamond,
            getEmployeeTotalLossPiecesDiamond,
            getCategoryStartingQty,
            getCategoryStartingQtyGold,
            getCategoryStartingQtyGoldRaw,
            getCategoryStartingQtyDiamond,
            getCategoryStartingQtyDiamondRaw,
            getCategoryIssuedQty,
            getCategoryIssuedQtyGold,
            getCategoryIssuedQtyGoldRaw,
            getCategoryIssuedQtyDiamond,
            getCategoryIssuedQtyDiamondRaw,
            getCategoryLossQty,
            getCategoryLossQtyGold,
            getCategoryLossQtyGoldRaw,
            getCategoryLossQtyDiamond,
            getCategoryLossQtyDiamondRaw,
            getCategoryScrapQty,
            getCategoryScrapQtyGold,
            getCategoryScrapQtyGoldRaw,
            getCategoryScrapQtyDiamond,
            getCategoryScrapQtyDiamondRaw,
            getCategoryBalanceQty,
            getCategoryBalanceQtyGold,
            getCategoryBalanceQtyGoldRaw,
            getCategoryBalanceQtyDiamond,
            getCategoryBalanceQtyDiamondRaw,
            getBagStartingQtyGoldRaw,
            getBagIssuedQtyGoldRaw,
            getBagLossQtyGoldRaw,
            getBagScrapQtyGoldRaw,
            getBagBalanceQtyGoldRaw,
            getBagStartingQtyDiamondRaw,
            getBagIssuedQtyDiamondRaw,
            getBagLossQtyDiamondRaw,
            getBagScrapQtyDiamondRaw,
            getBagBalanceQtyDiamondRaw,
            getEmpBagStartingQtyGoldRaw,
            getEmpBagIssuedQtyGoldRaw,
            getEmpBagLossQtyGoldRaw,
            getEmpBagScrapQtyGoldRaw,
            getEmpBagBalanceQtyGoldRaw,
            getEmpBagStartingQtyDiamondRaw,
            getEmpBagIssuedQtyDiamondRaw,
            getEmpBagLossQtyDiamondRaw,
            getEmpBagScrapQtyDiamondRaw,
            getEmpBagBalanceQtyDiamondRaw,
            getBagPureWeight,
            getBagPureLoss,
            getBagPurityFactor,
            getEmpBagPureWeight,
            getEmpBagPureLoss,
            getDeptTotals,
            getEmpTotals,
            getEmpBagPurityFactor,
            getBagNsUrl,
            getEmpBagNsUrl,
            getDeptNsUrl,
            getEmpNsUrl,
            getStyleNsUrlPerBag,
            getCategoryNsUrl,
            getSubCategoryNsUrl,
            totalDeptActualProductionGold,
            totalDeptGrossLossGold,
            totalDeptActualProductionDiamond,
            totalDeptGrossLossDiamond,
            totalDeptGoldRecoveryWeight,
            totalDeptNetLossGold,
            totalDeptTmProductionGold,
            totalDeptTmProductionDiamond,
            totalDeptBagCount,
            totalDeptStartingQuantityGold,
            totalDeptStartingQuantityDiamond,
            totalDeptIssuedQtyGold,
            totalDeptLossQtyGold,
            totalDeptScrapQtyGold,
            totalDeptReceivedQtyGold,
            totalDeptIssuedNetWeightGold,
            totalDeptIssuedQtyDiamond,
            totalDeptLossQtyDiamond,
            totalEmpActualProductionGold,
            totalEmpGrossLossGold,
            totalEmpActualProductionDiamond,
            totalEmpGrossLossDiamond,
            totalEmpGoldRecoveryWeight,
            totalEmpNetLossGold,
            totalEmpTmProductionGold,
            totalEmpTmProductionDiamond,
            totalEmpBagCount,
            totalEmpStartingQuantityGold,
            totalEmpIssuedQuantityGold,
            totalEmpLossQuantityGold,
            totalEmpScrapQtyGold,
            totalEmpReceivedQtyGold,
            totalEmpIssuedNetWeightGold,
            totalEmpStartingQuantityDiamond,
            totalEmpIssuedQuantityDiamond,
            totalEmpLossQuantityDiamond,
            totalEmpActualProductionGoldCalculated,
            totalEmpActualProductionDiamondCalculated,
            totalDeptPureWeight,
            totalDeptPureLoss,
            totalEmpPureWeight,
            totalEmpPureLoss,
            showNoDataMessage,
            fetchEfficiencyAnalysisData,
            globalAvgRecoveryPercentage,
            calculateGoldLossPercentage,
            calculateDiamondLossPercentage,
            getEmployeeCategoryStartingQty,
            getEmployeeCategoryStartingQtyGold,
            getEmployeeCategoryStartingQtyGoldRaw,
            getEmployeeCategoryStartingQtyDiamond,
            getEmployeeCategoryStartingQtyDiamondRaw,
            getEmployeeCategoryIssuedQty,
            getEmployeeCategoryIssuedQtyGold,
            getEmployeeCategoryIssuedQtyGoldRaw,
            getEmployeeCategoryIssuedQtyDiamond,
            getEmployeeCategoryIssuedQtyDiamondRaw,
            getEmployeeCategoryLossQty,
            getEmployeeCategoryLossQtyGold,
            getEmployeeCategoryLossQtyGoldRaw,
            getEmployeeCategoryLossQtyDiamond,
            getEmployeeCategoryLossQtyDiamondRaw,
            getEmployeeCategoryScrapQty,
            getEmployeeCategoryScrapQtyGold,
            getEmployeeCategoryScrapQtyGoldRaw,
            getEmployeeCategoryScrapQtyDiamond,
            getEmployeeCategoryScrapQtyDiamondRaw,
            getEmployeeCategoryBalanceQty,
            getEmployeeCategoryBalanceQtyGold,
            getEmployeeCategoryBalanceQtyGoldRaw,
            getEmployeeCategoryBalanceQtyDiamond,
            getEmployeeCategoryBalanceQtyDiamondRaw


        };
    }
};
</script>



<style scoped>

/* ✅ Date Picker Container */
:deep(.dp__menu) {
    border-radius: 12px;
    box-shadow: 0px 6px 12px rgba(0, 0, 0, 0.15);
    padding: 10px;
    border: 1px solid #e5e7eb;
}

/* ✅ Selected Date (Gray Background) */
:deep(.dp__active_date) {
    background-color: #4a4a4b !important;
    /* Dark Gray */
    color: white !important;
    border-radius: 6px;
    font-weight: bold;
    transition: all 0.2s ease-in-out;
}

/* ✅ Today’s Date (Outlined Circle) */
:deep(.dp__today) {
    border: 2px solid #b0b0b3 !important;
    /* Medium Gray */
    border-radius: 50%;
    font-weight: bold;
    color: #909195 !important;
}

/* ✅ Selected Date (Start and End - Fully Rounded) */
:deep(.dp__range_start) {
    background-color: #e3e3f1 !important;
    /* Dark Gray */
    color: white !important;
    border-radius: 50% !important;
    /* Fully rounded */
    font-weight: bold;
}

/* ✅ Selected Date (Start and End - Fully Rounded) */
:deep(.dp__range_end) {
    background-color: #e3e3f1 !important;
    /* Dark Gray */
    color: white !important;
    border-radius: 50% !important;
    /* Fully rounded */
    font-weight: bold;
}

/* ✅ Range Between (Softer Gray Background, No Rounded Borders) */
:deep(.dp__range_between) {
    background-color: rgba(120, 120, 120, 0.2) !important;
    /* Light Gray */
    color: #5a5c64 !important;
    border-radius: 0px !important;
    /* Remove rounding */
}

/* ✅ Hover Effect */
:deep(.dp__range_between:hover) {
    background-color: rgba(90, 90, 90, 0.3) !important;
}


/* ✅ Navigation Arrows */
:deep(.dp__pointer) {
    color: #5a5a5b !important;
    /* Dark Gray */
    transition: transform 0.2s ease-in-out;
}



/* ✅ Calendar Header */
:deep(.dp__month_year_row) {
    font-weight: bold;
    color: #5a5a5b;
    /* Dark Gray */
}

/* ✅ Weekday Headers */
:deep(.dp__weekday_header) {
    font-weight: bold;
    color: #6b6d6f;
    /* Medium Gray */
}

/* ✅ Date Picker Input Box */
.custom-datepicker {

    /* Adjust width to match UI */
    padding: 10px 14px;
    font-size: 8px;

    /* Dark gray background */

    /* Darker border */
    border-radius: 25px;
    /* Rounded corners */
    color: #ffffff;

}



/* Hide scrollbar but keep scrolling functionality */
.scrollbar-hidden::-webkit-scrollbar {
    width: 0px;
    display: none;
}

.scrollbar-hidden {
    scrollbar-width: none;
    /* For Firefox */
}

/* Custom scrollbar styling */
.scrollbar-custom {
    scrollbar-width: thin;
    scrollbar-color: #cbd5e1 #f1f5f9;
}

.scrollbar-custom::-webkit-scrollbar {
    width: 8px;
    height: 8px;
}

.scrollbar-custom::-webkit-scrollbar-track {
    background-color: #f1f5f9;
    border-radius: 4px;
}

.scrollbar-custom::-webkit-scrollbar-thumb {
    background-color: #cbd5e1;
    border-radius: 4px;
    border: 2px solid #f1f5f9;
}

.scrollbar-custom::-webkit-scrollbar-thumb:hover {
    background-color: #94a3b8;
}

.chart-container {
    width: 100%;
    height: 100%;
    /* or set a specific height for your needs */
}

.table-container {
    max-width: 100%;
    overflow-x: auto;
}

table {
    width: 100%;
    font-size: 12px;
    /* Reduced font size */
    border-spacing: 0;
    border-collapse: collapse;
}

th,
td {
    padding: 6px 8px;
    /* Reduced padding */
    text-align: left;
    border: 1px solid #E5E7EB;
}

th {
    font-weight: 600;
    background-color: #F3F4F6;
    text-transform: uppercase;
    color: #374151;
}

tr:hover {
    background-color: #F9FAFB;
}
</style>

<style>
@media (max-width: 1508px) {
    .main-card-grid {
        grid-template-columns: repeat(4, minmax(100px, 1fr)) !important;
    }
}
@media (max-width: 1271px) {
    .main-card-grid {
        grid-template-columns: repeat(3, minmax(100px, 1fr)) !important;
    }
}
@media (max-width: 1046px) {
    .main-card-grid {
        grid-template-columns: repeat(2, minmax(100px, 1fr)) !important;
    }
}
@media (max-width: 802px) {
    .main-card-grid {
        grid-template-columns: repeat(1, minmax(100px, 1fr)) !important;
    }
}
</style>