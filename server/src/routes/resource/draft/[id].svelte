<script context="module">
  export async function preload({ params, query }, { user }) {
    if (!user || !user.isAdmin) {
      this.error(401, "You must be an admin to access this page.");
    }

    const resourceResponse = await this.fetch(
      `/api/resource/draft/${params.id}`
    );
    const resourceData = await resourceResponse.json();

    if (resourceResponse.status !== 200) {
      this.error(resourceResponse.status, resourceData.message);
    } else {
      // see if we have an existing resource corresponding to this draft
      const existingResourceResponse = await this.fetch(
        `/api/resource/${resourceData.draftResource.id}`
      );
      if (existingResourceResponse.status === 404) {
        return {
          draftResource: resourceData.draftResource,
          existingResource: null
        };
      }
      const existingResourceData = await existingResourceResponse.json();
      if (existingResourceResponse.status !== 200) {
        this.error(
          existingResourceResponse.status,
          existingResourceData.message
        );
      } else {
        return {
          draftResource: resourceData.draftResource,
          existingResource: existingResourceData.resource
        };
      }
    }
  }

  function deleteDraft(id) {
    isDeleting = true;

    fetch(`/api/resource/draft/delete/${id}`, { method: "POST" })
      .then(_res => {
        if (_res.status >= 400) {
          throw new Error(_res);
        }
        window.location.href = "/resource";
      })
      .catch(e => (deleteError = e))
      .finally(() => (isDeleting = false));
  }

  function approveUpdate(id) {
    fetch(`/api/resource/draft/approve/${id}`, { method: "POST" })
      .then(_res => {
        if (_res.status >= 400) {
          throw new Error(_res);
        }
        window.location.href = "/resource";
      })
      .catch(e => (approveError = e))
      .finally(() => (isDeleting = false));
  }
</script>

<script>
  import ResourceDisplay from "../../../components/ResourceDisplay.svelte";
  import ResourceDiff from "../../../components/ResourceDiff.svelte";

  export let draftResource;
  export let existingResource; // resource in the directory which this draft would update; null for new resources

  let isDeleting = false; // Whether we've issued a call to the server to delete the draft resource
  let deleteError = null; // error? Poupulated with the error from a detete attempt, if there has been one

  let isApproving = false; // Whether we've issued a call to the server to approve the draft resource
  let approveError = null; // error? Poupulated with the error from an approve attempt, if there has been one
</script>

<svelte:head>
  <title>Upswyng: {draftResource.name} [draft]</title>
</svelte:head>

{#if existingResource}
  <h1>Update Resource: {existingResource.name}</h1>
  <span>ID: {existingResource.id}</span>
  <ResourceDiff leftResource={existingResource} rightResource={draftResource} />
{:else}
  <h1>Create New Resource</h1>
  <ResourceDisplay resource={draftResource} />
{/if}

<div>
  <button
    type="button"
    preventDefault
    disabled={isDeleting || isApproving}
    on:click={() => deleteDraft(draftResource._id)}>
    Delete Draft
  </button>
  <button
    type="button"
    preventDefault
    disabled={isDeleting || isApproving}
    on:click={() => approveUpdate(draftResource._id)}>
    Approve Update
  </button>
</div>
<div>
  {#if approveError}
    <p>There was an error approving the resource: {approveError.meessage}</p>
  {/if}
  {#if deleteError}
    <p>There was an error deleting the resource: {deleteError.meessage}</p>
  {/if}
</div>
