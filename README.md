import React, { useState, useMemo } from "react";
import {
  LayoutDashboard, Map as MapIcon, AlertTriangle, Truck, Warehouse,
  TrendingUp, Sparkles, FileBarChart, Search, Bell, User, X,
  ArrowUp, ArrowDown, CheckCircle2, PlayCircle, XCircle, ChevronRight,
  Package, Gauge, Clock, ArrowRight, SlidersHorizontal, Radio, MapPin
} from "lucide-react";
import {
  LineChart, Line, BarChart, Bar, AreaChart, Area, XAxis, YAxis,
  CartesianGrid, Tooltip, ResponsiveContainer, Legend, ReferenceLine
} from "recharts";

/* ============================= DESIGN TOKENS =============================
   Base:      #F3F5F8 (frio, técnico)      Painel: #FFFFFF
   Navy 900:  #071B33   Navy 800: #0C2A4D   Navy 600: #16406F
   Azul médio (ação): #1B5FAE      Azul claro (dados): #4C8DD9
   Verde: #17924F   Amarelo: #C88A12   Vermelho: #C0342A
   Tipografia: "Inter" (UI) + "IBM Plex Mono" (dados/métricas, tabular)
============================================================================ */

const FONT_STYLE = `
  @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&family=IBM+Plex+Mono:wght@400;500;600;700&display=swap');
  .font-ui { font-family: 'Inter', system-ui, sans-serif; }
  .font-data { font-family: 'IBM Plex Mono', ui-monospace, monospace; font-variant-numeric: tabular-nums; }
`;

/* ============================================================= MOCK DATA */

const CENTERS = [
  { id: "CTB", name: "Curitiba", uf: "PR", regiao: "Sul", x: 53, y: 79, volume: 14200, capacidade: 15000, ocupacao: 94, risco: "Alto", previsao: "+18% amanhã", entradaHora: 1250, processHora: 980, fila: 2840, tempoMedio: "4h20", tipoGargalo: "Capacidade de triagem" },
  { id: "SP", name: "São Paulo", uf: "SP", regiao: "Sudeste", x: 56, y: 72, volume: 38400, capacidade: 42000, ocupacao: 91, risco: "Alto", previsao: "+12% amanhã", entradaHora: 3620, processHora: 3140, fila: 6900, tempoMedio: "3h40", tipoGargalo: "Volume acima da capacidade" },
  { id: "RJ", name: "Rio de Janeiro", uf: "RJ", regiao: "Sudeste", x: 62, y: 68, volume: 21400, capacidade: 30000, ocupacao: 71, risco: "Médio", previsao: "+5%", entradaHora: 1980, processHora: 1870, fila: 1100, tempoMedio: "2h55", tipoGargalo: "Rota crítica" },
  { id: "BH", name: "Belo Horizonte", uf: "MG", regiao: "Sudeste", x: 60, y: 60, volume: 15800, capacidade: 24000, ocupacao: 66, risco: "Baixo", previsao: "+3%", entradaHora: 1420, processHora: 1390, fila: 420, tempoMedio: "2h10", tipoGargalo: "Veículo insuficiente" },
  { id: "BSB", name: "Brasília", uf: "DF", regiao: "Centro-Oeste", x: 55, y: 50, volume: 7200, capacidade: 11800, ocupacao: 61, risco: "Baixo", previsao: "-2%", entradaHora: 690, processHora: 705, fila: 180, tempoMedio: "1h50", tipoGargalo: "Sem gargalo identificado" },
  { id: "POA", name: "Porto Alegre", uf: "RS", regiao: "Sul", x: 50, y: 91, volume: 8900, capacidade: 12000, ocupacao: 76, risco: "Médio", previsao: "+4%", entradaHora: 810, processHora: 770, fila: 640, tempoMedio: "2h35", tipoGargalo: "Rota crítica" },
  { id: "SSA", name: "Salvador", uf: "BA", regiao: "Nordeste", x: 72, y: 43, volume: 9600, capacidade: 16000, ocupacao: 60, risco: "Baixo", previsao: "+1%", entradaHora: 880, processHora: 900, fila: 210, tempoMedio: "1h55", tipoGargalo: "Sem gargalo identificado" },
  { id: "REC", name: "Recife", uf: "PE", regiao: "Nordeste", x: 85, y: 29, volume: 7100, capacidade: 11000, ocupacao: 65, risco: "Baixo", previsao: "+2%", entradaHora: 640, processHora: 620, fila: 260, tempoMedio: "2h05", tipoGargalo: "Sem gargalo identificado" },
  { id: "FOR", name: "Fortaleza", uf: "CE", regiao: "Nordeste", x: 77, y: 22, volume: 6800, capacidade: 10500, ocupacao: 65, risco: "Baixo", previsao: "+3%", entradaHora: 610, processHora: 598, fila: 240, tempoMedio: "2h00", tipoGargalo: "Sem gargalo identificado" },
  { id: "MAO", name: "Manaus", uf: "AM", regiao: "Norte", x: 25, y: 16, volume: 4200, capacidade: 8000, ocupacao: 52, risco: "Baixo", previsao: "0%", entradaHora: 390, processHora: 402, fila: 90, tempoMedio: "1h40", tipoGargalo: "Sem gargalo identificado" },
];

