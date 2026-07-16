<script setup>
import { computed, reactive, watch, ref } from 'vue';

const credito = reactive({
    monto: 5000000,     // pesos
    plazo: 24,          // meses
    tasaAnual: 18,      // % anual
    ingreso: 800000,    // ingreso mensual declarado
});
/* ============================================================
   COMPUTED — matemática financiera derivada de los parámetros.
   Fórmula de cuota (sistema francés / amortización):
     cuota = P · i / (1 − (1 + i)^−n)
   donde i = tasa mensual, n = número de meses.
============================================================ */
const tasaMensual = computed(() => credito.tasaAnual / 12 / 100);

const cuota = computed(() => {
    const i = tasaMensual.value;
    const n = credito.plazo;
    const P = credito.monto;
    if (i === 0) return P / n;                 // sin interés
    return (P * i) / (1 - Math.pow(1 + i, -n));
});
const totalPagar = computed(() => cuota.value * credito.plazo);
const totalInteres = computed(() => totalPagar.value - credito.monto);

const cargaMensual = computed(() =>
    credito.ingreso > 0 ? Math.round((cuota.value / credito.ingreso) * 100) : 0
);

const peso = (n) => Math.round(n).toLocaleString('es-CL');

const alerta = ref('');

watch(cargaMensual, (pct) => {
    if (pct > 40) alerta.value = '🔴 Riesgoso: la cuota supera el 40% de tu ingreso.';
    else if (pct > 30) alerta.value = '🟡 Ajustado: la cuota pasa el 30% recomendado.';
    else alerta.value = '🟢 Saludable: la cuota está dentro de lo recomendado.';
}, { immediate: true });

</script>


<template>
    <div class="wrap">
        <h3>3 · 💰 Simulador de crédito</h3>

        <div class="cols">
            <!-- ===== Controles (v-model.number + range) ===== -->
            <section class="panel">
                <label>Monto del crédito: <b>${{ peso(credito.monto) }}</b></label>
                <input type="range" min="500000" max="30000000" step="500000" v-model.number="credito.monto" />

                <label>Plazo: <b>{{ credito.plazo }} meses</b></label>
                <input type="range" min="6" max="72" step="6" v-model.number="credito.plazo" />

                <label>Tasa anual: <b>{{ credito.tasaAnual }}%</b></label>
                <input type="range" min="0" max="40" step="0.5" v-model.number="credito.tasaAnual" />

                <label>Tu ingreso mensual: <b>${{ peso(credito.ingreso) }}</b></label>
                <input type="range" min="300000" max="3000000" step="50000" v-model.number="credito.ingreso" />
            </section>

            <!-- ===== Resultado (todo computed) ===== -->
            <section class="panel result">
                <div class="cuota">
                    <span>Cuota mensual</span>
                    <strong>${{ peso(cuota) }}</strong>
                </div>
                <div class="linea"><span>Total a pagar</span><b>${{ peso(totalPagar) }}</b></div>
                <div class="linea"><span>Total intereses</span><b class="int">${{ peso(totalInteres) }}</b></div>
                <div class="linea"><span>Cuota vs ingreso</span><b>{{ cargaMensual }}%</b></div>

                <div class="barra">
                    <div class="rel" :style="{ width: Math.min(cargaMensual, 100) + '%' }"></div>
                </div>
                <div class="alerta" :class="{ r: cargaMensual > 40, y: cargaMensual > 30 && cargaMensual <= 40 }">
                    {{ alerta }}
                </div>
            </section>
        </div>
        <small class="hint">Fórmula sistema francés: cuota = P·i / (1 − (1+i)⁻ⁿ). Mueve los sliders y observa cómo todo
            reacciona.</small>
    </div>

</template>


<style scoped>
.wrap {
    font-family: system-ui, sans-serif;
    color: #1e293b;
    max-width: 720px;
}

h3 {
    color: #35495e;
}

.cols {
    display: flex;
    gap: 1rem;
    flex-wrap: wrap;
}

.panel {
    flex: 1;
    min-width: 260px;
    background: #f8fafc;
    border: 1px solid #e2e8f0;
    border-radius: 10px;
    padding: 1.25rem 1.5rem;
}

label {
    display: block;
    font-weight: 600;
    margin: .8rem 0 .3rem;
    color: #334155;
    font-size: .88rem;
}

label:first-child {
    margin-top: 0;
}

label b {
    color: #35495e;
}

input[type="range"] {
    width: 100%;
    accent-color: #42b883;
}

.result {
    background: #0f172a;
    color: #e2e8f0;
    border-color: #0f172a;
}

.cuota {
    text-align: center;
    padding-bottom: .8rem;
    border-bottom: 1px solid #334155;
    margin-bottom: .8rem;
}

.cuota span {
    display: block;
    color: #94a3b8;
    font-size: .8rem;
    text-transform: uppercase;
    letter-spacing: .5px;
}

.cuota strong {
    font-size: 2rem;
    color: #86efac;
}

.linea {
    display: flex;
    justify-content: space-between;
    padding: .35rem 0;
    font-size: .9rem;
}

.linea b {
    color: #e2e8f0;
}

.linea .int {
    color: #fbbf24;
}

.barra {
    background: #334155;
    border-radius: 999px;
    height: 8px;
    overflow: hidden;
    margin: .8rem 0 .5rem;
}

.rel {
    background: #42b883;
    height: 100%;
    transition: width .3s, background .3s;
}

.alerta {
    font-size: .82rem;
    font-weight: 700;
    color: #86efac;
}

.alerta.y {
    color: #fbbf24;
}

.alerta.r {
    color: #f87171;
}

.hint {
    display: block;
    color: #64748b;
    margin-top: .8rem;
    font-size: .78rem;
}
</style>