<script setup>
import { computed, reactive, watch, ref } from 'vue';


const form = reactive({
    nombre: '',
    email: '',
    password: '',
    confirmar: ''
});
const enviado = ref(false);

const emailValido = computed(() => /^[^@]+@[^@]+\.[^@]+$/.test(form.email));

const fortalezaClave = computed(() => {
    const pass = form.password;
    let puntos = 0;
    if (pass.length >= 8) puntos++;
    if (/[A-Z]/.test(pass)) puntos++;
    if (/[0-9]/.test(pass)) puntos++;
    if (/[^A-Za-z0-9]/.test(pass)) puntos++;
    return puntos;
});

const nivelFortaleza = computed(() => {
    const pass = form.password;
    if (pass.length === 0) return { texto: '—', clase: '', ancho: 0 };
    if (fortalezaClave.value <= 1) return { texto: 'Débil', clase: 'debil', ancho: 33 };
    if (fortalezaClave.value <= 3) return { texto: 'Media', clase: 'media', ancho: 66 };
    return { texto: 'Fuerte', clase: 'fuerte', ancho: 100 };
});

const coincide = computed(() =>
    form.confirmar.length > 0 && form.password === form.confirmar
);

const formularioValido = computed(() =>
    form.nombre.trim() !== '' &&
    emailValido.value &&
    fuerza.value >= 3 &&
    coincide.value
);

const consejo = ref('');


watch(() => form.password, (nueva) => {
    const faltan = [];
    if (nueva.length < 8) faltan.push('8+ caracteres');
    if (!/[A-Z]/.test(nueva)) faltan.push('una mayúscula');
    if (!/[0-9]/.test(nueva)) faltan.push('un número');
    if (!/[^A-Za-z0-9]/.test(nueva)) faltan.push('un símbolo');
    consejo.value = faltan.length ? `Agrega: ${faltan.join(', ')}` : '¡Contraseña robusta! 💪';
});

function registrar() {
    if (!formularioValido.value) return;
    enviado.value = true;
}

</script>


<template>
    <div class="wrap">
        <h3>2 · 🔐 Registro con validación en vivo</h3>

        <div v-if="enviado" class="ok">
            ✅ ¡Cuenta creada, {{ form.nombre }}! Te enviamos un correo a {{ form.email }}.
        </div>

        <form v-else @submit.prevent="registrar" class="panel">
            <label>Nombre *</label>
            <input type="text" v-model.trim="form.nombre" placeholder="Tu nombre" />

            <label>Email *</label>
            <input type="email" v-model.trim="form.email" placeholder="tucorreo@mail.com" />
            <small v-if="form.email && !emailValido" class="err">Email inválido</small>

            <label>Contraseña *</label>
            <input type="password" v-model="form.password" placeholder="Crea una contraseña" />
            <!-- Medidor de fuerza (computed nivelFuerza) -->
            <div class="meter" v-if="form.password">
                <div class="track">
                    <div class="fill" :class="nivelFortaleza.clase" :style="{ width: nivelFortaleza.ancho + '%' }"></div>
                </div>
                <span class="lbl" :class="nivelFortaleza.clase">{{ nivelFortaleza.texto }}</span>
            </div>
            <small v-if="consejo && form.password" class="consejo">{{ consejo }}</small>

            <label>Repetir contraseña *</label>
            <input type="password" v-model="form.confirmar" placeholder="Repite la contraseña" />
            <small v-if="form.confirmar && !coincide" class="err">Las contraseñas no coinciden</small>

            <button class="btn" type="submit" :disabled="!formularioValido">
                {{ formularioValido ? 'Crear cuenta ✔' : 'Completa el formulario' }}
            </button>
        </form>
    </div>

</template>



<style scoped>
.wrap {
    font-family: system-ui, sans-serif;
    color: #1e293b;
    max-width: 460px;
}

h3 {
    color: #35495e;
}

.panel {
    background: #f8fafc;
    border: 1px solid #e2e8f0;
    border-radius: 10px;
    padding: 1.25rem 1.5rem;
}

label {
    display: block;
    font-weight: 600;
    margin: .7rem 0 .25rem;
    color: #334155;
    font-size: .9rem;
}

input {
    width: 100%;
    padding: .55rem .7rem;
    border: 1px solid #cbd5e1;
    border-radius: 8px;
    font-size: .95rem;
}

input:focus {
    outline: none;
    border-color: #42b883;
    box-shadow: 0 0 0 3px rgba(66, 184, 131, .25);
}

.err {
    color: #dc2626;
    font-size: .8rem;
}

.meter {
    display: flex;
    align-items: center;
    gap: .6rem;
    margin-top: .5rem;
}

.track {
    flex: 1;
    background: #e2e8f0;
    border-radius: 999px;
    height: 7px;
    overflow: hidden;
}

.fill {
    height: 100%;
    transition: width .3s, background .3s;
}

.fill.debil {
    background: #f87171;
}

.fill.media {
    background: #fbbf24;
}

.fill.fuerte {
    background: #42b883;
}

.lbl {
    font-size: .78rem;
    font-weight: 700;
    width: 48px;
}

.lbl.debil {
    color: #dc2626;
}

.lbl.media {
    color: #d97706;
}

.lbl.fuerte {
    color: #16a34a;
}

.consejo {
    display: block;
    color: #64748b;
    font-size: .8rem;
    margin-top: .3rem;
}

.btn {
    background: #42b883;
    color: #0f172a;
    border: none;
    padding: .6rem 1.2rem;
    border-radius: 8px;
    font-weight: 700;
    cursor: pointer;
    margin-top: 1rem;
    width: 100%;
}

.btn:disabled {
    background: #cbd5e1;
    color: #64748b;
    cursor: not-allowed;
}

.ok {
    background: #dcfce7;
    color: #166534;
    padding: 1rem;
    border-radius: 10px;
    font-weight: 600;
}
</style>