const ROUTES = [
  ["CTB", "SP", 18400], ["SP", "RJ", 12000], ["SP", "BH", 9000], ["SP", "BSB", 7000],
  ["BSB", "SSA", 5000], ["BSB", "REC", 4000], ["SSA", "REC", 3000], ["REC", "FOR", 2500],
  ["BSB", "MAO", 2000], ["POA", "CTB", 9800], ["BH", "RJ", 5200],
];

const VEHICLES_BASE = [
  { id: "TR-2048", origem: "Curitiba", destino: "São Paulo", ocupacao: 96, saida: "14:30", chegada: "22:10", status: "Atenção" },
  { id: "TR-3012", origem: "Curitiba", destino: "Porto Alegre", ocupacao: 78, saida: "15:00", chegada: "21:30", status: "Normal" },
  { id: "TR-1822", origem: "São Paulo", destino: "Brasília", ocupacao: 98, saida: "16:20", chegada: "02:40", status: "Crítico" },
  { id: "TR-4410", origem: "Rio de Janeiro", destino: "Belo Horizonte", ocupacao: 82, saida: "13:10", chegada: "18:45", status: "Normal" },
  { id: "TR-5501", origem: "Salvador", destino: "Recife", ocupacao: 70, saida: "09:00", chegada: "15:20", status: "Normal" },
  { id: "TR-6120", origem: "Brasília", destino: "Manaus", ocupacao: 91, saida: "11:40", chegada: "23:10", status: "Atenção" },
  { id: "TR-2277", origem: "São Paulo", destino: "Curitiba", ocupacao: 64, saida: "08:15", chegada: "14:00", status: "Normal" },
];

const DEMANDA_30D = Array.from({ length: 30 }, (_, i) => {
  const dia = i + 1;
  const base = 88 + Math.sin(i / 3.4) * 2.6 + (i / 30) * 3.4;
  return { dia: `${dia}/07`, prazo: Math.round(base * 10) / 10 };
});

const CAPACIDADE_REDE = [
  { categoria: "Rede Nacional", disponivel: 640000, utilizada: 552000, projetada: 618000 },
];

const FORECAST_ROUTE = [
  { periodo: "-6d", historico: 15200, atual: null, previsto: null },
  { periodo: "-5d", historico: 15800, atual: null, previsto: null },
  { periodo: "-4d", historico: 16400, atual: null, previsto: null },
  { periodo: "-3d", historico: 17100, atual: null, previsto: null },
  { periodo: "-2d", historico: 17600, atual: null, previsto: null },
  { periodo: "-1d", historico: 18000, atual: null, previsto: null },
  { periodo: "Hoje", historico: null, atual: 18400, previsto: 18400 },
  { periodo: "+1d", historico: null, atual: null, previsto: 23100 },
];

const INITIAL_RECOMENDACOES = [
  {
    id: "001", prioridade: "Alta",
    titulo: "Redistribuir carga do Centro de Tratamento Curitiba",
    problema: "O centro deve atingir 98% de ocupação nas próximas 8 horas.",
    acao: "Transferir 3.200 encomendas para o centro de São Paulo utilizando a rota alternativa.",
    impacto: { congestionamento: 31, risco: 18, custo: 8450, protegidas: 2700 },
    status: "pendente",
  },
  {
    id: "002", prioridade: "Média",
    titulo: "Antecipar saída do veículo TR-2048 em 40 minutos",
    problema: "A rota Curitiba → São Paulo apresenta aumento previsto de 14% no volume.",
    acao: "Antecipar horário de saída para reduzir concentração de carga no pico das 18h.",
    impacto: { congestionamento: 9, risco: 12, custo: 0, protegidas: 320 },
    status: "pendente",
  },
  {
    id: "003", prioridade: "Média",
    titulo: "Reforçar triagem no Centro de São Paulo",
    problema: "Aumento de 22% no volume previsto para amanhã eleva o risco de fila acima da capacidade de triagem.",
    acao: "Alocar equipe extra no turno da tarde e abrir 2 esteiras adicionais.",
    impacto: { congestionamento: 15, risco: 14, custo: 5200, protegidas: 1900 },
    status: "pendente",
  },
];

const ALERTAS = [
  { nivel: "critico", titulo: "Centro de Tratamento Curitiba", texto: "Risco de congestionamento em 8 horas. Volume previsto: 14.200 encomendas. Capacidade estimada: 10.000.", risco: 87, recId: "001" },
  { nivel: "atencao", titulo: "Rota Curitiba → São Paulo", texto: "Ocupação prevista acima de 95%. Recomendação disponível.", recId: "002" },
  { nivel: "atencao", titulo: "Centro de Tratamento São Paulo", texto: "Aumento de 22% no volume previsto para amanhã.", recId: "003" },
];

