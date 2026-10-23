<template>
  <div class="dashboard-wrapper">
    <div v-if="usuario" class="dashboard-container">
      
      <nav class="glass-nav">
        <div class="nav-wrapper">
          <div class="brand">
            <img :src="logoImg" alt="Logo" class="app-logo" />
            <span class="brand-text">CAREER<span>TRACKER</span></span>
          </div>
          <div class="user-pill">
            <div class="avatar-container">
              <img v-if="usuario.photoURL" :src="usuario.photoURL" class="user-img" />
              <div v-else class="user-initial">{{ usuario.email.charAt(0).toUpperCase() }}</div>
            </div>
            <span class="user-display-name hide-mobile">{{ nombreCorto }}</span>
            <button @click="cerrarSesion" class="btn-logout" title="Cerrar Sesión">➜</button>
          </div>
        </div>
      </nav>

      <main class="main-content">
        <header class="hero-section">
          <div class="hero-text">
            <h1 class="glitch-title">Hola, {{ nombreCorto }}</h1>
            <p class="hero-subtitle">Registra vacantes y convierte el feedback en oportunidades.</p>
          </div>
          <button class="btn-gradient hide-mobile" @click="abrirModalNuevo">
            <span class="btn-content">+ Nueva Vacante</span>
          </button>
        </header>

        <button class="fab-mobile show-mobile-flex" @click="abrirModalNuevo" title="Nueva Vacante">
          <span class="fab-content">+</span>
        </button>

        <div v-if="misDatos.length > 0" class="action-bar-container">
          <div class="action-bar-glass">
            
            <div class="segmented-control">
              
              <button 
                class="segment-btn" 
                :class="{ active: filtroActivo === 'activas' }" 
                @click="filtroActivo = 'activas'"
                style="display: flex; flex-direction: row; align-items: center; justify-content: center; gap: 6px;"
              >
                <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="#06b6d4" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
                  <polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"></polygon>
                </svg>
                <span>Activas</span>
                <span class="badge-mini" v-if="contadorActivas > 0">{{ contadorActivas }}</span>
              </button>

              <button 
                class="segment-btn" 
                :class="{ active: filtroActivo === 'archivo' }" 
                @click="filtroActivo = 'archivo'"
                style="display: flex; flex-direction: row; align-items: center; justify-content: center; gap: 6px;"
              >
                <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="#8b5cf6" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
                  <polyline points="21 8 21 21 3 21 3 8"></polyline>
                  <rect x="1" y="3" width="22" height="5"></rect>
                  <line x1="10" y1="12" x2="14" y2="12"></line>
                </svg>
                <span>Archivo</span>
                <span class="badge-mini archive-badge" v-if="contadorArchivo > 0">{{ contadorArchivo }}</span>
              </button>

              <button 
                class="segment-btn" 
                :class="{ active: filtroActivo === 'metricas' }" 
                @click="filtroActivo = 'metricas'"
                style="display: flex; flex-direction: row; align-items: center; justify-content: center; gap: 6px;"
              >
                <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="#e879f9" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
                  <line x1="18" y1="20" x2="18" y2="10"></line>
                  <line x1="12" y1="20" x2="12" y2="4"></line>
                  <line x1="6" y1="20" x2="6" y2="14"></line>
                </svg>
                <span>Métricas</span>
              </button>

              <div class="segment-slider" :style="filtroActivo === 'archivo' ? 'transform: translateX(100%)' : filtroActivo === 'metricas' ? 'transform: translateX(200%)' : 'transform: translateX(0)'"></div>
            </div>

            <div class="controls-group" v-show="filtroActivo !== 'metricas'">
              
              <div class="search-wrapper" :class="{ focused: isSearchFocused }">
                <svg class="search-icon" xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
                  <circle cx="11" cy="11" r="8"></circle>
                  <line x1="21" y1="21" x2="16.65" y2="16.65"></line>
                </svg>
                <input 
                  v-model="searchQuery" 
                  type="text" 
                  class="search-input" 
                  placeholder="Buscar empresa o tag..."
                  @focus="isSearchFocused = true"
                  @blur="isSearchFocused = false"
                />
              </div>

              <div class="sort-container">
                <button class="btn-icon-glass" @click.stop="toggleSortMenu" title="Ordenar por">
                  <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <line x1="4" y1="6" x2="20" y2="6"></line>
                    <line x1="10" y1="12" x2="20" y2="12"></line>
                    <line x1="16" y1="18" x2="20" y2="18"></line>
                  </svg>
                </button>
                <transition name="fade-menu">
                  <div v-if="isSortMenuOpen" class="sort-dropdown">
                    <button @click="setSort('recientes')" :class="{ active: ordenActivo === 'recientes' }">Más recientes</button>
                    <button @click="setSort('actualizacion')" :class="{ active: ordenActivo === 'actualizacion' }">Última actualización</button>
                    <button @click="setSort('antiguas')" :class="{ active: ordenActivo === 'antiguas' }">Más antiguas</button>
                  </div>
                </transition>
              </div>

            </div>
          </div>
        </div>

        <InsightsPanel v-if="filtroActivo === 'metricas'" :vacantes="misDatos" />

        <template v-else>
          <section v-if="vacantesFiltradas.length > 0" class="cards-grid">
            <VacancyCard 
              v-for="item in vacantesFiltradas" 
              :key="item.id" 
              :id="item.id" 
              :title="item.nombreVacante" 
              :company="item.empresa" 
              :status="item.estatus || 'Enviada'" 
              :linkCV="item.linkCV" 
              :tipoCV="item.tipoCV"
              :tags="item.tags || []" 
              :fechaUltimaActividad="item.fechaActualizacion?.seconds || item.fechaCreacion?.seconds || 0"
              @update-status="actualizarEstatus"
              @delete-vacancy="eliminarVacante"
              @edit-vacancy="abrirEditor"
              @edit-notes="abrirNotasPanel"
            />
          </section>

          <div v-else-if="isSearching" class="empty-state-pro">
            <div class="glow-icon" style="color: #38bdf8; text-shadow: 0 0 20px rgba(56, 189, 248, 0.5);">🛰️</div>
            <h2 class="welcome-title">Sin resultados en el radar</h2>
            <p class="welcome-subtitle">No encontramos coincidencias para "<strong>{{ searchQuery }}</strong>".</p>
          </div>

          <div v-else-if="misDatos.length > 0 && filtroActivo === 'activas'" class="empty-state-pro">
            <div class="glow-icon">✦</div>
            <h2 class="welcome-title">¡Todo al día!</h2>
            <p class="welcome-subtitle">No tienes procesos activos en este momento.</p>
            <div class="onboarding-box">
              <p><span>1️⃣</span> Da clic en <strong>+ Nueva Vacante</strong> para registrar tu próxima postulación.</p>
              <p><span>2️⃣</span> Si el proceso se detiene, usa el icono <strong>🗃️</strong> para <strong>Mover a Historial</strong>.</p>
            </div>
          </div>

          <div v-else-if="misDatos.length > 0 && filtroActivo === 'archivo'" class="empty-state-pro">
            <div class="glow-icon" style="color: #6b7280; text-shadow: 0 0 20px rgba(107, 114, 128, 0.5);">✧</div>
            <h2>Archivo limpio</h2>
            <p>Aún no tienes vacantes rechazadas o archivadas.</p>
          </div>

          <div v-else class="empty-state-pro">
            <div class="glow-icon">✦</div>
            <h2 class="welcome-title">¡Bienvenida a tu Carrera Tech! 🚀</h2>
            <p class="welcome-subtitle">Tu tablero está limpio, ¡es hora de llenarlo de oportunidades!</p>
            <div class="onboarding-box">
              <p><span>1️⃣</span> Da clic en <strong>+ Nueva Vacante</strong> arriba para agregar tu primera postulación.</p>
              <p><span>2️⃣</span> Si el proceso no avanza, usa el icono <strong>📥</strong> en la tarjeta para <strong>Mover a Historial</strong>.</p>
            </div>
          </div>
        </template>
      </main>

      <div v-if="showModal" class="modal-overlay" @click.self="cerrarModal">
        <div class="modal-glass">
          <h3>{{ isEditing ? 'Editar Oportunidad' : 'Nueva Oportunidad' }}</h3>
          <form @submit.prevent="guardarVacante">
            <div class="form-group">
              <label>Puesto</label>
              <input v-model="nuevaVacante.nombreVacante" required placeholder="Ej. Java Developer" />
            </div>
            <div class="form-group">
              <label>Empresa</label>
              <input v-model="nuevaVacante.empresa" required placeholder="Ej. Google" />
            </div>
            <div class="form-group">
              <label>Estatus</label>
              <select v-model="nuevaVacante.estatus">
                <option>Enviada</option>
                <option>Entrevista</option>
                <option>Prueba</option>
                <option>Oferta</option>
                <option>Rechazada</option>
              </select>
            </div>
            
            <div class="form-group">
              <label>Enlace del CV (Drive, LinkedIn, PDF)</label>
              <input v-model="nuevaVacante.linkCV" placeholder="https://..." />
            </div>

            <div class="form-group">
              <label>Etiquetas / Habilidades (Presiona Enter o Coma)</label>
              <input 
                v-model="tagInput" 
                @keydown="manejarKeydownTag" 
                placeholder="Ej. Vue.js, Inglés B2, Figma..." 
              />
              <div class="tags-preview" v-if="nuevaVacante.tags && nuevaVacante.tags.length > 0">
                <span v-for="(tag, index) in nuevaVacante.tags" :key="index" class="tag-pill-edit">
                  {{ tag }}
                  <button type="button" @click="removerTag(index)" class="btn-remove-tag">×</button>
                </span>
              </div>
            </div>

            <div class="modal-actions">
              <button type="button" class="btn-cancel" @click="cerrarModal">Cancelar</button>
              <button type="submit" class="btn-gradient btn-save-modal">
                <span class="btn-content save-content">
                  {{ isEditing ? 'Actualizar' : 'Guardar' }}
                </span>
              </button>
            </div>
          </form>
        </div>
      </div>

      <ArchiveModal 
        :isOpen="showRechazoModal"
        :vacanteActiva="vacanteParaArchivar"
        @close="cancelarRechazo"
        @confirmar-archivo="ejecutarArchivado"
      />

      <NotesModal 
       v-if="showNotes" 
      :vacante="vacanteSeleccionadaNotas"
       @close="showNotes = false" 
       />

      <transition name="toast-slide">
        <div v-if="toast.show" class="toast-notification" :class="toast.type">
          <div class="toast-icon">
            <svg v-if="toast.type === 'success'" fill="none" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" class="toast-svg">
              <path stroke-linecap="round" stroke-linejoin="round" d="M9 12.75L11.25 15 15 9.75M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
            </svg>
            <svg v-if="toast.type === 'error'" fill="none" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" class="toast-svg">
              <path stroke-linecap="round" stroke-linejoin="round" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
            </svg>
          </div>
          <span class="toast-msg">{{ toast.message }}</span>
        </div>
      </transition>

    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, computed } from 'vue';
