<script lang="ts">
  type LinkInfo = string | {name: string; title: string; hash?: string};
  export let links: LinkInfo[];
  import NavLink from './NavLink.svelte';
  const base: string = window.basepath || '/';
</script>

<div
  class="absolute w-full h-1 -m-1 z-40"
  style={`background: url(${base}images/multicolor_line_x8.png);`} />
<ul class="flex m-1">
  {#each links as link}
    <NavLink
      name={typeof link === 'string' ? link : link.name}
      hash={link.hash}>
      {typeof link === 'string' ? link : link.title}
    </NavLink>
  {/each}
</ul>
<div
  class="absolute w-full h-1 -m-1 z-40"
  style={`background: url(${base}images/multicolor_line_x8.png);`} />