const NAV_ITEMS = [
  { key: "visao", label: "Visão Geral", icon: LayoutDashboard },
  { key: "mapa", label: "Mapa Logístico", icon: MapIcon },
  { key: "gargalos", label: "Gargalos", icon: AlertTriangle },
  { key: "transportes", label: "Transportes", icon: Truck },
  { key: "centros", label: "Centros de Tratamento", icon: Warehouse },
  { key: "previsao", label: "Previsão de Demanda", icon: TrendingUp },
  { key: "recomendacoes", label: "Recomendações", icon: Sparkles },
  { key: "relatorios", label: "Relatórios", icon: FileBarChart },
];

/* =============================================================== HELPERS */

const fmt = (n) => n.toLocaleString("pt-BR");
const fmtR$ = (n) => n.toLocaleString("pt-BR", { style: "currency", currency: "BRL", maximumFractionDigits: 0 });
const clamp = (n, a, b) => Math.max(a, Math.min(b, n));

function riscoColor(risco) {
  if (risco === "Alto") return { text: "text-[#C0342A]", bg: "bg-[#C0342A]", soft: "bg-[#FBEAE8]", border: "border-[#C0342A]" };
  if (risco === "Médio") return { text: "text-[#B4790E]", bg: "bg-[#C88A12]", soft: "bg-[#FBF1DD]", border: "border-[#C88A12]" };
  return { text: "text-[#17924F]", bg: "bg-[#17924F]", soft: "bg-[#E7F5ED]", border: "border-[#17924F]" };
}

function ocupacaoColor(oc) {
  if (oc >= 90) return "#C0342A";
  if (oc >= 70) return "#C88A12";
  return "#17924F";
}

function RiskBadge({ risco }) {
  const c = riscoColor(risco);
  return (
    <span className={`inline-flex items-center gap-1.5 px-2 py-0.5 rounded text-xs font-semibold font-ui ${c.soft} ${c.text}`}>
      <span className={`w-1.5 h-1.5 rounded-full ${c.bg}`} />
      {risco}
    </span>
  );
}

function StatusBadge({ status }) {
  const map = {
    Normal: { text: "text-[#17924F]", soft: "bg-[#E7F5ED]" },
    Atenção: { text: "text-[#B4790E]", soft: "bg-[#FBF1DD]" },
    Crítico: { text: "text-[#C0342A]", soft: "bg-[#FBEAE8]" },
  };
  const c = map[status] || map.Normal;
  return <span className={`px-2 py-0.5 rounded text-xs font-semibold font-ui ${c.soft} ${c.text}`}>{status}</span>;
}

function Delta({ value, invert = false }) {
  const up = value >= 0;
  const good = invert ? !up : up;
  return (
    <span className={`inline-flex items-center gap-0.5 text-xs font-semibold font-data ${good ? "text-[#17924F]" : "text-[#C0342A]"}`}>
      {up ? <ArrowUp size={12} /> : <ArrowDown size={12} />}
      {Math.abs(value).toFixed(1)}%
    </span>
  );
}

function Card({ children, className = "" }) {
  return <div className={`bg-white rounded-lg border border-slate-200 shadow-sm ${className}`}>{children}</div>;
}

function SectionTitle({ eyebrow, title, action }) {
  return (
    <div className="flex items-end justify-between mb-4">
      <div>
        {eyebrow && <div className="font-data text-[11px] tracking-widest uppercase text-[#4C8DD9] mb-1">{eyebrow}</div>}
        <h2 className="font-ui text-lg font-bold text-[#0C2A4D]">{title}</h2>
      </div>
      {action}
    </div>
  );
}

/* ================================================================ LAYOUT */

function Sidebar({ active, onNavigate }) {
  return (
    <aside className="w-64 shrink-0 h-screen sticky top-0 flex flex-col text-white font-ui"
      style={{ background: "linear-gradient(180deg,#071B33 0%,#0C2A4D 100%)" }}>
      <div className="px-5 py-5 border-b border-white/10">
        <div className="flex items-center gap-2.5">
          <div className="w-9 h-9 rounded-md bg-[#1B5FAE] flex items-center justify-center shrink-0">
            <Radio size={18} className="text-white" />
          </div>
          <div className="leading-tight">
            <div className="font-bold text-[15px]">Correios</div>
            <div className="text-[13px] text-[#8FB4E0] tracking-wide">LogiSense</div>
          </div>
        </div>
      </div>
      <nav className="flex-1 py-3 px-2 space-y-0.5 overflow-y-auto">
        {NAV_ITEMS.map((item) => {
          const Icon = item.icon;
          const isActive = active === item.key;
          return (
            <button
              key={item.key}
              onClick={() => onNavigate(item.key)}
              className={`w-full flex items-center gap-3 px-3 py-2.5 rounded-md text-[13.5px] font-medium transition-colors
                ${isActive ? "bg-[#1B5FAE] text-white" : "text-[#B8CCE3] hover:bg-white/5 hover:text-white"}`}
            >
              <Icon size={17} className="shrink-0" />
              <span className="text-left">{item.label}</span>
              {isActive && <ChevronRight size={14} className="ml-auto opacity-70" />}
            </button>
          );
        })}
      </nav>
      <div className="px-4 py-4 border-t border-white/10 text-[11px] text-[#6E8BAE] font-data leading-relaxed">
        Protótipo conceitual · dados 100% simulados<br />Sem integração com sistemas reais dos Correios
      </div>
    </aside>
  );
}

