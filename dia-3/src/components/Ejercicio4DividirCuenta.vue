<script setup>
import { computed, reactive, watch, ref } from 'vue';
const cuenta = reactive({
    total: 48000,
    personas: 3,
    propinaPct: 10,
});
const propina = computed(() => cuenta.total * (cuenta.propinaPct / 100));
const totalFinal = computed(() => cuenta.total + propina.value);
const porPersona = computed(() =>
    cuenta.personas > 0 ? totalFinal.value / cuenta.personas : 0
);
const peso = (n) => Math.round(n).toLocaleString('es-CL');

const propinasSugeridas = [0, 10, 15, 20];

const nota = ref('');

watch(porPersona, (monto) => {
    if (monto > 25000) nota.value = '😅 Salió cara la salida… ¿sumamos a alguien más?';
    else if (monto > 0) nota.value = '👍 Reparto razonable.';
    else nota.value = '';
}, { immediate: true });

function ajustarPersonas(delta) {
    const n = cuenta.personas + delta;
    if (n >= 1) cuenta.personas = n;
}
</script>


<template>
    <div class="wrap">
        <h3>4 · 🍽️ Dividir la cuenta</h3>

        <div class="cols">
            <!-- ===== Controles ===== -->
            <section class="panel">
                <label>Total de la boleta</label>
                <div class="money">
                    <span>$</span>
                    <input type="number" min="0" step="1000" v-model.number="cuenta.total" />
                </div>

                <label>¿Cuántos son?</label>
                <div class="stepper">
                    <button class="btn sm" @click="ajustarPersonas(-1)">−</button>
                    <b>{{ cuenta.personas }}</b>
                    <button class="btn sm" @click="ajustarPersonas(1)">+</button>
                </div>

                <label>Propina: {{ cuenta.propinaPct }}%</label>
                <div class="chips">
                    <button v-for="p in propinasSugeridas" :key="p" class="chip"
                        :class="{ on: cuenta.propinaPct === p }" @click="cuenta.propinaPct = p">{{ p }}%</button>
                </div>
                <input type="range" min="0" max="30" v-model.number="cuenta.propinaPct" />
            </section>

            <!-- ===== Resultado (computed) ===== -->
            <section class="panel result">
                <div class="linea"><span>Subtotal</span><b>${{ peso(cuenta.total) }}</b></div>
                <div class="linea"><span>Propina ({{ cuenta.propinaPct }}%)</span><b class="prop">+${{ peso(propina)
                        }}</b></div>
                <div class="linea total"><span>Total</span><b>${{ peso(totalFinal) }}</b></div>
                <hr />
                <div class="cada">
                    <span>Cada uno paga</span>
                    <strong>${{ peso(porPersona) }}</strong>
                    <small>entre {{ cuenta.personas }} persona(s)</small>
                </div>
                <div class="nota">{{ nota }}</div>
            </section>
        </div>
    </div>

</template>
<style scoped>
.wrap { font-family: system-ui, sans-serif; color: #1e293b; max-width: 700px; }
h3 { color: #35495e; }
.cols { display: flex; gap: 1rem; flex-wrap: wrap; }
.panel { flex: 1; min-width: 250px; background: #f8fafc; border: 1px solid #e2e8f0; border-radius: 10px; padding: 1.25rem 1.5rem; }
label { display: block; font-weight: 600; margin: .9rem 0 .35rem; color: #334155; font-size: .88rem; }
label:first-child { margin-top: 0; }
.money { display: flex; align-items: center; gap: .4rem; }
.money span { font-weight: 700; color: #64748b; }
.money input { flex: 1; padding: .55rem .7rem; border: 1px solid #cbd5e1; border-radius: 8px; font-size: 1.1rem; font-weight: 700; }
input:focus { outline: none; border-color: #42b883; box-shadow: 0 0 0 3px rgba(66,184,131,.25); }
.stepper { display: flex; align-items: center; gap: 1rem; }
.stepper b { font-size: 1.3rem; min-width: 24px; text-align: center; }
.chips { display: flex; gap: .4rem; margin-bottom: .5rem; }
.chip { flex: 1; padding: .4rem; border: 1px solid #cbd5e1; background: #fff; border-radius: 8px; cursor: pointer; font-weight: 600; font-size: .85rem; }
.chip.on { background: #42b883; color: #0f172a; border-color: #42b883; }
input[type="range"] { width: 100%; accent-color: #42b883; }
.result { background: #0f172a; color: #e2e8f0; border-color: #0f172a; }
.linea { display: flex; justify-content: space-between; padding: .35rem 0; font-size: .92rem; }
.linea b { color: #e2e8f0; } .linea .prop { color: #fbbf24; }
.linea.total b { color: #86efac; font-size: 1.1rem; }
hr { border: none; border-top: 1px solid #334155; margin: .6rem 0; }
.cada { text-align: center; padding: .5rem 0; }
.cada span { display: block; color: #94a3b8; font-size: .8rem; text-transform: uppercase; letter-spacing: .5px; }
.cada strong { font-size: 2.2rem; color: #86efac; display: block; }
.cada small { color: #94a3b8; }
.nota { text-align: center; font-size: .82rem; font-weight: 600; color: #fbbf24; margin-top: .5rem; }
.btn.sm { background: #42b883; color: #0f172a; border: none; width: 38px; height: 38px; border-radius: 8px; font-weight: 700; font-size: 1.1rem; cursor: pointer; }
</style>