<script>
    import Moveable from "svelte-moveable";
    import { onMount, tick } from "svelte";
    import { Frame } from "scenejs";
    import keycon from "keycon";

    const KeyController = keycon.setGlobal();
    const frameMap = new Map();
    let targets = [];
    let moveable;
    let container;
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
            // translate: [0, 0],
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
        const target = e.target;
        if (container === target) {
            targets = [];
            return;
        }
        console.log(moveable.isMoveableElement(target) );
        if (moveable.isMoveableElement(target) || targets.indexOf(target) > -1) {
            return;
        }
        if (KeyController.shiftKey) {
            targets = [...targets, target];
        } else {
            targets = [target];
        }

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

    onMount(() => {
        requestAnimationFrame(() => {
            targets = [document.querySelector(".target")];

            // onWindowResize();
        });
    });
</script>

<style>

    .container {
        border: 1px solid #333;
        width: 800px;
        height: 300px;
    }

    div.target {
        position: absolute;
    }

    .target {
        position: absolute;
        width: 50px;
        height: 50px;
        border: 1px solid #333;
        text-align: center;
        box-sizing: border-box;
        z-index: 20;
    }
</style>

<div class="container" bind:this={container} on:mousedown={onMouseDown}>
    <div class="target" aria-disabled="true" >Target</div>
    <div class="target" aria-disabled="true" style="left: 100px" >Target</div>
</div>
<Moveable
        className="moveable"
        container={container}
        bind:this={moveable}
        target={targets}
        draggable={true}
        throttleDrag={0}
        on:render={({ detail }) => {
            onRender(detail);
          }}
        on:renderGroup={({ detail }) => {
        detail.targets.forEach(target => onRender({ target }));
      }}
        on:clickGroup={({ detail }) => {
        onClickGroup(detail);
      }}
        on:dragStart={({ detail }) => {
            onDragStart(detail);
          }}
        on:drag={({ detail }) => {
            onDrag(detail);
          }}
        on:dragGroupStart={({ detail }) => {
            detail.events.forEach(onDragStart);
          }}
        on:dragGroup={({ detail }) => {
            detail.events.forEach(onDrag);
          }}

        resizable={true}
        throttleResize={0}
        on:resizeStart={({ detail: {target, set, setOrigin, dragStart }}) => {
                const frame = getFrame(target);
            // Set origin if transform-orgin use %.
            setOrigin(["%", "%"]);

            // If cssSize and offsetSize are different, set cssSize. (no box-sizing)
            const style = window.getComputedStyle(target);
            const cssWidth = parseFloat(style.width);
            const cssHeight = parseFloat(style.height);
            set([cssWidth, cssHeight]);

            // If a drag event has already occurred, there is no dragStart.
            //dragStart && dragStart.set(frame.get('translate'));
        }}
        on:resize={({ detail: { target, width, height, drag }}) => {
                const frame = getFrame(target);
            target.style.width = `${width}px`;
            target.style.height = `${height}px`;

            // get drag event
            frame.set('translate',drag.beforeTranslate);
            target.style.transform
                = `translate(${drag.beforeTranslate[0]}px, ${drag.beforeTranslate[1]}px)`;
        }}
        on:resizeEnd={({ detail: { target, isDrag, clientX, clientY }}) => {
            console.log("onResizeEnd", target, isDrag);
        }}
/>
