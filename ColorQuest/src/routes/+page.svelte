<script>
    import {onMount} from 'svelte';
    let width,height;
    let s=0,v=100;
    let h = 0;
    let canvas;
    let output;
    let ctx;


    function pickerUpdate(){
        let colorStopsX =[ ];
        for (let i = 0; i < 5; i++) {
            // @ts-ignore
            h = (isNaN(h)) ? 0 : h;
            let rgb=convert.hsv.rgb(Number(h),(i*25),(100));
            colorStopsX.push(`rgb(${rgb[0]},${rgb[1]},${rgb[2]})`);
        }
        
    let colorStopsY =[ ];
    for (let i = 4; i > (-1) ; i--) {
        // @ts-ignore
        h = (isNaN(h)) ? 0 : h;
        let rgb=convert.hsv.rgb(Number(h),(0),(i*25));
        colorStopsY.push(`rgb(${rgb[0]},${rgb[1]},${rgb[2]})`);
    }

    ctx.clearRect(0, 0, width, height);
    // @ts-ignore
    let gradientX = ctx.createLinearGradient(0, 0, width, 0);
    // @ts-ignore
    let gradientY = ctx.createLinearGradient(0, 0, 0, height);
    
    gradientX.addColorStop(0, colorStopsX[0] );
    gradientX.addColorStop(0.25, colorStopsX[1] );
    gradientX.addColorStop(0.5, colorStopsX[2] );
    gradientX.addColorStop(0.75, colorStopsX[3] );
    gradientX.addColorStop(1, colorStopsX[4] );
    // @ts-ignore
    ctx.fillStyle = gradientX;
    // @ts-ignore
    ctx.fillRect(0, 0, width, height);
    // @ts-ignore
    ctx.globalCompositeOperation = 'multiply';

    gradientY.addColorStop(0, colorStopsY[0] );
    gradientY.addColorStop(0.25, colorStopsY[1] );
    gradientY.addColorStop(0.5, colorStopsY[2] );
    gradientY.addColorStop(0.75, colorStopsY[3] );
    gradientY.addColorStop(1, colorStopsY[4] );
    // @ts-ignore
    ctx.fillStyle = gradientY;
    // @ts-ignore
    ctx.fillRect(0, 0, width, height);
    // @ts-ignore
    ctx.globalCompositeOperation = 'source-over';
    }

    function handleClick(e) {
        // x=e.offsetX;
        // y=e.offsetY;
        // @ts-ignore
        s=(e.offsetX/width)*100;
        // @ts-ignore
        v=100 - (e.offsetY/height)*100;
        s=Math.round(s);
        v=Math.round(v);
        pickerUpdate();
    }

    function handleHueChange() {
        pickerUpdate();
    }

    onMount(() => {
        // @ts-ignore
        width = canvas.offsetWidth;
        // @ts-ignore
        height = canvas.offsetHeight;
        // @ts-ignore
        ctx = canvas.getContext('2d');
        pickerUpdate();
    })

</script>

<div>
    <div class="w-fit">
        <h1 id="silder"class="text-3xl font-bold text-white m-6">ColorQuest</h1>
    </div>
    <div class="grid place-items-center">
        <div class="flex md:flex-row flex-col gap-4 p-4 items-center">
            <div class="border-2">
                <canvas bind:this={canvas} on:click={handleClick}></canvas>
            </div>
            <div>
                <canvas bind:this={output} width="100" height="100"></canvas>
            </div>
        </div>
        <label for="silder" class="p-4">
            <input on:change={handleHueChange} type="range" min="0" max="360" bind:value={h} class="w-[300px]">
        </label>
        <div>
            <label for="hex" class="text-white items-center flex flex-col md:flex-row">
                HEX: 
                <input type="text" class="bg-slate-900 rounded-full m-2 p-[8px]" >
            </label>
        </div>
        <label for="rgb" class="text-white">
            RGB: 
            <input type="text" class="bg-slate-900 rounded-full m-2 p-[8px]" >
        </label>
        <label for="cmyk" class="text-white">
            CMYK: 
            <input type="text" class="bg-slate-900 rounded-full m-2 p-[8px]" >
        </label>
        <label for="hsl" class="text-white">
            HSL: 
            <input type="text" class="bg-slate-900 rounded-full m-2 p-[8px]" >
        </label>
        <label for="hsv" class="text-white">
            HSV: 
            <input type="text" class="bg-slate-900 rounded-full m-2 p-[8px]" >
        </label>

    </div>
</div>

