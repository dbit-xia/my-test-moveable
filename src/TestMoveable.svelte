<script>
    import Moveable from "svelte-moveable";
    import { onMount } from "svelte";
    import { Frame } from "scenejs";
    import keycon from "keycon";
    import Selecto from "svelte-selecto";

    
    let nextTick=()=> {
        return new Promise(res => setTimeout(res));
    };
    
    const KeyController = keycon.setGlobal();
    const frameMap = new Map();
    let targets = [];
    let moveable;
    let container;
    let elementGuidelines;
    // const moveable = new Moveable(document.body);
    // moveable.target = document.querySelector(".target");
    //
    // document.body.innerHTML = `
    //     <moveable>
    //         <p>This is some slotted content</p>
    //     </moveable>
    // `;

    // const frame = {
    //     translate: [0, 0],
    // };
    let target1;

    function log(...args){
        args.unshift(parseInt(Date.now() / 1000));
        console.log.apply(console,args);
    }
    
    function newFrame(el) {
        log('newFrame')
        const frame = new Frame({
            translate: [0, 0],
            transform: {
                translateX: "0px",
                translateY: "0px",
                rotate: "0deg",
                scaleX: 1,
                scaleY: 1
            }
        });

        frameMap.set(el, frame);

        return frame;
    }
    function getFrame(target) {
        return frameMap.get(target) || newFrame(target);
    }
    function onRender({ target }) {
        target.style.cssText += getFrame(target).toCSS();
    }
    function onDragStart({ target, set }) {
        const frame = getFrame(target);

        set([
            parseFloat(frame.get("transform", "translateX")),
            parseFloat(frame.get("transform", "translateY"))
        ]);
    }
    function onDrag({ target, beforeTranslate }) {
        const frame = getFrame(target);

        frame.set("transform", "translateX", `${beforeTranslate[0]}px`);
        frame.set("transform", "translateY", `${beforeTranslate[1]}px`);
    }

    async function onMouseDown(e) {
        log('onMouseDown');
        if (selecto.continueSelect) return;
        if (selectoDraging) return;
        // const target = e.target;
        // if (container === target) {
        //     targets = [];
        //     return;
        // }
        // log(moveable.isMoveableElement(target) );
        // if (moveable.isMoveableElement(target) || targets.indexOf(target) > -1) {
        //     return;
        // }
        // if (KeyController.shiftKey) {
        //     targets = [...targets, target];
        // } else {
        //     targets = [target];
        // }
        //
        await nextTick();
        moveable.dragStart(e);
    }
    async function onMouseUp(e) {
        log('onMouseUp');
        // moveable.dragEnd(e);
        await nextTick();
        selectoDraging = false;
        selecto.continueSelect = false;
        groupDraging = false;
        resizing = false;
        groupResizing = false;
    }
    
    function onClickGroup(e) {
        // const target = e.inputTarget;
        //
        // if (!target.classList.contains("target")) {
        //     return;
        // }
        // const index = targets.indexOf(target);
        // if (KeyController.shiftKey) {
        //     if (index === -1) {
        //         targets = [...targets, target];
        //     } else {
        //         targets.splice(index, 1);
        //         targets = [...targets];
        //     }
        // } else {
        //     targets = [target];
        // }
    }

    function onResizeStart({target, set, setOrigin, dragStart }) {
        const frame = getFrame(target);
        // Set origin if transform-orgin use %.
        setOrigin(["%", "%"]);
    
        // If cssSize and offsetSize are different, set cssSize. (no box-sizing)
        const style = window.getComputedStyle(target);
        const cssWidth = parseFloat(style.width);
        const cssHeight = parseFloat(style.height);
        set([cssWidth, cssHeight]);
    
        // If a drag event has already occurred, there is no dragStart.
        dragStart && dragStart.set(frame.get('translate'));
    }

    function onResize ({ target, width, height, drag })  {
        //if (groupResizing) return;
    
        const frame = getFrame(target);
        target.style.left =`${drag.left}px`;
        target.style.top = `${drag.top}px`;
        target.style.width = `${width}px`;
        target.style.height = `${height}px`;
    
        // get drag event
        frame.set('translate',drag.beforeTranslate);
        target.style.transform = `translate(${drag.beforeTranslate[0]}px, ${drag.beforeTranslate[1]}px)`;
    }

    // let throttleRotate = 0;
    // function onShift() {
    //     throttleRotate = KeyController.shiftKey ? 30 : 0;
    // }

    let groupDraging=false;
    let resizing=false;
    let groupResizing=false;
    let selecto;
    let selectoDraging=false;
    onMount(() => {

        // setGuides();

        //全选中
        elementGuidelines = [...document.querySelectorAll(".container .target")];

        // targets.forEach((target) => {
        //     const frame = new Frame({
        //         translate: [0, 0],
        //         transform: {
        //             translateX: "0px",
        //             translateY: "0px",
        //             rotate: "0deg",
        //             scaleX: 1,
        //             scaleY: 1
        //         }
        //     });
        //     frameMap.set(target, frame);
        // });


        // import { getElementInfo } from "moveable";
        // selecto = new Selecto({
        //     // The container to add a selection element
        //     container: container,
        //     // The area to drag selection element (default: container)
        //     // dragContainer: window,
        //     // Targets to select. You can register a queryselector or an Element.
        //     selectableTargets: [".target"],
        //     // Whether to select by click (default: true)
        //     selectByClick: true,
        //     // Whether to select from the target inside (default: true)
        //     selectFromInside: false,
        //     // After the select, whether to select the next target with the selected target (deselected if the target is selected again).
        //     continueSelect: false,
        //     // Determines which key to continue selecting the next target via keydown and keyup.
        //     // toggleContinueSelect: "shift",
        //     // The container for keydown and keyup events
        //     keyContainer: container,
        //     // The rate at which the target overlaps the drag area to be selected. (default: 100)
        //     hitRate: 50,
        //     // getElementRect: getElementInfo,
        // });
        // KeyController.keydown("shift", ()=>{
        //     selecto.continueSelect = true;
        // });
        // KeyController.keyup("shift", ()=>{
        //     selecto.continueSelect = false;
        // });
        //
        // selecto.on("dragStart", e => {
        //     log('selecto dragStart','groupDraging='+groupDraging)
        //     if (groupDraging || resizing) e.stop();
        //     selectoDraging = true;
        //    
        // }).on("select", e => {
        //     
        //
        // });

    });

    function selectoSelect(e){
        log('selecto select', 'groupDraging=' + groupDraging, 'targets.length=' + e.removed.length)
        if (groupDraging) return;

        e.removed.forEach(target => {
            // target.classList.remove("current");
            log('select removed',target === targets[0]);

            target.classList.remove("current");

            //选中多个后,单击有问题,所以注释掉
            const index = targets.indexOf(target);
            if (index > -1) {
                targets.splice(index, 1);
                targets = [...targets];
            }
        });

        e.added.forEach(target => {
            // target.classList.add("current");

            const index = targets.indexOf(target);
            if (index === -1) {
                targets = [...targets, target];
            }
            let firstTarget = targets[0];
            (firstTarget === target) && firstTarget.classList.add("current");

        });
    }

