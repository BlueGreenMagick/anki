<!--
Copyright: Ankitects Pty Ltd and contributors
License: GNU AGPL, version 3 or later; http://www.gnu.org/licenses/agpl.html
-->
<script lang="typescript">
    import ButtonGroupItem from "./ButtonGroupItem.svelte";
    import { setContext } from "svelte";
    import { writable } from "svelte/store";
    import { buttonGroupKey } from "./contextKeys";
    import type { Identifier } from "./identifier";
    import { insert, add } from "./identifier";
    import type { ButtonRegistration } from "./buttons";
    import { ButtonPosition } from "./buttons";
    import type { SvelteComponent } from "./registration";
    import { makeInterface } from "./registration";

    export let id: string | undefined = undefined;
    let className: string = "";
    export { className as class };

    function makeRegistration(): ButtonRegistration {
        const detach = writable(false);
        const position = writable(ButtonPosition.Standalone);
        return { detach, position };
    }

    const {
        registerComponent,
        items,
        dynamicItems,
        getDynamicInterface,
    } = makeInterface(makeRegistration);

    $: for (const [index, item] of $items.entries()) {
        if ($items.length === 1) {
            item.position.set(ButtonPosition.Standalone);
        } else if (index === 0) {
            item.position.set(ButtonPosition.Leftmost);
        } else if (index === $items.length - 1) {
            item.position.set(ButtonPosition.Rightmost);
        } else {
            item.position.set(ButtonPosition.Center);
        }
    }

    setContext(buttonGroupKey, registerComponent);

    export let api: Record<string, unknown> | undefined = undefined;
    let buttonGroupRef: HTMLDivElement;

    $: if (api && buttonGroupRef) {
        const { addComponent, updateRegistration } = getDynamicInterface(
            buttonGroupRef
        );

        const insertButton = (button: SvelteComponent, position: Identifier = 0) =>
            addComponent(button, (added, parent) => insert(added, parent, position));
        const appendButton = (button: SvelteComponent, position: Identifier = -1) =>
            addComponent(button, (added, parent) => add(added, parent, position));

        const showButton = (id: Identifier) =>
            updateRegistration(({ detach }) => detach.set(false), id);
        const hideButton = (id: Identifier) =>
            updateRegistration(({ detach }) => detach.set(true), id);
        const toggleButton = (id: Identifier) =>
            updateRegistration(
                ({ detach }) => detach.update((old: boolean): boolean => !old),
                id
            );

        Object.assign(api, {
            insertButton,
            appendButton,
            showButton,
            hideButton,
            toggleButton,
        });
    }
</script>

<style lang="scss">
    div {
        flex-wrap: var(--toolbar-wrap);
        padding: calc(var(--toolbar-size) / 10);
        margin: 0;
    }
</style>

<div
    bind:this={buttonGroupRef}
    {id}
    class={`btn-group ${className}`}
    dir="ltr"
    role="group">
    <slot />
    {#each $dynamicItems as item}
        <ButtonGroupItem id={item[0].id} registration={item[1]}>
            <svelte:component this={item[0].component} {...item[0].props} />
        </ButtonGroupItem>
    {/each}
</div>