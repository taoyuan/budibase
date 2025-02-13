<script>
  import {
    Layout,
    Heading,
    Body,
    Button,
    Select,
    Divider,
    Modal,
    Search,
    Page,
    Table,
  } from "@budibase/bbui"
  import { onMount } from "svelte"
  import { plugins, admin } from "stores/portal"
  import AddPluginModal from "./_components/AddPluginModal.svelte"
  import PluginNameRenderer from "./_components/PluginNameRenderer.svelte"
  import EditPluginRenderer from "./_components/EditPluginRenderer.svelte"

  const schema = {
    name: {
      width: "2fr",
      minWidth: "200px",
    },
    version: {
      width: "1fr",
    },
    "schema.type": {
      width: "1fr",
      displayName: "Type",
      capitalise: true,
      minWidth: "120px",
    },
    edit: {
      width: "auto",
      borderLeft: true,
      displayName: "",
    },
  }
  const customRenderers = [
    { column: "name", component: PluginNameRenderer },
    { column: "edit", component: EditPluginRenderer },
  ]

  let modal
  let searchTerm = ""
  let filter = "all"
  let filterOptions = [
    { label: "All plugins", value: "all" },
    { label: "Components", value: "component" },
  ]

  if (!$admin.cloud) {
    filterOptions.push({ label: "Datasources", value: "datasource" })
  }

  $: filteredPlugins = $plugins
    .filter(plugin => {
      return filter === "all" || plugin.schema.type === filter
    })
    .filter(plugin => {
      return (
        !searchTerm ||
        plugin?.name?.toLowerCase().includes(searchTerm.toLowerCase())
      )
    })

  onMount(async () => {
    await plugins.load()
  })
</script>

<Page narrow>
  <Layout noPadding>
    <Layout gap="XS" noPadding>
      <Heading size="M">Plugins</Heading>
      <Body>Add your own custom datasources and components</Body>
    </Layout>
    <Divider />

    <div class="controls">
      <div>
        <Button on:click={modal.show} cta>Add plugin</Button>
      </div>
      {#if $plugins?.length}
        <div class="filters">
          <div class="select">
            <Select
              bind:value={filter}
              placeholder={null}
              options={filterOptions}
              autoWidth
            />
          </div>
          <Search bind:value={searchTerm} placeholder="Search" />
        </div>
      {/if}
    </div>

    <Table
      {schema}
      data={filteredPlugins}
      allowEditColumns={false}
      allowEditRows={false}
      allowSelectRows={false}
      allowClickRows={false}
      {customRenderers}
    />
  </Layout>
</Page>

<Modal bind:this={modal}>
  <AddPluginModal />
</Modal>

<style>
  .filters {
    display: flex;
    gap: var(--spacing-xl);
  }
  .controls {
    display: flex;
    gap: var(--spacing-xl);
    justify-content: space-between;
    flex-wrap: wrap;
  }
  .controls :global(.spectrum-Search) {
    width: 200px;
  }

  @media (max-width: 640px) {
    .filters {
      display: grid;
      grid-template-columns: 1fr 1fr;
    }
    .controls :global(.spectrum-Search) {
      width: auto;
    }
  }
</style>
