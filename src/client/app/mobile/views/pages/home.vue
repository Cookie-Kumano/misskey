<template>
<mk-ui>
	<template #func>
		<button @click="fn"><fa icon="pencil-alt"/></button>
	</template>

	<main>

		<header class="zahtxcqi">
				<div :data-active="src == 'home'" @click="src = 'home'"><fa icon="home"/> </div>
				<div :data-active="src == 'local'" @click="src = 'local'" v-if="enableLocalTimeline"><fa :icon="['far', 'comments']"/> </div>
				<div :data-active="src == 'hybrid'" @click="src = 'hybrid'" v-if="enableLocalTimeline"><fa icon="share-alt"/> </div>
				<div :data-active="src == 'global'" @click="src = 'global'" v-if="enableGlobalTimeline"><fa icon="globe"/> </div>
				<div :data-active="src == 'tag'" @click="src = 'tag'" v-if="tagTl"><fa icon="hashtag"/> {{ tagTl.title }}</div>
				<div :data-active="src == 'list'" @click="src = 'list'" v-if="list"><fa icon="list"/> {{ list.name }}</div>
				<div class="buttons">
					<button :data-active="src == 'mentions'" @click="src = 'mentions'" :title="$t('mentions')"><fa icon="at"/><i class="indicator" v-if="$store.state.i.hasUnreadMentions"><fa icon="circle"/></i></button>
					<button :data-active="src == 'messages'" @click="src = 'messages'" :title="$t('messages')"><fa :icon="['far', 'envelope']"/><i class="indicator" v-if="$store.state.i.hasUnreadSpecifiedNotes"><fa icon="circle"/></i></button>
					<button @click="chooseTag" :title="$t('hashtag')" ref="tagButton"><fa icon="hashtag"/></button>
					<button @click="chooseList" :title="$t('list')" ref="listButton"><fa icon="list"/></button>
				</div>
		</header>

		<div class="tl">
			<x-tl v-if="src == 'home'" ref="tl" key="home" src="home"/>
			<x-tl v-if="src == 'local'" ref="tl" key="local" src="local"/>
			<x-tl v-if="src == 'hybrid'" ref="tl" key="hybrid" src="hybrid"/>
			<x-tl v-if="src == 'global'" ref="tl" key="global" src="global"/>
			<x-tl v-if="src == 'mentions'" ref="tl" key="mentions" src="mentions"/>
			<x-tl v-if="src == 'messages'" ref="tl" key="messages" src="messages"/>
			<x-tl v-if="src == 'tag'" ref="tl" key="tag" src="tag" :tag-tl="tagTl"/>
			<mk-user-list-timeline v-if="src == 'list'" ref="tl" :key="list.id" :list="list"/>
		</div>
	</main>
</mk-ui>
</template>

<script lang="ts">
import Vue from 'vue';
import i18n from '../../../i18n';
import Progress from '../../../common/scripts/loading';
import XTl from './home.timeline.vue';

export default Vue.extend({
	i18n: i18n('mobile/views/pages/home.vue'),

	components: {
		XTl
	},

	data() {
		return {
			src: 'home',
			list: null,
			lists: null,
			tagTl: null,
			showNav: false,
			enableLocalTimeline: false,
			enableGlobalTimeline: false,
		};
	},

	watch: {
		src() {
			this.showNav = false;
			this.saveSrc();
		},

		list(x) {
			this.showNav = false;
			this.saveSrc();
			if (x != null) this.tagTl = null;
		},

		tagTl(x) {
			this.showNav = false;
			this.saveSrc();
			if (x != null) this.list = null;
		},

		showNav(v) {
			if (v && this.lists === null) {
				this.$root.api('users/lists/list').then(lists => {
					this.lists = lists;
				});
			}
		}
	},

	created() {
		this.$root.getMeta().then((meta: Record<string, any>) => {
			if (!(
				this.enableGlobalTimeline = !meta.disableGlobalTimeline || this.$store.state.i.isModerator || this.$store.state.i.isAdmin
			) && this.src === 'global') this.src = 'local';
			if (!(
				this.enableLocalTimeline = !meta.disableLocalTimeline || this.$store.state.i.isModerator || this.$store.state.i.isAdmin
			) && ['local', 'hybrid'].includes(this.src)) this.src = 'home';
		});

		if (this.$store.state.device.tl) {
			this.src = this.$store.state.device.tl.src;
			if (this.src == 'list') {
				this.list = this.$store.state.device.tl.arg;
			} else if (this.src == 'tag') {
				this.tagTl = this.$store.state.device.tl.arg;
			}
		}
	},

	mounted() {
		document.title = this.$root.instanceName;

		Progress.start();

		(this.$refs.tl as any).$once('loaded', () => {
			Progress.done();
		});
	},

	methods: {
		fn() {
			this.$post();
		},

		saveSrc() {
			this.$store.commit('device/setTl', {
				src: this.src,
				arg: this.src == 'list' ? this.list : this.tagTl
			});
		},

		warp() {

		}
	}
});
</script>

<style lang="stylus" scoped>
main
	header.zahtxcqi
		display flex
		position fixed 
		bottom 0
		left 0
		flex-wrap wrap
		padding 0 8px
		z-index 10
		background var(--faceHeader)
		box-shadow 0 var(--lineWidth) var(--desktopTimelineHeaderShadow)

		> *
			flex-shrink 0

		> .buttons
			margin-left auto

			> button
				padding 0 8px
				font-size 0.9em
				line-height 42px
				color var(--faceTextButton)

				> .indicator
					position absolute
					top -4px
					right 4px
					font-size 10px
					color var(--notificationIndicator)
					animation blink 1s infinite

				&:hover
					color var(--faceTextButtonHover)

				&[data-active]
					color var(--primary)
					cursor default

					&:before
						content ""
						display block
						position absolute
						bottom 0
						left 0
						width 100%
						height 2px
						background var(--primary)

		> div:not(.buttons)
			padding 0 10px
			line-height 42px
			font-size 12px
			user-select none

			&[data-active]
				color var(--primary)
				cursor default
				font-weight bold

				&:before
					content ""
					display block
					position absolute
					bottom 0
					left -8px
					width calc(100% + 16px)
					height 2px
					background var(--primary)

			&:not([data-active])
				color var(--desktopTimelineSrc)
				cursor pointer

				&:hover
					color var(--desktopTimelineSrcHover)

	> .nav
		> .body
			position fixed
			z-index 10001
			top 56px
			left 0
			right 0
			width 300px
			max-height calc(100% - 70px)
			margin 0 auto
			overflow auto
			-webkit-overflow-scrolling touch
			background var(--popupBg)
			border-radius 8px
			box-shadow 0 0 16px rgba(#000, 0.1)

			> div
				padding 8px 0

				> .hr
					margin 8px 0
					border-top solid 1px var(--faceDivider)

				> *:not(.hr)
					display block
					padding 8px 16px
					color var(--text)

					&[data-active]
						color var(--primaryForeground)
						background var(--primary)

					&:not([data-active]):hover
						background var(--mobileHomeTlItemHover)

					> .badge
						margin-left 6px
						font-size 10px
						color var(--notificationIndicator)

</style>

<style lang="stylus" module>
.title
	[data-icon]
		margin-right 4px

.badge
	margin-left 6px
	font-size 10px
	color var(--notificationIndicator)
	vertical-align middle

</style>
