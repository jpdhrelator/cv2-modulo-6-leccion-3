<script setup>
import { computed, reactive, watch, ref } from 'vue';


const equipos = ['Argentina 🇦🇷', 'Francia 🇫🇷', 'Brasil 🇧🇷', 'España 🇪🇸'];

const partidos = reactive([
    { local: 0, visita: 1, gl: null, gv: null },
    { local: 2, visita: 3, gl: null, gv: null },
    { local: 0, visita: 2, gl: null, gv: null },
    { local: 1, visita: 3, gl: null, gv: null },
    { local: 0, visita: 3, gl: null, gv: null },
    { local: 1, visita: 2, gl: null, gv: null },
]);

const jugado = (p) => Number.isFinite(p.gl) && Number.isFinite(p.gv);

const tabla = computed(() => {
    const filas = equipos.map((equi) => ({ nombre: equi, pj: 0, pg: 0, pe: 0, pp: 0, gf: 0, gc: 0, dg: 0, pts: 0, }))

    for (const p of partidos) {
        if (!jugado(p)) continue;

        const L = filas[p.local];
        const V = filas[p.visita];

        L.pj++;
        V.pj++;
        L.gf += p.gl;
        L.gc += p.gv;

        V.gf += p.gv;
        V.gc += p.gl;

        if (p.gl > p.gv) { L.pg++; L.pts += 3; V.pp++; }
        else if (p.gl < p.gv) { V.pg++; V.pt += 3; L.pp++; }
        else { L.pe++; V.pe++; L.pt += 1; V.pt += 1 }
    }
    filas.forEach((f) => (f.dg = f.gf - f.gc));

    return [...filas].sort((a, b) => b.pts - a.pts || b.dg - a.dg || b.gf - a.gf
    );
});

const partidosJugados = computed(() => partidos.filter((p) => jugado(p)).length);

const anuncio = ref('Ingresa resultados para ver la tabla moverse ⚽');


watch(tabla, (nueva) => {
    if (partidosJugados.value === 6) {
        anuncio.value = `✅ Clasifican: ${nueva[0].nombre} y ${nueva[1].nombre}`
    } else {
        anuncio.value = `Van ${partidosJugados.value}/6 partidos · lidera ${nueva[0].nombre}`;
    }
}, { deep: true });

function simularAzar() {
    partidos.forEach((p, i) => {
        p.gl = (i * 3) % 5;
        p.gv = (i * 2) % 4;
    });
}
function reiniciar() {
    partidos.forEach((p) => { p.gl = null; p.gv = null; });
}



</script>


<template>
    <div class="wrap">
        <h3>1 · 🏆 Mundial 2026 — Simulador de Grupo</h3>

        <div class="anuncio">{{ anuncio }}</div>
        <section class="panel">
            <h4>Resultados del grupo</h4>
            <div class="match" v-for="(p, i) in partidos" :key="i">
                <span class="team">{{ equipos[p.local] }}</span>
                <input type="number" min="0" v-model.number="p.gl" />
                <span class="vs">-</span>
                <input type="number" min="0" v-model.number="p.gv" />
                <span class="team right">{{ equipos[p.visita] }}</span>
            </div>
        </section>
    </div>
    <div class="acciones">
        <button class="btn" @click="simularAzar">Rellenar</button>
        <button class="btn ghost" @click="reiniciar">Reiniciar</button>
    </div>

    <section class="panel wide">
        <h4>Tabla de Posiciones </h4>
        <table>
            <thead>
                <tr>
                    <th>#</th>
                    <th class="l">Equipo</th>
                    <th>PJ</th>
                    <th>PG</th>
                    <th>PE</th>
                    <th>PP</th>
                    <th>GF</th>
                    <th>GC</th>
                    <th>DG</th>
                    <th>Pts</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="(f,i) in tabla" :key="f.nombre" :class="{ clasifica: i < 2 }">
                    <td>{{ i + 1 }}</td>
                    <td class="l">{{ f.nombre }}</td>
                    <td>{{ f.pj }}</td>
                    <td>{{ f.pg }}</td>
                    <td>{{ f.pe }}</td>
                    <td>{{ f.pp }}</td>
                    <td>{{ f.gf }}</td>
                    <td>{{ f.gc }}</td>
                    <td :class="{ pos: f.dg > 0, neg: f.dg < 0 }">{{ f.dg > 0 ? '+' + f.dg : f.dg }}</td>
                    <td>{{ f.pts }}</td>
                </tr>
            </tbody>
        </table>
         <small class="hint">🟩 Los 2 primeros clasifican a octavos. Orden: Puntos → Diferencia de gol → Goles a favor.</small>
    </section>
</template>



<style scoped>
.wrap {
    font-family: system-ui, sans-serif;
    color: #1e293b;
    max-width: 820px;
}

h3 {
    color: #35495e;
}

h4 {
    color: #35495e;
    margin: 0 0 .7rem;
}

.anuncio {
    background: #0f172a;
    color: #86efac;
    padding: .7rem 1rem;
    border-radius: 10px;
    font-weight: 700;
    margin: 1rem 0;
    text-align: center;
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
    padding: 1rem 1.25rem;
}

.panel.wide {
    flex: 1.4;
    min-width: 300px;
}

.match {
    display: flex;
    align-items: center;
    gap: .4rem;
    margin-bottom: .5rem;
}

.team {
    flex: 1;
    font-size: .85rem;
    font-weight: 600;
}

.team.right {
    text-align: right;
}

.vs {
    color: #94a3b8;
}

.match input {
    width: 44px;
    padding: .35rem;
    text-align: center;
    border: 1px solid #cbd5e1;
    border-radius: 6px;
}

.match input:focus {
    outline: none;
    border-color: #42b883;
    box-shadow: 0 0 0 3px rgba(66, 184, 131, .25);
}

.acciones {
    display: flex;
    gap: .5rem;
    margin-top: .8rem;
}

table {
    width: 100%;
    border-collapse: collapse;
    font-size: .8rem;
}

th,
td {
    padding: .35rem .25rem;
    text-align: center;
}

th {
    color: #64748b;
    font-size: .72rem;
    text-transform: uppercase;
    border-bottom: 2px solid #e2e8f0;
}

td {
    border-bottom: 1px solid #eef2f7;
}

.l {
    text-align: left;
}

tr.clasifica {
    background: rgba(66, 184, 131, .12);
}

tr.clasifica td:first-child {
    border-left: 3px solid #42b883;
    font-weight: 800;
}

.pos {
    color: #16a34a;
}

.neg {
    color: #dc2626;
}

.hint {
    display: block;
    color: #64748b;
    margin-top: .6rem;
    font-size: .78rem;
}

.btn {
    background: #42b883;
    color: #0f172a;
    border: none;
    padding: .5rem 1rem;
    border-radius: 8px;
    font-weight: 700;
    cursor: pointer;
    font-size: .85rem;
}

.btn.ghost {
    background: transparent;
    border: 2px solid #42b883;
    color: #35495e;
}
</style>