import { auth, db } from '../firebase';
import { signOut } from 'firebase/auth'; 
import { 
  collection, query, where, onSnapshot, addDoc, doc, updateDoc, deleteDoc, serverTimestamp 
} from 'firebase/firestore';

import VacancyCard from '../components/VacancyCard.vue';
import ArchiveModal from '../components/ArchiveModal.vue';
import NotesModal from '../components/NotesModal.vue';
import InsightsPanel from '../components/InsightsPanel.vue'; 

import logoImg from '@/assets/careertracker-logo-no-bkg.png';
import { useRouter } from 'vue-router'; 

const router = useRouter();
const usuario = ref(null);
const misDatos = ref([]);
const showModal = ref(false);
const isEditing = ref(false);
const vacanteIdActual = ref(null);
const filtroActivo = ref('activas');

const searchQuery = ref('');

// CIRUGÍA 1: Cambiamos el valor por defecto para que ordene siempre por lo más actualizado
const ordenActivo = ref('actualizacion'); 

const isSearchFocused = ref(false);
const isSortMenuOpen = ref(false);

const toggleSortMenu = () => { isSortMenuOpen.value = !isSortMenuOpen.value; };
const setSort = (type) => { 
  ordenActivo.value = type; 
  isSortMenuOpen.value = false; 
};