function TopBar({ pageLabel, search, setSearch, onSearchSubmit, notifOpen, setNotifOpen, onOpenRec }) {
  return (
    <div className="sticky top-0 z-20 bg-white border-b border-slate-200 px-6 py-3 flex items-center gap-4 font-ui">
      <div className="min-w-[180px]">
        <div className="text-[11px] text-slate-400 font-data uppercase tracking-wide">Correios LogiSense</div>
        <div className="text-[15px] font-bold text-[#0C2A4D]">{pageLabel}</div>
      </div>
      <div className="flex items-center gap-1.5 px-2.5 py-1 rounded-full bg-[#E7F5ED] text-[#17924F] text-xs font-semibold font-data">
        <span className="relative flex h-2 w-2">
          <span className="animate-ping absolute inline-flex h-full w-full rounded-full bg-[#17924F] opacity-60" />
          <span className="relative inline-flex rounded-full h-2 w-2 bg-[#17924F]" />
        </span>
        Operação atualizada há 2 min
      </div>

      <div className="flex-1 max-w-md relative ml-4">
        <Search size={16} className="absolute left-3 top-1/2 -translate-y-1/2 text-slate-400" />
        <input
          value={search}
          onChange={(e) => setSearch(e.target.value)}
          onKeyDown={(e) => e.key === "Enter" && onSearchSubmit()}
          placeholder="Buscar centro, rota ou veículo..."
          className="w-full pl-9 pr-3 py-2 rounded-md bg-slate-100 text-sm text-slate-700 outline-none focus:ring-2 focus:ring-[#1B5FAE]/40 focus:bg-white border border-transparent focus:border-[#1B5FAE]/30"
        />
      </div>

      <div className="ml-auto flex items-center gap-3 relative">
        <button
          onClick={() => setNotifOpen((v) => !v)}
          className="relative w-9 h-9 rounded-md hover:bg-slate-100 flex items-center justify-center text-slate-500"
        >
          <Bell size={18} />
          <span className="absolute top-1.5 right-1.5 w-2 h-2 rounded-full bg-[#C0342A]" />
        </button>
        {notifOpen && (
          <div className="absolute right-24 top-11 w-80 bg-white border border-slate-200 rounded-lg shadow-lg overflow-hidden">
            <div className="px-4 py-2.5 border-b border-slate-100 text-xs font-bold text-slate-500 uppercase tracking-wide font-ui">Notificações</div>
            {ALERTAS.map((a, i) => (
              <button key={i} onClick={() => { onOpenRec(a.recId); setNotifOpen(false); }}
                className="w-full text-left px-4 py-3 border-b border-slate-50 last:border-0 hover:bg-slate-50 flex gap-2.5">
                <span className={`w-2 h-2 mt-1.5 rounded-full shrink-0 ${a.nivel === "critico" ? "bg-[#C0342A]" : "bg-[#C88A12]"}`} />
                <div>
                  <div className="text-[13px] font-semibold text-[#0C2A4D]">{a.titulo}</div>
                  <div className="text-xs text-slate-500 mt-0.5 line-clamp-2">{a.texto}</div>
                </div>
              </button>
            ))}
          </div>
        )}
        <div className="w-px h-6 bg-slate-200" />
        <div className="flex items-center gap-2">
          <div className="w-8 h-8 rounded-full bg-[#1B5FAE] flex items-center justify-center text-white">
            <User size={15} />
          </div>
          <div className="leading-tight hidden sm:block">
            <div className="text-[13px] font-semibold text-[#0C2A4D]">Gestor Logístico</div>
            <div className="text-[11px] text-slate-400">Perfil operacional</div>
          </div>
        </div>
      </div>
    </div>
  );
}

/* ================================================================ KPI CARD */

function KPICard({ icon: Icon, label, value, delta, invert, suffix = "" }) {
  return (
    <Card className="p-4 flex flex-col gap-2.5">
      <div className="flex items-center justify-between">
        <span className="text-xs font-semibold text-slate-500 font-ui">{label}</span>
        <Icon size={16} className="text-[#4C8DD9]" />
      </div>
      <div className="flex items-end justify-between">
        <span className="font-data text-2xl font-bold text-[#0C2A4D]">{value}{suffix}</span>
        {delta !== undefined && <Delta value={delta} invert={invert} />}
      </div>
    </Card>
  );
}

