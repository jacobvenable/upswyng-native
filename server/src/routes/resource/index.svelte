<script context="module">
  export async function preload({ params, query }, { user }) {
    const { categories } = await this.fetch("/api/categories").then(r =>
      r.json()
    );
    const { uncategorizedResources } = await this.fetch(
      "/api/resources/uncategorized"
    ).then(r => r.json());
    // TODO: Remove displaying all resources once they get too big
    const { resources: allResources } = await this.fetch("/api/resources").then(
      r => r.json()
    );
    const { draftResources } = await this.fetch("/api/resources/drafts").then(
      r => r.json()
    );
    return {
      allResources,
      categories,
      draftResources,
      uncategorizedResources,
      user
    };
  }
</script>

<script>
  export let categories = null;
  export let draftResources = null;
  export let uncategorizedResources = null;
  export let allResources = null;
  export let user = null;
</script>

<svelte:head>
  <title>Resources</title>
</svelte:head>

<h1>
  <a href="/resource/create">Create a New Resource</a>
</h1>

<h1>Resource Categories</h1>

{#if categories.length}
  <ul>
    {#each categories as category}
      <li>
        <a href={`/category/${category.stub}`}>{category.name}</a>
      </li>
    {/each}
  </ul>
{:else}
  <span>No categories found.</span>
{/if}

{#if user && user.isAdmin}
  <h1>Draft Resources</h1>

  {#if draftResources.length}
    <ul>
      {#each draftResources as draftResource}
        <li>
          <a href={`/resource/draft/${draftResource._id}`}>
            {draftResource.name}
          </a>
        </li>
      {/each}
    </ul>
  {:else}
    <span>No drafts at this time.</span>
  {/if}
{/if}

<h1>Uncategorized Resources</h1>

{#if uncategorizedResources.length}
  <ul>
    {#each uncategorizedResources as resource}
      <li>
        <a href={`/resource/${resource.id}`}>{resource.name}</a>
      </li>
    {/each}
  </ul>
{:else}
  <span>No uncategorized resources.</span>
{/if}

<h1>All Resources</h1>
{#if allResources.length}
  <ul>
    {#each allResources as resource}
      <li>
        <a href={`/resource/${resource.id}`}>{resource.name}</a>
      </li>
    {/each}
  </ul>
{:else}
  <span>There an no resources.</span>
{/if}