const closeMenuListener = (e) => {
  if (!e.target.closest('.sort-container')) {
    isSortMenuOpen.value = false;
  }
};
onMounted(() => window.addEventListener('click', closeMenuListener));
onUnmounted(() => window.removeEventListener('click', closeMenuListener));

const nuevaVacante = ref({ nombreVacante: '', empresa: '', estatus: 'Enviada', linkCV: '', tags: [] });
const tagInput = ref('');

const manejarKeydownTag = (e) => {
  if (e.key === 'Enter' || e.key === ',') {
    e.preventDefault();
    const val = tagInput.value.trim().replace(/,$/, '');
    if (val && !nuevaVacante.value.tags.includes(val)) {
      nuevaVacante.value.tags.push(val);
    }
    tagInput.value = '';
  }
};

const removerTag = (index) => {
  nuevaVacante.value.tags.splice(index, 1);
};

const showRechazoModal = ref(false);
const idPendienteRechazo = ref(null);
const datosPendientesGuardar = ref(null);
const vacanteParaArchivar = ref(null); 

const showNotes = ref(false);
const vacanteSeleccionadaNotas = ref(null);

const toast = ref({ show: false, message: '', type: 'success' });

const mostrarToast = (mensaje, tipo = 'success') => {
  toast.value = { show: true, message: mensaje, type: tipo };
  setTimeout(() => {
    toast.value.show = false;
  }, 3000);
};

const nombreCorto = computed(() => {
  if (usuario.value) {
    if (usuario.value.displayName) return usuario.value.displayName.split(' ')[0];
    return usuario.value.email.split('@')[0];
  }
  return '';
});

const isSearching = computed(() => searchQuery.value.trim().length > 0);