/* ============================================================== DASHBOARD */

function Dashboard({ goto, openCenter }) {
  return (
    <div className="space-y-6">
      <div className="grid grid-cols-2 md:grid-cols-3 xl:grid-cols-6 gap-4">
        <KPICard icon={Package} label="Em processamento" value={fmt(184320)} delta={1.6} />
        <KPICard icon={Truck} label="Em trânsito" value={fmt(96540)} delta={0.8} />
        <KPICard icon={Gauge} label="Entregas no prazo" value="91,8" suffix="%" delta={2.4} />
        <KPICard icon={AlertTriangle} label="Centros em risco" value="7" delta={-12.5} invert />
        <KPICard icon={Truck} label="Veículos em operação" value={fmt(1284)} delta={0.4} />
        <KPICard icon={Gauge} label="Ocupação média veículos" value="87" suffix="%" delta={1.1} />
      </div>

      <div className="grid grid-cols-1 xl:grid-cols-3 gap-6">
        <Card className="p-5 xl:col-span-2">
          <SectionTitle eyebrow="Série histórica · 30 dias" title="Entregas dentro do prazo" />
          <ResponsiveContainer width="100%" height={230}>
            <AreaChart data={DEMANDA_30D}>
              <defs>
                <linearGradient id="prazoFill" x1="0" y1="0" x2="0" y2="1">
                  <stop offset="0%" stopColor="#1B5FAE" stopOpacity={0.25} />
                  <stop offset="100%" stopColor="#1B5FAE" stopOpacity={0} />
                </linearGradient>
              </defs>
              <CartesianGrid strokeDasharray="3 3" stroke="#E7EBF1" vertical={false} />
              <XAxis dataKey="dia" tick={{ fontSize: 10, fill: "#94A3B8" }} interval={4} axisLine={false} tickLine={false} />
              <YAxis domain={[80, 100]} tick={{ fontSize: 10, fill: "#94A3B8" }} axisLine={false} tickLine={false} unit="%" width={38} />
              <Tooltip formatter={(v) => [`${v}%`, "No prazo"]} contentStyle={{ fontFamily: "IBM Plex Mono", fontSize: 12, borderRadius: 8, border: "1px solid #E2E8F0" }} />
              <Area type="monotone" dataKey="prazo" stroke="#1B5FAE" strokeWidth={2} fill="url(#prazoFill)" />
            </AreaChart>
          </ResponsiveContainer>
        </Card>

        <Card className="p-5">
          <SectionTitle eyebrow="Capacidade" title="Rede nacional" />
          <ResponsiveContainer width="100%" height={230}>
            <BarChart data={CAPACIDADE_REDE} barGap={6}>
              <CartesianGrid strokeDasharray="3 3" stroke="#E7EBF1" vertical={false} />
              <XAxis dataKey="categoria" tick={{ fontSize: 10, fill: "#94A3B8" }} axisLine={false} tickLine={false} />
              <YAxis tick={{ fontSize: 10, fill: "#94A3B8" }} axisLine={false} tickLine={false} tickFormatter={(v) => `${v / 1000}k`} width={36} />
              <Tooltip formatter={(v) => fmt(v)} contentStyle={{ fontFamily: "IBM Plex Mono", fontSize: 12, borderRadius: 8, border: "1px solid #E2E8F0" }} />
              <Legend wrapperStyle={{ fontSize: 11, fontFamily: "Inter" }} />
              <Bar dataKey="disponivel" name="Disponível" fill="#CBD9EA" radius={[4, 4, 0, 0]} />
              <Bar dataKey="utilizada" name="Utilizada" fill="#1B5FAE" radius={[4, 4, 0, 0]} />
              <Bar dataKey="projetada" name="Projetada" fill="#C88A12" radius={[4, 4, 0, 0]} />
            </BarChart>
          </ResponsiveContainer>
        </Card>
      </div>

      <Card className="p-5">
        <SectionTitle eyebrow="Monitoramento em tempo real" title="Alertas críticos" />
        <div className="grid md:grid-cols-3 gap-4">
          {ALERTAS.map((a, i) => {
            const critico = a.nivel === "critico";
            return (
              <div key={i} className={`rounded-md border-l-4 ${critico ? "border-[#C0342A] bg-[#FBEAE8]/40" : "border-[#C88A12] bg-[#FBF1DD]/40"} border border-slate-200 p-4 flex flex-col gap-2`}>
                <div className="flex items-center gap-2">
                  <span className={`w-2 h-2 rounded-full ${critico ? "bg-[#C0342A]" : "bg-[#C88A12]"}`} />
                  <span className="font-ui font-bold text-sm text-[#0C2A4D]">{a.titulo}</span>
                </div>
                <p className="text-[13px] text-slate-600 leading-snug">{a.texto}</p>
                {a.risco && <div className="font-data text-xs text-slate-500">Risco estimado: <span className="font-bold text-[#C0342A]">{a.risco}%</span></div>}
                <button onClick={() => goto("recomendacoes", a.recId)}
                  className="mt-1 self-start text-xs font-semibold text-[#1B5FAE] hover:underline flex items-center gap-1">
                  Ver solução <ArrowRight size={12} />
                </button>
              </div>
            );
          })}
        </div>
      </Card>
    </div>
  );
}

