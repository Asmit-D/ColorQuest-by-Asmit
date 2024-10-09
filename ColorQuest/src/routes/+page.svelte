<script>
    import {onMount, afterUpdate} from 'svelte';
    import convert from 'color-convert';
    let width,height;
    let s=0,v=100;
    let h = 0;
    let canvas;
    let output;
    let ctx;
    let hex,rgb,cmyk,hsl,hsv;
    let grabbed;

    function updateOutput(){// updates the output canvas
        output.getContext('2d').fillStyle = `rgb(${convert.hsv.rgb(h,s,v)[0]}, ${convert.hsv.rgb(h,s,v)[1]}, ${convert.hsv.rgb(h,s,v)[2]})`;
        output.getContext('2d').fillRect(0, 0, output.width, output.height);
    }

    function circleUpdate(){// updates the position of the circle on the canvas
        ctx.globalCompositeOperation = 'source-over';
        ctx.beginPath();
        ctx.arc((s*width/100),((100-v)*height/100),5,0,2*Math.PI);
        ctx.strokeStyle = 'black';
        ctx.lineWidth = 3;
        ctx.stroke();
        ctx.closePath();
        ctx.beginPath();
        ctx.arc((s*width/100),((100-v)*height/100),6,0,2*Math.PI);
        ctx.strokeStyle = 'white';
        ctx.lineWidth = 1;
        ctx.stroke();
        ctx.closePath();
        
    }

    function codeUpdate(){// updates different color codes from the values of h,s,v
        h = (isNaN(h)) ? 0 : h;
        s = (isNaN(s)) ? 0 : s;
        v = (isNaN(v)) ? 0 : v;
        hex= `#${convert.hsv.hex(h,s,v)}`;
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
        updateOutput();
    }

    function handleHueChange() {// updates the canvas when value of h changes
        pickerUpdate();
        codeUpdate();
        circleUpdate();
        updateOutput();
    }

// updating value of h,s,v on changing different color codes
    function handleHexChange(e){
        let inputCode = e.target.value;
        inputCode=inputCode.replace('#','');
        h=convert.hex.hsv(inputCode)[0];
        s=convert.hex.hsv(inputCode)[1];
        v=convert.hex.hsv(inputCode)[2];
        codeUpdate();
        pickerUpdate();
        circleUpdate();
        updateOutput();
    }

    function handleRgbChange(e){
        let inputCode = e.target.value;
        inputCode=inputCode.replace('rgb','').replace(/,/g,'').split(' ');
        h=convert.rgb.hsv(Number(inputCode[0]),Number(inputCode[1]),Number(inputCode[2]))[0];
        s=convert.rgb.hsv(Number(inputCode[0]),Number(inputCode[1]),Number(inputCode[2]))[1];
        v=convert.rgb.hsv(Number(inputCode[0]),Number(inputCode[1]),Number(inputCode[2]))[2];
        codeUpdate();
        pickerUpdate();
        circleUpdate();
        updateOutput();
    }

    function handleCmykChange(e){
        let inputCode = e.target.value+', ';
        inputCode=inputCode.replace(/%/g,'').replace(/,/g,'').split(' ');
        h=convert.cmyk.hsv(Number(inputCode[0]),Number(inputCode[1]),Number(inputCode[2]),Number(inputCode[3]))[0];
        s=convert.cmyk.hsv(Number(inputCode[0]),Number(inputCode[1]),Number(inputCode[2]),Number(inputCode[3]))[1];
        v=convert.cmyk.hsv(Number(inputCode[0]),Number(inputCode[1]),Number(inputCode[2]),Number(inputCode[3]))[2];
        console.log(inputCode);
        console.log(inputCode[0], inputCode[1], inputCode[2], inputCode[3]);
        console.log(Number(inputCode[0]), Number(inputCode[1]), Number(inputCode[2]), Number(inputCode[3]));
        
        codeUpdate();
        pickerUpdate();
        circleUpdate();
        updateOutput();
    }

    function handleHslChange(e){
        let inputCode = e.target.value+', ';
        inputCode=inputCode.replace(/°/g,'').replace(/%/g,'').replace(/,/g,'').split(' ');
        h=convert.hsl.hsv(Number(inputCode[0]),Number(inputCode[1]),Number(inputCode[2]))[0];
        s=convert.hsl.hsv(Number(inputCode[0]),Number(inputCode[1]),Number(inputCode[2]))[1];
        v=convert.hsl.hsv(Number(inputCode[0]),Number(inputCode[1]),Number(inputCode[2]))[2];
        codeUpdate();
        pickerUpdate();
        circleUpdate();
        updateOutput();
    }

    function handleHsvChange(e){
        let inputCode = e.target.value+', ';
        inputCode=inputCode.replace(/°/g,'').replace(/%/g,'').replace(/,/g,'').split(' ');
        h=Number(inputCode[0]);
        s=Number(inputCode[1]);
        v=Number(inputCode[2]);
        codeUpdate();
        pickerUpdate();
        circleUpdate();
        updateOutput();
    }


    function circleGrabbed(x,y) {// returns true if the mouse is inside the circle
        x-= (s*width/100);
        y-= ((100-v)*height/100);
        return Math.sqrt(Math.pow(x,2) + Math.pow(y,2)) < 5;
    }

    function handleMouseDown(e) {// check position of mouse when the user clicks on the canvas
        let x = e.offsetX;
        let y = e.offsetY;
        grabbed = circleGrabbed(x,y);
    }

    function handleMouseMove(e) {// move circle with mouse if user has grabbed it
        if(grabbed){
            let x = e.offsetX;
            let y = e.offsetY;
            s = Math.round((x/width)*100);
            v = 100 - Math.round((y/height)*100);
            s = 100<s ? 100 : s; // limit the values of s and v
            s = s<0 ? 0 : s;
            v = 100<v ? 100 : v;
            v = v<0 ? 0 : v;
            console.log(x,y, s,v);
            
            codeUpdate();
            pickerUpdate();
            circleUpdate();
            updateOutput();
        }
    }

    function handleMouseUp() {// release the circle
        grabbed = false;
    }


    onMount(() => {// initializing the canvas and getting dimensions of the canvas
        width = canvas.offsetWidth;
        height = canvas.offsetHeight;
        ctx = canvas.getContext('2d');
        codeUpdate();
        pickerUpdate();
        circleUpdate();
        updateOutput();
    })

    afterUpdate(() => {// smoothly update canvas with change in hue
        pickerUpdate();
        circleUpdate();
        codeUpdate();
        updateOutput();
    })