const vacantesFiltradas = computed(() => {
  let baseDatos = misDatos.value;
  if (filtroActivo.value === 'activas') {
    baseDatos = baseDatos.filter(v => v.estatus !== 'Rechazada');
  } else if (filtroActivo.value === 'archivo') {
    baseDatos = baseDatos.filter(v => v.estatus === 'Rechazada');
  }

  const term = searchQuery.value.toLowerCase().trim();
  if (term) {
    baseDatos = baseDatos.filter(v => {
      const puesto = (v.nombreVacante || '').toLowerCase();
      const empresa = (v.empresa || '').toLowerCase();
      const tagsStr = (v.tags || []).join(' ').toLowerCase();
      return puesto.includes(term) || empresa.includes(term) || tagsStr.includes(term);
    });
  }

  // CIRUGÍA 2: Replicamos la inteligencia de Ghosting para que el Tablero la entienda
  const ahoraEnSegundos = Math.floor(Date.now() / 1000);
  const isGhosted = (v) => {
    if (v.estatus === 'Rechazada' || v.estatus === 'Oferta') return false;
    const ultAct = v.fechaActualizacion?.seconds || v.fechaCreacion?.seconds || 0;
    if (!ultAct) return false;
    return ((ahoraEnSegundos - ultAct) / 86400) > 14;
  };

  return [...baseDatos].sort((a, b) => {
    // REGLA MAESTRA: Si una tarjeta está ghosteada, se hunde hasta el fondo de la lista
    const ghostA = isGhosted(a);
    const ghostB = isGhosted(b);
    if (ghostA && !ghostB) return 1;  
    if (!ghostA && ghostB) return -1; 

    // REGLA SECUNDARIA: Ordenamiento normal para las tarjetas vivas
    const getSeconds = (dateField) => dateField?.seconds || (Date.now() / 1000);
    
    const creadoA = getSeconds(a.fechaCreacion);
    const creadoB = getSeconds(b.fechaCreacion);
    
    const actA = a.fechaActualizacion ? getSeconds(a.fechaActualizacion) : creadoA;
    const actB = b.fechaActualizacion ? getSeconds(b.fechaActualizacion) : creadoB;

    if (ordenActivo.value === 'recientes') return creadoB - creadoA;
    if (ordenActivo.value === 'antiguas') return creadoA - creadoB;
    if (ordenActivo.value === 'actualizacion') return actB - actA; // Ahora este es el orden por defecto
    
    return 0;
  });
});

const contadorActivas = computed(() => misDatos.value.filter(v => v.estatus !== 'Rechazada').length);
const contadorArchivo = computed(() => misDatos.value.filter(v => v.estatus === 'Rechazada').length);

const cerrarSesion = async () => {
  if (!navigator.onLine) {
    const confirmarSalida = confirm(
      "⚠️ ADVERTENCIA DE DESCONEXIÓN ⚠️\n\n" +
      "Actualmente no tienes internet. Si cierras sesión ahora:\n" +
      "1. No podrás volver a entrar hasta que recuperes la señal.\n" +
      "2. PERDERÁS los cambios recientes que aún no se han sincronizado a la nube.\n\n" +
      "Recomendamos cancelar y esperar a tener conexión. ¿Deseas salir de todos modos y perder tus cambios?"
    );
    
    if (!confirmarSalida) {
      mostrarToast("Cierre de sesión cancelado. Protegiendo tus datos.", "success");
      return; 
    }
  }

  try {
    await signOut(auth);
    localStorage.removeItem('careerTracker_vacantes_cache');
    router.push('/'); 
  } catch (error) {
    mostrarToast("Error al cerrar sesión", "error");
  }
};

const abrirModalNuevo = () => {
  isEditing.value = false;
  nuevaVacante.value = { nombreVacante: '', empresa: '', estatus: 'Enviada', linkCV: '', tags: [] };
  tagInput.value = '';
  showModal.value = true;
};

const abrirEditor = (datos) => {
  isEditing.value = true;
  vacanteIdActual.value = datos.id;
  nuevaVacante.value = { 
    nombreVacante: datos.title, 
    empresa: datos.company, 
    estatus: datos.status,
    linkCV: datos.linkCV || '',
    tags: datos.tags ? [...datos.tags] : []
  };
  tagInput.value = '';
  showModal.value = true;
};

const cerrarModal = () => {
  showModal.value = false;
  isEditing.value = false;
  vacanteIdActual.value = null;
  nuevaVacante.value = { nombreVacante: '', empresa: '', estatus: 'Enviada', linkCV: '', tags: [] };
  tagInput.value = '';
};

const abrirNotasPanel = (datos) => {
  vacanteSeleccionadaNotas.value = datos;
  showNotes.value = true;
};

const cancelarRechazo = () => {
  showRechazoModal.value = false;
  idPendienteRechazo.value = null;
  datosPendientesGuardar.value = null;
  vacanteParaArchivar.value = null; 
};

