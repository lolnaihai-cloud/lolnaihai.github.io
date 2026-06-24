<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<meta name="referrer" content="strict-origin-when-cross-origin">
<meta name="theme-color" content="#14110F">
<title>Резонанс — мой плеер</title>
<style>
  :root{
    --bg:#14110F; --surface:#1E1A17; --surface-2:#26211D; --line:#322B25;
    --text:#F2EDE6; --muted:#9A8F84; --faint:#6A6058;
    --spotify:#1DB954; --soundcloud:#FF5500; --youtube:#FF0033; --vk:#5AA9F0;
    --radius:14px; --gap:14px;
    --ff: ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, sans-serif;
    --mono: ui-monospace, "SF Mono", "Cascadia Code", Menlo, Consolas, monospace;
  }
  [data-theme="light"]{
    --bg:#F4F1EC; --surface:#FFFFFF; --surface-2:#F0EBE3; --line:#E2DAD0;
    --text:#1A1612; --muted:#6E655C; --faint:#A89E92;
  }
  *{box-sizing:border-box; margin:0; padding:0}
  html,body{height:100%}
  body{
    font-family:var(--ff); background:var(--bg); color:var(--text);
    -webkit-font-smoothing:antialiased; overflow:hidden;
    display:flex; flex-direction:column;
  }
  button{font-family:inherit; cursor:pointer; border:none; background:none; color:inherit}
  input{font-family:inherit}
  ::selection{background:var(--text); color:var(--bg)}

  /* ---------- Layout ---------- */
  .app{flex:1; display:grid; grid-template-columns:248px 1fr; min-height:0}
  @media(max-width:780px){ .app{grid-template-columns:1fr} }

  /* ---------- Sidebar ---------- */
  .side{
    border-right:1px solid var(--line); background:var(--surface);
    display:flex; flex-direction:column; min-height:0;
  }
  .brand{
    padding:20px 20px 16px; display:flex; align-items:baseline; gap:10px;
    border-bottom:1px solid var(--line);
  }
  .brand h1{font-size:19px; font-weight:650; letter-spacing:-.02em}
  .brand .eq{display:flex; align-items:flex-end; gap:2px; height:16px}
  .brand .eq i{width:3px; background:var(--text); animation:eq 1.1s ease-in-out infinite}
  .brand .eq i:nth-child(2){animation-delay:.2s} .brand .eq i:nth-child(3){animation-delay:.4s}
  @keyframes eq{0%,100%{height:4px}50%{height:16px}}
  body:not(.playing) .brand .eq i{animation-play-state:paused; height:5px}

  .nav{padding:12px; overflow-y:auto; flex:1; min-height:0}
  .nav-label{font-size:11px; letter-spacing:.08em; text-transform:uppercase; color:var(--faint); padding:14px 10px 6px}
  .nav-item{
    width:100%; text-align:left; padding:9px 10px; border-radius:9px; font-size:14px;
    color:var(--muted); display:flex; align-items:center; gap:10px; transition:.12s;
  }
  .nav-item:hover{background:var(--surface-2); color:var(--text)}
  .nav-item.active{background:var(--surface-2); color:var(--text)}
  .nav-item .dot{width:8px; height:8px; border-radius:50%; flex:none}
  .nav-item .cnt{margin-left:auto; font-family:var(--mono); font-size:11px; color:var(--faint)}
  .nav-item.pl{position:relative}
  .nav-item .del{margin-left:auto; opacity:0; font-size:15px; color:var(--faint); padding:0 2px}
  .nav-item.pl:hover .del{opacity:1}
  .nav-item.pl:hover .cnt{display:none}
  .add-pl{margin-top:4px; color:var(--faint); font-size:13px}
  .add-pl:hover{color:var(--text)}

  .side-foot{padding:12px; border-top:1px solid var(--line); display:flex; gap:8px}
  .side-foot button{
    flex:1; padding:8px; font-size:12px; border-radius:8px; background:var(--surface-2);
    color:var(--muted); border:1px solid var(--line); transition:.12s;
  }
  .side-foot button:hover{color:var(--text); border-color:var(--muted)}

  /* ---------- Main ---------- */
  .main{display:flex; flex-direction:column; min-height:0; min-width:0}
  .topbar{
    padding:16px 22px; display:flex; gap:12px; align-items:center;
    border-bottom:1px solid var(--line);
  }
  .menu-btn{display:none; font-size:22px; line-height:1; padding:4px}
  @media(max-width:780px){ .menu-btn{display:block} }

  .add{flex:1; display:flex; gap:8px; position:relative}
  .add input{
    flex:1; background:var(--surface); border:1px solid var(--line); color:var(--text);
    padding:11px 14px; border-radius:10px; font-size:14px; outline:none; transition:.12s;
  }
  .add input:focus{border-color:var(--muted)}
  .add input::placeholder{color:var(--faint)}
  .add button{
    padding:11px 18px; background:var(--text); color:var(--bg); border-radius:10px;
    font-size:14px; font-weight:600; transition:.12s; white-space:nowrap;
  }
  .add button:hover{opacity:.85}
  .add button:disabled{opacity:.4; cursor:default}

  .search{
    background:var(--surface); border:1px solid var(--line); color:var(--text);
    padding:11px 14px; border-radius:10px; font-size:14px; outline:none; width:200px;
  }
  .search:focus{border-color:var(--muted)}
  @media(max-width:780px){ .search{display:none} }

  .list{flex:1; overflow-y:auto; min-height:0; padding:8px 14px 120px}
  .view-title{padding:18px 8px 10px; display:flex; align-items:baseline; gap:10px}
  .view-title h2{font-size:22px; font-weight:650; letter-spacing:-.02em}
  .view-title span{font-family:var(--mono); font-size:12px; color:var(--faint)}

  .track{
    display:flex; align-items:center; gap:14px; padding:11px 14px 11px 12px;
    border-radius:11px; cursor:pointer; position:relative; transition:.1s;
  }
  .track:hover{background:var(--surface)}
  .track.current{background:var(--surface)}
  .track .spine{
    position:absolute; left:0; top:9px; bottom:9px; width:3px; border-radius:3px;
    background:var(--src); opacity:0; transition:.15s;
  }
  .track:hover .spine, .track.current .spine{opacity:1}
  .track .ico{
    width:38px; height:38px; border-radius:9px; flex:none; display:grid; place-items:center;
    background:var(--surface-2); color:var(--src); font-weight:700; font-size:13px;
    border:1px solid var(--line);
  }
  .track .meta{min-width:0; flex:1}
  .track .nm{font-size:14.5px; font-weight:500; white-space:nowrap; overflow:hidden; text-overflow:ellipsis}
  .track.current .nm{color:var(--src)}
  .track .sub{font-size:12px; color:var(--muted); margin-top:2px; display:flex; align-items:center; gap:8px}
  .track .sub .src-tag{color:var(--src); font-weight:600}
  .track .acts{display:flex; gap:2px; opacity:0; transition:.12s}
  .track:hover .acts{opacity:1}
  .track .acts button{padding:7px; border-radius:7px; color:var(--muted); font-size:15px; line-height:1}
  .track .acts button:hover{background:var(--surface-2); color:var(--text)}
  .track .playing-eq{display:none; gap:2px; align-items:flex-end; height:14px; width:18px}
  .track.current.is-playing .playing-eq{display:flex}
  .track.current.is-playing .ico{display:none}
  .playing-eq i{width:3px; background:var(--src); animation:eq 1s ease-in-out infinite}
  .playing-eq i:nth-child(2){animation-delay:.25s} .playing-eq i:nth-child(3){animation-delay:.5s}

  .empty{padding:60px 24px; text-align:center; color:var(--muted); max-width:480px; margin:0 auto}
  .empty h3{font-size:18px; color:var(--text); margin-bottom:10px; font-weight:600}
  .empty p{font-size:14px; line-height:1.6; margin-bottom:8px}
  .empty code{font-family:var(--mono); font-size:12px; background:var(--surface); padding:2px 6px; border-radius:5px; color:var(--text)}

  /* ---------- Player dock ---------- */
  .dock{
    position:fixed; left:248px; right:0; bottom:0; background:var(--surface);
    border-top:1px solid var(--line); transition:transform .28s cubic-bezier(.4,0,.2,1);
  }
  @media(max-width:780px){ .dock{left:0} }
  .dock .progress{height:2px; background:var(--src); width:var(--pct,0%); transition:width .25s linear}
  .dock-bar{display:flex; align-items:center; gap:14px; padding:12px 22px}
  .dock-now{flex:1; min-width:0; display:flex; align-items:center; gap:12px}
  .dock-now .badge{width:42px; height:42px; border-radius:9px; flex:none; display:grid; place-items:center; background:var(--surface-2); color:var(--src); border:1px solid var(--line); font-size:12px; font-weight:700}
  .dock-now .t{min-width:0}
  .dock-now .t .nm{font-size:14px; font-weight:500; white-space:nowrap; overflow:hidden; text-overflow:ellipsis}
  .dock-now .t .s{font-size:11.5px; color:var(--muted)}
  .dock-now .t .s b{color:var(--src)}
  .controls{display:flex; align-items:center; gap:4px}
  .controls button{padding:9px; border-radius:9px; color:var(--muted); font-size:18px; line-height:1; transition:.12s}
  .controls button:hover{color:var(--text); background:var(--surface-2)}
  .controls button.on{color:var(--src)}
  .controls .play{
    width:44px; height:44px; border-radius:50%; background:var(--text); color:var(--bg);
    display:grid; place-items:center; font-size:18px;
  }
  .controls .play:hover{background:var(--text); opacity:.85}
  .expand{padding:9px; border-radius:9px; color:var(--muted); font-size:16px}
  .expand:hover{color:var(--text); background:var(--surface-2)}
  .stage{height:0; overflow:hidden; transition:height .28s cubic-bezier(.4,0,.2,1); background:var(--bg)}
  .dock.open .stage{height:200px}
  .stage-inner{padding:14px 22px 18px; height:100%}
  .stage-inner iframe, .stage-inner > div{width:100%; height:100%; border:none; border-radius:12px}
  .dock-empty{padding:14px 22px; color:var(--faint); font-size:13px; text-align:center}

  /* ---------- Modal / toast ---------- */
  .scrim{position:fixed; inset:0; background:rgba(0,0,0,.5); display:none; z-index:40; backdrop-filter:blur(2px)}
  .scrim.show{display:block}
  .sheet{
    position:fixed; z-index:50; background:var(--surface); border:1px solid var(--line);
    border-radius:16px; padding:8px; min-width:200px; box-shadow:0 20px 60px rgba(0,0,0,.4);
  }
  .sheet button{display:block; width:100%; text-align:left; padding:10px 14px; border-radius:9px; font-size:14px; color:var(--text)}
  .sheet button:hover{background:var(--surface-2)}
  .sheet .hd{font-size:11px; text-transform:uppercase; letter-spacing:.06em; color:var(--faint); padding:8px 14px 4px}
  .toast{
    position:fixed; bottom:auto; top:18px; left:50%; transform:translateX(-50%) translateY(-80px);
    background:var(--text); color:var(--bg); padding:11px 20px; border-radius:11px; font-size:13.5px;
    font-weight:500; z-index:60; opacity:0; transition:.3s; pointer-events:none;
  }
  .toast.show{transform:translateX(-50%) translateY(0); opacity:1}

  /* mobile sidebar slide-over */
  @media(max-width:780px){
    .side{position:fixed; inset:0 auto 0 0; width:84%; max-width:300px; z-index:45; transform:translateX(-105%); transition:transform .26s}
    .side.open{transform:none; box-shadow:0 0 80px rgba(0,0,0,.5)}
  }
  .list::-webkit-scrollbar, .nav::-webkit-scrollbar{width:8px}
  .list::-webkit-scrollbar-thumb, .nav::-webkit-scrollbar-thumb{background:var(--line); border-radius:8px}

  /* ---------- Spotify connect ---------- */
  .side-foot-wrap{border-top:1px solid var(--line)}
  .side-foot-wrap .side-foot{border-top:none}
  .sp-connect{
    width:calc(100% - 24px); margin:12px 12px 0; padding:9px 12px; border-radius:9px;
    background:var(--surface-2); border:1px solid var(--line); color:var(--muted);
    display:flex; align-items:center; gap:10px; font-size:13px; transition:.12s;
  }
  .sp-connect:hover{color:var(--text); border-color:var(--muted)}
  .sp-connect .sp-dot{width:8px; height:8px; border-radius:50%; background:var(--faint); flex:none}
  .sp-connect .sp-label{white-space:nowrap; overflow:hidden; text-overflow:ellipsis}

  .modal{position:fixed; inset:0; z-index:55; display:none; align-items:center; justify-content:center; padding:20px}
  .modal.show{display:flex}
  .modal-card{
    background:var(--surface); border:1px solid var(--line); border-radius:18px;
    width:100%; max-width:460px; max-height:88vh; overflow-y:auto; padding:24px;
    box-shadow:0 30px 80px rgba(0,0,0,.5);
  }
  .modal-hd{display:flex; align-items:center; justify-content:space-between; margin-bottom:14px}
  .modal-hd h3{font-size:18px; font-weight:650; letter-spacing:-.01em}
  .modal-hd button{font-size:16px; color:var(--muted); padding:6px; border-radius:8px}
  .modal-hd button:hover{color:var(--text); background:var(--surface-2)}
  .sp-desc{font-size:13.5px; color:var(--muted); line-height:1.55; margin-bottom:14px}
  .sp-steps{list-style:none; counter-reset:s; margin-bottom:16px; display:flex; flex-direction:column; gap:8px}
  .sp-steps li{counter-increment:s; position:relative; padding-left:30px; font-size:13.5px; line-height:1.5; color:var(--text)}
  .sp-steps li::before{content:counter(s); position:absolute; left:0; top:-1px; width:20px; height:20px; border-radius:50%; background:var(--spotify); color:#06150c; font-size:11px; font-weight:700; display:grid; place-items:center}
  .sp-steps b{font-family:var(--mono); font-size:12.5px; background:var(--surface-2); padding:1px 5px; border-radius:5px}
  .sp-field{display:block; margin-bottom:14px}
  .sp-field > span{display:block; font-size:11px; text-transform:uppercase; letter-spacing:.06em; color:var(--faint); margin-bottom:6px}
  .sp-field input{width:100%; background:var(--bg); border:1px solid var(--line); color:var(--text); padding:10px 12px; border-radius:9px; font-size:13.5px; outline:none; font-family:var(--mono)}
  .sp-field input:focus{border-color:var(--muted)}
  .sp-redirect{display:flex; gap:8px; align-items:stretch}
  .sp-redirect code{flex:1; background:var(--bg); border:1px solid var(--line); border-radius:9px; padding:10px 12px; font-family:var(--mono); font-size:12px; color:var(--text); overflow-x:auto; white-space:nowrap}
  .sp-redirect button{background:var(--surface-2); border:1px solid var(--line); border-radius:9px; padding:0 12px; font-size:12px; color:var(--muted); white-space:nowrap}
  .sp-redirect button:hover{color:var(--text)}
  .sp-warn{display:none; background:rgba(255,85,0,.1); border:1px solid rgba(255,85,0,.3); color:var(--soundcloud); font-size:12.5px; line-height:1.5; padding:10px 12px; border-radius:9px; margin-bottom:14px}
  .sp-actions{display:flex; gap:8px; flex-wrap:wrap}
  .sp-primary{background:var(--spotify); color:#06150c; font-weight:600; padding:11px 18px; border-radius:10px; font-size:14px}
  .sp-primary:hover{opacity:.9}
  .sp-ghost{background:var(--surface-2); border:1px solid var(--line); color:var(--text); padding:11px 16px; border-radius:10px; font-size:14px}
  .sp-ghost:hover{border-color:var(--muted)}
  .sp-lists{margin-top:16px; display:flex; flex-direction:column; gap:2px; max-height:240px; overflow-y:auto}
  .sp-hint{color:var(--faint); font-size:13px; padding:10px 4px}
  .sp-row{display:flex; align-items:center; gap:10px; padding:9px 10px; border-radius:9px; font-size:13.5px}
  .sp-row:hover{background:var(--surface-2)}
  .sp-row > span:first-child{flex:1; white-space:nowrap; overflow:hidden; text-overflow:ellipsis}
  .sp-row .sp-cnt{font-family:var(--mono); font-size:11px; color:var(--faint)}
  .sp-row button{width:26px; height:26px; border-radius:7px; background:var(--spotify); color:#06150c; font-size:15px; font-weight:700; flex:none}
</style>
</head>
<body class="not-playing">

<div class="app">
  <!-- Sidebar -->
  <aside class="side" id="side">
    <div class="brand">
      <div class="eq"><i></i><i></i><i></i></div>
      <h1>Резонанс</h1>
    </div>
    <nav class="nav" id="nav"></nav>
    <div class="side-foot-wrap">
      <button class="sp-connect" id="spConnectBtn"><span class="sp-dot"></span><span class="sp-label">Подключить Spotify</span></button>
      <button class="sp-connect" id="syncBtn"><span class="sp-dot" style="background:var(--vk)"></span><span class="sync-label">Синхронизация</span></button>
      <div class="side-foot">
        <button id="btnExport" title="Сохранить библиотеку в файл">Экспорт</button>
        <button id="btnImport" title="Загрузить библиотеку из файла">Импорт</button>
        <button id="btnTheme" title="Сменить тему" style="flex:none; width:42px">◐</button>
      </div>
    </div>
  </aside>

  <!-- Main -->
  <main class="main">
    <div class="topbar">
      <button class="menu-btn" id="menuBtn">☰</button>
      <div class="add">
        <input id="addInput" placeholder="Вставь ссылку: Spotify · SoundCloud · YouTube Music" autocomplete="off">
        <button id="addBtn">Добавить</button>
      </div>
      <input class="search" id="searchInput" placeholder="Поиск…" autocomplete="off">
    </div>
    <div class="list" id="list"></div>
  </main>
</div>

<!-- Player dock -->
<div class="dock" id="dock">
  <div class="progress" id="progress"></div>
  <div class="stage"><div class="stage-inner"><div id="stage"></div></div></div>
  <div class="dock-bar" id="dockBar">
    <div class="dock-now" id="dockNow">
      <div class="dock-empty" style="padding:0">Выбери трек, чтобы начать</div>
    </div>
    <div class="controls">
      <button id="cShuffle" title="Перемешать">⤮</button>
      <button id="cPrev" title="Предыдущий">⏮</button>
      <button class="play" id="cPlay" title="Играть / пауза">▶</button>
      <button id="cNext" title="Следующий">⏭</button>
      <button id="cRepeat" title="Повтор">⟲</button>
      <button class="expand" id="cExpand" title="Показать плеер площадки">⌄</button>
    </div>
  </div>
</div>

<div class="modal" id="spModal">
  <div class="modal-card">
    <div class="modal-hd"><h3>Подключение Spotify</h3><button id="spClose" title="Закрыть">✕</button></div>
    <p class="sp-desc">Разворачивает плейлисты и альбомы Spotify в отдельные треки в твоей библиотеке. Нужно один раз бесплатно зарегистрировать своё приложение Spotify — пароль и секретный ключ не требуются.</p>
    <ol class="sp-steps">
      <li>Открой <b>developer.spotify.com/dashboard</b> → Create app.</li>
      <li>В поле <b>Redirect URI</b> вставь адрес ниже и сохрани приложение.</li>
      <li>Скопируй <b>Client ID</b> из настроек приложения и вставь его сюда.</li>
    </ol>
    <label class="sp-field"><span>Redirect URI — зарегистрируй ровно это значение</span>
      <div class="sp-redirect"><code id="spRedirect"></code><button id="spCopyRedirect">копировать</button></div>
    </label>
    <div class="sp-warn" id="spFileWarn">⚠ Файл открыт локально (file://). Spotify не примет такой Redirect URI. Выложи файл на хостинг по HTTPS (например GitHub Pages) или запусти локальный сервер на http://127.0.0.1 и открой плеер оттуда.</div>
    <label class="sp-field"><span>Client ID</span>
      <input id="spClientId" placeholder="вставь Client ID" autocomplete="off" spellcheck="false"></label>
    <div class="sp-actions">
      <button class="sp-primary" id="spLoginBtn">Подключить</button>
      <button class="sp-ghost" id="spMyBtn" style="display:none">Мои плейлисты</button>
      <button class="sp-ghost" id="spLogoutBtn" style="display:none">Отключить</button>
    </div>
    <div class="sp-lists" id="spLists"></div>
  </div>
</div>

<div class="modal" id="syncModal">
  <div class="modal-card">
    <div class="modal-hd"><h3>Общая библиотека на устройствах</h3><button id="syncClose" title="Закрыть">✕</button></div>
    <p class="sp-desc">Библиотека хранится в твоём приватном GitHub Gist — бесплатно, данные видишь только ты. Введи один и тот же токен и Gist ID на каждом устройстве, чтобы библиотека была общей.</p>
    <ol class="sp-steps">
      <li>Открой <b>github.com/settings/tokens</b> → Generate new token (classic).</li>
      <li>Отметь право <b>gist</b>, создай и скопируй токен.</li>
      <li>Вставь токен ниже и нажми <b>Выгрузить</b> — создастся Gist, его ID появится в поле.</li>
      <li>На другом устройстве вставь тот же токен и <b>Gist ID</b>, нажми <b>Загрузить</b>.</li>
    </ol>
    <label class="sp-field"><span>GitHub-токен (право gist)</span>
      <input id="syncToken" type="password" placeholder="ghp_…" autocomplete="off" spellcheck="false"></label>
    <label class="sp-field"><span>Gist ID — заполнится сам после первой выгрузки</span>
      <div class="sp-redirect"><input id="syncGist" placeholder="вставь на другом устройстве" autocomplete="off" spellcheck="false" style="flex:1; background:var(--bg); border:1px solid var(--line); color:var(--text); border-radius:9px; padding:10px 12px; font-family:var(--mono); font-size:12px; outline:none"><button id="syncCopyGist">копировать</button></div></label>
    <label style="display:flex; align-items:center; gap:10px; font-size:13.5px; color:var(--text); margin-bottom:16px; cursor:pointer">
      <input type="checkbox" id="syncAuto" style="width:auto; accent-color:#5AA9F0"> Автосинхронизация — тянуть при запуске, выгружать при изменениях</label>
    <div class="sp-actions">
      <button class="sp-primary" id="syncPushBtn" style="background:var(--vk); color:#06121f">Выгрузить</button>
      <button class="sp-ghost" id="syncPullBtn">Загрузить</button>
    </div>
    <div class="sp-hint" id="syncStatus" style="margin-top:14px">Ещё не синхронизировано</div>
  </div>
</div>

<div class="scrim" id="scrim"></div>
<div class="toast" id="toast"></div>
<input type="file" id="fileInput" accept="application/json" hidden>

<!-- platform SDKs -->
<script src="https://www.youtube.com/iframe_api"></script>
<script src="https://w.soundcloud.com/player/api.js"></script>
<script src="https://open.spotify.com/embed/iframe-api/v1" async></script>

<script>
"use strict";
//============================ Состояние ============================
const SRC = {
  spotify:   { name:'Spotify',     color:'var(--spotify)',    tag:'SP' },
  soundcloud:{ name:'SoundCloud',  color:'var(--soundcloud)', tag:'SC' },
  youtube:   { name:'YouTube',     color:'var(--youtube)',    tag:'YT' },
  vk:        { name:'VK Музыка',   color:'var(--vk)',         tag:'VK' },
};
const KEY = 'rezonans.v1';
let state = load() || { tracks:[], playlists:[], settings:{ theme:'dark', shuffle:false, repeat:'off' } };
let view = { type:'all' };          // {type:'all'|'spotify'|'soundcloud'|'youtube'|'playlist', id?}
let queue = [];                     // упорядоченный список id для перехода
let currentId = null;
let isPlaying = false;

function load(){ try{ return JSON.parse(localStorage.getItem(KEY)); }catch(e){ return null; } }
let _remoteTs=null, _pushTimer=null;
function save(){
  try{
    state.meta = state.meta || {};
    state.meta.updatedAt = (_remoteTs!=null) ? _remoteTs : Date.now();
    _remoteTs = null;
    localStorage.setItem(KEY, JSON.stringify(state));
  }catch(e){}
  scheduleAutoPush();
}
function persistQuiet(){ try{ localStorage.setItem(KEY, JSON.stringify(state)); }catch(e){} }
function scheduleAutoPush(){
  const c = state.sync;
  if(!(c && c.auto && c.token && c.gistId)) return;
  clearTimeout(_pushTimer);
  _pushTimer = setTimeout(()=>syncPush(false), 1500);
}
const uid = () => Math.random().toString(36).slice(2,10);
const $ = s => document.querySelector(s);

//============================ Разбор ссылок ============================
function parseUrl(raw){
  const url = raw.trim();
  if(!url) return null;
  let m;
  // Spotify: playlist / album / track
  m = url.match(/(?:open\.spotify\.com\/|spotify:)(?:embed\/)?(playlist|album|track)[:\/]([A-Za-z0-9]{22})/);
  if(m) return { source:'spotify', kind:m[1], ref:m[2], url };
  // YouTube playlist (ссылка на плейлист без конкретного видео)
  m = url.match(/[?&]list=([A-Za-z0-9_-]+)/);
  if(m && /youtu/.test(url) && !/[?&]v=/.test(url)) return { source:'youtube', kind:'playlist', ref:m[1], url };
  // YouTube / YT Music — видео
  m = url.match(/(?:youtu\.be\/|youtube\.com\/(?:watch\?v=|embed\/|shorts\/)|music\.youtube\.com\/watch\?v=)([A-Za-z0-9_-]{11})/);
  if(m) return { source:'youtube', kind:'track', ref:m[1], url };
  // SoundCloud — сет (плейлист)
  if(/soundcloud\.com\/[^\/]+\/sets\//.test(url)) return { source:'soundcloud', kind:'playlist', ref:url.split('?')[0], url };
  // SoundCloud — трек
  if(/soundcloud\.com\/[^\/]+\/[^\/]+/.test(url) || /on\.soundcloud\.com\//.test(url))
    return { source:'soundcloud', kind:'track', ref:url.split('?')[0], url };
  // VK Музыка
  if(/(?:^|\.|\/)vk\.com\//.test(url) || /vk\.ru\//.test(url)){
    if(/\/music\/(?:playlist|album)\//.test(url)) return { source:'vk', kind:'playlist', ref:url.split('?')[0], url };
    if(/audio-?\d+_\d+/.test(url) || /\/audios?-?\d/.test(url) || /\/music/.test(url) || /\/audio/.test(url))
      return { source:'vk', kind:'track', ref:url.split('?')[0], url };
  }
  return null;
}

function addTrack(raw){
  const p = parseUrl(raw);
  if(!p){ toast('Не распознал ссылку. Поддержка: Spotify, SoundCloud, YouTube Music'); return; }
  if(p.kind==='playlist' || p.kind==='album'){
    if(p.source==='spotify'){ importSpotify(p.kind, p.ref); return; }
    if(p.source==='vk'){
      if(!state.tracks.some(t => t.source==='vk' && t.ref===p.ref)){
        state.tracks.unshift({ id:uid(), source:'vk', ref:p.ref, url:p.url, name:'Плейлист VK', added:Date.now() });
        save(); render();
      }
      toast('VK закрыл аудио-API — разбить плейлист на треки нельзя. Добавил как ссылку, откроется в VK.'); return;
    }
    toast('Разбивка плейлистов на треки доступна для Spotify. Для YouTube/SoundCloud добавь ссылку на отдельный трек.'); return;
  }
  if(state.tracks.some(t => t.source===p.source && t.ref===p.ref)){ toast('Этот трек уже в библиотеке'); return; }
  const t = { id:uid(), source:p.source, ref:p.ref, url:p.url, name:'', added:Date.now() };
  state.tracks.unshift(t); save(); render();
  toast(`Добавлено в ${SRC[p.source].name}`);
}

//============================ Отрисовка ============================
function nav(){
  const counts = { all:state.tracks.length };
  for(const k in SRC) counts[k] = state.tracks.filter(t=>t.source===k).length;
  const item = (active, dot, label, count, onclick, extra='') =>
    `<button class="nav-item ${active?'active':''}" ${onclick}>${dot?`<span class="dot" style="background:${dot}"></span>`:''}<span>${label}</span>${extra||`<span class="cnt">${count}</span>`}</button>`;

  let h = '';
  h += `<div class="nav-label">Библиотека</div>`;
  h += item(view.type==='all', '', 'Все треки', counts.all, `data-v="all"`);
  for(const k in SRC) h += item(view.type===k, SRC[k].color, SRC[k].name, counts[k], `data-v="${k}"`);

  h += `<div class="nav-label">Плейлисты</div>`;
  for(const pl of state.playlists){
    h += `<button class="nav-item pl ${view.type==='playlist'&&view.id===pl.id?'active':''}" data-pl="${pl.id}">
            <span>${esc(pl.name)}</span>
            <span class="cnt">${pl.trackIds.length}</span>
            <span class="del" data-delpl="${pl.id}" title="Удалить плейлист">×</span>
          </button>`;
  }
  h += `<button class="nav-item add-pl" id="addPl">+ Новый плейлист</button>`;
  $('#nav').innerHTML = h;
}

function currentTracks(){
  let arr;
  if(view.type==='all') arr = state.tracks;
  else if(view.type==='playlist'){
    const pl = state.playlists.find(p=>p.id===view.id);
    arr = pl ? pl.trackIds.map(id=>state.tracks.find(t=>t.id===id)).filter(Boolean) : [];
  } else arr = state.tracks.filter(t=>t.source===view.type);

  const q = $('#searchInput').value.trim().toLowerCase();
  if(q) arr = arr.filter(t => (t.name||t.url).toLowerCase().includes(q));
  return arr;
}

function list(){
  const arr = currentTracks();
  queue = arr.map(t=>t.id);
  const title = view.type==='all' ? 'Все треки'
    : view.type==='playlist' ? (state.playlists.find(p=>p.id===view.id)?.name||'Плейлист')
    : SRC[view.type].name;

  let h = `<div class="view-title"><h2>${esc(title)}</h2><span>${arr.length} ${plural(arr.length)}</span></div>`;

  if(arr.length===0){
    h += emptyState();
  } else {
    for(const t of arr){
      const s = SRC[t.source];
      const cur = t.id===currentId;
      h += `<div class="track ${cur?'current':''} ${cur&&isPlaying?'is-playing':''}" style="--src:${s.color}" data-id="${t.id}">
        <span class="spine"></span>
        <div class="playing-eq"><i></i><i></i><i></i></div>
        <div class="ico">${s.tag}</div>
        <div class="meta">
          <div class="nm">${esc(t.name || prettyUrl(t))}</div>
          <div class="sub"><span class="src-tag">${s.name}</span> · <span>${esc(shortRef(t))}</span></div>
        </div>
        <div class="acts">
          <button data-rename="${t.id}" title="Переименовать">✎</button>
          <button data-addto="${t.id}" title="В плейлист">+</button>
          <button data-remove="${t.id}" title="Удалить">×</button>
        </div>
      </div>`;
    }
  }
  $('#list').innerHTML = h;
}

function emptyState(){
  return `<div class="empty">
    <h3>Здесь пока пусто</h3>
    <p>Вставь ссылку на трек в поле сверху — я определю площадку автоматически.</p>
    <p style="margin-top:14px; color:var(--faint)">Примеры ссылок:</p>
    <p><code>open.spotify.com/track/…</code></p>
    <p><code>soundcloud.com/artist/track</code></p>
    <p><code>music.youtube.com/watch?v=…</code></p>
    <p style="margin-top:16px; font-size:13px; color:var(--faint)">Для полного воспроизведения Spotify войди в свой аккаунт Spotify в этом же браузере (Premium — целиком, иначе превью 30 сек). YouTube и SoundCloud играют целиком.</p>
  </div>`;
}

function render(){ nav(); list(); renderDock(); }

//============================ Воспроизведение ============================
let ytPlayer=null, scWidget=null, spController=null, spReady=null, spApi=null, watchdog=null;

window.onSpotifyIframeApiReady = (api)=>{ spApi = api; if(spReady) spReady(); };

function clearPlayers(){
  if(watchdog){ clearInterval(watchdog); watchdog=null; }
  try{ ytPlayer && ytPlayer.destroy(); }catch(e){}
  try{ scWidget = null; }catch(e){}
  try{ spController && spController.destroy(); }catch(e){}
  ytPlayer=null; scWidget=null; spController=null;
  $('#stage').innerHTML='';
}

function playTrack(id){
  const t = state.tracks.find(x=>x.id===id);
  if(!t) return;
  // если трека нет в текущей очереди (другой раздел) — оставляем очередь как есть, но переключаемся
  if(!queue.includes(id)){ queue = currentTracks().map(x=>x.id); }
  currentId = id;
  state.settings.lastTrackId = id; save();
  clearPlayers();
  const stage = $('#stage');

  if(t.source==='vk'){
    window.open(t.url, '_blank', 'noopener');
    $('#stage').innerHTML = `<div style="display:grid;place-items:center;height:100%;text-align:center;color:var(--muted);font-size:13px;padding:20px;line-height:1.6">Трек открыт в VK в новой вкладке.<br>Встроенное воспроизведение VK&nbsp;Музыки недоступно — у VK нет официального плеера для встраивания.</div>`;
    $('#progress').style.setProperty('--pct','0%');
    setPlaying(false); list(); renderDock();
    return;
  }

  if(t.source==='youtube'){
    const host = document.createElement('div'); stage.appendChild(host);
    const make = ()=> { ytPlayer = new YT.Player(host, {
      videoId:t.ref, host:'https://www.youtube-nocookie.com',
      playerVars:{autoplay:1, rel:0, modestbranding:1, playsinline:1, origin:location.origin},
      events:{
        onReady:e=>{ e.target.playVideo(); setPlaying(true); },
        onError:onYtError,
        onStateChange:e=>{
          if(e.data===YT.PlayerState.ENDED) onEnded();
          else if(e.data===YT.PlayerState.PLAYING) setPlaying(true);
          else if(e.data===YT.PlayerState.PAUSED) setPlaying(false);
        }
      }
    }); startProgress(()=> ytPlayer && ytPlayer.getCurrentTime? {pos:ytPlayer.getCurrentTime(), dur:ytPlayer.getDuration()} : null); };
    (window.YT && YT.Player) ? make() : (window.onYouTubeIframeAPIReady = make);
  }

  else if(t.source==='soundcloud'){
    const ifr = document.createElement('iframe');
    ifr.allow='autoplay'; ifr.scrolling='no';
    ifr.src = `https://w.soundcloud.com/player/?url=${encodeURIComponent(t.ref)}&auto_play=true&hide_related=true&show_comments=false&visual=false&color=ff5500`;
    stage.appendChild(ifr);
    const bind = ()=>{ scWidget = SC.Widget(ifr);
      scWidget.bind(SC.Widget.Events.READY, ()=>{ setPlaying(true); });
      scWidget.bind(SC.Widget.Events.FINISH, onEnded);
      scWidget.bind(SC.Widget.Events.PLAY, ()=>setPlaying(true));
      scWidget.bind(SC.Widget.Events.PAUSE, ()=>setPlaying(false));
      scWidget.bind(SC.Widget.Events.PLAY_PROGRESS, e=>{
        updateProgress(e.currentPosition, scProgDur||0);
      });
      scWidget.getDuration(d=>{ scProgDur=d; });
    };
    window.SC && SC.Widget ? bind() : ifr.addEventListener('load', bind);
  }

  else if(t.source==='spotify'){
    const host = document.createElement('div'); stage.appendChild(host);
    const make = ()=>{
      spApi.createController(host, { uri:`spotify:track:${t.ref}`, width:'100%', height:'100%' }, ctrl=>{
        spController = ctrl;
        ctrl.addListener('ready', ()=>{ ctrl.play(); setPlaying(true); });
        let lastPos=0, ended=false;
        ctrl.addListener('playback_update', e=>{
          const d=e.data; setPlaying(!d.isPaused);
          updateProgress(d.position, d.duration);
          // эвристика конца: позиция почти у конца и встала на паузу
          if(d.duration>0 && d.position>0){
            if(d.position >= d.duration-1200 && !ended){ ended=true; setTimeout(()=>{ if(currentId===id) onEnded(); }, 1200); }
            if(d.position < lastPos-2000) ended=false;
            lastPos=d.position;
          }
        });
      });
    };
    spApi ? make() : (spReady = make);
  }

  document.body.classList.remove('not-playing'); document.body.classList.add('playing');
  list(); renderDock();
}

let scProgDur=0;
function startProgress(getter){
  if(watchdog) clearInterval(watchdog);
  watchdog = setInterval(()=>{ const v=getter && getter(); if(v && v.dur) updateProgress(v.pos*1000, v.dur*1000); }, 500);
}
function updateProgress(pos, dur){
  const pct = dur>0 ? Math.min(100, (pos/dur)*100) : 0;
  $('#progress').style.setProperty('--pct', pct+'%');
}

function onYtError(e){
  const c=e.data, t=state.tracks.find(x=>x.id===currentId);
  let msg;
  if(c===101||c===150) msg='Владелец видео запретил его встраивание на других сайтах.';
  else if(c===153) msg='Ошибка 153: YouTube требует запуск по http(s). Открой плеер с хостинга или с http://127.0.0.1, а не из файла.';
  else if(c===100) msg='Видео удалено или приватное.';
  else if(c===2) msg='Неверная ссылка на видео.';
  else msg='YouTube не смог воспроизвести (код '+c+').';
  $('#stage').innerHTML = `<div style="display:grid;place-items:center;height:100%;text-align:center;color:var(--muted);font-size:13px;padding:20px;gap:12px;line-height:1.5">
    <div>${msg}</div>
    ${t?`<a href="${esc(t.url)}" target="_blank" rel="noopener" style="color:var(--youtube);font-weight:600;text-decoration:none">Открыть на YouTube ↗</a>`:''}</div>`;
  setPlaying(false);
}

function onEnded(){
  if(state.settings.repeat==='one'){ playTrack(currentId); return; }
  next(true);
}
function setPlaying(v){
  isPlaying=v;
  $('#cPlay').textContent = v ? '⏸' : '▶';
  document.body.classList.toggle('playing', v);
  document.body.classList.toggle('not-playing', !v);
  document.querySelectorAll('.track').forEach(el=>{
    el.classList.toggle('is-playing', v && el.dataset.id===currentId);
  });
}

function togglePlay(){
  if(!currentId){ if(queue.length) playTrack(queue[0]); return; }
  const t = state.tracks.find(x=>x.id===currentId);
  if(!t) return;
  if(t.source==='youtube' && ytPlayer) isPlaying?ytPlayer.pauseVideo():ytPlayer.playVideo();
  else if(t.source==='soundcloud' && scWidget) scWidget.toggle();
  else if(t.source==='spotify' && spController) spController.togglePlay();
}

function next(auto){
  if(!queue.length) return;
  let i = queue.indexOf(currentId);
  if(state.settings.shuffle){
    if(queue.length===1 && auto && state.settings.repeat!=='all') return;
    let n; do{ n=Math.floor(Math.random()*queue.length); }while(n===i && queue.length>1);
    playTrack(queue[n]); return;
  }
  if(i<queue.length-1) playTrack(queue[i+1]);
  else if(state.settings.repeat==='all') playTrack(queue[0]);
}
function prev(){
  if(!queue.length) return;
  let i = queue.indexOf(currentId);
  if(i>0) playTrack(queue[i-1]); else playTrack(queue[Math.max(0,i)]);
}

function renderDock(){
  const t = state.tracks.find(x=>x.id===currentId);
  const now = $('#dockNow');
  if(!t){ now.innerHTML = `<div class="dock-empty" style="padding:0">Выбери трек, чтобы начать</div>`; $('#dock').style.setProperty('--src','var(--muted)'); return; }
  const s=SRC[t.source];
  $('#dock').style.setProperty('--src', s.color);
  now.innerHTML = `<div class="badge" style="color:${s.color}">${s.tag}</div>
    <div class="t"><div class="nm">${esc(t.name||prettyUrl(t))}</div><div class="s"><b style="color:${s.color}">${s.name}</b></div></div>`;
}

//============================ Плейлисты / действия ============================
function newPlaylist(){
  const name = prompt('Название плейлиста:');
  if(!name) return;
  state.playlists.push({ id:uid(), name:name.trim(), trackIds:[] }); save(); render();
}
function delPlaylist(id){
  if(!confirm('Удалить плейлист? Сами треки останутся в библиотеке.')) return;
  state.playlists = state.playlists.filter(p=>p.id!==id);
  if(view.type==='playlist'&&view.id===id) view={type:'all'};
  save(); render();
}
function addToPlaylist(trackId, x, y){
  if(!state.playlists.length){ toast('Сначала создай плейлист'); return; }
  let h = `<div class="hd">Добавить в плейлист</div>`;
  for(const pl of state.playlists){
    const has = pl.trackIds.includes(trackId);
    h += `<button data-pick="${pl.id}">${has?'✓ ':''}${esc(pl.name)}</button>`;
  }
  openSheet(h, x, y, el=>{
    const id = el.dataset.pick; if(!id) return;
    const pl = state.playlists.find(p=>p.id===id);
    if(pl.trackIds.includes(trackId)) pl.trackIds = pl.trackIds.filter(t=>t!==trackId);
    else pl.trackIds.push(trackId);
    save(); render(); closeSheet();
  });
}
function renameTrack(id){
  const t=state.tracks.find(x=>x.id===id); if(!t) return;
  const n = prompt('Название трека:', t.name||'');
  if(n!==null){ t.name=n.trim(); save(); render(); }
}
function removeTrack(id){
  state.tracks = state.tracks.filter(t=>t.id!==id);
  state.playlists.forEach(p=>p.trackIds = p.trackIds.filter(t=>t!==id));
  if(currentId===id){ currentId=null; clearPlayers(); $('#progress').style.setProperty('--pct','0%'); }
  save(); render();
}

//============================ Экспорт / импорт ============================
function exportLib(){
  const clone = JSON.parse(JSON.stringify(state));
  if(clone.spotify){ delete clone.spotify.accessToken; delete clone.spotify.refreshToken; delete clone.spotify.expiresAt; }
  if(clone.sync){ delete clone.sync.token; }
  const blob = new Blob([JSON.stringify(clone,null,2)], {type:'application/json'});
  const a=document.createElement('a'); a.href=URL.createObjectURL(blob);
  a.download = `rezonans-${new Date().toISOString().slice(0,10)}.json`; a.click();
  toast('Библиотека сохранена в файл');
}
function importLib(file){
  const r=new FileReader();
  r.onload=()=>{ try{
    const d=JSON.parse(r.result);
    if(!d.tracks) throw 0;
    const keepSp = state.spotify, keepSync = state.sync;
    state=Object.assign({tracks:[],playlists:[],settings:state.settings}, d);
    if(keepSp) state.spotify = keepSp;
    if(keepSync) state.sync = keepSync;
    save(); applyTheme(); render(); updateSpotifyUI(); updateSyncUI(); toast('Библиотека загружена');
  }catch(e){ toast('Не удалось прочитать файл'); } };
  r.readAsText(file);
}

//============================ Утилиты ============================
function esc(s){ return String(s).replace(/[&<>"]/g, c=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;'}[c])); }
function prettyUrl(t){
  if(t.source==='soundcloud'){ const p=t.ref.replace(/\/$/,'').split('/'); return decodeURIComponent(p.slice(-2).join(' — ')).replace(/-/g,' '); }
  if(t.source==='spotify') return 'Трек Spotify · '+t.ref.slice(0,6);
  if(t.source==='vk') return 'Трек VK';
  return 'Видео YouTube · '+t.ref;
}
function shortRef(t){ return (t.source==='soundcloud'||t.source==='vk') ? t.ref.replace(/^https?:\/\//,'') : t.ref; }
function plural(n){ const a=Math.abs(n)%100,b=a%10; if(a>10&&a<20)return'треков'; if(b>1&&b<5)return'трека'; if(b===1)return'трек'; return'треков'; }

let toastT;
function toast(msg){ const el=$('#toast'); el.textContent=msg; el.classList.add('show'); clearTimeout(toastT); toastT=setTimeout(()=>el.classList.remove('show'),2200); }

function openSheet(html,x,y,onClick){
  let sh=$('#sheet');
  if(!sh){ sh=document.createElement('div'); sh.className='sheet'; sh.id='sheet'; document.body.appendChild(sh); }
  sh.innerHTML=html; sh.style.display='block';
  const w=Math.min(240, innerWidth-24);
  sh.style.left=Math.min(x, innerWidth-w-12)+'px';
  sh.style.top=Math.min(y, innerHeight-sh.offsetHeight-12)+'px';
  $('#scrim').classList.add('show');
  sh.onclick=e=>{ const b=e.target.closest('button'); if(b) onClick(b); };
}
function closeSheet(){ const sh=$('#sheet'); if(sh) sh.style.display='none'; $('#scrim').classList.remove('show'); }

function applyTheme(){ document.documentElement.setAttribute('data-theme', state.settings.theme); 
  $('meta[name=theme-color]').setAttribute('content', state.settings.theme==='light'?'#F4F1EC':'#14110F'); }

//============================ Синхронизация (GitHub Gist) ============================
const GIST_FILE='rezonans-library.json';
function syncCfg(){ return state.sync || (state.sync={}); }
function ghReq(method, path, body){
  return fetch('https://api.github.com'+path, {
    method,
    headers:{ 'Authorization':'Bearer '+syncCfg().token, 'Accept':'application/vnd.github+json', 'Content-Type':'application/json' },
    body: body ? JSON.stringify(body) : undefined
  });
}
function syncPayload(){
  const p = JSON.parse(JSON.stringify(state));
  p.sync = {};
  if(p.spotify){ delete p.spotify.accessToken; delete p.spotify.refreshToken; delete p.spotify.expiresAt; }
  return p;
}
async function syncPush(manual){
  const c = syncCfg();
  if(!c.token){ if(manual) toast('Сначала вставь GitHub-токен'); return; }
  if(!manual && c.lastSyncTs && state.meta && state.meta.updatedAt <= c.lastSyncTs) return;
  try{
    const files = {}; files[GIST_FILE] = { content: JSON.stringify(syncPayload(), null, 2) };
    const r = c.gistId
      ? await ghReq('PATCH', '/gists/'+c.gistId, { files })
      : await ghReq('POST', '/gists', { description:'Резонанс — библиотека', public:false, files });
    if(!r.ok){ if(manual) toast('Ошибка выгрузки ('+r.status+')'); return; }
    const d = await r.json();
    c.gistId = d.id; c.lastSync = Date.now(); c.lastSyncTs = (state.meta&&state.meta.updatedAt)||Date.now();
    persistQuiet(); updateSyncUI();
    if(manual) toast('Выгружено в облако');
  }catch(e){ if(manual) toast('Сбой сети при выгрузке'); }
}
async function syncPull(manual){
  const c = syncCfg();
  if(!c.token || !c.gistId){ if(manual) toast('Нужны токен и Gist ID'); return; }
  try{
    const r = await ghReq('GET', '/gists/'+c.gistId);
    if(!r.ok){ if(manual) toast('Ошибка загрузки ('+r.status+')'); return; }
    const d = await r.json();
    const f = d.files && (d.files[GIST_FILE] || Object.values(d.files)[0]);
    if(!f){ if(manual) toast('В Gist нет данных'); return; }
    let content = f.content;
    if(f.truncated && f.raw_url){ content = await (await fetch(f.raw_url)).text(); }
    const remote = JSON.parse(content);
    if(!remote.tracks){ if(manual) toast('Неверный формат в облаке'); return; }
    const rTs = (remote.meta && remote.meta.updatedAt) || 0;
    const lTs = (state.meta && state.meta.updatedAt) || 0;
    if(!manual && rTs <= lTs) return;
    const keepSync = state.sync, keepSp = state.spotify;
    _remoteTs = rTs;
    state = Object.assign({ tracks:[], playlists:[], settings:state.settings }, remote);
    state.sync = keepSync; state.spotify = keepSp;
    save(); applyTheme(); render(); updateSpotifyUI();
    c.lastSync = Date.now(); c.lastSyncTs = rTs; persistQuiet(); updateSyncUI();
    if(manual) toast('Загружено из облака');
  }catch(e){ if(manual) toast('Сбой сети при загрузке'); }
}
function openSyncModal(){
  $('#syncModal').classList.add('show'); $('#scrim').classList.add('show'); updateSyncUI();
}
function closeSyncModal(){ $('#syncModal').classList.remove('show'); $('#scrim').classList.remove('show'); }
function updateSyncUI(){
  const c = syncCfg();
  const btn=$('#syncBtn'); if(btn) btn.querySelector('.sync-label').textContent = c.gistId ? 'Синхронизация: вкл' : 'Синхронизация';
  const tk=$('#syncToken'); if(tk && document.activeElement!==tk) tk.value = c.token||'';
  const gi=$('#syncGist'); if(gi && document.activeElement!==gi) gi.value = c.gistId||'';
  const au=$('#syncAuto'); if(au) au.checked = !!c.auto;
  const st=$('#syncStatus'); if(st) st.textContent = c.lastSync ? ('Последняя синхронизация: '+new Date(c.lastSync).toLocaleString('ru')) : 'Ещё не синхронизировано';
}

//============================ Spotify ============================
const SP_AUTH='https://accounts.spotify.com', SP_API='https://api.spotify.com/v1';
const SP_SCOPE='playlist-read-private playlist-read-collaborative';
function sp(){ return state.spotify || (state.spotify={}); }
function redirectUri(){ return location.origin + location.pathname; }
function spConnected(){ return !!sp().refreshToken; }

function b64url(buf){ let s=''; const b=new Uint8Array(buf); for(const x of b) s+=String.fromCharCode(x);
  return btoa(s).replace(/\+/g,'-').replace(/\//g,'_').replace(/=+$/,''); }
async function sha256(str){ return crypto.subtle.digest('SHA-256', new TextEncoder().encode(str)); }
function randHex(n){ const a=new Uint8Array(n); crypto.getRandomValues(a); return Array.from(a,x=>('0'+x.toString(16)).slice(-2)).join(''); }

async function spLogin(){
  const cid=sp().clientId;
  if(!cid){ toast('Сначала вставь Client ID'); return; }
  if(location.protocol==='file:'){ toast('Из локального файла вход не работает — см. предупреждение в окне'); return; }
  const verifier=randHex(48);
  localStorage.setItem('sp_verifier', verifier);
  const challenge=b64url(await sha256(verifier));
  const p=new URLSearchParams({ client_id:cid, response_type:'code', redirect_uri:redirectUri(),
    code_challenge_method:'S256', code_challenge:challenge, scope:SP_SCOPE });
  location.href=`${SP_AUTH}/authorize?`+p.toString();
}
async function spExchange(code){
  const verifier=localStorage.getItem('sp_verifier');
  const body=new URLSearchParams({ client_id:sp().clientId, grant_type:'authorization_code', code,
    redirect_uri:redirectUri(), code_verifier:verifier });
  const r=await fetch(`${SP_AUTH}/api/token`,{method:'POST',headers:{'Content-Type':'application/x-www-form-urlencoded'},body});
  if(!r.ok){ toast('Не удалось войти в Spotify'); return; }
  storeTok(await r.json()); localStorage.removeItem('sp_verifier');
  await spWhoAmI(); updateSpotifyUI(); toast('Spotify подключён');
}
function storeTok(d){ const s=sp(); s.accessToken=d.access_token; if(d.refresh_token) s.refreshToken=d.refresh_token;
  s.expiresAt=Date.now()+(d.expires_in-60)*1000; save(); }
async function spRefresh(){
  const r=await fetch(`${SP_AUTH}/api/token`,{method:'POST',headers:{'Content-Type':'application/x-www-form-urlencoded'},
    body:new URLSearchParams({ client_id:sp().clientId, grant_type:'refresh_token', refresh_token:sp().refreshToken })});
  if(!r.ok){ spLogout(); throw new Error('refresh failed'); }
  storeTok(await r.json());
}
async function spToken(){ if(Date.now()>=(sp().expiresAt||0)) await spRefresh(); return sp().accessToken; }
async function spApiGet(path){
  const tok=await spToken();
  const r=await fetch(path.startsWith('http')?path:SP_API+path, { headers:{ Authorization:'Bearer '+tok } });
  if(r.status===401){ await spRefresh(); return spApiGet(path); }
  if(!r.ok) throw new Error('Spotify API '+r.status);
  return r.json();
}
async function spWhoAmI(){ try{ const me=await spApiGet('/me'); sp().userName=me.display_name||me.id; save(); }catch(e){} }
function spLogout(){ const s=sp(); delete s.accessToken; delete s.refreshToken; delete s.expiresAt; delete s.userName; save(); updateSpotifyUI(); }

async function importSpotify(kind, id){
  if(!spConnected()){ openSpotifyModal(); toast('Подключи Spotify, чтобы разворачивать плейлисты'); return; }
  toast('Загружаю треки из Spotify…');
  try{
    let name, items=[];
    if(kind==='playlist'){
      const meta=await spApiGet(`/playlists/${id}?fields=name`); name=meta.name;
      let url=`/playlists/${id}/tracks?limit=100&fields=items(track(id,name,type,artists(name))),next`;
      while(url){ const page=await spApiGet(url); items.push(...page.items.map(i=>i.track)); url=page.next; }
    } else {
      const meta=await spApiGet(`/albums/${id}`); name=meta.name;
      const albArtist=meta.artists.map(a=>a.name).join(', ');
      let url=`/albums/${id}/tracks?limit=50`;
      while(url){ const page=await spApiGet(url); items.push(...page.items.map(t=>Object.assign({_alb:albArtist},t))); url=page.next; }
    }
    let added=0; const ids=[];
    for(const tr of items){
      if(!tr || !tr.id || tr.type==='episode') continue;
      const artist=(tr.artists&&tr.artists.length)?tr.artists.map(a=>a.name).join(', '):(tr._alb||'');
      const nm=(artist?artist+' — ':'')+tr.name;
      let ex=state.tracks.find(t=>t.source==='spotify'&&t.ref===tr.id);
      if(!ex){ ex={ id:uid(), source:'spotify', ref:tr.id, url:`https://open.spotify.com/track/${tr.id}`, name:nm, added:Date.now() };
        state.tracks.unshift(ex); added++; }
      ids.push(ex.id);
    }
    let pl=state.playlists.find(p=>p.name===name);
    if(!pl){ pl={ id:uid(), name, trackIds:[] }; state.playlists.push(pl); }
    for(const tid of ids) if(!pl.trackIds.includes(tid)) pl.trackIds.push(tid);
    save(); render(); closeSpotifyModal();
    toast(`«${name}»: добавлено ${added}, в плейлисте ${ids.length}`);
  }catch(e){ toast('Не удалось загрузить плейлист (проверь доступ и подключение)'); }
}

async function importMyPlaylists(){
  const box=document.getElementById('spLists'); if(box) box.innerHTML='<div class="sp-hint">Загружаю список…</div>';
  try{
    let url='/me/playlists?limit=50', all=[];
    while(url){ const page=await spApiGet(url); all.push(...page.items); url=page.next; }
    if(!box) return;
    box.innerHTML = all.length ? '' : '<div class="sp-hint">Плейлистов не найдено</div>';
    for(const pl of all){
      if(!pl) continue;
      const row=document.createElement('div'); row.className='sp-row';
      row.innerHTML=`<span>${esc(pl.name)}</span><span class="sp-cnt">${pl.tracks.total}</span><button data-imp="${pl.id}" title="Импортировать">+</button>`;
      box.appendChild(row);
    }
    box.onclick=e=>{ const b=e.target.closest('[data-imp]'); if(b) importSpotify('playlist', b.dataset.imp); };
  }catch(e){ if(box) box.innerHTML='<div class="sp-hint">Не удалось получить плейлисты</div>'; }
}

function openSpotifyModal(){
  $('#spModal').classList.add('show'); $('#scrim').classList.add('show');
  $('#spClientId').value = sp().clientId||'';
  $('#spRedirect').textContent = redirectUri();
  $('#spFileWarn').style.display = location.protocol==='file:' ? 'block' : 'none';
  updateSpotifyUI();
}
function closeSpotifyModal(){ $('#spModal').classList.remove('show'); $('#scrim').classList.remove('show'); }
function updateSpotifyUI(){
  const conn=spConnected();
  const btn=$('#spConnectBtn');
  if(btn){ btn.querySelector('.sp-dot').style.background = conn?'var(--spotify)':'var(--faint)';
    btn.querySelector('.sp-label').textContent = conn ? `Spotify · ${sp().userName||'подключён'}` : 'Подключить Spotify'; }
  const lg=$('#spLoginBtn'); if(lg) lg.textContent = conn?'Переподключить':'Подключить';
  const out=$('#spLogoutBtn'); if(out) out.style.display = conn?'':'none';
  const my=$('#spMyBtn'); if(my) my.style.display = conn?'':'none';
}

//============================ События ============================
$('#addBtn').onclick=()=>{ addTrack($('#addInput').value); $('#addInput').value=''; };
$('#addInput').onkeydown=e=>{ if(e.key==='Enter'){ addTrack($('#addInput').value); $('#addInput').value=''; } };
$('#searchInput').oninput=()=>list();

$('#nav').onclick=e=>{
  const v=e.target.closest('[data-v]'); const pl=e.target.closest('[data-pl]');
  const del=e.target.closest('[data-delpl]'); const add=e.target.closest('#addPl');
  if(del){ e.stopPropagation(); delPlaylist(del.dataset.delpl); return; }
  if(add){ newPlaylist(); return; }
  if(v){ view={type:v.dataset.v}; closeSide(); render(); }
  else if(pl){ view={type:'playlist', id:pl.dataset.pl}; closeSide(); render(); }
};

$('#list').onclick=e=>{
  const rn=e.target.closest('[data-rename]'); const at=e.target.closest('[data-addto]');
  const rm=e.target.closest('[data-remove]'); const tr=e.target.closest('.track');
  if(rn){ e.stopPropagation(); renameTrack(rn.dataset.rename); return; }
  if(at){ e.stopPropagation(); const r=at.getBoundingClientRect(); addToPlaylist(at.dataset.addto, r.left, r.bottom+6); return; }
  if(rm){ e.stopPropagation(); removeTrack(rm.dataset.remove); return; }
  if(tr){ playTrack(tr.dataset.id); $('#dock').classList.add('open'); }
};

$('#cPlay').onclick=togglePlay;
$('#cNext').onclick=()=>next(false);
$('#cPrev').onclick=prev;
$('#cShuffle').onclick=()=>{ state.settings.shuffle=!state.settings.shuffle; save(); $('#cShuffle').classList.toggle('on',state.settings.shuffle); toast(state.settings.shuffle?'Перемешивание включено':'Перемешивание выключено'); };
$('#cRepeat').onclick=()=>{
  const order=['off','all','one']; let i=order.indexOf(state.settings.repeat);
  state.settings.repeat=order[(i+1)%3]; save();
  const r=state.settings.repeat;
  $('#cRepeat').classList.toggle('on', r!=='off');
  $('#cRepeat').textContent = r==='one'?'⟳':'⟲';
  toast(r==='off'?'Повтор выключен':r==='all'?'Повтор всех':'Повтор трека');
};
$('#cExpand').onclick=()=>{ const d=$('#dock'); d.classList.toggle('open'); $('#cExpand').textContent=d.classList.contains('open')?'⌃':'⌄'; };

$('#btnTheme').onclick=()=>{ state.settings.theme = state.settings.theme==='light'?'dark':'light'; save(); applyTheme(); };
$('#btnExport').onclick=exportLib;
$('#btnImport').onclick=()=>$('#fileInput').click();
$('#fileInput').onchange=e=>{ if(e.target.files[0]) importLib(e.target.files[0]); e.target.value=''; };

$('#menuBtn').onclick=()=>$('#side').classList.add('open');
$('#scrim').onclick=()=>{ closeSheet(); closeSide(); closeSpotifyModal(); closeSyncModal(); };

$('#syncBtn').onclick=openSyncModal;
$('#syncClose').onclick=closeSyncModal;
$('#syncToken').oninput=e=>{ syncCfg().token=e.target.value.trim(); persistQuiet(); };
$('#syncGist').oninput=e=>{ syncCfg().gistId=e.target.value.trim(); persistQuiet(); };
$('#syncAuto').onchange=e=>{ syncCfg().auto=e.target.checked; persistQuiet(); updateSyncUI(); };
$('#syncPushBtn').onclick=()=>syncPush(true);
$('#syncPullBtn').onclick=()=>syncPull(true);
$('#syncCopyGist').onclick=()=>{ navigator.clipboard && navigator.clipboard.writeText(syncCfg().gistId||''); toast('Gist ID скопирован'); };

$('#spConnectBtn').onclick=openSpotifyModal;
$('#spClose').onclick=closeSpotifyModal;
$('#spClientId').oninput=e=>{ sp().clientId=e.target.value.trim(); save(); };
$('#spLoginBtn').onclick=spLogin;
$('#spLogoutBtn').onclick=()=>{ spLogout(); toast('Spotify отключён'); };
$('#spMyBtn').onclick=importMyPlaylists;
$('#spCopyRedirect').onclick=()=>{ navigator.clipboard && navigator.clipboard.writeText(redirectUri()); toast('Redirect URI скопирован'); };
function closeSide(){ $('#side').classList.remove('open'); }

document.addEventListener('keydown', e=>{
  if(e.target.tagName==='INPUT') return;
  if(e.code==='Space'){ e.preventDefault(); togglePlay(); }
  else if(e.code==='ArrowRight'&&e.ctrlKey) next(false);
  else if(e.code==='ArrowLeft'&&e.ctrlKey) prev();
});

//============================ Старт ============================
applyTheme();
$('#cShuffle').classList.toggle('on', state.settings.shuffle);
if(state.settings.repeat && state.settings.repeat!=='off'){ $('#cRepeat').classList.add('on'); if(state.settings.repeat==='one')$('#cRepeat').textContent='⟳'; }
render();

(function spInit(){
  const q=new URLSearchParams(location.search);
  if(q.get('code') && sp().clientId){
    spExchange(q.get('code')).finally(()=>{ history.replaceState({}, '', redirectUri()); render(); });
  } else if(q.get('error')){
    history.replaceState({}, '', redirectUri()); toast('Доступ к Spotify не выдан');
  }
  updateSpotifyUI();
})();

updateSyncUI();
if(syncCfg().auto && syncCfg().token && syncCfg().gistId){ syncPull(false); }
</script>
</body>
</html>
