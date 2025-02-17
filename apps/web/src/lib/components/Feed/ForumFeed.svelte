<script lang="ts">
	import { derived } from "svelte/store";
	import { NDKEvent, NDKEventId, NDKFilter, NDKKind, NDKUser } from "@nostr-dev-kit/ndk";
	import { ndk } from "@kind0/ui-common";
	import { onDestroy } from "svelte";
	import ForumFeedItem from "./ForumFeedItem.svelte";
	import NewPost from "./NewPost.svelte";

    export let user: NDKUser;
    export let filters: NDKFilter[] = [
        { kinds: [NDKKind.GroupNote, NDKKind.GroupReply], "#h": [user.pubkey] }
    ]

    const feed = $ndk.storeSubscribe(filters);

    onDestroy(() => { feed.unsubscribe(); });

    const perNoteLatestActivity = new Map<NDKEventId, number>();

    const renderFeed = derived(feed, $feed => {
        const topLevelNotes = new Map<NDKEventId, NDKEvent>();

        const markLatestActivity = (event: NDKEvent, topNoteId: NDKEventId) => {
            const latestActivity = perNoteLatestActivity.get(topNoteId) ?? 0;

            if (event.created_at! > latestActivity)
                perNoteLatestActivity.set(topNoteId, event.created_at!);
        }

        for (const event of $feed.values()) {
            if (event.kind === NDKKind.GroupNote || event.kind === NDKKind.Text) {
                topLevelNotes.set(event.id, event);
                markLatestActivity(event, event.id);
            } else if (event.kind === NDKKind.GroupReply) {
                event.getMatchingTags("e").forEach(tag => {
                    const parentId = tag[1];
                    markLatestActivity(event, parentId);
                });
            }
        }

        return Array.from(topLevelNotes.values()).sort((a, b) => {
            const aLatest = perNoteLatestActivity.get(a.id) ?? 0;
            const bLatest = perNoteLatestActivity.get(b.id) ?? 0;

            return bLatest - aLatest;
        });
    });

    let newPostOpened = false;
</script>

<div class="flex flex-col w-full justify-stretch">

    <div class="discussion-wrapper w-full flex flex-col">
        <div class="w-full bg-white/5">
            <NewPost creatorUser={user} bind:opened={newPostOpened} />
        </div>
        {#each $renderFeed as event, i (event.id)}
            <ForumFeedItem
                creatorUser={user}
                {event}
                position={i}
                mostRecentActivity={perNoteLatestActivity.get(event.id)}
                skipReply={true}
            />
        {/each}
    </div>
</div>

<style lang="postcss">
    :global(.article p) {
        @apply font-light text-white text-opacity-60 text-lg leading-7;
    }
</style>