</script>

<div>
    <div class="w-fit">
        <h1 id="silder"class="text-3xl font-bold text-white m-6">ColorQuest</h1>
    </div>
    <div class="grid place-items-center">
        <div class="flex md:flex-row flex-col gap-4 p-4 items-center">
            <div class="">
                <canvas on:mousedown={handleMouseDown} on:mousemove={handleMouseMove} on:mouseup={handleMouseUp} bind:this={canvas} on:click={handleClick}></canvas>
            </div>
            <div class="border-2">
                <canvas bind:this={output} width="100" height="100"></canvas>
            </div>
        </div>
        <label for="silder" class="p-4">
            <input on:change={handleHueChange} type="range" min="0" max="360" bind:value={h} class="w-[300px] ">
        </label>
        <div>
            <label for="hex" class="text-white items-center flex flex-col md:flex-row">
                HEX: 
                <input on:change={handleHexChange} type="text" class="bg-slate-900 rounded-full m-2 p-[8px]" value={hex} >
            </label>
        </div>
        <label for="rgb" class="text-white">
            RGB: 
            <input on:change={handleRgbChange} type="text" class="bg-slate-900 rounded-full m-2 p-[8px]" value={rgb}>
        </label>
        <label for="cmyk" class="text-white">
            CMYK: 
            <input on:change={handleCmykChange} type="text" class="bg-slate-900 rounded-full m-2 p-[8px]" value={cmyk}>
        </label>
        <label for="hsl" class="text-white">
            HSL: 
            <input on:change={handleHslChange} type="text" class="bg-slate-900 rounded-full m-2 p-[8px]" value={hsl}>
        </label>
        <label for="hsv" class="text-white">
            HSV: 
            <input on:change={handleHsvChange} type="text" class="bg-slate-900 rounded-full m-2 p-[8px]" value={hsv}>
        </label>

    </div>
</div>

<!-- styling the slider -->
<style>
    input[type="range"] {
    -webkit-appearance: none; /* Override default styling */
    width: 300px;
    height: 12px;
    border-radius: 10px;
    background: linear-gradient(to right, 
    #ff0000 0%,
    #ff9900 10%,
    #ccff00 20%,
    #33ff00 30%,
    #00ff66 40%,
    #00ffff 50%,
    #0066ff 60%,
    #3300ff 70%,
    #cc00ff 80%,
    #ff0099 90%,
    #ff0000 100%
    );
    outline: none;
    opacity: 0.9;
    transition: opacity .15s ease-in-out;
  }
</style>