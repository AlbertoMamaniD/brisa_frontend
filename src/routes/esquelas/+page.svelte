<script lang="ts">
  import { onMount } from 'svelte';
  import { apiClient } from '$lib/services/api.js';
  import type { EsquelaResponse, CodigoEsquela } from '$lib/types/api.js';
  import CreateEsquelaModal from '$lib/components/CreateEsquelaModal.svelte';

  let searchQuery = '';
  let esquelas: EsquelaResponse[] = [];
  let codigos: CodigoEsquela[] = [];
  let loading = true;
  let error = '';
  let showCreateModal = false;

  // Mapeo de códigos a iconos y tipos
  const tipoIconos: Record<string, string> = {
    'reconocimiento': '🏆',
    'orientacion': '📋',
    'academico': '📚',
    'deportivo': '🥇',
    'comportamiento': '❤️',
    'participacion': '⭐'
  };

  let categoriaSeleccionada: string | null = null;

  onMount(async () => {
    await cargarDatos();
  });

  async function cargarDatos() {
    try {
      loading = true;
      const [esquelasData, codigosData] = await Promise.all([
        apiClient.getEsquelas(),
        apiClient.getCodigosEsquelas()
      ]);
      esquelas = esquelasData;
      codigos = codigosData;
      error = '';
    } catch (err: any) {
      console.error('Error cargando datos:', err);
      error = err.message || 'Error cargando datos';
    } finally {
      loading = false;
    }
  }

  function obtenerCategorias() {
    const categoriaCount: Record<string, number> = {};
    
    esquelas.forEach(esquela => {
      esquela.codigos.forEach(codigo => {
        const tipo = codigo.tipo;
        categoriaCount[tipo] = (categoriaCount[tipo] || 0) + 1;
      });
    });

    return Object.entries(categoriaCount).map(([nombre, cantidad]) => ({
      nombre,
      cantidad,
      icono: tipoIconos[nombre] || '📄'
    }));
  }

  function filtrarEsquelas() {
    let filtradas = esquelas;
    
    if (categoriaSeleccionada) {
      filtradas = filtradas.filter(esquela => 
        esquela.codigos.some(codigo => codigo.tipo === categoriaSeleccionada)
      );
    }
    
    if (searchQuery) {
      filtradas = filtradas.filter(esquela => 
        esquela.observaciones.toLowerCase().includes(searchQuery.toLowerCase()) ||
        esquela.codigos.some(codigo => 
          codigo.descripcion.toLowerCase().includes(searchQuery.toLowerCase())
        )
      );
    }
    
    return filtradas;
  }

  function seleccionarCategoria(categoria: string) {
    categoriaSeleccionada = categoriaSeleccionada === categoria ? null : categoria;
  }

  function formatearFecha(fecha: string) {
    return new Date(fecha).toLocaleDateString('es-ES', {
      day: 'numeric',
      month: 'long',
      year: 'numeric'
    });
  }

  function obtenerPrimerCodigo(esquela: EsquelaResponse) {
    return esquela.codigos[0] || { tipo: 'general', descripcion: 'Sin código', codigo: '' };
  }

  async function eliminarEsquela(id: number) {
    if (confirm('¿Está seguro de que desea eliminar esta esquela?')) {
      try {
        await apiClient.deleteEsquela(id);
        await cargarDatos(); // Recargar datos
      } catch (err: any) {
        alert('Error eliminando esquela: ' + err.message);
      }
    }
  }
</script>

