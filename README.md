# CRE-Deal-Analyzer
A web application that parses commercial real estate offering memorandums to evaluate profitability based on NOI, gross profit, and expenses.
import React, { useEffect, useState } from "react";
import { useParams, useNavigate, Link } from "react-router-dom";
import { base44 } from "@/api/base44Client";
import ReactMarkdown from "react-markdown";
import { Button } from "@/components/ui/button";
import { ArrowLeft, Loader2, TrendingUp, Percent, Wallet, Building2, History } from "lucide-react";

const fmt = (n) =>
  n == null || !isFinite(n)
    ? "—"
    : "$" + Number(n).toLocaleString(undefined, { maximumFractionDigits: 0 });

const fmtPct = (n) =>
  n == null || !isFinite(n) ? "—" : Number(n).toFixed(2) + "%";

const verdictStyles = {
  Buy: "bg-emerald-500/15 text-emerald-400 ring-emerald-500/30",
  Hold: "bg-amber-500/15 text-amber-400 ring-amber-500/30",
  Pass: "bg-rose-500/15 text-rose-400 ring-rose-500/30",
};

export default function AnalysisResults() {
  const { id } = useParams();
  const navigate = useNavigate();
  const [record, setRecord] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState("");

  useEffect(() => {
    let active = true;
    (async () => {
      setLoading(true);
      try {
        const rec = await base44.entities.Property.get(id);
        if (!active) return;
        setRecord(rec);
      } catch (err) {
        if (active) setError(err.message || "Could not load analysis.");
      } finally {
        if (active) setLoading(false);
      }
    })();
    return () => { active = false; };
  }, [id]);

  if (loading) {
    return (
      <div className="flex items-center justify-center py-32">
        <Loader2 className="w-7 h-7 text-primary animate-spin" />
      </div>
    );
  }

  if (error || !record) {
    return (
      <div className="px-5 md:px-10 py-12 max-w-3xl mx-auto">
        <p className="text-sm text-muted-foreground">{error || "Analysis not found."}</p>
        <Button variant="outline" className="mt-4" onClick={() => navigate("/")}>Back to Dashboard</Button>
      </div>
    );
  }

  const metrics = [
    { label: "Net Operating Income", value: fmt(record.noi), icon: TrendingUp, hint: "Gross Income − Operating Expenses" },
    { label: "Cap Rate", value: fmtPct(record.cap_rate), icon: Percent, hint: "NOI ÷ Purchase Price" },
    { label: "Cash-on-Cash Return", value: fmtPct(record.cash_on_cash), icon: Wallet, hint: "Mock 25% down / 4% debt structure" },
  ];

  return (
    <div className="px-5 md:px-10 py-8 md:py-12 max-w-5xl mx-auto">
      <div className="flex items-center justify-between mb-8 flex-wrap gap-3">
        <Link to="/" className="inline-flex items-center gap-2 text-sm text-muted-foreground hover:text-foreground transition-colors">
          <ArrowLeft className="w-4 h-4" />
          New analysis
        </Link>
        <Link to="/history">
          <Button variant="outline" size="sm">
            <History className="w-4 h-4" />
            View history
          </Button>
        </Link>
      </div>

      <div className="flex items-start gap-4 mb-8">
        <div className="flex items-center justify-center w-12 h-12 rounded-2xl bg-primary/15 ring-1 ring-primary/30 shrink-0">
          <Building2 className="w-6 h-6 text-primary" />
        </div>
        <div className="min-w-0">
          <h1 className="text-2xl md:text-3xl font-semibold tracking-tight truncate">{record.property_name}</h1>
          <div className="flex items-center gap-2 mt-1.5 flex-wrap">
            <span className="text-xs px-2 py-0.5 rounded-full bg-accent text-accent-foreground">{record.asset_type}</span>
            <span className="text-xs px-2 py-0.5 rounded-full bg-accent text-accent-foreground capitalize">{record.source} input</span>
            {record.verdict && (
              <span className={`text-xs px-2 py-0.5 rounded-full ring-1 font-medium ${verdictStyles[record.verdict] || ""}`}>
                Verdict: {record.verdict}
              </span>
            )}
          </div>
        </div>
      </div>

      {/* Metric cards */}
      <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
        {metrics.map((m) => {
          const Icon = m.icon;
          return (
            <div key={m.label} className="rounded-2xl border border-border bg-card/60 backdrop-blur p-5">
              <div className="flex items-center justify-between mb-3">
                <span className="text-xs text-muted-foreground">{m.label}</span>
                <Icon className="w-4 h-4 text-primary/70" />
              </div>
              <p className="text-2xl md:text-3xl font-semibold tracking-tight">{m.value}</p>
              <p className="text-[11px] text-muted-foreground mt-2">{m.hint}</p>
            </div>
          );
        })}
      </div>

      {/* Deal inputs */}
      <div className="mt-4 rounded-2xl border border-border bg-card/40 p-5 grid grid-cols-2 md:grid-cols-4 gap-4 text-sm">
        <div>
          <p className="text-[11px] text-muted-foreground">Purchase Price</p>
          <p className="font-medium mt-0.5">{fmt(record.purchase_price)}</p>
        </div>
        <div>
          <p className="text-[11px] text-muted-foreground">Gross Rental Income</p>
          <p className="font-medium mt-0.5">{fmt(record.gross_rental_income)}</p>
        </div>
        <div>
          <p className="text-[11px] text-muted-foreground">Operating Expenses</p>
          <p className="font-medium mt-0.5">{fmt(record.operating_expenses)}</p>
        </div>
        <div>
          <p className="text-[11px] text-muted-foreground">Analyzed</p>
          <p className="font-medium mt-0.5">{new Date(record.created_date).toLocaleDateString()}</p>
        </div>
      </div>

      {/* AI analysis */}
      <div className="mt-8">
        <div className="flex items-center gap-2 mb-3">
          <span className="text-xs font-medium uppercase tracking-[0.18em] text-primary">AI Investment Memo</span>
        </div>
        <div className="rounded-2xl border border-border bg-card/60 backdrop-blur p-6 md:p-8 prose-cre">
          <ReactMarkdown
            components={{
              h2: ({ node, ...props }) => <h2 className="text-base font-semibold mt-6 mb-3 first:mt-0 flex items-center gap-2" {...props} />,
              ul: ({ node, ...props }) => <ul className="space-y-2 my-3 list-disc pl-5" {...props} />,
              li: ({ node, ...props }) => <li className="text-sm leading-relaxed text-muted-foreground" {...props} />,
              p: ({ node, ...props }) => <p className="text-sm leading-relaxed text-muted-foreground my-2" {...props} />,
              strong: ({ node, ...props }) => <strong className="text-foreground font-semibold" {...props} />,
            }}
          >
            {record.analysis_markdown || ""}
          </ReactMarkdown>
        </div>
      </div>
    </div>
  );
}
