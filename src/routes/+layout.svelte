<script lang="ts">
   import { page } from '$app/stores';
	
	import { ProgressBar } from '@prgm/sveltekit-progress-bar';
	import { ModeWatcher } from 'mode-watcher';
	import { CodeSolid, HeartRegular } from 'svelte-awesome-icons';
	import { MetaTags } from 'svelte-meta-tags';
	import '../app.css';
	$: location = $page.data.location;
   import "../app.css";
  </script>
  
  <slot />