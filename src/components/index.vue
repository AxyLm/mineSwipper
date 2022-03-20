<template>
	<div>
		<main class>
			<div v-for="row in gridList" class="flex flex-1 justify-center">
				<button class="block border w-10 h-10" v-for="grid in row" @click="onClick(grid)">
					<template v-if="grid.mine">*</template>
					<template v-else>{{grid.x}}</template>
				</button>
			</div>
		</main>
	</div>
</template>
<script setup lang="ts">
	import { ref } from "vue";
	interface Grid {
		x: number;
		y: number;
		revealed: boolean;
		mine?: boolean;
		flagged?: boolean;
		adjacentMines: number;
	}

	const WIDTH = 10;
	const HEIGHT = 10;

	let gridList: Grid[][] = ref();

	function initGrid() {
		gridList = Array.from(new Array(WIDTH), (_, x) => {
			return Array.from(new Array(HEIGHT), (_, y) => {
				return {
					x,
					y,
					revealed: false,
					adjacentMines: 0,
				};
			});
		});
	}

	function onClick(grid: Grid) {
		console.log(grid.x);
	}
	console.log(gridList);
</script>