const guardarVacante = async () => {
  if (!usuario.value) return;

  let tipoDetectado = 'ninguno';
  if (nuevaVacante.value.linkCV) {
    const url = nuevaVacante.value.linkCV.toLowerCase();
    if (url.includes('drive.google.com')) tipoDetectado = 'drive';
    else if (url.includes('linkedin.com/in/')) tipoDetectado = 'linkedin';
    else if (url.endsWith('.pdf')) tipoDetectado = 'pdf';
    else tipoDetectado = 'otro';
  }

  if (tagInput.value.trim() !== '') {
    nuevaVacante.value.tags.push(tagInput.value.trim());
    tagInput.value = '';
  }

  if (nuevaVacante.value.estatus === 'Rechazada') {
    datosPendientesGuardar.value = {
      isEditing: isEditing.value,
      id: vacanteIdActual.value,
      payload: {
        nombreVacante: nuevaVacante.value.nombreVacante,
        empresa: nuevaVacante.value.empresa,
        estatus: nuevaVacante.value.estatus,
        linkCV: nuevaVacante.value.linkCV,
        tipoCV: tipoDetectado,
        tags: nuevaVacante.value.tags 
      }
    };
    
    let etapaAnterior = 'Enviada';
    if (isEditing.value && vacanteIdActual.value) {
      const v = misDatos.value.find(x => x.id === vacanteIdActual.value);
      if (v) etapaAnterior = v.estatus;
    }
    
    vacanteParaArchivar.value = { etapa: etapaAnterior };
    showModal.value = false;
    showRechazoModal.value = true; 
    return; 
  }

  const datosTemporales = { ...nuevaVacante.value };
  const esEdicion = isEditing.value;
  const idActual = vacanteIdActual.value;
  
  filtroActivo.value = 'activas';
  cerrarModal();

  try {
    if (esEdicion) {
      const docRef = doc(db, "vacantes", idActual);
      await updateDoc(docRef, {
        nombreVacante: datosTemporales.nombreVacante,
        empresa: datosTemporales.empresa,
        estatus: datosTemporales.estatus,
        linkCV: datosTemporales.linkCV,
        tipoCV: tipoDetectado,
        tags: datosTemporales.tags,
        fechaActualizacion: serverTimestamp() 
      });
      mostrarToast("Oportunidad actualizada con éxito");
    } else {
      await addDoc(collection(db, "vacantes"), {
        userId: usuario.value.uid,
        nombreVacante: datosTemporales.nombreVacante,
        empresa: datosTemporales.empresa,
        estatus: datosTemporales.estatus || 'Enviada',
        linkCV: datosTemporales.linkCV,
        tipoCV: tipoDetectado,
        tags: datosTemporales.tags, 
        fechaCreacion: serverTimestamp()
      });
      mostrarToast("Nueva oportunidad registrada");
    }
  } catch (error) {
    console.error("Error al guardar:", error);
    mostrarToast("Guardado localmente. Se sincronizará al recuperar internet.", "success");
  }
};

const actualizarEstatus = async (id, nuevoEstatus) => {
  if (!id) return;

  if (nuevoEstatus === 'Rechazada') {
    idPendienteRechazo.value = id;
    datosPendientesGuardar.value = null;
    
    const vacante = misDatos.value.find(v => v.id === id);
    vacanteParaArchivar.value = vacante ? { etapa: vacante.estatus } : { etapa: 'Enviada' };
    
    showRechazoModal.value = true;
    return; 
  }

  const docRef = doc(db, "vacantes", id);
  try {
    await updateDoc(docRef, { 
      estatus: nuevoEstatus,
      fechaActualizacion: serverTimestamp() 
    });
    mostrarToast(`Estatus actualizado a: ${nuevoEstatus}`);
    if (filtroActivo.value === 'archivo') filtroActivo.value = 'activas';
  } catch (error) {
    console.error("Error al actualizar estatus:", error);
    mostrarToast("Actualización en cola. Esperando red...", "success");
  }
};

const ejecutarArchivado = async (datosDelModal) => {
  try {
    if (datosPendientesGuardar.value) {
      const payloadConMotivo = { 
        ...datosPendientesGuardar.value.payload, 
        motivoRechazo: datosDelModal.motivoRechazo,
        detallesRechazo: datosDelModal.detallesRechazo || null,
        fechaActualizacion: serverTimestamp() 
      };

      if (datosPendientesGuardar.value.isEditing) {
        await updateDoc(doc(db, "vacantes", datosPendientesGuardar.value.id), payloadConMotivo);
      } else {
        payloadConMotivo.userId = usuario.value.uid;
        payloadConMotivo.fechaCreacion = serverTimestamp();
        await addDoc(collection(db, "vacantes"), payloadConMotivo);
      }
    } else {
      await updateDoc(doc(db, "vacantes", idPendienteRechazo.value), { 
        estatus: 'Rechazada',
        motivoRechazo: datosDelModal.motivoRechazo,
        detallesRechazo: datosDelModal.detallesRechazo || null,
        fechaActualizacion: serverTimestamp() 
      });
    }

    mostrarToast("Vacante archivada con análisis inteligente");
    cancelarRechazo(); 
    cerrarModal(); 
    filtroActivo.value = 'archivo'; 

  } catch (error) {
    console.error("Error al archivar:", error);
    mostrarToast("Archivo en cola. Sincronizando...", "success");
  }
};