/* ============================================================ MAPA LOGÍSTICO */

function MapaLogistico({ onOpenCenter, goto }) {
  const [selected, setSelected] = useState(null);
  const center = CENTERS.find((c) => c.id === selected);

  return (
    <div className="grid grid-cols-1 xl:grid-cols-3 gap-6">
      <Card className="p-5 xl:col-span-2">
        <SectionTitle eyebrow="Malha nacional · tempo real" title="Mapa Logístico" />
        <div className="relative rounded-md overflow-hidden border border-slate-200"
          style={{
            background: "radial-gradient(circle at 30% 20%, #0F3054 0%, #071B33 70%)",
            height: 520,
            backgroundImage:
              "linear-gradient(rgba(255,255,255,.05) 1px, transparent 1px), linear-gradient(90deg, rgba(255,255,255,.05) 1px, transparent 1px)",
            backgroundSize: "24px 24px",
          }}
        >
          <svg viewBox="0 0 100 100" className="absolute inset-0 w-full h-full">
            {ROUTES.map(([a, b, vol], i) => {
              const ca = CENTERS.find((c) => c.id === a);
              const cb = CENTERS.find((c) => c.id === b);
              const w = clamp(vol / 3500, 0.4, 3.2);
              return (
                <line key={i} x1={ca.x} y1={ca.y} x2={cb.x} y2={cb.y}
                  stroke="#4C8DD9" strokeOpacity={0.45} strokeWidth={w / 4} />
              );
            })}
            {CENTERS.map((c) => (
              <g key={c.id} onClick={() => setSelected(c.id)} className="cursor-pointer">
                <circle cx={c.x} cy={c.y} r={2.6} fill={ocupacaoColor(c.ocupacao)} fillOpacity={0.25}>
                  <animate attributeName="r" values="2.6;4.2;2.6" dur="2.4s" repeatCount="indefinite" />
                </circle>
                <circle cx={c.x} cy={c.y} r={1.5} fill={ocupacaoColor(c.ocupacao)} stroke="#071B33" strokeWidth={0.3} />
              </g>
            ))}
          </svg>
          {CENTERS.map((c) => (
            <button key={c.id} onClick={() => setSelected(c.id)}
              style={{ left: `${c.x}%`, top: `${c.y}%` }}
              className="absolute -translate-x-1/2 -translate-y-full mt-[-6px] text-[10.5px] font-data font-semibold text-white/90 bg-black/30 px-1.5 py-0.5 rounded backdrop-blur-sm whitespace-nowrap hover:bg-black/60 transition-colors">
              {c.name}
            </button>
          ))}
          <div className="absolute bottom-3 left-3 flex items-center gap-3 text-[11px] font-data text-white/80 bg-black/30 rounded px-3 py-1.5">
            <span className="flex items-center gap-1"><span className="w-2 h-2 rounded-full bg-[#17924F]" />&lt;70%</span>
            <span className="flex items-center gap-1"><span className="w-2 h-2 rounded-full bg-[#C88A12]" />70–90%</span>
            <span className="flex items-center gap-1"><span className="w-2 h-2 rounded-full bg-[#C0342A]" />&gt;90%</span>
          </div>
        </div>
      </Card>

      <Card className="p-5">
        {!center ? (
          <div className="h-full flex flex-col items-center justify-center text-center text-slate-400 py-16 gap-2">
            <MapPin size={26} />
            <p className="text-sm font-ui">Selecione um centro no mapa<br />para ver detalhes operacionais.</p>
          </div>
        ) : (
          <div className="space-y-4">
            <div className="flex items-center justify-between">
              <div>
                <div className="font-data text-[11px] uppercase text-slate-400">{center.uf} · {center.regiao}</div>
                <h3 className="font-ui font-bold text-lg text-[#0C2A4D]">{center.name}</h3>
              </div>
              <button onClick={() => setSelected(null)} className="text-slate-400 hover:text-slate-600"><X size={18} /></button>
            </div>
            <RiskBadge risco={center.risco} />
            <div className="grid grid-cols-2 gap-3 font-data text-sm">
              <Metric label="Volume atual" value={fmt(center.volume)} />
              <Metric label="Capacidade" value={fmt(center.capacidade)} />
              <Metric label="Ocupação" value={`${center.ocupacao}%`} color={ocupacaoColor(center.ocupacao)} />
              <Metric label="Recebidas/h" value={fmt(center.entradaHora)} />
              <Metric label="Processadas/h" value={fmt(center.processHora)} />
              <Metric label="Tempo médio" value={center.tempoMedio} />
            </div>
            <div className="pt-2 border-t border-slate-100">
              <div className="text-xs text-slate-500 font-ui mb-1">Previsão para as próximas 24h</div>
              <div className="text-sm font-data font-semibold text-[#0C2A4D]">{center.previsao} em relação a hoje</div>
            </div>
            <div className="flex flex-col gap-2 pt-2">
              <button onClick={() => goto("simulador")}
                className="w-full py-2 rounded-md bg-[#1B5FAE] text-white text-sm font-semibold font-ui hover:bg-[#164C8C] transition-colors flex items-center justify-center gap-2">
                <SlidersHorizontal size={14} /> Simular redistribuição
              </button>
              <button onClick={() => onOpenCenter(center.id)}
                className="w-full py-2 rounded-md border border-slate-200 text-[#0C2A4D] text-sm font-semibold font-ui hover:bg-slate-50 transition-colors">
                Ver detalhes completos
              </button>
            </div>
          </div>
        )}
      </Card>
    </div>
  );
}