</script>

<style>

    .container {
        border: 1px solid #333;
        width: 800px;
        height: 300px;
        background: ButtonFace;
    }

    .target {
        position: absolute;
        width: 50px;
        height: 50px;
        border: 1px solid #333;
        text-align: center;
        box-sizing: border-box;
        background: white;
        min-width: 2px;
        min-height: 2px;
    }

    .target.current {
        position: absolute;
        border: 2px solid #f55;
    }

</style>

<div class="target current" style="display: none"></div>

<button>容器外按钮</button>
<div class="container" bind:this={container} on:mousedown={onMouseDown} on:mouseup={onMouseUp}>
    <div class="target" >Target</div>
    <div class="target" style="left: 100px" >Target</div>
    <div class="target" style="left: 200px" >Target</div>
    <a type="text" class="target" style="position: absolute" >456</a>
    
    <Moveable
            className="moveable"
            container={container}
            bind:this={moveable}
            target={targets}
            draggable={true}
            throttleDrag={1}
            snappable={true}
            snapThreshold={1}
            snapCenter={false}

            elementGuidelines={elementGuidelines}

            on:render={({ detail }) => {
                onRender(detail);
            }}
            on:renderGroup={({ detail }) => {
                detail.targets.forEach(target => onRender({ target }));
            }}
            on:clickGroup={({ detail }) => {
                log('clickGroup',"groupDraging="+groupDraging);
                if (groupDraging) return;
                onClickGroup(detail);
            }}
            on:dragStart={({ detail }) => {
                log('dragStart','isMoveableElement='+moveable.isMoveableElement(detail.target));
                onDragStart(detail);
            }}
            on:drag={({ detail }) => {
                onDrag(detail);
            }}
            on:dragGroupStart={({ detail: { targets,events }}) => {
                log('dragGroupStart');
                groupDraging=true;
                events.forEach(({target,set}, i) => {
                    onDragStart({target,set})
                });
            }}
            on:dragGroup={({ detail: { targets, events }}) => {
                events.forEach(({ target, beforeTranslate }, i) => {
                    onDrag({target,beforeTranslate});
                });
            }}
            on:dragGroupEnd={async ({ detail: { targets, isDrag, clientX, clientY }}) => {
                await nextTick();
                groupDraging=false;
                log("onDragGroupEnd", targets, isDrag);
            }}

            resizable={true}
            keepRatio={false}
            throttleResize={0}
            on:resizeStart={({detail})=>{
                log('resizeStart')
                //if (groupResizing) return;
                resizing=true;
                onResizeStart(detail);
            }}
            on:resize={({detail})=>{
                onResize(detail);
            }}
            on:resizeEnd={async ({ detail: { target, isDrag, clientX, clientY }}) => {
                await nextTick();
                resizing=false;
                log("onResizeEnd", target, isDrag);
            }}

            on:resizeGroupStart={({detail:{events}}) => {
                log('resizeGroupStart')
                groupResizing=true;
                //events.forEach((event)=>{
                //    onResizeStart(event);
                //});
                events.forEach((ev, i) => {
                    let {target}=ev;
                    const frame = getFrame(target);
        
                    // Set origin if transform-orgin use %.
                    ev.setOrigin(["%", "%"]);
        
                    // If cssSize and offsetSize are different, set cssSize.
                    const style = window.getComputedStyle(ev.target);
                    const cssWidth = parseFloat(style.width);
                    const cssHeight = parseFloat(style.height);
                    ev.set([cssWidth, cssHeight]);
        
                    // If a drag event has already occurred, there is no dragStart.
                    ev.dragStart && ev.dragStart.set(frame.get('translate'));
                });
            }}
            on:resizeGroup={(eventInfo) => {
                //每一个target的x和y首次拖拉时,会不精确!!!
                let {detail:{ targets, events,direction,delta,dist }}=eventInfo;
                //log(delta,eventInfo.detail);
                events.forEach(event => {
                    let {target, width, height, drag } = event;
                    const frame = getFrame(target);
                    //等距缩放
                    const style = window.getComputedStyle(target);
                    if (direction[0]!==0) target.style.width = `${parseFloat(style.width)+delta[0]}px`;
                    if (direction[1]!==0) target.style.height = `${parseFloat(style.height)+delta[1]}px`;
                    //等比缩放
                    //target.style.width = `${width}px`;
                    //target.style.height = `${height}px`;
                    // get drag event
                    frame.set('translate',drag.beforeTranslate);
                    target.style.transform = `translate(${drag.beforeTranslate[0]}px, ${drag.beforeTranslate[1]}px)`;
                });
            }}
            on:resizeGroupEnd={async (eventInfo) => {
                await nextTick();
                groupResizing=false;
            }}


        />
        <Selecto bind:this={selecto}
                 container={container}
                 keyContainer={container}
                 selectableTargets={[".target"]}
                 selectByClick={true}
                 selectFromInside={false}
                 continueSelect={false}
                 toggleContinueSelect={"shift"}
                 hitRate={50}
                 on:dragStart={({ detail: e }) => {
                    log('selecto dragStart', 'groupDraging=' + groupDraging)
                    if (groupDraging || resizing) return e.stop();
                    selectoDraging = true;
                  }} 
                 on:select={({ detail: e }) => {
                  selectoSelect(e);
                 }}
        />
    </div>