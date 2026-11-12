<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>晶圓廠第一人稱探索 EUV 微影模擬</title>
    <style>
        :root {
            --bg-color: #05070a;
            --panel-bg: #161b22;
            --accent-cyan: #00f0ff;
            --accent-purple: #9d4edd;
            --accent-green: #00ff87;
            --accent-red: #ff0055;
            --text-color: #c9d1d9;
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            user-select: none;
        }

        body {
            background-color: var(--bg-color);
            color: var(--text-color);
            font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
            padding: 15px;
            overflow-x: hidden;
        }

        header {
            text-align: center;
            margin-bottom: 12px;
        }

        h1 {
            color: var(--accent-cyan);
            font-size: 22px;
            letter-spacing: 1.5px;
            margin-bottom: 4px;
            text-shadow: 0 0 10px rgba(0, 240, 255, 0.4);
        }

        p.subtitle {
            font-size: 13px;
            color: #8b949e;
        }

        #splash-screen {
            position: fixed;
            top: 0; left: 0;
            width: 100vw; height: 100vh;
            background: #000;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 999;
            perspective: 1000px;
            overflow: hidden;
        }

        .scanlines {
            position: absolute;
            top: 0; left: 0;
            width: 100%; height: 100%;
            background: linear-gradient(rgba(18, 16, 16, 0) 50%, rgba(0, 0, 0, 0.25) 50%), 
                        linear-gradient(90deg, rgba(255, 0, 0, 0.03), rgba(0, 255, 0, 0.01), rgba(0, 0, 255, 0.03));
            background-size: 100% 4px, 6px 100%;
            pointer-events: none;
            z-index: 10;
        }

        .machine-viewport {
            position: relative;
            width: 800px;
            height: 420px;
            border-radius: 12px;
            overflow: hidden;
            border: 2px solid rgba(0, 240, 255, 0.3);
            box-shadow: 0 0 50px rgba(0, 0, 0, 0.9), 0 0 30px rgba(0, 240, 255, 0.2);
            transform-style: preserve-3d;
            transition: transform 0.1s ease-out;
        }

        .intro-overlay {
            position: absolute;
            bottom: 25px;
            left: 50%;
            transform: translateX(-50%);
            z-index: 20;
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 12px;
            width: 90%;
            text-align: center;
        }

        .intro-title {
            color: #fff;
            font-size: 20px;
            letter-spacing: 2px;
            text-shadow: 0 0 10px #00f0ff, 0 0 20px #00f0ff;
        }

        .btn-enter-machine {
            background: linear-gradient(135deg, #ff0055, #9d4edd);
            color: #fff;
            border: 1px solid #ff0055;
            padding: 16px 45px;
            font-size: 17px;
            font-weight: bold;
            border-radius: 40px;
            cursor: pointer;
            letter-spacing: 2px;
            transition: all 0.3s ease;
            box-shadow: 0 0 25px rgba(255, 0, 85, 0.6);
            animation: pulse-red 2s infinite;
        }

        .btn-enter-machine:hover {
            transform: scale(1.08);
            box-shadow: 0 0 40px rgba(255, 0, 85, 0.9);
            background: linear-gradient(135deg, #ff0055, #00f0ff);
        }

        @keyframes pulse-red {
            0% { box-shadow: 0 0 15px rgba(255, 0, 85, 0.4); }
            50% { box-shadow: 0 0 35px rgba(255, 0, 85, 0.8); }
            100% { box-shadow: 0 0 15px rgba(255, 0, 85, 0.4); }
        }

        .main-container {
            display: flex;
            flex-direction: column;
            gap: 12px;
            max-width: 1440px;
            width: 100%;
            align-items: center;
            opacity: 0;
            transition: opacity 1s ease;
        }

        #canvas-container {
            position: relative;
            box-shadow: 0 0 35px rgba(0, 0, 0, 0.9), 0 0 20px rgba(0, 240, 255, 0.25);
            border-radius: 10px;
            border: 1px solid rgba(0, 240, 255, 0.4);
            overflow: hidden;
            background: #05070a;
        }

        canvas { display: block; }

        .guide-box {
            background: rgba(22, 27, 34, 0.9);
            border-left: 4px solid var(--accent-green);
            padding: 10px 15px;
            border-radius: 4px;
            width: 100%;
            font-size: 13px;
            line-height: 1.5;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .guide-box strong { color: #fff; }

        .controls-panel {
            background: var(--panel-bg);
            border-radius: 10px;
            padding: 15px 20px;
            width: 100%;
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
            gap: 12px;
            border: 1px solid rgba(255, 255, 255, 0.08);
        }

        .control-group {
            display: flex;
            flex-direction: column;
            gap: 5px;
            background: rgba(0,0,0,0.4);
            padding: 8px 12px;
            border-radius: 6px;
            border: 1px solid rgba(255,255,255,0.05);
        }

        .control-group label {
            font-size: 12px;
            font-weight: 600;
            color: #fff;
            display: flex;
            justify-content: space-between;
        }

        .control-group label span {
            color: var(--accent-cyan);
            font-family: monospace;
            font-size: 13px;
        }

        .control-group input[type=number] {
            width: 72px;
            padding: 4px 6px;
            border: 1px solid rgba(0, 240, 255, 0.35);
            border-radius: 4px;
            background: #0d1117;
            color: var(--text-color);
            font: 13px monospace;
        }

        .angle-controls {
            display: flex;
            align-items: center;
            gap: 10px;
        }

        .angle-controls input[type=range] {
            flex: 1;
            min-width: 0;
        }

        input[type=range] {
            -webkit-appearance: none;
            width: 100%;
            background: #0d1117;
            height: 6px;
            border-radius: 3px;
            outline: none;
        }

        input[type=range]::-webkit-slider-thumb {
            -webkit-appearance: none;
            appearance: none;
            width: 14px; height: 14px;
            border-radius: 50%;
            background: var(--accent-cyan);
            cursor: pointer;
            box-shadow: 0 0 8px var(--accent-cyan);
        }

        .btn-container {
            grid-column: 1 / -1;
            display: flex;
            justify-content: center;
            align-items: center;
            gap: 15px;
        }

        .btn-fire {
            background: linear-gradient(135deg, #2ea043, #238636);
            color: #fff;
            border: none;
            padding: 11px 28px;
            font-size: 14px;
            font-weight: bold;
            border-radius: 6px;
            cursor: pointer;
            transition: all 0.2s;
            box-shadow: 0 0 12px rgba(46, 160, 67, 0.4);
        }

        .btn-fire:hover {
            background: linear-gradient(135deg, #3fb950, #2ea043);
            transform: translateY(-1px);
        }

        .key-hint {
            font-size: 12px;
            color: var(--accent-cyan);
            background: rgba(0, 240, 255, 0.1);
            padding: 4px 8px;
            border-radius: 4px;
            border: 1px solid rgba(0, 240, 255, 0.3);
        }

        .rpg-shell {
            position: relative;
            width: 100%;
            background: linear-gradient(145deg, #111923, #080d14);
            border: 1px solid rgba(0, 240, 255, 0.25);
            border-radius: 12px;
            padding: 18px;
            box-shadow: 0 0 24px rgba(0, 0, 0, 0.5);
        }
        #rpgSection { max-width: 1440px; }

        .rpg-heading {
            display: flex;
            justify-content: space-between;
            gap: 14px;
            align-items: center;
            margin-bottom: 14px;
        }

        .rpg-heading h2, .rpg-shell h3 { color: #fff; }
        .rpg-heading p, .rpg-intro { color: #9ba8b7; font-size: 13px; line-height: 1.6; }
        .role-grid, .station-list, .choice-list {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 12px;
            margin-top: 14px;
        }

        .role-card, .station-button, .choice-button, .rpg-action {
            background: #111c29;
            border: 1px solid rgba(0, 240, 255, 0.25);
            color: var(--text-color);
            border-radius: 8px;
            padding: 14px;
            text-align: left;
            cursor: pointer;
            transition: border-color 0.2s, background 0.2s, transform 0.2s;
        }

        .role-card:hover, .choice-button:hover, .rpg-action:hover {
            border-color: var(--accent-cyan);
            background: #17283a;
            transform: translateY(-1px);
        }

        .role-card strong, .station-button strong { color: var(--accent-cyan); display: block; margin-bottom: 6px; }
        .role-card span, .station-button span { color: #9ba8b7; font-size: 12px; line-height: 1.5; }
        .rpg-topline { display: flex; justify-content: space-between; align-items: center; gap: 10px; margin-bottom: 10px; }
        .rpg-topline strong { color: var(--accent-green); }
        .factory-layout {
            display: grid;
            grid-template-columns: minmax(0, 1.35fr) minmax(320px, 0.85fr);
            gap: 16px;
            align-items: start;
        }
        .factory-visual-column, .factory-info-column { min-width: 0; }
        .factory-info-column {
            position: sticky;
            top: 12px;
            max-height: calc(100vh - 30px);
            overflow-y: auto;
            padding-right: 4px;
        }
        .factory-info-column .station-list { grid-template-columns: 1fr; margin-top: 0; }
        .factory-info-column .knowledge-card { margin-top: 12px; }
        .factory-info-column .knowledge-card:not(.hidden) { max-height: 62vh; overflow-y: auto; }
        #factoryCanvas {
            width: 100%;
            max-height: 460px;
            border: 1px solid rgba(0, 240, 255, 0.35);
            border-radius: 8px;
            background: #07101a;
        }
        .rpg-controls { display: flex; flex-wrap: wrap; justify-content: center; gap: 8px; margin-top: 10px; }
        .rpg-action { text-align: center; padding: 9px 15px; }
        .station-button { display: flex; align-items: center; justify-content: space-between; gap: 10px; }
        .station-button { cursor: default; }
        .station-button.done { border-color: var(--accent-green); }
        .knowledge-card {
            margin-top: 14px;
            padding: 16px;
            border-left: 4px solid var(--accent-cyan);
            background: rgba(0, 0, 0, 0.28);
            border-radius: 6px;
            line-height: 1.65;
        }
        .dialogue-scene {
            display: grid;
            grid-template-columns: 70px 1fr;
            gap: 12px;
            align-items: start;
            margin-top: 12px;
        }
        .dialogue-avatar {
            width: 64px;
            height: 64px;
            display: grid;
            place-items: center;
            border-radius: 50%;
            color: #061018;
            font-size: 25px;
            font-weight: 800;
            background: radial-gradient(circle at 35% 30%, #b5fbff, #00b6ce 70%);
            box-shadow: 0 0 22px rgba(0, 240, 255, 0.3);
        }
        .dialogue-copy {
            min-height: 76px;
            padding: 11px 14px;
            border: 1px solid rgba(0, 240, 255, 0.22);
            border-radius: 8px;
            background: rgba(5, 12, 20, 0.78);
        }
        .dialogue-copy strong { color: var(--accent-cyan); }
        .dialogue-copy p { margin-top: 5px; color: #e0e8ef; }
        .story-briefing {
            margin-top: 12px;
            padding: 12px 14px;
            background: rgba(0, 240, 255, 0.07);
            border-radius: 7px;
            color: #cbd7e2;
        }
        .scene-enter { animation: scene-enter 0.55s ease-out; }
        @keyframes scene-enter {
            from { opacity: 0.35; transform: scale(0.985); filter: brightness(1.7); }
            to { opacity: 1; transform: scale(1); filter: brightness(1); }
        }
        .knowledge-card p { margin-top: 8px; color: #b8c4d1; }
        .rpg-footer { margin-top: 12px; color: #8996a5; font-size: 12px; text-align: center; }
        .role-completion-message {
            margin-top: 12px;
            padding: 14px;
            border: 1px solid rgba(0, 255, 135, 0.48);
            border-radius: 8px;
            background: rgba(0, 255, 135, 0.09);
            color: #a7ffd1 !important;
            font-weight: 700;
        }
        .knowledge-intro {
            padding: 18px;
            border: 1px solid rgba(0, 240, 255, 0.25);
            border-radius: 10px;
            background: rgba(0, 0, 0, 0.25);
        }
        .knowledge-intro h2, .knowledge-intro h3 { color: #fff; }
        .knowledge-intro > p { margin-top: 8px; color: #b8c4d1; line-height: 1.7; }
        .term-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(190px, 1fr));
            gap: 10px;
            margin-top: 14px;
        }
        .term-card {
            padding: 12px;
            border: 1px solid rgba(0, 240, 255, 0.18);
            border-radius: 8px;
            background: #0b1520;
        }
        .term-card strong { color: var(--accent-cyan); }
        .term-card p { margin-top: 6px; color: #b8c4d1; font-size: 13px; line-height: 1.55; }
        .tutorial-overlay {
            position: fixed;
            inset: 0;
            z-index: 1000;
            display: grid;
            place-items: center;
            padding: 18px;
            background: rgba(0, 0, 0, 0.82);
        }
        .tutorial-dialog {
            width: min(680px, 100%);
            max-height: 90vh;
            overflow-y: auto;
            padding: 22px;
            border: 1px solid rgba(0, 240, 255, 0.5);
            border-radius: 12px;
            background: #101923;
            box-shadow: 0 0 36px rgba(0, 240, 255, 0.18);
        }
        .tutorial-dialog h2 { color: var(--accent-cyan); }
        .tutorial-dialog p, .tutorial-dialog li { color: #c5d0dc; line-height: 1.7; }
        .tutorial-dialog ul { padding-left: 22px; margin: 12px 0; }
        .hint-button { border-color: #e8ba5b; color: #ffe09a; }
        .hint-panel {
            position: absolute;
            z-index: 10;
            top: 62px;
            right: 18px;
            width: min(420px, calc(100% - 36px));
            padding: 16px;
            border: 1px solid rgba(255, 210, 100, 0.55);
            border-radius: 9px;
            background: rgba(12, 17, 24, 0.97);
            box-shadow: 0 8px 26px rgba(0, 0, 0, 0.55);
            line-height: 1.65;
        }
        .hint-panel h3 { color: #ffe09a; }
        .hint-panel p { margin-top: 8px; color: #d3dce6; }
        .block-lab {
            margin-top: 12px;
            padding: 18px;
            border: 1px solid rgba(0, 240, 255, 0.28);
            border-radius: 10px;
            background: #09121c;
        }
        .block-lab h2, .block-lab h3 { color: #fff; }
        .block-lab p { color: #bac8d6; line-height: 1.65; }
        .block-board {
            display: grid;
            grid-template-columns: minmax(0, 1.2fr) minmax(250px, 0.8fr);
            gap: 14px;
            margin-top: 14px;
        }
        .block-panel {
            padding: 14px;
            border: 1px solid rgba(255, 255, 255, 0.12);
            border-radius: 8px;
            background: rgba(0, 0, 0, 0.24);
        }
        .block-tray { display: flex; flex-wrap: wrap; gap: 8px; margin-top: 10px; }
        .material-block {
            padding: 9px 11px;
            border: 1px solid #54c9dd;
            border-radius: 7px;
            background: #112c3a;
            color: #e4faff;
            cursor: grab;
        }
        .material-block.selected { outline: 2px solid #ffe16b; }
        .material-block:active { cursor: grabbing; }
        .mask-slot, .wafer-drop-zone {
            min-height: 76px;
            margin-top: 10px;
            padding: 10px;
            border: 2px dashed rgba(0, 240, 255, 0.42);
            border-radius: 8px;
            background: rgba(0, 240, 255, 0.045);
        }
        .mask-slot.drop-ready, .wafer-drop-zone.drop-ready { border-color: #ffe16b; background: rgba(255, 225, 107, 0.08); }
        .wafer-drop-zone {
            min-height: 270px;
            display: grid;
            place-items: center;
            overflow: auto;
            touch-action: pan-y;
        }
        .wafer-model {
            position: relative;
            display: flex;
            flex-direction: column;
            justify-content: flex-end;
            align-items: center;
            gap: 5px;
            width: min(380px, 100%);
            min-height: 230px;
            padding: 12px 0;
            pointer-events: none;
        }
        .silicon-base, .stack-layer {
            position: relative;
            display: grid;
            place-items: center;
            font-size: 12px;
            font-weight: 800;
            text-align: center;
            box-sizing: border-box;
        }
        .silicon-base {
            width: 92%;
            min-height: 64px;
            border: 2px solid #a9c7df;
            border-radius: 6px;
            color: #e5f1ff;
            background: linear-gradient(180deg, #506a84, #26394d);
            box-shadow: 0 4px 0 #172839;
        }
        .stack-layer {
            width: 78%;
            min-height: 42px;
            padding: 5px;
            border: 1px solid rgba(255, 255, 255, 0.65);
            border-radius: 5px;
            color: #07121b;
            box-shadow: 0 3px 0 rgba(7, 37, 47, 0.85);
        }
        .stack-layer.film { background: linear-gradient(145deg, #a5fff0, #3cb8b1); }
        .stack-layer.resist { background: linear-gradient(145deg, #efb8ff, #ac69d5); }
        .stack-layer.gate-layer { background: linear-gradient(145deg, #ffe597, #d88432); }
        .film-pattern { display: flex; width: 100%; gap: 4px; }
        .film-pattern span { flex: 1; min-height: 32px; display: grid; place-items: center; border-radius: 3px; background: rgba(15, 88, 86, 0.75); color: #f0fffc; font-size: 10px; }
        .film-pattern span.gap { background: rgba(5, 14, 20, 0.88); border: 1px dashed #d4f4ff; }
        .resist-pattern { display: flex; width: 100%; gap: 4px; }
        .resist-pattern span { flex: 1; min-height: 22px; border-radius: 3px; background: #8e51ba; }
        .resist-pattern span.gap { background: rgba(5, 14, 20, 0.88); border: 1px dashed #fff; }
        .process-buttons { display: grid; grid-template-columns: repeat(2, minmax(0, 1fr)); gap: 8px; margin-top: 12px; }
        .process-buttons .rpg-action { min-height: 44px; }
        .process-buttons .rpg-action:disabled { opacity: 0.4; cursor: not-allowed; transform: none; }
        #blockStatus { min-height: 68px; margin-top: 12px; padding: 10px; border-left: 3px solid var(--accent-cyan); background: rgba(0, 240, 255, 0.06); }
        .exposure-lab { margin-top: 12px; padding: 18px; border: 1px solid rgba(255, 214, 110, 0.34); border-radius: 10px; background: #100f17; }
        .exposure-lab h2, .exposure-lab h3 { color: #ffe4a0; }
        .exposure-lab p { color: #d3cedb; line-height: 1.65; }
        .exposure-steps { display: flex; flex-wrap: wrap; gap: 8px; margin: 14px 0; }
        .exposure-step-indicator { padding: 7px 11px; border: 1px solid #554b3b; border-radius: 20px; color: #aaa4b2; background: #1a1720; }
        .exposure-step-indicator.active { border-color: #ffd76f; color: #fff0bf; background: rgba(255, 215, 111, 0.12); }
        .exposure-step-indicator.done { border-color: #55cba0; color: #a4ffdc; }
        .exposure-instructions { min-height: 48px; margin: 10px 0; padding: 10px 12px; border-left: 3px solid #ffd76f; background: rgba(255, 215, 111, 0.07); }
        .coating-canvas-wrap { display: grid; justify-items: center; padding: 12px; border: 1px solid rgba(255,255,255,0.12); border-radius: 8px; background: #0a0c12; }
        #coatingCanvas { display: block; width: min(560px, 100%); height: auto; border-radius: 8px; touch-action: none; cursor: crosshair; }
        .exposure-metrics { display: flex; flex-wrap: wrap; gap: 10px 22px; margin: 10px 0; color: #e4dff0; }
        .exposure-metrics strong { color: #ffe4a0; }
        .exposure-actions { display: flex; flex-wrap: wrap; gap: 8px; margin-top: 12px; }
        .alignment-arena { position: relative; width: min(560px, 100%); height: 300px; margin: 14px auto; overflow: hidden; border: 1px solid rgba(255,255,255,0.14); border-radius: 8px; background: radial-gradient(ellipse at center, #202638, #0a0c12 72%); touch-action: none; }
        .alignment-wafer { position: absolute; left: 50%; top: 53%; width: min(220px, 48vw); aspect-ratio: 1; transform: translate(-50%, -50%); border: 3px solid #9aacc8; border-radius: 50%; background: radial-gradient(circle, #384762, #1a2535 72%); box-shadow: inset 0 0 30px rgba(0,0,0,.5), 0 0 20px rgba(95,150,255,.13); }
        .wafer-fiducial, .mask-fiducial { position: absolute; left: 50%; top: 50%; width: 28px; height: 28px; transform: translate(-50%, -50%); }
        .wafer-fiducial::before, .wafer-fiducial::after, .mask-fiducial::before, .mask-fiducial::after { position: absolute; content: ""; background: #fff0a8; }
        .wafer-fiducial::before, .mask-fiducial::before { left: 13px; top: 0; width: 2px; height: 28px; }
        .wafer-fiducial::after, .mask-fiducial::after { left: 0; top: 13px; width: 28px; height: 2px; }
        .alignment-mask { position: absolute; z-index: 2; width: 142px; height: 76px; display: grid; place-items: center; border: 2px solid #f3c66e; border-radius: 5px; background: repeating-linear-gradient(90deg, rgba(255,218,122,.24) 0 8px, rgba(19,24,37,.9) 8px 16px); color: #fff1bd; font-size: 12px; font-weight: 700; cursor: grab; touch-action: none; user-select: none; }
        .alignment-mask:active { cursor: grabbing; }
        .align-status { text-align: center; color: #ffe4a0 !important; }
        .wavelength-options { display: grid; grid-template-columns: repeat(4, minmax(0, 1fr)); gap: 8px; margin: 12px 0; }
        .wavelength-option { min-height: 58px; padding: 8px; border: 1px solid #51495e; border-radius: 7px; color: #e2dcea; background: #17141e; cursor: pointer; }
        .wavelength-option[aria-pressed="true"] { border-color: #ffe07d; color: #fff0bd; background: rgba(255, 215, 111, 0.14); box-shadow: 0 0 12px rgba(255,215,111,.1); }
        .wavelength-option small { display: block; margin-top: 4px; color: #aaa3b4; }
        .exposure-control { margin: 16px 0; padding: 12px; border: 1px solid rgba(255,255,255,.1); border-radius: 8px; background: rgba(0,0,0,.18); }
        .exposure-control label { display: flex; justify-content: space-between; gap: 10px; color: #eee7f5; }
        .exposure-control input[type="range"] { width: 100%; margin-top: 12px; accent-color: #ffd76f; }
        .exposure-preview { padding: 10px; border-radius: 7px; background: #090b11; color: #dad3e5; }
        .circuit-cross-section { margin: 14px 0; padding: 16px; border: 1px solid rgba(255,215,111,.28); border-radius: 8px; background: #080b12; }
        .cross-section-label { margin-bottom: 8px; color: #ffe4a0; text-align: center; font-weight: 700; }
        .cross-section-layer { position: relative; min-height: 62px; display: grid; place-items: center; border: 1px solid rgba(255,255,255,.15); color: #e4e6ed; font-size: 12px; }
        .cross-section-layer.resist-layer { background: linear-gradient(180deg, #623d80, #362348); }
        .cross-section-layer.oxide-layer { min-height: 38px; background: #58a69c; color: #071712; }
        .cross-section-layer.silicon-layer { min-height: 48px; background: linear-gradient(180deg, #536987, #27384e); }
        .circuit-traces { position: relative; height: 62px; overflow: hidden; border: 1px solid rgba(255,255,255,.14); background: linear-gradient(180deg, #392641, #22182d); }
        .circuit-traces span { position: absolute; top: 10px; height: 40px; border: 1px solid #ffe2a0; border-radius: 3px 3px 0 0; background: linear-gradient(90deg, #ffdb77, #c98538); }
        .circuit-traces.shorted::after { position: absolute; top: 29px; left: 0; right: 0; height: 5px; content: ""; background: #ff5a69; box-shadow: 0 0 10px #ff5a69; }
        .circuit-traces.broken span { border-style: dashed; background: repeating-linear-gradient(180deg, #ffdb77 0 8px, transparent 8px 14px); }
        .yield-score { margin: 14px 0; color: #ffe4a0; font-size: clamp(24px, 5vw, 38px); font-weight: 800; text-align: center; }
        .yield-explanation { padding: 12px; border-radius: 7px; background: rgba(255,255,255,.05); }
        .exposure-lab .rpg-action:disabled { opacity: .45; cursor: not-allowed; transform: none; }
        .hidden { display: none !important; }

        @media (max-width: 640px) {
            .main-container { max-width: 100%; }
            .rpg-shell { padding: 12px; }
            .rpg-heading { align-items: flex-start; flex-direction: column; }
            .block-board { grid-template-columns: 1fr; }
            .wavelength-options { grid-template-columns: repeat(2, minmax(0, 1fr)); }
            .alignment-arena { height: 270px; }
            .factory-layout { grid-template-columns: 1fr; }
            .factory-info-column { position: static; max-height: none; overflow: visible; padding-right: 0; }
            .factory-info-column .knowledge-card:not(.hidden) { max-height: none; }
            .tutorial-dialog { padding: 16px; }
        }
    </style>
</head>
<body>

    <div id="splash-screen">
        <div class="scanlines"></div>
        <div class="machine-viewport" id="viewport">
            <canvas id="coverCanvas" width="800" height="420"></canvas>
            <div class="intro-overlay" id="introOverlay">
                <div class="intro-title">即將進入高能真空曝光腔體</div>
                <button class="btn-enter-machine" onclick="startApproachAnimation()">靠近並啟動機台</button>
            </div>
        </div>
    </div>

    <header>
        <h1>晶圓廠探索</h1>
        <p class="subtitle">從光阻、EUV 曝光到晶片堆疊，親自探索半導體製程</p>
    </header>

    <div class="main-container" id="mainApp">
        <section class="rpg-shell" id="rpgSection">
            <section id="knowledgeIntro" class="knowledge-intro" aria-labelledby="knowledgeIntroTitle">
                <h2 id="knowledgeIntroTitle">開始前先認識半導體微影</h2>
                <p>晶片不是把零件直接黏在矽上，而是用薄膜、微影與蝕刻反覆製作微小圖案。先認識下列材料與步驟，再進入工廠操作。</p>
                <div class="term-grid">
                    <article class="term-card"><strong>矽晶圓</strong><p>由高純度矽製成的圓片，是製作晶片的基板；電晶體與導線會逐層形成在表面。</p></article>
                    <article class="term-card"><strong>光阻劑</strong><p>會因曝光而改變化學性質的薄膜。顯影後留下圖案，暫時保護底下不希望被加工的區域。</p></article>
                    <article class="term-card"><strong>光罩</strong><p>帶有電路圖形的模板。曝光時將圖案投影到光阻上；它是圖案模板，不是最後留在晶片內的材料層。</p></article>
                    <article class="term-card"><strong>軟烤、曝光後烘烤與硬烤</strong><p>軟烤在曝光前去除光阻溶劑；部分光阻會在曝光後烘烤以促進化學反應；顯影後硬烤可加強圖案耐受後續製程的能力。</p></article>
                    <article class="term-card"><strong>曝光與顯影</strong><p>曝光把光罩圖案寫入光阻形成潛像；顯影液選擇性溶解光阻，讓圖案開口顯現。</p></article>
                    <article class="term-card"><strong>蝕刻</strong><p>沿著光阻留下的開口移除下方材料，把圖案轉移到薄膜；之後通常會去除剩餘光阻。</p></article>
                </div>
                <div class="rpg-controls"><button class="rpg-action" id="continueToRolesButton" type="button">了解了，選擇職務</button></div>
            </section>
            <div id="roleSelect" class="hidden">
                <div class="rpg-heading">
                    <div>
                        <h2>晶圓廠新人報到</h2>
                        <p>你將以第一人稱走進半導體工廠，完成職務任務並在現場理解晶片如何製造。</p>
                    </div>
                    <span class="key-hint">操作：W 前進 · S 後退 · E 互動 · 完成站點後 D 右轉</span>
                </div>
                <p class="rpg-intro">選擇你的職務背景，跟著資深工程師走進不同廠區。你會看到製程如何運作，親自操作簡化設備，從現象一步步理解晶片如何形成。</p>
                <div class="role-grid" id="roleChoices"></div>
            </div>
            <div id="factoryExplore" class="hidden">
                <div class="rpg-topline">
                    <strong id="roleName"></strong>
                    <span id="questProgress"></span>
                    <button class="rpg-action" id="changeRoleButton" type="button">重新選擇職務</button>
                    <button class="rpg-action hint-button" id="hintButton" type="button">提示（30 秒）</button>
                </div>
                <aside id="hintPanel" class="hint-panel hidden" aria-live="polite">
                    <h3>場效電晶體：電壓控制的開關</h3>
                    <p>以 MOSFET 為例，閘極與通道之間隔著絕緣層。閘極電壓產生電場，改變半導體表面的載子分布；當通道形成，源極與汲極之間可以導電，電晶體就像「開」的開關。移除或反轉控制電壓，通道消失或變弱，電流被關閉。大量電晶體以這種方式組成邏輯電路。</p>
                    <button class="rpg-action" id="closeHintButton" type="button">關閉原理說明</button>
                </aside>
                <div class="factory-layout">
                    <section class="factory-visual-column" aria-label="工廠場景與操作">
                        <div class="rpg-controls" aria-label="音效設定">
                            <button class="rpg-action" id="voiceToggleButton" type="button" aria-pressed="true">語音朗讀：開</button>
                            <button class="rpg-action" id="soundToggleButton" type="button" aria-pressed="true">遊戲音效：開</button>
                        </div>
                        <canvas id="factoryCanvas" width="960" height="420" aria-label="第一人稱半導體工廠探索畫面"></canvas>
                        <div class="rpg-controls">
                            <button class="rpg-action" id="moveBackButton" type="button">後退 S</button>
                            <button class="rpg-action" id="interactButton" type="button">互動 E</button>
                            <button class="rpg-action" id="turnRightButton" type="button" disabled>右轉 D</button>
                            <button class="rpg-action" id="moveForwardButton" type="button">前進 W</button>
                        </div>
                        <div class="rpg-footer">按住 W 前進、S 後退；抵達站點按 E。完成站點後按 D 向右轉，再按 W 前往下一站。對話按 Enter 繼續。</div>
                    </section>
                    <aside class="factory-info-column" aria-label="站點位置與任務對話">
                        <div class="station-list" id="stationList"></div>
                        <div class="knowledge-card hidden" id="knowledgeCard" aria-live="polite"></div>
                    </aside>
                </div>
            </div>
            <section id="blockLab" class="block-lab hidden" aria-labelledby="blockLabTitle">
                <h2 id="blockLabTitle">微影積木實驗室</h2>
                <p>把薄膜與光阻拖到晶圓上，再放好光罩並依序執行製程。觸控裝置可先點選積木，再點目標區。</p>
                <div class="block-board">
                    <div class="block-panel">
                        <h3>材料積木</h3>
                        <div class="block-tray" id="blockTray">
                            <button class="material-block" type="button" draggable="true" data-block-id="film">二氧化矽薄膜</button>
                            <button class="material-block" type="button" draggable="true" data-block-id="resist">光阻劑</button>
                            <button class="material-block" type="button" draggable="true" data-block-id="mask">光罩</button>
                            <button class="material-block" type="button" draggable="true" data-block-id="gate">閘極導體</button>
                        </div>
                        <h3 style="margin-top:16px">光罩座</h3>
                        <div class="mask-slot" id="maskDropZone" data-drop-target="mask">將光罩積木拖曳到這裡</div>
                        <h3 style="margin-top:16px">晶圓剖面（2D 示意）</h3>
                        <div class="wafer-drop-zone" id="waferDropZone" data-drop-target="wafer" aria-label="拖曳薄膜與光阻至晶圓剖面圖">
                            <div class="wafer-model" id="waferModel">
                                <div class="silicon-base">矽晶圓基板</div>
                            </div>
                        </div>
                    </div>
                    <div class="block-panel">
                        <h3>製程操作</h3>
                        <p>每個步驟會說明材料如何變化。請依序完成軟烤、曝光、曝光後烘烤、顯影、硬烤、蝕刻、去光阻，再堆上閘極導體。</p>
                        <div class="process-buttons">
                            <button class="rpg-action" type="button" data-process-step="softBake">軟烤</button>
                            <button class="rpg-action" type="button" data-process-step="exposure">曝光</button>
                            <button class="rpg-action" type="button" data-process-step="postBake">曝光後烘烤</button>
                            <button class="rpg-action" type="button" data-process-step="develop">顯影</button>
                            <button class="rpg-action" type="button" data-process-step="hardBake">硬烤</button>
                            <button class="rpg-action" type="button" data-process-step="etch">蝕刻</button>
                            <button class="rpg-action" type="button" data-process-step="strip">去除光阻</button>
                        </div>
                        <div id="blockStatus" aria-live="polite">第一步：將「二氧化矽薄膜」拖到矽晶圓上。</div>
                        <button class="rpg-action hidden" id="finishBlockLabButton" type="button">完成積木製程，返回工廠</button>
                    </div>
                </div>
            </section>
            <section id="exposureLab" class="exposure-lab hidden" aria-labelledby="exposureLabTitle">
                <h2 id="exposureLabTitle">黃光區微影曝光實驗</h2>
                <p>親手塗布光阻、對準光罩，再選擇波長與曝光時間。每個結果都是教學示意，不代表實際機台配方。</p>
                <div class="exposure-steps" aria-label="曝光流程">
                    <span class="exposure-step-indicator active" data-exposure-indicator="coat">1 塗布光阻</span>
                    <span class="exposure-step-indicator" data-exposure-indicator="align">2 對焦與對準</span>
                    <span class="exposure-step-indicator" data-exposure-indicator="expose">3 曝光控制</span>
                    <span class="exposure-step-indicator" data-exposure-indicator="result">4 良率結算</span>
                </div>
                <section id="coatingStep" class="exposure-step-panel">
                    <h3>第一步：滑動塗布光阻</h3>
                    <p class="exposure-instructions">在矽晶圓圓面上用滑鼠或手指來回塗抹。目標平均厚度約 0.8–1.3；覆蓋太少、塗層太薄或反覆塗得太厚都會降低良率。紫色代表已塗區，偏粉色表示塗層較厚。</p>
                    <div class="coating-canvas-wrap">
                        <canvas id="coatingCanvas" width="560" height="360" aria-label="在矽晶圓上滑動塗布光阻"></canvas>
                    </div>
                    <div class="exposure-metrics">
                        <span>覆蓋率 <strong id="coatCoverage">0%</strong></span>
                        <span>平均厚度 <strong id="coatThickness">0.00</strong></span>
                        <span>厚度判定 <strong id="coatThicknessStatus">待塗布</strong></span>
                        <span>均勻度 <strong id="coatUniformity">待塗布</strong></span>
                    </div>
                    <div class="exposure-actions">
                        <button class="rpg-action" id="resetCoatingButton" type="button">重新塗布</button>
                        <button class="rpg-action" id="continueToAlignmentButton" type="button" disabled>完成塗布，進入對準</button>
                    </div>
                </section>
                <section id="alignmentStep" class="exposure-step-panel hidden">
                    <h3>第二步：拖曳光罩對準基準點</h3>
                    <p class="exposure-instructions">拖曳金色光罩，讓光罩中央十字與晶圓中央十字重合。位置偏差會造成圖案錯位。</p>
                    <div class="alignment-arena" id="alignmentArena" aria-label="拖曳光罩對準晶圓基準點">
                        <div class="alignment-wafer" aria-label="矽晶圓">
                            <span class="wafer-fiducial" aria-hidden="true"></span>
                        </div>
                        <div class="alignment-mask" id="alignmentMask" role="slider" tabindex="0" aria-label="可拖曳光罩對準晶圓基準點" aria-valuetext="尚未對準">
                            EUV 光罩 <span class="mask-fiducial" aria-hidden="true"></span>
                        </div>
                    </div>
                    <p class="align-status" id="alignmentStatus" aria-live="polite">拖曳光罩，將兩個十字對齊。</p>
                    <div class="exposure-actions">
                        <button class="rpg-action" id="continueToExposureButton" type="button" disabled>對準完成，設定曝光</button>
                    </div>
                </section>
                <section id="exposureControlStep" class="exposure-step-panel hidden">
                    <h3>第三步：選擇光源與曝光時間</h3>
                    <p class="exposure-instructions">比較不同波長的解析能力。波長越短，示意線寬越細；曝光時間則影響線條是否完整或過度擴張。</p>
                    <div class="wavelength-options" id="wavelengthOptions" role="group" aria-label="光源波長">
                        <button class="wavelength-option" type="button" data-wavelength="550" aria-pressed="false">可見光<small>550 nm</small></button>
                        <button class="wavelength-option" type="button" data-wavelength="365" aria-pressed="false">UV<small>365 nm</small></button>
                        <button class="wavelength-option" type="button" data-wavelength="248" aria-pressed="false">DUV<small>248 nm</small></button>
                        <button class="wavelength-option" type="button" data-wavelength="13.5" aria-pressed="true">EUV<small>13.5 nm</small></button>
                    </div>
                    <div class="exposure-control">
                        <label for="exposureTimeInput"><span>曝光時間</span><strong><output id="exposureTimeValue">1.1 秒</output></strong></label>
                        <input id="exposureTimeInput" type="range" min="0.5" max="2" step="0.1" value="1.1">
                    </div>
                    <div class="exposure-preview" id="exposurePreview" aria-live="polite"></div>
                    <div class="exposure-actions">
                        <button class="rpg-action" id="runExposureButton" type="button">開始曝光並結算</button>
                    </div>
                </section>
                <section id="exposureResultStep" class="exposure-step-panel hidden" aria-live="polite">
                    <h3>第四步：曝光結果與良率</h3>
                    <div class="yield-score" id="yieldScore"></div>
                    <div class="circuit-cross-section" aria-label="放大一萬倍的電路剖面示意">
                        <div class="cross-section-label">電路剖面示意 · 放大 10,000 倍</div>
                        <div class="cross-section-layer resist-layer">光阻圖案與曝光邊界</div>
                        <div class="circuit-traces" id="circuitTraces" aria-label="曝光形成的電路線條"></div>
                        <div class="cross-section-layer oxide-layer">絕緣薄膜</div>
                        <div class="cross-section-layer silicon-layer">矽晶圓基板</div>
                    </div>
                    <div class="yield-explanation" id="yieldExplanation"></div>
                    <div class="exposure-actions">
                        <button class="rpg-action" id="retryExposureButton" type="button">重新操作</button>
                        <button class="rpg-action" id="finishExposureButton" type="button">完成任務，返回工廠</button>
                    </div>
                </section>
            </section>
        </section>
        <div id="gameplayTutorial" class="tutorial-overlay hidden" role="dialog" aria-modal="true" aria-labelledby="tutorialTitle">
            <div class="tutorial-dialog">
                <h2 id="tutorialTitle">操作方式</h2>
                <ul>
                    <li><strong>W</strong>：面向前方直線前進，靠近設備時畫面會放大；<strong>S</strong>：後退。</li>
                    <li>抵達定點後按 <strong>E</strong> 互動並完成站點；只有完成當前站點後才能按 <strong>D</strong> 向右轉。</li>
                    <li>右轉後繼續按 <strong>W</strong>，穿過前方投影螢幕，走到下一個定點。</li>
                    <li id="roleTutorialHint">依照職務提示完成站點操作；不熟悉時可使用右上角提示，冷卻 30 秒。</li>
                </ul>
                <button class="rpg-action" id="beginExplorationButton" type="button">開始探索</button>
            </div>
        </div>
        <section id="simulatorSection" class="hidden">
        <div class="guide-box">
            <div><strong>當前任務：</strong> <span id="levelTitle">加載中...</span></div>
            <div><strong>提示：</strong>按 <strong>[Space 空白鍵]</strong> 發射高能電子束</div>
        </div>

        <div id="canvas-container">
            <canvas id="simCanvas" width="920" height="480"></canvas>
        </div>

        <div class="controls-panel" id="controlsContainer"></div>
        <div class="rpg-controls">
            <button class="rpg-action" id="returnToFactoryButton" type="button">返回工廠探索</button>
        </div>
        </section>
    </div>

<script>
/*
================================================================================
                            遊戲步驟流程圖 (Flowchart)
================================================================================

[玩家進入網頁]
       │
       ▼
┌───────────────┐
│  開場動畫畫面  │
│ (封面與微影機) │
└───────┬───────┘
       │
       ▼
 點擊「靠近並啟動機台」
       │
       ▼
┌───────────────┐
│ 鏡頭縮放動畫  │
│ (進入真空腔體) │
└───────┬───────┘
       │
       ▼
┌───────────────┐
│ 載入目前關卡  │
│ (初始化光路與 │
│  反射鏡角度)  │
└───────┬───────┘
       │
       ├─────────────────────────────────┐
       ▼                                 │
┌───────────────┐                        │
│ 調整元件角度  │                        │
│ (滑動角度或輸入數值) │                        │
└───────┬───────┘                        │
       │                                 │
       ▼                                 │
按下 [空白鍵] 或發射按鈕                  │
       │                                 │
       ▼                                 │
┌───────────────┐                        │
│ 電子槍發射    │                        │
│ 高能電子束    │                        │
└───────┬───────┘                        │
       │                                 │
       ▼                                 │
 命中錫滴 (Tin Droplet)？                │
       ├─────────────────┐               │
     [否]               [是]             │
       │                 │               │
       ▼                 ▼               │
 光束持續前進      產生電漿並激發        │
 直至離開螢幕      EUV 極紫外光束         │
                         │               │
                         ▼               │
                   ┌───────────┐         │
                   │ 光學路徑  │         │
                   │ 反射計算  │         │
                   └─────┬─────┘         │
                         │               │
                         ▼               │
               ┌──────────────────┐      │
               │ 光束命中晶圓判定 │      │
               └────────┬─────────┘      │
                        │                │
       ┌────────────────┼────────────────┐
       ▼                ▼                ▼
   [0 條命中]      [1-4 條命中]      [5 條全中]
       │                │                │
       ▼                ▼                ▼
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│ 顯示失敗訊息│  │ 顯示部分偏離│  │ 顯示成功訊息│
│ (未命中晶圓)│  │ (調整角度)  │  │ (累計過關數)│
└──────┬──────┘  └──────┬──────┘  └──────┬──────┘
       │                │                │
       └────────────────┴───────┬────────┘
                                │
                                ▼
                           是否過關？
                                │
                      ┌─────────┴─────────┐
                    [失敗/部分]         [成功]
                      │                   │
                      └────────┐          ▼
                               │   進入下一關卡
                               │   (載入新配置)
                               │          │
                               └──────────┘

================================================================================
*/

// --- 開場動畫 ---
const coverCanvas = document.getElementById('coverCanvas');
const cctx = coverCanvas.getContext('2d');
const viewport = document.getElementById('viewport');
const splashScreen = document.getElementById('splash-screen');
const mainApp = document.getElementById('mainApp');

let cameraZ = 0;
let cameraShake = 0;
let isApproaching = false;
let isStarted = false;

function renderMachineExternal(zoom) {
    cctx.clearRect(0, 0, coverCanvas.width, coverCanvas.height);
    cctx.fillStyle = '#020408';
    cctx.fillRect(0, 0, coverCanvas.width, coverCanvas.height);

    cctx.save();
    let shakeX = (Math.random() - 0.5) * cameraShake;
    let shakeY = (Math.random() - 0.5) * cameraShake;
    
    cctx.translate(coverCanvas.width / 2 + shakeX, coverCanvas.height / 2 + shakeY);
    let scale = 1 + zoom * 0.012;
    cctx.scale(scale, scale);
    cctx.translate(-coverCanvas.width / 2, -coverCanvas.height / 2);

    cctx.strokeStyle = 'rgba(0, 240, 255, 0.3)';
    cctx.lineWidth = 2;
    cctx.strokeRect(100, 40, 600, 340);
    cctx.strokeStyle = 'rgba(157, 78, 221, 0.2)';
    cctx.strokeRect(130, 65, 540, 290);

    cctx.fillStyle = '#0d131f';
    cctx.fillRect(140, 75, 520, 270);

    cctx.fillStyle = '#00205b';
    cctx.fillRect(160, 90, 480, 40);
    
    cctx.fillStyle = '#ffffff';
    cctx.font = '900 22px Arial';
    cctx.fillText("半導體", 180, 118);
    cctx.font = '12px Arial';
    cctx.fillStyle = '#00f0ff';
    cctx.fillText("TWINSCAN NXE:3600D — HIGH-NA EUV SYSTEM", 260, 115);

    let portalGlow = Math.min(1, zoom / 200);
    cctx.fillStyle = `rgba(15, 23, 42, ${1 - portalGlow * 0.8})`;
    cctx.fillRect(250, 160, 300, 150);
    
    cctx.strokeStyle = `rgba(0, 240, 255, ${0.4 + portalGlow * 0.6})`;
    cctx.lineWidth = 3;
    cctx.strokeRect(250, 160, 300, 150);

    cctx.restore();
}

window.addEventListener('mousemove', (e) => {
    if (isApproaching) return;
    let dx = (e.clientX - window.innerWidth / 2) * 0.015;
    let dy = (e.clientY - window.innerHeight / 2) * 0.015;
    viewport.style.transform = `rotateY(${dx}deg) rotateX(${-dy}deg)`;
});

function startApproachAnimation() {
    if (isApproaching || isStarted) return;
    document.getElementById('introOverlay').style.display = 'none';
    isApproaching = true;
    
    function animateZoom() {
        cameraZ += 4;
        cameraShake = Math.min(15, cameraZ * 0.08);
        renderMachineExternal(cameraZ);

        if (cameraZ < 250) {
            requestAnimationFrame(animateZoom);
        } else {
            splashScreen.style.transition = 'opacity 0.6s ease';
            splashScreen.style.opacity = '0';
            setTimeout(() => {
                splashScreen.style.display = 'none';
                mainApp.style.opacity = '1';
                isStarted = true;
            }, 600);
        }
    }
    animateZoom();
}

renderMachineExternal(0);

// --- 模擬器核心與關卡角度/晶圓位置設定 ---
const canvas = document.getElementById('simCanvas');
const ctx = canvas.getContext('2d');
const MIRROR_LENGTH = 90;

let currentLevelIndex = 0;
let successCount = 0;

const levels = [
    {
        name: "第一關：經典光路 (M1:135°, M2修正, 光罩:45°)",
        gun: { x: 70, y: 380, dirX: 1, dirY: 0 },
        tin: { x: 200, y: 380 },
        wafer: { x: 650, y: 380, width: 140, height: 20, isVertical: false },
        elements: [
            { id: 'm1', label: '1. 集光鏡', x: 380, y: 380, currentAngle: 20, min: 0, max: 180, flipNormal: false },
            { id: 'm2', label: '2. 轉折鏡', x: 380, y: 100, currentAngle: 68, min: 0, max: 180, flipNormal: true },
            { id: 'mask', label: '3. EUV光罩', x: 650, y: 100, currentAngle: 54, min: 0, max: 180, isMask: true, flipNormal: false }
        ]
    },
    {
        name: "第二關：右上角垂直晶圓 (轉折鏡角度校正)",
        gun: { x: 70, y: 380, dirX: 1, dirY: 0 },
        tin: { x: 180, y: 380 },
        wafer: { x: 750, y: 120, width: 20, height: 140, isVertical: true },
        elements: [
            { id: 'm1', label: '1. 集光鏡', x: 320, y: 380, currentAngle: 54, min: 0, max: 180, flipNormal: false },
            { id: 'm2', label: '2. 轉折鏡 A', x: 320, y: 120, currentAngle: 21, min: 0, max: 180, flipNormal: true },
            { id: 'm3', label: '3. 轉折鏡 B', x: 550, y: 380, currentAngle: 24, min: 0, max: 180, flipNormal: false },
            { id: 'mask', label: '4. EUV光罩', x: 550, y: 120, currentAngle: 125, min: 0, max: 180, isMask: true, flipNormal: true }
        ]
    },
    {
        name: "第三關：中央垂直晶圓聚焦 (M1:135°, M2修正, Wafer旋轉90°)",
        gun: { x: 70, y: 380, dirX: 1, dirY: 0 },
        tin: { x: 180, y: 380 },
        wafer: { x: 480, y: 250, width: 20, height: 140, isVertical: true },
        elements: [
            { id: 'm1', label: '1. 第一集光鏡', x: 280, y: 380, currentAngle: 127, min: 0, max: 180, flipNormal: false },
            { id: 'm2', label: '2. 高空導光鏡', x: 280, y: 70, currentAngle: 155, min: 0, max: 180, flipNormal: true },
            { id: 'm3', label: '3. 側向平移鏡', x: 750, y: 70, currentAngle: 97, min: 0, max: 180, flipNormal: false },
            { id: 'mask', label: '4. 投射光罩', x: 750, y: 250, currentAngle: 2, min: 0, max: 180, isMask: true, flipNormal: false }
        ]
    },
    {
        name: "第四關：雙鏡中心垂直晶圓 (M1藍條修正, M1:45°, M2:135°, Wafer旋轉90°)",
        gun: { x: 300, y: 60, dirX: 0, dirY: 1 },
        tin: { x: 300, y: 180 },
        wafer: { x: 400, y: 250, width: 20, height: 140, isVertical: true },
        elements: [
            { id: 'm1', label: '1. 頂部集光鏡', x: 300, y: 380, currentAngle: 52, min: 0, max: 180, flipNormal: true },
            { id: 'm2', label: '2. 中繼反彈鏡', x: 500, y: 380, currentAngle: 5, min: 0, max: 180, flipNormal: false },
            { id: 'm3', label: '3. 頂層反射鏡', x: 500, y: 100, currentAngle: 93, min: 0, max: 180, flipNormal: true },
            { id: 'm4', label: '4. 右端導航鏡', x: 800, y: 100, currentAngle: 28, min: 0, max: 180, flipNormal: false },
            { id: 'mask', label: '5. 聚焦光罩', x: 800, y: 250, currentAngle: 50, min: 0, max: 180, isMask: true, flipNormal: false }
        ]
    }
];

let currentLevel = levels[0];
let gun = { x: 70, y: 380, dirX: 1, dirY: 0, width: 40, height: 24 };
let tinDroplet = { x: 200, y: 380, radius: 8, hit: false, plasmaTimer: 0 };
let waferTarget = { x: 650, y: 380, width: 140, height: 20, isVertical: false };
let electrons = [];
let euvBeams = [];
let beamOpacity = 0;
let shotInProgress = false;
let statusState = { text: "機台就緒！請微調角度後按 [空白鍵] 發射", color: "#00ff87", type: "IDLE" };

const jobProfiles = [
    {
        id: 'optics',
        name: 'EUV 光路校準工程師',
        description: '追蹤錫滴如何產生 EUV，逐面調整反射鏡並完成曝光機光路校準。',
        stations: [
            { name: 'EUV 錫滴光源站', title: '任務 1：啟動錫滴電漿光源', lesson: '雷射脈衝照射錫滴形成高溫電漿，電漿放出 EUV 光。集光鏡收集部分輻射，再送往曝光機光學系統。', question: 'EUV 光源如何產生？', choices: ['雷射加熱錫滴形成電漿並放出 EUV', '白光穿過玻璃透鏡', '晶圓加熱後自行發光'], answer: 0 },
            { name: 'EUV 反射鏡校準站', title: '任務 2：逐面校準 EUV 光路', lesson: 'EUV 容易被空氣和一般材料吸收，因此曝光機使用多層膜反射鏡而非玻璃透鏡。逐面調整鏡子，讓光束經過光罩後準確抵達晶圓。', question: 'EUV 曝光機為何使用反射鏡導光？', choices: ['一般玻璃會吸收 EUV', '反射鏡能讓光不需要路徑', '鏡子可以增加光子的波長'], answer: 0 },
            { name: '真空與污染監控站', title: '任務 3：維持光學腔體環境', lesson: '真空可減少 EUV 被氣體分子吸收；控制污染則能維持鏡面的反射效率。', question: 'EUV 曝光機維持真空的主要原因是什麼？', choices: ['避免 EUV 被空氣吸收', '讓晶圓重量變輕', '讓光罩產生電漿'], answer: 0 }
        ]
    },
    {
        id: 'blocks',
        name: '積木式半導體製程工程師',
        description: '拖放材料、操作微影與蝕刻步驟，親手把圖案一層層做在矽晶圓上。',
        stations: [
            { name: '微影積木實驗室', title: '任務 1：用光罩做出薄膜圖案', lesson: '把薄膜與光阻依序放到矽晶圓上。軟烤去除溶劑、曝光把光罩圖案寫入光阻、顯影形成開口，再以硬烤加強圖案並蝕刻底層薄膜。', question: '顯影在圖案製作中負責什麼？', choices: ['選擇性溶解光阻，形成圖案開口', '把光罩永久黏到晶圓上', '讓矽晶圓自行發光'], answer: 0, blockLab: true },
            { name: '蝕刻與圖案轉移台', title: '任務 2：把光阻圖案轉移到薄膜', lesson: '光阻圖案像暫時的遮罩，蝕刻會移除開口下方的材料。蝕刻後去除光阻，底層薄膜仍保留已轉移的結構。', question: '蝕刻後為什麼要去除光阻？', choices: ['光阻是暫時遮罩，不是晶片永久結構', '光阻會變成矽晶圓', '光阻負責提供電源'], answer: 0 },
            { name: '電晶體與量測站', title: '任務 3：認識晶片中的開關', lesson: '電晶體與絕緣、導電材料共同構成電路。場效電晶體利用閘極電壓產生的電場控制通道，決定電流能否通過；量測可確認圖案與元件是否符合設計。', question: '場效電晶體的閘極主要用什麼控制通道？', choices: ['電場', '機械齒輪', '光罩本身的重量'], answer: 0 }
        ]
    },
    {
        id: 'exposure',
        name: '黃光區曝光工程師',
        description: '親手塗布光阻、對準光罩並調整曝光條件，觀察波長如何影響晶片線路與良率。',
        stations: [
            { name: '黃光區微影曝光站', title: '任務 1：完成一次晶片曝光', lesson: '先在矽晶圓上均勻塗布光阻，再將光罩圖案對準晶圓基準點，最後選擇波長與曝光時間。曝光後的光阻圖案可經顯影呈現，並在後續製程轉移到材料層。', question: '為什麼曝光機需要精確對準光罩與晶圓？', choices: ['避免不同層的圖案錯位，影響線路連接', '讓光阻自動變成矽', '增加晶圓的厚度'], answer: 0, exposureLab: true }
        ]
    }
];

let selectedJob = null;
let currentStationIndex = 0;
let completedStations = new Set();
let playerPosition = 0;
let movementKeys = new Set();
let lastMovementFrame = 0;
const STATION_SPACING = 720;
const INTERACTION_RANGE = 155;
let gameplayStarted = false;
let turnAnimationStart = 0;
let hintCooldownUntil = 0;
let hintCooldownTimer = null;
let blockState = null;
let selectedBlockId = null;
let exposureGame = null;
let isPaintingCoating = false;
let lastCoatingPoint = null;
let coatingPointerId = null;
let alignmentDragPointer = null;
const COATING_CANVAS_WIDTH = 560;
const COATING_CANVAS_HEIGHT = 360;
const COATING_GRID_COLUMNS = 56;
const COATING_GRID_ROWS = 36;
const COATING_CELL_SIZE = 10;
let voiceEnabled = true;
let soundEnabled = true;
let audioContext = null;
let footstepsTimer = null;

function stopSpeech() {
    if ('speechSynthesis' in window) window.speechSynthesis.cancel();
}

function speakText(text) {
    if (!voiceEnabled || !('speechSynthesis' in window) || !text) return;
    window.speechSynthesis.cancel();
    const utterance = new SpeechSynthesisUtterance(text);
    utterance.lang = 'zh-TW';
    utterance.rate = 1.35;
    const voices = window.speechSynthesis.getVoices();
    const chineseVoice = voices.find(voice => voice.lang.toLowerCase().startsWith('zh-tw')) ||
        voices.find(voice => voice.lang.toLowerCase().startsWith('zh'));
    if (chineseVoice) utterance.voice = chineseVoice;
    window.speechSynthesis.speak(utterance);
}

function getAudioContext() {
    if (!soundEnabled) return null;
    const AudioContextConstructor = window.AudioContext || window.webkitAudioContext;
    if (!AudioContextConstructor) return null;
    if (!audioContext) audioContext = new AudioContextConstructor();
    if (audioContext.state === 'suspended') audioContext.resume();
    return audioContext;
}

function playTone(frequency, duration = 0.09, type = 'sine', volume = 0.035, delay = 0) {
    const context = getAudioContext();
    if (!context) return;
    const oscillator = context.createOscillator();
    const gain = context.createGain();
    const startAt = context.currentTime + delay;
    oscillator.type = type;
    oscillator.frequency.setValueAtTime(frequency, startAt);
    gain.gain.setValueAtTime(0.0001, startAt);
    gain.gain.exponentialRampToValueAtTime(volume, startAt + 0.012);
    gain.gain.exponentialRampToValueAtTime(0.0001, startAt + duration);
    oscillator.connect(gain);
    gain.connect(context.destination);
    oscillator.start(startAt);
    oscillator.stop(startAt + duration + 0.02);
}

function playGameSound(eventName) {
    if (!soundEnabled) return;
    if (eventName === 'step') {
        playTone(105, 0.045, 'triangle', 0.018);
    } else if (eventName === 'interact') {
        playTone(520, 0.07, 'sine', 0.03);
        playTone(760, 0.09, 'sine', 0.025, 0.07);
    } else if (eventName === 'station') {
        playTone(440, 0.11, 'sine', 0.03);
        playTone(660, 0.14, 'sine', 0.03, 0.1);
    } else if (eventName === 'activity') {
        playTone(260, 0.18, 'triangle', 0.035);
        playTone(390, 0.22, 'triangle', 0.028, 0.16);
    } else if (eventName === 'complete') {
        playTone(523, 0.12, 'sine', 0.035);
        playTone(659, 0.12, 'sine', 0.035, 0.1);
        playTone(784, 0.2, 'sine', 0.04, 0.2);
    } else if (eventName === 'locked') {
        playTone(155, 0.12, 'square', 0.022);
        playTone(125, 0.15, 'square', 0.02, 0.12);
    }
}

function startFootsteps() {
    if (!soundEnabled || footstepsTimer !== null) return;
    footstepsTimer = window.setInterval(() => {
        if (movementKeys.size) playGameSound('step');
        else stopFootsteps();
    }, 360);
}

function stopFootsteps() {
    if (footstepsTimer === null) return;
    window.clearInterval(footstepsTimer);
    footstepsTimer = null;
}
let dialogueIndex = 0;
let stationActivity = null;
let activityInProgress = false;
let activityComplete = false;
let factoryAnimationFrame = 0;
let activityTimer = null;

const stationStories = {
    'process-0': {
        dialogue: [
            ['林工程師', '歡迎來到塗佈區。這片矽晶圓很平，但我們要先在上面準備一層非常薄的「感光材料」。'],
            ['新人', '就是像底片一樣，先塗上去，之後光才有地方留下圖案嗎？'],
            ['林工程師', '正是如此。這種材料叫光阻。現在看著晶圓旋轉，我們一起把液態光阻攤成均勻薄膜。']
        ],
        briefing: '光阻會把曝光光能轉成化學變化。旋轉塗佈讓液體向外攤開；膜厚不均可能讓不同區域的曝光與顯影結果不一致。',
        actionKey: 'coat',
        actionLabel: '啟動旋轉塗佈，觀察薄膜如何鋪開',
        result: '觀察重點：晶圓旋轉後液體逐漸向外攤平。實際製程還會控制光阻用量、轉速與烘烤條件，確保薄膜厚度與附著性。'
    },
    'process-1': {
        dialogue: [
            ['林工程師', '塗好光阻後，晶圓會送進曝光機。曝光機不是拿刀去刻晶圓，而是把設計好的圖案投影到光阻上。'],
            ['新人', '那麼光罩就像投影片，光把它的圖案帶到晶圓上？'],
            ['林工程師', '可以這樣理解。EUV 波長約 13.5 奈米，光罩上的圖案經反射光學系統縮小投影，光阻受光區域會發生化學變化。']
        ],
        briefing: 'EUV 光子能量高、波長短，但極易被空氣和一般材料吸收，所以曝光機內採真空與反射式多層膜鏡面。曝光只改變光阻，不會直接雕刻出完整電路。',
        actionKey: 'expose',
        actionLabel: '開始一次曝光，觀察圖案如何寫入光阻',
        result: '曝光完成：光罩圖案以光學方式轉印到光阻，形成肉眼看不見的潛像。下一步要靠顯影把潛像變成實際的光阻圖案。'
    },
    'process-2': {
        dialogue: [
            ['林工程師', '曝光後的圖案還藏在光阻的化學狀態裡。現在來到顯影區，這一步會選擇性溶解光阻。'],
            ['新人', '所以沖洗之後留下的不是矽電路本身，而是可以拿來當遮罩的圖案？'],
            ['林工程師', '沒錯。接著蝕刻會移除沒有被光阻保護的材料，圖案才轉移到下方薄膜。最後再移除光阻，進入下一層製程。']
        ],
        briefing: '微影是「圖案化」流程的一部分：塗光阻 → 曝光 → 顯影 → 蝕刻/沉積等圖案轉移。先形成光阻圖案，才有辦法精確處理下層材料。',
        actionKey: 'develop',
        actionLabel: '開始顯影，揭開光阻潛像',
        result: '顯影後，光阻上出現清晰的線條與開口。這些開口會決定下方材料哪裡被蝕刻或加工；製程工程師接著量測線寬與缺陷。'
    },
    'blocks-0': {
        dialogue: [
            ['林工程師', '我們先用積木把薄膜、光阻和光罩擺好。記住：光罩負責提供圖案，真正留在晶圓上的不是光罩。'],
            ['新人', '所以要先準備底層，再塗上感光的光阻，讓曝光和顯影形成開口？'],
            ['林工程師', '沒錯。請把薄膜與光阻拖到晶圓上，把光罩放到光罩座，再按順序操作右側製程按鈕。']
        ],
        briefing: '這個積木模型簡化呈現正光阻微影：塗佈、軟烤、曝光、曝光後烘烤、顯影、硬烤、蝕刻，最後去除光阻。實際製程會依材料與光阻類型調整條件。',
        actionKey: 'blockLab',
        actionLabel: '開啟積木實驗室，開始製作圖案',
        result: '圖案已由光阻轉移到薄膜。光罩和光阻都只是製程中使用的暫時工具；蝕刻後留下的薄膜結構才是這個示意步驟的成果。'
    },
    'blocks-1': {
        dialogue: [
            ['林工程師', '現在檢查蝕刻後的薄膜。開口下方的材料被移除，受光阻保護的區域則留下來。'],
            ['新人', '去掉光阻後，底下的圖案還會保留，對嗎？'],
            ['林工程師', '對。這就是圖案轉移。實際蝕刻還要控制選擇比、方向性與均勻度，避免傷到不該移除的材料。']
        ],
        briefing: '顯影只是把光阻圖案顯現出來；蝕刻才把圖案轉移到下方薄膜。蝕刻完成後會去光阻，並檢查線寬、側壁與缺陷。',
        actionKey: 'etch',
        actionLabel: '啟動圖案轉移與蝕刻檢查',
        result: '薄膜圖案已完成轉移。保護區與開口區的材料變化不同，形成晶片製程中重要的微細結構。'
    },
    'blocks-2': {
        dialogue: [
            ['陳工程師', '許多微影與薄膜步驟重複堆疊後，會形成電晶體與連接它們的導線。'],
            ['新人', '那麼場效電晶體是用電壓控制電流嗎？'],
            ['陳工程師', '正是如此。閘極電壓產生電場，控制源極與汲極之間的通道；通道形成時導通，通道關閉時電流受阻。']
        ],
        briefing: 'MOSFET 可視為電壓控制的開關。閘極與通道之間有絕緣層，閘極電場改變通道載子分布；數十億個這類開關組合後，能執行邏輯與記憶功能。',
        actionKey: 'transistor',
        actionLabel: '觀察場效電晶體開關示意',
        result: '你已把材料圖案化和電晶體開關原理連起來：製程建立結構，結構再決定電路能如何控制電流。'
    },
    'optics-0': {
        dialogue: [
            ['周工程師', '這裡是曝光機的光源端。雷射先照準一串極小的錫滴，錫滴的大小和位置都需要穩定。'],
            ['新人', '錫滴被雷射打中後，為什麼會變成曝光用的光？'],
            ['周工程師', '雷射把錫加熱成高溫電漿，電漿會放出 EUV 輻射。看看這道脈衝如何命中錫滴，再把能量交給集光鏡。']
        ],
        briefing: 'EUV 光源可簡化理解為「雷射 → 錫滴 → 電漿 → EUV」。產生的光很弱且方向分散，集光鏡會收集其中一部分送入後續光學系統。',
        actionKey: 'source',
        actionLabel: '觸發雷射脈衝，觀察錫滴形成電漿',
        result: '錫滴被脈衝擊中後形成短暫電漿並發出 EUV。真實系統會連續控制錫滴噴射、雷射時序與集光效率，維持穩定光源。'
    },
    'optics-1': {
        dialogue: [
            ['周工程師', 'EUV 的波長短，普通玻璃會吸收它，不能像一般相機那樣用玻璃透鏡聚焦。'],
            ['新人', '那曝光機怎麼把光轉彎，還能讓圖案準確落在晶圓上？'],
            ['周工程師', '靠多層膜反射鏡。每面鏡子只反射一部分 EUV，所以角度稍有偏差，後面的光路就會逐步偏離。接下來你來校準。']
        ],
        briefing: '反射遵循入射角等於反射角。鏡面方向決定光束轉向；多個反射鏡依序把光導向光罩與晶圓。調整時要看每一段路徑，而不是只看最後落點。',
        actionKey: 'optics',
        actionLabel: '進入 EUV 光路校準，逐面調整反射鏡',
        result: '校準完成後，回到這裡追蹤真空狀態。光路校準的核心是理解每次反射如何改變下一段光線方向。'
    },
    'optics-2': {
        dialogue: [
            ['周工程師', '光路必須待在真空腔體裡。這不是為了讓光走得更快，而是因為 EUV 很容易被空氣吸收。'],
            ['新人', '鏡面如果沾到微粒或錫污染，也會讓反射變差吧？'],
            ['周工程師', '正確。真空、溫度與污染監控要一起看。啟動抽氣並觀察壓力下降，再檢查光學腔體是否達到穩定條件。']
        ],
        briefing: '降低氣體分子數量可以減少 EUV 傳播時被吸收；維持潔淨也能避免污染物降低鏡面反射率。真空與光學表面狀態共同影響可用光量。',
        actionKey: 'vacuum',
        actionLabel: '啟動抽氣系統，觀察腔體壓力下降',
        result: '腔體逐步進入穩定真空狀態。實際設備會持續監控壓力與污染，並按維護流程管理光學元件。'
    },
    'exposure-0': {
        dialogue: [
            ['陳工程師', '歡迎來到黃光區。今天你要親手完成一次微影曝光：先把光阻均勻塗在矽晶圓上，再對準光罩。'],
            ['新人', '曝光時可以自己選光源嗎？可見光、紫外光和 EUV 做出來的線路會一樣嗎？'],
            ['陳工程師', '波長會影響可形成的細節。你可以比較可見光、UV、DUV 和 EUV，再調整曝光時間，最後看放大的電路剖面和良率。']
        ],
        briefing: '微影曝光包含塗布、對準、曝光與顯影。光阻厚度不均會讓製程表現不穩；光罩與晶圓錯位會讓不同層的電路接不起來。波長越短，通常越能支援較小的圖案尺寸；曝光時間則需要足以形成完整圖案，過度曝光也可能讓線條變寬。實際解析度與良率還受光學系統、光阻材料及製程條件共同影響。',
        actionKey: 'exposureLab',
        actionLabel: '進入黃光區，開始塗布與曝光',
        result: '你已完成一輪曝光實驗，並比較了塗布均勻度、光罩對準、曝光波長與曝光時間對圖案及良率的影響。'
    }
};

function buildRoleChoices() {
    const choices = document.getElementById('roleChoices');
    choices.innerHTML = '';
    jobProfiles.forEach(job => {
        const button = document.createElement('button');
        button.className = 'role-card';
        button.type = 'button';
        const title = document.createElement('strong');
        title.textContent = job.name;
        const description = document.createElement('span');
        description.textContent = job.description;
        button.append(title, description);
        button.addEventListener('click', () => selectJob(job));
        choices.appendChild(button);
    });
}

function selectJob(job) {
    window.clearTimeout(activityTimer);
    selectedJob = job;
    currentStationIndex = 0;
    playerPosition = -STATION_SPACING;
    movementKeys.clear();
    completedStations = new Set();
    gameplayStarted = false;
    turnAnimationStart = 0;
    blockState = null;
    selectedBlockId = null;
    stationActivity = null;
    activityInProgress = false;
    activityComplete = false;
    document.getElementById('roleSelect').classList.add('hidden');
    document.getElementById('factoryExplore').classList.remove('hidden');
    document.getElementById('blockLab').classList.add('hidden');
    document.getElementById('exposureLab').classList.add('hidden');
    document.getElementById('roleName').textContent = `職務：${job.name}`;
    document.getElementById('knowledgeCard').classList.add('hidden');
    document.getElementById('hintPanel').classList.add('hidden');
    const roleHint = document.getElementById('roleTutorialHint');
    roleHint.textContent = job.id === 'blocks'
        ? '積木職務：拖曳材料到晶圓或光罩座，依序操作微影與蝕刻步驟，完成薄膜圖案製作。'
        : job.id === 'exposure'
            ? '黃光區職務：在晶圓上滑動塗布光阻、拖曳光罩對準基準點，再選擇波長與曝光時間。'
            : 'EUV 光路職務：先觀察錫滴電漿光源，再逐面調整反射鏡校準光路。';
    document.getElementById('gameplayTutorial').classList.remove('hidden');
    updateInteractionPrompt();
    buildStationButtons();
    renderFactory();
    transitionFactoryScene();
    updateQuestProgress();
}

function beginExploration() {
    gameplayStarted = true;
    document.getElementById('gameplayTutorial').classList.add('hidden');
}

function transitionFactoryScene() {
    const scene = document.getElementById('factoryCanvas');
    scene.classList.remove('scene-enter');
    void scene.offsetWidth;
    scene.classList.add('scene-enter');
    window.setTimeout(() => scene.classList.remove('scene-enter'), 600);
}

function buildStationButtons() {
    const list = document.getElementById('stationList');
    list.innerHTML = '';
    selectedJob.stations.forEach((station, index) => {
        const button = document.createElement('div');
        button.className = `station-button${completedStations.has(index) ? ' done' : ''}`;
        const label = document.createElement('span');
        label.innerHTML = `<strong>站點 ${index + 1}</strong>${station.name}`;
        const state = document.createElement('span');
        const stationDistance = Math.round(Math.abs(playerPosition - index * STATION_SPACING));
        state.textContent = completedStations.has(index) ? '已完成' :
            (index === currentStationIndex
                ? `目前位置 · ${stationDistance} m`
                : (index <= completedStations.size ? `距離 ${stationDistance} m` : '未解鎖'));
        button.append(label, state);
        list.appendChild(button);
    });
}

function updateQuestProgress() {
    document.getElementById('questProgress').textContent = completedStations.size === selectedJob.stations.length
        ? `職務巡檢完成：${completedStations.size} / ${selectedJob.stations.length}`
        : `任務進度：${completedStations.size} / ${selectedJob.stations.length} · 位置 ${currentStationIndex + 1} / ${selectedJob.stations.length}`;
    buildStationButtons();
}

function updateStationDistances() {
    const stations = document.querySelectorAll('#stationList .station-button');
    stations.forEach((station, index) => {
        const state = station.lastElementChild;
        if (completedStations.has(index)) {
            state.textContent = '已完成';
        } else if (index <= completedStations.size) {
            const distance = Math.round(Math.abs(playerPosition - index * STATION_SPACING));
            state.textContent = index === currentStationIndex
                ? `目前位置 · ${distance} m`
                : `距離 ${distance} m`;
        } else {
            state.textContent = '未解鎖';
        }
    });
}

function getRoleCompletionMessage(station) {
    return `恭喜你已成功體驗完畢「${selectedJob.name}」職業！你完成了「${station.name}」的任務，並走過本職務的全部站點。`;
}

function getFactoryMaxPosition() {
    if (!selectedJob) return 0;
    return Math.min(
        (selectedJob.stations.length - 1) * STATION_SPACING + INTERACTION_RANGE,
        currentStationIndex * STATION_SPACING + INTERACTION_RANGE
    );
}

function getFactoryMinPosition() {
    return currentStationIndex === 0
        ? -STATION_SPACING
        : (currentStationIndex - 1) * STATION_SPACING;
}

function moveThroughFactory(direction, deltaSeconds = 0.08) {
    if (!selectedJob || !gameplayStarted) return;
    const maxPosition = getFactoryMaxPosition();
    const previousPosition = playerPosition;
    playerPosition = Math.max(
        getFactoryMinPosition(),
        Math.min(maxPosition, playerPosition + direction * 210 * deltaSeconds)
    );
    if (previousPosition !== playerPosition && direction > 0 && playerPosition === maxPosition &&
        !completedStations.has(currentStationIndex)) {
        playGameSound('locked');
    }
    updateStationDistances();
    updateInteractionPrompt();
}

function turnRightAtStation() {
    if (!selectedJob || !gameplayStarted) return;
    const atStation = Math.abs(playerPosition - currentStationIndex * STATION_SPACING) <= INTERACTION_RANGE;
    if (!completedStations.has(currentStationIndex) ||
        currentStationIndex >= selectedJob.stations.length - 1 || !atStation) {
        playGameSound('locked');
        return;
    }

    playerPosition = currentStationIndex * STATION_SPACING;
    currentStationIndex++;
    turnAnimationStart = performance.now();
    dialogueIndex = 0;
    stationActivity = null;
    activityInProgress = false;
    activityComplete = false;
    window.clearTimeout(activityTimer);
    document.getElementById('knowledgeCard').classList.add('hidden');
    transitionFactoryScene();
    playGameSound('station');
    updateQuestProgress();
    updateInteractionPrompt();
}

function updateInteractionPrompt() {
    if (!selectedJob) return;
    const distance = Math.abs(playerPosition - currentStationIndex * STATION_SPACING);
    const canInteract = distance <= INTERACTION_RANGE;
    const interactButton = document.getElementById('interactButton');
    const stationLocked = currentStationIndex > completedStations.size;
    const turnButton = document.getElementById('turnRightButton');
    interactButton.disabled = !gameplayStarted || !canInteract || stationLocked;
    interactButton.textContent = !canInteract
        ? `直行前往定點 (${Math.round(distance)} m)`
        : stationLocked ? '前置站點尚未完成'
        : completedStations.has(currentStationIndex) ? '站點已完成' : '互動 E';
    turnButton.disabled = !gameplayStarted ||
        !completedStations.has(currentStationIndex) ||
        currentStationIndex >= selectedJob.stations.length - 1 ||
        !canInteract;
    turnButton.textContent = currentStationIndex >= selectedJob.stations.length - 1
        ? (canInteract ? '最後一站已抵達' : '直行前往最後一站')
        : completedStations.has(currentStationIndex) && canInteract
            ? '右轉前往下一站 D'
            : '右轉 D';
}

function renderFactory() {
    const factoryCanvas = document.getElementById('factoryCanvas');
    const factoryCtx = factoryCanvas.getContext('2d');
    const station = selectedJob.stations[currentStationIndex];
    const w = factoryCanvas.width;
    const h = factoryCanvas.height;
    factoryCtx.clearRect(0, 0, w, h);
    const sceneIndex = currentStationIndex;
    const stationDistance = playerPosition - currentStationIndex * STATION_SPACING;
    const walkBob = movementKeys.size ? Math.sin(performance.now() / 75) * 2.5 : 0;
    factoryCtx.save();
    if (turnAnimationStart) {
        const turnProgress = (performance.now() - turnAnimationStart) / 650;
        if (turnProgress >= 1) {
            turnAnimationStart = 0;
        } else {
            factoryCtx.translate(w / 2, h / 2);
            factoryCtx.rotate(Math.sin(turnProgress * Math.PI) * 0.22);
            factoryCtx.translate(-w / 2, -h / 2);
        }
    }
    factoryCtx.translate(0, walkBob);
    const themes = [
        { ceiling: '#17293a', wall: '#253443', floor: '#101821', glow: '#34e6d0' },
        { ceiling: '#16243b', wall: '#263653', floor: '#101623', glow: '#8f9cff' },
        { ceiling: '#1c2438', wall: '#34374c', floor: '#11151f', glow: '#c58cff' }
    ];
    const theme = selectedJob.id === 'exposure'
        ? { ceiling: '#302516', wall: '#493b27', floor: '#20190f', glow: '#ffd76f' }
        : themes[sceneIndex % themes.length];

    const ceiling = factoryCtx.createLinearGradient(0, 0, 0, h * 0.6);
    ceiling.addColorStop(0, theme.ceiling);
    ceiling.addColorStop(1, theme.wall);
    factoryCtx.fillStyle = ceiling;
    factoryCtx.fillRect(0, 0, w, h * 0.58);
    factoryCtx.fillStyle = theme.floor;
    factoryCtx.fillRect(0, h * 0.58, w, h * 0.42);

    const atmosphericLight = factoryCtx.createRadialGradient(w * 0.5, h * 0.48, 18, w * 0.5, h * 0.48, w * 0.68);
    atmosphericLight.addColorStop(0, `${theme.glow}22`);
    atmosphericLight.addColorStop(1, 'rgba(0,0,0,0)');
    factoryCtx.fillStyle = atmosphericLight;
    factoryCtx.fillRect(0, 0, w, h);

    factoryCtx.fillStyle = theme.wall;
    factoryCtx.beginPath();
    factoryCtx.moveTo(0, 0);
    factoryCtx.lineTo(w * 0.35, h * 0.25);
    factoryCtx.lineTo(w * 0.35, h * 0.75);
    factoryCtx.lineTo(0, h);
    factoryCtx.closePath();
    factoryCtx.fill();
    factoryCtx.beginPath();
    factoryCtx.moveTo(w, 0);
    factoryCtx.lineTo(w * 0.65, h * 0.25);
    factoryCtx.lineTo(w * 0.65, h * 0.75);
    factoryCtx.lineTo(w, h);
    factoryCtx.closePath();
    factoryCtx.fill();

    factoryCtx.strokeStyle = 'rgba(150, 190, 220, 0.16)';
    factoryCtx.lineWidth = 1;
    const floorPhase = ((playerPosition * 0.09) % 52 + 52) % 52;
    for (let i = 1; i <= 7; i++) {
        const y = h * 0.58 + ((i * 36 + floorPhase) % (h * 0.42));
        factoryCtx.beginPath();
        const spread = (y - h * 0.58) * 0.72;
        factoryCtx.moveTo(w * 0.5 - spread, y);
        factoryCtx.lineTo(w * 0.5 + spread, y);
        factoryCtx.stroke();
    }
    for (let i = 0; i <= 8; i++) {
        const bottomX = w * (0.29 + i * 0.052);
        factoryCtx.beginPath();
        factoryCtx.moveTo(w * 0.5, h * 0.58);
        factoryCtx.lineTo(bottomX, h);
        factoryCtx.stroke();
    }

    factoryCtx.strokeStyle = theme.glow;
    factoryCtx.lineWidth = 3;
    factoryCtx.beginPath();
    factoryCtx.moveTo(0, h * 0.12);
    factoryCtx.lineTo(w * 0.35, h * 0.32);
    factoryCtx.moveTo(w, h * 0.12);
    factoryCtx.lineTo(w * 0.65, h * 0.32);
    factoryCtx.stroke();

    const ceilingPhase = playerPosition % 210;
    for (let i = 0; i < 5; i++) {
        const depth = ((i * 210 + ceilingPhase) % 1050) / 1050;
        const x = w * (0.5 + (i % 2 === 0 ? -1 : 1) * (0.08 + depth * 0.28));
        const y = h * (0.07 + depth * 0.28);
        factoryCtx.fillStyle = theme.glow;
        factoryCtx.fillRect(x - 18 - depth * 15, y, 36 + depth * 30, 4 + depth * 2);
    }

    const approachProgress = 1 - Math.min(Math.abs(stationDistance) / STATION_SPACING, 1);
    const approach = 0.58 + approachProgress * 1.05;
    const machineX = w * 0.5;
    const machineY = h * 0.58;
    factoryCtx.save();
    factoryCtx.translate(machineX, machineY);
    factoryCtx.scale(approach, approach);
    factoryCtx.translate(-machineX, -machineY);
    factoryCtx.fillStyle = 'rgba(4, 9, 15, 0.7)';
    factoryCtx.fillRect(30, h * 0.3, 112, 88);
    factoryCtx.strokeStyle = `${theme.glow}99`;
    factoryCtx.strokeRect(30, h * 0.3, 112, 88);
    factoryCtx.fillStyle = theme.glow;
    factoryCtx.font = 'bold 10px sans-serif';
    factoryCtx.textAlign = 'left';
    factoryCtx.fillText(selectedJob.id === 'optics' ? 'OPTICS BAY' : selectedJob.id === 'exposure' ? 'YELLOW-LIGHT BAY' : selectedJob.id === 'process' ? 'LITHO BAY' : 'WAFER LINE', 40, h * 0.3 + 17);
    factoryCtx.fillStyle = '#a9c3d1';
    for (let i = 0; i < 4; i++) {
        factoryCtx.fillRect(41, h * 0.3 + 29 + i * 10, 76 + (i % 2) * 16, 3);
    }
    factoryCtx.fillStyle = 'rgba(5, 10, 16, 0.78)';
    factoryCtx.fillRect(w - 145, h * 0.31, 112, 77);
    factoryCtx.strokeStyle = 'rgba(157, 78, 221, 0.7)';
    factoryCtx.strokeRect(w - 145, h * 0.31, 112, 77);
    factoryCtx.fillStyle = '#b5c9d6';
    factoryCtx.font = '10px sans-serif';
    factoryCtx.fillText('SYSTEM STATUS', w - 135, h * 0.31 + 16);
    for (let i = 0; i < 3; i++) {
        factoryCtx.fillStyle = i === 2 && activityInProgress ? '#ffcf48' : '#00ff87';
        factoryCtx.beginPath();
        factoryCtx.arc(w - 132, h * 0.31 + 30 + i * 13, 3, 0, Math.PI * 2);
        factoryCtx.fill();
        factoryCtx.fillStyle = '#91a8b7';
        factoryCtx.fillRect(w - 123, h * 0.31 + 28 + i * 13, 75, 4);
    }

    factoryCtx.fillStyle = 'rgba(0,0,0,0.38)';
    factoryCtx.beginPath();
    factoryCtx.ellipse(machineX, machineY + 48, 180, 19, 0, 0, Math.PI * 2);
    factoryCtx.fill();
    factoryCtx.fillStyle = '#0a1119';
    factoryCtx.fillRect(machineX - 145, machineY - 105, 290, 165);
    factoryCtx.strokeStyle = '#00cfe0';
    factoryCtx.lineWidth = 3;
    factoryCtx.strokeRect(machineX - 145, machineY - 105, 290, 165);
    factoryCtx.fillStyle = '#15293a';
    factoryCtx.fillRect(machineX - 105, machineY - 78, 210, 110);
    factoryCtx.strokeStyle = 'rgba(157, 78, 221, 0.8)';
    factoryCtx.strokeRect(machineX - 105, machineY - 78, 210, 110);

    const engineerX = w * 0.82;
    const engineerY = h * 0.57;
    factoryCtx.fillStyle = 'rgba(7, 14, 22, 0.85)';
    factoryCtx.beginPath();
    factoryCtx.ellipse(engineerX, engineerY + 37, 32, 8, 0, 0, Math.PI * 2);
    factoryCtx.fill();
    factoryCtx.fillStyle = '#d9e7ef';
    factoryCtx.beginPath();
    factoryCtx.moveTo(engineerX - 19, engineerY + 31);
    factoryCtx.lineTo(engineerX - 14, engineerY - 3);
    factoryCtx.lineTo(engineerX, engineerY - 12);
    factoryCtx.lineTo(engineerX + 14, engineerY - 3);
    factoryCtx.lineTo(engineerX + 20, engineerY + 31);
    factoryCtx.closePath();
    factoryCtx.fill();
    factoryCtx.fillStyle = '#b8d5e2';
    factoryCtx.beginPath();
    factoryCtx.arc(engineerX, engineerY - 23, 13, 0, Math.PI * 2);
    factoryCtx.fill();
    factoryCtx.fillStyle = '#172d3b';
    factoryCtx.beginPath();
    factoryCtx.ellipse(engineerX, engineerY - 24, 14, 7, 0, Math.PI, Math.PI * 2);
    factoryCtx.fill();
    factoryCtx.fillStyle = '#00d7e8';
    factoryCtx.fillRect(engineerX - 5, engineerY + 5, 10, 5);

    if (selectedJob.id === 'process' && currentStationIndex === 0) {
        factoryCtx.fillStyle = '#13202b';
        factoryCtx.fillRect(machineX - 92, machineY - 55, 184, 80);
        factoryCtx.save();
        factoryCtx.translate(machineX, machineY - 14);
        factoryCtx.rotate(0.15);
        factoryCtx.fillStyle = '#b8d9e8';
        factoryCtx.beginPath();
        factoryCtx.ellipse(0, 0, 43, 14, 0, 0, Math.PI * 2);
        factoryCtx.fill();
        factoryCtx.strokeStyle = '#e7fbff';
        factoryCtx.stroke();
        factoryCtx.fillStyle = '#00f0ff';
        factoryCtx.beginPath();
        factoryCtx.arc(0, 0, 5, 0, Math.PI * 2);
        factoryCtx.fill();
        factoryCtx.restore();
        if (stationActivity === 'coat' || activityComplete) {
            factoryCtx.strokeStyle = 'rgba(0, 240, 255, 0.7)';
            factoryCtx.beginPath();
            factoryCtx.ellipse(machineX, machineY - 14, 48, 17, 0, 0, Math.PI * 2);
            factoryCtx.stroke();
        }
    } else if (selectedJob.id === 'process' && currentStationIndex === 1) {
        factoryCtx.fillStyle = '#0a111b';
        factoryCtx.fillRect(machineX - 92, machineY - 55, 184, 80);
        factoryCtx.fillStyle = '#d7e5ef';
        factoryCtx.fillRect(machineX - 39, machineY - 23, 78, 25);
        factoryCtx.fillStyle = '#ffcf48';
        for (let i = -2; i <= 2; i++) {
            factoryCtx.fillRect(machineX + i * 14 - 2, machineY - 18, 4, 15);
        }
        if (stationActivity === 'expose' || activityComplete) {
            factoryCtx.save();
            factoryCtx.globalAlpha = 0.8;
            factoryCtx.fillStyle = '#ac65ff';
            factoryCtx.beginPath();
            factoryCtx.moveTo(machineX - 8, machineY - 65);
            factoryCtx.lineTo(machineX + 8, machineY - 65);
            factoryCtx.lineTo(machineX + 38, machineY - 1);
            factoryCtx.lineTo(machineX - 38, machineY - 1);
            factoryCtx.closePath();
            factoryCtx.fill();
            factoryCtx.restore();
        }
    } else if (selectedJob.id === 'process' && currentStationIndex === 2) {
        factoryCtx.fillStyle = '#101722';
        factoryCtx.fillRect(machineX - 92, machineY - 55, 184, 80);
        factoryCtx.fillStyle = '#b8d9e8';
        factoryCtx.beginPath();
        factoryCtx.ellipse(machineX, machineY - 15, 48, 16, 0, 0, Math.PI * 2);
        factoryCtx.fill();
        const reveal = stationActivity === 'develop' || activityComplete ? 1 : 0.15;
        factoryCtx.strokeStyle = '#392b61';
        factoryCtx.lineWidth = 4;
        for (let i = -3; i <= 3; i++) {
            if (Math.abs(i) / 3 < reveal) {
                factoryCtx.beginPath();
                factoryCtx.moveTo(machineX + i * 10, machineY - 25);
                factoryCtx.lineTo(machineX + i * 10, machineY - 5);
                factoryCtx.stroke();
            }
        }
        if (stationActivity === 'develop' || activityComplete) {
            factoryCtx.fillStyle = 'rgba(0, 210, 255, 0.2)';
            factoryCtx.fillRect(machineX - 52, machineY - 35, 104, 40);
        }
    } else if (selectedJob.id === 'optics' && currentStationIndex === 0) {
        factoryCtx.fillStyle = '#ffcc00';
        factoryCtx.beginPath();
        factoryCtx.arc(machineX, machineY - 20, 18, 0, Math.PI * 2);
        factoryCtx.fill();
        factoryCtx.strokeStyle = '#ffcc00';
        factoryCtx.beginPath();
        factoryCtx.moveTo(machineX - 75, machineY - 45);
        factoryCtx.lineTo(machineX - 20, machineY - 20);
        factoryCtx.lineTo(machineX + 70, machineY - 55);
        factoryCtx.stroke();
        if (stationActivity === 'source' || activityComplete) {
            factoryCtx.strokeStyle = 'rgba(255, 210, 65, 0.9)';
            factoryCtx.lineWidth = 5;
            factoryCtx.beginPath();
            factoryCtx.moveTo(machineX - 110, machineY - 55);
            factoryCtx.lineTo(machineX - 12, machineY - 21);
            factoryCtx.stroke();
            factoryCtx.fillStyle = '#ed91ff';
            factoryCtx.beginPath();
            factoryCtx.arc(machineX - 10, machineY - 20, 13, 0, Math.PI * 2);
            factoryCtx.fill();
        }
    } else if (selectedJob.id === 'optics' && currentStationIndex === 1) {
        factoryCtx.strokeStyle = '#00eaff';
        factoryCtx.lineWidth = 6;
        factoryCtx.beginPath();
        factoryCtx.moveTo(machineX - 75, machineY - 34);
        factoryCtx.lineTo(machineX - 20, machineY - 4);
        factoryCtx.stroke();
        factoryCtx.strokeStyle = '#ffd65a';
        factoryCtx.beginPath();
        factoryCtx.moveTo(machineX - 18, machineY - 4);
        factoryCtx.lineTo(machineX + 68, machineY - 35);
        factoryCtx.stroke();
        factoryCtx.fillStyle = '#9d4edd';
        factoryCtx.fillRect(machineX + 52, machineY - 8, 25, 16);
    } else if (selectedJob.id === 'optics' && currentStationIndex === 2) {
        factoryCtx.strokeStyle = '#65ebff';
        factoryCtx.lineWidth = 3;
        factoryCtx.beginPath();
        factoryCtx.arc(machineX, machineY - 18, 29, Math.PI * 0.75, Math.PI * 1.25);
        factoryCtx.stroke();
        factoryCtx.fillStyle = '#00ff87';
        factoryCtx.beginPath();
        factoryCtx.arc(machineX, machineY - 18, 5, 0, Math.PI * 2);
        factoryCtx.fill();
    } else if (selectedJob.id === 'exposure' && currentStationIndex === 0) {
        factoryCtx.fillStyle = '#263448';
        factoryCtx.beginPath();
        factoryCtx.ellipse(machineX, machineY - 13, 61, 23, 0, 0, Math.PI * 2);
        factoryCtx.fill();
        factoryCtx.fillStyle = stationActivity === 'exposureLab' || activityComplete ? '#b77bd0' : '#60748c';
        factoryCtx.beginPath();
        factoryCtx.ellipse(machineX, machineY - 15, 47, 14, 0, 0, Math.PI * 2);
        factoryCtx.fill();
        factoryCtx.strokeStyle = '#ffe08a';
        factoryCtx.lineWidth = 2;
        factoryCtx.beginPath();
        factoryCtx.ellipse(machineX, machineY - 15, 47, 14, 0, 0, Math.PI * 2);
        factoryCtx.stroke();
        factoryCtx.fillStyle = '#ffd76f';
        factoryCtx.fillRect(machineX - 26, machineY - 72, 52, 5);
        factoryCtx.strokeStyle = 'rgba(255,215,111,0.5)';
        factoryCtx.setLineDash([5, 5]);
        factoryCtx.beginPath();
        factoryCtx.moveTo(machineX, machineY - 66);
        factoryCtx.lineTo(machineX, machineY - 34);
        factoryCtx.stroke();
        factoryCtx.setLineDash([]);
        factoryCtx.strokeStyle = '#fff0bd';
        factoryCtx.beginPath();
        factoryCtx.moveTo(machineX - 10, machineY - 15);
        factoryCtx.lineTo(machineX + 10, machineY - 15);
        factoryCtx.moveTo(machineX, machineY - 24);
        factoryCtx.lineTo(machineX, machineY - 6);
        factoryCtx.stroke();
    } else {
        factoryCtx.fillStyle = '#00f0ff';
        factoryCtx.fillRect(machineX - 70, machineY - 40, 140, 5);
        factoryCtx.fillStyle = '#70dcff';
        factoryCtx.font = 'bold 16px sans-serif';
        factoryCtx.textAlign = 'center';
        factoryCtx.fillText(selectedJob.id === 'process' ? 'LITHOGRAPHY BAY' : 'FAB PROCESS LINE', machineX, machineY + 20);
    }

    if (currentStationIndex > 0) {
        const gateDistance = (currentStationIndex - 0.5) * STATION_SPACING - playerPosition;
        if (gateDistance >= 0) {
            const gateScale = Math.max(0.3, Math.min(1.5, 1.2 - gateDistance / 650));
            const gateWidth = 250 * gateScale;
            const gateHeight = 170 * gateScale;
            const gateX = machineX - gateWidth / 2;
            const gateY = h * 0.39 - gateHeight / 2;
            const gateColor = '#00eaff';
            factoryCtx.fillStyle = 'rgba(2, 8, 16, 0.84)';
            factoryCtx.fillRect(gateX, gateY, gateWidth, gateHeight);
            factoryCtx.strokeStyle = gateColor;
            factoryCtx.lineWidth = Math.max(2, 3 * gateScale);
            factoryCtx.strokeRect(gateX, gateY, gateWidth, gateHeight);
            factoryCtx.fillStyle = 'rgba(0, 234, 255, 0.16)';
            factoryCtx.fillRect(gateX + 10 * gateScale, gateY + 10 * gateScale, gateWidth - 20 * gateScale, gateHeight - 20 * gateScale);
            factoryCtx.fillStyle = gateColor;
            factoryCtx.font = `bold ${Math.max(11, 14 * gateScale)}px sans-serif`;
            factoryCtx.textAlign = 'center';
            factoryCtx.fillText('投影螢幕 · 直行通過', machineX, gateY + gateHeight * 0.55);
        }
    }

    factoryCtx.fillStyle = '#ffffff';
    factoryCtx.font = 'bold 21px sans-serif';
    factoryCtx.textAlign = 'center';
    factoryCtx.fillText(station.name, w * 0.5, 34);
    factoryCtx.fillStyle = '#9fb4c4';
    factoryCtx.font = '13px sans-serif';
    factoryCtx.fillText('無塵室 · 第一人稱巡檢視角', w * 0.5, 57);
    factoryCtx.strokeStyle = 'rgba(255,255,255,0.65)';
    factoryCtx.lineWidth = 2;
    factoryCtx.beginPath();
    factoryCtx.arc(machineX, h * 0.53, 7, 0, Math.PI * 2);
    factoryCtx.moveTo(machineX - 13, h * 0.53);
    factoryCtx.lineTo(machineX + 13, h * 0.53);
    factoryCtx.moveTo(machineX, h * 0.53 - 13);
    factoryCtx.lineTo(machineX, h * 0.53 + 13);
    factoryCtx.stroke();
    factoryCtx.fillStyle = '#00f0ff';
    factoryCtx.font = '12px sans-serif';
    const distance = Math.abs(stationDistance);
    const prompt = distance <= INTERACTION_RANGE
        ? completedStations.has(currentStationIndex) ? '站點完成 · D 向右轉前往下一站' : 'E 互動 · 已抵達定點'
        : `W 直行中 · 距離定點 ${Math.round(distance)} m`;
    factoryCtx.fillText(activityInProgress ? '設備運轉中 · 正在觀察製程變化' : prompt, w * 0.5, h - 16);
    factoryCtx.restore();
    factoryCtx.restore();
    updateInteractionPrompt();
}

function interactWithStation() {
    if (!gameplayStarted) return;
    const distance = Math.abs(playerPosition - currentStationIndex * STATION_SPACING);
    if (distance > INTERACTION_RANGE) {
        updateInteractionPrompt();
        return;
    }
    if (currentStationIndex > completedStations.size) {
        playGameSound('locked');
        const card = document.getElementById('knowledgeCard');
        card.innerHTML = '<h3>巡檢流程尚未完成</h3><p>你可以先在廠區自由移動與觀察，但必須依序完成前面的站點任務，才能開始這裡的操作。</p>';
        card.classList.remove('hidden');
        return;
    }
    const station = selectedJob.stations[currentStationIndex];
    if (completedStations.has(currentStationIndex)) {
        const card = document.getElementById('knowledgeCard');
        card.classList.remove('hidden');
        if (currentStationIndex === selectedJob.stations.length - 1) {
            card.innerHTML = `<h3>職業體驗完成</h3><p class="role-completion-message">${getRoleCompletionMessage(station)}</p>`;
        } else {
            card.innerHTML = `<h3>${station.title}</h3><p>站點紀錄已完成。按 D 向右轉，再按 W 直行前往下一站。</p>`;
        }
        return;
    }
    movementKeys.clear();
    stopFootsteps();
    dialogueIndex = 0;
    stationActivity = null;
    activityInProgress = false;
    activityComplete = false;
    playGameSound('interact');
    renderStoryDialogue();
}

function getCurrentStationStory() {
    return stationStories[`${selectedJob.id}-${currentStationIndex}`];
}

function renderStoryDialogue() {
    const card = document.getElementById('knowledgeCard');
    const station = selectedJob.stations[currentStationIndex];
    const story = getCurrentStationStory();
    card.innerHTML = '';
    card.classList.remove('hidden');
    const heading = document.createElement('h3');
    heading.textContent = station.title;
    card.appendChild(heading);
    if (dialogueIndex < story.dialogue.length) {
        const [speaker, line] = story.dialogue[dialogueIndex];
        const dialogueScene = document.createElement('div');
        dialogueScene.className = 'dialogue-scene';
        const avatar = document.createElement('div');
        avatar.className = 'dialogue-avatar';
        avatar.textContent = speaker === '新人' ? '你' : speaker.slice(0, 1);
        const copy = document.createElement('div');
        copy.className = 'dialogue-copy';
        const speakerName = document.createElement('strong');
        speakerName.textContent = speaker;
        const dialogueText = document.createElement('p');
        dialogueText.textContent = line;
        copy.append(speakerName, dialogueText);
        dialogueScene.append(avatar, copy);
        card.appendChild(dialogueScene);
        speakText(line);
        playGameSound('interact');
        const nextButton = document.createElement('button');
        nextButton.type = 'button';
        nextButton.className = 'rpg-action';
        nextButton.textContent = dialogueIndex === story.dialogue.length - 1
            ? '了解了，看看設備如何運作 (Enter)'
            : '繼續對話 (Enter)';
        nextButton.addEventListener('click', advanceDialogue);
        card.appendChild(nextButton);
        return;
    }

    const briefing = document.createElement('div');
    briefing.className = 'story-briefing';
    const firstPrinciple = document.createElement('p');
    firstPrinciple.innerHTML = '<strong>原理拆解</strong>';
    const explanation = document.createElement('p');
    explanation.textContent = story.briefing;
    const observation = document.createElement('p');
    observation.textContent = `等一下請觀察：${station.lesson}`;
    briefing.append(firstPrinciple, explanation, observation);
    card.appendChild(briefing);
    speakText(`原理拆解。${story.briefing} 等一下請觀察。${station.lesson}`);

    if (activityComplete) {
        showActivityResult(card, station, story);
        return;
    }
    const activityButton = document.createElement('button');
    activityButton.type = 'button';
    activityButton.className = 'rpg-action';
    activityButton.disabled = activityInProgress;
    activityButton.textContent = activityInProgress ? '設備運轉中，請觀察場景變化…' : story.actionLabel;
    activityButton.addEventListener('click', beginStationActivity);
    card.appendChild(activityButton);
}

function advanceDialogue() {
    if (!selectedJob || document.getElementById('knowledgeCard').classList.contains('hidden')) return;
    const story = getCurrentStationStory();
    if (dialogueIndex < story.dialogue.length) {
        dialogueIndex++;
        renderStoryDialogue();
        return;
    }
    const activityButton = document.querySelector('#knowledgeCard button.rpg-action');
    if (activityButton && !activityButton.disabled) {
        activityButton.click();
        return;
    }
    const finishButton = document.querySelector('#knowledgeCard button.rpg-action');
    if (finishButton && !finishButton.disabled) finishButton.click();
}

function beginStationActivity() {
    const story = getCurrentStationStory();
    stationActivity = story.actionKey;
    if (selectedJob.id === 'blocks' && currentStationIndex === 0) {
        movementKeys.clear();
        stopFootsteps();
        stopSpeech();
        resetBlockLab();
        document.getElementById('factoryExplore').classList.add('hidden');
        document.getElementById('blockLab').classList.remove('hidden');
        return;
    }
    if (story.actionKey === 'exposureLab') {
        movementKeys.clear();
        stopFootsteps();
        stopSpeech();
        document.getElementById('factoryExplore').classList.add('hidden');
        document.getElementById('exposureLab').classList.remove('hidden');
        resetExposureLab();
        return;
    }
    if (story.actionKey === 'optics') {
        selectedJob.opticsSuccessStart = successCount;
        document.getElementById('rpgSection').classList.add('hidden');
        document.getElementById('simulatorSection').classList.remove('hidden');
        isStarted = true;
        currentLevelIndex = 0;
        loadLevel(currentLevelIndex);
        return;
    }

    activityInProgress = false;
    activityComplete = false;
    stopSpeech();
    activityComplete = true;
    playGameSound('complete');
    renderStoryDialogue();
}

function resetExposureLab() {
    exposureGame = {
        step: 'coat',
        coating: new Float32Array(COATING_GRID_COLUMNS * COATING_GRID_ROWS),
        coverage: 0,
        averageThickness: 0,
        thicknessVariation: 0,
        coatScore: 0,
        maskX: 0,
        maskY: 0,
        alignmentDistance: Infinity,
        wavelength: 13.5,
        exposureTime: 1.1,
        lineWidth: 0,
        yieldScore: null
    };
    isPaintingCoating = false;
    lastCoatingPoint = null;
    document.getElementById('coatingStep').classList.remove('hidden');
    document.getElementById('alignmentStep').classList.add('hidden');
    document.getElementById('exposureControlStep').classList.add('hidden');
    document.getElementById('exposureResultStep').classList.add('hidden');
    document.getElementById('continueToAlignmentButton').disabled = true;
    document.getElementById('continueToExposureButton').disabled = true;
    document.getElementById('exposureTimeInput').value = '1.1';
    document.getElementById('exposureTimeValue').textContent = '1.1 秒';
    document.querySelectorAll('.wavelength-option').forEach(button => {
        const selected = Number(button.dataset.wavelength) === exposureGame.wavelength;
        button.setAttribute('aria-pressed', String(selected));
    });
    updateExposureStepIndicators('coat');
    drawCoatingWafer();
    updateCoatingMetrics();
    updateExposurePreview();
    requestAnimationFrame(resetAlignmentMaskPosition);
}

function updateExposureStepIndicators(step) {
    if (!exposureGame) return;
    const steps = ['coat', 'align', 'expose', 'result'];
    const activeIndex = steps.indexOf(step);
    document.querySelectorAll('[data-exposure-indicator]').forEach(indicator => {
        const index = steps.indexOf(indicator.dataset.exposureIndicator);
        indicator.classList.toggle('active', index === activeIndex);
        indicator.classList.toggle('done', index < activeIndex);
    });
}

function setExposureStep(step) {
    if (!exposureGame) return;
    exposureGame.step = step;
    document.getElementById('coatingStep').classList.toggle('hidden', step !== 'coat');
    document.getElementById('alignmentStep').classList.toggle('hidden', step !== 'align');
    document.getElementById('exposureControlStep').classList.toggle('hidden', step !== 'expose');
    document.getElementById('exposureResultStep').classList.toggle('hidden', step !== 'result');
    updateExposureStepIndicators(step);
    if (step === 'align') requestAnimationFrame(resetAlignmentMaskPosition);
}

function drawCoatingWafer() {
    const canvas = document.getElementById('coatingCanvas');
    const context = canvas.getContext('2d');
    const centerX = COATING_CANVAS_WIDTH / 2;
    const centerY = COATING_CANVAS_HEIGHT / 2;
    const radius = 130;
    context.clearRect(0, 0, COATING_CANVAS_WIDTH, COATING_CANVAS_HEIGHT);
    context.fillStyle = '#090c13';
    context.fillRect(0, 0, COATING_CANVAS_WIDTH, COATING_CANVAS_HEIGHT);
    context.beginPath();
    context.arc(centerX, centerY, radius, 0, Math.PI * 2);
    context.fillStyle = '#384961';
    context.fill();
    context.save();
    context.beginPath();
    context.arc(centerX, centerY, radius - 1, 0, Math.PI * 2);
    context.clip();
    context.strokeStyle = 'rgba(255,255,255,0.12)';
    context.lineWidth = 1;
    for (let index = 1; index < 6; index++) {
        const offset = index * 36;
        context.beginPath();
        context.moveTo(centerX - radius, centerY - radius + offset);
        context.lineTo(centerX + radius, centerY - radius + offset);
        context.stroke();
        context.beginPath();
        context.moveTo(centerX - radius + offset, centerY - radius);
        context.lineTo(centerX - radius + offset, centerY + radius);
        context.stroke();
    }
    if (exposureGame) {
        for (let row = 0; row < COATING_GRID_ROWS; row++) {
            for (let column = 0; column < COATING_GRID_COLUMNS; column++) {
                const index = row * COATING_GRID_COLUMNS + column;
                const thickness = exposureGame.coating[index];
                if (thickness <= 0) continue;
                const x = column * COATING_CELL_SIZE;
                const y = row * COATING_CELL_SIZE;
                const alpha = Math.min(0.82, 0.22 + thickness * 0.28);
                const color = thickness > 1.55 ? `rgba(255, 104, 151, ${alpha})` : `rgba(217, 134, 255, ${alpha})`;
                context.fillStyle = color;
                context.fillRect(x, y, COATING_CELL_SIZE, COATING_CELL_SIZE);
            }
        }
    }
    context.restore();
    context.strokeStyle = '#bed0e5';
    context.lineWidth = 3;
    context.beginPath();
    context.arc(centerX, centerY, radius, 0, Math.PI * 2);
    context.stroke();
    context.fillStyle = '#f2f5fb';
    context.font = 'bold 15px sans-serif';
    context.textAlign = 'center';
    context.fillText('矽晶圓', centerX, centerY - radius - 14);
    context.font = '12px sans-serif';
    context.fillStyle = 'rgba(255,255,255,0.78)';
    context.fillText('光阻塗布區', centerX, centerY + radius + 24);
}

function getCoatingPoint(event) {
    const canvas = document.getElementById('coatingCanvas');
    const rect = canvas.getBoundingClientRect();
    return {
        x: (event.clientX - rect.left) * COATING_CANVAS_WIDTH / rect.width,
        y: (event.clientY - rect.top) * COATING_CANVAS_HEIGHT / rect.height
    };
}

function applyCoatingDab(point) {
    const waferCenterX = COATING_CANVAS_WIDTH / 2;
    const waferCenterY = COATING_CANVAS_HEIGHT / 2;
    const waferRadius = 130;
    const brushRadius = 25;
    if (Math.hypot(point.x - waferCenterX, point.y - waferCenterY) > waferRadius + brushRadius) return;
    const minColumn = Math.max(0, Math.floor((point.x - brushRadius) / COATING_CELL_SIZE));
    const maxColumn = Math.min(COATING_GRID_COLUMNS - 1, Math.ceil((point.x + brushRadius) / COATING_CELL_SIZE));
    const minRow = Math.max(0, Math.floor((point.y - brushRadius) / COATING_CELL_SIZE));
    const maxRow = Math.min(COATING_GRID_ROWS - 1, Math.ceil((point.y + brushRadius) / COATING_CELL_SIZE));
    for (let row = minRow; row <= maxRow; row++) {
        for (let column = minColumn; column <= maxColumn; column++) {
            const x = column * COATING_CELL_SIZE + COATING_CELL_SIZE / 2;
            const y = row * COATING_CELL_SIZE + COATING_CELL_SIZE / 2;
            if (Math.hypot(x - waferCenterX, y - waferCenterY) > waferRadius) continue;
            if (Math.hypot(x - point.x, y - point.y) > brushRadius) continue;
            const index = row * COATING_GRID_COLUMNS + column;
            exposureGame.coating[index] = Math.min(3, exposureGame.coating[index] + 0.08);
        }
    }
}

function coatAlongPath(point) {
    if (!exposureGame || exposureGame.step !== 'coat') return;
    if (!lastCoatingPoint) {
        applyCoatingDab(point);
        lastCoatingPoint = point;
    } else {
        const distance = Math.hypot(point.x - lastCoatingPoint.x, point.y - lastCoatingPoint.y);
        const samples = Math.max(1, Math.ceil(distance / 8));
        for (let index = 1; index <= samples; index++) {
            const ratio = index / samples;
            applyCoatingDab({
                x: lastCoatingPoint.x + (point.x - lastCoatingPoint.x) * ratio,
                y: lastCoatingPoint.y + (point.y - lastCoatingPoint.y) * ratio
            });
        }
        lastCoatingPoint = point;
    }
    drawCoatingWafer();
    updateCoatingMetrics();
}

function updateCoatingMetrics() {
    if (!exposureGame) return;
    const waferRadius = 130;
    let totalCells = 0;
    let paintedCells = 0;
    let thicknessSum = 0;
    let squareSum = 0;
    for (let row = 0; row < COATING_GRID_ROWS; row++) {
        for (let column = 0; column < COATING_GRID_COLUMNS; column++) {
            const x = column * COATING_CELL_SIZE + COATING_CELL_SIZE / 2;
            const y = row * COATING_CELL_SIZE + COATING_CELL_SIZE / 2;
            if (Math.hypot(x - COATING_CANVAS_WIDTH / 2, y - COATING_CANVAS_HEIGHT / 2) > waferRadius) continue;
            totalCells++;
            const thickness = exposureGame.coating[row * COATING_GRID_COLUMNS + column];
            if (thickness <= 0) continue;
            paintedCells++;
            thicknessSum += thickness;
            squareSum += thickness * thickness;
        }
    }
    const coverage = totalCells ? paintedCells / totalCells : 0;
    const averageThickness = paintedCells ? thicknessSum / paintedCells : 0;
    const variance = paintedCells ? Math.max(0, squareSum / paintedCells - averageThickness ** 2) : 0;
    const thicknessVariation = averageThickness ? Math.sqrt(variance) / averageThickness : 0;
    exposureGame.coverage = coverage;
    exposureGame.averageThickness = averageThickness;
    exposureGame.thicknessVariation = thicknessVariation;
    exposureGame.coatScore = Math.max(0,
        Math.min(25, coverage / 0.95 * 25) +
        Math.max(0, 10 - Math.abs(averageThickness - 1.05) * 12) +
        Math.max(0, 5 - thicknessVariation * 10));
    document.getElementById('coatCoverage').textContent = `${Math.round(coverage * 100)}%`;
    document.getElementById('coatThickness').textContent = averageThickness.toFixed(2);
    document.getElementById('coatThicknessStatus').textContent = coverage < 0.01
        ? '待塗布'
        : averageThickness < 0.8 ? '偏薄' : averageThickness > 1.3 ? '偏厚' : '適中';
    document.getElementById('coatUniformity').textContent = coverage < 0.01
        ? '待塗布'
        : thicknessVariation < 0.22 ? '均勻' : thicknessVariation < 0.48 ? '略不均' : '不均勻';
    document.getElementById('continueToAlignmentButton').disabled = coverage < 0.7;
}

function resetCoatingSurface() {
    if (!exposureGame) return;
    exposureGame.coating.fill(0);
    lastCoatingPoint = null;
    drawCoatingWafer();
    updateCoatingMetrics();
}

function getAlignmentMaskCenter() {
    const mask = document.getElementById('alignmentMask');
    const halfWidth = mask.offsetWidth / 2;
    const halfHeight = mask.offsetHeight / 2;
    return {
        x: exposureGame.maskX + halfWidth,
        y: exposureGame.maskY + halfHeight
    };
}

function resetAlignmentMaskPosition() {
    if (!exposureGame) return;
    const arena = document.getElementById('alignmentArena');
    if (!arena.clientWidth) return;
    exposureGame.maskX = arena.clientWidth * 0.64;
    exposureGame.maskY = arena.clientHeight * 0.37;
    updateAlignmentStatus();
}

function updateAlignmentStatus() {
    if (!exposureGame) return;
    const arena = document.getElementById('alignmentArena');
    const mask = document.getElementById('alignmentMask');
    if (!arena.clientWidth) return;
    const targetX = arena.clientWidth * 0.5;
    const targetY = arena.clientHeight * 0.53;
    const maskCenter = getAlignmentMaskCenter();
    exposureGame.alignmentDistance = Math.hypot(maskCenter.x - targetX, maskCenter.y - targetY);
    mask.style.left = `${exposureGame.maskX}px`;
    mask.style.top = `${exposureGame.maskY}px`;
    const aligned = exposureGame.alignmentDistance <= 16;
    mask.setAttribute('aria-valuetext', aligned ? '已對準晶圓基準點' : `偏移 ${Math.round(exposureGame.alignmentDistance)} 像素`);
    document.getElementById('alignmentStatus').textContent = aligned
        ? '基準點已對準！光罩圖案可以正確投影。'
        : `目前偏移 ${Math.round(exposureGame.alignmentDistance)} px，繼續拖曳直到兩個十字重合。`;
    document.getElementById('continueToExposureButton').disabled = !aligned;
}

function updateExposurePreview() {
    if (!exposureGame) return;
    const baseWidthByWavelength = new Map([[550, 85], [365, 58], [248, 36], [13.5, 13.5]]);
    const wavelength = exposureGame.wavelength;
    const time = exposureGame.exposureTime;
    const lineWidth = baseWidthByWavelength.get(wavelength) * time / 1.1;
    const shorted = lineWidth >= 40;
    const underexposed = time < 0.7;
    exposureGame.lineWidth = lineWidth;
    const waveName = wavelength === 550 ? '可見光' : wavelength === 365 ? 'UV' : wavelength === 248 ? 'DUV' : 'EUV';
    const riskText = shorted
        ? '線寬大於示意線距，圖案容易相連，形成短路風險。'
        : underexposed
            ? '曝光時間偏短，部分線條可能無法完整形成。'
            : time > 1.5
                ? '曝光時間偏長，圖案有擴張風險。'
                : '示意線條可清楚分開，曝光時間接近本模型的建議範圍。';
    document.getElementById('exposurePreview').textContent =
        `${waveName} ${wavelength} nm · 預估線寬約 ${lineWidth.toFixed(1)} nm（線距 40 nm）。${riskText} 這是用來比較趨勢的教學模型，並非真實設備配方。`;
}

function runExposureAndCalculateYield() {
    if (!exposureGame || exposureGame.step !== 'expose') return;
    const underexposed = exposureGame.exposureTime < 0.7;
    const shorted = exposureGame.lineWidth >= 40;
    const alignmentScore = Math.max(0, 20 - exposureGame.alignmentDistance * 0.25);
    const resolutionScore = 15 * Math.max(0, Math.min(1, (40 - exposureGame.lineWidth) / (40 - 13.5)));
    const doseScore = 10 * Math.max(0, 1 - Math.abs(exposureGame.exposureTime - 1.1) / 0.9);
    const integrityScore = shorted || underexposed ? 0 : 15;
    const exposureScore = resolutionScore + doseScore + integrityScore;
    exposureGame.yieldScore = Math.round(Math.min(100, exposureGame.coatScore + alignmentScore + exposureScore));
    const traces = document.getElementById('circuitTraces');
    traces.classList.toggle('shorted', shorted);
    traces.classList.toggle('broken', underexposed);
    traces.replaceChildren();
    const traceWidth = Math.min(90, exposureGame.lineWidth * 0.5);
    for (let index = 0; index < 5; index++) {
        const trace = document.createElement('span');
        trace.style.left = `${4 + index * 20}%`;
        trace.style.width = `${traceWidth}%`;
        traces.appendChild(trace);
    }
    document.getElementById('yieldScore').textContent = `良率得分：${exposureGame.yieldScore} / 100`;
    const coatingMessage = exposureGame.coverage < 0.9
        ? `光阻覆蓋率 ${Math.round(exposureGame.coverage * 100)}%，仍有未完整塗布區域。`
        : `光阻覆蓋率 ${Math.round(exposureGame.coverage * 100)}%，平均厚度 ${exposureGame.averageThickness.toFixed(2)}。`;
    const alignmentMessage = exposureGame.alignmentDistance <= 16
        ? '光罩已對準基準點。'
        : `光罩偏移 ${Math.round(exposureGame.alignmentDistance)} px。`;
    const patternMessage = shorted
        ? '線路過寬並互相接觸，示意為短路。'
        : underexposed
            ? '曝光不足，示意線路斷裂。'
            : '線路彼此分離，圖案清楚。';
    document.getElementById('yieldExplanation').textContent =
        `${coatingMessage} ${alignmentMessage} 使用 ${exposureGame.wavelength} nm 光源、曝光 ${exposureGame.exposureTime.toFixed(1)} 秒，估計線寬約 ${exposureGame.lineWidth.toFixed(1)} nm。${patternMessage} 計分明細：塗布 ${exposureGame.coatScore.toFixed(1)}/40、對準 ${alignmentScore.toFixed(1)}/20、曝光 ${exposureScore.toFixed(1)}/40（解析度 ${resolutionScore.toFixed(1)}/15、曝光時間 ${doseScore.toFixed(1)}/10、線路完整性 ${integrityScore}/15）。真實良率還受更多材料與設備因素影響。`;
    setExposureStep('result');
    playGameSound('complete');
}

function finishExposureLab() {
    if (!exposureGame || exposureGame.yieldScore === null || !selectedJob || selectedJob.id !== 'exposure') return;
    completedStations.add(currentStationIndex);
    activityComplete = true;
    activityInProgress = false;
    document.getElementById('exposureLab').classList.add('hidden');
    document.getElementById('factoryExplore').classList.remove('hidden');
    const card = document.getElementById('knowledgeCard');
    card.innerHTML = '';
    const completionHeading = document.createElement('h3');
    completionHeading.textContent = '黃光區曝光任務完成';
    const scoreMessage = document.createElement('p');
    scoreMessage.textContent = `本次良率得分：${exposureGame.yieldScore} / 100。${getCurrentStationStory().result}`;
    const roleMessage = document.createElement('p');
    roleMessage.className = 'role-completion-message';
    roleMessage.textContent = getRoleCompletionMessage(selectedJob.stations[currentStationIndex]);
    card.append(completionHeading, scoreMessage, roleMessage);
    card.classList.remove('hidden');
    playGameSound('complete');
    speakText(card.textContent);
    updateQuestProgress();
    updateInteractionPrompt();
    renderFactory();
}

const coatingCanvas = document.getElementById('coatingCanvas');
coatingCanvas.addEventListener('pointerdown', event => {
    if (!exposureGame || exposureGame.step !== 'coat') return;
    event.preventDefault();
    isPaintingCoating = true;
    coatingPointerId = event.pointerId;
    coatingCanvas.setPointerCapture(event.pointerId);
    coatAlongPath(getCoatingPoint(event));
});
coatingCanvas.addEventListener('pointermove', event => {
    if (!isPaintingCoating || coatingPointerId !== event.pointerId) return;
    coatAlongPath(getCoatingPoint(event));
});
const stopCoatingPointer = event => {
    if (coatingPointerId !== event.pointerId) return;
    isPaintingCoating = false;
    coatingPointerId = null;
    lastCoatingPoint = null;
};
coatingCanvas.addEventListener('pointerup', stopCoatingPointer);
coatingCanvas.addEventListener('pointercancel', stopCoatingPointer);
coatingCanvas.addEventListener('lostpointercapture', stopCoatingPointer);
document.getElementById('resetCoatingButton').addEventListener('click', resetCoatingSurface);
document.getElementById('continueToAlignmentButton').addEventListener('click', () => {
    if (exposureGame && exposureGame.coverage >= 0.7) setExposureStep('align');
});
document.getElementById('alignmentMask').addEventListener('pointerdown', event => {
    if (!exposureGame || exposureGame.step !== 'align') return;
    event.preventDefault();
    const arena = document.getElementById('alignmentArena').getBoundingClientRect();
    alignmentDragPointer = {
        id: event.pointerId,
        offsetX: event.clientX - arena.left - exposureGame.maskX,
        offsetY: event.clientY - arena.top - exposureGame.maskY
    };
    event.currentTarget.setPointerCapture(event.pointerId);
});
document.getElementById('alignmentMask').addEventListener('pointermove', event => {
    if (!alignmentDragPointer || alignmentDragPointer.id !== event.pointerId) return;
    const arena = document.getElementById('alignmentArena').getBoundingClientRect();
    const halfWidth = event.currentTarget.offsetWidth / 2;
    const halfHeight = event.currentTarget.offsetHeight / 2;
    exposureGame.maskX = Math.max(0, Math.min(arena.width - event.currentTarget.offsetWidth, event.clientX - arena.left - alignmentDragPointer.offsetX));
    exposureGame.maskY = Math.max(0, Math.min(arena.height - event.currentTarget.offsetHeight, event.clientY - arena.top - alignmentDragPointer.offsetY));
    updateAlignmentStatus();
});
const stopAlignmentDrag = event => {
    if (alignmentDragPointer && alignmentDragPointer.id === event.pointerId) alignmentDragPointer = null;
};
document.getElementById('alignmentMask').addEventListener('pointerup', stopAlignmentDrag);
document.getElementById('alignmentMask').addEventListener('pointercancel', stopAlignmentDrag);
document.getElementById('alignmentMask').addEventListener('lostpointercapture', stopAlignmentDrag);
document.getElementById('alignmentMask').addEventListener('keydown', event => {
    if (!['ArrowUp', 'ArrowDown', 'ArrowLeft', 'ArrowRight'].includes(event.key) || !exposureGame) return;
    event.preventDefault();
    const movement = 5;
    if (event.key === 'ArrowLeft') exposureGame.maskX -= movement;
    if (event.key === 'ArrowRight') exposureGame.maskX += movement;
    if (event.key === 'ArrowUp') exposureGame.maskY -= movement;
    if (event.key === 'ArrowDown') exposureGame.maskY += movement;
    const arena = document.getElementById('alignmentArena');
    exposureGame.maskX = Math.max(72, Math.min(arena.clientWidth - 72, exposureGame.maskX));
    exposureGame.maskY = Math.max(38, Math.min(arena.clientHeight - 38, exposureGame.maskY));
    updateAlignmentStatus();
});
document.getElementById('continueToExposureButton').addEventListener('click', () => {
    if (exposureGame && exposureGame.alignmentDistance <= 16) setExposureStep('expose');
});
document.getElementById('wavelengthOptions').addEventListener('click', event => {
    const button = event.target.closest('[data-wavelength]');
    if (!button || !exposureGame) return;
    exposureGame.wavelength = Number(button.dataset.wavelength);
    document.querySelectorAll('.wavelength-option').forEach(option => {
        option.setAttribute('aria-pressed', String(option === button));
    });
    updateExposurePreview();
});
document.getElementById('exposureTimeInput').addEventListener('input', event => {
    if (!exposureGame) return;
    exposureGame.exposureTime = Number(event.currentTarget.value);
    document.getElementById('exposureTimeValue').textContent = `${exposureGame.exposureTime.toFixed(1)} 秒`;
    updateExposurePreview();
});
document.getElementById('runExposureButton').addEventListener('click', runExposureAndCalculateYield);
document.getElementById('retryExposureButton').addEventListener('click', resetExposureLab);
document.getElementById('finishExposureButton').addEventListener('click', finishExposureLab);

function resetBlockLab() {
    blockState = {
        filmPlaced: false,
        resistPlaced: false,
        maskPlaced: false,
        gatePlaced: false,
        softBaked: false,
        exposed: false,
        postBaked: false,
        developed: false,
        hardBaked: false,
        etched: false,
        stripped: false
    };
    selectedBlockId = null;
    document.querySelectorAll('.material-block').forEach(block => block.classList.remove('selected'));
    document.getElementById('finishBlockLabButton').classList.add('hidden');
    document.getElementById('blockStatus').textContent = '第一步：將「二氧化矽薄膜」拖到矽晶圓上。';
    renderBlockStack();
    updateBlockProcessButtons();
}

function showBlockStatus(message) {
    document.getElementById('blockStatus').textContent = message;
    speakText(message);
}

function placeBlock(blockId, target) {
    if (!blockState) return;
    if (target === 'mask' && blockId === 'mask' && !blockState.maskPlaced) {
        blockState.maskPlaced = true;
        showBlockStatus('光罩已放入光罩座。它是曝光時的圖案模板，不會堆進晶圓。');
    } else if (target === 'wafer' && blockId === 'film' &&
        !blockState.filmPlaced && !blockState.resistPlaced) {
        blockState.filmPlaced = true;
        showBlockStatus('二氧化矽薄膜已放到矽晶圓上。接著放上感光光阻。');
    } else if (target === 'wafer' && blockId === 'resist' &&
        blockState.filmPlaced && !blockState.resistPlaced) {
        blockState.resistPlaced = true;
        showBlockStatus('光阻已覆蓋薄膜。曝光後它的化學性質會改變，顯影才會產生開口。');
    } else if (target === 'wafer' && blockId === 'gate' &&
        blockState.stripped && !blockState.gatePlaced) {
        blockState.gatePlaced = true;
        showBlockStatus('閘極導體已堆到圖案化薄膜上。這是電晶體結構的簡化示意，真實元件還包含更多材料與製程。');
    } else {
        showBlockStatus('這個積木目前不能放在該位置。請先放薄膜，再放光阻；光罩要放在光罩座。');
        return;
    }
    selectedBlockId = null;
    document.querySelectorAll('.material-block').forEach(block => block.classList.remove('selected'));
    renderBlockStack();
    updateBlockProcessButtons();
}

function renderBlockStack() {
    if (!blockState) return;
    const model = document.getElementById('waferModel');
    model.replaceChildren();
    if (blockState.resistPlaced && !blockState.stripped) {
        const resist = document.createElement('div');
        resist.className = 'stack-layer resist';
        resist.setAttribute('aria-label', blockState.developed ? '已顯影的光阻圖案' : '光阻層');
        resist.textContent = blockState.developed ? '光阻圖案 · 已顯影' : '光阻層';
        if (blockState.developed) {
            resist.innerHTML = '<div class="resist-pattern"><span></span><span class="gap"></span><span></span></div>';
        }
        model.appendChild(resist);
    }
    if (blockState.gatePlaced) {
        const gate = document.createElement('div');
        gate.className = 'stack-layer gate-layer';
        gate.textContent = '閘極導體';
        model.appendChild(gate);
    }
    if (blockState.filmPlaced) {
        const film = document.createElement('div');
        film.className = 'stack-layer film';
        film.setAttribute('aria-label', blockState.etched ? '已蝕刻的薄膜圖案' : '二氧化矽薄膜');
        if (blockState.etched) {
            film.innerHTML = '<div class="film-pattern"><span>保留</span><span class="gap">蝕刻開口</span><span>保留</span></div>';
        } else {
            film.textContent = '二氧化矽薄膜';
        }
        model.appendChild(film);
    }
    const wafer = document.createElement('div');
    wafer.className = 'silicon-base';
    wafer.textContent = '矽晶圓基板';
    model.appendChild(wafer);

    const maskZone = document.getElementById('maskDropZone');
    maskZone.textContent = blockState.maskPlaced
        ? '光罩已放置 · 圖案朝向晶圓'
        : '將光罩積木拖曳到這裡';
    document.getElementById('finishBlockLabButton').classList.toggle('hidden', !blockState.gatePlaced);
}

function updateBlockProcessButtons() {
    if (!blockState) return;
    const requirements = {
        softBake: blockState.resistPlaced && !blockState.softBaked,
        exposure: blockState.softBaked && blockState.maskPlaced && !blockState.exposed,
        postBake: blockState.exposed && !blockState.postBaked,
        develop: blockState.postBaked && !blockState.developed,
        hardBake: blockState.developed && !blockState.hardBaked,
        etch: blockState.hardBaked && !blockState.etched,
        strip: blockState.etched && !blockState.stripped
    };
    document.querySelectorAll('[data-process-step]').forEach(button => {
        button.disabled = !requirements[button.dataset.processStep];
    });
}

function runBlockProcess(step) {
    if (!blockState) return;
    const messages = {
        softBake: '軟烤：加熱光阻以去除部分溶劑，讓薄膜穩定並提高附著性。',
        exposure: '曝光：光罩把圖案投影到光阻，曝光區域產生化學變化，形成看不見的潛像。',
        postBake: '曝光後烘烤：部分化學放大型光阻會在這一步促進反應，讓潛像與圖案尺寸更穩定；實際流程依材料而定。',
        develop: '顯影：顯影液選擇性溶解光阻，打開薄膜上需要加工的區域。',
        hardBake: '硬烤：顯影後加熱強化剩餘光阻，讓它在蝕刻時更能保護下方薄膜。',
        etch: '蝕刻：移除光阻開口下方的薄膜，留下受保護的材料圖案。',
        strip: '去光阻：移除暫時使用的光阻與光罩，已轉移到薄膜的圖案仍保留。'
    };
    if (!messages[step] || document.querySelector(`[data-process-step="${step}"]`).disabled) return;
    if (step === 'softBake') blockState.softBaked = true;
    if (step === 'exposure') blockState.exposed = true;
    if (step === 'postBake') blockState.postBaked = true;
    if (step === 'develop') blockState.developed = true;
    if (step === 'hardBake') blockState.hardBaked = true;
    if (step === 'etch') blockState.etched = true;
    if (step === 'strip') {
        blockState.stripped = true;
        blockState.resistPlaced = false;
        blockState.maskPlaced = false;
    }
    showBlockStatus(step === 'strip'
        ? `${messages[step]}接著拖入「閘極導體」積木，觀察另一層元件結構。`
        : messages[step]);
    playGameSound('activity');
    renderBlockStack();
    updateBlockProcessButtons();
}

function finishBlockLab() {
    if (!blockState || !blockState.gatePlaced || !selectedJob || selectedJob.id !== 'blocks') return;
    completedStations.add(0);
    activityComplete = true;
    activityInProgress = false;
    document.getElementById('blockLab').classList.add('hidden');
    document.getElementById('factoryExplore').classList.remove('hidden');
    const card = document.getElementById('knowledgeCard');
    card.textContent = '積木製程完成：光罩把圖案寫入光阻，顯影形成開口，蝕刻將圖案轉移到薄膜，去光阻後再堆上閘極導體。這是場效電晶體的簡化層次示意；現在可以按 D 向右轉，前往下一站。';
    card.classList.remove('hidden');
    playGameSound('complete');
    speakText(card.textContent);
    updateQuestProgress();
    updateInteractionPrompt();
    renderFactory();
}

function showActivityResult(card, station, story) {
    const result = document.createElement('div');
    result.className = 'story-briefing';
    const resultHeading = document.createElement('strong');
    resultHeading.textContent = '現場觀察';
    const resultText = document.createElement('p');
    resultText.textContent = story.result;
    result.append(resultHeading, resultText);
    card.appendChild(result);
    speakText(`現場觀察。${story.result}`);

    const finishButton = document.createElement('button');
    finishButton.type = 'button';
    finishButton.className = 'rpg-action';
    finishButton.textContent = '記錄觀察結果，完成站點';
    finishButton.addEventListener('click', () => {
        completedStations.add(currentStationIndex);
        playGameSound('complete');
        updateQuestProgress();
        const completion = document.createElement('p');
        const isFinalStation = currentStationIndex === selectedJob.stations.length - 1;
        completion.textContent = isFinalStation
            ? getRoleCompletionMessage(station)
            : `已完成「${station.name}」巡檢。按 D 向右轉，再按 W 直行前往下一站。`;
        completion.classList.toggle('role-completion-message', isFinalStation);
        if (!isFinalStation) completion.style.color = '#00ff87';
        finishButton.remove();
        card.appendChild(completion);
        speakText(completion.textContent);
    });
    card.appendChild(finishButton);
}

function returnToFactory() {
    document.getElementById('simulatorSection').classList.add('hidden');
    document.getElementById('rpgSection').classList.remove('hidden');
    if (selectedJob && selectedJob.id === 'optics' && successCount - selectedJob.opticsSuccessStart >= levels.length) {
        completedStations.add(1);
        playGameSound('complete');
        speakText('EUV 光路校準完成。你已完成四組曝光路徑，下一個區域現在開放。');
        document.getElementById('knowledgeCard').classList.remove('hidden');
        document.getElementById('knowledgeCard').textContent = 'EUV 光路校準完成！你已逐面調整反射鏡，完成四組曝光路徑。前往真空與污染監控站，完成本次巡檢。';
    }
    buildStationButtons();
    updateQuestProgress();
    renderFactory();
}

function setMovementKey(direction, isPressed) {
    if (isPressed) {
        movementKeys.delete(direction === 'forward' ? 'back' : 'forward');
        movementKeys.add(direction);
    } else {
        movementKeys.delete(direction);
    }
    if (movementKeys.size) startFootsteps();
    else stopFootsteps();
}

function bindMovementButton(buttonId, direction) {
    const button = document.getElementById(buttonId);
    button.addEventListener('pointerdown', event => {
        event.preventDefault();
        setMovementKey(direction, true);
        button.setPointerCapture(event.pointerId);
    });
    button.addEventListener('pointerup', () => setMovementKey(direction, false));
    button.addEventListener('pointercancel', () => setMovementKey(direction, false));
    button.addEventListener('lostpointercapture', () => setMovementKey(direction, false));
}

bindMovementButton('moveForwardButton', 'forward');
bindMovementButton('moveBackButton', 'back');
document.getElementById('turnRightButton').addEventListener('click', turnRightAtStation);
document.getElementById('interactButton').addEventListener('click', interactWithStation);
document.getElementById('returnToFactoryButton').addEventListener('click', returnToFactory);
document.getElementById('continueToRolesButton').addEventListener('click', () => {
    document.getElementById('knowledgeIntro').classList.add('hidden');
    document.getElementById('roleSelect').classList.remove('hidden');
});
document.getElementById('beginExplorationButton').addEventListener('click', beginExploration);
document.getElementById('hintButton').addEventListener('click', () => {
    if (Date.now() < hintCooldownUntil) return;
    const panel = document.getElementById('hintPanel');
    panel.classList.remove('hidden');
    hintCooldownUntil = Date.now() + 30000;
    const button = document.getElementById('hintButton');
    button.disabled = true;
    const updateCooldown = () => {
        const remaining = Math.max(0, Math.ceil((hintCooldownUntil - Date.now()) / 1000));
        if (remaining === 0) {
            button.disabled = false;
            button.textContent = '提示（30 秒）';
            window.clearInterval(hintCooldownTimer);
            hintCooldownTimer = null;
            return;
        }
        button.textContent = `提示冷卻 ${remaining} 秒`;
    };
    updateCooldown();
    if (hintCooldownTimer !== null) window.clearInterval(hintCooldownTimer);
    hintCooldownTimer = window.setInterval(updateCooldown, 250);
    speakText(panel.querySelector('p').textContent);
});
document.getElementById('closeHintButton').addEventListener('click', () => {
    document.getElementById('hintPanel').classList.add('hidden');
});
document.getElementById('blockTray').addEventListener('click', event => {
    const block = event.target.closest('.material-block');
    if (!block) return;
    selectedBlockId = block.dataset.blockId;
    document.querySelectorAll('.material-block').forEach(item => {
        item.classList.toggle('selected', item === block);
    });
    document.getElementById('blockStatus').textContent =
        `已選擇「${block.textContent.trim()}」。請點擊相對應的晶圓堆疊區或光罩座。`;
});
document.getElementById('blockTray').addEventListener('dragstart', event => {
    const block = event.target.closest('.material-block');
    if (!block || !event.dataTransfer) return;
    selectedBlockId = block.dataset.blockId;
    event.dataTransfer.setData('text/plain', selectedBlockId);
    event.dataTransfer.effectAllowed = 'copy';
});
document.querySelectorAll('[data-drop-target]').forEach(target => {
    target.addEventListener('dragover', event => {
        event.preventDefault();
        target.classList.add('drop-ready');
    });
    target.addEventListener('dragleave', () => target.classList.remove('drop-ready'));
    target.addEventListener('drop', event => {
        event.preventDefault();
        target.classList.remove('drop-ready');
        const blockId = event.dataTransfer?.getData('text/plain') || selectedBlockId;
        if (blockId) placeBlock(blockId, target.dataset.dropTarget);
    });
    target.addEventListener('click', () => {
        if (selectedBlockId) placeBlock(selectedBlockId, target.dataset.dropTarget);
    });
});
document.querySelectorAll('[data-process-step]').forEach(button => {
    button.addEventListener('click', () => runBlockProcess(button.dataset.processStep));
});
document.getElementById('finishBlockLabButton').addEventListener('click', finishBlockLab);
document.getElementById('changeRoleButton').addEventListener('click', () => {
    window.clearTimeout(activityTimer);
    selectedJob = null;
    gameplayStarted = false;
    stopSpeech();
    movementKeys.clear();
    stopFootsteps();
    document.getElementById('gameplayTutorial').classList.add('hidden');
    document.getElementById('blockLab').classList.add('hidden');
    document.getElementById('exposureLab').classList.add('hidden');
    document.getElementById('factoryExplore').classList.add('hidden');
    document.getElementById('roleSelect').classList.remove('hidden');
});
document.getElementById('voiceToggleButton').addEventListener('click', event => {
    voiceEnabled = !voiceEnabled;
    const button = event.currentTarget;
    button.textContent = `語音朗讀：${voiceEnabled ? '開' : '關'}`;
    button.setAttribute('aria-pressed', String(voiceEnabled));
    if (!voiceEnabled) {
        stopSpeech();
    } else {
        const dialogue = document.querySelector('#knowledgeCard .dialogue-copy');
        if (dialogue) {
            speakText(`${dialogue.querySelector('strong').textContent}。${dialogue.querySelector('p').textContent}`);
        }
    }
});
document.getElementById('soundToggleButton').addEventListener('click', event => {
    soundEnabled = !soundEnabled;
    const button = event.currentTarget;
    button.textContent = `遊戲音效：${soundEnabled ? '開' : '關'}`;
    button.setAttribute('aria-pressed', String(soundEnabled));
    if (soundEnabled) {
        playGameSound('interact');
        if (movementKeys.size) startFootsteps();
    } else {
        stopFootsteps();
        if (audioContext) audioContext.suspend();
    }
});
window.addEventListener('keydown', event => {
    if (document.getElementById('factoryExplore').classList.contains('hidden') ||
        !document.getElementById('gameplayTutorial').classList.contains('hidden') ||
        !document.getElementById('blockLab').classList.contains('hidden') ||
        !gameplayStarted || event.repeat) return;
    const target = event.target;
    const isTyping = target instanceof HTMLElement && target.matches('input, textarea, select, [contenteditable="true"]');
    if (isTyping) return;

    if (event.code === 'KeyW' || event.code === 'ArrowUp') {
        event.preventDefault();
        setMovementKey('forward', true);
    } else if (event.code === 'KeyS' || event.code === 'ArrowDown') {
        event.preventDefault();
        setMovementKey('back', true);
    } else if (event.code === 'KeyE') {
        event.preventDefault();
        interactWithStation();
    } else if (event.code === 'KeyD') {
        event.preventDefault();
        turnRightAtStation();
    } else if (event.code === 'Enter') {
        event.preventDefault();
        event.stopImmediatePropagation();
        advanceDialogue();
    }
}, true);
window.addEventListener('keyup', event => {
    if (event.code === 'KeyW' || event.code === 'ArrowUp') setMovementKey('forward', false);
    if (event.code === 'KeyS' || event.code === 'ArrowDown') setMovementKey('back', false);
});
window.addEventListener('blur', () => {
    movementKeys.clear();
    stopFootsteps();
    stopSpeech();
});
buildRoleChoices();
document.getElementById('knowledgeIntro').classList.remove('hidden');
document.getElementById('roleSelect').classList.add('hidden');
function animateFactoryScene() {
    if (selectedJob && !document.getElementById('factoryExplore').classList.contains('hidden')) {
        const now = performance.now();
        const deltaSeconds = lastMovementFrame ? Math.min((now - lastMovementFrame) / 1000, 0.05) : 0;
        lastMovementFrame = now;
        const movingForward = movementKeys.has('forward');
        const movingBack = movementKeys.has('back');
        if (movingForward !== movingBack) {
            moveThroughFactory(movingForward ? 1 : -1, deltaSeconds);
        }
        renderFactory();
    } else {
        lastMovementFrame = 0;
    }
    factoryAnimationFrame = requestAnimationFrame(animateFactoryScene);
}
factoryAnimationFrame = requestAnimationFrame(animateFactoryScene);

function buildControlsUI() {
    document.getElementById('levelTitle').innerText = currentLevel.name;
    const container = document.getElementById('controlsContainer');
    container.innerHTML = '';

    currentLevel.elements.forEach((m) => {
        const div = document.createElement('div');
        div.className = 'control-group';
        div.innerHTML = `
            <label for="input_${m.id}">${m.label}: <span id="val_${m.id}">${m.currentAngle}°</span></label>
            <div class="angle-controls">
                <input type="range" id="input_${m.id}" min="${m.min}" max="${m.max}" value="${m.currentAngle}" step="1" aria-label="${m.label} angle slider">
                <input type="number" id="number_${m.id}" min="${m.min}" max="${m.max}" value="${m.currentAngle}" step="1" aria-label="${m.label} angle in degrees">
            </div>
        `;
        container.appendChild(div);

        const slider = div.querySelector(`#input_${m.id}`);
        const numberInput = div.querySelector(`#number_${m.id}`);
        const valueLabel = div.querySelector(`#val_${m.id}`);
        const updateAngle = (value) => {
            if (value === '') {
                slider.value = m.currentAngle;
                numberInput.value = m.currentAngle;
                return;
            }
            const parsedAngle = Number(value);
            if (!Number.isFinite(parsedAngle)) return;
            m.currentAngle = Math.round(Math.min(m.max, Math.max(m.min, parsedAngle)));
            slider.value = m.currentAngle;
            numberInput.value = m.currentAngle;
            valueLabel.innerText = `${m.currentAngle}°`;
        };

        slider.addEventListener('input', () => updateAngle(slider.value));
        numberInput.addEventListener('input', () => updateAngle(numberInput.value));
        numberInput.addEventListener('change', () => updateAngle(numberInput.value));
    });

    const btnDiv = document.createElement('div');
    btnDiv.className = 'btn-container';
    btnDiv.innerHTML = `
        <button class="btn-fire" onclick="fireSingleElectron()">發射電子束 (按空白鍵)</button>
        <span class="key-hint">提示：直接按 [空白鍵 Space] 發射</span>
    `;
    container.appendChild(btnDiv);
}

function loadLevel(idx) {
    currentLevel = levels[idx];
    
    gun.x = currentLevel.gun.x;
    gun.y = currentLevel.gun.y;
    gun.dirX = currentLevel.gun.dirX;
    gun.dirY = currentLevel.gun.dirY;

    tinDroplet.x = currentLevel.tin.x;
    tinDroplet.y = currentLevel.tin.y;
    tinDroplet.hit = false;
    tinDroplet.plasmaTimer = 0;
    electrons = [];
    shotInProgress = false;
    
    waferTarget.x = currentLevel.wafer.x;
    waferTarget.y = currentLevel.wafer.y;
    waferTarget.width = currentLevel.wafer.width;
    waferTarget.height = currentLevel.wafer.height;
    waferTarget.isVertical = currentLevel.wafer.isVertical;

    euvBeams = [];
    beamOpacity = 0;
    statusState = { text: "機台就緒！請微調反射鏡與光罩角度", color: "#00ff87", type: "IDLE" };

    buildControlsUI();
}

window.addEventListener('keydown', (e) => {
    if (e.code === 'Space' && isStarted) {
        e.preventDefault();
        fireSingleElectron();
    }
});

function fireSingleElectron() {
    if (!isStarted || shotInProgress || statusState.type === "SUCCESS") return;
    shotInProgress = true;
    tinDroplet.hit = false;
    playGameSound('activity');
    electrons.push({
        x: gun.x,
        y: gun.y,
        vx: gun.dirX * 15,
        vy: gun.dirY * 15,
        radius: 4.5
    });
}

function raySegmentIntersection(originX, originY, dirX, dirY, elem) {
    const angle = elem.currentAngle * Math.PI / 180;
    const segmentX = Math.cos(angle) * MIRROR_LENGTH;
    const segmentY = Math.sin(angle) * MIRROR_LENGTH;
    const startX = elem.x - segmentX / 2;
    const startY = elem.y - segmentY / 2;
    const offsetX = startX - originX;
    const offsetY = startY - originY;
    const denominator = dirX * segmentY - dirY * segmentX;

    if (Math.abs(denominator) < 1e-9) return null;

    const distance = (offsetX * segmentY - offsetY * segmentX) / denominator;
    const segmentPosition = (offsetX * dirY - offsetY * dirX) / denominator;
    if (distance <= 1e-4 || segmentPosition < 0 || segmentPosition > 1) return null;

    return {
        distance,
        x: originX + dirX * distance,
        y: originY + dirY * distance
    };
}

function rayRectangleIntersection(originX, originY, dirX, dirY, left, top, right, bottom) {
    const intersections = [];
    if (Math.abs(dirX) > 1e-9) {
        [left, right].forEach(x => {
            const distance = (x - originX) / dirX;
            const y = originY + dirY * distance;
            if (distance > 1e-4 && y >= top && y <= bottom) intersections.push({ distance, x, y });
        });
    }
    if (Math.abs(dirY) > 1e-9) {
        [top, bottom].forEach(y => {
            const distance = (y - originY) / dirY;
            const x = originX + dirX * distance;
            if (distance > 1e-4 && x >= left && x <= right) intersections.push({ distance, x, y });
        });
    }
    return intersections.sort((a, b) => a.distance - b.distance)[0] || null;
}

function calculateEUVPathOnHit() {
    euvBeams = [];
    let hitCount = 0;
    const spreadAngles = [-0.02, -0.01, 0, 0.01, 0.02];

    let leftBound = waferTarget.x - waferTarget.width / 2;
    let rightBound = waferTarget.x + waferTarget.width / 2;
    let topBound = waferTarget.y - waferTarget.height / 2;
    let bottomBound = waferTarget.y + waferTarget.height / 2;

    spreadAngles.forEach(angleOffset => {
        let path = [{ x: tinDroplet.x, y: tinDroplet.y }];
        let originX = tinDroplet.x;
        let originY = tinDroplet.y;
        let rayAngle = Math.atan2(gun.dirY, gun.dirX) + angleOffset;
        let dirX = Math.cos(rayAngle);
        let dirY = Math.sin(rayAngle);
        let rayHitWafer = false;
        let rayEnded = false;

        for (let bounce = 0; bounce < Math.max(12, currentLevel.elements.length * 3); bounce++) {
            let nearestMirror = null;
            currentLevel.elements.forEach(elem => {
                const intersection = raySegmentIntersection(originX, originY, dirX, dirY, elem);
                if (intersection && (!nearestMirror || intersection.distance < nearestMirror.distance)) {
                    nearestMirror = { ...intersection, elem };
                }
            });

            const wallHit = rayRectangleIntersection(originX, originY, dirX, dirY, 0, 0, canvas.width, canvas.height);
            const waferHit = rayRectangleIntersection(originX, originY, dirX, dirY, leftBound, topBound, rightBound, bottomBound);
            const mirrorDistance = nearestMirror ? nearestMirror.distance : Infinity;
            const wallDistance = wallHit ? wallHit.distance : Infinity;

            if (waferHit && waferHit.distance < mirrorDistance && waferHit.distance <= wallDistance) {
                path.push({ x: waferHit.x, y: waferHit.y });
                rayHitWafer = true;
                rayEnded = true;
                break;
            }

            if (nearestMirror && mirrorDistance < wallDistance) {
                path.push({ x: nearestMirror.x, y: nearestMirror.y });
                const mirrorRad = nearestMirror.elem.currentAngle * Math.PI / 180;
                const nx = Math.cos(mirrorRad + Math.PI / 2);
                const ny = Math.sin(mirrorRad + Math.PI / 2);
                const dot = dirX * nx + dirY * ny;
                dirX -= 2 * dot * nx;
                dirY -= 2 * dot * ny;
                originX = nearestMirror.x;
                originY = nearestMirror.y;
                continue;
            }

            const endPoint = wallHit || { x: originX + dirX * 1000, y: originY + dirY * 1000 };
            path.push({ x: endPoint.x, y: endPoint.y });
            rayEnded = true;
            break;
        }

        if (!rayEnded) {
            const endPoint = rayRectangleIntersection(originX, originY, dirX, dirY, 0, 0, canvas.width, canvas.height);
            if (endPoint) path.push({ x: endPoint.x, y: endPoint.y });
        }

        if (rayHitWafer) hitCount++;
        euvBeams.push(path);
    });

    beamOpacity = 1.0;

    if (hitCount === 5) {
        playGameSound('complete');
        speakText("曝光成功，五條 EUV 光線精準命中晶圓。");
        successCount++;
        statusState = { 
            text: `曝光成功！5條 EUV 光線精準命中晶圓 (累計過關: ${successCount} 次)`, 
            color: "#00ff87", 
            type: "SUCCESS" 
        };

        setTimeout(() => {
            if (selectedJob && selectedJob.id === 'optics' &&
                successCount - selectedJob.opticsSuccessStart >= levels.length) {
                statusState = {
                    text: "四組 EUV 光路校準完成！可返回工廠繼續巡檢。",
                    color: "#00ff87",
                    type: "SUCCESS"
                };
                return;
            }
            currentLevelIndex = (currentLevelIndex + 1) % levels.length;
            loadLevel(currentLevelIndex);
        }, 1800);

    } else if (hitCount > 0) {
        playGameSound('locked');
        statusState = { 
            text: `警告：光束部分偏離晶圓 (${hitCount}/5 條命中)！請調整角度`, 
            color: "#ffcc00", 
            type: "WARN" 
        };
    } else {
        playGameSound('locked');
        statusState = { 
            text: "曝光失敗：光束未命中晶圓！", 
            color: "#ff0055", 
            type: "FAIL" 
        };
    }
}

function update() {
    electrons.forEach(e => {
        e.x += e.vx;
        e.y += e.vy;

        if (!tinDroplet.hit && Math.hypot(e.x - tinDroplet.x, e.y - tinDroplet.y) < tinDroplet.radius + e.radius) {
            tinDroplet.hit = true;
            tinDroplet.plasmaTimer = 25;
            shotInProgress = false;
            e.remove = true;
            calculateEUVPathOnHit();
        }
    });

    electrons = electrons.filter(e => !e.remove && e.x < canvas.width && e.y < canvas.height);
    if (electrons.length === 0) shotInProgress = false;

    if (tinDroplet.plasmaTimer > 0) tinDroplet.plasmaTimer--;

    if (beamOpacity > 0) {
        beamOpacity -= 0.012;
        if (beamOpacity < 0) beamOpacity = 0;
    }

    draw();
    requestAnimationFrame(update);
}

function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    ctx.strokeStyle = '#1e293b';
    ctx.lineWidth = 4;
    ctx.strokeRect(10, 10, canvas.width - 20, canvas.height - 20);

    ctx.fillStyle = 'rgba(15, 23, 42, 0.6)';
    ctx.fillRect(130, 40, 760, 410);

    // 電子槍
    ctx.save();
    ctx.translate(gun.x, gun.y);
    let gunAngle = Math.atan2(gun.dirY, gun.dirX);
    ctx.rotate(gunAngle);
    ctx.fillStyle = '#1f2937';
    ctx.fillRect(-gun.width/2, -gun.height/2, gun.width, gun.height);
    ctx.strokeStyle = '#00f0ff';
    ctx.strokeRect(-gun.width/2, -gun.height/2, gun.width, gun.height);
    ctx.restore();

    // 錫滴與電漿
    if (tinDroplet.plasmaTimer > 0) {
        let grad = ctx.createRadialGradient(tinDroplet.x, tinDroplet.y, 2, tinDroplet.x, tinDroplet.y, 35);
        grad.addColorStop(0, '#ffffff');
        grad.addColorStop(0.4, '#9d4edd');
        grad.addColorStop(1, 'transparent');
        ctx.fillStyle = grad;
        ctx.beginPath();
        ctx.arc(tinDroplet.x, tinDroplet.y, 35, 0, Math.PI * 2);
        ctx.fill();
    }
    ctx.beginPath();
    ctx.arc(tinDroplet.x, tinDroplet.y, tinDroplet.radius, 0, Math.PI * 2);
    ctx.fillStyle = tinDroplet.hit ? '#9d4edd' : '#a0aec0';
    ctx.fill();

    // 電子束
    electrons.forEach(e => {
        ctx.fillStyle = '#00f0ff';
        ctx.beginPath();
        ctx.arc(e.x, e.y, e.radius, 0, Math.PI * 2);
        ctx.fill();
    });

    // 繪製反射鏡與光罩
    currentLevel.elements.forEach(elem => {
        drawElement(elem, elem.label, elem.isMask ? "#ffcc00" : "#00f0ff");
    });

    // 繪製 WAFER 晶圓 Target (支援 90 度直立繪製)
    ctx.fillStyle = statusState.type === "SUCCESS" ? '#00ff87' : '#3b82f6';
    ctx.fillRect(waferTarget.x - waferTarget.width/2, waferTarget.y - waferTarget.height/2, waferTarget.width, waferTarget.height);
    ctx.strokeStyle = '#fff';
    ctx.strokeRect(waferTarget.x - waferTarget.width/2, waferTarget.y - waferTarget.height/2, waferTarget.width, waferTarget.height);
    
    ctx.save();
    ctx.translate(waferTarget.x, waferTarget.y);
    if (waferTarget.isVertical) ctx.rotate(-Math.PI / 2);
    ctx.fillStyle = '#000';
    ctx.font = 'bold 11px sans-serif';
    ctx.textAlign = 'center';
    ctx.fillText("WAFER 晶圓", 0, 4);
    ctx.restore();

    // EUV 光路渲染
    if (beamOpacity > 0 && euvBeams.length > 0) {
        ctx.save();
        ctx.lineWidth = 2.5;

        euvBeams.forEach(path => {
            ctx.strokeStyle = `rgba(157, 78, 221, ${0.9 * beamOpacity})`;
            ctx.beginPath();
            ctx.moveTo(path[0].x, path[0].y);
            for (let i = 1; i < path.length; i++) {
                ctx.lineTo(path[i].x, path[i].y);
            }
            ctx.stroke();
        });
        ctx.restore();
    }

    // 頂部狀態列
    ctx.fillStyle = statusState.color;
    ctx.font = 'bold 15px sans-serif';
    ctx.textAlign = 'left';
    ctx.fillText(statusState.text, 25, 30);
}

// 畫圖函式：依據 flipNormal 精確翻轉鏡面與鍍膜
function drawElement(elem, label, color) {
    ctx.save();
    ctx.translate(elem.x, elem.y);
    ctx.rotate(elem.currentAngle * Math.PI / 180);
    
    let isFlipped = elem.flipNormal;

    // 深色背板
    ctx.fillStyle = '#334155';
    ctx.fillRect(-MIRROR_LENGTH / 2, isFlipped ? 0 : -8, MIRROR_LENGTH, 8);
    
    // 亮彩鍍膜反射面 (藍/黃)
    ctx.fillStyle = color;
    ctx.fillRect(-MIRROR_LENGTH / 2, isFlipped ? -4 : 0, MIRROR_LENGTH, 4);

    ctx.restore();
    
    // 標籤文字
    ctx.fillStyle = '#94a3b8';
    ctx.font = '12px sans-serif';
    ctx.textAlign = 'center';
    ctx.fillText(label, elem.x, elem.y - 18);
}

loadLevel(0);
update();
</script>
</body>
</html>