const eliminarVacante = async (id) => {
  if (confirm("¿Estás seguro de eliminar esta vacante? ¡Se perderán los datos para tu analítica!")) {
    try {
      await deleteDoc(doc(db, "vacantes", id));
      mostrarToast("Vacante eliminada permanentemente");
    } catch (error) {
      console.error("Error al eliminar:", error);
      mostrarToast("Eliminación programada", "success");
    }
  }
};

const cargarCacheOffline = () => {
  try {
    const datosCacheados = localStorage.getItem('careerTracker_vacantes_cache');
    if (datosCacheados) {
      misDatos.value = JSON.parse(datosCacheados);
      console.log("Caché local inyectado con éxito (Modo Offline)");
    }
  } catch (e) {
    console.error("Error al leer la caché local", e);
  }
};

onMounted(() => {
  auth.onAuthStateChanged((user) => {
    if (user) {
      usuario.value = user;
      
      cargarCacheOffline();

      const q = query(collection(db, "vacantes"), where("userId", "==", user.uid));
      onSnapshot(q, (snapshot) => {
        const datosFrescos = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
        misDatos.value = datosFrescos;
        
        try {
          localStorage.setItem('careerTracker_vacantes_cache', JSON.stringify(datosFrescos));
        } catch (e) {
          console.error("No se pudo guardar la caché", e);
        }
      }, (error) => {
        console.error("Operando en modo sin conexión", error);
        mostrarToast("Modo sin conexión activo. Mostrando datos guardados.", "success");
      });
    } else {
      router.push('/'); 
    }
  });
});
</script>

