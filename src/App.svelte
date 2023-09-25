<script>
	const numFormat = new Intl.NumberFormat("en-US", {minimumFractionDigits: 1, maximumFractionDigits: 1,signDisplay:"always"});
	const alignments = [
		'Min',
		'Mid',
		'Max',
	];

	const scalings = [
		'meet',
		'slice',
		'none',
	]

	const publicCode = 
`let viewBox = {
	minX: -150,
	minY: -180,
	width: 1200,
	height: 400,
	alignmentX: 'Mid',
	alignmentY: 'Mid',
	scaling: 'meet',
}

function scaleViewBox(viewBox, elementWidth, elementHeight) {
	if(viewBox.scaling === 'none') {
		return {
			minX: viewBox.minX,
			minY: viewBox.minY,
			width: viewBox.width,
			height: viewBox.height,
		}
	} else {
		const factor = {
			'meet': Math.max,
			'slice': Math.min
		}[viewBox.scaling].call(Math, viewBox.width/elementWidth, viewBox.height/elementHeight)

		const actualWidth = elementWidth * factor
		const actualHeight = elementHeight * factor
		const extraWidth = actualWidth - viewBox.width
		const extraHeight = actualHeight - viewBox.height

		const alignmentWeights = {
			'Min': 0,
			'Mid': 0.5,
			'Max': 1,
		};
		
		const extraWeightingX = alignmentWeights[viewBox.alignmentX];
		const extraWeightingY = alignmentWeights[viewBox.alignmentY];

		return {
			minX:  viewBox.minX - extraWeightingX * extraWidth,
			minY: viewBox.minY - extraWeightingY * extraHeight,
			width: actualWidth,
			height: actualHeight,
		}
	}
}

function screenToSVG(x,y,elementX, elementY, elementWidth, elementHeight, localWidth, localHeight, viewBox) {
	const scaledVB = scaleViewBox(viewBox, localWidth, localHeight)

	return {
		x: scaledVB.minX + scaledVB.width * ((x - elementX) / elementWidth),
		y: scaledVB.minY + scaledVB.height * ((y - elementY) / elementHeight),
	}
}

function onMouseEvent(evt) {
	const target = mouseDownTarget;
	const svgRect = svg.getBoundingClientRect();
	const local = screenToSVG(
		evt.clientX, evt.clientY,
		svgRect.left, svgRect.top, 
		svgRect.width, svgRect.height, 
		svg.clientWidth, svg.clientHeight, 
		viewBox
	)
  //... local.x, local.y ...
}

function aspectRatioString(viewBox) {
	if(viewBox.scaling === 'none') {
		return 'none'
	} else {
		return \`x\${viewBox.alignmentX}Y\${viewBox.alignmentY} \${viewBox.scaling}\`
	}
}


function viewBoxString(viewBox) {
	return \`\${viewBox.minX} \${viewBox.minY} \${viewBox.width} \${viewBox.height}\`
}
`
	
	let viewBox = {
		minX: -150,
		minY: -180,
		width: 1200,
		height: 400,
		alignmentX: 'Mid',
		alignmentY: 'Mid',
		scaling: 'meet',
	}

	let element =  {
		width: 400,
		height: 400,
		scaleX: 1,
		scaleY: 1,
	}

	let circlePos = {
		x:0,
		y:0,
	}

	function scaleViewBox(viewBox, elementWidth, elementHeight) {
		if(viewBox.scaling === 'none') {
			return {
				minX: viewBox.minX,
				minY: viewBox.minY,
				width: viewBox.width,
				height: viewBox.height,
			}
		} else {
			const relWidth = viewBox.width/elementWidth
			const relHeight = viewBox.height/elementHeight
			
			const factor = {
				'meet': Math.max,
				'slice': Math.min
			}[viewBox.scaling].call(Math, relWidth, relHeight)

			const actualWidth = elementWidth * factor
			const actualHeight = elementHeight * factor
			const extraWidth = actualWidth - viewBox.width
			const extraHeight = actualHeight - viewBox.height

			const alignmentWeights = {
				'Min': 0,
				'Mid': 0.5,
				'Max': 1,
			};
			
			const extraWeightingX = alignmentWeights[viewBox.alignmentX];
			const extraWeightingY = alignmentWeights[viewBox.alignmentY];

			return {
				minX:  viewBox.minX - extraWeightingX * extraWidth,
				minY: viewBox.minY - extraWeightingY * extraHeight,
				width: actualWidth,
				height: actualHeight,
			}
		}
	}

	function screenToSVG(x,y,elementX, elementY, elementWidth, elementHeight, localWidth, localHeight, viewBox) {
			const offsetX = x - elementX
			const offsetY = y - elementY
			const relativeX = offsetX / elementWidth
			const relativeY = offsetY / elementHeight

			const scaledVB = scaleViewBox(viewBox, localWidth, localHeight)
		  //console.log(elementWidth, elementHeight)
		  //console.log(localWidth, localHeight)

			return {
				x: scaledVB.minX + scaledVB.width * relativeX,
				y: scaledVB.minY + scaledVB.height * relativeY,
			}
	}

	let mouseDownTarget = null
	
	function onMouseDown(evt) {
		evt.preventDefault()
		mouseDownTarget = evt.currentTarget
		onMouseMove(evt)
	}

	function onMouseUp() {
		mouseDownTarget = null
	}

	function onMouseMove(evt) {
		if(!mouseDownTarget) {
			return
		}
		const target = mouseDownTarget;
		const targetRect = target.getBoundingClientRect();
		//console.log(target)
		var local = screenToSVG(
			evt.clientX, evt.clientY,
			targetRect.left, targetRect.top, 
			targetRect.width, targetRect.height, 
			target.clientWidth, target.clientHeight, 
			viewBox
		)
		circlePos.x = local.x
		circlePos.y = local.y
	}

	function aspectRatioString(viewBox) {
		if(viewBox.scaling === 'none') {
			return 'none'
		} else {
			return `x${viewBox.alignmentX}Y${viewBox.alignmentY} ${viewBox.scaling}`
		}
	}


	function viewBoxString(viewBox) {
		return `${viewBox.minX} ${viewBox.minY} ${viewBox.width} ${viewBox.height}`
	}
