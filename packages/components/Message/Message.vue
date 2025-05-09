<script setup lang="ts">
	import type { MessageProps } from './types'
	import { computed, onMounted, ref, watch } from 'vue'
	import { getLastBottomOffset } from './methods'
	import { delay, bind } from 'lodash-es'
	import { useEventListener, useOffset } from '@veyra/hooks'
	import { RenderVnode, typeIconMap } from '@veyra/utils'
	import VrIcon from '../Icon/Icon.vue'

	defineOptions({
		name: 'VrMessage',
	})

	const props = withDefaults(defineProps<MessageProps>(), {
		type: 'info',
		duration: 3000,
		offset: 10,
		transitionName: 'fade-up',
	})

	const visible = ref(false)
	const messageRef = ref<HTMLDivElement>()

	const iconName = computed(() => typeIconMap.get(props.type) ?? 'circle-info')

	// div 的高度
	const boxHeight = ref(0)

	const { topOffset, bottomOffset } = useOffset({
		getLastBottomOffset: bind(getLastBottomOffset, props),
		offset: props.offset,
		boxHeight,
	})

	const cssStyle = computed(() => ({
		top: topOffset.value + 'px',
		zIndex: props.zIndex,
	}))

	let timer: number
	function startTimmer() {
		if (props.duration === 0) return
		timer = delay(close, props.duration)
	}

	function clearTimer() {
		clearTimeout(timer)
	}

	function close() {
		visible.value = false
	}

	onMounted(() => {
		visible.value = true
		startTimmer()
	})

	useEventListener(document, 'keydown', (e: Event) => {
		const { code } = e as KeyboardEvent
		if (code === 'Escape') {
			close()
		}
	})

	watch(visible, (val) => {
		if (!val) boxHeight.value = -props.offset // 退出动画更流畅
	})

	defineExpose({
		bottomOffset,
		close,
	})
</script>

<template>
	<Transition
		:name="transitionName"
		@enter="boxHeight = messageRef!.getBoundingClientRect().height"
		@after-leave="!visible && onDestory()">
		<div
			ref="messageRef"
			class="vr-message"
			:class="{
				[`vr-message--${type}`]: type,
				'is-close': showClose,
				'text-center': center,
			}"
			:style="cssStyle"
			v-show="visible"
			role="alert"
			@mouseenter="clearTimer"
			@mouseleave="startTimmer">
			<vr-icon class="vr-message__icon" :icon="iconName" />
			<div class="vr-message__content">
				<slot>
					<render-vnode v-if="message" :vNode="message" />
				</slot>
			</div>
			<div class="vr-message__close" v-if="showClose">
				<vr-icon icon="xmark" @click.stop="close" />
			</div>
		</div>
	</Transition>
</template>

<style scoped>
	@import './style.css';
</style>
