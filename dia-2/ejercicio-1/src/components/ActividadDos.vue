<script setup>
import { computed, reactive, ref, toRefs, watch } from 'vue';

const catalogo = [
    { id: 1, nombre: 'Teclado mecánico', precio: 29900 },
    { id: 2, nombre: 'Mouse inalámbrico', precio: 15900 },
    { id: 3, nombre: 'Audífonos', precio: 24900 },
    { id: 4, nombre: 'Webcam HD', precio: 19900 },
];

const pedido = reactive({
    items: [],
    cliente: { nombre: '', email: '' },
    envioExpress: false
});


const paso = ref(1); // 1 = carrito, 2 = datos, 3 = confirmación
const enviando = ref(false);
const exito = ref(false);

function agregar({ id, nombre, precio }) {
    const enCarrito = pedido.items.find(i => i.id === id);
    if (enCarrito) enCarrito.cantidad++;
    else pedido.items.push({ id, nombre, precio, cantidad: 1 })
}
function quitar(id) {
    const indice = pedido.items.find(i => i.id === id);
    if (indice != -1) pedido.items.splice(indice, 1);
}


const cantidadTotal = computed(() =>
    pedido.items.reduce((acc, val) => acc + val.cantidad, 0)
);
const subtotal = computed(() =>
    pedido.items.reduce((acc, i) => acc + i.precio * i.cantidad, 0)
);
const despacho = computed(() => {
    if (subtotal.value >= 30000) return 0;
    return pedido.envioExpress ? 5990 : 2990;
});

const total = computed(() => subtotal.value + despacho.value);

const emailValido = computed(() => /^[^@]+@[^@]+\.[^@]+$/.test(pedido.cliente.email));
const datosOk = computed(() => pedido.cliente.nombre.trim() !== '' && emailValido.value);


const aviso = ref('');

watch(subtotal, (valN, valV) => {
    console.log(valN, valV);
    
    if (valN >= 30000) {
        aviso.value = '🎉 ¡Alcanzaste envío gratis!';
    } else {
        aviso.value = `Te faltan $${(30000 - valN).toLocaleString('es-CL')} para el envío gratis`;
    }
});


function siguiente() { if (paso.value < 3) paso.value++; }
function volver() { if (paso.value > 1) paso.value--; }

function confirmar() {
    enviando.value = true;
    setTimeout(() => {
        enviando.value = false;
        exito.value = true;
    }, 1200);
}
function reiniciar() {
    pedido.items = [];
    pedido.cliente.nombre = '';
    pedido.cliente.email = '';
    pedido.envioExpress = false;
    paso.value = 1;
    exito.value = false;
    aviso.value = '';
}

</script>

<template>
    <div class="wrap">
        <ol class="steps">
            <li :class="{ on: paso >= 1 }">Carrito</li>
            <li :class="{ on: paso >= 2 }">Datos</li>
            <li :class="{ on: paso >= 3 }">Confirmar</li>
        </ol>

        <div v-if="exito" class="ok">
            ✅ ¡Gracias {{ pedido.cliente.nombre }}! Tu pedido de
            <b>{{ cantidadTotal }}</b> producto(s) por
            <b>${{ total.toLocaleString('es-CL') }}</b> fue registrado.
            <button class="btn ghost" @click="reiniciar">Nuevo pedido</button>
        </div>
        <div v-else>
            <section v-if="paso === 1">
                <h4>Catálogo</h4>
                <div class="cat">
                    <div v-for="p in catalogo" :key="p.id" class="prod">
                        <span>{{ p.nombre }}</span>
                        <b>${{ p.precio.toLocaleString('es-CL') }}</b>
                        <button class="btn sm" @click="agregar(p)">+ Agregar</button>
                    </div>
                </div>
                <h4>Tu carrito ({{ cantidadTotal }})</h4>
                <p v-if="pedido.items.length === 0" class="muted">Aún no agregas productos.</p>
                <ul v-else class="cart">
                    <li v-for="i in pedido.items" :key="i.id">
                        {{ i.nombre }} × {{ i.cantidad }}
                        <span>${{ (i.precio * i.cantidad).toLocaleString('es-CL') }}</span>
                        <button class="link" @click="quitar(i.id)">quitar</button>
                    </li>
                </ul>
                <label class="check">
                    <input type="checkbox" v-model="pedido.envioExpress" />
                    Envío express (+$3.000 aprox.)
                </label>
                <p v-if="aviso" class="aviso">{{ aviso }}</p>
            </section>
            <section v-if="paso === 2">
                <h4>Tus datos</h4>
                <label>Nombre *</label>
                <input v-model.trim="pedido.cliente.nombre" type="text" placeholder="Tu nombre" />
                <label>Email *</label>
                <input v-model.trim="pedido.cliente.email" type="email" placeholder="tucorreo@mail.com" />
                <small v-if="pedido.cliente.email && !emailValido" class="err">Email inválido</small>
            </section>
            <section v-if="paso === 3">
                <h4>Revisa tu pedido</h4>
                <ul class="cart">
                    <li v-for="i in pedido.items" :key="i.id">
                        {{ i.nombre }} × {{ i.cantidad }}
                        <span>${{ (i.precio * i.cantidad).toLocaleString('es-CL') }}</span>
                    </li>
                </ul>
                <p>Cliente: <b>{{ pedido.cliente.nombre }}</b> · {{ pedido.cliente.email }}</p>
            </section>

            <div class="resumen">
                <div>Subtotal <b>${{ subtotal.toLocaleString('es-CL') }}</b></div>
                <div>Despacho <b>{{ despacho === 0 ? 'GRATIS' : '$' + despacho.toLocaleString('es-CL') }}</b></div>
                <div class="tot">Total <b>${{ total.toLocaleString('es-CL') }}</b></div>
            </div>
            <div class="nav">
                <button class="btn ghost" @click="volver" :disabled="paso === 1">← Volver</button>

                <button v-if="paso === 1" class="btn" @click="siguiente" :disabled="pedido.items.length === 0">
                    Continuar →
                </button>
                <button v-else-if="paso === 2" class="btn" @click="siguiente" :disabled="!datosOk">
                    Continuar →
                </button>
                <button v-else class="btn" @click="confirmar" :disabled="enviando">
                    {{ enviando ? 'Enviando…' : 'Confirmar compra ✔' }}
                </button>
            </div>
        </div>

    </div>
