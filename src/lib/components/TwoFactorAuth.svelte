<script lang="ts">
	import {onMount} from 'svelte';
	import IconLock from '~icons/mdi/lock';

	let digits: string[] = Array(6).fill('');
	let inputRefs: HTMLInputElement[] = [];
	let remainingDigits = 6;
	let isError = false;
	let isSuccess = false;
	let isShaking = false;
	let isAnimating = false;

	const handleInput = (index: number, event: Event) => {
		const input = event.target as HTMLInputElement;
		const value = input.value;

		// Only allow numbers
		if (!/^\d*$/.test(value)) {
			input.value = digits[index];
			return;
		}

		// Handle backspace
		if (value === '') {
			digits[index] = '';
			if (index > 0) {
				inputRefs[index - 1].focus();
			}
		}
		// Handle regular input
		else {
			digits[index] = value.slice(-1);
			if (index < 5) {
				inputRefs[index + 1].focus();
			}
		}

		digits = [...digits];
		remainingDigits = 6 - digits.filter((d) => d !== '').length;

		// Check if all digits are filled
		if (remainingDigits === 0) {
			validateCode();
		}
	};

	const handleKeydown = (index: number, event: KeyboardEvent) => {
		if (event.key === 'Backspace' && index > 0 && digits[index] === '') {
			inputRefs[index - 1].focus();
		} else if (event.key === 'ArrowLeft' && index > 0) {
			inputRefs[index - 1].focus();
		} else if (event.key === 'ArrowRight' && index < 5) {
			inputRefs[index + 1].focus();
		}
	};

	const handlePaste = (event: ClipboardEvent) => {
		event.preventDefault();
		const pastedData = event.clipboardData?.getData('text');
		if (!pastedData || !/^\d{1,6}$/.test(pastedData)) return;

		digits = [...pastedData.slice(0, 6).padEnd(6, '').split('')];
		remainingDigits = 6 - digits.filter((d) => d !== '').length;

		if (remainingDigits === 0) {
			validateCode();
		}
	};

	function validateCode() {
		const code = digits.join('');
		if (code === '123456') {
			handleSuccess();
		} else {
			handleError();
		}
	}

	const handleSuccess = () => {
		isSuccess = true;
		setTimeout(() => {
			isAnimating = true;
		}, 50);
	};

	const handleError = () => {
		isError = true;
		isShaking = true;

		if (window.navigator.vibrate) {
			window.navigator.vibrate(100);
		}

		setTimeout(() => {
			isShaking = false;
		}, 400);

		const firstEmptyIndex = digits.findIndex((d) => d === '');
		const focusIndex = firstEmptyIndex === -1 ? 0 : firstEmptyIndex;
		inputRefs[focusIndex]?.focus();
	};

	const reset = () => {
		digits = Array(6).fill('');
		remainingDigits = 6;
		isError = false;
		isSuccess = false;
		inputRefs[0].focus();
	};

	const getIconColorClass = () => {
		if (isSuccess) return 'text-green-500';
		if (isError) return 'text-red-500';
		return 'text-blue-500';
	};

	onMount(() => {
		inputRefs[0].focus();
	});
</script>

<div class="flex max-w-md flex-col items-center gap-[12px] rounded-3xl bg-white p-8 shadow-lg">
	<div class="relative">
		<!-- Pulse animation container -->
		{#if isSuccess || isError}
			<div
				class:bg-green-100={isSuccess}
				class:bg-red-100={isError}
				class="absolute inset-0 animate-pulse rounded-full"
			></div>
		{/if}

		<!-- Lock icon container -->
		<div
			class:bg-blue-100={!isSuccess && !isError}
			class:bg-green-100={isSuccess}
			class:bg-red-100={isError}
			class="relative flex h-20 w-20 items-center justify-center rounded-full"
		>
			<IconLock class="h-8 w-8 {getIconColorClass()}" />
		</div>
	</div>

	<h1 class="mb-[12px] text-3xl font-bold">Easy peasy</h1>
	<p class="mb-[24px] text-sm text-gray-500">Enter 6-digit code from your two factor authenticator APP.</p>

	<!-- Input Grid -->
	<div class="mb-6 grid grid-cols-6 gap-3 {isShaking ? 'animate-error' : ''}">
		{#each digits as digit, i}
			<input
				bind:this={inputRefs[i]}
				type="text"
				inputmode="numeric"
				maxlength="1"
				value={digit}
				on:input={(e) => handleInput(i, e)}
				on:keydown={(e) => handleKeydown(i, e)}
				on:paste={handlePaste}
				class="h-14 w-12 rounded-lg border-2 text-center text-xl font-semibold focus:outline-none"
				class:border-blue-500={digit === '' && remainingDigits > 0}
				class:border-green-500={isSuccess}
				class:border-red-500={isError}
			/>
		{/each}
	</div>

	<!-- Status Message -->
	{#if !isSuccess && !isError}
		<div class="w-full rounded-lg bg-gray-100 px-6 py-3 text-center text-gray-500">
			{remainingDigits} digits left
		</div>
	{:else if isSuccess}
		<button
			on:click={() => reset()}
			class="success-button w-full cursor-pointer rounded-lg border-2 border-blue-500 px-6 py-3 {isAnimating
				? 'text-white'
				: 'text-blue-500'}"
			class:animate={isAnimating}
		>
			Let's go!
		</button>
	{:else}
		<button
			on:click={() => reset()}
			class=" w-full cursor-pointer rounded-lg bg-red-500 px-6 py-3 text-white {isShaking ? 'animate-error' : ''}"
		>
			Wrong code!
		</button>
	{/if}
</div>

<style>
	.animate-pulse {
		animation: pulse 1.2s ease;
	}

	@keyframes pulse {
		0% {
			opacity: 1;
			transform: scale(1);
		}

		100% {
			opacity: 0;
			transform: scale(1.8);
		}
	}

	.animate-error {
		animation: shake 0.4s cubic-bezier(0.36, 0.07, 0.19, 0.97) both;
	}

	@keyframes shake {
		0%,
		100% {
			transform: translateX(0);
		}
		20% {
			transform: translateX(-4px);
		}
		40% {
			transform: translateX(4px);
		}
		60% {
			transform: translateX(-4px);
		}
		80% {
			transform: translateX(4px);
		}
	}

	.success-button {
		position: relative;
		isolation: isolate;
		overflow: hidden;
	}

	.success-button::after {
		content: '';
		position: absolute;
		inset: 0;
		background-color: rgb(59 130 246);
		transform: translateX(100%);
		transition: transform 0.5s ease;
		z-index: -1;
	}

	.success-button.animate::after {
		transform: translateX(0);
	}

	input::-webkit-outer-spin-button,
	input::-webkit-inner-spin-button {
		-webkit-appearance: none;
		margin: 0;
	}
</style>
