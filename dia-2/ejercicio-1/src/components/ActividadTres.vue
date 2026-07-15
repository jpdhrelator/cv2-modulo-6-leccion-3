<script setup>
import { computed, reactive, ref, toRefs, watch } from 'vue';
const CLAVE_STORAGE = 'talentotasks';

function useFiltros() {
    const filtros = reactive({
        busqueda: '',
        estado: 'todas',       // todas | pendientes | completadas
        prioridad: 'todas',    // todas | alta | media | baja
    });

    function limpiarFiltros() {
        filtros.busqueda = '';
        filtros.estado = 'todas';
        filtros.prioridad = 'todas';
    }
    return { ...toRefs(filtros), limpiarFiltros };
}
const { busqueda, estado, prioridad, limpiarFiltros } = useFiltros();

const guardado = localStorage.getItem(CLAVE_STORAGE);
const tareas = ref(
    guardado ? JSON.parse(guardado)
        : [
            { id: 1, titulo: 'Diseñar el mockup del login', responsable: 'Ana', prioridad: 'alta', completada: false },
            { id: 2, titulo: 'Configurar el proyecto Vite', responsable: 'Beto', prioridad: 'media', completada: true },
        ]
);

const nuevaTarea = reactive({
    titulo: '',
    responsable: '',
    prioridad: 'media',
});

const tareasFiltradas = computed(() =>
    tareas.value.filter(t => {
        const coincideTexto =
            t.titulo.toLowerCase().includes(busqueda.value.toLowerCase()) ||
            t.responsable.toLowerCase().includes(busqueda.value.toLowerCase());
        const coincideEstado =
            estado.value === 'todas' ||
            (estado.value === 'completadas' && t.completada) ||
            (estado.value === 'pendientes' && !t.completada);
        const coincidePrioridad =
            prioridad.value === 'todas' || t.prioridad === prioridad.value;
        return coincideTexto && coincideEstado && coincidePrioridad;
    })

);

const stats = computed(() => {
    const total = tareas.value.length;
    const completadas = tareas.value.filter(t => t.completada).length;
    const pendientes = total - completadas;
    const progreso = total === 0 ? 0 : Math.round((completadas / total) * 100);
    return { total, completadas, pendientes, progreso };
});

const formularioValido = computed(() => nuevaTarea.titulo.trim() !== '');

function agregarTarea() {
    if (!formularioValido.value) return;
    tareas.value.push({
        id: Date.now(),
        titulo: nuevaTarea.titulo.trim(),
        responsable: nuevaTarea.responsable.trim() || 'Sin asignar',
        prioridad: nuevaTarea.prioridad,
        completada: false,
    });
    // Reseteamos el formulario (mutamos las propiedades del reactive)
    nuevaTarea.titulo = '';
    nuevaTarea.responsable = '';
    nuevaTarea.prioridad = 'todas';
}
function alternarCompletada(id) {
    const t = tareas.value.find(x => x.id === id);
    if (t) t.completada = !t.completada;
}

function eliminarTarea(id) {
    tareas.value = tareas.value.filter(x => x.id !== id);
}

watch(tareas,
    (lista) => localStorage.setItem(CLAVE_STORAGE, JSON.stringify(lista)),
    { deep: true }
);

</script>

<template>
    <div class="wrap">
        <h3>3 · TalentoTasks — panel de tareas del equipo</h3>
        <!-- ===== TABLERO DE MÉTRICAS (computed stats) ===== -->
        <div class="metrics">
            <div class="metric"><span>{{ stats.total }}</span>Total</div>
            <div class="metric"><span>{{ stats.pendientes }}</span>Pendientes</div>
            <div class="metric"><span>{{ stats.completadas }}</span>Completadas</div>
            <div class="metric prog">
                <div class="bar">
                    <div class="fill" :style="{ width: stats.progreso + '%' }"></div>
                </div>
                {{ stats.progreso }}% avance
            </div>
        </div>

        <div class="cols">
            <section class="panel">
                <h4>➕ Nueva tarea</h4>
                <label>Título *</label>
                <input v-model.trim="nuevaTarea.titulo" type="text" placeholder="¿Qué hay que hacer?"
                    @keyup.enter="agregarTarea" />
                <label>Responsable</label>
                <input v-model.trim="nuevaTarea.responsable" type="text" placeholder="¿Quién lo toma?" />
                <label>Prioridad</label>
                <select v-model="nuevaTarea.prioridad">
                    <option value="alta">🔴 Alta</option>
                    <option value="media">🟡 Media</option>
                    <option value="baja">🟢 Baja</option>
                </select>
                <button class="btn" :disabled="!formularioValido" @click="agregarTarea">
                    {{ formularioValido ? 'Agregar tarea' : 'Escribe un título' }}
                </button>
            </section>
            <section class="panel wide">
                <div class="filtros">
                    <input v-model="busqueda" type="text" placeholder="🔍 Buscar por título o responsable…" />
                    <select v-model="estado">
                        <option value="todas">Todas</option>
                        <option value="pendientes">Pendientes</option>
                        <option value="completadas">Completadas</option>
                    </select>
                    <select v-model="prioridad">
                        <option value="todas">Toda prioridad</option>
                        <option value="alta">🔴 Alta</option>
                        <option value="media">🟡 Media</option>
                        <option value="baja">🟢 Baja</option>
                    </select>
                    <button class="btn ghost sm" @click="limpiarFiltros">Limpiar</button>
                </div>

                <p v-if="tareasFiltradas.length === 0" class="empty">
                    No hay tareas que coincidan. ¿Creamos una? 👈
                </p>

                <ul class="lista">
                    <li v-for="t in tareasFiltradas" :key="t.id" :class="{ done: t.completada }">
                        <input type="checkbox" :checked="t.completada" @change="alternarCompletada(t.id)" />
                        <span class="prio" :class="t.prioridad"></span>
                        <div class="txt">
                            <b>{{ t.titulo }}</b>
                            <small>{{ t.responsable }}</small>
                        </div>
                        <button class="link" @click="eliminarTarea(t.id)">✕</button>
                    </li>
                </ul>
            </section>
        </div>
    </div>
