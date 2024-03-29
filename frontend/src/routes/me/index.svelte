<script context="module">
    import timeago from "timeago-simple";
    import Card from "$lib/components/Card.svelte";
</script>

<script lang="ts">
    import { API_BASE_URL } from "$lib/env";
    import { goto } from "$app/navigation";
    import { onMount } from "svelte";
    import { token } from "$lib/stores/local";
    import Loading from "$lib/components/Loading.svelte";
    import { browser } from "$app/env";

    function timeout(ms: number) {
        return new Promise((resolve) => setTimeout(resolve, ms));
    }

    onMount(async () => {
        if (!$token.secret) {
            return await goto("/me/login");
        }
    });

    const me = async () => {
        const response = await fetch(`${API_BASE_URL}/me/`, {
            headers: {
                Authorization: $token.secret
            }
        });
        await timeout(5000);
        if (response.status === 200) {
            return await response.json();
        } else {
            throw new Error("could not fetch user data.");
        }
    };
</script>

<svelte:head>
    <title>Me | Feather</title>
</svelte:head>

{#if browser}
    {#await me()}
        <Loading />
    {:then { id, login, name, tokens }}
        <div class="container mx-auto flex flex-col px-4 my-8">
            <h1 class="text-4xl font-bold mt-8">Account settings</h1>
            <div class="flex flex-col">
                <div>
                    <h2 class="text-xl font-bold mt-8">Profile Information</h2>
                    <Card class="mt-4">
                        <div class="flex items-center">
                            <img
                                src={`https://avatars.githubusercontent.com/u/${id}`}
                                alt={name}
                                class="h-16 w-16 mr-4 rounded-sm"
                            />
                            <ul>
                                <li>
                                    <span class="font-bold">Name:</span>
                                    <span>{name}</span>
                                </li>
                                <li>
                                    <span class="font-bold">Github Account:</span>
                                    <span>{login}</span>
                                </li>
                            </ul>
                        </div>
                    </Card>
                </div>
                <div>
                    <h2 class="text-xl font-bold mt-8 mb-2">API Access</h2>
                    <p class="leading-6">
                        If you want to use plugin commands from the command line, you will need to
                        login with <code>cargo quill login (token)</code> using one of the tokens listed
                        below.
                    </p>
                    <p class="leading-6">
                        When working in shared environments, supplying the token on the command line
                        could expose it to prying eyes. To avoid this, enter <code
                            >cargo quill login</code
                        > and supply your token when prompted.
                    </p>
                    <h3 class="text-xl mt-4">Tokens</h3>
                    <ul>
                        <li class="mt-4">
                            <Card>
                                <div class="flex justify-between">
                                    <input
                                        type="text"
                                        class="flex flex-1 px-4 py-2 border border-feather-light box-border mr-4 dark:bg-gray-800 outline-none dark:border-nether-netherrack-dark rounded"
                                        placeholder="New token name"
                                    />
                                    <button
                                        class="bg-feather-dark dark:bg-nether-netherbrick text-white px-4 py-2 hover:text-feather-light font-bold"
                                    >
                                        Create
                                    </button>
                                </div>
                            </Card>
                        </li>
                        {#each tokens as { name, used_total, created_at, token }}
                            <li class="mt-4">
                                <Card>
                                    <div class="flex justify-between">
                                        <span class="text-lg font-bold">{name}</span>
                                        <div class="flex">
                                            <div class="flex flex-col text-sm font-light mr-4">
                                                <span
                                                    >Created {timeago.simple(
                                                        new Date(created_at)
                                                    )}</span
                                                >
                                                {#if used_total > 0}
                                                    <span>Used {used_total} times</span>
                                                {:else}
                                                    <span>Never used </span>
                                                {/if}
                                            </div>
                                            <button
                                                class="bg-feather-dark dark:bg-nether-netherbrick text-white px-4 py-2 hover:text-feather-light font-bold"
                                            >
                                                Revoke
                                            </button>
                                        </div>
                                    </div>
                                    {#if token}
                                        <p>
                                            Please record this token somewhere, you cannot retrieve
                                            its value again. For use on the command line you can
                                            save it to ~/.cargo/quill with:
                                        </p>
                                        <div class="border-2 px-4 py-2 mt-4">
                                            <code>cargo quill login {token}</code>
                                        </div>
                                    {/if}
                                </Card>
                            </li>
                        {/each}
                    </ul>
                </div>
            </div>
        </div>
    {:catch error}
        {error}
    {/await}
{/if}
