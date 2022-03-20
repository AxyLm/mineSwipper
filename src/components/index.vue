<template>
	<div>
		<main class>
			<p class="text-center">{{mines}}</p>
			<div v-for="row in gridList" class="flex flex-1 justify-center">
				<button
					class="block border w-10 h-10 m-px"
					:class="getBlockClass(grid)"
					v-for="grid in row"
					@click="onClick(grid)"
				>
					<template v-if="grid.revealed || isDev">
						<template v-if="grid.mine">💣</template>
						<span v-else>{{grid.adjacentMines || ""}}</span>
					</template>
				</button>
			</div>
			<div class="text-center">
				<button @click="isDev = !isDev" class="border p-1 m-1">dev</button>
				<button @click="reset()" class="border p-1 m-1">reset</button>
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
	const mines = ref(0);
	const mineGenerated = ref(false);
	const gridList = ref<Grid[][]>();
	const isDev = ref(false);
	const directions = [
		[1, 1],
		[1, 0],
		[1, -1],
		[0, -1],
		[-1, -1],
		[-1, 0],
		[-1, 1],
		[0, 1],
	];

	const numberColors = [
		"text-transparent",
		"text-blue-500",
		"text-green-500",
		"text-yellow-500",
		"text-orange-500",
		"text-red-500",
		"text-purple-500",
		"text-pink-500",
		"text-teal-500",
	];
	function reset() {
		isDev.value = false;
		mines.value = 0;
		mineGenerated.value = false;
		gridList.value = Array.from({ length: WIDTH }, (_, x) =>
			Array.from(
				{ length: HEIGHT },
				(_, y): Grid => ({
					x,
					y,
					revealed: false,
					adjacentMines: 0,
				})
			)
		);
	}
	function getBlockClass(block: Grid) {
		if (block.flagged) return "bg-gray-500/10";
		if (!block.revealed) return "bg-gray-500/10 hover:bg-gray-500/20";

		return block.mine ? "bg-red-500/50" : numberColors[block.adjacentMines];
	}
	// const directions = [
	// 	[1, 1], [1, 0], [1, -1],
	// 	[0, -1],        [-1, -1],
	//   [-1, 0], [-1, 1], [0, 1],
	// ];
	function onClick(grid: Grid) {
		grid.revealed = true;
		if (!mineGenerated.value) {
			generateMines(grid);
			mineGenerated.value = true;
		} else {
			openGrid(grid);
		}
		autoOpen(grid);
	}

	function generateMines(initial: Grid) {
		const { x, y } = initial;

		for (const row of gridList.value) {
			for (const col of row) {
				// 第一次点击的四周不出现炸弹
				if (Math.abs(y - col.y) <= 1 && Math.abs(x - col.x) <= 1) {
					break;
				}
				// 炸弹出现几率
				col.mine = Math.random() < 0.2;
				if (col.mine) {
					// 记录总数
					mines.value += 1;
				}
			}
		}
		updateNumbers();
	}

	function updateNumbers() {
		for (const row of gridList.value) {
			for (const col of row) {
				if (col.mine) continue;
				getSilder(col).forEach((e) => {
					if (e.mine) {
						col.adjacentMines += 1;
					}
				});
			}
		}
	}

	function openGrid(grid: Grid) {
		if (grid.mine) {
			alert("loast");
			return;
		}
	}

	function getSilder(grid: Grid) {
		// 四周坐标循环查找
		return directions
			.map(([dx, dy]) => {
				const x1 = grid.x + dx;
				const y1 = grid.y + dy;
				// 边界跳过
				if (x1 >= WIDTH || x1 <= -1 || y1 >= HEIGHT || y1 <= -1) {
					return undefined;
				}
				return gridList.value[x1][y1];
			})
			.filter(Boolean) as Grid[];
	}

	function autoOpen(grid: Grid) {
		if (grid.adjacentMines) {
			return;
		}
		getSilder(grid).forEach((e) => {
			if (!e.revealed && !e.mine) {
				e.revealed = true;
				autoOpen(e);
			}
		});
	}

	reset();
</script>