</template>


<style scoped>
.wrap {
    font-family: system-ui, sans-serif;
    color: #1e293b;
    max-width: 760px;
}

h3 {
    color: #35495e;
}

h4 {
    color: #35495e;
    margin: 0 0 .6rem;
}

code {
    background: #0b1220;
    color: #42b883;
    padding: .1rem .35rem;
    border-radius: 5px;
    font-size: .88em;
}

/* métricas */
.metrics {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: .6rem;
    margin: 1rem 0;
}

.metric {
    background: #0f172a;
    color: #e2e8f0;
    border-radius: 10px;
    padding: .7rem;
    text-align: center;
    font-size: .78rem;
}

.metric span {
    display: block;
    font-size: 1.5rem;
    font-weight: 800;
    color: #86efac;
}

.metric.prog {
    grid-column: span 1;
    display: flex;
    flex-direction: column;
    justify-content: center;
    gap: .3rem;
}

.bar {
    background: #334155;
    border-radius: 999px;
    height: 8px;
    overflow: hidden;
}

.fill {
    background: #42b883;
    height: 100%;
    transition: width .3s;
}

.cols {
    display: flex;
    gap: 1rem;
    flex-wrap: wrap;
}

.panel {
    flex: 1;
    min-width: 240px;
    background: #f8fafc;
    border: 1px solid #e2e8f0;
    border-radius: 10px;
    padding: 1rem 1.25rem;
}

.panel.wide {
    flex: 2;
    min-width: 300px;
}

label {
    display: block;
    font-weight: 600;
    margin: .5rem 0 .2rem;
    color: #334155;
    font-size: .88rem;
}

input[type="text"],
select {
    width: 100%;
    padding: .5rem .7rem;
    border: 1px solid #cbd5e1;
    border-radius: 8px;
    font-size: .92rem;
    background: #fff;
}

input:focus,
select:focus {
    outline: none;
    border-color: #42b883;
    box-shadow: 0 0 0 3px rgba(66, 184, 131, .25);
}

.filtros {
    display: flex;
    gap: .5rem;
    flex-wrap: wrap;
    margin-bottom: .8rem;
}

.filtros input {
    flex: 2;
    min-width: 160px;
}

.filtros select {
    flex: 1;
    min-width: 120px;
}

.lista {
    list-style: none;
    padding: 0;
    margin: 0;
}

.lista li {
    display: flex;
    align-items: center;
    gap: .6rem;
    padding: .55rem .4rem;
    border-bottom: 1px solid #eef2f7;
}

.lista li.done .txt b {
    text-decoration: line-through;
    color: #94a3b8;
}

.lista input[type="checkbox"] {
    width: 18px;
    height: 18px;
    accent-color: #42b883;
}

.prio {
    width: 10px;
    height: 10px;
    border-radius: 50%;
    flex-shrink: 0;
}

.prio.alta {
    background: #f87171;
}

.prio.media {
    background: #fbbf24;
}

.prio.baja {
    background: #42b883;
}

.txt {
    display: flex;
    flex-direction: column;
    line-height: 1.2;
}

.txt small {
    color: #64748b;
    font-size: .78rem;
}

.link {
    margin-left: auto;
    background: none;
    border: none;
    color: #f87171;
    cursor: pointer;
    font-size: 1rem;
}

.empty {
    color: #64748b;
    font-style: italic;
}

.btn {
    background: #42b883;
    color: #0f172a;
    border: none;
    padding: .55rem 1rem;
    border-radius: 8px;
    font-weight: 700;
    cursor: pointer;
    margin-top: .8rem;
}

.btn.sm {
    padding: .4rem .7rem;
    margin: 0;
    font-size: .82rem;
}

.btn.ghost {
    background: transparent;
    border: 2px solid #42b883;
    color: #35495e;
}

.btn:disabled {
    background: #cbd5e1;
    color: #64748b;
    cursor: not-allowed;
}

.hint {
    color: #64748b;
    font-size: .85rem;
    margin-top: 1rem;
}
</style>