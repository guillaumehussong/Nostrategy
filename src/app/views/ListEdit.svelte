<script>
  import {pluck, find} from "ramda"
  import {Tags, displayRelay} from "src/util/nostr"
  import {modal, toast} from "src/partials/state"
  import Heading from "src/partials/Heading.svelte"
  import PersonBadge from "src/app/shared/PersonBadge.svelte"
  import Content from "src/partials/Content.svelte"
  import Button from "src/partials/Button.svelte"
  import Input from "src/partials/Input.svelte"
  import MultiSelect from "src/partials/MultiSelect.svelte"
  import {searchTopics, searchPeople, searchRelays, getPersonWithFallback} from "src/agent/db"
  import user from "src/agent/user"

  export let list

  const tags = Tags.wrap(list?.tags || [])

  let values = {
    name: tags.getMeta("d") || "",
    params: tags.type(["t", "p"]).all(),
    relays: tags.type("r").all(),
  }

  const search = q => {
    if (q.startsWith("#")) {
      return $searchTopics(q)
        .slice(0, 5)
        .map(({name}) => ["t", name])
    }

    return $searchPeople(q)
      .slice(0, 5)
      .map(({pubkey}) => ["p", pubkey])
  }

  const _searchRelays = q => pluck("url", $searchRelays(q)).map(url => ["r", url])

  const submit = () => {
    if (!values.name) {
      return toast.show("error", "A name is required for your list")
    }

    if (
      find(e => e.id !== list?.id && Tags.from(e).getMeta("d") === values.name, user.getLists())
    ) {
      return toast.show("error", "That name is already in use")
    }

    const {name, params, relays} = values

    user.putList(list?.id, name, params, relays)
    toast.show("info", "Your list has been saved!")
    modal.pop()
  }
</script>

<form on:submit|preventDefault={submit}>
  <Content>
    <Heading class="text-center">{values.id ? "Edit" : "Add"} list</Heading>
    <div class="flex w-full flex-col gap-8">
      <div class="flex flex-col gap-1">
        <strong>Name</strong>
        <Input bind:value={values.name} placeholder="My list" />
        <p class="text-sm text-gray-4">
          Lists are identified by their name, so this has to be unique.
        </p>
      </div>
      <div class="flex flex-col gap-1">
        <strong>Topics and People</strong>
        <MultiSelect {search} bind:value={values.params}>
          <div slot="item" let:item>
            {#if item[0] === "p"}
              <div class="-my-1">
                <PersonBadge inert person={getPersonWithFallback(item[1])} />
              </div>
            {:else}
              <strong>#{item[1]}</strong>
            {/if}
          </div>
        </MultiSelect>
        <p class="text-sm text-gray-4">Type "@" to look for people, and "#" to look for topics.</p>
      </div>
      <div class="flex flex-col gap-1">
        <strong>Relays</strong>
        <MultiSelect search={_searchRelays} bind:value={values.relays}>
          <div slot="item" let:item>
            {displayRelay({url: item[1]})}
          </div>
        </MultiSelect>
        <p class="text-sm text-gray-4">
          Select which relays to limit this list to. If you leave this blank, your default relays
          will be used.
        </p>
      </div>
      <Button type="submit" class="text-center">Save</Button>
    </div>
  </Content>
</form>