<div class="esquelas-container">
  <!-- Header -->
  <div class="header">
    <div>
      <h1 class="titulo">Esquelas y Reconocimientos</h1>
      <p class="subtitulo">Registra reconocimientos y felicitaciones estudiantiles</p>
    </div>
    <button class="btn-nueva" on:click={() => showCreateModal = true}>
      <span class="plus">+</span>
      Nueva Esquela
    </button>
  </div>

  {#if loading}
    <div class="loading">
      <div class="spinner"></div>
      <p>Cargando esquelas...</p>
    </div>
  {:else if error}
    <div class="error">
      <p>❌ {error}</p>
      <button class="btn-retry" on:click={cargarDatos}>Reintentar</button>
    </div>
  {:else}
    <!-- Categorías -->
    <div class="categorias">
      {#each obtenerCategorias() as categoria}
        <button 
          class="categoria-card"
          class:active={categoriaSeleccionada === categoria.nombre}
          on:click={() => seleccionarCategoria(categoria.nombre)}
        >
          <div class="categoria-header">
            <span class="categoria-titulo">{categoria.nombre}</span>
            <span class="categoria-icono">{categoria.icono}</span>
          </div>
          <span class="categoria-cantidad">{categoria.cantidad}</span>
        </button>
      {/each}
    </div>

    <!-- Buscador -->
    <div class="buscador">
      <span class="icono-buscar">🔍</span>
      <input 
        type="text" 
        placeholder="Buscar por observaciones o tipo de código..."
        bind:value={searchQuery}
      />
    </div>

    <!-- Grid de Esquelas -->
    <div class="esquelas-grid">
      {#each filtrarEsquelas() as esquela}
        {@const primerCodigo = obtenerPrimerCodigo(esquela)}
        <div class="esquela-card" class:reconocimiento={primerCodigo.tipo === 'reconocimiento'}>
          <div class="esquela-header">
            <span class="esquela-icono">{tipoIconos[primerCodigo.tipo] || '📄'}</span>
            <span class="esquela-fecha">{formatearFecha(esquela.fecha)}</span>
          </div>
          
          <h3 class="esquela-titulo">{primerCodigo.descripcion}</h3>
          
          <div class="esquela-info">
            <div class="info-item">
              <span class="info-label">Código:</span>
              <span class="info-valor">{primerCodigo.codigo}</span>
            </div>
            <div class="info-item">
              <span class="info-label">Tipo:</span>
              <span class="info-valor">{primerCodigo.tipo}</span>
            </div>
          </div>
          
          <p class="esquela-descripcion">{esquela.observaciones}</p>
          
          {#if esquela.codigos.length > 1}
            <div class="codigos-adicionales">
              <span class="codigos-label">Códigos adicionales:</span>
              {#each esquela.codigos.slice(1) as codigo}
                <span class="codigo-tag">{codigo.codigo}</span>
              {/each}
            </div>
          {/if}
          
          <div class="esquela-footer">
            <span class="esquela-id">ID: {esquela.id_esquela}</span>
            <button 
              class="btn-eliminar"
              on:click={() => eliminarEsquela(esquela.id_esquela)}
              title="Eliminar esquela"
            >
              🗑️
            </button>
          </div>
        </div>
      {/each}

      {#if filtrarEsquelas().length === 0}
        <div class="no-results">
          <p>No se encontraron esquelas que coincidan con los filtros.</p>
        </div>
      {/if}
    </div>
  {/if}

  <!-- Modal para crear esquela -->
  <CreateEsquelaModal 
    bind:show={showCreateModal} 
    {codigos}
    on:created={cargarDatos}
  />
</div>

<style>
  .esquelas-container {
    padding: 2rem;
    background-color: #f5f7fa;
    min-height: 100vh;
  }

  .header {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    margin-bottom: 2rem;
  }

  .titulo {
    font-size: 1.875rem;
    font-weight: 600;
    color: #1e293b;
    margin: 0 0 0.5rem 0;
  }

  .subtitulo {
    color: #64748b;
    font-size: 0.95rem;
    margin: 0;
  }

  .btn-nueva {
    background: linear-gradient(135deg, #22d3ee 0%, #06b6d4 100%);
    color: white;
    border: none;
    padding: 0.75rem 1.5rem;
    border-radius: 8px;
    font-weight: 500;
    cursor: pointer;
    display: flex;
    align-items: center;
    gap: 0.5rem;
    font-size: 0.95rem;
    transition: transform 0.2s, box-shadow 0.2s;
    box-shadow: 0 4px 12px rgba(34, 211, 238, 0.3);
  }

  .btn-nueva:hover {
    transform: translateY(-2px);
    box-shadow: 0 6px 16px rgba(34, 211, 238, 0.4);
  }

  .plus {
    font-size: 1.25rem;
    font-weight: 300;
  }

  .categorias {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 1rem;
    margin-bottom: 2rem;
  }

  .categoria-card {
    background: white;
    border: 2px solid #e2e8f0;
    border-radius: 12px;
    padding: 1.25rem;
    cursor: pointer;
    transition: all 0.3s;
  }

  .categoria-card:hover {
    border-color: #22d3ee;
    transform: translateY(-2px);
    box-shadow: 0 4px 12px rgba(34, 211, 238, 0.15);
  }

  .categoria-card.active {
    border-color: #22d3ee;
    background: linear-gradient(135deg, rgba(34, 211, 238, 0.1) 0%, rgba(6, 182, 212, 0.05) 100%);
  }

  .categoria-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 0.75rem;
  }

  .categoria-titulo {
    color: #64748b;
    font-size: 0.9rem;
    font-weight: 500;
  }

  .categoria-icono {
    font-size: 1.5rem;
    opacity: 0.7;
  }

  .categoria-cantidad {
    font-size: 2rem;
    font-weight: 600;
    color: #22d3ee;
  }

  .buscador {
    background: white;
    border: 2px solid #e2e8f0;
    border-radius: 12px;
    padding: 0.875rem 1.25rem;
    display: flex;
    align-items: center;
    gap: 0.75rem;
    margin-bottom: 2rem;
  }

  .icono-buscar {
    font-size: 1.25rem;
    color: #94a3b8;
  }

  .buscador input {
    border: none;
    outline: none;
    flex: 1;
    font-size: 0.95rem;
    color: #1e293b;
  }

  .buscador input::placeholder {
    color: #94a3b8;
  }

  .esquelas-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(380px, 1fr));
    gap: 1.5rem;
  }

  .esquela-card {
    background: white;
    border: 2px solid #e2e8f0;
    border-radius: 12px;
    padding: 1.5rem;
    transition: all 0.3s;
  }

  .esquela-card:hover {
    transform: translateY(-4px);
    box-shadow: 0 8px 24px rgba(0, 0, 0, 0.08);
  }

  .esquela-card.reconocimiento {
    border-color: #22d3ee;
    background: linear-gradient(135deg, rgba(34, 211, 238, 0.05) 0%, white 100%);
  }

  .loading {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 4rem;
    color: #64748b;
  }

  .spinner {
    width: 40px;
    height: 40px;
    border: 4px solid #e2e8f0;
    border-top: 4px solid #22d3ee;
    border-radius: 50%;
    animation: spin 1s linear infinite;
    margin-bottom: 1rem;
  }

  @keyframes spin {
    0% { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
  }

  .error {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 2rem;
    background: #fef2f2;
    border: 2px solid #fca5a5;
    border-radius: 12px;
    margin: 2rem 0;
  }

  .error p {
    color: #dc2626;
    margin: 0 0 1rem 0;
  }

  .btn-retry {
    background: #dc2626;
    color: white;
    border: none;
    padding: 0.5rem 1rem;
    border-radius: 6px;
    cursor: pointer;
    font-size: 0.9rem;
  }

  .btn-retry:hover {
    background: #b91c1c;
  }

  .codigos-adicionales {
    margin: 1rem 0;
    padding-top: 1rem;
    border-top: 1px solid #e2e8f0;
  }

  .codigos-label {
    display: block;
    font-size: 0.8rem;
    color: #64748b;
    margin-bottom: 0.5rem;
  }

  .codigo-tag {
    display: inline-block;
    background: #f1f5f9;
    color: #475569;
    padding: 0.25rem 0.5rem;
    border-radius: 4px;
    font-size: 0.75rem;
    margin-right: 0.5rem;
    margin-bottom: 0.25rem;
  }

  .btn-eliminar {
    background: none;
    border: none;
    font-size: 1.2rem;
    cursor: pointer;
    padding: 0.25rem;
    border-radius: 4px;
    transition: background-color 0.2s;
  }

  .btn-eliminar:hover {
    background: #fef2f2;
  }

  .esquela-id {
    font-size: 0.8rem;
    color: #94a3b8;
  }

  .no-results {
    grid-column: 1 / -1;
    text-align: center;
    padding: 4rem;
    color: #64748b;
  }

  .esquela-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 1rem;
  }

  .esquela-icono {
    font-size: 2rem;
    background: linear-gradient(135deg, #f0fdfa 0%, #ccfbf1 100%);
    width: 50px;
    height: 50px;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 10px;
  }

  .esquela-fecha {
    background: #22d3ee;
    color: white;
    padding: 0.375rem 0.875rem;
    border-radius: 20px;
    font-size: 0.8rem;
    font-weight: 500;
  }

  .esquela-titulo {
    font-size: 1.25rem;
    font-weight: 600;
    color: #1e293b;
    margin: 0 0 1rem 0;
  }

  .esquela-info {
    display: flex;
    gap: 1.5rem;
    margin-bottom: 1rem;
  }

  .info-item {
    display: flex;
    flex-direction: column;
    gap: 0.25rem;
  }

  .info-label {
    font-size: 0.8rem;
    color: #64748b;
  }

  .info-valor {
    font-weight: 600;
    color: #1e293b;
  }

  .esquela-descripcion {
    color: #475569;
    font-size: 0.9rem;
    line-height: 1.5;
    margin: 0 0 1rem 0;
  }

  .esquela-footer {
    padding-top: 1rem;
    border-top: 1px solid #e2e8f0;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }


</style>