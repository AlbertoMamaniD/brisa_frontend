<script lang="ts">
    import { onMount } from "svelte";
    import { getEstudiantes } from "./services";
    import type { Estudiante } from "./types";

    let estudiantes: Estudiante[] = [];
    let loading = true;
    let error = "";

    onMount(async () => {
        try {
            estudiantes = await getEstudiantes();
        } catch (e) {
            error = "No se pudieron cargar los estudiantes";
        } finally {
            loading = false;
        }
    });
</script>

<h1>Lista de Estudiantes</h1>

{#if loading}
    <p>Cargando estudiantes...</p>
{:else if error}
    <p class="error">{error}</p>
{:else}
    <ul>
        {#each estudiantes as estudiante}
            <li>
                <strong>{estudiante.nombre}</strong> - {estudiante.curso}
            </li>
        {/each}
    </ul>
{/if}

<style>
    h1 {
        font-size: 2rem;
        margin-bottom: 1rem;
    }
    ul {
        list-style: none;
        padding: 0;
    }
    li {
        padding: 0.5rem;
        border-bottom: 1px solid #ccc;
    }
    .error {
        color: red;
    }
</style>
