<script context="module" lang="ts">
  export function preload() {
    return this.fetch("projects.json")
      .then((r) => r.json())
      .then((projects) => {
        return { projects };
      });
  }
</script>

<script lang="ts">
  import type { Project } from "./projects/_projects";

  export let projects: Project[];
</script>

<svelte:head>
  <title>Projects · Cassidy Bandy</title>
</svelte:head>

<h1>Projects</h1>
<ul class="list">
  {#each projects as project}
    <li>
      <a href={"projects/" + project.slug} class="text-link">{project.title}</a>
      - {project.summary}
    </li>
  {/each}
</ul>