<style scoped>
/* RESET Y FONDO GLOBAL */
:global(body) { margin: 0; padding: 0; background-color: #09090b !important; color: white !important; }
.dashboard-wrapper { min-height: 100vh; width: 100%; background-color: #09090b; color: white; overflow-y: auto; }
.glass-nav { background: rgba(0,0,0,0.4); backdrop-filter: blur(12px); border-bottom: 1px solid rgba(255,255,255,0.08); position: sticky; top: 0; z-index: 1000; padding: 12px 0; }
.nav-wrapper { max-width: 1200px; margin: 0 auto; display: flex; justify-content: space-between; align-items: center; padding: 0 24px; }
.brand { display: flex; align-items: center; gap: 12px; }
.app-logo { height: 35px; }
.brand-text { 
  font-weight: 800; 
  font-size: 1.3rem; 
  background: linear-gradient(90deg, #6366f1, #06b6d4); 
  -webkit-background-clip: text; 
  background-clip: text; 
  color: transparent; 
}
.user-pill { background: rgba(255,255,255,0.05); padding: 4px; border-radius: 30px; display: flex; align-items: center; gap: 8px; border: 1px solid rgba(255,255,255,0.08); }
.user-img, .user-initial { width: 32px; height: 32px; border-radius: 50%; object-fit: cover; }
.user-initial { background: #6366f1; display: flex; align-items: center; justify-content: center; font-weight: bold; }
.btn-logout { background: transparent; border: none; color: #ccc; cursor: pointer; padding: 0 10px; font-size: 1.1rem; }

.main-content { max-width: 1200px; margin: 0 auto; padding: 40px 24px; }

/* HERO SECTION INTACTO */
.hero-section { display: flex; justify-content: space-between; align-items: flex-end; margin-bottom: 35px; }
.glitch-title { font-size: 3rem; font-weight: 800; margin: 0; color: white; }
.hero-subtitle { color: #888; font-size: 1.1rem; margin-top: 5px; }

.btn-gradient { background: linear-gradient(90deg, #6366f1, #06b6d4); padding: 2px; border-radius: 12px; border: none; cursor: pointer; display: flex; overflow: hidden; align-self: center; }
.btn-content { background: #09090b; color: white; padding: 12px 24px; border-radius: 10px; display: block; font-weight: 700; transition: 0.3s; }
.btn-gradient:hover .btn-content { background: transparent; }

/* COMMAND BAR */
.action-bar-container { 
  display: flex; 
  justify-content: flex-end; 
  margin-bottom: 35px; 
  width: 100%;
  position: relative;
  z-index: 100;
}

.action-bar-glass {
  background: rgba(200, 220, 255, 0.10);
  backdrop-filter: blur(12px);
  border: 1px solid rgba(139, 92, 246, 0.15);
  border-radius: 20px; 
  padding: 6px 10px;
  display: flex;
  align-items: center;
  gap: 24px;
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.4);
  width: 100%;
  transition: all 0.4s ease;
}

/* SEGMENTED CONTROL */
.segmented-control {
  position: relative;
  display: flex;
  background: rgba(0, 0, 0, 0.4);
  border-radius: 14px;
  padding: 4px;
  width: 320px; 
  flex-shrink: 0;
}

.segment-btn {
  flex: 1;
  background: transparent;
  border: none;
  color: #888;
  font-size: 0.85rem;
  font-weight: 600;
  padding: 8px 0;
  cursor: pointer;
  z-index: 2;
  transition: color 0.3s ease;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 6px;
}

.segment-btn.active { color: white; }

.segment-slider {
  position: absolute;
  top: 4px; left: 4px;
  width: calc(33.333% - 4px); 
  height: calc(100% - 8px);
  background: rgba(255, 255, 255, 0.1);
  border-radius: 10px;
  transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  z-index: 1;
}

.badge-mini {
  background: rgba(6, 182, 212, 0.2);
  color: #38bdf8;
  font-size: 0.7rem;
  padding: 2px 6px;
  border-radius: 10px;
  font-weight: 800;
}
.archive-badge { 
  background: rgba(139, 92, 246, 0.2);
  color: #8b5cf6; 
}

/* CONTROLES DERECHOS */
.controls-group {
  display: flex;
  align-items: center;
  gap: 12px;
  flex: 1;
  justify-content: flex-end;
}

.search-wrapper {
  display: flex;
  align-items: center;
  background: rgba(0, 0, 0, 0.3);
  border: 1px solid rgba(255, 255, 255, 0.05);
  border-radius: 12px;
  padding: 0 12px;
  width: 180px; 
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

.search-wrapper.focused {
  width: 320px; 
  background: rgba(0, 0, 0, 0.5);
  border-color: rgba(56, 189, 248, 0.3);
  box-shadow: 0 0 15px rgba(56, 189, 248, 0.05);
}

.search-icon { color: #64748b; margin-right: 8px; flex-shrink: 0; }
.search-input {
  background: transparent;
  border: none;
  color: white;
  font-size: 0.85rem;
  width: 100%;
  outline: none;
  padding: 8px 0;
}
.search-input::placeholder { color: #64748b; font-style: italic; }

.sort-container { position: relative; }

.btn-icon-glass {
  background: rgba(255, 255, 255, 0.03);
  border: 1px solid rgba(255, 255, 255, 0.08);
  color: #94a3b8;
  padding: 8px;
  border-radius: 12px;
  cursor: pointer;
  transition: 0.3s;
  display: flex; align-items: center; justify-content: center;
}

.btn-icon-glass:hover { background: rgba(255, 255, 255, 0.08); color: white; }

.sort-dropdown {
  position: absolute;
  top: calc(100% + 12px);
  right: 0;
  background: #18181b;
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 12px;
  padding: 6px;
  min-width: 180px;
  box-shadow: 0 10px 25px rgba(0,0,0,0.5);
  z-index: 105;
}

.sort-dropdown button {
  width: 100%; padding: 10px 12px; background: transparent; border: none;
  border-radius: 8px; color: #94a3b8; text-align: left; font-size: 0.85rem; cursor: pointer; transition: 0.2s;
}

.sort-dropdown button.active { color: #38bdf8; background: rgba(56, 189, 248, 0.1); font-weight: 600; }
.sort-dropdown button:hover:not(.active) { background: rgba(255,255,255,0.05); color: white; }

.fade-menu-enter-active, .fade-menu-leave-active { transition: opacity 0.2s, transform 0.2s; }
.fade-menu-enter-from, .fade-menu-leave-to { opacity: 0; transform: translateY(-10px); }

/* GRILLAS Y MODALES */
.cards-grid { 
  display: grid; 
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr)); 
  gap: 25px; 
  position: relative;
  z-index: 10;
}

.empty-state-pro { text-align: center; padding: 80px 0; background: #18181b; border-radius: 24px; border: 1px solid rgba(255,255,255,0.08); margin-top: 20px; }
.glow-icon { font-size: 3rem; color: #06b6d4; margin-bottom: 15px; text-shadow: 0 0 20px rgba(6, 182, 212, 0.5); }
.welcome-title { color: white; margin-bottom: 5px; text-shadow: 0 0 10px rgba(255,255,255,0.2); }
.welcome-subtitle { color: #9ca3af; margin-bottom: 25px; font-size: 1rem; }
.onboarding-box { background: rgba(0, 0, 0, 0.3); border: 1px solid rgba(255, 255, 255, 0.05); padding: 15px 20px; border-radius: 12px; display: inline-block; text-align: left; }
.onboarding-box p { margin: 10px 0; font-size: 0.95rem; color: #cffafe; }
.onboarding-box strong { color: white; }

.modal-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.8); display: flex; align-items: center; justify-content: center; z-index: 2000; padding: 20px; box-sizing: border-box; }
.modal-glass { background: #111113 !important; border: 1px solid rgba(255,255,255,0.1); padding: 30px; border-radius: 20px; width: 100%; max-width: 400px; max-height: 85vh; overflow-y: auto; }
.modal-glass::-webkit-scrollbar { width: 6px; }
.modal-glass::-webkit-scrollbar-thumb { background: rgba(139, 92, 246, 0.4); border-radius: 10px; }
.modal-glass h3 { color: white !important; margin-bottom: 25px; text-align: center; }

.form-group { margin-bottom: 20px; }
.form-group label { display: block; margin-bottom: 8px; color: #888; font-size: 0.9rem; }
.form-group input, .form-group select { width: 100%; padding: 12px; background: #1c1c1f !important; border: 1px solid rgba(255,255,255,0.1); color: white !important; border-radius: 8px; box-sizing: border-box; transition: 0.3s; }

.tags-preview { display: flex; flex-wrap: wrap; gap: 8px; margin-top: 10px; }
.tag-pill-edit { background: linear-gradient(#111113, #111113) padding-box, linear-gradient(90deg, #06b6d4, #8b5cf6) border-box; border: 1px solid transparent; color: #e0e7ff; padding: 4px 10px; border-radius: 16px; font-size: 0.8rem; display: flex; align-items: center; gap: 6px; box-shadow: 0 0 8px rgba(139, 92, 246, 0.3); }
.btn-remove-tag { background: none; border: none; color: #f43f5e; cursor: pointer; font-size: 1.1rem; font-weight: bold; padding: 0; line-height: 1; display: flex; align-items: center; }
.btn-remove-tag:hover { text-shadow: 0 0 5px #f43f5e; }

.modal-actions { display: flex; gap: 15px; margin-top: 30px; }
.btn-cancel { background: transparent; border: 1px solid rgba(255,255,255,0.1); border-radius: 12px; color: #888; flex: 1; padding: 12px; cursor: pointer; transition: 0.3s; }
.btn-cancel:hover { background: rgba(255,255,255,0.05); color: white; }
.btn-save-modal { flex: 1; }
.save-content { width: 100%; text-align: center; padding: 10px; }

.toast-notification { position: fixed; bottom: 30px; right: 30px; padding: 14px 24px; border-radius: 16px; display: flex; align-items: center; gap: 12px; z-index: 3000; background: rgba(9, 9, 11, 0.85); backdrop-filter: blur(16px); color: white; font-weight: 600; font-size: 0.95rem; }
.toast-notification.success { border: 1px solid rgba(6, 182, 212, 0.5); box-shadow: 0 8px 32px rgba(0,0,0,0.5), 0 0 15px rgba(6, 182, 212, 0.2), inset 0 0 10px rgba(6, 182, 212, 0.1); }
.toast-notification.error { border: 1px solid rgba(244, 63, 94, 0.5); box-shadow: 0 8px 32px rgba(0,0,0,0.5), 0 0 15px rgba(244, 63, 94, 0.2), inset 0 0 10px rgba(244, 63, 94, 0.1); }
.toast-icon { display: flex; align-items: center; justify-content: center; }
.toast-svg { width: 22px; height: 22px; filter: drop-shadow(0 0 6px currentColor); }
.toast-notification.success .toast-svg { color: #06b6d4; }
.toast-notification.error .toast-svg { color: #f43f5e; }
.toast-slide-enter-active, .toast-slide-leave-active { transition: all 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
.toast-slide-enter-from { opacity: 0; transform: translateX(50px) scale(0.9); }
.toast-slide-leave-to { opacity: 0; transform: translateY(20px) scale(0.9); }

/* MEDIA QUERIES */
@media (max-width: 768px) {
  .toast-notification { bottom: 24px; right: 20px; left: 20px; justify-content: center; }
  .hero-section { flex-direction: column; align-items: flex-start; gap: 20px; } 
  .glitch-title { font-size: 2.2rem; } 
  .hide-mobile { display: none; } 
  
  .action-bar-container { margin-bottom: 25px; }
  
  .action-bar-glass {
    flex-direction: column;
    align-items: stretch;
    border-radius: 16px;
    padding: 12px;
    gap: 16px;
  }
  
  .segmented-control { width: 100%; }
  
  .controls-group { 
    width: 100%; 
    justify-content: space-between; 
  }
  
  .search-wrapper { flex: 1; min-width: 0; width: auto; }
  .search-wrapper.focused { width: auto; }
}

/* FAB */
.fab-mobile {
  position: fixed;
  bottom: 30px;
  right: 24px;
  width: 60px;
  height: 60px;
  background: linear-gradient(90deg, #6366f1, #06b6d4);
  padding: 2px;
  border-radius: 50%;
  border: none;
  cursor: pointer;
  z-index: 2000;
  box-shadow: 0 8px 32px rgba(99, 102, 241, 0.4);
  display: none; 
  transition: transform 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}

.fab-content {
  background: #09090b;
  color: white;
  width: 100%;
  height: 100%;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 2rem;
  font-weight: 400;
  transition: 0.3s;
}

.fab-mobile:hover {
  transform: scale(1.1) translateY(-5px);
}

.fab-mobile:hover .fab-content {
  background: transparent; 
}

.show-mobile-flex { display: none; }

@media (max-width: 768px) {
  .hide-mobile { display: none !important; }
  .show-mobile-flex { display: flex !important; }
}
</style>