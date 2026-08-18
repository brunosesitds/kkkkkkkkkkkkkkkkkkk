<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Correios LogiSense</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&family=IBM+Plex+Mono:wght@400;500;600;700&display=swap" rel="stylesheet">
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.4/chart.umd.min.js"></script>
<style>
  :root{
    --navy-900:#071B33; --navy-800:#0C2A4D; --navy-600:#16406F;
    --azul:#1B5FAE; --azul-hover:#164C8C; --azul-claro:#4C8DD9;
    --verde:#17924F; --verde-soft:#E7F5ED;
    --amarelo:#C88A12; --amarelo-text:#B4790E; --amarelo-soft:#FBF1DD;
    --vermelho:#C0342A; --vermelho-soft:#FBEAE8;
    --bg:#F3F5F8; --line:#E7EBF1; --border:#E2E8F0;
    --ink:#0F172A; --muted:#64748B; --muted-2:#94A3B8;
  }
  *{box-sizing:border-box;}
  html,body{margin:0;padding:0;}
  body{font-family:'Inter',system-ui,sans-serif;background:var(--bg);color:var(--ink);}
  .mono{font-family:'IBM Plex Mono',ui-monospace,monospace;font-variant-numeric:tabular-nums;}
  button,select,input{font-family:inherit;}
  button{cursor:pointer;border:none;background:none;}
  a{text-decoration:none;color:inherit;}

  /* ---------- layout ---------- */
  #app{display:flex;min-height:100vh;}
  #sidebar{
    width:250px;flex-shrink:0;min-height:100vh;position:sticky;top:0;
    background:linear-gradient(180deg,#071B33 0%,#0C2A4D 100%);
    color:#fff;display:flex;flex-direction:column;
  }
  .sb-header{padding:20px;border-bottom:1px solid rgba(255,255,255,.1);display:flex;align-items:center;gap:10px;}
  .sb-logo{width:36px;height:36px;border-radius:8px;background:var(--azul);display:flex;align-items:center;justify-content:center;flex-shrink:0;}
  .sb-title{font-weight:800;font-size:15px;line-height:1.1;}
  .sb-sub{font-size:13px;color:#8FB4E0;letter-spacing:.03em;}
  .sb-nav{flex:1;padding:12px 8px;display:flex;flex-direction:column;gap:2px;overflow-y:auto;}
  .sb-item{
    display:flex;align-items:center;gap:11px;width:100%;padding:10px 12px;border-radius:8px;
    font-size:13.5px;font-weight:500;color:#B8CCE3;transition:background .15s,color .15s;text-align:left;
  }
  .sb-item:hover{background:rgba(255,255,255,.06);color:#fff;}
  .sb-item.active{background:var(--azul);color:#fff;}
  .sb-item svg{flex-shrink:0;}
  .sb-item .chev{margin-left:auto;opacity:.7;}
  .sb-footer{padding:16px;border-top:1px solid rgba(255,255,255,.1);font-size:11px;color:#6E8BAE;line-height:1.5;}

  #main-col{flex:1;min-width:0;}
  #topbar{
    position:sticky;top:0;z-index:20;background:#fff;border-bottom:1px solid #E2E8F0;
    padding:12px 24px;display:flex;align-items:center;gap:16px;flex-wrap:wrap;
  }
  .tb-brand{min-width:170px;}
  .tb-eyebrow{font-size:11px;color:#94A3B8;text-transform:uppercase;letter-spacing:.05em;}
  .tb-title{font-size:15px;font-weight:700;color:var(--navy-800);}
  .tb-live{
    display:inline-flex;align-items:center;gap:6px;padding:4px 10px;border-radius:999px;
    background:var(--verde-soft);color:var(--verde);font-size:12px;font-weight:700;
  }
  .pulse-dot{position:relative;width:8px;height:8px;}
  .pulse-dot::before,.pulse-dot::after{content:'';position:absolute;inset:0;border-radius:999px;background:var(--verde);}
  .pulse-dot::before{animation:ping 1.6s cubic-bezier(0,0,.2,1) infinite;opacity:.6;}
  @keyframes ping{75%,100%{transform:scale(2.4);opacity:0;}}
  .tb-search{flex:1;max-width:400px;position:relative;margin-left:8px;min-width:180px;}
  .tb-search input{
    width:100%;padding:9px 12px 9px 34px;border-radius:8px;background:#F1F5F9;border:1px solid transparent;
    font-size:13.5px;color:#334155;outline:none;
  }
  .tb-search input:focus{background:#fff;border-color:rgba(27,95,174,.3);box-shadow:0 0 0 3px rgba(27,95,174,.12);}
  .tb-search svg{position:absolute;left:11px;top:50%;transform:translateY(-50%);color:#94A3B8;}
  .tb-actions{margin-left:auto;display:flex;align-items:center;gap:12px;position:relative;}
  .icon-btn{position:relative;width:36px;height:36px;border-radius:8px;display:flex;align-items:center;justify-content:center;color:#64748B;}
  .icon-btn:hover{background:#F1F5F9;}
  .dot-badge{position:absolute;top:6px;right:6px;width:8px;height:8px;border-radius:999px;background:var(--vermelho);}
  .divider-v{width:1px;height:24px;background:#E2E8F0;}
  .tb-profile{display:flex;align-items:center;gap:8px;}
  .avatar{width:32px;height:32px;border-radius:999px;background:var(--azul);color:#fff;display:flex;align-items:center;justify-content:center;}
  .profile-name{font-size:13px;font-weight:600;color:var(--navy-800);}
  .profile-role{font-size:11px;color:#94A3B8;}
  #notif-panel{
    position:absolute;right:150px;top:46px;width:320px;background:#fff;border:1px solid #E2E8F0;border-radius:10px;
    box-shadow:0 10px 30px rgba(15,23,42,.15);overflow:hidden;z-index:30;
  }
  .notif-header{padding:10px 16px;border-bottom:1px solid #F1F5F9;font-size:11px;font-weight:700;color:#64748B;text-transform:uppercase;letter-spacing:.04em;}
  .notif-item{display:flex;gap:10px;text-align:left;width:100%;padding:12px 16px;border-bottom:1px solid #F8FAFC;}
  .notif-item:hover{background:#F8FAFC;}
  .notif-item .nd{width:8px;height:8px;border-radius:999px;margin-top:6px;flex-shrink:0;}
  .notif-title{font-size:13px;font-weight:600;color:var(--navy-800);}
  .notif-text{font-size:12px;color:#64748B;margin-top:2px;line-height:1.3;}

  main{padding:24px;max-width:1400px;margin:0 auto;}
  .section-gap{display:flex;flex-direction:column;gap:24px;}

  /* ---------- cards & primitives ---------- */
  .card{background:#fff;border-radius:10px;border:1px solid #E2E8F0;box-shadow:0 1px 2px rgba(15,23,42,.04);}
  .p-4{padding:16px;} .p-5{padding:20px;}
  .section-head{display:flex;align-items:flex-end;justify-content:space-between;margin-bottom:16px;gap:12px;flex-wrap:wrap;}
  .section-eyebrow{font-family:'IBM Plex Mono';font-size:11px;letter-spacing:.08em;text-transform:uppercase;color:var(--azul-claro);margin-bottom:3px;}
  .section-title{font-size:17px;font-weight:700;color:var(--navy-800);}

  .badge{display:inline-flex;align-items:center;gap:6px;padding:2px 9px;border-radius:6px;font-size:12px;font-weight:700;}
  .badge .bd{width:6px;height:6px;border-radius:999px;}
  .badge-alto{background:var(--vermelho-soft);color:var(--vermelho);}
  .badge-medio{background:var(--amarelo-soft);color:var(--amarelo-text);}
  .badge-baixo{background:var(--verde-soft);color:var(--verde);}
  .status-normal{background:var(--verde-soft);color:var(--verde);}
  .status-atencao{background:var(--amarelo-soft);color:var(--amarelo-text);}
  .status-critico{background:var(--vermelho-soft);color:var(--vermelho);}

  .delta{display:inline-flex;align-items:center;gap:2px;font-weight:700;font-size:12px;}
  .delta-good{color:var(--verde);} .delta-bad{color:var(--vermelho);}

  /* grids */
  .grid{display:grid;gap:16px;}
  .grid-6{grid-template-columns:repeat(6,1fr);}
  .grid-5{grid-template-columns:repeat(5,1fr);}
  .grid-4{grid-template-columns:repeat(4,1fr);}
  .grid-3{grid-template-columns:repeat(3,1fr);}
  .grid-2{grid-template-columns:repeat(2,1fr);}
  @media(max-width:1100px){.grid-6{grid-template-columns:repeat(3,1fr);} .grid-5{grid-template-columns:repeat(3,1fr);} .grid-4{grid-template-columns:repeat(2,1fr);} .grid-3{grid-template-columns:repeat(2,1fr);}}
  @media(max-width:640px){.grid-6,.grid-5,.grid-4,.grid-3,.grid-2{grid-template-columns:repeat(2,1fr);}}

  .kpi{display:flex;flex-direction:column;gap:10px;}
  .kpi-top{display:flex;align-items:center;justify-content:space-between;}
  .kpi-label{font-size:12px;font-weight:600;color:#64748B;}
  .kpi-bottom{display:flex;align-items:flex-end;justify-content:space-between;}
  .kpi-value{font-size:24px;font-weight:700;color:var(--navy-800);}

  .metric{background:#F8FAFC;border-radius:8px;padding:9px 12px;}
  .metric-label{font-size:10.5px;color:#94A3B8;}
  .metric-value{font-weight:700;color:var(--navy-800);}

  table{width:100%;border-collapse:collapse;font-size:14px;}
  thead tr{background:#F8FAFC;color:#64748B;font-size:11.5px;text-transform:uppercase;letter-spacing:.03em;}
  th{text-align:left;padding:10px 14px;font-weight:600;}
  td{padding:12px 14px;border-top:1px solid #F1F5F9;}
  tbody tr:hover{background:#F8FAFC;cursor:pointer;}
  .table-head-row{padding:16px 20px;border-bottom:1px solid #F1F5F9;display:flex;justify-content:space-between;align-items:center;}

  .btn{display:inline-flex;align-items:center;gap:7px;padding:9px 16px;border-radius:8px;font-size:13.5px;font-weight:700;transition:background .15s;}
  .btn-primary{background:var(--azul);color:#fff;} .btn-primary:hover{background:var(--azul-hover);}
  .btn-outline{background:#fff;border:1px solid #E2E8F0;color:var(--navy-800);} .btn-outline:hover{background:#F8FAFC;}
  .btn-ghost{color:#94A3B8;} .btn-ghost:hover{color:#475569;}
  .btn-block{width:100%;justify-content:center;}

  .filter-bar{display:flex;flex-wrap:wrap;gap:14px;align-items:center;}
  .filter-label{display:flex;align-items:center;gap:8px;font-size:12px;}
  .filter-label span{color:#64748B;font-weight:600;}
  .filter-label select{border:1px solid #E2E8F0;border-radius:8px;padding:6px 10px;font-size:12px;color:#334155;background:#fff;}

  /* map */
  .map-wrap{position:relative;height:520px;border-radius:8px;overflow:hidden;border:1px solid #E2E8F0;
    background:radial-gradient(circle at 30% 20%,#0F3054 0%,#071B33 70%);
    background-image:linear-gradient(rgba(255,255,255,.05) 1px,transparent 1px),linear-gradient(90deg,rgba(255,255,255,.05) 1px,transparent 1px);
    background-size:24px 24px,24px 24px;
  }
  .map-svg{position:absolute;inset:0;width:100%;height:100%;}
  .map-label{
    position:absolute;transform:translate(-50%,calc(-100% - 6px));font-family:'IBM Plex Mono';font-size:10.5px;font-weight:600;
    color:rgba(255,255,255,.9);background:rgba(0,0,0,.3);padding:2px 6px;border-radius:5px;white-space:nowrap;backdrop-filter:blur(2px);
  }
  .map-label:hover{background:rgba(0,0,0,.6);}
  .map-legend{position:absolute;bottom:12px;left:12px;display:flex;gap:12px;font-family:'IBM Plex Mono';font-size:11px;color:rgba(255,255,255,.8);background:rgba(0,0,0,.3);padding:6px 12px;border-radius:6px;}
  .map-legend span{display:flex;align-items:center;gap:5px;}
  .leg-dot{width:8px;height:8px;border-radius:999px;}

  .center-empty{height:100%;min-height:400px;display:flex;flex-direction:column;align-items:center;justify-content:center;text-align:center;color:#94A3B8;gap:8px;padding:40px 0;}

  /* recommendation cards */
  .rec-card.applied,.rec-card.ignored{opacity:.6;}
  .rec-flags{display:flex;align-items:center;gap:8px;margin-bottom:4px;flex-wrap:wrap;}
  .rec-id{font-family:'IBM Plex Mono';font-size:11px;font-weight:700;color:#94A3B8;}
  .rec-priority{font-size:12px;font-weight:700;display:inline-flex;align-items:center;gap:4px;}
  .rec-body{display:grid;grid-template-columns:1fr 1fr;gap:16px;margin-top:10px;font-size:13.5px;}
  @media(max-width:700px){.rec-body{grid-template-columns:1fr;}}
  .rec-body h4{font-size:11px;font-weight:700;color:#94A3B8;text-transform:uppercase;margin:0 0 3px;}
  .rec-body p{margin:0;color:#475569;}
  .rec-impact{display:grid;grid-template-columns:repeat(4,1fr);gap:10px;margin-top:14px;}
  @media(max-width:640px){.rec-impact{grid-template-columns:repeat(2,1fr);}}
  .rec-actions{display:flex;gap:8px;margin-top:14px;flex-wrap:wrap;}

  .toast{position:fixed;bottom:24px;right:24px;z-index:50;background:var(--navy-800);color:#fff;padding:14px 20px;border-radius:10px;
    box-shadow:0 10px 30px rgba(0,0,0,.25);display:flex;align-items:center;gap:10px;font-size:13.5px;font-weight:500;}

  /* simulador */
  .field{display:block;margin-bottom:16px;}
  .field-label-row{display:flex;justify-content:space-between;margin-bottom:5px;font-size:12px;}
  .field-label-row span:first-child{color:#64748B;font-weight:500;}
  .field-label-row span:last-child{font-family:'IBM Plex Mono';font-weight:700;color:var(--navy-800);}
  input[type=range]{width:100%;accent-color:var(--azul);}
  .field select{width:100%;margin-top:5px;border:1px solid #E2E8F0;border-radius:8px;padding:9px 10px;font-size:13.5px;color:#334155;}
  .scenario-title{font-family:'IBM Plex Mono';font-size:11px;text-transform:uppercase;margin-bottom:12px;}
  .result-row{display:flex;justify-content:space-between;align-items:center;margin-bottom:10px;}
  .result-row span:first-child{font-size:13.5px;color:#64748B;}
  .result-row span:last-child{font-family:'IBM Plex Mono';font-weight:700;font-size:16px;}
  .savings-card{background:var(--verde-soft);border:1px solid rgba(23,146,79,.25);display:flex;justify-content:space-between;align-items:center;flex-wrap:wrap;gap:8px;}
  .savings-card b{color:var(--navy-800);font-size:13.5px;}
  .savings-card .val{font-family:'IBM Plex Mono';font-size:20px;font-weight:700;color:var(--verde);}

  /* centros grid */
  .centro-card{cursor:pointer;transition:box-shadow .15s;}
  .centro-card:hover{box-shadow:0 4px 14px rgba(15,23,42,.08);}
  .centro-top{display:flex;justify-content:space-between;align-items:flex-start;margin-bottom:10px;}
  .centro-uf{font-family:'IBM Plex Mono';font-size:11px;text-transform:uppercase;color:#94A3B8;}
  .centro-name{font-weight:700;color:var(--navy-800);}
  .bar-track{width:100%;height:6px;background:#F1F5F9;border-radius:999px;overflow:hidden;margin-bottom:8px;}
  .bar-fill{height:100%;border-radius:999px;}
  .centro-stats{display:flex;justify-content:space-between;font-family:'IBM Plex Mono';font-size:12px;color:#64748B;}

  .causa-item{display:flex;align-items:flex-start;gap:10px;font-size:13.5px;color:#475569;margin-bottom:11px;}
  .causa-num{font-family:'IBM Plex Mono';font-size:11px;font-weight:700;color:#fff;background:var(--navy-800);width:20px;height:20px;border-radius:999px;display:flex;align-items:center;justify-content:center;flex-shrink:0;margin-top:1px;}

  .report-row{display:flex;justify-content:space-between;align-items:center;padding:14px 20px;border-top:1px solid #F8FAFC;}
  .report-row:hover{background:#F8FAFC;}
  .report-left{display:flex;align-items:center;gap:12px;}
  .report-icon{width:36px;height:36px;border-radius:8px;background:#EAF1FA;color:var(--azul);display:flex;align-items:center;justify-content:center;flex-shrink:0;}
  .report-name{font-weight:600;font-size:13.5px;color:var(--navy-800);}
  .report-period{font-size:11px;color:#94A3B8;font-family:'IBM Plex Mono';}
  .report-type{font-size:11px;font-weight:700;color:#94A3B8;border:1px solid #E2E8F0;border-radius:6px;padding:3px 8px;font-family:'IBM Plex Mono';}

  .conf-bar{width:100%;background:#F1F5F9;border-radius:999px;height:8px;}
  .conf-fill{height:100%;border-radius:999px;background:var(--verde);}

  .factors-list{list-style:none;margin:0;padding:0;display:flex;flex-direction:column;gap:9px;}
  .factors-list li{display:flex;align-items:center;gap:9px;font-size:13.5px;color:#475569;}
  .factors-list li::before{content:'';width:6px;height:6px;border-radius:999px;background:var(--azul);flex-shrink:0;}

  .horizonte-toggle{display:flex;gap:2px;background:#F1F5F9;border-radius:8px;padding:4px;}
  .horizonte-toggle button{padding:6px 12px;border-radius:6px;font-size:12px;font-weight:700;color:#64748B;}
  .horizonte-toggle button.active{background:#fff;box-shadow:0 1px 2px rgba(0,0,0,.08);color:var(--navy-800);}

  .chart-box{position:relative;width:100%;}
  .footnote{font-size:11.5px;color:#94A3B8;margin-top:6px;}
</style>
</head>
<body>
<div id="app"></div>

<script>
/* =============================================================== ICONS === */
const ICONS = {
  dashboard:'<svg width="17" height="17" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="3" width="7" height="9"/><rect x="14" y="3" width="7" height="5"/><rect x="14" y="12" width="7" height="9"/><rect x="3" y="16" width="7" height="5"/></svg>',
  map:'<svg width="17" height="17" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polygon points="1 6 1 22 8 18 16 22 23 18 23 2 16 6 8 2 1 6"/><line x1="8" y1="2" x2="8" y2="18"/><line x1="16" y1="6" x2="16" y2="22"/></svg>',
  alert:'<svg width="17" height="17" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"/><line x1="12" y1="9" x2="12" y2="13"/><line x1="12" y1="17" x2="12.01" y2="17"/></svg>',
  truck:'<svg width="17" height="17" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="1" y="3" width="15" height="13"/><polygon points="16 8 20 8 23 11 23 16 16 16 16 8"/><circle cx="5.5" cy="18.5" r="2.5"/><circle cx="18.5" cy="18.5" r="2.5"/></svg>',
  warehouse:'<svg width="17" height="17" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M3 21V10l9-6 9 6v11"/><path d="M9 21v-6h6v6"/></svg>',
  trend:'<svg width="17" height="17" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="23 6 13.5 15.5 8.5 10.5 1 18"/><polyline points="17 6 23 6 23 12"/></svg>',
  sparkles:'<svg width="17" height="17" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 3l1.9 5.3L19 10l-5.1 1.7L12 17l-1.9-5.3L5 10l5.1-1.7L12 3z"/></svg>',
  report:'<svg width="17" height="17" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/><line x1="9" y1="15" x2="15" y2="15"/><line x1="9" y1="11" x2="12" y2="11"/></svg>',
  search:'<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/></svg>',
  bell:'<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M18 8A6 6 0 0 0 6 8c0 7-3 9-3 9h18s-3-2-3-9"/><path d="M13.73 21a2 2 0 0 1-3.46 0"/></svg>',
  user:'<svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/><circle cx="12" cy="7" r="4"/></svg>',
  chev:'<svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="9 18 15 12 9 6"/></svg>',
  arrowUp:'<svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round"><line x1="12" y1="19" x2="12" y2="5"/><polyline points="5 12 12 5 19 12"/></svg>',
  arrowDown:'<svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round"><line x1="12" y1="5" x2="12" y2="19"/><polyline points="19 12 12 19 5 12"/></svg>',
  pin:'<svg width="26" height="26" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"><path d="M21 10c0 7-9 13-9 13s-9-6-9-13a9 9 0 0 1 18 0z"/><circle cx="12" cy="10" r="3"/></svg>',
  x:'<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg>',
  sliders:'<svg width="15" height="15" 
              
              
              
              
              
              
              
              
              
              
              
              <div>
                <div className="font-data text-[11px] uppercase text-slate-400">{c.uf} · {c.regiao}</div>
                <div className="font-ui font-bold text-[#0C2A4D]">{c.name}</div>
              </div>
              <RiskBadge risco={c.risco} />
            </div>
            <div className="w-full h-1.5 bg-slate-100 rounded-full overflow-hidden">
              <div className="h-full rounded-full" style={{ width: `${clamp(c.ocupacao, 0, 100)}%`, background: ocupacaoColor(c.ocupacao) }} />
            </div>
            <div className="flex justify-between text-xs font-data text-slate-500">
              <span>{fmt(c.volume)} / {fmt(c.capacidade)}</span>
              <span className="font-semibold" style={{ color: ocupacaoColor(c.ocupacao) }}>{c.ocupacao}%</span>
            </div>
          </button>
        </Card>
      ))}
    </div>
  );
}

/* ============================================================ CENTRO DETALHE */

function CentroDetalhe({ centerId, goto }) {
  const c = CENTERS.find((x) => x.id === centerId) || CENTERS[0];
  const isCritico = c.ocupacao >= 90;
  const timeline = Array.from({ length: 12 }, (_, i) => ({
    hora: `${i * 2}h`,
    entrada: Math.round(c.entradaHora * (0.85 + Math.sin(i / 2) * 0.18)),
    processamento: Math.round(c.processHora * (0.88 + Math.sin(i / 2 + 0.4) * 0.16)),
  }));
  const causas = [
    `Aumento de ${c.previsao.replace("+", "").replace(" amanhã", "")} no volume`,
    "Redução de capacidade em uma rota",
    "Veículo atrasado na chegada",
    "Capacidade de triagem próxima do limite",
  ];

  return (
    <div className="space-y-6">
      <Card className="p-5">
        <div className="flex flex-wrap items-center justify-between gap-4">
          <div>
            <div className="font-data text-[11px] uppercase text-slate-400">{c.uf} · {c.regiao}</div>
            <h1 className="font-ui text-xl font-bold text-[#0C2A4D]">Centro de Tratamento {c.name}</h1>
            <div className="mt-1 flex items-center gap-2">
              <span className={`text-xs font-semibold font-ui ${isCritico ? "text-[#C0342A]" : "text-[#B4790E]"}`}>
                {isCritico ? "⚠️ Crítico" : "⚠️ Atenção"}
              </span>
              <RiskBadge risco={c.risco} />
            </div>
          </div>
          <button onClick={() => goto("recomendacoes")}
            className="px-4 py-2 rounded-md bg-[#1B5FAE] text-white text-sm font-semibold font-ui hover:bg-[#164C8C] flex items-center gap-2">
            <Sparkles size={14} /> Ver recomendações
          </button>
        </div>
      </Card>

      <div className="grid grid-cols-2 md:grid-cols-4 gap-4">
        <KPICard icon={Gauge} label="Ocupação atual" value={c.ocupacao} suffix="%" />
        <KPICard icon={Package} label="Capacidade diária" value={fmt(c.capacidade)} />
        <KPICard icon={Package} label="Volume atual" value={fmt(c.volume)} />
        <KPICard icon={Clock} label="Fila atual" value={fmt(c.fila)} />
        <KPICard icon={TrendingUp} label="Entrada/hora" value={fmt(c.entradaHora)} />
        <KPICard icon={TrendingUp} label="Processamento/hora" value={fmt(c.processHora)} />
        <KPICard icon={Clock} label="Tempo médio de processamento" value={c.tempoMedio} />
        <KPICard icon={TrendingUp} label="Previsão 24h" value={c.previsao} />
      </div>

      <div className="grid xl:grid-cols-2 gap-6">
        <Card className="p-5">
          <SectionTitle eyebrow="Últimas 24 horas" title="Entrada × Processamento" />
          <ResponsiveContainer width="100%" height={220}>
            <LineChart data={timeline}>
              <CartesianGrid strokeDasharray="3 3" stroke="#E7EBF1" vertical={false} />
              <XAxis dataKey="hora" tick={{ fontSize: 10, fill: "#94A3B8" }} axisLine={false} tickLine={false} />
              <YAxis tick={{ fontSize: 10, fill: "#94A3B8" }} axisLine={false} tickLine={false} width={40} />
              <Tooltip contentStyle={{ fontFamily: "IBM Plex Mono", fontSize: 12, borderRadius: 8, border: "1px solid #E2E8F0" }} />
              <Legend wrapperStyle={{ fontSize: 11, fontFamily: "Inter" }} />
              <Line type="monotone" dataKey="entrada" name="Entrada" stroke="#4C8DD9" strokeWidth={2} dot={false} />
              <Line type="monotone" dataKey="processamento" name="Processamento" stroke="#C88A12" strokeWidth={2} dot={false} />
            </LineChart>
          </ResponsiveContainer>
        </Card>

        <Card className="p-5">
          <SectionTitle eyebrow="Principais causas do risco" title="Diagnóstico" />
          <ul className="space-y-2.5">
            {causas.map((causa, i) => (
              <li key={i} className="flex items-start gap-3 text-sm font-ui text-slate-600">
                <span className="font-data text-xs font-bold text-white bg-[#0C2A4D] w-5 h-5 rounded-full flex items-center justify-center shrink-0 mt-0.5">{i + 1}</span>
                {causa}
              </li>
            ))}
          </ul>
        </Card>
      </div>
    </div>
  );
}

/* ============================================================ PREVISÃO */

function PrevisaoDemanda() {
  const [horizonte, setHorizonte] = useState("24h");
  return (
    <div className="space-y-6">
      <Card className="p-5">
        <div className="flex items-center justify-between flex-wrap gap-3 mb-4">
          <SectionTitle eyebrow="Inteligência preditiva" title="Previsão de demanda — Curitiba → São Paulo" />
          <div className="flex gap-1 bg-slate-100 rounded-md p-1">
            {["24h", "7 dias", "30 dias"].map((h) => (
              <button key={h} onClick={() => setHorizonte(h)}
                className={`px-3 py-1.5 rounded text-xs font-semibold font-ui transition-colors ${horizonte === h ? "bg-white shadow-sm text-[#0C2A4D]" : "text-slate-500"}`}>
                {h}
              </button>
            ))}
          </div>
        </div>
        <ResponsiveContainer width="100%" height={260}>
          <LineChart data={FORECAST_ROUTE}>
            <CartesianGrid strokeDasharray="3 3" stroke="#E7EBF1" vertical={false} />
            <XAxis dataKey="periodo" tick={{ fontSize: 11, fill: "#94A3B8" }} axisLine={false} tickLine={false} />
            <YAxis tick={{ fontSize: 10, fill: "#94A3B8" }} axisLine={false} tickLine={false} width={44} tickFormatter={(v) => `${v / 1000}k`} />
            <Tooltip formatter={(v) => v && fmt(v)} contentStyle={{ fontFamily: "IBM Plex Mono", fontSize: 12, borderRadius: 8, border: "1px solid #E2E8F0" }} />
            <Legend wrapperStyle={{ fontSize: 11, fontFamily: "Inter" }} />
            <Line type="monotone" dataKey="historico" name="Demanda histórica" stroke="#94A3B8" strokeWidth={2} dot={{ r: 3 }} connectNulls />
            <Line type="monotone" dataKey="atual" name="Demanda atual" stroke="#1B5FAE" strokeWidth={0} dot={{ r: 5 }} />
            <Line type="monotone" dataKey="previsto" name="Demanda prevista" stroke="#C88A12" strokeWidth={2} strokeDasharray="5 4" dot={{ r: 4 }} connectNulls />
          </LineChart>
        </ResponsiveContainer>
      </Card>

      <div className="grid xl:grid-cols-3 gap-6">
        <Card className="p-5 xl:col-span-2">
          <SectionTitle title="Resumo da previsão" />
          <div className="grid sm:grid-cols-3 gap-4">
            <Metric label="Volume atual" value={fmt(18400)} />
            <Metric label="Previsão amanhã" value={fmt(23100)} />
            <Metric label="Variação prevista" value="+25,5%" color="#C88A12" />
          </div>
          <div className="mt-5 flex items-center gap-3">
            <div className="w-full bg-slate-100 rounded-full h-2">
              <div className="h-2 rounded-full bg-[#17924F]" style={{ width: "92%" }} />
            </div>
            <span className="font-data text-sm font-bold text-[#17924F] whitespace-nowrap">92% confiança</span>
          </div>
          <p className="text-xs text-slate-400 mt-1 font-ui">Confiança da previsão calculada com base no modelo estatístico simulado.</p>
        </Card>

        <Card className="p-5">
          <SectionTitle title="Fatores que influenciaram" />
          <ul className="space-y-2 text-sm font-ui text-slate-600">
            {["Histórico dos últimos 30 dias", "Sazonalidade", "Dia da semana", "Volume atual", "Tendência regional"].map((f) => (
              <li key={f} className="flex items-center gap-2"><span className="w-1.5 h-1.5 rounded-full bg-[#1B5FAE]" />{f}</li>
            ))}
          </ul>
        </Card>
      </div>
    </div>
  );
}

/* ========================================================= RECOMENDAÇÕES */

function Recomendacoes({ recomendacoes, setRecomendacoes, goto }) {
  const [toast, setToast] = useState(null);

  const applyRec = (id) => {
    setRecomendacoes((prev) => prev.map((r) => (r.id === id ? { ...r, status: "aplicada" } : r)));
    setToast({ type: "aplicada", id });
    setTimeout(() => setToast(null), 3200);
  };
  const ignoreRec = (id) => {
    setRecomendacoes((prev) => prev.map((r) => (r.id === id ? { ...r, status: "ignorada" } : r)));
  };

  return (
    <div className="space-y-6">
      {toast && (
        <div className="fixed bottom-6 right-6 z-30 bg-[#0C2A4D] text-white px-5 py-3.5 rounded-lg shadow-xl flex items-center gap-3 font-ui text-sm">
          <CheckCircle2 size={18} className="text-[#4ADE80]" />
          Recomendação #{toast.id} aplicada. Indicadores atualizados.
        </div>
      )}

      <Card className="p-5 flex items-center justify-between flex-wrap gap-3">
        <SectionTitle eyebrow="Motor de otimização" title="Central de Recomendações" />
        <button onClick={() => goto("simulador")}
          className="px-4 py-2 rounded-md border border-slate-200 text-sm font-semibold font-ui text-[#0C2A4D] hover:bg-slate-50 flex items-center gap-2">
          <SlidersHorizontal size={14} /> Abrir simulador
        </button>
      </Card>

      <div className="space-y-4">
        {recomendacoes.map((r) => {
          const alta = r.prioridade === "Alta";
          const applied = r.status === "aplicada";
          const ignored = r.status === "ignorada";
          return (
            <Card key={r.id} className={`p-5 ${applied ? "opacity-70" : ignored ? "opacity-50" : ""}`}>
              <div className="flex items-start justify-between flex-wrap gap-3">
                <div>
                  <div className="flex items-center gap-2 mb-1">
                    <span className="font-data text-xs font-bold text-slate-400">RECOMENDAÇÃO #{r.id}</span>
                    <span className={`inline-flex items-center gap-1 text-xs font-bold font-ui ${alta ? "text-[#C0342A]" : "text-[#B4790E]"}`}>
                      {alta ? "🔴" : "🟡"} {r.prioridade} prioridade
                    </span>
                    {applied && <span className="text-xs font-bold text-[#17924F] flex items-center gap-1"><CheckCircle2 size={13} /> Aplicada</span>}
                    {ignored && <span className="text-xs font-bold text-slate-400 flex items-center gap-1"><XCircle size={13} /> Ignorada</span>}
                  </div>
                  <h3 className="font-ui font-bold text-[#0C2A4D] text-[15px]">{r.titulo}</h3>
                </div>
              </div>

              <div className="grid md:grid-cols-2 gap-4 mt-3 text-sm font-ui">
                <div>
                  <div className="text-xs font-semibold text-slate-400 uppercase mb-0.5">Problema</div>
                  <p className="text-slate-600">{r.problema}</p>
                </div>
                <div>
                  <div className="text-xs font-semibold text-slate-400 uppercase mb-0.5">Ação sugerida</div>
                  <p className="text-slate-600">{r.acao}</p>
                </div>
              </div>

              <div className="grid grid-cols-2 sm:grid-cols-4 gap-3 mt-4">
                <Metric label="Redução congestionamento" value={`${r.impacto.congestionamento}%`} color="#17924F" />
                <Metric label="Redução risco de atraso" value={`${r.impacto.risco}%`} color="#17924F" />
                <Metric label="Custo adicional" value={r.impacto.custo ? fmtR$(r.impacto.custo) : "R$ 0"} />
                <Metric label="Encomendas protegidas" value={fmt(r.impacto.protegidas)} />
              </div>

              {!applied && !ignored && (
                <div className="flex gap-2 mt-4">
                  <button onClick={() => applyRec(r.id)}
                    className="px-4 py-2 rounded-md bg-[#1B5FAE] text-white text-sm font-semibold font-ui hover:bg-[#164C8C] flex items-center gap-1.5">
                    <CheckCircle2 size={14} /> Aplicar recomendação
                  </button>
                  <button onClick={() => goto("simulador")}
                    className="px-4 py-2 rounded-md border border-slate-200 text-sm font-semibold font-ui text-[#0C2A4D] hover:bg-slate-50 flex items-center gap-1.5">
                    <PlayCircle size={14} /> Simular
                  </button>
                  <button onClick={() => ignoreRec(r.id)}
                    className="px-4 py-2 rounded-md text-sm font-semibold font-ui text-slate-400 hover:text-slate-600 flex items-center gap-1.5">
                    <XCircle size={14} /> Ignorar
                  </button>
                </div>
              )}
            </Card>
          );
        })}
      </div>
    </div>
  );
}

/* =============================================================== SIMULADOR */

function Simulador() {
  const [volume, setVolume] = useState(25000);
  const [veiculos, setVeiculos] = useState(18);
  const [capacidade, setCapacidade] = useState(30000);
  const [rotas, setRotas] = useState("Normal");
  const [prioridade, setPrioridade] = useState("Padrão");
  const [resultado, setResultado] = useState(null);

  const rodar = () => {
    const baseOcupacao = clamp(Math.round((volume / capacidade) * 100), 0, 160);
    const atual = {
      ocupacao: baseOcupacao,
      risco: clamp(Math.round(baseOcupacao * 0.55 - veiculos * 0.9 + 8), 3, 98),
      custo: Math.round((volume * 7.28) / 10) * 10,
    };
    const otimOcupacao = Math.round(baseOcupacao * 0.86);
    const otimizado = {
      ocupacao: otimOcupacao,
      risco: clamp(Math.round(otimOcupacao * 0.55 - veiculos * 0.9 - 8), 2, 95),
      custo: Math.round((atual.custo * 0.964) / 10) * 10,
    };
    setResultado({ atual, otimizado, economia: atual.custo - otimizado.custo });
  };

  return (
    <div className="grid xl:grid-cols-3 gap-6">
      <Card className="p-5">
        <SectionTitle eyebrow="Teste de cenários" title="Simulador de Operação" />
        <div className="space-y-4">
          <SliderInput label="Volume de encomendas" value={volume} min={5000} max={60000} step={500} onChange={setVolume} />
          <SliderInput label="Veículos disponíveis" value={veiculos} min={4} max={40} step={1} onChange={setVeiculos} />
          <SliderInput label="Capacidade do centro" value={capacidade} min={10000} max={60000} step={500} onChange={setCapacidade} />
          <label className="block text-xs font-ui">
            <span className="text-slate-500 font-medium">Disponibilidade de rotas</span>
            <select value={rotas} onChange={(e) => setRotas(e.target.value)}
              className="mt-1 w-full border border-slate-200 rounded-md px-3 py-2 text-sm text-slate-700 outline-none focus:ring-2 focus:ring-[#1B5FAE]/30">
              <option>Reduzida</option><option>Normal</option><option>Ampliada</option>
            </select>
          </label>
          <label className="block text-xs font-ui">
            <span className="text-slate-500 font-medium">Prioridade de encomendas</span>
            <select value={prioridade} onChange={(e) => setPrioridade(e.target.value)}
              className="mt-1 w-full border border-slate-200 rounded-md px-3 py-2 text-sm text-slate-700 outline-none focus:ring-2 focus:ring-[#1B5FAE]/30">
              <option>Padrão</option><option>Express prioritário</option><option>Econômico</option>
            </select>
          </label>
          <button onClick={rodar}
            className="w-full py-2.5 rounded-md bg-[#1B5FAE] text-white text-sm font-bold font-ui hover:bg-[#164C8C] flex items-center justify-center gap-2">
            <PlayCircle size={16} /> Executar simulação
          </button>
        </div>
      </Card>

      <div className="xl:col-span-2 space-y-6">
        {!resultado ? (
          <Card className="p-10 flex flex-col items-center justify-center text-center text-slate-400 gap-2 h-full">
            <Gauge size={28} />
            <p className="text-sm font-ui">Ajuste os parâmetros e execute a simulação<br />para comparar cenários.</p>
          </Card>
        ) : (
          <>
            <div className="grid sm:grid-cols-2 gap-6">
              <Card className="p-5 border-l-4 border-l-slate-300">
                <div className="font-data text-[11px] uppercase text-slate-400 mb-3">Cenário atual</div>
                <div className="space-y-3">
                  <ResultRow label="Risco de atraso" value={`${resultado.atual.risco}%`} color="#C0342A" />
                  <ResultRow label="Ocupação" value={`${resultado.atual.ocupacao}%`} color={ocupacaoColor(resultado.atual.ocupacao)} />
                  <ResultRow label="Custo" value={fmtR$(resultado.atual.custo)} />
                </div>
              </Card>
              <Card className="p-5 border-l-4 border-l-[#17924F]">
                <div className="font-data text-[11px] uppercase text-[#17924F] mb-3">Cenário otimizado</div>
                <div className="space-y-3">
                  <ResultRow label="Risco de atraso" value={`${resultado.otimizado.risco}%`} color="#17924F" />
                  <ResultRow label="Ocupação" value={`${resultado.otimizado.ocupacao}%`} color={ocupacaoColor(resultado.otimizado.ocupacao)} />
                  <ResultRow label="Custo" value={fmtR$(resultado.otimizado.custo)} />
                </div>
              </Card>
            </div>
            <Card className="p-5 bg-[#E7F5ED] border-[#17924F]/30 flex items-center justify-between">
              <span className="font-ui font-semibold text-[#0C2A4D]">Economia estimada com o cenário otimizado</span>
              <span className="font-data text-xl font-bold text-[#17924F]">{fmtR$(resultado.economia)}</span>
            </Card>
          </>
        )}
      </div>
    </div>
  );
}

function SliderInput({ label, value, min, max, step, onChange }) {
  return (
    <label className="block text-xs font-ui">
      <div className="flex justify-between mb-1">
        <span className="text-slate-500 font-medium">{label}</span>
        <span className="font-data font-bold text-[#0C2A4D]">{fmt(value)}</span>
      </div>
      <input type="range" min={min} max={max} step={step} value={value}
        onChange={(e) => onChange(Number(e.target.value))}
        className="w-full accent-[#1B5FAE]" />
    </label>
  );
}

function ResultRow({ label, value, color }) {
  return (
    <div className="flex items-center justify-between">
      <span className="text-sm text-slate-500 font-ui">{label}</span>
      <span className="font-data font-bold text-base" style={{ color: color || "#0C2A4D" }}>{value}</span>
    </div>
  );
}

/* ================================================================ RELATÓRIOS */

function Relatorios() {
  const reports = [
    { nome: "Relatório executivo semanal", periodo: "11 – 17 ago 2026", tipo: "PDF" },
    { nome: "Desempenho por centro de tratamento", periodo: "Agosto 2026", tipo: "XLSX" },
    { nome: "Histórico de recomendações aplicadas", periodo: "Últimos 30 dias", tipo: "PDF" },
    { nome: "Ocupação de frota por região", periodo: "Julho 2026", tipo: "XLSX" },
  ];
  return (
    <Card className="p-0 overflow-hidden">
      <div className="px-5 py-4 border-b border-slate-100">
        <h2 className="font-ui text-lg font-bold text-[#0C2A4D]">Relatórios disponíveis</h2>
        <p className="text-xs text-slate-400 font-ui mt-0.5">Dados simulados para fins de demonstração.</p>
      </div>
      {reports.map((r, i) => (
        <div key={i} className="flex items-center justify-between px-5 py-4 border-t border-slate-50 hover:bg-slate-50">
          <div className="flex items-center gap-3">
            <div className="w-9 h-9 rounded-md bg-[#EAF1FA] text-[#1B5FAE] flex items-center justify-center"><FileBarChart size={16} /></div>
            <div>
              <div className="font-ui font-semibold text-sm text-[#0C2A4D]">{r.nome}</div>
              <div className="text-xs text-slate-400 font-data">{r.periodo}</div>
            </div>
          </div>
          <span className="text-xs font-data font-bold text-slate-400 border border-slate-200 rounded px-2 py-1">{r.tipo}</span>
        </div>
      ))}
    </Card>
  );
}

/* =================================================================== APP */

export default function App() {
  const [page, setPage] = useState("visao");
  const [selectedCenter, setSelectedCenter] = useState("CTB");
  const [search, setSearch] = useState("");
  const [notifOpen, setNotifOpen] = useState(false);
  const [recomendacoes, setRecomendacoes] = useState(INITIAL_RECOMENDACOES);
  const [highlightRec, setHighlightRec] = useState(null);

  const pageLabel = NAV_ITEMS.find((n) => n.key === page)?.label ||
    (page === "centroDetalhe" ? "Detalhes do Centro" : page === "simulador" ? "Simulador de Operação" : "");

  const goto = (key, recId) => {
    setPage(key);
    if (recId) setHighlightRec(recId);
    setNotifOpen(false);
  };
  const openCenter = (id) => { setSelectedCenter(id); setPage("centroDetalhe"); };
  const onSearchSubmit = () => {
    const found = CENTERS.find((c) => c.name.toLowerCase().includes(search.trim().toLowerCase()));
    if (found) openCenter(found.id);
  };

  return (
    <div className="flex min-h-screen bg-[#F3F5F8] font-ui" style={{ color: "#0F172A" }}>
      <style>{FONT_STYLE}</style>
      <Sidebar active={page === "centroDetalhe" || page === "simulador" ? "" : page} onNavigate={goto} />
      <div className="flex-1 min-w-0">
        <TopBar pageLabel={pageLabel} search={search} setSearch={setSearch} onSearchSubmit={onSearchSubmit}
          notifOpen={notifOpen} setNotifOpen={setNotifOpen} onOpenRec={(id) => goto("recomendacoes", id)} />
        <main className="p-6 max-w-[1400px] mx-auto">
          {page === "visao" && <Dashboard goto={goto} openCenter={openCenter} />}
          {page === "mapa" && <MapaLogistico onOpenCenter={openCenter} goto={goto} />}
          {page === "gargalos" && <Gargalos onOpenCenter={openCenter} />}
          {page === "transportes" && <Transportes />}
          {page === "centros" && <CentrosTratamento onOpenCenter={openCenter} />}
          {page === "centroDetalhe" && <CentroDetalhe centerId={selectedCenter} goto={goto} />}
          {page === "previsao" && <PrevisaoDemanda />}
          {page === "recomendacoes" && <Recomendacoes recomendacoes={recomendacoes} setRecomendacoes={setRecomendacoes} goto={goto} />}
          {page === "simulador" && <Simulador />}
          {page === "relatorios" && <Relatorios />}
        </main>
      </div>
    </div>
  );
}
https://claude.ai/public/artifacts/84704e60-bcfb-475d-a1f3-fd27cb2503ae