</script>

<style>
	svg {
		outline: 1px solid #0cf;
		background: #00ccff0a;
		display: block;
	}

	.radio-list {
		display: flex;
		gap: 0.5em 1em;
	}

	label:has(> input) {
		cursor: pointer;
	}

	input {
		margin: 0;
		padding: 0;
	}

	.fourgrid {
		display: grid;
		grid-template-rows: auto auto;
		grid-template-columns: auto auto;
		justify-content: start;
		align-content: start;
		align-items: start;
		justify-items: start;
		gap: 1em;
	}

	.canvas {
		position: absolute;
		inset: 0;
		padding-left: 20em;
		display: grid;
		justify-content: center;
		align-content: center;
	}

	.control-panel {
		z-index: 10;
		position: absolute;
		inset: 0;
		padding: 0.5em 1em 1em;
		background: #fffc;
		border: 2px solid #aaaa;
		margin: 1em;
		max-width: 25em;
		overflow: auto;
	}

	h2 {
		margin: 0;
	}

	pre {
		display: block;
		border: 1px solid gray;
		padding: 1em;
		overflow: auto;
	}

	details {
		margin-top: 2em
	}

	summary {
		cursor: pointer 
	}
</style>

<div class="control-panel">
	<h2>Manual SVG ViewBox calculation</h2>
	<p>
		This is an experiment to transform coordinates from the 
		viewport space (clientX, clientY) into the local SVG without using  
		<code>svg.getScreenCTM()</code>. The reason for not using <code>getScreenCTM()</code> is to be able to do the calculations without updating the SVG DOM element.
	</p>
	<p>
		Configure the SVG element and ViewBox size and alignment as you like and then click inside the SVG. The red dot shoult be placed correctly at the click position.
	</p>
	<div class="radio-list">
		<label>
			ViewBox Width:<br>
			<input type="range" bind:value={viewBox.width} min="100" max="1200" step="100" />
		</label>
		<label>
			ViewBox Height:<br>
			<input type="range" bind:value={viewBox.height} min="100" max="1200" step="100" />
		</label>
	</div>
	<div class="radio-list">
		<label>
			Element Width:<br>
			<input type="range" bind:value={element.width} min="100" max="700" step="100" />
		</label>
		<label>
			Element Height:<br>
			<input type="range" bind:value={element.height} min="100" max="700" step="100" />
		</label>
	</div>
	<div class="radio-list">
		<label>
			CSS Scale X({element.scaleX}):<br>
			<input type="range" bind:value={element.scaleX} min="0.1" max="2" step="0.1" />
		</label>
		<label>
			CSS Scale Y ({element.scaleY}):<br>
			<input type="range" bind:value={element.scaleY} min="0.1" max="2" step="0.1" />
		</label>
	</div>
<br>


<div class="fourgrid">
	<strong style:align-self="center" style:justify-self="center" style:transform="rotate(-20deg)">Alignment</strong>

<div class="radio-list">
	{#each alignments as a}
		<label><input style:accent-color="#05f" type="radio" value={a} bind:group={viewBox.alignmentX} /> X{a}</label>
	{/each}
</div>
<div class="radio-list" style:flex-direction="column">
	{#each alignments as a}
		<label><input style:accent-color="#05f" type="radio" value={a} bind:group={viewBox.alignmentY} /> Y{a}</label>
	{/each}
</div>
	<div>
		<strong>Scaling</strong>
<div class="radio-list">
	{#each scalings as s}
		<label><input style:accent-color="#05f" type="radio" value={s} bind:group={viewBox.scaling} /> {s}</label>
	{/each}
</div>
	</div>
</div>
<details>
	<summary>Read the Code</summary>
	<pre>{publicCode}</pre>
</details>

<center>
	<a href="https://tools.laszlokorte.de">More Educational tools</a>
</center>
</div>

<svelte:window on:mousemove={onMouseMove} on:mouseup={onMouseUp} />


<div class="canvas">
	<svg style:transform="scale({element.scaleX}, {element.scaleY})" role="button" tabindex="0" on:keydown|preventDefault on:mousedown={onMouseDown} width={element.width} height={element.height} viewBox={viewBoxString(viewBox)} preserveAspectRatio={aspectRatioString(viewBox)}>
		<g pointer-events="none">
			<rect vector-effect="non-scaling-stroke" stroke="#f80" stroke-dasharray="3 3" x="{viewBox.minX}" y="{viewBox.minY}" width="{viewBox.width}" height="{viewBox.height}" fill="#ff880033"></rect>
			<circle cx={circlePos.x} cy={circlePos.y} r="10" fill="#f08"></circle>
			<text x={circlePos.x} y={circlePos.y- 20} fill="#000" text-anchor="middle" font-size="40">({numFormat.format(circlePos.x)}, {numFormat.format(circlePos.y)})</text>
		</g>
	</svg>
</div>