<script lang="ts">
    import { onDestroy, onMount } from "svelte";


	type Complex = {
		r: number;
		i: number;
	};
	type ComplexPolar = {
		r: number;
		theta: number;
	};

	let cw = $state(0);
	let Q = 1;
	let W = $derived(Q*cw)
	let H = $derived(W*3/4);
	let Z = $derived(Math.round(W/100));

	let canvas: HTMLCanvasElement|undefined = $state();
	let ctx: CanvasRenderingContext2D|undefined = $state();
	
	let speed = $state(0.001);
	let paused = $state(false);

	let points: ComplexPolar[] = $state([
	])
	let pointsArithmetic: Complex[] = $derived(points.map(polarToComplex));

	let cs: ComplexPolar[] = $state([]);
	let t = $state(0);
	let final: Complex[] = $state([]);

	let bgImg: HTMLImageElement|undefined = $state();

	$effect(() => {
		if(!canvas) return;
		canvas.width = W;
		canvas.height = H;
		const context = canvas.getContext('2d');
		if(!context) return;
		ctx = context;
	});

	onMount(() => {
		restart();
		window.requestAnimationFrame(draw);
	});


	function restart() {
		calculateCs();
		t = 0;
		prepareFinal();
	}

	function calculateCs() {
		console.log(points);
		
		cs = [];
		for(let i = 0; i < points.length; i++) {
			const k = getSequenceItem(i);
			cs.push(ck(k));
		}
		
		
	}

	function ck(k: number): ComplexPolar {
		let sum: Complex = {r: 0, i: 0};
		for(let j = 0; j < points.length; j++) {
			const p = points[j];
			const cp: ComplexPolar = {
				r: p.r,
				theta: p.theta - k*2*Math.PI*j/points.length,
			}
			const c1 = polarToComplex(cp);
			sum.r += c1.r;
			sum.i += c1.i;
		}
		return complexToPolar({
			r: sum.r / points.length,
			i: sum.i / points.length,
		});
	}

	function draw() {
		window.requestAnimationFrame(draw);

		if(!ctx) return;
		if(!canvas) return;
		if(paused) return;
		
		t += speed;

		ctx.clearRect(0, 0, canvas.width, canvas.height);

		if(bgImg) {
			const ratio = bgImg.width / bgImg.height;
			let newW = W;
			let newH = H;
			if(ratio > W/H) {
				newH = W/ratio;
			} else {
				newW = H*ratio;
			}
			ctx.drawImage(bgImg, W/2-newW/2, H/2-newH/2, newW, newH);
		}

		ctx.strokeStyle = '#8ecae6';
		ctx.fillStyle = '#219ebc';
		drawCircles(t);

		ctx.strokeStyle = '#ffb703';
		ctx.beginPath();
		ctx.moveTo(W/2+final[0].r*Z, H/2+final[0].i*Z);
		for(const c of final) {
			ctx.lineTo(W/2+c.r*Z, H/2+c.i*Z);
		}
		ctx.lineTo(W/2+final[0].r*Z, H/2+final[0].i*Z);
		ctx.stroke();

		ctx.fillStyle = '#fb8500';
		for(const pt of pointsArithmetic) {
			dot(pt.r, pt.i);
		}
		
	}

	function prepareFinal() {
		final = [];
		for(let i = 0; i < 1; i += 0.01) {
			const c = drawCircles(i);
			if(c) final.push(c);
		}
	}

	function drawCircles(time: number): Complex|undefined {
		if(!ctx) return;
		
		let center: Complex = {r: 0, i: 0};
		for(let i = 0; i < cs.length; i++) {
			const c = cs[i];
			const j = getSequenceItem(i);
			circle(center.r, center.i, c.r);
			const dc: Complex = polarToComplex({r: c.r, theta: c.theta + j*time*2*Math.PI});
			center.r += dc.r;
			center.i += dc.i;
			if(i === cs.length-1) {
				ctx.fillStyle = '#8ecae6';
				dot(center.r, center.i, 8);
			}
			else dot(center.r, center.i);
		}
		return center;
	}

	function circle(cx: number, cy: number, r: number) {
		if(!ctx) return;
		ctx.beginPath();
		ctx.arc(W/2+cx*Z, H/2+cy*Z, r*Z, 0, 2 * Math.PI);
		ctx.stroke();
	}

	function dot(cx: number, cy: number, size = 4) {
		if(!ctx) return;
		ctx.beginPath();
		ctx.arc(W/2+cx*Z, H/2+cy*Z, size, 0, 2 * Math.PI);
		ctx.fill();
	}

	function getSequenceItem(i: number): number {
		if (i === 0) return 0;
		return i % 2 === 0 ? i / 2 : -(i + 1) / 2;
    }

	function polarToComplex(p: ComplexPolar): Complex {
		return {
			r: p.r * Math.cos(p.theta),
			i: p.r * Math.sin(p.theta),
		};
	}

	function complexToPolar(c: Complex): ComplexPolar {
		return {
			r: Math.sqrt(c.r*c.r + c.i*c.i),
			theta: Math.atan2(c.i, c.r),
		};
	}

	function addPoint(e: MouseEvent) {
		const arith: Complex = {
			r: (e.offsetX*Q - W/2) / Z,
			i: (e.offsetY*Q - H/2) / Z,
		}
		points.push(complexToPolar(arith));
		restart();
	}

	function savePoints() {
		const data = JSON.stringify(points);
		const blob = new Blob([data], {type: 'text/plain'});
		const url = URL.createObjectURL(blob);
		const a = document.createElement('a');
		a.href = url;
		a.download = 'points.json';
		a.click();
		URL.revokeObjectURL(url);
	}

	function loadPoints() {
		const input = document.createElement('input');
		input.type = 'file';
		input.accept = '.json';
		input.onchange = (e) => {
			const file = (e.target as HTMLInputElement).files?.[0];
			if(!file) return;
			const reader = new FileReader();
			reader.onload = (e) => {
				const data = e.target?.result;
				if(!data) return;
				points = JSON.parse(data as string);
				restart();
			}
			reader.readAsText(file);
		}
		input.click();
	}

	function uploadBackgroundImage() {
		const input = document.createElement('input');
		input.type = 'file';
		input.accept = 'image/*';
		input.onchange = (e) => {
			const file = (e.target as HTMLInputElement).files?.[0];
			if(!file) return;
			const reader = new FileReader();
			reader.onload = (e) => {
				const data = e.target?.result;
				if(!data) return;
				const img = new Image();
				img.src = data as string;
				bgImg = img;
			}
			reader.readAsDataURL(file);
		}
		input.click();
	}

</script>

<div>
	<canvas bind:this={canvas} bind:clientWidth={cw} onclick={addPoint}></canvas>
	<div class="buttons">
		<button onclick={() => {
			points = [];
			restart();
		}}>Clear</button>
		<button onclick={() => paused = false}>Play</button>
		<button onclick={() => paused = true}>Pause</button>
		<button onclick={() => speed = Number(prompt("Speed"))/10000}>Change speed</button>
		<button onclick={() => {if(points.length>0) points.pop(); restart()}}>Undo</button>
		<button onclick={savePoints}>Save</button>
		<button onclick={loadPoints}>Load</button>
		<button onclick={uploadBackgroundImage}>Background</button>
		<button onclick={() => bgImg = undefined}>Remove Background</button>
	</div>
</div>

<style>

	canvas {
		background: #023047;
		height: 70vh;
		aspect-ratio: 4/3;
	}

</style>