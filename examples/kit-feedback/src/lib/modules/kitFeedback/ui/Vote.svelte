<script lang="ts">
	import { config } from '../utils/config';
	import { KQL_Issue, KQL_UpdateComment } from '$lib/graphql/_kitql/graphqlStores';
	import type { Reaction } from '$lib/graphql/_kitql/graphqlTypes';
	// import { resolveTheme, theme } from '$lib/theme';
	import type { Class } from '../utils/theme';
	import type { CommentMetadata, CommentVoteType } from '../utils/types';
	import Icon from '@iconify/svelte';

	export let type: CommentVoteType;
	export let metadataCommentId: string;
	export let metadata: CommentMetadata;
	export let votes: string[] = [];

	const classes: Record<CommentVoteType, Class> = {
		up: 'comment-vote-up',
		down: 'comment-vote-down'
	};

	const icons: Record<CommentVoteType, string> = {
		up: 'bxs:upvote',
		down: 'bxs:downvote'
	};

	$: active = votes.includes($config.identifier());

	const vote = async () => {
		const identifier = $config.identifier();
		let newMetadata: CommentMetadata = metadata;
		if (active) {
			newMetadata.votes[type] = metadata.votes[type].filter((vote) => vote !== identifier);
		} else {
			newMetadata.votes[type] = [...metadata.votes[type], identifier];
		}
		await KQL_UpdateComment.mutate({
			variables: {
				fields: {
					commentID: metadataCommentId,
					body: JSON.stringify(newMetadata)
				}
			}
		});
		await KQL_Issue.query({ settings: { policy: 'network-only' } });
	};
</script>

<!-- <div
	class="
        {resolveTheme($theme, 'comment-vote')}
        {resolveTheme($theme, active ? 'comment-vote-active' : undefined)}
        {resolveTheme($theme, classes[type])}
    "
	on:click={vote}
>
	<Icon icon={icons[type]} />
	{votes.length}
</div> -->