function Metric({ label, value, color }) {
  return (
    <div className="bg-slate-50 rounded-md px-3 py-2">
      <div className="text-[10.5px] font-ui text-slate-400">{label}</div>
      <div className="font-bold" style={{ color: color || "#0C2A4D" }}>{value}</div>
    </div>
  );
}

/* =================================================================== GARGALOS */

function Gargalos({ onOpenCenter }) {
  const [regiao, setRegiao] = useState("Todas");
  const [risco, setRisco] = useState("Todos");
  const [tipo, setTipo] = useState("Todos");

  const regioes = ["Todas", ...new Set(CENTERS.map((c) => c.regiao))];
  const tipos = ["Todos", ...new Set(CENTERS.map((c) => c.tipoGargalo))];

  const filtered = CENTERS.filter((c) =>
    (regiao === "Todas" || c.regiao === regiao) &&
    (risco === "Todos" || c.risco === risco) &&
    (tipo === "Todos" || c.tipoGargalo === tipo)
  ).sort((a, b) => b.ocupacao - a.ocupacao);

  return (
    <div className="space-y-6">
      <Card className="p-4 flex flex-wrap gap-3 items-center">
        <SelectFilter label="Região" value={regiao} onChange={setRegiao} options={regioes} />
        <SelectFilter label="Nível de risco" value={risco} onChange={setRisco} options={["Todos", "Alto", "Médio", "Baixo"]} />
        <SelectFilter label="Tipo de gargalo" value={tipo} onChange={setTipo} options={tipos} />
        <SelectFilter label="Período" value="Próximas 24h" onChange={() => {}} options={["Próximas 24h", "Próximos 7 dias", "Próximos 30 dias"]} />
      </Card>

      <Card className="p-5">
        <SectionTitle eyebrow={`${filtered.length} centro(s)`} title="Gargalos previstos nas próximas 24 horas" />
        <ResponsiveContainer width="100%" height={200}>
          <BarChart data={filtered} layout="vertical" margin={{ left: 10 }}>
            <CartesianGrid strokeDasharray="3 3" stroke="#E7EBF1" horizontal={false} />
            <XAxis type="number" domain={[0, 100]} tick={{ fontSize: 10, fill: "#94A3B8" }} unit="%" axisLine={false} tickLine={false} />
            <YAxis type="category" dataKey="name" width={90} tick={{ fontSize: 11, fill: "#334155" }} axisLine={false} tickLine={false} />
            <Tooltip formatter={(v) => [`${v}%`, "Ocupação"]} contentStyle={{ fontFamily: "IBM Plex Mono", fontSize: 12, borderRadius: 8, border: "1px solid #E2E8F0" }} />
            <ReferenceLine x={90} stroke="#C0342A" strokeDasharray="4 4" />
            <Bar dataKey="ocupacao" radius={[0, 4, 4, 0]}>
              {filtered.map((c, i) => <Bar key={i} fill={ocupacaoColor(c.ocupacao)} />)}
            </Bar>
          </BarChart>
        </ResponsiveContainer>
      </Card>

      <Card className="p-0 overflow-hidden">
        <div className="px-5 py-4 border-b border-slate-100">
          <h2 className="font-ui text-lg font-bold text-[#0C2A4D]">Detalhamento por centro</h2>
        </div>
        <table className="w-full text-sm font-ui">
          <thead>
            <tr className="bg-slate-50 text-slate-500 text-xs uppercase tracking-wide">
              <th className="text-left px-5 py-2.5 font-semibold">Centro</th>
              <th className="text-left px-3 py-2.5 font-semibold">Ocupação</th>
              <th className="text-left px-3 py-2.5 font-semibold">Volume</th>
              <th className="text-left px-3 py-2.5 font-semibold">Capacidade</th>
              <th className="text-left px-3 py-2.5 font-semibold">Risco</th>
              <th className="text-left px-3 py-2.5 font-semibold">Previsão</th>
              <th className="px-3 py-2.5" />
            </tr>
          </thead>
          <tbody>
            {filtered.map((c) => (
              <tr key={c.id} className="border-t border-slate-100 hover:bg-slate-50 cursor-pointer" onClick={() => onOpenCenter(c.id)}>
                <td className="px-5 py-3 font-semibold text-[#0C2A4D]">{c.name} <span className="text-slate-400 font-normal">· {c.uf}</span></td>
                <td className="px-3 py-3 font-data font-semibold" style={{ color: ocupacaoColor(c.ocupacao) }}>{c.ocupacao}%</td>
                <td className="px-3 py-3 font-data">{fmt(c.volume)}</td>
                <td className="px-3 py-3 font-data">{fmt(c.capacidade)}</td>
                <td className="px-3 py-3"><RiskBadge risco={c.risco} /></td>
                <td className="px-3 py-3 font-data text-slate-600">{c.previsao}</td>
                <td className="px-3 py-3 text-slate-300"><ChevronRight size={16} /></td>
              </tr>
            ))}
          </tbody>
        </table>
      </Card>
    </div>
  );
}

