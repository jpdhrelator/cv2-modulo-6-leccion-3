<script setup>
import { computed, reactive, ref, toRefs, watch} from 'vue';

const nombre= ref('Camila');
const rol=ref('Frontend');
const nivel= ref(3);
const bioRef= computed(()=>`${nombre.value}, ${rol.value} (nivel ${nivel.value})`);

const perfil = reactive({
    nombre:'Pablo',
    rol:'Backend',
    nivel:2
});
const bioReactive= computed(()=> `${perfil.nombre}, ${perfil.rol} (nivel ${perfil.nivel})`);

const otroObjetoCualquiera= reactive({
    contador:0
});
const {contador}=toRefs(otroObjetoCualquiera);

const {rol:rolConectado}= toRefs(perfil);
//const rolConectado=ref();
function subirNivel() {
   perfil.nivel++;
   nivel.value++ 
}

setInterval(()=>otroObjetoCualquiera.contador= Math.floor(Math.random() * 100),1000);


const bitacora= ref([]);
watch(nivel,(valorNuevo,valorViejo)=>{
    bitacora.value.unshift(`🅰️ ref: nivel ${valorViejo} → ${valorNuevo}`);
});

watch(perfil,(valorNuevo,valorViejo)=>{
     bitacora.value.unshift(`🅱️ reactive: nivel ${valorViejo.nivel} → ${valorNuevo.nivel}`);
    if (valorNuevo.nivel === 5) bitacora.value.unshift('🏆 ¡Nivel máximo alcanzado!');
});

</script>

<template>
    <div class="wrap">
        <h3>1 · <code>ref</code> vs <code>reactive</code> — el mismo estado, dos formas</h3>
        <div class="cols">
            <section class="panel">
                <h4>🅰️ Con <code>ref</code></h4>
                <label>Nombre</label>
                <input type="text" v-model="nombre">
                <label>Rol</label>
                <input type="text" v-model="rol">
                <label>Nivel {{ nivel }}</label>
                <input  type="range" min="1" max="5" v-model.number="nivel">
                <p class="bio"> {{ bioRef }}</p>
            </section>

            <section class="panel">
                <h4>🅱️ Con <code>reactive</code></h4>
                <label>Nombre</label>
                <input type="text" v-model="perfil.nombre">
                <label>Rol</label>
                <input type="text" v-model="perfil.rol">
                <label>Nivel {{ perfil.nivel }}</label>
                <input  type="range" min="1" max="5" v-model.number="perfil.nivel">
                <p class="bio"> {{ bioReactive }}</p>
            </section>


            <button @click="subirNivel" class="btn" type="button"> Subir de Nivel</button>
<br>
            
        </div>
    </div>
    <div class="gotcha">
      <strong>⚠️ toRefs en acción:</strong>
      escribe en el "Rol" de la columna 🅱️ y mira cómo este valor,
      que salió de <code>toRefs(perfil)</code>, sigue conectado:
      <b>{{ rolConectado }}</b>
    </div>


    <div class="watch-box">
        <strong>⏱️ watch en acción</strong>
        <p v-if="bitacora.length ===0" class="muted">Aún no cambia nada…</p>
        <ul v-else>
            <li v-for="(linea, index) in bitacora" :key="i"> {{ linea }}</li>
        </ul>
    </div>



    <hr>
    <small>Valor Contador desde el Objeto:{{otroObjetoCualquiera.contador}}</small>
    <hr>
    <small>Valor Contador desde la variabl contador:{{contador}}</small>
</template>


<style  scoped>
.wrap { font-family: system-ui, sans-serif; color: #1e293b; }
h3 { color: #35495e; }
code { background: #0b1220; color: #42b883; padding: .1rem .35rem; border-radius: 5px; font-size: .9em; }
.cols { display: flex; gap: 1rem; flex-wrap: wrap; }
.panel {
  flex: 1; min-width: 240px; background: #f8fafc; border: 2px dashed #cbd5e1;
  border-radius: 10px; padding: 1rem 1.25rem;
}
.panel.primary { border-color: #42b883; }
.panel h4 { margin-top: 0; }
label { display: block; font-weight: 600; margin: .6rem 0 .25rem; color: #334155; }
input[type="text"] { width: 100%; padding: .5rem .7rem; border: 1px solid #cbd5e1; border-radius: 8px; }
input[type="range"] { width: 100%; accent-color: #42b883; }
.bio { background: #0f172a; color: #86efac; padding: .5rem .7rem; border-radius: 8px; font-family: monospace; }
.muted { color: #64748b; }
.gotcha { background: rgba(251,191,36,.15); border-left: 4px solid #fbbf24; padding: .8rem 1rem; border-radius: 0 8px 8px 0; margin: 1rem 0; }
.gotcha b { color: #b45309; }
.btn { background: #42b883; color: #0f172a; border: none; padding: .6rem 1.2rem; border-radius: 8px; font-weight: 700; cursor: pointer; }
.watch-box { background: #0f172a; color: #e2e8f0; border-radius: 10px; padding: .8rem 1rem; margin-top: 1rem; }
.watch-box strong { color: #38bdf8; }
.watch-box small { color: #94a3b8; }
.watch-box ul { list-style: none; padding: 0; margin: .6rem 0 0; max-height: 140px; overflow-y: auto; font-family: monospace; font-size: .82rem; }
.watch-box li { padding: .15rem 0; border-bottom: 1px solid #1e293b; color: #86efac; }
.watch-box .muted { color: #64748b; }
</style>