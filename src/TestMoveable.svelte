<script>
    import Moveable from "svelte-moveable";
    import { onMount, tick } from "svelte";
    import { Frame } from "scenejs";
    import keycon from "keycon";
    import Selecto from "selecto";

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

    function newFrame(el) {
        console.log('newFrame')
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

    function onMouseDown(e) {
        // const target = e.target;
        // if (container === target) {
        //     targets = [];
        //     return;
        // }
        // console.log(moveable.isMoveableElement(target) );
        // if (moveable.isMoveableElement(target) || targets.indexOf(target) > -1) {
        //     return;
        // }
        // if (KeyController.shiftKey) {
        //     targets = [...targets, target];
        // } else {
        //     targets = [target];
        // }

        setTimeout(() => {
            moveable.dragStart(e);
        });
    }
    function onClickGroup(e) {
        const target = e.inputTarget;

        if (!target.classList.contains("target")) {
            return;
        }
        const index = targets.indexOf(target);
        if (KeyController.shiftKey) {
            if (index === -1) {
                targets = [...targets, target];
            } else {
                targets.splice(index, 1);
                targets = [...targets];
            }
        } else {
            targets = [target];
        }
    }

    let throttleRotate = 0;
    function onShift() {
        throttleRotate = KeyController.shiftKey ? 30 : 0;
    }

    let groupDraging=false;
    let resizing=false;
    onMount(() => {

        // setGuides();

        //全选中
        elementGuidelines = [...document.querySelectorAll(".container .target")];

        // KeyController.keydown("shift", onShift);
        // KeyController.keyup("shift", onShift);

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
        const selecto = new Selecto({
            // The container to add a selection element
            container: container,
            // The area to drag selection element (default: container)
            // dragContainer: window,
            // Targets to select. You can register a queryselector or an Element.
            selectableTargets: [".target"],
            // Whether to select by click (default: true)
            selectByClick: true,
            // Whether to select from the target inside (default: true)
            selectFromInside: false,
            // After the select, whether to select the next target with the selected target (deselected if the target is selected again).
            continueSelect: false,
            // Determines which key to continue selecting the next target via keydown and keyup.
            toggleContinueSelect: "ctrl",
            // The container for keydown and keyup events
            keyContainer: container,
            // The rate at which the target overlaps the drag area to be selected. (default: 100)
            hitRate: 50,
            // getElementRect: getElementInfo,
        });

        selecto.on("dragStart", e => {
            console.log('selecto dragStart','groupDraging='+groupDraging)
            if (groupDraging || resizing) e.stop();

        }).on("select", e => {
            console.log('selecto select', 'groupDraging=' + groupDraging, 'targets.length=' + e.removed.length)
            if (groupDraging) return;

            e.removed.forEach(target => {
                // target.classList.remove("current");
                console.log('select removed',target === targets[0]);

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

        });

    });


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


    <div class="container" bind:this={container} on:mousedown={onMouseDown}>
        <div class="target" >Target</div>
        <div class="target" style="left: 100px" >Target</div>
        <div class="target" style="left: 200px" >Target</div>
        <a type="text" class="target" style="position: absolute" >456</a>
    </div>
    <button>容器外按钮</button>
    <Moveable
            className="moveable"
            container={container}
            bind:this={moveable}
            target={targets}
            draggable={true}
            throttleDrag={0}
            snappable={true}
            snapThreshold={2}
            snapCenter={false}

            elementGuidelines={elementGuidelines}

            on:render={({ detail }) => {
                onRender(detail);
            }}
            on:renderGroup={({ detail }) => {
                detail.targets.forEach(target => onRender({ target }));
            }}
            on:clickGroup={({ detail }) => {
                console.log('clickGroup',"groupDraging="+groupDraging);
                if (groupDraging) return;
                onClickGroup(detail);
            }}
            on:dragStart={({ detail }) => {
                console.log('dragStart','isMoveableElement='+moveable.isMoveableElement(detail.target));
                onDragStart(detail);
            }}
            on:drag={({ detail }) => {
                onDrag(detail);
            }}
            on:dragGroupStart={({ detail: { targets,events }}) => {
                console.log('dragGroupStart');
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
            on:dragGroupEnd={({ detail: { targets, isDrag, clientX, clientY }}) => {
                setTimeout(()=>{
                    groupDraging=false;
                });
                console.log("onDragGroupEnd", targets, isDrag);
            }}

            resizable={true}
            throttleResize={0}
            on:resizeStart={({ detail: {target, set, setOrigin, dragStart }}) => {
                resizing=true;
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
            }}
            on:resize={({ detail: { target, width, height, drag }}) => {
                const frame = getFrame(target);
                target.style.left =`${drag.left}px`;
                target.style.top = `${drag.top}px`;
                target.style.width = `${width}px`;
                target.style.height = `${height}px`;

                // get drag event
                frame.set('translate',drag.beforeTranslate);
                target.style.transform = `translate(${drag.beforeTranslate[0]}px, ${drag.beforeTranslate[1]}px)`;
            }}
            on:resizeEnd={({ detail: { target, isDrag, clientX, clientY }}) => {
                setTimeout(()=>{
                    resizing=false;
                });
                console.log("onResizeEnd", target, isDrag);
            }}


    />
