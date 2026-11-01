<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MediNutrición - Sistema de Registro Clínico y Nutricional | Hospital Moscati</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Google Fonts Inter -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; }
        ::-webkit-scrollbar { width: 6px; height: 6px; }
        ::-webkit-scrollbar-track { background: #f1f5f9; }
        ::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 4px; }
        ::-webkit-scrollbar-thumb:hover { background: #94a3b8; }
    </style>
</head>
<body class="bg-slate-50 text-slate-800 antialiased min-h-screen flex flex-col">

    <!-- Header Principal -->
    <header class="bg-gradient-to-r from-blue-700 to-sky-600 text-white shadow-md sticky top-0 z-30">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-4 flex flex-col sm:flex-row justify-between items-center gap-4">
            <div class="flex items-center space-x-3">
                <div class="bg-white/10 p-2.5 rounded-xl backdrop-blur-sm border border-white/20">
                    <i class="fa-solid fa-hospital-user text-2xl text-white"></i>
                </div>
                <div>
                    <h1 class="text-xl font-bold tracking-tight">MediNutrición Hospitalaria</h1>
                    <p class="text-xs text-blue-100">Control Clínico, Dietético y Tamizaje Moscati (Sync Drive 6:00 a.m.)</p>
                </div>
            </div>
            <div class="flex items-center gap-3 flex-wrap">
                <button onclick="syncDriveData()" class="bg-emerald-600 hover:bg-emerald-500 text-white font-medium px-3.5 py-2.5 rounded-xl border border-emerald-500/50 transition-all flex items-center gap-2 text-sm shadow-sm" title="Sincronizar con Drive (6:00 AM)">
                    <i class="fa-solid fa-cloud-arrow-down"></i> <span class="hidden md:inline">Sincronizar Drive</span>
                </button>
                <button onclick="openPatientModal()" class="bg-white text-blue-700 hover:bg-blue-50 font-semibold px-4 py-2.5 rounded-xl shadow-sm transition-all flex items-center gap-2 text-sm">
                    <i class="fa-solid fa-user-plus"></i> Nuevo Paciente
                </button>
                <button onclick="exportData()" class="bg-blue-600 hover:bg-blue-500 text-white font-medium px-3.5 py-2.5 rounded-xl border border-blue-500/50 transition-all flex items-center gap-2 text-sm" title="Respaldar Datos">
                    <i class="fa-solid fa-download"></i> <span class="hidden md:inline">Exportar JSON</span>
                </button>
            </div>
        </div>
    </header>

    <!-- Main Content Container -->
    <main class="flex-1 max-w-7xl w-full mx-auto px-4 sm:px-6 lg:px-8 py-6 space-y-6">
        
        <!-- Dashboard Metrics / Stats Bar -->
        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
            <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-100 flex items-center justify-between">
                <div>
                    <p class="text-xs font-medium text-slate-500 uppercase tracking-wider">Total Pacientes</p>
                    <h3 id="statTotal" class="text-2xl font-bold text-slate-800 mt-1">0</h3>
                </div>
                <div class="w-12 h-12 bg-blue-50 text-blue-600 rounded-xl flex items-center justify-center text-xl">
                    <i class="fa-solid fa-bed-pulse"></i>
                </div>
            </div>
            <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-100 flex items-center justify-between">
                <div>
                    <p class="text-xs font-medium text-slate-500 uppercase tracking-wider">Consumo Promedio</p>
                    <h3 id="statConsumo" class="text-2xl font-bold text-emerald-600 mt-1">0%</h3>
                </div>
                <div class="w-12 h-12 bg-emerald-50 text-emerald-600 rounded-xl flex items-center justify-center text-xl">
                    <i class="fa-solid fa-utensils"></i>
                </div>
            </div>
            <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-100 flex items-center justify-between">
                <div>
                    <p class="text-xs font-medium text-slate-500 uppercase tracking-wider">Tamiz Pendiente</p>
                    <h3 id="statTamiz" class="text-2xl font-bold text-amber-600 mt-1">0</h3>
                </div>
                <div class="w-12 h-12 bg-amber-50 text-amber-600 rounded-xl flex items-center justify-center text-xl">
                    <i class="fa-solid fa-clipboard-question"></i>
                </div>
            </div>
            <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-100 flex items-center justify-between">
                <div>
                    <p class="text-xs font-medium text-slate-500 uppercase tracking-wider">Con ANCP / Alergias</p>
                    <h3 id="statAncp" class="text-2xl font-bold text-rose-600 mt-1">0</h3>
                </div>
                <div class="w-12 h-12 bg-rose-50 text-rose-600 rounded-xl flex items-center justify-center text-xl">
                    <i class="fa-solid fa-ban"></i>
                </div>
            </div>
        </div>

        <!-- Search and Filter Bar -->
        <div class="bg-white p-4 rounded-2xl shadow-sm border border-slate-100 flex flex-col sm:flex-row items-center justify-between gap-4">
            <div class="relative w-full sm:w-96">
                <span class="absolute inset-y-0 left-0 pl-3.5 flex items-center pointer-events-none text-slate-400">
                    <i class="fa-solid fa-magnifying-glass"></i>
                </span>
                <input type="text" id="searchInput" oninput="filterPatients()" placeholder="Buscar por paciente, dieta o diagnóstico..." 
                    class="w-full pl-10 pr-4 py-2.5 bg-slate-50 border border-slate-200 rounded-xl text-sm focus:outline-none focus:ring-2 focus:ring-blue-500/20 focus:border-blue-600 transition-all">
            </div>
            <div class="flex items-center gap-3 w-full sm:w-auto justify-end flex-wrap">
                <select id="filterDieta" onchange="filterPatients()" class="bg-slate-50 border border-slate-200 rounded-xl px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-blue-500/20">
                    <option value="all">Todas las Dietas</option>
                    <option value="Normal">Dieta Normal</option>
                    <option value="Blanda">Dieta Blanda</option>
                    <option value="Liquida">Dieta Líquida</option>
                    <option value="Restriccion">Restricción / Especial</option>
                </select>
                <select id="filterTamiz" onchange="filterPatients()" class="bg-slate-50 border border-slate-200 rounded-xl px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-blue-500/20">
                    <option value="all">Estatus Tamiz (Todos)</option>
                    <option value="Evaluado">Ya evaluado</option>
                    <option value="Pendiente">Pendiente de evaluar</option>
                </select>
            </div>
        </div>

        <!-- Patients Table Card -->
        <div class="bg-white rounded-2xl shadow-sm border border-slate-100 overflow-hidden">
            <div class="px-6 py-4 border-b border-slate-100 flex justify-between items-center bg-slate-50/50">
                <h2 class="font-bold text-slate-800 text-base flex items-center gap-2">
                    <i class="fa-solid fa-list-check text-blue-600"></i> Directorio de Pacientes y Tamizaje Moscati (Actualizado Diario 6:00 AM)
                </h2>
                <span id="patientCountBadge" class="text-xs bg-blue-100 text-blue-700 font-semibold px-2.5 py-1 rounded-full">0 registros</span>
            </div>
            <div class="overflow-x-auto">
                <table class="w-full text-left border-collapse">
                    <thead>
                        <tr class="bg-slate-50 text-slate-500 text-xs font-semibold uppercase tracking-wider border-b border-slate-100">
                            <th class="py-3 px-4">Paciente / Edad / Sexo</th>
                            <th class="py-3 px-4">Antropometría (Peso/Talla/IMC)</th>
                            <th class="py-3 px-4">Diagnóstico & Dx Nutricio</th>
                            <th class="py-3 px-4">Tipo de Dieta & Consumo</th>
                            <th class="py-3 px-4">Estatus Tamiz</th>
                            <th class="py-3 px-4 text-center">Acciones y Descarga</th>
                        </tr>
                    </thead>
                    <tbody id="patientTableBody" class="divide-y divide-slate-100 text-sm text-slate-700">
                        <!-- Populated via JavaScript -->
                    </tbody>
                </table>
            </div>
            <!-- Empty State -->
            <div id="emptyState" class="hidden py-16 text-center">
                <div class="w-16 h-16 bg-blue-50 text-blue-400 rounded-full flex items-center justify-center mx-auto mb-3 text-2xl">
                    <i class="fa-solid fa-folder-open"></i>
                </div>
                <h3 class="text-slate-700 font-semibold text-base">No hay pacientes registrados</h3>
                <p class="text-slate-400 text-sm mt-1">Sincroniza desde Drive o agrega un nuevo expediente manualmente.</p>
            </div>
        </div>
    </main>

    <!-- Modal de Registro / Edición con Todos los Campos -->
    
