<script lang="ts">
	import '../app.css';
	import favicon from '$lib/assets/favicon.svg';
	import { onMount } from 'svelte';
	import { authService } from '$lib/services/auth.js';
	
	let { children } = $props();
	let isAuthenticated = $state(false);
	let currentUser = $state<any>(null);
	let currentPath = $state('');

	onMount(() => {
		currentPath = window.location.pathname;
		isAuthenticated = authService.isAuthenticated();
		currentUser = authService.getUserData();

		// Proteger rutas
		if (!isAuthenticated && currentPath !== '/login') {
			window.location.href = '/login';
		}
	});

	async function handleLogout() {
		if (confirm('¿Está seguro que desea cerrar sesión?')) {
			await authService.logout();
			window.location.href = '/login';
		}
	}
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
</svelte:head>

{#if currentPath === '/login'}
	{@render children?.()}
{:else if isAuthenticated}
	<div class="app-layout">
		<nav class="main-nav">
			<div class="nav-brand">
				<h1>BRISA</h1>
			</div>
			<div class="nav-links">
				<a href="/esquelas" class="nav-link">Esquelas</a>
				<a href="/codigos" class="nav-link">Códigos</a>
			</div>
			<div class="nav-user">
				{#if currentUser}
					<span class="user-name">
						{currentUser.nombres || currentUser.usuario}
						{#if currentUser.roles && currentUser.roles.length > 0}
							<span class="user-role">({currentUser.roles[0].nombre})</span>
						{/if}
					</span>
				{/if}
				<button class="btn-logout" on:click={handleLogout}>
					Salir
				</button>
			</div>
		</nav>
		<main class="main-content">
			{@render children?.()}
		</main>
	</div>
{/if}

<style>
	.app-layout {
		min-height: 100vh;
		display: flex;
		flex-direction: column;
	}

	.main-nav {
		background: linear-gradient(135deg, #1e293b 0%, #334155 100%);
		padding: 1rem 2rem;
		display: flex;
		justify-content: space-between;
		align-items: center;
		box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
	}

	.nav-brand h1 {
		margin: 0;
		color: #22d3ee;
		font-size: 1.75rem;
		font-weight: 700;
	}

	.nav-links {
		display: flex;
		gap: 1.5rem;
		flex: 1;
		justify-content: center;
	}

	.nav-link {
		color: #e2e8f0;
		text-decoration: none;
		font-weight: 500;
		padding: 0.5rem 1rem;
		border-radius: 6px;
		transition: all 0.2s;
	}

	.nav-link:hover {
		background: rgba(34, 211, 238, 0.1);
		color: #22d3ee;
	}

	.nav-user {
		display: flex;
		align-items: center;
		gap: 1rem;
	}

	.user-name {
		color: #e2e8f0;
		font-size: 0.9rem;
	}

	.user-role {
		color: #22d3ee;
		font-weight: 600;
		margin-left: 0.25rem;
	}

	.btn-logout {
		background: rgba(239, 68, 68, 0.1);
		color: #ef4444;
		border: 1px solid #ef4444;
		padding: 0.5rem 1rem;
		border-radius: 6px;
		font-weight: 500;
		cursor: pointer;
		transition: all 0.2s;
		font-size: 0.9rem;
	}

	.btn-logout:hover {
		background: #ef4444;
		color: white;
	}

	.main-content {
		flex: 1;
	}
</style>
