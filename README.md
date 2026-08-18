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
              
              
              
              
              
              
              
              
              
              
              
