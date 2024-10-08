<script>
    import {onMount} from 'svelte';
    import convert from 'color-convert';
    let width,height;
    let s=0,v=100;
    let h = 0;
    let canvas;
    let output;
    let ctx;
    let hex,rgb,cmyk,hsl,hsv;

    function circleUpdate(){// updates the position of the circle on the canvas
        ctx.globalCompositeOperation = 'overlay';
        ctx.beginPath();
        ctx.arc((s*width/100),((100-v)*height/100),5,0,2*Math.PI);
        ctx.strokeStyle = 'black';
        ctx.lineWidth = 1;
        ctx.stroke();
        ctx.globalCompositeOperation = 'source-over';
    }

    function codeUpdate(){// updates different color codes from the values of h,s,v
        h = (isNaN(h)) ? 0 : h;
        let hexCode = convert.hsv.hex(h,s,v);
        hex= `#${hexCode}`;
        rgb= `${convert.hsv.rgb(h,s,v)[0]}, ${convert.hsv.rgb(h,s,v)[1]}, ${convert.hsv.rgb(h,s,v)[2]}`;
        cmyk= `${convert.hsv.cmyk(h,s,v)[0]}%, ${convert.hsv.cmyk(h,s,v)[1]}%, ${convert.hsv.cmyk(h,s,v)[2]}%, ${convert.hsv.cmyk(h,s,v)[3]}%`;
        hsl= `${convert.hsv.hsl(h,s,v)[0]}°, ${convert.hsv.hsl(h,s,v)[1]}%, ${convert.hsv.hsl(h,s,v)[2]}%`;
        hsv= `${h}°, ${s}%, ${v}%`;
    }

    function pickerUpdate(){// updates the canvas based on the value of h
        let colorStopsX =[ ]; // contains the color stops of the saturation gradient
        for (let i = 0; i < 5; i++) { 
            h = (isNaN(h)) ? 0 : h;
            let rgb=convert.hsv.rgb(Number(h),(i*25),(100));
            colorStopsX.push(`rgb(${rgb[0]},${rgb[1]},${rgb[2]})`);
        }
        
        let colorStopsY =[ ]; // contains the color stops of the value gradient
        for (let i = 4; i > (-1) ; i--) {
            h = (isNaN(h)) ? 0 : h;
            let rgb=convert.hsv.rgb(Number(h),(0),(i*25));
            colorStopsY.push(`rgb(${rgb[0]},${rgb[1]},${rgb[2]})`);
        }

        // clearing the canvas
        ctx.clearRect(0, 0, width, height);

        // creating the gradient along x and y axis
        let gradientX = ctx.createLinearGradient(0, 0, width, 0);
        let gradientY = ctx.createLinearGradient(0, 0, 0, height);

        // adding the colors to the saturation gradient
        gradientX.addColorStop(0, colorStopsX[0] );
        gradientX.addColorStop(0.25, colorStopsX[1] );
        gradientX.addColorStop(0.5, colorStopsX[2] );
        gradientX.addColorStop(0.75, colorStopsX[3] );
        gradientX.addColorStop(1, colorStopsX[4] );
        ctx.fillStyle = gradientX;
        ctx.fillRect(0, 0, width, height);// drawing the saturation gradient on the canvas

        ctx.globalCompositeOperation = 'multiply'; /* "multiply" multplies the new color with the old one 
                                                    i.e. the resulting color comes out to be more natural looking */

        // adding the colors to the value gradient
        gradientY.addColorStop(0, colorStopsY[0] );
        gradientY.addColorStop(0.25, colorStopsY[1] );
        gradientY.addColorStop(0.5, colorStopsY[2] );
        gradientY.addColorStop(0.75, colorStopsY[3] );
        gradientY.addColorStop(1, colorStopsY[4] );
        ctx.fillStyle = gradientY;
        ctx.fillRect(0, 0, width, height);// drawing the value gradient on the canvas

    }

    function handleClick(e) {// updates the values of h,s,v when the user clicks on the canvas
        s=(e.offsetX/width)*100;
        v=100 - (e.offsetY/height)*100;
        s=Math.round(s);
        v=Math.round(v);
        codeUpdate();
        pickerUpdate();
        circleUpdate();
    }

    function handleHueChange() {// updates the canvas when value of h changes
        pickerUpdate();
        codeUpdate();
        circleUpdate();
    }

    onMount(() => {// initializing the canvas and getting dimensions of the canvas
        width = canvas.offsetWidth;
        height = canvas.offsetHeight;
        ctx = canvas.getContext('2d');
        codeUpdate();
        pickerUpdate();
        circleUpdate();
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
            <input on:change={handleHueChange} type="range" min="0" max="360" bind:value={h} class="w-[300px] ">
        </label>
        <div>
            <label for="hex" class="text-white items-center flex flex-col md:flex-row">
                HEX: 
                <input type="text" class="bg-slate-900 rounded-full m-2 p-[8px]" value={hex} >
            </label>
        </div>
        <label for="rgb" class="text-white">
            RGB: 
            <input type="text" class="bg-slate-900 rounded-full m-2 p-[8px]" value={rgb}>
        </label>
        <label for="cmyk" class="text-white">
            CMYK: 
            <input type="text" class="bg-slate-900 rounded-full m-2 p-[8px]" value={cmyk}>
        </label>
        <label for="hsl" class="text-white">
            HSL: 
            <input type="text" class="bg-slate-900 rounded-full m-2 p-[8px]" value={hsl}>
        </label>
        <label for="hsv" class="text-white">
            HSV: 
            <input type="text" class="bg-slate-900 rounded-full m-2 p-[8px]" value={hsv}>
        </label>

    </div>
</div>