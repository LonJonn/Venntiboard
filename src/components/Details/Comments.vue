<template>
	<div class="mt-5">
		<input v-model="content" type="text" placeholder="type comment here..." />
		<br />
		<button @click="addComment()" class="text-blue-600">Add Comment</button>
		<ul v-for="comment in comments" :key="comment.id" class="p-3 rounded-lg bg-gray-200 mt-3">
			<li>content: {{ comment.content }}</li>
			<li>posted: {{ comment.createdAt.toDate() }}</li>
			<li>
				by:
				<img
					v-bind:src="comment.owner.photoUrl"
					width="30px"
					alt="User Icon"
					class="inline rounded-full"
				/>
				{{ comment.owner.displayName }}
			</li>
			<li
				v-if="isOwner(comment.owner.id)"
				@click="deleteComment(comment.id)"
				class="text-red-500 cursor-pointer"
			>Delete</li>
		</ul>
	</div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";
import { db, auth } from "@/firebase";
import { firestore } from "firebase/app";
import * as AppTypes from "@/typings";

@Component
export default class CommentsComponent extends Vue {
	// Props
	@Prop() ideaId: string;

	// State
	content = "";
	comments: AppTypes.Comment[] = [];
	ref = db
		.collection("ideas")
		.doc(this.ideaId)
		.collection("comments");

	// Hooks
	async mounted() {
		await this.$bind("comments", this.ref.orderBy("createdAt"));
		// this.$emit("loaded");
	}

	// Methods
	async addComment() {
		if (!this.content) return alert("Can't have empty comment!");

		const newLike: AppTypes.Comment = {
			createdAt: firestore.Timestamp.now(),
			owner: {
				id: auth.currentUser.uid,
				displayName: auth.currentUser.displayName,
				photoUrl: auth.currentUser.photoURL
			},
			content: this.content
		};

		this.content = "";

		await this.ref.add(newLike);
	}

	async deleteComment(commentId: string) {
		await this.ref.doc(commentId).delete();
	}

	isOwner(id: string) {
		return auth.currentUser.uid === id;
	}
}
</script>