function SelectFilter({ label, value, onChange, options }) {
  return (
    <label className="flex items-center gap-2 text-xs font-ui">
      <span className="text-slate-500 font-medium">{label}</span>
      <select value={value} onChange={(e) => onChange(e.target.value)}
        className="border border-slate-200 rounded-md px-2.5 py-1.5 text-slate-700 bg-white outline-none focus:ring-2 focus:ring-[#1B5FAE]/30 text-xs">
        {options.map((o) => <option key={o} value={o}>{o}</option>)}
      </select>
    </label>
  );
}

/* ================================================================ TRANSPORTES */

function Transportes() {
  const ativos = VEHICLES_BASE.length + 1277;
  return (
    <div className="space-y-6">
      <div className="grid grid-cols-2 md:grid-cols-5 gap-4">
        <KPICard icon={Truck} label="Veículos ativos" value={fmt(1284)} />
        <KPICard icon={Truck} label="Disponíveis" value={fmt(142)} />
        <KPICard icon={SlidersHorizontal} label="Em manutenção" value={fmt(38)} />
        <KPICard icon={Gauge} label="Ocupação média" value="87" suffix="%" />
        <KPICard icon={AlertTriangle} label="Rotas críticas" value="4" />
      </div>
      <Card className="p-0 overflow-hidden">
        <div className="px-5 py-4 border-b border-slate-100 flex items-center justify-between">
          <h2 className="font-ui text-lg font-bold text-[#0C2A4D]">Frota em operação</h2>
          <span className="text-xs text-slate-400 font-data">{VEHICLES_BASE.length} veículos monitorados</span>
        </div>
        <table className="w-full text-sm font-ui">
          <thead>
            <tr className="bg-slate-50 text-slate-500 text-xs uppercase tracking-wide">
              <th className="text-left px-5 py-2.5 font-semibold">Veículo</th>
              <th className="text-left px-3 py-2.5 font-semibold">Origem</th>
              <th className="text-left px-3 py-2.5 font-semibold">Destino</th>
              <th className="text-left px-3 py-2.5 font-semibold">Ocupação</th>
              <th className="text-left px-3 py-2.5 font-semibold">Saída</th>
              <th className="text-left px-3 py-2.5 font-semibold">Chegada prevista</th>
              <th className="text-left px-3 py-2.5 font-semibold">Status</th>
            </tr>
          </thead>
          <tbody>
            {VEHICLES_BASE.map((v) => (
              <tr key={v.id} className="border-t border-slate-100 hover:bg-slate-50">
                <td className="px-5 py-3 font-data font-semibold text-[#0C2A4D]">{v.id}</td>
                <td className="px-3 py-3">{v.origem}</td>
                <td className="px-3 py-3">{v.destino}</td>
                <td className="px-3 py-3 font-data font-semibold" style={{ color: ocupacaoColor(v.ocupacao) }}>{v.ocupacao}%</td>
                <td className="px-3 py-3 font-data">{v.saida}</td>
                <td className="px-3 py-3 font-data">{v.chegada}</td>
                <td className="px-3 py-3"><StatusBadge status={v.status} /></td>
              </tr>
            ))}
          </tbody>
        </table>
      </Card>
    </div>
  );
}

/* =========================================================== CENTROS LISTA */

function CentrosTratamento({ onOpenCenter }) {
  return (
    <div className="grid sm:grid-cols-2 xl:grid-cols-3 gap-4">
      {CENTERS.map((c) => (
        <Card key={c.id} className="p-4 cursor-pointer hover:shadow-md transition-shadow" >
          <button onClick={() => onOpenCenter(c.id)} className="w-full text-left space-y-3">
            <div className="flex items-start justify-between">
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