</template>


<style scoped>
.wrap {
    font-family: system-ui, sans-serif;
    color: #1e293b;
    max-width: 620px;
}

h3 {
    color: #35495e;
}

h4 {
    color: #35495e;
    margin: 1.2rem 0 .5rem;
}

.steps {
    list-style: none;
    display: flex;
    gap: .5rem;
    padding: 0;
}

.steps li {
    flex: 1;
    text-align: center;
    padding: .4rem;
    border-radius: 999px;
    background: #e2e8f0;
    color: #64748b;
    font-size: .85rem;
    font-weight: 600;
}

.steps li.on {
    background: #42b883;
    color: #0f172a;
}

.cat {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: .6rem;
}

.prod {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: .5rem;
    background: #f8fafc;
    border: 1px solid #e2e8f0;
    border-radius: 8px;
    padding: .5rem .7rem;
    font-size: .9rem;
}

.cart {
    list-style: none;
    padding: 0;
}

.cart li {
    display: flex;
    align-items: center;
    gap: .6rem;
    padding: .4rem 0;
    border-bottom: 1px solid #e2e8f0;
}

.cart li span {
    margin-left: auto;
    font-weight: 700;
}

.link {
    background: none;
    border: none;
    color: #f87171;
    cursor: pointer;
    text-decoration: underline;
}

label {
    display: block;
    font-weight: 600;
    margin: .6rem 0 .25rem;
    color: #334155;
}

input[type="text"],
input[type="email"] {
    width: 100%;
    padding: .55rem .7rem;
    border: 1px solid #cbd5e1;
    border-radius: 8px;
}

.check {
    display: flex;
    align-items: center;
    gap: .5rem;
    margin-top: .8rem;
}

.check input {
    width: 18px;
    height: 18px;
    accent-color: #42b883;
}

.err {
    color: #dc2626;
}

.aviso {
    background: rgba(56, 189, 248, .12);
    border-left: 4px solid #38bdf8;
    padding: .5rem .8rem;
    border-radius: 0 8px 8px 0;
}

.resumen {
    background: #0f172a;
    color: #e2e8f0;
    border-radius: 10px;
    padding: .8rem 1rem;
    margin: 1rem 0;
    display: grid;
    gap: .3rem;
}

.resumen .tot {
    border-top: 1px solid #334155;
    margin-top: .3rem;
    padding-top: .4rem;
    font-size: 1.1rem;
}

.resumen b {
    color: #86efac;
    float: right;
}

.nav {
    display: flex;
    justify-content: space-between;
    gap: 1rem;
}

.ok {
    background: #dcfce7;
    color: #166534;
    padding: 1rem;
    border-radius: 10px;
    font-weight: 600;
}

.muted {
    color: #64748b;
}

.btn {
    background: #42b883;
    color: #0f172a;
    border: none;
    padding: .6rem 1.2rem;
    border-radius: 8px;
    font-weight: 700;
    cursor: pointer;
}

.btn.sm {
    padding: .3rem .7rem;
    font-size: .8rem;
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
    border-color: #cbd5e1;
}
</style>